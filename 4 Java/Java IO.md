FileWriter：线程安全write

File

|        | 字节流       | 字符流 |
| ------ | ------------ | ------ |
| 输入流 | InputStream  | Reader |
| 输出流 | OutputStream | Writer |





## java.io 基于流模型实现，交互方式是同步、阻塞

BIO（Blocking I/O，阻塞IO）

File 抽象

输入输出流

 java.net



## java.nio 多路复用的、同步非阻塞

NIO（Nonblocking I/O，非阻塞IO）：NIO的单线程能处理连接的数量比BIO要高出很多

Channel

Selector

Buffer



## NIO 2 异步非阻塞，基于事件和回调机制





基础 API 功能与设计， InputStream/OutputStream 和 Reader/Writer 的关系和区别。

NIO、NIO 2 的基本组成。

给定场景，分别用不同模型实现，分析 BIO、NIO 等模式的设计和实现原理。

NIO 提供的高性能数据操作方式是基于什么原理，如何使用？

或者，从开发者的角度来看，你觉得 NIO 自身实现存在哪些问题？有什么改进的想法吗？



## 参考

[Netty入门教程——认识Netty](https://www.jianshu.com/p/b9f3f6a16911)

