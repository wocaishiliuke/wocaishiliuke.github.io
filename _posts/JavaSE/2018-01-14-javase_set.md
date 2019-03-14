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


---

# I.Set

Set是Collection的子接口。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/Set.png)

- 无索引
- 元素唯一（List的add()永远返回true，而Set的可能是false）
- 无序，存取不一致

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

- 无索引；元素唯一；无序，存取不一致
- 添加、删除、判断元素是否存在，效率很高，时间复杂度为O(1)
- 可以存null

基于以上特点，HashSet一般有以下使用场景：

- 乱序排重：如果对排重后的元素没有顺序要求，可以使用HashSet
- 集合运算：进行数学集合中的运算，如交集、并集等

## 2.成员变量

```java
public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable {
private transient HashMap<E,Object> map;
// 底层Map的傀儡值v
private static final Object PRESENT = new Object();
}
```

## 3.构造方法

```java
//无参构造，调用HashMap的无参构造（初始容量16，负载因子0.75）
public HashSet() {
    map = new HashMap<>();
}
//有参构造，调用HashMap(int initialCapacity)有参构造（初始容量≥16，负载因子0.75）
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
//有参构造，调用HashMap(int initialCapacity)有参构造（负载因子0.75）
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}
//有参构造，调用HashMap(int initialCapacity, float loadFactor)有参构造，自定义容量和负载因子
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}
```

## 4.常用方法

#### 4.1 add(E e)

如果set中没有该元素则加入，如果已经存在，保持set不变，返回false。

> HashMap.put(k,v)依赖hashCode()和equals()，返回值是v。所以HashMap的key类型（即HashSet的元素类型）需要重写hashCode()和equals()。

```java
public boolean add(E e) {
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

同HashMapSet依赖HashMap一样，LinkedHashSet依赖LinkedHashMap而实现。只是value都是同一个特定值PRESENT，所有key便组成了LinkedHashSet的元素，**并能保证元素按插入有序**

> **LinkedHashSet不仅元素唯一，也是唯一一个存取顺序一致的Set（无索引）**。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/LinkedHashSet.png)

> HashSet基于HashMap实现，HashSet中的方法实际操作的，是内部的HashMap实例。如果让HashSet中的方法操作LinkedHashMap实例，就能轻松的实现LinkedHashSet。实际Java API中也是这样实现的，**通过HashSet.HashSet()方法完成狸猫换太子（LinkedHashMap换HashMap）**。

## 1.特点

实现了Set接口，内部是通过LinkedHashMap实现，这也决定了LinkedHashSet有如下特点：

- 无索引；元素唯一；有序，存取一致
- 添加、删除、判断元素是否存在，效率很高，时间复杂度为O(1)
- 可以存null

以下是LinkedHashSet的全部源码。自己没有什么方法，都是继承自HashSet。

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
// 底层Map的傀儡值v
private static final Object PRESENT = new Object();
```

## 3.构造方法

4个构造都是通过super调用父类HashSet.HashSet()方法实现的。

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

**在父类HashSet中的default HashSet()中，创建LinkedHashMap对象赋给map成员变量，完成"狸猫换太子"。**

> 之后HashSet的方法对于map的操作，实际操作的都是LinkedHashMap实例

```java
public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable {

    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
    ...
}
```

## 4.常用方法

其他常用方法，就是HashSet中的方法，只不过map指向的是LinkedHashMap实例（多态），不是HashMap实例。详见上述。


---

# IV.TreeSet


---

# V.EnumSet