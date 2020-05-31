## String、StringBuffer、StringBuilder的区别

从性能、底层实现、源码、编码方面分析

JVM对象缓存机制、intern方法、PermGen永久代、FullGC、OOM、堆、MetaSpace元数据区



### String

- 字符串对象类型
- 拼接时，使用 + 符号，会产生新的String对象（中间对象）

### StringBuffer（线程安全）

- 用于拼接字符串
- append、add方法拼接字符串，toString方法返回字符串
- 底层为char、byte[] value数组
- 数组扩容、arraycopy

```java
 public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, Comparable<StringBuffer>, CharSequence
{
   
    public StringBuffer() {
        super(16); // 默认初始容量为16
    }
   
    @Override
    @HotSpotIntrinsicCandidate
    public synchronized StringBuffer append(String str) {
        toStringCache = null;
        super.append(str);
        return this;
    }
   
}
```



### StringBuilder（线程不安全）

- 字符串拼接首选

```java
public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, Comparable<StringBuilder>, CharSequence
{
}
```



### AbstractStringBuilder

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
  
    byte[] value;

     public AbstractStringBuilder append(String str) {
        if (str == null) {
            return appendNull();
        }
        int len = str.length();
        ensureCapacityInternal(count + len);
        putStringAt(count, str);
        count += len;
        return this;
    }
  
    private void ensureCapacityInternal(int minimumCapacity) {
        // overflow-conscious code
        int oldCapacity = value.length >> coder;
        if (minimumCapacity - oldCapacity > 0) {
            value = Arrays.copyOf(value,
                    newCapacity(minimumCapacity) << coder); // 扩容
        }
    }  
}
```

|      | String                                           | StringBuffer                                   | StringBuilder               |
| ---- | ------------------------------------------------ | ---------------------------------------------- | --------------------------- |
|      | 线程安全<br>Immutable 类、final class、final属性 | 线程安全<br>修改数据的方法都有synchronized修饰 | 线程不安全                  |
|      |                                                  | 继承自AbstractStringBuilder                    | 继承自AbstractStringBuilder |
|      |                                                  |                                                |                             |







## 字符串排重





## 参考



