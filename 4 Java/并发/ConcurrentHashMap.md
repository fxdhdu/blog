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
- ConcurrentHashMap是如何让多线程同时参与扩容？
- LinkedBlockingQueue、DelayQueue是如何实现的？
- CopyOnWriteArrayList是如何保证线程安全的？