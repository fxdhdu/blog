[TOC]

## Zookeeper的基本概念

- ZooKeeper集群下机器节点的角色
  - Leader：为客户端提供读写服务
  - Follower：提供读服务
  - Observer：提供读服务

- 数据节点Znode的四种类型
  - 持久节点：只能通过delete来进行删除
  - 临时节点：创建该节点的客户端崩溃或关闭了与zk的连接时，节点删除。（会话超时删除、主动关闭删除、主动删除3中删除方式）
  - 持久有序节点
  - 临时有序节点
  
- 会话（Session）

- 版本（Znode上的stat数据结构）

- Watcher（事件监听器）：基于通知机制代替客户端的轮询，客户端向zk注册需要接收通知的znode，通过对znode设置监视点来接收通知。

- ACL策略（Access Control Lists权限控制）下的5种权限
  - CREATE
  - READ
  - WRITE
  - DELETE
  - ADMIN

## Paxos协议

分布式数据一致性解决方案。原语，

## ZAB协议

zk atomic 广播协议，zk集群中的数据一致性通过zab协议来保证

- ZAB协议的两种模式

  - 恢复模式（故障恢复、选主）
  - 广播模式（数据同步阶段）

  

- ZAB协议的四个阶段

  - 选举
  - 发现
  - 同步
  - 广播

  

- 事务编号Zxid：zk为每个请求分配一个全局唯一的递增编号

- epoch


## ZooKeeper服务器的运行模式

独立模式

仲裁模式

## ZooKeeper的会话机制

ZooKeeper中client与server通过TCP长连接进行通行。使用TCP长连接后无论是否传输数据，连接都会保持，减少了3次握手和4次挥手的开销。ZooKeeper能够使用长连接是因为，通常一个client只与一个server保持连接（客户端只打开一个会话），无论多么平凡的数据交互，都只要一个长连接即可。客户端的请求将全部以FIFO顺序执行。TCP协议通过保活定时器（Keepalive Timer）保持长连接。[ZooKeeper的会话机制Session](https://blog.csdn.net/muerhuoxu/article/details/86218115)

## ZooKeeper典型应用场景

### 分布式锁

### Kafka中的使用

保存原数据，检测崩溃，实现topic的发现

### 作为dubbo的注册中心。



## ZooKeeper实践

### Mac下zk的单点模式安装

1. 官网下载tar.gz包：https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz 最好下载3.4.14版本

2. conf目录下添加配置文件zoo.cfg，可以直接拷贝zoo_sample.cfg

3. 直接启动：sh ./zkServer.sh  start

4. 检查状态：sh ./zkServer.sh  status

5. 停止：sh ./zkServer.sh  stop

6. 使用客户端连接本地zk服务：sh ./zkCli.sh

7. 常用命令create、ls、get、set、delete

   查看dubbo路径

   ```shell
   ls /dubbo
   ```

   退出

   ```shell
   quit
   ```

8. zk的客户端：zkCli.sh、Java原生Api、ZkClient、Curator

## 参考

[从Paxos到Zookeeper](https://book.douban.com/subject/26292004/)
