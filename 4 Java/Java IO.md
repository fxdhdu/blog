FileWriter：线程安全write

File

|        | 字节流       | 字符流 |
| ------ | ------------ | ------ |
| 输入流 | InputStream  | Reader |
| 输出流 | OutputStream | Writer |





同步：等待

异步：回调

阻塞：CPU等待

非阻塞：轮询



## java.io 基于流模型实现，交互方式是同步、阻塞

BIO（Blocking I/O，阻塞IO）

File 抽象

输入输出流

 java.net



## java.nio 多路复用的、同步非阻塞

NIO（Nonblocking I/O，非阻塞IO）：NIO的单线程能处理连接的数量比BIO要高出很多

### Channel

通道是数据来源或数据写入的目的地

![](./assert/channel.png)

- FileChannel：文件通道，用于文件的读和写
- DatagramChannel：用于 UDP 连接的接收和发送
- SocketChannel：把它理解为 TCP 连接通道，简单理解就是 TCP 客户端
- ServerSocketChannel：TCP 对应的服务端，用于监听某个端口进来的请求

### Selector

### Buffer

一个 Buffer 本质上是内存中的一块，我们可以将数据写入这块内存，之后从这块内存获取数据。

![](./assert/6.png)



ByteBuffer：核心

MappedByteBuffer 用于实现内存映射文件

position

limit

capacity

mark



写入模式

读出模式

## NIO 2 异步非阻塞，基于事件和回调机制





基础 API 功能与设计， InputStream/OutputStream 和 Reader/Writer 的关系和区别。

NIO、NIO 2 的基本组成。

给定场景，分别用不同模型实现，分析 BIO、NIO 等模式的设计和实现原理。

NIO 提供的高性能数据操作方式是基于什么原理，如何使用？

或者，从开发者的角度来看，你觉得 NIO 自身实现存在哪些问题？有什么改进的想法吗？



## 参考

