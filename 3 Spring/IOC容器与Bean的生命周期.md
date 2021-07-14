[TOC]

## Bean的生命周期定义

可以从两个层面来定义：

- Bean的作用范围
- 实例化Bean时所经历的一系列阶段。Spring IOC 容器初始化的过程也就是spring中Bean的生命周期的前几个阶段。



## BeanFactory中Bean的生命周期

Spring着手实例化Bean  -> 销毁Bean



## ApplicationContext（应用上下文/Spring容器）中Bean的生命周期

通过类路径下的xxxx.xml配置文件启动Spring容器。这是IOC的核心。

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationfile.xml");
}
```

| ApplicationContext的实现类         | 说明                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| ClassPathXmlApplicationContext     | 从类名可知，通过类路径下的xml文件来构造ApplicationContext。通常叫application.xml |
| AnnotationConfigApplicationContext | 基于注解的方式启动容器。spring boot中用这种方式？            |
| FileSystemXmlApplicationContext    | 需要的 xml 配置文件在系统中的路径下。                        |

ApplicationContext接口向下的继承体系。以及针对特定场景的实现类。

![](./asset/ApplicationContext继承结构.png)

ApplicationContext接口向上的继承体系。

```java
public interface ApplicationContext extends 
  EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
}
```

ListableBeanFactory 接口，理解为 Listable 的意思就是，通过这个接口，我们可以获取多个 Bean。

HierarchicalBeanFactory接口表示，在应用中可以起多个BeanFactory，并设置它们的父子关系。

它们最顶层为BeanFactory接口。BeanFactory可以理解为生产Bean的工厂，它负责生产和管理各个 bean 实例。它的方法都是获取单个的Bean。

```java
public interface ListableBeanFactory extends BeanFactory {
}

public interface HierarchicalBeanFactory extends BeanFactory {
	BeanFactory getParentBeanFactory();
}

public interface BeanFactory {
	Object getBean(String name) throws BeansException;
}
```

ApplicationContext的抽象实现类中，持有一个实例化BeanFactory。

```java
public abstract class AbstractRefreshableApplicationContext extends AbstractApplicationContext {

	/** Bean factory for this context */
	private DefaultListableBeanFactory beanFactory;  
  
}
```

BeanFactory的继承体系。

![](./asset/92EB51A7-5D4E-4DE5-9C44-AAC396CB1AAC.png)

### BeanDefinition

BeanDefinition 就是我们所说的 Spring 的 Bean。Bean 在代码层面上可以认为是 BeanDefinition 的实例。

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

}
```



### refresh方法源码阅读

```java
    public static void main(String[] args) {
        // 用我们的配置文件来启动一个 ApplicationContext
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:application.xml");
    }
```


```java
@Override
public void refresh() throws BeansException, IllegalStateException {
   // 来个锁，不然 refresh() 还没结束，你又来个启动或销毁容器的操作，那不就乱套了嘛
   synchronized (this.startupShutdownMonitor) {

      // 准备工作，记录下容器的启动时间、标记“已启动”状态、处理配置文件中的占位符
      prepareRefresh();

      // 这步比较关键，这步完成后，配置文件就会解析成一个个 Bean 定义，注册到 BeanFactory 中，
      // 当然，这里说的 Bean 还没有初始化，只是配置信息都提取出来了，
      // 注册也只是将这些信息都保存到了注册中心(注册中心说到底核心是一个 beanName-> beanDefinition 的 map)
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // 设置 BeanFactory 的类加载器，（为后面初始化bean做准备？）
      // 添加几个 BeanPostProcessor，手动注册几个特殊的 bean
      // 这块待会会展开说
      prepareBeanFactory(beanFactory);

      try { // 为什么try catch要加在这里呢？因为后边会执行业务逻辑吧？
         // 【这里需要知道 BeanFactoryPostProcessor 这个知识点，Bean 如果实现了此接口，
         // 那么在容器初始化以后，Spring 会负责调用里面的 postProcessBeanFactory 方法。】

         // 这里是提供给子类的扩展点，到这里的时候，所有的 Bean 都加载、注册完成了，但是都还没有初始化
         // 具体的子类可以在这步的时候添加一些特殊的 BeanFactoryPostProcessor 的实现类或做点什么事
         postProcessBeanFactory(beanFactory);
        
         // 调用 BeanFactoryPostProcessor 各个实现类的 postProcessBeanFactory(factory) 方法
         // ServiceClassPostProcessor执行postProcessBeanDefinitionRegistry方法。
         invokeBeanFactoryPostProcessors(beanFactory);

         // 注册 BeanPostProcessor 的实现类，注意看和 BeanFactoryPostProcessor 的区别
         // 此接口两个方法: postProcessBeforeInitialization 和 postProcessAfterInitialization
         // 两个方法分别在 Bean 初始化之前和初始化之后得到执行。注意，到这里 Bean 还没初始化
         // 具体做法是，根据BeanPostProcessor.class，去BeanFactory获取
         // 如果实现了PriorityOrdered、Ordered接口
         /* dubbo在这里实现了一些后处理器：DubboConfigAliasPostProcessor、					
        			DubboConfigDefaultPropertyValueBeanPostProcessor、
                DubboConfigEarlyInitializationPostProcessor*/
         registerBeanPostProcessors(beanFactory);

         // 初始化当前 ApplicationContext 的 MessageSource，国际化这里就不展开说了，不然没完没了了
         initMessageSource();

         // 初始化当前 ApplicationContext 的事件广播器，这里也不展开了
         initApplicationEventMulticaster();

         // 从方法名就可以知道，典型的模板方法(钩子方法)，
         // 具体的子类可以在这里初始化一些特殊的 Bean（在初始化 singleton beans 之前）
         onRefresh();

         // 注册事件监听器，监听器需要实现 ApplicationListener 接口。这也不是我们的重点，过
         registerListeners();

         // 重点，重点，重点
         // 初始化所有的 singleton beans
         //（lazy-init 的除外）
         finishBeanFactoryInitialization(beanFactory);

         // 最后，广播事件，ApplicationContext 初始化完成
         finishRefresh();
      }

      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // Destroy already created singletons to avoid dangling resources.
         // 销毁已经初始化的 singleton 的 Beans，以免有些 bean 会一直占用资源
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // 把异常往外抛
         throw ex;
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
```

- prepareRefresh
- obtainFreshBeanFactory
  - customizeBeanFactory
    - 是否允许BeanDefinition覆盖配置
    - 是否允许循环引用配置
  - loadBeanDefinitions   
    - 配置文件转换为DOM树
    - profile处理，不能包含的profile直接跳过处理
    - default namespace涉及的四个标签<import />、<alias />、<bean /> 和 <beans />的处理。
      - <bean />配置 -> BeanDefinition -> BeanDefinition注册 -> 发送注册事件
    - custom namespace涉及的标签处理
      - xmlns、.xsd文件、对应的xxxHandler处理器。
- prepareBeanFactory
  - 设置类加载器
  - 添加几个BeanPostProcessor
  - 注册environment这个Bean
  - 注册systemProperties这个Bean
  - 注册systemEnvironment这个Bean
- postProcessBeanFactory
- invokeBeanFactoryPostProcessors
  - BeanDefinitionRegistryPostProcessor接口执行，BeanDefinition注册后做一些处理。
  - BeanFactoryPostProcessor接口执行：BeanFactory后置处理器，是对BeanDefinition对象进行修改。（BeanDefinition：存储bean标签的信息，用来生成bean实例）
- registerBeanPostProcessors
  - BeanPostProcessor接口注册：Bean后置处理器，是对生成的Bean对象进行修改。生成代理对象，修改对象属性等。
- initMessageSource
- initApplicationEventMulticaster
- onRefresh
- registerListeners
- finishBeanFactoryInitialization
  - singleton beans 实例化
- finishRefresh



<bean /> 标签中可以定义哪些属性

| **Property**             |                                                              |
| ------------------------ | ------------------------------------------------------------ |
| class                    | 类的全限定名                                                 |
| name                     | 可指定 id、name(用逗号、分号、空格分隔)                      |
| scope                    | 作用域                                                       |
| constructor arguments    | 指定构造参数                                                 |
| properties               | 设置属性的值                                                 |
| autowiring mode          | no(默认值)、byName、byType、 constructor                     |
| lazy-initialization mode | 是否懒加载(如果被非懒加载的bean依赖了那么其实也就不能懒加载了) |
| initialization method    | bean 属性设置完成后，会调用这个方法                          |
| destruction method       | bean 销毁后的回调方法                                        |



### Spring循环依赖

1. 构造器循环依赖：Spring无法解决，会抛出BeanCurrentlyInCreationException异常表示循环依赖。
2. setter循环依赖：Spring容器提前暴露刚完成构造器注入但未完成其他步骤（如setter注入）的bean来完成的，而且只能解决单例作用域的bean循环依赖。通过提前暴露一个单例工厂方法（ObjectFactory），从而使其他bean能引用到该bean
3. prototype范围的依赖处理：Spring容器无法完成依赖注入，因为Spring容器不进行缓存“prototype”作用域的bean，因此无法提前暴露一个创建中的bean



- doCreateBean
  - remove
  - createBeanInstance
    - 工厂方法
    - 构造函数
  - applyMergedBeanDefinitionPostProcessors
  - addSingletonFactory
  - populateBean
  - initializeBean
  - getSingleton
  - registerDisposableBeanIfNecessary (destroy-method)



## Bean的五个作用域

作用域影响Bean的生命周期和创建方式

### singleton（Bean的默认作用域）

无状态或者状态不可变的类

### prototype



### request



### session



### globalSession



## 第九章 Spring Bean生命周期

![image-20201101132949101](/Users/fanxudong/Library/Application Support/typora-user-images/image-20201101132949101.png)

### BeanDefinition元信息配置阶段

通过xml或注解方式配置Bean的元信息。

  - 面向资源
    - XML
    - Properties：PropertiesBeanDefinitionReader，默认以ISO编码读取。通过Resource + EncodedResource来以UTF-8读取
    - Grovvy
  - 面向注解
    - @Component
    - @Bean
  - 面向API

### Spring Bean元信息解析阶段

对xml或注解配置的Bean的元信息进行解析。

  - 面向资源BeanDefinition解析
    - BeanDefinitionReader：需要load
    - XML解析器 BeanDefinitionParser
  - 面向注解BeanDefinition解析
    - AnnotatedBeanDefinitionReader：componentScan/没有注解的类也能注册。register方法 

### Spring Bean注册阶段：BeanDefinition与单体Bean注册

  - DefaultListableBeanFactory
    - registerBeanDefinition
      - beanDefinitionNames：用来确保注册顺序
      - beanDefinitionMap：无序

### Spring BeanDefinition合并阶段

  - 父子BeanDefinition合并
    - 当前BeanFactory查找
    - 层次BeanFactory查找
  - BeanDefinition 接口
  - RootBeanDefinition 不需要合并，不存在parent
  - GenericBeanDefinition 普通的bean定义，默认
  - 经过合并后GenericBeanDefinition变成RootBeanDefinition
  - xml（可指定编码）继承配置，通过parent
  - ConfigurableBeanFactory接口
    - getMergedBeanDefinition 递归合并，在AbstractBeanFactory中实现
      - cloneBeanDefinition：不改变原始的Bean，clone一个新的（spring框架不希望改变bean过去的阶段，来实现bean合并）

### Spring Bean Class加载阶段

  Class\ClassLoader

  - ClassLoader类加载
  - Java Security安全控制
  - ConfigurableBeanFactory临时ClassLoader

### Spring Bean实例化前阶段

  - 非主流生命周期  
  - InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation 返回非空时，用于替换spring ioc容器默认实例化的bean。可用于生成代理。

### Spring Bean实例化阶段：Java反射创建？

  -  传统实例化方式
     - 实例化策略 InstantiationStrategy
       - CglibSubclassingInstantiationStrategy
       - SimpleInstantiationStrategy  默认无参构造器：clazz.getDeclaredConstructor()  -->  newInstance()
  -  构造器依赖注入
     - autowire=“constructor” 按照类型注入: beanClass.getConstructors()反射获取构造器
       - 参数名，参数类型解析，初始化依赖注入对象

### Spring Bean实例化后阶段

  - Bean 属性赋值（Populate）判断
    - InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation  return true则执行bean population，return false则对象属性不会赋值

### Spring Bean属性赋值前阶段

  - Bean属性值元信息
    - PropertyValues 可以来自xml配置文件中的< properties  >
  - Bean属性赋值前回调：可以对PropertyValues进行修改
    - Spring 1.2-5: InstantiationAwareBeanPostProcessor#postProcessPropertyValues
    - Spring 5.1: InstantiationAwareBeanPostProcessor#postProcessProperties

### Aware接口回调阶段

  - Spring Aware接口
    - BeanNameAware 标记接口

### Spring Bean初始化前阶段BeanPostProcessor#postProcessBeforeInitialization

### Spring Bean初始化阶段@PostConstruct InitializingBean以及自定一方法

### Spring Bean初始化后节点：BeanPostProcessor#postProcessAfterInitialization

### Spring Bean初始化完成阶段：DestructionAwareBeanPostProcessor

### Spring Bean销毁前阶段@PreDestory、DisposableBean

### Spring Bean垃圾收集



## 问题

1、spring中用到ioc这个概念，或者说spring实现了这个ioc概念，为什么要用ioc模型、带来的好处？你的项目中有没有利用这个特点去做一个什么样的事情？

先大致解释下IOC和DI的概念，以及spring的实现原理：

控制反转（Inversion of Control,IoC）：将对某一**接口具体实现类的选择控制权**从调用类中移除，交由spring容器借由Bean配置来进行控制。

DI依赖注入：调用类对某一**接口实现类的依赖关系**由第三方注入，以移除调用类对某一接口实现类的依赖。

spring通过配置文件或注解描述类和类之间的依赖关系，来自动完成类的初始化和依赖注入工作。



IOC带来的好处是：

1、（解耦）如果依赖的类修改了，比如修改了构造函数，如果没有依赖注入，则需要修改依赖对象调用着，如果依赖注入则不需要。

2、（对象的使用更方便了，只要直接注入就可以了）对象的构建如果依赖非常多的对象，且层次很深，外层在构造对象时很麻烦且不一定知道如何构建这么多层次的对象。 IOC 帮我们管理对象的创建，只需要在配置文件里指定如何构建，每一个对象的配置文件都在类编写的时候指定了，所以最外层对象不需要关心深层次对象如何创建的，前人都写好了。



项目中的应用：

1、项目中模块之间解耦，A模块中使用某个接口，接口实现在另外一个B模块中。A模块不依赖B模块，无法直接调用B中的实现类，通过spring容器完成实现类的注入。

2、通过spring容器获取某个接口的所有实现类。用于实现策略模式？



Spring 中获取 ApplicationContext  的方法？



Spring 中Bean怎么继承父Bean的配置信息？



Spring 中Bean实例的生成方式：1）反射 2）工厂方法


**参考：**

[Spring IOC 容器源码分析](https://www.javadoop.com/post/spring-ioc) 

[一文告诉你Spring是如何利用“三级缓存“巧妙解决Bean的循环依赖问题的【享学Spring】](https://blog.csdn.net/f641385712/article/details/92801300?biz_id=102&utm_term=spring%20%E4%B8%89%E7%BA%A7%E7%BC%93%E5%AD%98&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-92801300&spm=1018.2118.3001.4187)

