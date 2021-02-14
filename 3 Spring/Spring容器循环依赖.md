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

