[TOC]

Seata的全称: Simple Extensible Autonomous Transaction Architecture，最初开源时叫Fescar。

特点：高性能、零侵入



![seata架构图](./asset/seata架构图.png)

## 分布式事务概念

一个分布式事务是一种全局的事务，它由一系列分支事务或者说本地事务组成。



## Seata的3个基本组件

- Transaction Coordinator(TC)：事务协调者，管理全局和分支事务的状态，用于全局性事务的提交和回滚。
- Transaction Manager(TM)：事务管理器，用于开启全局事务、提交或者回滚全局事务，是全局事务的开启者。
- Resource Manager(RM)：资源管理器，用于分支事务上的资源管理，向TC注册分支事务，上报分支事务的状态，接受TC的命令来提交或回滚分支事务。



## 典型的，由Seata管理的分布式事务生命周期：

1. TM事务管理器，告诉TC事务协调者开始一个新的全局事务。TC产生一个XID表示一个全局事务。
2. XID通过微服务调用链传播
3. RM资源管理器，注册本地事务作为一个分支事务（XID）注册到TC。
4. TM通知TC提交或者回滚XID表示的全局事务。
5. TC控制XID表示的全局事务下所有分支事务提交或回滚。



## seata提供的事务模式

### AT 模式

#### 先决条件

- 关系型数据库支持本地ACID特性。
- Java应用通过JDBC访问数据库。

`UNDO_LOG` table is required by SEATA AT mode.

```sql
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

#### 整体机制

从二阶段提交协议演变过来

- 阶段一：在同一个本地事务中，commit业务数据和回滚日志，然后释放锁和连接资源。
- 阶段二：
  - 对于commit场景，异步地、快速地完成
  - 对于回滚场景，基于阶段一创建的回滚日志进行补偿。

### TCC 模式

回顾一下描述：一个分布式全局事务，整体上是一个两阶段提交模型。全局事务由多个分支事务组成。分支事务必须满足两阶段提交模型，每个分支事务必须有其：

- 一阶段准备行为
- 二阶段提交或回滚行为

对于两阶段行为模式，我们把分支事务分为原子事务模式和TCC事务模式。



TCC模式不依赖底层数据源的事务支持：

- 一阶段准备行为：调用用户的准备逻辑
- 二阶段提交行为：调用用户的提交逻辑
- 二阶段回滚行为：调用用户的回滚逻辑

所谓的TCC模式，即将用户的分支事务提交到全局事务管理器。

### SAGA 模式

saga模式是长事务解决方案

### XA



## 使用场景

用户购买商品，整个业务涉及3个微服务。库存服务、工单服务、账户服务

创建一个数据库，配置3个数据源

用Seata来保证Dubbo微服务间的数据一致性。



seata的配置文件入口为[registry.conf](https://github.com/seata/seata/blob/develop/config/src/main/resources/registry.conf)对应代码为[ConfigurationFactory](https://github.com/seata/seata/blob/develop/config/src/main/java/com/alibaba/fescar/config/ConfigurationFactory.java)





## 参考

[seata github](https://github.com/seata/seata)

[quick start](http://seata.io/en-us/docs/user/quickstart.html)



