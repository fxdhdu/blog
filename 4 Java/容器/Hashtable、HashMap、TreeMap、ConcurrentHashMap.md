[TOC]

## Hashtable、HashMap、TreeMap的区别

### HashTable

HashTable对元素的操作是线程安全的，其put、get等方法都加了synchronized关键字。不支持null键、值，传入null时会抛出空指针异常。

```java
    public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode(); // key不能为null，否则这里抛出空指针
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
```

### HashMap

【背】

java中的HashMap，内部数据存储结构，在Java8与Java7中略有不同。

Java7中为数组 + 链表的方式存储数据。即元素存储在一个Entry数组中，而每个Entry为一个单向链表。Java8中为了降低在链表上查找数据的开销，当链表中元素达到8个时，将链表(Node)转换成红黑树(TreeNode)。使查找的复杂度从O(n)降为O(logn)

HashMap在调用构造函数时，设置容量和负载因子。容量用来初始化数组的大小，默认值为16，负载因子默认值为0.75f。容量 * 负载因子即为HashMap的阈值，表示能够存放元素的个数，当存放的元素超出阈值时，HashMap会发生扩容。容量始终保持2的幂数，当用户提供的容量大小不是2的幂时，构造函数中会调用tableSizeFor方法，将传入容量向上取最近的2的n次方。扩容时容量为原来的2倍。HashMap这么做，一方面是为了使散列更加均匀 （存储下标计算为（n-1）& hash），另一方面是为了扩容迁移链表元素时，可以方便的通过hash值的位判断来确定是否需要重新计算下标，把链表分为low和high两个链表，low链表下标不变，high链表。

HashMap首次调用put方法时，会通过resize方法初始化数组。后续每次执行put方法时，先计算key的hash值，然后根据公式（n-1）& hash 计算元素存放的下标，此时如果没有发生hash冲突则直接放在数组下标元素位置。如果发生冲突需要判断是否为同一个元素，同一个元素需要根据onlyIfAbsent判断是否需要覆盖元素。不存在相同元素则将新元素放入链表中或插入红黑树中。元素插入后，判断是否容量超过阈值，超过则调用resize进行扩容。 

#### java 8

- 构造函数
  - 负载因子：默认为0.75f
  - 容量：始终保持2的幂数、默认初始化容量16。可以扩容，扩容后数组大小为当前的 2 倍。
    - 2的幂可以使散列更加均匀(结合下标计算方法考虑，2^n -1 二进制各个位都为1)
    - 2的幂可以使扩容时，通过位判断来确定是否需要重新计算下标
- put方法 （常数时间复杂度；非同步方法，多线程下不安全（可能会死循环？））
  - 元素哈希值计算`(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16); `支持null键、值`
  - 初始化（lazy-load原则，put方法中初始化而不是在构造函数中）
  - 元素存储存放下标计算`(n - 1) & hash`
  - “拉链法”解决哈希冲突
  - 重复Key值判断
  - 树化
    - 阈值：`static final int TREEIFY_THRESHOLD = 8;` final类型不可修改
  - 扩容
    - 阈值：容量 * 负载因子，put元素时size超过阈值则扩容。
    - 容量：原来的2倍
    - 数据迁移
    - 重新计算下标
- get方法（基于数组下标随机访问数据的特性，常数时间复杂度）

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
  
    private static final long serialVersionUID = 362498820763181265L;

    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    static final int MAXIMUM_CAPACITY = 1 << 30;

    static final float DEFAULT_LOAD_FACTOR = 0.75f; // loadFactor的默认值

    static final int TREEIFY_THRESHOLD = 8; // 树化阈值，插入链表第8个元素时，会进行树化。

    static final int UNTREEIFY_THRESHOLD = 6;

    static final int MIN_TREEIFY_CAPACITY = 64;
  
    //HashMap是一个数组，数组中每个元素是一个单向链表。
    //单向链表中每个节点是，嵌套类 Entry 的实例。
    //Entry 包含四个属性：key, value, hash 值和用于单向链表的 next。
    transient Node<K,V>[] table;
  
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
    }
  
    int threshold; // 扩容的阈值，等于 capacity * loadFactor
  
    final float loadFactor; //负载因子  
  
    transient int size; //键值对数目
  
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16); //key可以为null
    }
 
// 第四个参数 onlyIfAbsent 如果是 true，那么只有在不存在该 key 时才会进行 put 操作。key存在时，返回oldValue。
// 第五个参数 evict 我们这里不关心  
  final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab;  // 指向table的引用
        Node<K,V> p; 
        int n,  ；// tab的长度
        i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length; // 第一次执行
     // 找到具体的数组下标，如果此位置没有值，那么直接初始化一下 Node 并放置在这个位置就可以了
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else { // 数组该位置有数据
            Node<K,V> e; K k;
          // 首先，判断该位置的第一个数据和我们要插入的数据，key 是不是"相等"，如果是，取出这个节点
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode) // 红黑树
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
              // 到这里，说明数组该位置上是一个链表
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                      // TREEIFY_THRESHOLD 为 8，所以，如果新插入的值是链表中的第 8 个
                    // 会触发下面的 treeifyBin，也就是将链表转换为红黑树
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                  // 如果在该链表中找到了"相等"的 key(== 或 equals)
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                      // 此时 break，那么 e 为链表中[与要插入的新值的 key "相等"]的 node
                        break;
                    p = e;
                }
            }
        // e!=null 说明存在旧值的key与要插入的key"相等"
        // 对于我们分析的put操作，下面这个 if 其实就是进行 "值覆盖"，然后返回旧值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
    // 如果 HashMap 由于新插入这个值导致 size 已经超过了阈值，需要进行扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }  
  
}
```



### LinkedHashMap(有序，HashMap + 双向链表)

LinkedHashMap数据存储是有序的，accessOrder=true，表示按访问顺序排序，accessOrder=false，表示按插入顺序排序即调用put方法的顺序

LinkedHashMap将HashMap中的Node结构扩展为双向链表。并覆盖了HashMap中的newNode方法。因此在创建新节点时，会将新节点连接到双向链表末尾。

如果是以访问顺序排序，则get一个节点以后，会把此节点移动到双向链表末尾。

LinkedHashMap中，实现了afterNodeInsertion方法，在新元素put之后支持删除eldestentry。因此可以用LinkedHashMap实现LRU缓存。




```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{
		final boolean accessOrder;
  
    transient LinkedHashMap.Entry<K,V> head;

    transient LinkedHashMap.Entry<K,V> tail;
  
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }  
  
}
```

[图解LinkedHashMap原理](https://www.jianshu.com/p/8f4f58b4b8ab)





### TreeMap（TreeSet）

- 红黑树
- 顺序访问原始（通过平衡二叉树的遍历）
- 指定Comparator、Comparable（自然顺序）



## ConcurrentHashMap

ConcurrentHashMap是java并发包中，线程安全的HashMap实现。

在Java7中ConcurrentHashMap通过分段锁思想实现线程安全。数据是以一个 Segment 数组，Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

多少个Segment，理论上就支持最多多少个线程并发写入。

在Java8中ConcurrentHashMap存储结构上与HashMap基本一样，使用数组+链表/红黑树。

ConcurrentHashMap初始化方法中的并发问题是通过对 sizeCtl 进行一个 CAS 操作来控制的。sizeCtl < 0表示有其他现在在进行初始化，调用Thread.yield()，进行自旋。否则通过CAS操作将sizeCtl设置为-1，CAS成功则对Node数组进行初始化

put方法中，通过一个for循环放入新值。如果数组对应下标位置为空，则用CAS操作将新值放入数组，如果成功则break循环，如果失败，表示有并发操作，则进入下一个循环。如果数组对应下标位置中有元素，且hash值为MOVED，则表示正在进行扩容，调用helpTransfer方法帮助迁移数据。 否则使用synchronized锁住数组该位置的头节点（并发上限和数组大小有关），将新节点插入链表或红黑树。

ConcurrentHashMap 树化方法treeifyBin，不一定就会进行红黑树转换，也可能是仅仅做数组扩容。数组长度>=64时进行树化。树化要通过synchronized对头节点加锁。

扩容

数据迁移

stride步长，表示当前线程需要迁移的hash桶的数目。

transferIndex标识数组上hash桶的迁移工作已经进行到哪个位置。

sizeCtl 用于记录当前并发扩容的线程数量。

用来安排哪个线程执行哪个任务，将大的迁移任务分成一个个任务包。

[ConcurrentHashMap的transfer](https://blog.csdn.net/luzhensmart/article/details/105968886)

![4](../assert/Java8ConcurrentHashMap结构.png)

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
  
  //并行级别、并发数、Segment 数。默认是 16，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
    private static final float LOAD_FACTOR = 0.75f;
  
  
      static class Segment<K,V> extends ReentrantLock implements Serializable {
        private static final long serialVersionUID = 2249069246763182397L;
        final float loadFactor;
        Segment(float lf) { this.loadFactor = lf; }
    }
  
    public ConcurrentHashMap() {
    }  
  
    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }  
  
      static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;
    }
  
    static final class TreeNode<K,V> extends Node<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
    }

    /**
     * A node inserted at head of bins during transfer operations.
     */
    static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
    }
}
```



## hashCode和equals的基本约定

对象的散列码是为了更好的支持基于哈希机制的Java集合类，例如 Hashtable, HashMap, HashSet 等。

### 原始的hashCode

Object中，equals默认使用==比较两个对象的地址。只有this 和 obj引用同一个对象，才会返回true。

```java
public boolean equals(Object obj) {
	return (this == obj);
}
```

### 原始的equals

Object中，hashCode默认使用对象的地址计算散列码；

```java
public native int hashCode();
```

重写equals和hashCode

2. 重写equals()方法需要满足5个约定

   - 自反性：x.equals(x) 一定是true
   - 对null：x.equals(null) 一定是false
   - 对称性：x.equals(y) 和 y.equals(x)结果一致
   - 传递性：a 和 b equals , b 和 c equals，那么 a 和 c也一定equals。
   - 一致性：在某个运行时期间，2个对象的状态的改变不会不影响equals的决策结果，那么，在这个运行时期间，无论调用多少次equals，都返回相同的结果。

3. 重写hashCode方法的3个要求为

   - 在某个运行时期间，只要对象的（字段的）变化不会影响equals方法的决策结果，那么，在这个期间，无论调用多少次hashCode，都必须返回同一个散列码。

   - **通过equals调用返回true 的2个对象的hashCode一定一样。**（重写equals必需重写hashCode的原因之一）

   - 通过equasl返回false 的2个对象的散列码不需要不同，也就是他们的hashCode方法的返回值允许出现相同的情况。

   总结一句话：等价的(调用equals返回true)对象必须产生相同的散列码。不等价的对象，不要求产生的散列码不相同。
   
### 相关编程规范

![](../assert/8ECBE9B8-702E-47DE-B590-479A46FC8916.png)



## 哈希码的有效性



## 参考

[一次性搞清楚equals和hashCode](https://mp.weixin.qq.com/s/uQEdjCq-zP5C8Z5jwHyoag)

[Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析](https://www.javadoop.com/post/hashmap)

[20 | 散列表（下）：为什么散列表和链表经常会一起使用？](https://time.geekbang.org/column/article/64858)

[安琪拉](https://mp.weixin.qq.com/s/oRx-8XXbgage9Hf97WrDQQ)

