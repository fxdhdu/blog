## Aop的应用

事务管理、安全审计、日志处理

## Aop的作用

使非业务代码更加模块化，使模块更加内聚。减少重复的非业务代码。



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

