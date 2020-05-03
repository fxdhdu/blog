[TOC]



## ClassLoader的作用

动态加载class文件到内存当中



## Java默认提供的三个ClassLoader

### Bootstrap ClassLoader

启动类加载器。Java类加载层次中最顶层的类加载器，负责加载JDK中的核心类库，如：rt.jar、resources.jar、charsets.jar等。Bootstrap ClassLoader不继承自ClassLoader，因为它不是一个普通的Java类，底层由C++编写，已嵌入到了JVM内核当中，当JVM启动后，Bootstrap ClassLoader也随着启动，负责加载完核心类库后，并构造Extension ClassLoader和App ClassLoader类加载器。

### Extension ClassLoader

扩展类加载器，负责加载Java的扩展类库，默认加载JAVA_HOME/jre/lib/ext/目下的所有jar。

### App ClassLoader

系统类加载器，负责加载应用程序classpath目录下的所有jar和class文件。



## ClassLoader加载类的原理

### 双亲委托模型

每个ClassLoader实例都包含一个父类加载器的引用。

当一个ClassLoader实例需要加载某个类时，它先把这个任务委托给它的父类加载器（递归调用）。

首先由最顶层的类加载器Bootstrap ClassLoader试图加载，如果没加载到，则把任务转交给Extension ClassLoader试图加载，如果也没加载到，则转交给App ClassLoader 进行加载，如果它也没有加载得到的话，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。

如果它们都没有加载到这个类时，则抛出ClassNotFoundException异常。否则将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的Class实例对象。





怎么根据文件名获取到整个文件的路径？是怎么扫描的。

 首先用父类加载器加载资源。

 Find resources from the VM's built-in classloader.