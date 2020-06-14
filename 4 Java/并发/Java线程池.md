

[TOC]



- 线程池是一种生产者 - 消费者模式
  生产者：线程池使用方
任务队列、阻塞队列（队列满时，插入操作会阻塞。队列空时，获取元素操作会阻塞）
消费者：线程池



- 为什么要使用多线程？
  提升程序性能，降低延迟，提高吞吐量。在并发编程领域，提升性能本质上就是提升硬件的利用率，再具体点来说，就是提升 I/O 的利用率和 CPU 的利用率。解决 CPU 和 I/O 设备综合利用率问题，将硬件的性能发挥到极致。



- 多线程的应用场景
  要想“降低延迟，提高吞吐量”，对应的方法呢，基本上有两个方向：
优化算法，（算法范畴）
将硬件（I/O， CPU）的性能发挥到极致。（并发编程）



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



| 参数名               | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| corePoolSize         | 表示线程池保有的最小线程数。Java 在 1.6 版本增加了 allowCoreThreadTimeOut(boolean value) 方法，它可以让所有线程都支持超时，允许核心线程超时后关闭。<br> |
| maximumPoolSize      | 表示线程池创建的最大线程数。                                 |
| keepAliveTime & unit | 如果一个线程空闲了keepAliveTime & unit这么久，而且线程池的线程数大于 corePoolSize ，那么这个空闲的线程就要被回收了。 |
| workQueue            | 工作队列。Executors 提供的很多方法默认使用的都是无界的 LinkedBlockingQueue，高负载情境下，无界队列很容易导致 OOM，而 OOM 会导致所有请求都无法处理，这是致命问题。所以强烈建议使用有界队列。 |
| threadFactory        | 通过这个参数你可以自定义如何创建线程，例如你可以给线程指定一个有意义的名字。 |
| handler              | 通过这个参数你可以自定义任务的拒绝策略。如果线程池中所有的线程都在忙碌，并且工作队列也满了（前提是工作队列是有界队列），那么此时提交任务，线程池就会拒绝接收。至于拒绝的策略，你可以通过 handler 这个参数来指定。ThreadPoolExecutor 已经提供了以下 4 种策略。 |

### 拒绝策略接口RejectedExecutionHandler

| 拒绝策略（实现）    | 描述                                                         |
| ------------------- | :----------------------------------------------------------- |
| CallerRunsPolicy    | 提交任务的线程自己去执行该任务。                             |
| AbortPolicy         | 默认的拒绝策略，会 throws RejectedExecutionException。       |
| DiscardPolicy       | 直接丢弃任务，没有任何异常抛出。                             |
| DiscardOldestPolicy | 丢弃最老的任务，其实就是把最早进入工作队列的任务丢弃（队列头元素），然后把新任务加入到工作队列 |



### 线程池一般定多大，创建多少线程合适？

### （tomcat线程池、jdbc线程池）

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



## 线程池源码分析、实现原理（JDK 1.8）

- ctl变量

  对线程池的运行状态和池子中有效线程的数量进行控制.

  ctl 变量包含两部分信息: 线程池的运行状态 (runState) 和线程池内有效线程的数量 (workerCount).

  ctl 的高3位表示线程池的运行状态, 低29位表示线程池内有效线程的数量. 

- 常用的变量缩写

  rs：表示线程池运行状态

  wc：表示线程池中有效线程数量

  c：ctl

```java
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

rs、wc、c之间的转换

```java
    private static int runStateOf(int c)     { return c & ~COUNT_MASK; } // 计算rs的值
    private static int workerCountOf(int c)  { return c & COUNT_MASK; } // 计算wc的值
    private static int ctlOf(int rs, int wc) { return rs | wc; } // 计算ctl 的数值

    private static final int COUNT_BITS = Integer.SIZE - 3; // 32 - 3 = 29
    private static final int COUNT_MASK = (1 << COUNT_BITS) - 1;
```

### 线程池的五种状态（从小到大顺序）

```java
    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
```

| 状态                    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| **RUNNING (运行状态)**  | 能接受新提交的任务, 并且也能处理阻塞队列中的任务.            |
| **SHUTDOWN (关闭状态)** | 不再接受新提交的任务, 但却可以继续处理阻塞队列中已保存的任务. 在线程池处于 RUNNING 状态时, 调用 shutdown()方法会使线程池进入到该状态. 当然, finalize() 方法在执行过程中或许也会隐式地进入该状态. |
| **STOP**                | 不能接受新提交的任务, 也不能处理阻塞队列中已保存的任务, 并且会中断正在处理中的任务. 在线程池处于 RUNNING 或 SHUTDOWN 状态时, 调用 shutdownNow() 方法会使线程池进入到该状态. |
| **TIDYING (清理状态)**  | 所有的任务都已终止了, workerCount (有效线程数) 为0, 线程池进入该状态后会调用 terminated() 方法以让该线程池进入TERMINATED 状态. 当线程池处于 SHUTDOWN 状态时, 如果此后线程池内没有线程了并且阻塞队列内也没有待执行的任务了 (即: 二者都为空), 线程池就会进入到该状态. 当线程池处于 STOP 状态时, 如果此后线程池内没有线程了, 线程池就会进入到该状态. |
| **TERMINATED**          | terminated() 方法执行完后就进入该状态.                       |

<img src="/Users/fanxudong/IdeaProjects/blog/4 Java/assert/image-20200613233849383.png" alt="image-20200613233849383" style="zoom:50%;" />

### execute方法

```java
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
            
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```

> 什么时候创建线程？
>
> 什么时候任务进入队列？

创建corePoolSize个线程 -》 阻塞队列塞满 -> 创建到maximumPoolSize个线程 -》执行拒绝策略

1. 如果线程池内的有效线程数少于核心线程数 corePoolSize, 那么就创建并启动一个线程来执行新提交的任务.
2. 如果线程池内的有效线程数达到了核心线程数 corePoolSize, 并且线程池内的阻塞队列未满, 那么就将新提交的任务加入到该阻塞队列中.
3. 如果线程池内的有效线程数达到了核心线程数 corePoolSize 但却小于最大线程数 maximumPoolSize, 并且线程池内的阻塞队列已满, 那么就创建并启动一个线程来执行新提交的任务.
4. 如果线程池内的有效线程数达到了最大线程数 maximumPoolSize, 并且线程池内的阻塞队列已满, 那么就让 RejectedExecutionHandler 根据它的拒绝策略来处理该任务, 默认的处理方式是直接抛异常.

### addWorker方法

### 内部类Worker，及其runWorker方法

> 启动一个 Worker对象中包含的线程 thread, 就相当于要执行 runWorker()方法, 并将该 Worker对象作为该方法的参数.

### getTask方法

无线循环获取任务





### 什么时候执行拒绝策略？

1. 线程池处于SHUTDOWN状态时。（线程池的 isShutdown() 方法返回 true ）
2. 当线程池中处于运行状态的线程数达到maximumPoolSize，且阻塞队列满时, 继续提交的任务就会被拒绝.



## 题外话

### 怎么处理线程池中线程抛出的异常

1. 在线程中捕获所有异常并按需处理。



## 参考：

[22 | Executor与线程池：如何创建正确的线程池？](https://time.geekbang.org/column/article/90771)

[10 | Java线程（中）：创建多少线程才是合适的？](https://time.geekbang.org/column/article/86666)

[Java 线程池 ThreadPoolExecutor 源码分析](http://blog.csdn.net/clevergump/article/details/50688008)

