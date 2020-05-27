Spring的AOP通过动态代理实现的。



## 问题：

1. 考察你对反射机制的了解和掌握程度。
2. 动态代理解决了什么问题，在你业务系统中的应用场景是什么？
3. 1. 包装RPC调用
   2. AOP
4. JDK 动态代理在设计和实现上与 cglib 等方式有什么不同，进而如何取舍？

## 动态代理的实现方式、各自优势：

JDK Proxy：反射、实现接口

CGlib：实现子类、字节码


## 参考：

https://time.geekbang.org/column/article/7489

[05.2 mybatis源码阅读](evernote:///view/7378256/s36/9eb1b321-8c11-4ee9-8550-4cab35ad7eee/9eb1b321-8c11-4ee9-8550-4cab35ad7eee/) mybatis动态代理

[CGLIB动态代理](evernote:///view/7378256/s36/7328c9b5-cf7b-4760-8b8b-286accbec572/7328c9b5-cf7b-4760-8b8b-286accbec572/)

