并发包（java.util.concurrent）

同步包装器（Synchronized Wrapper）

Collections.synchronizedMap

## 并发容器

ConcurrentHashMap

CopyOnWriteArrayList

## 线程安全队列

Queue/Deque

ArrayBlockingQueue

SynchronousQueue

保证线程安全的方式

synchronize

分离锁



  了解LinkedBlockingQueue、ArrayBlockingQueue、DelayQueue、SynchronousQueue





了解实现原理、扩容时做的优化、与HashTable对比。



- ConcurrentHashMap是如何在保证并发安全的同时提高性能？

分段

- ConcurrentHashMap是如何让多线程同时参与扩容？
- LinkedBlockingQueue、DelayQueue是如何实现的？
- CopyOnWriteArrayList是如何保证线程安全的？





```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
  

    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;
    }
  
    static final class TreeNode<K,V> extends Node<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
    }

    /**
     * A node inserted at head of bins during transfer operations.
     */
    static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
    }
  
}
```



