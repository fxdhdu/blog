[TOC]



## Java 1.0 ～1.4



## Java 1.5 / Java 5

ReentrantLock

泛型、自动装箱、注释处理、可变参数函数

轻量锁与偏向锁

ReentrantLock

线程状态定义：java.lang.Thread.State



## Java 8

metaspace

Lambda

Stream

允许接口实现默认方法

```java
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
```





## Java 9

List.of()

Set.of()