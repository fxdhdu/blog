## 内部静态类

可不依赖外部类实例，而被实例化。而非内部静态类，必需通过所属类的实例进行实例化。

```java
public class StaticClassTest {


    class NotStaticClass {
        
    }
    
    static class StaticClass {
        
    }
    
    public static void main(String[] args) {
        StaticClassTest staticClassTest = new StaticClassTest();
        StaticClass staticClass = new StaticClass();
        NotStaticClass notStaticClass = staticClassTest.new NotStaticClass();

    }
}
```



## 静态方法



## 静态成员变量



## 参考

https://baijiahao.baidu.com/s?id=1636927461989417537&wfr=spider&for=pc

https://blog.csdn.net/kuangay/article/details/81485324