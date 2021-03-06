事务性查询缓存机制

本地的会话缓存：默认情况下启动，它仅仅对一个会话中的数据进行缓存。 

全局的二级缓存：

二级缓存是事务性的。这意味着，当 SqlSession 完成并提交时，或是完成并回滚，但没有执行 flushCache=true 的 insert/delete/update 语句时，缓存会获得更新。

在你的 SQL 映射文件xml中添加一行：

```xml
<cache/>
```

使用Java api时，使用 @CacheNamespaceRef 注解指定缓存作用域。



- 映射语句文件中的所有 select 语句的结果将会被缓存。（映射语句文件中）
- 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
- 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
  - `LRU` – 最近最少使用：移除最长时间不被使用的对象。（默认）
  - `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。
  - `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。
  - `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。
- 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
- 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
- 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。