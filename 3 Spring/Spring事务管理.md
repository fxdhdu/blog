## Spring事务传播机制

spring通过事务传播行为控制当前的事务，如何传播到被嵌套调用的目标服务接口方法中。spring在transactionDefinition接口中规定了7中类型的事务传播行为。它们规定了事务方法和事务方法发生嵌套调用时事务如何进行传播。



事务挂起会发生什么？会释放数据库链接吗？



Spring事务的实现机制：spring事务也是基于数据库事务，通过jdbc来实现的。



## 事务不生效的场景

调用本类的方法，会让注解，还是AOP切面无法注切入（具体看事务管理实现源码）。调用别的类的话，然后就会有事务切入，然后就会有管理。



## 事务实现源码

事务控制是通过aop加在service层上的



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

