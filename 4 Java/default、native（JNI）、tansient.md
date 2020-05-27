

[TOC]

## default

接口中使用default关键字修饰方法，可以为方法提供默认实现。

- Java8的新特性
- default常用于接口中，我们本来写Java接口的时候，方法是不能有方法体的。default关键字在接口中修饰方法时，方法可以有方法体。
- default关键字可以让接口中的方法可以有默认的方法体，当一个类实现这个接口时，可以不用去实现这个方法，当然，这个类若实现这个方法，就等于子类覆盖了这个方法，最终运行结果符合Java多态特性。


## native(JNI、Java Native Interface)

[认识&理解关键字 native 实战篇](https://www.cnblogs.com/Alandre/p/4456719.html)

```java
public native int hashCode();
```

## transient

http://www.importnew.com/21517.html

字面意思：adj形容词,短暂的;转瞬即逝的;临时的

### 什么时候使用

当实现Serilizable接口（自动序列化）时，可将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。

比如用户有一些敏感信息（如密码，银行卡号等），为了安全起见，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上transient关键字。

### 使用小结

1. 一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。
2. transient关键字只能修饰类变量，而不能修饰方法和类。
3. 方法局部变量是不能被transient关键字修饰的。
4. 变量如果是用户自定义类变量，则该类需要实现Serializable接口，才能被transient修饰。
5. 被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。
6. 若实现Externalizable接口（手动序列化），则可在writeExternal方法中指定所要序列化的变量，这与是否被transient修饰无关。此时被transient修饰的变量也可被序列化。

