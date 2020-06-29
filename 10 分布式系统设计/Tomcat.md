Tomcat是支持Servlet规范的Web容器。所以它支持Java Servlet规范中定义的Servlet Filter组件。

定义一个实现 javax.servlet.Filter 接口的过滤器类，并且将它配置在 web.xml 配置文件中。Web 容器启动的时候，会读取 web.xml 中的配置，创建过滤器对象。当有请求到来的时候，会先经过过滤器，然后才由 Servlet 来处理。

![img](https://static001.geekbang.org/resource/image/32/21/3296abd63a61ebdf4eff3a6530979e21.jpg)

