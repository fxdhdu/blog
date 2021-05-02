[TOC]

## Spring中这么来开启事务



## Spring事务传播机制、隔离级别

spring通过事务传播行为控制当前的事务，如何传播到被嵌套调用的目标服务接口方法中。spring在transactionDefinition接口中规定了7中类型的事务传播行为。它们规定了事务方法和事务方法发生嵌套调用时事务如何进行传播。



事务挂起会发生什么？会释放数据库链接吗？



Spring事务的实现机制：spring事务也是基于数据库事务，通过jdbc来实现的。



TransactionDefinition接口中定义了5种隔离级别。4种和数据库一致。还有一种是ISOLATION_DEFAULT，表示使用数据库默认的隔离级别。

```java
public interface TransactionDefinition {
	  int PROPAGATION_REQUIRED = 0; // 支持当前事务；如果不存在，则创建新事务。
  	int PROPAGATION_SUPPORTS = 1; // 支持当前事务；如果不存在事务，则以非事务方式执行。
  	int PROPAGATION_MANDATORY = 2; //支持当前事务；如果不存在当前事务，则抛出异常。
  	int PROPAGATION_REQUIRES_NEW = 3; // 创建一个新事务，如果已存在事务则挂起
	  int PROPAGATION_NOT_SUPPORTED = 4; // 不支持当前事务，以非事务执行
	  int PROPAGATION_NEVER = 5;         // 不支持当前事务，如果有当前事务则抛出异常
	  int PROPAGATION_NESTED = 6;        // 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。

  	int ISOLATION_DEFAULT = -1;   // 使用数据库默认的隔离级别。
  	int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;
  	int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;
  	int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;
  	int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;

  	int TIMEOUT_DEFAULT = -1;
}
```



## 事务不生效的场景

调用本类的方法，会让注解，还是AOP切面无法注切入（具体看事务管理实现源码）。调用别的类的话，然后就会有事务切入，然后就会有管理。



## 事务实现源码

事务控制是通过aop加在service层上的，而aop底层又是通过jdk动态代理和cglib动态代理实现的。注解事务不生效的场景和jdk动态代理、cglib动态代理的实现有关。



在TransactionSynchronizationManager中有两个方法，分别用来绑定和获得对应线程的事务控制实例。

```java
package org.springframework.transaction.support;

public abstract class TransactionSynchronizationManager {

	public static void bindResource(Object key, Object value) throws IllegalStateException {

  }
  
  public static Object getResource(Object key) {

  }
}
```

