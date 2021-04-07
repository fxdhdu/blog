Java8引入了流式计算Stream，它的特点是支持并行地对集合进行处理。通过链式编程对处理的结果进行再处理。

并行流parallelStream底层使用Fork/Join框架，其实就是把任务分片，然后通过多线程来计算。任务会被提交到ForkJoinPool线程池中。



Stream 是一个惰性求值的系统



Jdk7 中引入的ForkJoin 并行框架的ForkJoinTask

可以通过虚拟机启动参数

```
-Djava.util.concurrent.ForkJoinPool.common.parallelism=N
```

来设置worker的数量。



自己思考：



## 参考

[Stream和parallelStream](https://blog.csdn.net/yy1098029419/article/details/89452380?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161599570716780357261035%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161599570716780357261035&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-1-89452380.pc_search_result_cache&utm_term=parallelStream)

