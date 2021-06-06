[TOC]

## dubbo用户手册快速开始原理分析

https://dubbo.apache.org/zh/docs/v2.7/user/quick-start/



### provider启动流程

```java
public class Application {
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProviderConfiguration.class);
        context.start();
        System.in.read();
    }

    @Configuration
    @EnableDubbo(scanBasePackages = "org.apache.dubbo.demo.provider")
    @PropertySource("classpath:/spring/dubbo-provider.properties")
    static class ProviderConfiguration {
        @Bean
        public RegistryConfig registryConfig() {
            RegistryConfig registryConfig = new RegistryConfig();
            registryConfig.setAddress("zookeeper://127.0.0.1:2181");
            return registryConfig;
        }
    }
}
```

1. 使用AnnotationConfigApplicationContext类初始化spring容器，会先注册ProviderConfiguration这个bean。
2. refresh spring容器。
3. 执行ServiceClassPostProcessor中postProcessBeanDefinitionRegistry方法。通过扫描包路径，将xxx注册到spring容器中。解析出@DubboService 注解。也就是说dubbo是通过自定义BeanFactoryPostProcessor来把标有dubbo自定义注解的类，注册到spring容器中。



配置Bean上的@EnableDubbo注解。将dubbo配置对象注入容器中。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@EnableDubboConfig
@DubboComponentScan
public @interface EnableDubbo {
}
```

@EnableDubbo注解上实际起作用的两个注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Import(DubboConfigConfigurationRegistrar.class)
public @interface EnableDubboConfig {
}
```

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(DubboComponentScanRegistrar.class)
public @interface DubboComponentScan {
}
```

@Import注解可以和@Configuration注解配合生效。@Import注解实现**导入第三方的包的bean到容器**的功能。

DubboConfigConfigurationRegistrar类

```java
public class DubboConfigConfigurationRegistrar implements ImportBeanDefinitionRegistrar, ApplicationContextAware {
}
```

DubboComponentScanRegistrar类

```java
```





接口实现类上的 @DubboService 注解。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Inherited
public @interface DubboService {
}
```



解析类是ServiceAnnotationBeanPostProcessor



dubbo属性配置文件解析

属性：

- dubbo.application.name
- dubbo.protocol.name
- dubbo.protocol.port



### consumer启动流程



## 常见报错

```tex
java.lang.IllegalStateException: No registry config found or it's not a valid config! The registry config is: <dubbo:registry check="false" />
```

