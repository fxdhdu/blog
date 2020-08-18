[TOC]

> Apache Kafka是一种分布式的、基于发布/订阅的消息系统，由Scala语言编写而成。它具备快速、可扩展、可持久化的特点。

## Kafka的关键特性

- 近乎实时的消息处理能力。高效查询和存储消息。顺序读写磁盘。
- 批量读写消息，批量压缩消息。
- 消息分区，分区中消息保序，分区间可并发操作。
- 在线增加分区，在线水平扩展。
- 数据容灾能力：分区可创建多个副本，Leader副本负责读写，Follwer副本与Leader同步

## Kafka的应用场景

- 作为传统消息中间件：实现消息队列、消息发布/订阅（异步处理/高并发系统的削峰）
- 数据总线：子系统将产生的数据发送到kafka，之后流转到目的系统。（OPS监控系统/模块之间的解耦）
- 日志收集中心
- 基于Kafka设计数据库主从同步工具

## 以Kafka为中心的解决方案

- 解耦合

- 数据持久化：提供日志清理、日志压缩功能避免数据不断增长。顺序读写磁盘。

- 扩展与容灾
  - 消息冗余备份：每个Topic多个Partition，每个Partition多个Replica（副本）
  - 提高分区数量，可以实现水平扩展。提高副本数量，可以提高容灾能力。
  - 由Consumer决定何时、如何消费消息。Consumer使用pull方式拉取消息，并在Consumer端保存消费的位置。
  - 支持Consumer水平扩展：多个Consumer加入一个Consumer Group。一个Consumer中，每个分区分配给一个Consumer。

- 顺序保证

- 缓冲&峰值处理能力

- 异步通行

## Kafka核心概念

- 消息：
  - 由一串byte数组构成。主要包含key和value。key用来根据一定策略，将消息路由到指定的分区中。

- Topic&分区&Log：
  - 同一topic下不同分区消息不同。消息在分区中的唯一编号为offset，offset用于分区内的保序。
  - 同一topic的不同分区会分配在不同的broker。
  - 分区逻辑上对应一个Log，Log可以对应磁盘上一个文件夹。Log由多个Segment组成，每个Segment对应一个日志文件和索引文件。索引文件采用稀疏索引方式。

- 保留策略&日志压缩
  - 保留策略：根据消息保留的时间；根据Topic存储的数据大小
  - 日志压缩：在后台启动一个线程，定期将key相同的消息合并，只保留最新的value值

- Broker
  - 一个单独的kafka server就是一个broker，生产环境中，一个broker独占一台物理服务器。
  - broker的主要工作：接收生产者发送的消息，分配offset，保存到磁盘。接收消息者和其他broker的请求。

- 副本
  - 每个Partition可以有多个副本，副本中的消息是一样的
  - Leader副本处理读写请求，Follower副本从Leader副本拉取数据，更新到自己的Log中。一般情况下一个分区的多个副本被分配到不同的Broker上。

- ISR集合（Kafka通过ISR集合，HW和LEO设计解决了同步复制和异步复制两种方案的缺陷）
  - 目前“可用”且消息量与Leader相差不多的副本集合。
    - 副本所在节点必须维持着与Zookeeper的连接
    - 副本最后一条消息的offset与Leader副本的最后一条消息的offset之间差值不能超过阈值。

- HW&LEO
  - HW是一个特殊的offset，由Leader副本管理，消费者拉取消息时，只能拉取HW之前的消息。HW之前的消息状态被称为commit，这些消息在多个副本中同时存在。全部的Follower拉取HW指定的消息后，Leader副本递增HW的值。
  - LEO（Log Eng Offset）：每个副本都有，表示当前副本最后一个消息的offset
- 同步复制
  - 所有工作的Follwer副本复制完，才算消息提交成功。一个Follwer故障就会拖垮整个系统。
- 异步复制
  - Leader副本收到消息后，就认为消息提交成功。Leader副本宕机会出现消息丢失，状态不可控。

- Cluster&Controller
  - 多个Broker组成一个Cluster
  - 每个Cluster选举一个Broker担任Controller，负责管理分区状态、分区副本状态、监听Zookeeper中数据的变化。
  - Controller Leader、Controller Follower

- 生产者
  - 分区选择策略：根据key的hash值，轮训

- 消费者
  - 消费者自己维护所消费分区的offset

- Consumer Group（逻辑上的订阅者）
  - 用于实现消息的广播、独占模式
  - 实现消费者水平扩展、故障转移



**消息生产过程**

**消费者加入Consumer Group的过程**

![image-20200726111114684](/Users/fanxudong/IdeaProjects/blog/10 分布式系统设计/asset/image-20200726111114684.png)



### kafka的高性能、高可靠

![image-20200704190907521](/Users/fanxudong/Library/Application Support/typora-user-images/image-20200704190907521.png)



> 生产者消息分区。Kafka的消息组织方式实际上为一个3级结构，topic、分区、消息。对同一个topic下的数据进行分区，不同的分区可以放置到不同的机器节点上，消息的读写以分区为单位进行。分区可提升集群的可伸缩性、进行负载均衡，提高吞吐量。（分而治之）
>
> 消息压缩，通过生产者端一定的CPU消耗，来减少Broker端的磁盘空间消耗和网络传输带宽。可以在生产者和Broker端指定压缩策略。
>
> Cache Filesystem Cache PageCache缓存：使用操作系统页缓存来存储数据。
>
> 顺序读写磁盘，加快消息从磁盘写入和读区： 由于现代的操作系统提供了预读和写技术，磁盘的顺序写大多数情况下比随机写内存还要快。
>
> Zero-copy 零拷技术加快消费：服务端处理消费消息的逻辑是：文件 -- PageCache -- 应用程序内存 --Socket缓冲区。使用零拷贝技术时，直接把PageCache 中把数据复制到 Socket 缓冲区。由于不用把数据复制到用户内存空间，DMA 控制器可以直接完成数据复制，不需要 CPU 参与，速度更快。
>
> Batching of Messages 批量量处理，批量发送和处理消息。合并小的请求，然后以流的方式进行交互，直顶网络上限。客户端发送请求时，kafka先把消息存入缓存，然后异步批量发送。
>
> Pull 拉模式 使用拉模式进行消息的获取消费，与消费端处理能力相符。



> 怎么不丢失消息、不重复发送消息、不重复消费消息：数据备份、生产消息幂等、事务。对已提交的数据做有限度的持久化保存。
>
> 利用操作系统的缓存原理达到高性能，可分区，分布式，有不同语言的客户端







### kafka的分区策略

轮询策略

随机策略

按消息键保序策略

### 压缩算法（吞吐量和压缩比 的比较）

GZIP

zstd 压缩（建议）

## 消费者

消费者提交offset到offsets topic，消费者上下线时，consumer group rebalance操作，分区重新分配后，消费者可以读取offsets topic中记录的offset。

消费者可以自动提交offset、手动同步提交、手动异步提交

### 传递保证语义

1. <= 1 (At most once)

   消费者拉取消息后，先提交offset，再处理消息。可能会导致消息丢失。

2. 》=1（At least once）

   消费者拉取消息后，先处理消息，再提交offset。可能会导致消息重复处理。

3. ==1（Exactly once）

   1. 生产者不产生重复消息
      1. 一个分区一个生产者写入，异常时，生产者查询此分区最后一个消息，决定是否重传。
      2. 为每个消息添加一个全局唯一主键，异常时，生产者重传，消费者去重。（业务消息有合适的主键，或用一个全局ID生成器）
   2. 消费者不能重复拉取相同的消息。
      1. 消费者关闭自动提交offset的功能，且不再手动提交offset，这样就不使用offsets topic这个内部topic记录其offset，而由消费者自己保存offset。利用事务的原子性来实现exactly once语义。将offset和消息处理结果放到一个事务中，事务执行成功则认为此消息被消费，事务回滚则需要重新消费。当消费者宕机重启或rebalance操作时，消费者从关系型数据库中找到对应offset，然后调用KafkaConsumer.seek方法手动设置消费位置，进行消费。
      2. ConsumerRebalanceListener
         1. onPartitionRevoked：在拉取数据之后，rebalance开始之前手动提交offset
         2. onPartitionAssigned:在Rebalance之后，拉取数据之前，调整topic。

## 参考

[15 | Kafka如何实现高性能IO？](https://time.geekbang.org/column/article/126493)

[Kafka核心技术与实战](https://time.geekbang.org/column/intro/100029201)

[《Apache Kafka 源码剖析》](https://book.douban.com/subject/27038473/)

[kafka：简介、使用场景、设计原理、主要配置及集群搭建](https://www.cnblogs.com/likehua/p/3999538.html)

