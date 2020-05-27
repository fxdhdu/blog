

[TOC]



## 线程池是一种生产者 - 消费者模式

生产者：线程池使用方

任务队列

消费者：线程池



## 创建线程池参数

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {

}
```



- corePoolSize：表示线程池保有的最小线程数。Java 在 1.6 版本增加了 allowCoreThreadTimeOut(boolean value) 方法，它可以让所有线程都支持超时，允许核心线程超时后关闭。

- maximumPoolSize：表示线程池创建的最大线程数。
- keepAliveTime & unit：如果一个线程空闲了keepAliveTime & unit这么久，而且线程池的线程数大于 corePoolSize ，那么这个空闲的线程就要被回收了。
- workQueue：工作队列。Executors 提供的很多方法默认使用的都是无界的 LinkedBlockingQueue，高负载情境下，无界队列很容易导致 OOM，而 OOM 会导致所有请求都无法处理，这是致命问题。所以强烈建议使用有界队列。
- threadFactory：通过这个参数你可以自定义如何创建线程，例如你可以给线程指定一个有意义的名字。
- handler：通过这个参数你可以自定义任务的拒绝策略。如果线程池中所有的线程都在忙碌，并且工作队列也满了（前提是工作队列是有界队列），那么此时提交任务，线程池就会拒绝接收。至于拒绝的策略，你可以通过 handler 这个参数来指定。ThreadPoolExecutor 已经提供了以下 4 种策略。
  - CallerRunsPolicy：提交任务的线程自己去执行该任务。
  - AbortPolicy：默认的拒绝策略，会 throws RejectedExecutionException。
  - DiscardPolicy：直接丢弃任务，没有任何异常抛出。
  - DiscardOldestPolicy：丢弃最老的任务，其实就是把最早进入工作队列的任务丢弃，然后把新任务加入到工作队列

## 怎么处理线程池中线程抛出的异常

1. 在线程中捕获所有异常并按需处理。



## 线程池一般定多大

- Tomcat 的线程数

- Jdbc 连接池的连接数是多少



### 为什么要使用多线程？

提升程序性能，降低延迟，提高吞吐量。在并发编程领域，提升性能本质上就是提升硬件的利用率，再具体点来说，就是提升 I/O 的利用率和 CPU 的利用率。解决 CPU 和 I/O 设备综合利用率问题，将硬件的性能发挥到极致。



### 多线程的应用场景

要想“降低延迟，提高吞吐量”，对应的方法呢，基本上有两个方向：

优化算法，（算法范畴）

将硬件（I/O， CPU）的性能发挥到极致。（并发编程）



### 创建多少线程合适？

-  I/O 密集型计算

  我们的程序一般都是 CPU 计算和 I/O 操作交叉执行的，由于 I/O 设备的速度相对于 CPU 来说都很慢，所以大部分情况下，I/O 操作执行的时间相对于 CPU 计算来说都非常长。

  理论上：

  ```
  线程的数量 = CPU 核数
  ```

  再多创建线程也只是增加线程切换的成本。

  工程上：

  ```
  线程的数量 = CPU 核数 +1
  ```

  这样的话，当线程因为偶尔的内存页失效或其他原因导致阻塞时，这个额外的线程可以顶上，从而保证 CPU 的利用率。

- CPU 密集型

  CPU 密集型计算大部分场景下都是纯 CPU 计算。

  单核：

  ```
  最佳线程数 = 1 +（I/O 耗时 / CPU 耗时）
  ```

  我们令 R=I/O 耗时 / CPU 耗时，可以这样理解：当线程 A 执行 IO 操作时，另外 R 个线程正好执行完各自的 CPU 计算。这样 CPU 的利用率就达到了 100%。

  多核：

  ```
  最佳线程数 = CPU 核数 * [ 1 +（I/O 耗时 / CPU 耗时）]
  ```

  ![img](/Users/fanxudong/IdeaProjects/blog/4 Java/assert/98b71b72f01baf5f0968c7c3a2102fcb.png)



怎么获得I/O 、CPU耗时比例：压测，重点关注 CPU、I/O 设备的利用率和性能指标（响应时间、吞吐量）之间的关系。

## 参考：

[22 | Executor与线程池：如何创建正确的线程池？](https://time.geekbang.org/column/article/90771)

[10 | Java线程（中）：创建多少线程才是合适的？](https://time.geekbang.org/column/article/86666)

