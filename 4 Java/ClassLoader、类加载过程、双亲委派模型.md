[TOC]

> 类加载器的本质其实就是通过文件操作扫描到应用classpath下的Jar包，然后读取Jar包里的class文件，经过解析，校验等等一堆乱七八糟的操作之后，再将文件里的字节码内容加载到内存里，供后面类实例化为对象使用，实例化对象的时候会根据已加载的类信息去做分配内存，初始化成员变量等等一些工作。



类装载器ClassLoader的作用是动态加载Class文件到JVM内存当中



类加载的过程（又或者是说类的生命周期）主要包括以下几个步骤：加载、链接（校验、准备、解析）、初始化、使用、卸载。



## Java默认提供的三个ClassLoader

### Bootstrap ClassLoader （根装载器）

启动类加载器。

Java类加载层次中最顶层的类加载器，负责加载JDK中的核心类库，如：rt.jar、resources.jar、charsets.jar等。

Bootstrap ClassLoader不继承自ClassLoader，因为它不是一个普通的Java类，底层由C++编写，已嵌入到了JVM内核当中，当JVM启动后，Bootstrap ClassLoader也随着启动，负责加载完核心类库后，并构造Extension ClassLoader和App ClassLoader类加载器。



### Extension ClassLoader（扩展类装载器）

扩展类加载器，负责加载Java的扩展类库，默认加载JAVA_HOME/jre/lib/ext/目下的所有jar。



### App ClassLoader（应用类装载器）

系统类加载器，负责加载应用程序classpath目录下的所有jar和class文件。



## ClassLoader加载类的原理

### 双亲委托模型

每个ClassLoader实例都包含一个父类加载器的引用。

当一个ClassLoader实例需要加载某个类时，它先把这个任务委托给它的父类加载器（递归调用）。

首先由最顶层的类加载器Bootstrap ClassLoader试图加载，如果没加载到，则把任务转交给Extension ClassLoader试图加载，如果也没加载到，则转交给App ClassLoader 进行加载，如果它也没有加载得到的话，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。（自定义的java.lang.String不会被加载，但是会被编译）

如果它们都没有加载到这个类时，则抛出ClassNotFoundException异常。

否则将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的Class实例对象。

### 全盘负责委托机制

全盘负责：同时加载类及类所依赖和引用的类。

委托：双亲委托模型





## ClassLoader源码

```java
public abstract class ClassLoader {

    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }  
 
    public final ClassLoader getParent() {

    }
  
    public InputStream getResourceAsStream(String name) {

    }
  
}
```



## 参考

《精通 Spring 4.x 企业应用开发实战》