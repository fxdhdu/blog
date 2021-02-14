第九章 Spring Bean生命周期

![image-20201101132949101](/Users/fanxudong/Library/Application Support/typora-user-images/image-20201101132949101.png)

- **BeanDefinition元信息配置阶段**
  - 面向资源
    - XML
    - Properties：PropertiesBeanDefinitionReader，默认以ISO编码读取。通过Resource + EncodedResource来以UTF-8读取
    - Grovvy
  - 面向注解
    - @Component
    - @Bean
  - 面向API
  
- **Spring Bean元信息解析阶段**
  - 面向资源BeanDefinition解析
    - BeanDefinitionReader：需要load
    - XML解析器 BeanDefinitionParser
  - 面向注解BeanDefinition解析
    - AnnotatedBeanDefinitionReader：componentScan/没有注解的类也能注册。register方法 
  
- **Spring Bean注册阶段：BeanDefinition与单体Bean注册**
  - DefaultListableBeanFactory
    - registerBeanDefinition
      - beanDefinitionNames：用来确保注册顺序
      - beanDefinitionMap：无序
  
- **Spring BeanDefinition合并阶段**
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
  
- **Spring Bean Class加载阶段**
  
  Class\ClassLoader
  
  - ClassLoader类加载
  - Java Security安全控制
  - ConfigurableBeanFactory临时ClassLoader
  
- **Spring Bean实例化前阶段**

  - 非主流生命周期  
  - InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation 返回非空时，用于替换spring ioc容器默认实例化的bean。可用于生成代理。

- **Spring Bean实例化阶段：Java反射创建？**

  -  传统实例化方式
    - 实例化策略 InstantiationStrategy
      - CglibSubclassingInstantiationStrategy
      - SimpleInstantiationStrategy  默认无参构造器：clazz.getDeclaredConstructor()  -->  newInstance()
  - 构造器依赖注入
    - autowire=“constructor” 按照类型注入: beanClass.getConstructors()反射获取构造器
      - 参数名，参数类型解析，初始化依赖注入对象

- **Spring Bean实例化后阶段**

  - Bean 属性赋值（Populate）判断
    - InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation  return true则执行bean population，return false则对象属性不会赋值

- **Spring Bean属性赋值前阶段**

  - Bean属性值元信息
    - PropertyValues 可以来自xml配置文件中的< properties  >
  - Bean属性赋值前回调：可以对PropertyValues进行修改
    - Spring 1.2-5: InstantiationAwareBeanPostProcessor#postProcessPropertyValues
    - Spring 5.1: InstantiationAwareBeanPostProcessor#postProcessProperties

- **Aware接口回调阶段**

  - Spring Aware接口
    - BeanNameAware 标记接口

- **Spring Bean初始化前阶段BeanPostProcessor**

- **Spring Bean初始化阶段@PostConstruct InitializingBean以及自定一方法**

- **Spring Bean初始化后节点：BeanPostProcessor**

- **Spring Bean初始化完成阶段：DestructionAwareBeanPostProcessor**

- **Spring Bean销毁前阶段@PreDestory、DisposableBean**

- **Spring Bean垃圾收集**

