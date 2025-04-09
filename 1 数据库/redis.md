[TOC]

Redis采用的是基于内存的采用的是**单进程单线程**模型的 **KV 数据库**，**由C语言编写**，官方提供的数据是可以达到100000+的QPS（每秒内查询次数）。

1、redis完全基于内存，绝大部分请求是存粹的内存操作。

2、专门设计的底层数据结构，结构简单、操作简单。比如跳表等。

3、单线程处理网络请求，避免上下文切换和竞争条件。但是比较耗时的命令会导致并法度下降。在多核服务器上也可以启动多个Redis实例，组成master-master或master-slave形式，耗时命令在slave上执行。Redis 4.0开始在某些操作上支持多线程。

4、多路I/O复用模型、非阻塞IO。对多个网络链接（多个客户端），使用同一个线程处理IO事件。

6、自己构建了vm机制？

Redis的性能瓶颈不是CPU，最可能在机器内存大小或网络带宽。



## 参考

[为什么说Redis是单线程的以及Redis为什么这么快！](https://blog.csdn.net/xlgen157387/article/details/79470556?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161502687816780255220652%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161502687816780255220652&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-79470556.first_rank_v2_pc_rank_v29&utm_term=redis%E5%8D%95%E7%BA%BF%E7%A8%8B)

[官方文章How fast is Redis?](https://redis.io/topics/benchmarks)

渐进式 rehash：把一次性大量拷贝的开销，分摊到了多次处理请求的过程中，避免了耗时操作，保证了数据的快速访问。

跳表：多级索引

5.redis是单线程的吗，在redis写入的时候会不会对读阻塞

redis的string类型的扩容机制是怎样的

[字节跳动极高可用 KV 存储系统详解](https://zhuanlan.zhihu.com/p/614227806) 磁盘存储KV，容量比内存KV大，牺牲一点延迟。
[Abase2：字节跳动新一代高可用 NoSQL 数据库](https://maimai.cn/article/detail?fid=1735122374&efid=3dnKOpRsrT5HD6dYMVu0yA)