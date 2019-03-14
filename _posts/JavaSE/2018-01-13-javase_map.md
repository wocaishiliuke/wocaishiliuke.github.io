---
title: Map
date: 2018-01-13 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Map接口的相关成员。由于Set中的成员很多依赖Map，所以这里先介绍Map体系。

<!-- more -->

##### 目录
- [X] I.Map
- [X] II.HashMap


---

# I.Map

Map接口的方法，参见上篇[Collection&Map](http://blog.wocaishiliuke.cn/javase/2018/01/11/javase_collection&map/)


---

# II.HashMap

HashMap实现了Map接口，继承了抽象类AbstractMap，可克隆，可被序列化。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/HashMap.png)

## 成员变量

> 一些重要的成员变量

```java
//用来存储Node节点的数组，JDK1.8之前叫Entry<K,V>[] table
transient Node<K,V>[] table;
//扩容的阈值（capacity * load factor），超过此值进行扩容，即当前实际能存的键值对数
int threshold;
//负载因子
final float loadFactor;
//k-v对数
transient int size;
//修改次数
transient int modCount;
```

> 一些关键值

```java
//默认初始容量，必须是2的幂
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
//最大容量。即容量必须是2的幂，而且≤2^30.
static final int MAXIMUM_CAPACITY = 1 << 30;
//默认负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//将链表转换为红黑树的阈值
static final int TREEIFY_THRESHOLD = 8;
```

## 实现原理

#### 结构

结构实现上，HashMap采用【数组+链表/红黑树】（JDK1.8新增红黑树优化），即链地址法：

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/hashmap-structure.png)

其中，数组指Node<K,V>[] table。Node是HashMap的内部类（JDK1.8前叫Entry），实现了Map.Entry接口。Node本质就是K-V键值对，额外有2个属性hash和next，分别表示key的哈希值和相邻节点引用(组织链表)。

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

#### 原理

- Hash碰撞

如上图所示，HashMap使用哈希表来存储。哈希表存在hash冲突问题，可以采用开放地址法、链地址法等方式解决。**HashMap采用了链地址法，即数组+链表结合**，数组元素是链表。当数据被Hash后，得到数组下标，把数据放在对应下标的链表上。如：

> 就是因为有hash碰撞，HashMap才使用链地址法。如果仅用数组能完美实现HashMap，就不会搞这么复杂了。

```java
map.put("a", 1);
```

首先计算key的哈希值："a".hashCode()，然后根据哈希值定位该K-V的数组存储位置。当两个key定位到相同的位置，表示发生了**Hash碰撞**。当然Hash冲突的几率跟Hash算法有关，好的Hash算法计算结果分散均匀，Hash碰撞的概率就越小，HashMap的存取效率就会越高。

当然，如果哈希桶数组table很大，即使较差的Hash算法也会比较分散，如果数组长度很小，即使好的Hash算法也会出现较多碰撞。**这就需要在空间成本和时间成本之间权衡：即根据实际情况确定数组长度，并在此基础上设计好的Hash算法减少Hash碰撞**。

那么如何减少Hash碰撞的同时，又能使得table占用较少的空间呢？答案就是**好的Hash算法和扩容机制**。

- 扩容

首先，table的初始化长度默认值是16，Load factor负载因子默认值0.75，threshold = length * Load factor是当前实际所能容纳的最大Node数，超过该threshold阈值时进行扩容resize，扩容后的HashMap容量是之前的2倍。

**默认的负载因子0.75是对空间和时间效率的一个平衡选择**，建议不要修改，除非在时间和空间比较特殊的情况下。比如如果内存空间很多而且对时间效率要求很高，可以适当降低负载Load factor，如果内存空间紧张而对时间效率要求不高，可以适当增大负载loadFactor。

- 数组长度设计

size指HashMap中实际存入的键值对数。modCount用来记录HashMap内部结构发生变化的次数，便于迭代中的安全判断（并发修改异常）。这里的变化指的是结构变化，如put()时key已存在，只是value值的覆盖，就不属于结构变化。

在HashMap中，哈希桶数组table的长度必须为2的n次方，这是一种非常规的设计。[常规的设计是把桶的大小设计为素数](https://blog.csdn.net/liuqiyao_01/article/details/14475159)，因为素数导致冲突的概率要小于合数，比如Hashtable初始化桶大小就是11（但它扩容后不能保证还是素数）。HashMap采用这种非常规设计，主要是为了在取模和扩容时做优化，同时为了减少冲突，HashMap定位哈希桶索引位置时，也加入了高位参与运算的过程。

- 红黑树优化

即使负载因子和Hash算法设计的再合理，也可能出现拉链过长的情况。一旦出现拉链过长，会严重影响HashMap的性能。**在Java 8中，引入了红黑树进行优化：当链表长度过长（默认超过8并且数组长度大于64）时，就将链表转换为红黑树，利用红黑树增删改查快速的特点提高HashMap的性能**。


## 构造方法

#### HashMap()

空参构造

```java
//JDK1.8
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

//JDK1.7
public HashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR); // 16, 0.75
}
```

JDK1.8中的空参构造仅对负载因子进行了赋值，没有对threashold初始化。threshold的赋值放在了put()中，在第一次put操作时，会进行resize，并对threshold赋值。而JDK1.7中的空参构造会对loadFactor和threshold同时赋默认值。

#### HashMap(int initialCapacity, float loadFactor)

> 其中tableSizeFor()是JDK1.8的新方法，用移位实现，效率较高。

```java
//JDK1.8
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
//返回≥cap的、最小的、2的整数次幂
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

//JDK1.7
public HashMap(int initialCapacity, float loadFactor) {
    this.loadFactor = loadFactor;
    threshold = initialCapacity;
}
```



- 其他构造

```java
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```