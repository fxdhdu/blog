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

