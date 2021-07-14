[TOC]



## Java集合框架

![](../asset/675536edf1563b11ab7ead0def1215c7.png)

集合的根Collection

```java
public interface Collection<E> extends Iterable<E> {

}
```

### List

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    protected transient int modCount = 0; //当前集合的版本号，每次修改(增、删)集合都会加1

}
```



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

ArrayList底层基于数组（Object[] elementData）实现，是一个容量可自动增长的动态的数组队列。

ArrayList的add操作，支持元素顺序存储，其内部记录了一个size变量，表示数组中元素的个数，同时新元素也可以直接放在size表示的下标中 。 get操作支持随机访问元素。

- 动态数组：顺序存储、随机访问（实现RandomAccess接口）；元素后移
- 扩容：50%
- 非线程安全

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
 
    public E get(int index) {
        Objects.checkIndex(index, size);
        return elementData(index);
    }  
  
    E elementData(int index) {
        return (E) elementData[index];
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

LinkedList是基于链表实现的，数据存储在双向链表中。除了当做链表使用外，它也可以被当作堆栈、队列或双端队列进行操作。不是线程安全的，继承AbstractSequentialList实现List、Deque、Cloneable、Serializable。

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient Node<E> first;

    transient Node<E> last;
  
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
  
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
  
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
}
```

LinkedList的迭代器。当使用foreach语法遍历LinkedList时，本质上是先调用hasNext判断是否有下一个元素，然后再调用next获取下一个元素。因此当在foreach遍历时，删除（remove）LinkedList的最后两个元素不会抛出ConcurrentModificationException异常。因为hasNext中size在remove后减小了，返回false，不会进入next的checkForComodification方法中。

```java
    private class ListItr implements ListIterator<E> {
        public boolean hasNext() {
            return nextIndex < size;
        }

        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }
    }
```

不同的是，在ArrayList中使用foreacch删除元素时，删除倒数第二个不会抛出异常。因为只有倒数第二个元素删除时，下一次hasNext返回false，则不会进入next方法。

```java
    private class Itr implements Iterator<E> {
        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
    }
```



#### 总结

ArrayList和LinkedList是Java集合框架下，列表（不是链表，链表是底层数据结构）接口List的两种不同实现。

|          | ArrayList                                    | LinkedList             |
| -------- | -------------------------------------------- | ---------------------- |
| 数据结构 | Object数组                                   | 双向链表               |
|          | 非线程安全                                   | 非线程安全             |
| 访问     | get方法随机访问                              | get方法顺序访问        |
| 插入删除 | 顺序存储，直接在数组末尾插入。数组满时扩容。 | 可在链表头部或尾部插入 |

![å¨è¿éæå¥å¾çæè¿°](https://img-blog.csdnimg.cn/20190109164948676.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5ODk5MDg3,size_16,color_FFFFFF,t_70)

https://blog.csdn.net/binbigdata/article/details/92805197

### Set

#### TreeMap

存储结构

```java
    
		private transient Entry<K,V> root;

		static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;
        Entry<K,V> right;
        Entry<K,V> parent;
        boolean color = BLACK;
    }
```



#### TreeSet

有序的Set集合

TreeSet中的元素支持2种排序方式：自然排序 或者 根据创建TreeSet 时提供的 Comparator 进行排序。

TreeSet是非同步的

它的iterator 方法返回的迭代器是fail-fast的

- 主要方法
  - add：把节点添加到树上
  - remove：若传了Comparator，删除时根据Comparator比较，而不是Hash值
  - pollFirst：获取最左元素，并删除

![img](https://img-blog.csdnimg.cn/20190710200359248.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ExNDM5Nzc1NTIw,size_16,color_FFFFFF,t_70)

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{
    private transient NavigableMap<E,Object> m;
  
}
```

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
{
}
```



#### HashSet

#### LinkedHashSet

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