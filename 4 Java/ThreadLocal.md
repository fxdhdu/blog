## ThreadLocal、Thread、ThreadLocalMap

ThreadLocal称为线程本地变量、线程本地存储，用于线程间的数据隔离。

每个线程的Thread类实例，都有一个ThreadLocalMap属性的本地变量。存放了当前线程ThreadLocal变量的副本。
Entry为什么要使用弱引用（生命周期只能到下次gc前）

```java
public class Thread implements Runnable {
    ThreadLocal.ThreadLocalMap threadLocals = null;
}
```
```java
public class ThreadLocal<T> {
  
    static class ThreadLocalMap {

        static class Entry extends WeakReference<ThreadLocal<?>> {
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
}
```
非ThreadLocal变量，数据存储在对象实例中。

## ThreadLocal的方法
- get
- set
- remove

等

## 使用场景
1、数据库链接
2、session管理



## 参考

[彻底理解ThreadLocal](https://blog.csdn.net/lufeng20/article/details/24314381)