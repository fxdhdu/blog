[TOC]

## 使用场景

### 适合使用缓存的场景

- 读密集型的应用
- 存在热数据的应用
- 对响应实效要求较高
- 对一致性要求不高
- 需要实现分布式锁的时候

### 不适合使用缓存的场景

- 读少
- 更新频繁
- 对一致性要求严格



## 缓存在计算机中的应用

| 缓存级别             | 案例                                           |
| -------------------- | ---------------------------------------------- |
| 分布式缓存           | Redis                                          |
| 进程本地缓存         | MyBatis缓存、MySql内存索引                     |
| 内存                 | JVM                                            |
| L1\L2\L3             | L1D缓存数据、L1I缓存指令，L3缓存与内存同步数据 |
| 寄存器（1个CPU周期） |                                                |

对于CPU密集型的运算越少访问缓存越好

CPU通过定义内存类型及一致性协议来解决多核心下的性能和一致性问题



## 分布式缓存所面临的问题

### 缓存高可用

- 基于分片的主从实现缓存。
  - 通过分片来分割大数据的查询
  - 通过主从来完成高可用和高性能需求
  - 通过多个副本化解查询带来的性能压力
- 异步复制主从模型
- 主从复制

### 缓存高并发

纯网络IO问题

Mysql命中内存索引的情况下达到10万每秒QPS，仅仅是内存和网络操作。

分析缓存依赖，没有依赖关系的缓存访问并行执行，有依赖关系的保留串行执行。



## 常用分布式缓存实现方案

## Redis

数据结构丰富且简单易用

| 分布式缓存   | 数据结构                                               | 存储结构                | 线程模型                                                     | 持久                                                         | 高可用                              |
| ------------ | ------------------------------------------------------ | ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------- |
| Redis        | string String<br>list <br/>hash <br/>set<br/>zset<br/> | 跳表<br>字典<b r>压缩串 | 单线程。存储太大内容会阻塞其他请求。<br/>单线程非阻塞网络IO模型 | RDB：定时持久，宕机时可能会出现数据丢失<br/>AOF：基于操作日志的持久机制 | 主从<br>Sentinel（哨兵）<br>Cluster |
| Memcache     | key / value                                            |                         | 多线程                                                       | 依赖第三方组件（MemcacheDB）                                 | 第三方组件                          |
| Tair（阿里） | 同Redis                                                |                         | 多线程                                                       | 由存储结构决定                                               | Cluster                             |



| 分布式缓存   | 事务 | 数据淘汰策略 |      |      |      |
| ------------ | ---- | ------------ | ---- | ---- | ---- |
| Redis        |      |              |      |      |      |
| Memcache     |      | LRU算法      |      |      |      |
| Tair（阿里） |      |              |      |      |      |



Redis主从节点复制：从节点使用RDB和缓存的AOF命令进行同步和恢复。

Redis高可用方案：哨兵（集群管理工具），负责在节点中选出主节点，按照分布式集群的管理办法来操作集群节点的上线、下线、监控、提醒、自动故障切换（主备切换），实现了著名的RAFT选主协议，保证系统选主的一致性。

哨兵节点与主节点保持通行，

Redis异步复制（集群非强一致），数据到主节点后，主节点返回成功，数据被异步复制给从节点。

Redis集群分片机制



### 应用层访问缓存的模式

#### 双读双写

读操作：先读缓存，缓存不存在，则读数据库，读取数据库后再回写缓存

写操作：先写数据库，再写缓存

应用层需要处理读写顺序逻辑

#### 异步更新

应用层只读写缓存，全量数据保存在缓存中，不设置缓存系统过期时间。由异步更新服务将数据库里变更的或者新增的数据更新到缓存中。

#### 串联模式

应用直接读写缓存，缓存作为代理层与数据库进行读写操作



### 分布式缓存分片的三种模式

#### 客户端分片

#### 代理分片

#### 集群分片





## 怎么保证缓存与数据库的双写一致性

**项目中遇到的使用缓存的场景**

- 系统配置（低并发场景、系统本地缓存、存储数据较少）

**双写**

缓存与数据库双存储，更新数据库，既要写缓存又要写数据库，称为双写。

**考虑一致性前，先要思考我的系统需要严格的一致性吗？**

 本地系统配置不需要强一致性。严格的一致性会降低读操作的吞吐量。而缓存通常是为了提高读性能，降低数据库的压力。

**系统配置缓存和数据库怎么保证一致性（双存储操作，单一失败导致不一致；并发操作导致不一致）**

- 读请求、写请求通过内存队列（jvm队列）串行化（通过一个线程来处理）。需要根据所操作的数据作为唯一标识，同一个数据的操作路由到同一个节点去处理（Nginx的hash路由）。
- 异步化导致高并发读取时超时，更新操作挤压。需要通过压力测试，增加机器的方式来解决。

**Cache Aside Pattern（缓存模式）**

- 新增配置时：缓存新增，数据库中新增，什么先后顺序比较好。先新增数据库，再新增缓存，防止增加缓存后系统异常，导致数据丢失。
- 更新配置时：更新数据库，再**删除**缓存。（lazy计算思想）
- 删除配置时：先删除缓存，再删除数据库。
- 查询配置时：先读缓存，缓存中没有则读取数据库，并更新到缓存中。

**lazy计算思想**

缓存在集群中的一致性怎么保证

- 集群部署时，每个节点都有自己的缓存，需要节点间保持一致吗？

并发的量级怎么计算

- 每秒的 QPS

有什么缓存中间件可以替代系统本地缓存





这篇文章，我要把里面的关键概念都搞懂

然后画出里面方案的架构图

并最终把里面一些思想用到平时的工作中





## 相关文章

https://mp.weixin.qq.com/s/LMBeCGM-ZPfpIKndOUeCGQ

https://blog.csdn.net/chang384915878/article/details/86756463



