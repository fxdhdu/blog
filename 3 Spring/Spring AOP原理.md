[TOC]

## Spring AOP JDK和CGLib动态代理技术实现上的比较？

1. CGLib创建的动态代理对象性能大概是JDK所创建的动态代理对象的10倍。
2. CGLib创建代理对象所花费的时间是JDK动态代理的8倍。
3. 无需频繁创建代理对象，比如singleton的代理对象或具有实例池的代理适合使用CGLib，反之适合采用JDK动态代理技术。



## Aop的应用

事务管理、安全审计、日志处理、性能监控

## Aop的作用

使非业务代码更加模块化，使模块更加内聚。减少重复的非业务代码。



## Aop术语

### 连接点

1. 用方法表示程序执行点
2. 用相对位置表示方位

### 切点

用于定位特定的连接点

```java
package org.springframework.aop;

public interface Pointcut {

}
```

### 增强

1. 用于添加到目标连接点上的一段执行逻辑
2. 用于定位连接点的方位信息

```java
package org.springframework.aop;

public interface BeforeAdvice extends Advice {

}
```

### 目标对象

### 引介

为类添加属性和方法

### 织入

将增强添加到目标类的具体连接点上的过程。

1. 编译期织入 （AspectJ）
2. 类装载期织入 （AspectJ）
3. 动态代理织入 （Spring）

### 代理

代理类（实现原类的接口，或则使原类的子类）

### 切面

由切点和增强（引介）组成

## Aop源码阅读

开启 @AspectJ 的两种方式

1. 使用配置

```xml
<aop:aspectj-autoproxy/>
```

2. 使用注解
@EnableAspectJAutoProxy

它们的原理是一样的，都是通过注册一个 bean 来实现的 （AspectJAutoProxyBeanDefinitionParser）。



```java
package org.springframework.aop.config;

// 注册注解对应的解析器
public class AopNamespaceHandler extends NamespaceHandlerSupport {
   @Override
   public void init() {
      registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
      registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser()); //aspectj-autoproxy为spring自定义的注解
      registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());
      registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
   }
}

```

```java
package org.springframework.aop.config;

//aspectj-autoproxy注解对应的解析器
class AspectJAutoProxyBeanDefinitionParser implements BeanDefinitionParser {
   @Override
   public BeanDefinition parse(Element element, ParserContext parserContext) {
      //注册
      AopNamespaceUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(parserContext, element);
      //对于注解中子类的处理
      extendBeanDefinition(element, parserContext);
      return null;
   }
}

```

```java
public abstract class AopNamespaceUtils {
  
public static void registerAspectJAnnotationAutoProxyCreatorIfNecessary(
      ParserContext parserContext, Element sourceElement) {
  //注册或升级beanName为org.springframework.aop.config.internalAutoProxyCreator的BeanDefinition
   BeanDefinition beanDefinition = AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(
         parserContext.getRegistry(), parserContext.extractSource(sourceElement));
//对于proxy-target-class以及expose-proxy属性的处理
   useClassProxyingIfNecessary(parserContext.getRegistry(), sourceElement);
//注册组件并通知，便于监听器做进一步处理
   registerComponentIfNecessary(beanDefinition, parserContext);
}
  
public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry, Object source) {
   return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source);
}
  
private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry, Object source) {
   Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
	 //如果已经存在了自动代理创建器
   if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
	   //获取自动代理创建器
      BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
      //自动代理创建器与现在的不一致
      if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
				 //比较优先级
         int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
         int requiredPriority = findPriorityForClass(cls);
         if (currentPriority < requiredPriority) {
            //改变Bean
            apcDefinition.setBeanClassName(cls.getName());
         }
      }
      //无需创建
      return null;
   }
   //注册
   RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
   beanDefinition.setSource(source);
   beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
   beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
   registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
   return beanDefinition;
}
  
private static void useClassProxyingIfNecessary(BeanDefinitionRegistry registry, Element sourceElement) {
   if (sourceElement != null) {
//对proxy-target-class属性的处理。
//强制使用CGLIB代理时使用。
      boolean proxyTargetClass = Boolean.parseBoolean(sourceElement.getAttribute(PROXY_TARGET_CLASS_ATTRIBUTE));
      if (proxyTargetClass) {
         AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
      }
//对expose-proxy属性的处理。
//expose-proxy：当目标对象内部的自我调用无法实施切面中的增强时，使用此属性。
//((AService)AopContext.currentProxy()).b()
      boolean exposeProxy = Boolean.parseBoolean(sourceElement.getAttribute(EXPOSE_PROXY_ATTRIBUTE));
      if (exposeProxy) {
         AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
      }
   }
}
  
  //强制使用，属性设置
  public static void forceAutoProxyCreatorToUseClassProxying(BeanDefinitionRegistry registry) {
     if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
        BeanDefinition definition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
        definition.getPropertyValues().add("proxyTargetClass", Boolean.TRUE);
     }
  }
  
}

```

