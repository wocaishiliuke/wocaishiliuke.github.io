---
title: Set
date: 2018-01-14 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Collection的子接口Set的相关成员。

<!-- more -->

##### 目录
- [X] I.Set
- [X] II.HashSet
- [X] III.LinkedHashSet
- [X] IV.TreeSet
- [X] V.EnumSet
- [X] VI.参考


---

# I.Set

Set是Collection的子接口。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/Set.png)

- 无索引
- 元素唯一（List的add()永远返回true，而Set的可能是false）
- 存取无序（LinkedHashSet存取顺序一致）

```java
public interface Set<E> extends Collection<E> {
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);

    // Modification Operations
    boolean add(E e);
    boolean remove(Object o);

    // Bulk Operations
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean retainAll(Collection<?> c);
    boolean removeAll(Collection<?> c);
    void clear();

    // Comparison and hashing
    boolean equals(Object o);
    int hashCode();

    //返回Set的并行流Spliterator迭代器
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.DISTINCT);
    }
}
```

Set中没有特殊的方法，和Collection一样。Java中的Set跟数学中的集合概念一致（无序性、互斥性、确定性）。学习Set的关键在于，如何保证元素唯一（也是Map如何保证键唯一）

> Set接口也有抽象层AbstractSet类，实现一些通用方法。


---

# II.HashSet

**HashSet依靠HashMap来实现，其中HashMap所有的value都是同一个Object对象。HashSet保证元素唯一的方式，也是HashMap保证key唯一的方式：哈希算法**。

> 这里不说明HashSet保证元素唯一的原理，详见HashMap。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/HashSet.png)

## 1.特点

HashSet实现了Set接口，内部是通过HashMap实现的，决定了HashSet有如下特点：
- 无索引、元素唯一、存取无序
- 添加、删除、判断元素是否存在，效率很高，时间复杂度为O(1)
- 可以存null

基于以上特点，HashSet一般有以下使用场景：
- 乱序排重：如果对排重后的元素没有顺序要求，可以使用HashSet
- 集合运算：进行数学集合中的运算，如交集、并集等

## 2.成员变量

```java
public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable {
    private transient HashMap<E,Object> map;
    // 底层HashMap的傀儡value
    private static final Object PRESENT = new Object();
}
```

## 3.构造方法

```java
//无参构造，调用HashMap的无参构造（初始容量16，负载因子0.75）
public HashSet() {
    map = new HashMap<>();
}
//有参构造，调用HashMap(int initialCapacity)（初始容量≥16，负载因子0.75）
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
//有参构造，调用HashMap(int initialCapacity)（负载因子0.75）
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}
//有参构造，调用HashMap(int initialCapacity, float loadFactor)，自定义容量和负载因子
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}
```

## 4.常用方法

#### 4.1 add(E e)

如果set中没有该元素（HashMap中没有该key），则加入返回true（HashMap返回null），如果已经存在（HashMap返回PRESENT），保持set不变，返回false。

> HashMap.put(k,v)依赖hashCode()和equals()，返回值是v。所以HashMap的key类型（即HashSet的元素类型）需要重写hashCode()和equals()。

```java
public boolean add(E e) {
    // PRESENT就是傀儡value
    return map.put(e, PRESENT)==null;
}
```

#### 4.2 remove(Object o)

调用HashMap.remove()，返回PRESENT，表示Map中删除成功。

```java
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
```

#### 4.3 contains(Object o)

调用HashMap.containsKey(o)，判断key是否存在。

```java
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

## 5.遍历

Set没有索引，所以HashSet只能使用Collection共有的迭代方式：转数组和迭代器（包括foreach）。


---

# III.LinkedHashSet

和HashSet依赖HashMap一样，LinkedHashSet内部依赖LinkedHashMap实例而实现。只是value都是同一个特定值PRESENT，所有key组成了LinkedHashSet的元素，并能保证元素**按插入有序**。

> **LinkedHashSet不仅元素唯一，也是唯一一个存取顺序一致的Set（无索引）**。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/LinkedHashSet.png)

> HashSet基于HashMap实现，HashSet中的方法实际操作的，是内部的HashMap实例。如果让HashSet中的方法操作LinkedHashMap实例，就能轻松的实现LinkedHashSet。实际Java API中也是这样实现的，**通过HashSet.HashSet()方法完成狸猫换太子（LinkedHashMap换HashMap）**。

## 1.特点

实现了Set接口，内部是通过LinkedHashMap实现，这也决定了LinkedHashSet有如下特点：

- 无索引、元素唯一、存取有序
- 添加、删除、判断元素是否存在，效率很高，时间复杂度为O(1)
- 可以存null

以下是LinkedHashSet的全部源码。没有什么特有方法，除了构造和spliterator()，都继承自HashSet。

```java
public class LinkedHashSet<E> extends HashSet<E> implements Set<E>, Cloneable, java.io.Serializable {

    private static final long serialVersionUID = -2851667679971038690L;

    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }

    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }

    public LinkedHashSet() {
        super(16, .75f, true);
    }

    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }

    @Override
    public Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.DISTINCT | Spliterator.ORDERED);
    }
}
```

## 2.成员变量

全部继承自HashSet（同上），即：

```java
private transient HashMap<E,Object> map;
private static final Object PRESENT = new Object();//傀儡value
```

## 3.构造方法

4个构造都是通过super调用父类HashSet中的default HashSet()构造实现的。

```java
public LinkedHashSet() {
    super(16, .75f, true);
}

public LinkedHashSet(Collection<? extends E> c) {
    super(Math.max(2*c.size(), 11), .75f, true);
    addAll(c);
}

public LinkedHashSet(int initialCapacity) {
    super(initialCapacity, .75f, true);
}

public LinkedHashSet(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor, true);
}
```

```java
// HashSet
public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable {
    // 是个default构造方法（相当于钩子函数）
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
    ...
}
```

**在父类HashSet中的default HashSet()中，创建LinkedHashMap对象，赋给map引用，完成"狸猫换太子"**。之后对map引用的操作，实际操作的都是LinkedHashMap实例。

## 4.常用方法

其他方法都继承自HashSet，只不过map指向了LinkedHashMap实例（多态），不再是HashMap实例。详见上述。


---
# IV.TreeSet

和HashSet依赖HashMap一样，TreeSet基于TreeMap实例实现（也可以说红黑树）。所以TreeSet相比于HashSet，最突出的特点是，可以保证元素存取有序。

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/DataStructure/TreeSet.png)

TreeSet继承了AbstractSet，方便实现Set接口中的基本方法。另外TreeSet实现了NavigableSet接口，NavigableSet接口又继承了SortedSet接口，所以和TreeMap相似，TreeSet可以很方便的获取一定范围内的元素。

> TreeSet特点

- 实现自Set接口，具有Set的一切特性，如元素不可重复
- 相比于HashSet，TreeSet元素之间是有序的
- 基于TreeMap实现，所以添加、删除、判断元素是否存在，效率较高，时间复杂度为O(log2_T)

## 1.成员变量

TreeSet内部依靠NavigableMap类型的引用m完成各种操作，m实际接收的是TreeMap实例，所以说TreeSet基于TreeMap实现。PRESENT是傀儡value。

```java
/**
 * TreeSet是基于TreeMap的一个NavigableSet接口实现。元素间排序依靠自然排序或比较器排序。
 * TreeSet的基本操作，如add、remove、contains，仅需log(n)时间消耗。
 */
public class TreeSet<E> extends AbstractSet<E> implements NavigableSet<E>, Cloneable, java.io.Serializable {

    private static final long serialVersionUID = -2479143000061671589L;
    //The backing map.
    private transient NavigableMap<E,Object> m;
    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
    
    // methods
}
```

## 2.构造函数

```java
/*
* 空参构造。构造顺序：
* 1.创建TreeMap实例，即访问TreeMap的空参构造
* 2.调用TreeSet(NavigableMap<E,Object> m)构造，将TreeMap实例赋给成员变量m
*/
public TreeSet() {
    this(new TreeMap<E,Object>());
}

/*
* 通过比较器构造TreeSet。构造顺序：
* 1.使用比较器创建TreeMap实例，即访问TreeMap的有参构造
* 2.调用TreeSet(NavigableMap<E,Object> m)构造，将TreeMap实例赋给成员变量m
*/
public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}

/*
* 通过Collection集合构造TreeSet。构造顺序：
* 1.访问本类的空参构造，创建TreeSet实例，完成m的赋值
* 2.插入元素
*/
public TreeSet(Collection<? extends E> c) {
    this();
    addAll(c);
}

/*
* 通过SortedSet有序集合构造TreeSet。构造顺序：
* 1.使用SortedSet的比较器，访问本类的有参构造，创建TreeSet实例，完成m的赋值
* 2.插入元素
*/
public TreeSet(SortedSet<E> s) {
    this(s.comparator());
    addAll(s);
}
```

```java
// default的构造方法（包访问权限）。用于辅助前两个构造，完成m的赋值
TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}
```

## 3.实现接口

#### 3.1 SortedSet

```java
public interface SortedSet<E> extends Set<E> {

    // 返回SortedSet中用于元素排序的比较器，使用自然排序时comparator为null
    Comparator<? super E> comparator();

    /**
     * 返回SortedSet中在[fromElement,toElement)范围的所有元素组成的子SortedSet视图
     * 注意：对视图的改变会影响SortedSet的内容，反之亦然
     * @throws 当fromElement和toElement不能和当前set中元素比较时（自然排序、比较器排序），抛ClassCastException
     * @throws 当fromElement或toElement为null，并且该SortedSet不允许null时，抛NullPointerException
     * @throws 当fromElement大于toElement，或者它们在该SortedSet本身范围之外时，抛IllegalArgumentException
     */
    SortedSet<E> subSet(E fromElement, E toElement);

    /**
     * 返回SortedSet中小于toElement的所有元素组成的子SortedSet视图
     * 注意：对视图的改变会影响SortedSet的内容，反之亦然
     * @throws ClassCastException 同上
     * @throws NullPointerException 同上
     * @throws IllegalArgumentException 同上
     */
    SortedSet<E> headSet(E toElement);

    /**
     * 返回SortedSet大于等于fromElement的所有元素组成的子SortedSet视图
     * 注意：对视图的改变会影响SortedSet的内容，反之亦然
     * @throws ClassCastException 同上
     * @throws NullPointerException 同上
     * @throws IllegalArgumentException 同上
     */
    SortedSet<E> tailSet(E fromElement);

    /**
     * 返回该set中第一个元素
     * @throws NoSuchElementException 该set为空时
     */
    E first();

    /**
     * 返回该set中最后一个元素
     * @throws NoSuchElementException 该set为空时
     */
    E last();

    // Java8新方法，返回SortedSet的Spliterator迭代器
    @Override
    default Spliterator<E> spliterator() {
        return new Spliterators.IteratorSpliterator<E>(
                this, Spliterator.DISTINCT | Spliterator.SORTED | Spliterator.ORDERED) {
            @Override
            public Comparator<? super E> getComparator() {
                return SortedSet.this.comparator();
            }
        };
    }
}
```

#### 3.2 NavigableSet

```java
public interface NavigableSet<E> extends SortedSet<E> {
    /**
     * 返回NavigableSet中严格小于e的最大元素
     * @throws ClassCastException 当e不能和该set中的元素比较时
     * @throws NullPointerException 当e为null，且该set不允许null
     */
    E lower(E e);
    /**
     * 返回NavigableSet中严格大于e的最大元素
     * @throws ClassCastException 当e不能和该set中的元素比较时
     * @throws NullPointerException 当e为null，且该set不允许null
     */
    E higher(E e);
    /**
     * 返回NavigableSet中小于等于e的最大元素
     * @throws ClassCastException 当e不能和该set中的元素比较时
     * @throws NullPointerException 当e为null，且该set不允许null
     */
    E floor(E e);
    /**
     * 返回NavigableSet中大于等于e的最大元素
     * @throws ClassCastException 当e不能和该set中的元素比较时
     * @throws NullPointerException 当e为null，且该set不允许null
     */
    E ceiling(E e);

    /** 删除并返回NavigableSet中第一个元素，如果NavigableSet为空，返回null */
    E pollFirst();
    /** 删除并返回NavigableSet中最后一个元素，如果NavigableSet为空，返回null */
    E pollLast();

    /** 返回NavigableSet的正向迭代器 */
    Iterator<E> iterator();
    /** 返回NavigableSet的逆序迭代器，等效于descendingSet().iterator() */
    Iterator<E> descendingIterator();

    /**
     * 返回NavigableSet的逆序的NavigableSet视图
     * 注意：对视图的改变会影响SortedSet的内容，反之亦然
     */
    NavigableSet<E> descendingSet();
    /**
     * 返回NavigableSet中大于fromElement小于toElement的元素组成的子NavigableSet视图，
     * fromInclusive、toInclusive表示是否包含边界
     * 注意：对视图的改变会影响SortedMap的内容，反之亦然
     * @throws 当fromElement和toElement不能和当前set中元素比较时（自然排序、比较器排序），抛ClassCastException
     * @throws 当fromElement或toElement为null，并且该SortedSet不允许null时，抛NullPointerException
     * @throws 当fromElement大于toElement，或者它们在该SortedSet本身范围之外时，抛IllegalArgumentException
     */
    NavigableSet<E> subSet(E fromElement, boolean fromInclusive,
                           E toElement,   boolean toInclusive);
    /**
     * 返回NavigableSet中小于toElement的所有元素组成的子NavigableSet视图，inclusive表示是否包含边界
     * 对视图的改变会影响SortedMap的内容，反之亦然
     * @throws ClassCastException 同上
     * @throws NullPointerException 同上
     * @throws IllegalArgumentException 同上
     */
    NavigableSet<E> headSet(E toElement, boolean inclusive);
    /**
     * 返回NavigableSet中大于toElement的所有元素组成的子NavigableSet视图，inclusive表示是否包含边界
     * 对视图的改变会影响SortedMap的内容，反之亦然
     * @throws ClassCastException 同上
     * @throws NullPointerException 同上
     * @throws IllegalArgumentException 同上
     */
    NavigableSet<E> tailSet(E fromElement, boolean inclusive);

    /** 等价于subSet(fromElement, true, toElement, false) */
    SortedSet<E> subSet(E fromElement, E toElement);
    /** 等价于headSet(toElement, false) */
    SortedSet<E> headSet(E toElement);
    /** 等价于tailSet(fromElement, true) */
    SortedSet<E> tailSet(E fromElement);
}
```

## 4.常用方法

#### 4.1 add(E e)

调用TreeMap.put()，e为TreeMap的key，值为PRESENT，put返回null表示原来没有该键，添加成功。

```java
/**
 * 添加元素。e已经存在时，保持该set不变，返回false
 * @throws ClassCastException 当e不能和该set中的元素比较时（自然排序、比较器排序）
 * @throws NullPointerException 当e为null，且该set不允许null时
 */
public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
```

#### 4.2 remove(Object o)

调用TreeMap.remove()，返回PRESENT表示原来有对应的键且删除成功。

```java
/**
 * 删除元素。原本存在时，删除成功返回true
 * @throws ClassCastException 当e不能和该set中的元素比较时（自然排序、比较器排序）
 * @throws NullPointerException 当e为null，且该set使用自然排序时，或比较器排序时比较器不允许null时
 */
public boolean remove(Object o) {
    return m.remove(o)==PRESENT;
}
```

#### 4.3 contains(Object o)

调用TreeMap.containsKey()，判断TreeMap中是否包含对应的键。

```java
/**
 * 是否包含元素o
 * @throws ClassCastException 当e不能和该set中的元素比较时（自然排序、比较器排序）
 * @throws NullPointerException 当e为null，且该set使用自然排序时，或比较器排序时比较器不允许null时
 */
public boolean contains(Object o) {
    return m.containsKey(o);
}
```

#### 4.4 subSet

调用TreeMap.subMap()获取TreeMap的子视图，然后调用TreeSet的构造函数初始化TreeSet。

```java
/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException 当fromElement或toElement为null，且该set使用自然排序时，或比较器排序时比较器不允许null时
 */
public NavigableSet<E> subSet(E fromElement, boolean fromInclusive,
                              E toElement,   boolean toInclusive) {
    return new TreeSet<>(m.subMap(fromElement, fromInclusive,
                                   toElement,   toInclusive));
}
```

TreeMap的subMap()的返回值是一个NavigableMap对象，该对象内部的红黑树其实跟原TreeMap内部的红黑树共享同一片内存空间，所以subMap()返回的是原TreeMap的一个子视图，对视图的修改会同步影响到原TreeMap。调用TreeSet的构造函数，将subMap方法返回的NavigableMap对象作为参数传入，所以subSet是整个TreeSet的子视图。

> 其实TreeSet跟HashSet一样，实现都比较简单。HashSet主要通过调用HashMap的方法来实现，TreeSet主要是通过调用NavigableMap（TreeMap）的方法来实现。


---
# V.EnumSet

参考[EnumSet](http://lidol.top/java/1037/)。

---
# VI.参考

- [Java编程拾遗『TreeMap』](http://lidol.top/java/detail/986/)