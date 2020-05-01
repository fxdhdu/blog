[TOC]



## Java异常继承体系

![Java异常继承体系](./assert/C43DB5C8-7A74-469B-AACF-6BE37B3395E4.png)

- unchecked exceptions （运行时异常）
  
  - 派生于Error的错误不用处理，因为通常代码无法处理。
  - RuntimeException异常不用处理。
  - 如果出现了RuntimeException，就一定是程序员自身的问题。
  
  
  
- checked exceptions
  
  - 所有其他的异常。
  - Checked Exception 的假设是我们捕获了异常，然后恢复程序。
  - 编译器将检查你是否为所有的checked exceptions提供了异常处理机制。
  - 比如IOException就必须做异常处理，否则编译无法通过。
   ![image-20200501183447589](/Users/fanxudong/IdeaProjects/blog/4 Java/assert/image-20200501183447589.png)
  
  

## 从性能角度来审视 Java 的异常处理机制

- try-catch会影响 JVM 对代码进行优化。仅捕获有必要的代码段
- 不要用异常控制代码流程。
- Java 每实例化一个 Exception，都会对当时的栈进行快照。



## 问题

1. Exception和Error有什么区别？（NoClassDefFoundError 和 ClassNotFoundException 有什么区别？）
   1） 是否尝试恢复程序
   2） NoClassDefFoundError通常由maven未引入对应jar包导致，代码无法处理此异常。
   3） ClassNotFoundException通常由Class.forName方法来动态地加载类时，找不到类导致。

   

2. 运行时异常与一般异常的区别？

   1）运行时异常（RuntimeException）不用做异常处理，比如NullPointerException，通常是由代码bug导致，需要抛出。

   2）一般异常需要程序员在代码中捕获并处理，比如IOException。有时候也会捕获一般异常，并抛出RuntimeException。

   

## 参考

[详解Java中的checked异常和unchecked异常](https://blog.csdn.net/qq_14982047/article/details/50989761)

[第2讲 | Exception和Error有什么区别？](https://time.geekbang.org/column/article/6849)

