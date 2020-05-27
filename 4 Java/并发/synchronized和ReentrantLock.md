[TOC]

| synchronized                   | ReentrantLock                                                |
| ------------------------------ | ------------------------------------------------------------ |
| java内建同步机制               | java.util.concurrent.locks 包中提供                          |
| 无法控制公平性，其为非公平锁。 | 可以控制 fairness(公平性)。公平锁，倾向于将锁赋予等待时间最久的线程。可减少线程“饥饿”，但会降低吞吐量。 |
| 偏斜锁                         | 再入锁，当一个线程试图获取一个它已经获取的锁时，这个获取动作就自动成功。锁的持有是以线程为单位。 |



## synchronized

- 提供了互斥的语义和可见性
- 阻塞 
- 悲观锁
- 修饰静态方法，锁对象为类的Class对象（ClassName.class，永久代/Metaspace中）
- 修饰非静态方法，锁对象为this （当前类实例）
- 修饰代码块

### synchronized底层如何实现

一对monitorenter/monitorexit指令

Monitor 对象的三种不同的实现（实现代码在JVM中）

- 偏斜锁（Biased Locking）


- 轻量级锁

- 重量级锁

### 锁的升级、降级

JVM 优化 synchronized 运行的机制，当 JVM 检测到不同的竞争状况时，会自动切换到适合的锁实现，这种切换就是锁的升级、降级。



当没有竞争出现时，默认会使用偏斜锁。JVM 会利用 CAS 操作（compare and swap），在对象头（存在竞争的对象）上的 Mark Word 部分设置线程 ID，以表示这个对象偏向于当前线程，所以并不涉及真正的互斥锁。这样做的假设是基于在很多应用场景中，大部分对象生命周期中最多会被一个线程锁定，使用偏斜锁可以降低无竞争开销。

如果有另外的线程试图锁定某个已经被偏斜过的对象，JVM 就需要撤销（revoke）偏斜锁，并切换到轻量级锁实现。轻量级锁依赖 CAS 操作 Mark Word 来试图获取锁，如果重试成功，就使用普通的轻量级锁；否则，进一步升级为重量级锁。

## ReentrantLock
- java 5引入
- 使用时需显示获取和释放锁。

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

- 定义条件

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

