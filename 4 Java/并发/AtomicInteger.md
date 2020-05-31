```java
public class AtomicInteger extends Number implements java.io.Serializable {

    private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
  
    public final boolean compareAndSet(int expectedValue, int newValue) {
        return U.compareAndSetInt(this, VALUE, expectedValue, newValue);
    }
  
    public final int getAndIncrement() {
        return U.getAndAddInt(this, VALUE, 1);
    }  
  
}
```

```java
public final class Unsafe {
  
    @HotSpotIntrinsicCandidate
    public final native boolean compareAndSetInt(Object o, long offset,
                                                 int expected,
                                                 int x);

    @HotSpotIntrinsicCandidate
    public final int getAndAddInt(Object o, long offset, int delta) {
        int v;
        do {
            v = getIntVolatile(o, offset);
        } while (!weakCompareAndSetInt(o, offset, v, v + delta));
        return v;
    }
  
    @HotSpotIntrinsicCandidate
    public native int     getIntVolatile(Object o, long offset);  
}
```



通过CAS实现线程安全