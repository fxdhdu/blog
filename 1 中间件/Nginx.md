## Nginx

Nginx 是一个高性能的 Web 和反向代理服务器, 它具有很多非常优越的特性:

作为 **Web 服务器**：相比 Apache，Nginx 使用更少的资源，支持更多的并发连接，体现更高的效率，这点使 Nginx 尤其受到虚拟主机提供商的欢迎。能够支持高达 50,000 个并发连接数的响应，感谢 Nginx 为我们选择了 epoll and kqueue 作为开发模型.

作为**负载均衡服务器**：Nginx 既可以在内部直接支持 Rails 和 PHP，也可以支持作为 HTTP代理服务器 对外进行服务。Nginx 用 C 编写, 不论是系统资源开销还是 CPU 使用效率都比 Perlbal 要好的多。

nginx常用做**静态内容服务**和**代理服务器**（反向代理服务器），直面外来请求转发给后面的应用服务（tomcat，django什么的），tomcat更多用来做做一个应用容器，让java web app跑在里面的东西，对应同级别的有jboss,jetty等东西。



项目中的nginx用作Web服务器，请求不直接到达tomcat，先到达nginx，nginx会缓存静态资源，减轻tomcat的负担。

## 反向代理（Reverse Proxy）

**反向代理（Reverse Proxy）**方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

正向代理，代理服务器部署在客户端侧。反向代理，代理服务器部署在服务端侧。

![img](https://images2015.cnblogs.com/blog/305504/201611/305504-20161112125907030-1432469707.png)

![ScreenClip](/var/folders/8d/b6f6k4dj6cvcwxtgnc22zpsr0000gn/T/com.yinxiang.Mac/com.yinxiang.Mac/WebKitDnD.62NtRI/ScreenClip.png)



## nginx.conf

- 全局配置

- HttpGzip模块配置

- http服务器配置

- 负载均衡配置

  upstream是nginx的http upstream模块，这个模块通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡。
  - **upstream** myserver：字面意思为上游。用于指定负载均衡的名称为myserver。
  - 负载均衡算法

    - 轮训（默认）
    - 最少连接算法：将请求定向到当时具有最少活动连接的服务器。
    - ip hash：ip_hash调度算法，每个请求按IP的hash结果分配，同一个IP访问固定后端服务器，解决了动态服务器存在的session共享问题
  - 定义server的权重：weight
  - **server**
  - **location**

  > 1、注册7层负载均衡，实例的ip+后端端口，会被写到nginx的upstream中。upstream的名称，通过region+组件+schema等确定。
  >
  > 2、为什么域名+端口+location唯一：

- http_proxy设置

- server虚拟主机配置

- URL配置设置



## 参考

[nginx命令等](http://www.nginx.cn/nginxchscommandline )

[nginx中文文档](http://www.nginx.cn/doc/ )

[正向代理反向代理总结](https://www.cnblogs.com/Anker/p/6056540.html )

Nginx服务器配置文件nginx.conf实例详解：https://blog.csdn.net/field_yang/article/details/52278390

如何使用nginx配置负载均衡：http://www.nginx.cn/4996.html

nginx负载均衡配置：http://www.nginx.cn/78.html

https://blog.csdn.net/JackLiu16/article/details/79444327

