[TOC]

| synchronized                                                | ReentrantLock                                                |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| java内建同步机制，原生语法。                                | java.util.concurrent.locks （J.U.C）包中提供。api层面的互斥锁 |
| 悲观锁，阻塞同步                                            | 悲观锁，阻塞同步                                             |
| 无法控制公平性，其为非公平锁。                              | 可以控制 fairness(公平性)，默认为非公平。公平锁是指多个线程同时等待同一个锁时，倾向于将锁赋予等待时间最久的线程（按照申请锁的时间顺序来依次获得锁）。可减少线程“饥饿”，但会降低吞吐量。 |
| 偏斜锁（可重入）                                            | 再入锁，当一个线程试图获取一个它已经获取的锁时，这个获取动作就自动成功。锁的持有是以线程为单位。 |
| 可修饰实例方法、类方法（静态）                              | lock/unlock/try/finally                                      |
|                                                             | 等待可中断：当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待。 |
| 锁对象的wait、notify、notifyAll方法可以实现一个隐含的条件。 | 锁可以绑定多个条件。一个ReentrantLock对象可以同时绑定多个Condition对象，只需多次调用newCondition方法即可。 |

## synchronized

- 提供了互斥的语义和可见性
- 修饰静态方法，锁对象为类的Class对象（ClassName.class，永久代/Metaspace中）
- 修饰非静态方法，锁对象为this （当前类实例）
- 修饰代码块

### synchronized底层如何实现

synchronized关键字经过编译以后，会在同步块的前后分别形成monitorenter/monitorexit这两个字节码指令。

Monitor 对象的三种不同的实现（实现代码在JVM中）

- 偏斜锁（Biased Locking）


- 轻量级锁

- 重量级锁



### JDK 1.6对锁的优化（也是synchronized的实现机制）

#### 自旋锁与自适应自旋

线程获取锁失败时，通过自旋的方式等待一定时间，再尝试获取锁。以消耗一定的CPU时间的代价，消除线程挂起的开销。

1.6之前自旋锁默认关闭，1.6以后改为默认开启，并提供自适应的自旋锁。自适应的自旋锁根据上次自旋等待的时间和锁的状态，调整自旋次数。

#### 锁消除

虚拟机即时编译器在运行时，对一些代码上要求同步，但实际上检测到不需要同步的代码进行锁消除。检测的依据是逃逸分析。Java中有些代码看似没有加锁，但经过编译器优化后可能会加上锁，例如String变量的相加，被优化成使用StringBuilder类。这是如果检测到不需要进行同步，就会消除锁。

#### 锁粗化

通常开发代码时，都要求同步块尽量小，这样使锁占用的时间尽可能短，降低锁等待的时间。但是如果是对同一个对象反复加锁、甚至循环加锁时，反复的加锁和释放锁反而增加了锁的开销，此时虚拟机会粗化锁，即把加锁操作放到循环外。

#### 轻量级锁

关键字：cas、对象头、Mark Work、线程栈

轻量级锁设计的依据是绝大多数情况下，不存在竞争。则可以通过CAS的方式消除锁的开销。轻量级锁通过对象头中的Mark Work来实现。Mark Work字段中存储了对象的hash值、分代年龄、以及状态标志。状态表示表示当前对象是否被锁住和锁的状态。当线程获取锁对象时，在线程栈中创建一块区域，通过CAS操作把对象头中的Mark Work拷贝到线程栈中。如果拷贝成功表示加锁成功，并在对象头中设置执行当前线程的指针。如果对象已经存在指向某个线程的指针则判断是否指向当前线程，是则直接进入同步代码块。否则升级为重量级锁，重量级锁的对象是操作系统的互斥量。锁释放时，也是通过CAS操作把之前拷贝到线程的Mark Word字段拷贝回对象头。

#### 偏向锁

关键字：无竞争、对象头、线程id

偏向锁是在无竞争的情况下使用的，当存在竞争时，偏向锁就会升级为轻量级锁。

偏向锁会在锁对象头中设置当前线程的Id，此线程以后每次执行到同步块时都不需要再次获取锁。

#### 锁的升级、降级

JVM 优化 synchronized 运行的机制，当 JVM 检测到不同的竞争状况时，会自动切换到适合的锁实现，这种切换就是锁的升级、降级。

当没有竞争出现时，默认会使用偏斜锁。JVM 会利用 CAS 操作（compare and swap），在对象头（存在竞争的对象）上的 Mark Word 部分设置线程 ID，以表示这个对象偏向于当前线程，所以并不涉及真正的互斥锁。这样做的假设是基于在很多应用场景中，大部分对象生命周期中最多会被一个线程锁定，使用偏斜锁可以降低无竞争开销。

如果**有另外的线程试图锁定某个已经被偏斜过的对象**，JVM 就需要撤销（revoke）偏斜锁，并切换到轻量级锁实现。轻量级锁依赖 CAS 操作 Mark Word 来试图获取锁，如果重试成功，就使用普通的轻量级锁；否则，进一步升级为重量级锁。

## ReentrantLock
java 5引入

```java
public class ReentrantLock implements Lock, java.io.Serializable {
  
    private final Sync sync;
  
    public ReentrantLock() {
        sync = new NonfairSync(); //默认为非公平锁
    }  
  
    abstract static class Sync extends AbstractQueuedSynchronizer {
      
    }  
  
   //可以控制 fairness(公平性)
    static final class NonfairSync extends Sync {
    }  
  
    static final class FairSync extends Sync {
      
    }
  
    public void lock() { //获取锁
        sync.acquire(1);
    }  

    public void unlock() { //释放锁
        sync.release(1);
    }
  
}
```

### ReentrantLock如何实现公平和非公平锁是如何实现？



同步队列，不要叫阻塞队列了

这里阻塞的含义：线程入队后阻塞

> ReentrantLock是Java并发包中提供的Api层面的可重入互斥锁。它主要的特点是实现了锁的公平性、超时可中断，以及锁可绑定多个条件。ReentranLock内部通过继承AQS实现来同步器，对锁进行管理。了解锁的原理，就得先看下AQS。AQS主要有4成员变量，头节点，表示当前持有锁的线程。 尾节点，用于在阻塞队列尾部插入新的节点。头节点和尾节点之间构成一个先进先出的阻塞队列。锁的状态，0表示锁未被占用，大于0表示锁被占用，当本线程多次获取锁时，每次获取锁时状态+1。当前获取到锁的线程对象。
>
> 当线程获取锁时，首先尝试获取锁，通过判断AQS中锁的状态，如果为0，则表示锁未被占用，由于是公平锁此时判断等待队列中，是否有线程等待，如果没有则CAS获取锁。如果当前线程已经持有锁则对锁状态+1。否则挂起当前线程，并将当前线程封装成Node对象加入阻塞队列最后，期间如果存在竞争的情况，则通过自旋入队，直到成功。
>
> 通过前驱节点的状态判断当前线程是否需要挂起
>
> 把每个线程对象封装成一个Node对象，并组成一个双向阻塞队列。
>
> Unlock时释放锁并且唤醒等待队列中的第一个线程



waitStatus



condition



## 性能比较

- 低竞争场景
- 高竞争场景

##  线程安全
线程安全是一个多线程环境下正确性的概念，也就是保证多线程环境下共享的、可修改的状态（数据）的正确性。

- 保证线程安全的两个方法
  - 封装
  - 不可变
- 线程安全的基本特性
  - 原子性：同步机制
  - 可见性：volatile，线程本地状态反应到主内存上
  - 有序性：保证线程内串行语义，避免指令重排序





## 参考

[java 中的锁 -- 偏向锁、轻量级锁、自旋锁、重量级锁](https://blog.csdn.net/zqz_zqz/java/article/details/70233767)

