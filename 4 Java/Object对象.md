## Object对象有哪些方法？

getClass

equals、hashcode

clone

toString

notify\notifyAll

wait\wait(long timeoutMillis)\wait(long timeoutMillis, int nanos)



```java
package java.lang;

import jdk.internal.HotSpotIntrinsicCandidate;

public class Object {

    private static native void registerNatives();
    static {
        registerNatives();
    }

  
    @HotSpotIntrinsicCandidate
    public Object() {}

    @HotSpotIntrinsicCandidate
    public final native Class<?> getClass();


    @HotSpotIntrinsicCandidate
    public native int hashCode();

    public boolean equals(Object obj) {
        return (this == obj);
    }

    @HotSpotIntrinsicCandidate
    protected native Object clone() throws CloneNotSupportedException; 
  // 浅拷贝。它只会拷贝对象中的基本数据类型的数据（比如，int、long），以及引用对象（SearchWord）的内存地址，不会递归地拷贝引用对象本身。

    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }


    @HotSpotIntrinsicCandidate
    public final native void notify();


    @HotSpotIntrinsicCandidate
    public final native void notifyAll();

 
    public final void wait() throws InterruptedException {
        wait(0L);
    }


    public final native void wait(long timeoutMillis) throws InterruptedException;

    public final void wait(long timeoutMillis, int nanos) throws InterruptedException {
        if (timeoutMillis < 0) {
            throw new IllegalArgumentException("timeoutMillis value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0 && timeoutMillis < Long.MAX_VALUE) {
            timeoutMillis++;
        }

        wait(timeoutMillis);
    }

    @Deprecated(since="9")
    protected void finalize() throws Throwable { }
}

```

