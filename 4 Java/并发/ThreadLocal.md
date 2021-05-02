## ThreadLocal、Thread、ThreadLocalMap

![在这里插入图片描述](/Users/fanxudong/IdeaProjects/blog/4 Java/assert/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODYyODgy,size_16,color_FFFFFF,t_70.png)

ThreadLocal称为线程本地变量、线程本地存储，用于线程间的数据隔离。

每个线程的Thread类实例，都有一个ThreadLocalMap属性的本地变量。存放了当前线程ThreadLocal变量的副本。


```java
public class Thread implements Runnable {
    ThreadLocal.ThreadLocalMap threadLocals = null;
}
```
```java
public class ThreadLocal<T> {
  
    private final int threadLocalHashCode = nextHashCode();  
  
    static class ThreadLocalMap {

        static class Entry extends WeakReference<ThreadLocal<?>> { // key 为弱引用，ThreadLocal变量为弱引用
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

        private static final int INITIAL_CAPACITY = 16;

        private Entry[] table;

        private int size = 0;

        private int threshold; // Default to 0  
    }
  
}
```
获取变量时，根据公用的ThreadLocal变量的this作为key，从当前线程的map中获取Entry，并获取value值。
```java
public class ThreadLocal<T> {

    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value); 
        }
    }

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }  
  
    public T get() {
        Thread t = Thread.currentThread(); // 获取当前线程
        ThreadLocalMap map = getMap(t); // 获取map
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this); //key为ThreadLocal对象，数据存储在ThreadLocalMap中
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue(); //
    }  

    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
  
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null) {
             m.remove(this);
         }
     }  
  
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    } 
  
    private void remove(ThreadLocal<?> key) {
      Entry[] tab = table;
      int len = tab.length;
      int i = key.threadLocalHashCode & (len-1);
      for (Entry e = tab[i];
           e != null;
           e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
          e.clear();
          expungeStaleEntry(i);
          return;
        }
      }
    }  
}
```
非ThreadLocal变量，数据存储在对象实例中。

## ThreadLocal的方法
- get
- set
- remove

等

## Entry为什么要使用弱引用（生命周期只能到下次gc前）（解决key的泄漏问题）

因为线程会持有ThreadLocalMap，key为ThreadLocal引用，在使用线程池时，线程会复用。如果ThreadLocal引用为强引用，则线程不结束，就不会回收ThreadLocal对象。

## 使用完ThreadLocal后，为什么要执行remove（解决value的泄漏问题）

突然我们ThreadLocal是null了，也就是要被垃圾回收器回收了，但是此时我们的ThreadLocalMap生命周期和Thread的一样，它不会回收（线程池中的线程会重复使用），这时候就出现了一个现象。那就是ThreadLocalMap的key没了，但是value还在，这就造成了内存泄漏。所以ThreadLocal使用完之后，要执行remove，删除线程ThreadLocalMap中的值。



## 使用场景
1、数据库链接
2、session管理



## 参考

[彻底理解ThreadLocal](https://blog.csdn.net/lufeng20/article/details/24314381)

[ThreadLocal为什么要使用弱引用和内存泄露问题](https://blog.csdn.net/qq_42862882/article/details/89820017)