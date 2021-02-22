[TOC]

Spring IOC 容器初始化的过程也就是spring中Bean的生命周期。



## 从两个层面定义Bean的生命周期

Bean的作用范围

实例化Bean时所经历的一系列阶段



## BeanFactory中Bean的生命周期

Spring着手实例化Bean  -> 销毁Bean



## ApplicationContext中Bean的生命周期











## Spring核心依赖

```xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.11.RELEASE</version>
        </dependency>
        
        <!--使用spring事务-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>4.3.11.RELEASE</version>
        </dependency>
```

|                                    |                                                             |
| ---------------------------------- | ----------------------------------------------------------- |
| ClassPathXmlApplicationContext     | 从类名可知，通过类路径下的xml文件来构造ApplicationContext。 |
| AnnotationConfigApplicationContext | spring boot中用这种方式？                                   |

![](/Users/fanxudong/IdeaProjects/blog/4 Java/assert/ApplicationContext继承结构.png)

![](/Users/fanxudong/IdeaProjects/blog/3 Spring/assert/92EB51A7-5D4E-4DE5-9C44-AAC396CB1AAC.png)

最顶层 BeanFactory 接口的方法都是获取单个 Bean 的。

Listable：可获取多个 Bean。

Hierarchical：可以在应用中起多个 BeanFactory，然后可以将各个 BeanFactory 设置为父子关系。



### ApplicationContext （应用上下文/Spring容器）

Spring IOC的核心

可以创建多个吗？

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
      
      
		}
```



### BeanFactory （IOC容器）

```java
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

### BeanDefinition

BeanDefinition 就是我们所说的 Spring 的 Bean。Bean 在代码层面上可以认为是 BeanDefinition 的实例。

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

}
```



## 源码阅读



```java
    public static void main(String[] args) {
        // 用我们的配置文件来启动一个 ApplicationContext
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:application.xml");
    }
```



## BeanFactoryPostProcessor

BeanFactory后置处理器，是对BeanDefinition对象进行修改。（BeanDefinition：存储bean标签的信息，用来生成bean实例）



## BeanPostProcessor

Bean后置处理器，是对生成的Bean对象进行修改。





配置文件 -》 DOM树

**提问：**

- Spring 中获取 ApplicationContext  的方法？
- Spring 中Bean怎么继承父Bean的配置信息？
- Spring 中Bean实例的生成方式：1）反射 2）工厂方法



Spring循环依赖

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



Bean的五个作用域

作用域影响Bean的生命周期和创建方式

## singleton（Bean的默认作用域）

无状态或者状态不可变的类

## prototype



## request



## session



## globalSession



**参考：**

[Spring IOC 容器源码分析](https://www.javadoop.com/post/spring-ioc) 

[一文告诉你Spring是如何利用“三级缓存“巧妙解决Bean的循环依赖问题的【享学Spring】](https://blog.csdn.net/f641385712/article/details/92801300?biz_id=102&utm_term=spring%20%E4%B8%89%E7%BA%A7%E7%BC%93%E5%AD%98&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-92801300&spm=1018.2118.3001.4187)

