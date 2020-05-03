## 强引用
  - 必需对象，Object obj = new Object()
  - 不会被垃圾收集器回收

## 软引用
  - 有用但非必需对象(例如系统缓存功能)
  - 系统将要发生内存溢出异常前，会对这些对象进行二次回收
  - SoftReference  

## 弱引用
  - 非必需对象
  - 只能生存到下一次垃圾收集之前
  - WeakReference

## 虚引用、幽灵引用、幻影引用
  - 能在对象被收集器回收时收到一个系统通知
  - PhantomReference
  - java.lang.ref.Cleaner



对引用进行分类，可以优化垃圾回收。

不同的引用类型，主要体现的是对象不同的可达性（reachable）状态和对垃圾收集的影响。