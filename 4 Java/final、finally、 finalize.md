## final

final 修饰的 class 代表不可以继承扩展

final 的变量是不可以修改的

final 的方法也是不可以重写的（override）



1、lambda表达式中为什么要求外部变量为final？
2、java匿名内部类使用外部变量时，外部变量必须是final，为什么？

这是由Java对lambda表达式的实现决定的，在Java中lambda表达式是匿名类语法上的进一步简化，其本质还是调用对象的方法。
在Java中方法调用是值传递的，所以在lambda表达式中对变量的操作都是基于原变量的副本，不会影响到原变量的值。
综上，假定没有要求lambda表达式外部变量为final修饰，那么开发者会误以为外部变量的值能够在lambda表达式中被改变，而这实际是不可能的，所以要求外部变量为final是在编译期以强制手段确保用户不会在lambda表达式中做修改原变量值的操作。
另外，对lambda表达式的支持是拥抱函数式编程，而函数式编程本身不应为函数引入状态，从这个角度看，外部变量为final也一定程度迎合了这一特点。



## finally

try中的return语句和finally中代码的执行顺序。



## finalize

在对象被垃圾收集前调用

导致相应对象回收呈现数量级上的变慢

使用java.lang.ref.Cleaner代替finalize（利用幻象引用和引用队列）



## 参考

[第3讲 | 谈谈final、finally、 finalize有什么不同？](https://time.geekbang.org/column/article/6906)

《think in java》

https://blog.csdn.net/yangyifei2014/article/details/80068265

https://mp.weixin.qq.com/s/opzyyAdIGzXCC9Gw_AjtqQ