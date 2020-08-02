[TOC]



## 资料

[从Paxos到Zookeeper](https://book.douban.com/subject/26292004/)



## 基本概念

Paxos，分布式数据一致性解决方案。原语，TCP长连接

- ZooKeeper集群下机器节点的角色
  - Leader
  - Follower
  - Observer

- 数据节点Znode的四种类型
  - 持久节点
  - 临时节点
  - 持久有序节点
  - 临时有序节点
- 会话（Session）
- 版本（Znode上的stat数据结构）
- Watcher（事件监听器）
- ACL策略（Access Control Lists权限控制）下的5种权限
  - CREATE
  - READ
  - WRITE
  - DELETE
  - ADMIN

- ZAB协议的两种模式

  - 恢复模式（故障恢复、选主）
  - 广播模式（数据同步阶段）

  

- ZAB协议的四个阶段

  - 选举
  - 发现
  - 同步
  - 广播

  

- 事务编号Zxid
- epoch

## ZooKeeper典型应用场景

### 分布式锁

![image-20200802144910821](/Users/fanxudong/IdeaProjects/blog/10 分布式系统设计/asset/image-20200802144910821.png)

- 排他锁（Exclusive Locks，X锁，写锁，独占锁）
  - 定义锁：ZK上的数据节点表示一个锁
  - 获取锁：客户端通过调用create接口创建节点，创建成功则获取到锁。没有获取到锁的客户端在父节点注册Watcher监听
  - 释放锁：1）临时节点会在客户端宕机时释放。2）业务正常执行完后，客户端主动删除

- 共享锁（Shared Locks，S锁，读锁）
  - 定义锁：ZK上的数据节点，路径为 业务信息-请求类型-序号
  - 获取锁：读写请求分别创建对应请求类型的节点
  - 释放锁：同排他锁
  - 解决羊群效应：客户端只关注比自己序号小的那个相关节点的变更情况





## ZooKeeper的使用

### Mac下zk的单点模式安装

1. 官网下载tar.gz包：https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz 最好下载3.4.14版本
2. conf目录下添加配置文件zoo.cfg，可以直接拷贝zoo_sample.cfg
3. 直接启动：zkServer.sh  start
4. 检查状态：zkServer.sh  status
5. 使用客户端连接本地zk服务：sh ./zkCli.sh
6. 常用命令create、ls、get、set、delete
7. zk的客户端：zkCli.sh、Java原生Api、ZkClient、Curator







