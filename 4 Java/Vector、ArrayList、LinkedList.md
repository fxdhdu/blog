[TOC]



## Java集合框架

![](/Users/fanxudong/IdeaProjects/blog/4 Java/assert/675536edf1563b11ab7ead0def1215c7.png)

集合的根Collection

```java
public interface Collection<E> extends Iterable<E> {

}
```

### List

#### Vector

- 线程安全，方法用synchronized修饰。
- 动态数组
- 对象数组、Object数组
- 扩容：新建、拷贝、1倍

```java
public class Vector<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
  protected Object[] elementData;
  
  public synchronized void addElement(E obj) {
          modCount++;
          add(obj, elementData, elementCount);
      }
  
}
```

##### stack

```java
public class Stack<E> extends Vector<E> {
  
}
```



#### ArrayList

ArrayList基于数组实现，是一个动态的数组队列。但是它和Java中的数组又不一样，它的容量可以自动增长，类似于C语言中动态申请内存，动态增长内存！

- 非线程安全
- 动态数组：顺序存储、随机访问（实现RandomAccess接口）；元素后移
- 扩容：50%

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final int DEFAULT_CAPACITY = 10;
  
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     */
    private int size;
  
    public boolean add(E e) {
        modCount++;
        add(e, elementData, size);
        return true;
    }
  
    private void add(E e, Object[] elementData, int s) {
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e; // 顺序存储
        size = s + 1;
    }
  
    private Object[] grow() {
        return grow(size + 1);
    }

    private Object[] grow(int minCapacity) {
        return elementData = Arrays.copyOf(elementData, newCapacity(minCapacity));
    }  
  
    private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1); // 扩容50%
        if (newCapacity - minCapacity <= 0) {
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                return Math.max(DEFAULT_CAPACITY, minCapacity);
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return minCapacity;
        }
        return (newCapacity - MAX_ARRAY_SIZE <= 0)
            ? newCapacity
            : hugeCapacity(minCapacity);
    }
}
```



#### LinkedList

- 双向链表
- LinkedList是基于链表实现的，从源码可以看出是一个双向链表。除了当做链表使用外，它也可以被当作堆栈、队列或双端队列进行操作。不是线程安全的，继承AbstractSequentialList实现List、Deque、Cloneable、Serializable。



### Set

#### TreeSet

#### HashSet

##### LinkedHashSet

### Queue/Deque



## 集合框架中的排序算法

Arrays.sort()

Collections.sort()



原始数据类型采用双轴快速排序

对于对象数据类型：TimSort，思想上也是一种归并和二分插入排序（binarySort）结合的优化排序算法。



并行排序算法（直接使用 parallelSort 方法）





内部排序：归并、交换（冒泡、快排）、选择、插入

外部排序：利用内存、外部存储处理超大数据集

算法稳定性

最好最差情况