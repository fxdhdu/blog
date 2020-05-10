## 反射的用途

可通过ClassLoader获取Class对象，从Class对象中获取Class反射对象。通过Class反射对象对类实例进行操作。

## Class的反射对象

### Constructor

类的构造函数反射类

```java
package java.lang.reflect;

public final class Constructor<T> extends Executable {

    public void setAccessible(boolean flag) { // 访问private或protected成员变量和方法时，用于取消Java语言检查。否则抛出IllegalAccessException。

    }
  
    public T newInstance(Object ... initargs) {
		}  
  
}

```

### Method

类方法的反射类

```java
public final class Method extends Executable {

    @CallerSensitive
    @ForceInline // to ensure Reflection.getCallerClass optimization
    @HotSpotIntrinsicCandidate
    public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException
    {

    }  
  
}
```



### Field

类的成员变量的反射类

```java
public final
class Field extends AccessibleObject implements Member {
}
```



### Package



### AnnotatedElement

注解反射类

```java
public interface AnnotatedElement {
}
```

