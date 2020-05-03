## String、StringBuffer、StringBuilder的区别

从性能、底层实现、源码、编码方面分析

JVM对象缓存机制、intern方法、PermGen永久代、FullGC、OOM、堆、MetaSpace元数据区



### String

- 字符串对象类型
- Immutable 类、final class、final属性，线程安全
- 拼接时，使用 + 符号，会产生新的String对象（中间对象）

### StringBuffer（线程安全）

- 用于拼接字符串
- append、add方法拼接字符串
- 线程安全、修改数据的方法都有synchronized修饰
- 底层为char、byte数组
- 数组扩容、arraycopy

### StringBuilder（线程不安全）

- 字符串拼接首选
- 线程不安全



## 字符串排重

