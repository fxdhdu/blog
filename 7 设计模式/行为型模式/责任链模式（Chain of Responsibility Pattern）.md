## 责任链模式（Chain of Responsibility Pattern）


 为**请求**创建了一个**接收者**对象的链。这种模式给予请求的类型，对请求的发送者和接收者进行解耦。这种类型的设计模式属于**行为型模式**。

在这种模式中，通常**每个接收者都包含对另一个接收者的引用**。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。

**特点**

有一个set方法，用于设置下一个拦截器。

**应用**

1. activiti中使用了职责链模式，构造了命令拦截器链，用于在命令真正被执行之前做一系列操作。
2. spring aop 拦截器
3. Servlet Filter 的 Filter 和 FilterChain。递归来实现拦截方法前后添加逻辑（在一个doFilter中实现双向拦截？）
4. Spring Interceptor 的HandlerInterceptor 和 HandlerExecutionChain。把拦截方法前后要添加的逻辑放到两个方法中实现
5. MyBatis Plugin 的 Interceptor 和 InterceptorChain。采用嵌套动态代理的方法来实现

## 参考	

[责任链模式](http://www.runoob.com/design-pattern/chain-of-responsibility-pattern.html)

