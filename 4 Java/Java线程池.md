

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



## 参考：

[22 | Executor与线程池：如何创建正确的线程池？](https://time.geekbang.org/column/article/90771)

