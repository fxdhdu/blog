## 启动过程

Web容器启动（web.xml）  ->  Spring容器启动

### contextConfigLocation

用于配置spring容器配置文件的地址。

### ContextLoaderListener

在web容器启动时，监听读取spring的配置文件。等价于在编程方式下，直接通过new ClassPathXmlApplicationContext来读去spring的配置文件。

### ServletContext





## Spring MVC源码阅读

浏览器发出http请求经过xx、xx、xx

表单

servlet拦截所有的url

http请求到达DispatcherServlet

DispatcherServlet在函数xx中查询HandlerMapping

请求分发

视图

ModelAndView





扫描使用了@Controller注解的类的方法

检查是否使用了@RequestMapping注解

## 思考问题

Spring mvc如何转换postman发送的json体

1、返回html标签页面

2、返回json、xml格式

3、HttpMessageConvert

4、RequestMapping解析原理

## servlet的生命周期

