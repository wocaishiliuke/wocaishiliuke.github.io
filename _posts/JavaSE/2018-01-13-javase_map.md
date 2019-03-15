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
//将链表转换为红黑树的最小数组长度要求
static final int MIN_TREEIFY_CAPACITY = 64;
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

#### 构造1

空参构造HashMap()

```java
//JDK1.8
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

//JDK1.7
public HashMap() {
    //查看下面JDK1.7的HashMap(int, float)可知，虽然是DEFAULT_INITIAL_CAPACITY，但赋给了threshold
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR); // 16, 0.75
}
```

- JDK1.8：空参构造仅对负载因子进行赋值，不对threashold初始化。threshold的赋值放在了put()中，在第一次put操作时，会进行resize，并对threshold赋值
- JDK1.7：空参构造会对loadFactor和threshold（不是容量）同时赋默认值，分别为0.75和16。

#### 构造2

HashMap(int initialCapacity, float loadFactor)

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

以下述操作为例：

```java
Map<Stirng, Object> map = new HashMap(13, 0.75);
```

- JDK1.8：初始化得到的HashMap，loadFactor=0.75，threashold=16（这里的threashold的值实际是数组长度，在第一次put()时会进行resize，对threashold重新赋值）
- JDK1.7：初始化得到的HashMap，loadFactor=0.75，threashold=13


#### 构造3

HashMap(int initialCapacity)

其实调用的还是上述的HashMap(int initialCapacity, float loadFactor)，只不过loadFactor=0.75。

```java
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

#### 构造1

HashMap(Map<? extends K, ? extends V> m)

```java
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

通过一个Map初始化HashMap，负载因子loadFactor为默认值0.75。然后调用putMapEntries()，将Map中的键值对放入HashMap。

> putMapEntries方法第二个参数控制插入后是否进行检查，清除最老元素。LinkedHashMap按访问有序的情况下才会是true，关于这个参数，详见LinkedHashMap

```java
//将Map中的键值对插入到当前HashMap中
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        //1.如果table为null，说明当前HashMap还没进行过插入操作
        if (table == null) { // pre-size
            // 1.1 根据s计算HashMap的threshold，使其能够容纳m，
            // 1.1.1 ft为能够容纳m的hash桶数组length的最小值(因为数组长度必须为2的整数幂，所以ft不一定为最终值)
            float ft = ((float)s / loadFactor) + 1.0F;
            // 1.1.2 检查ft是否超过最大值，并取整为t
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            // 返回≥t的、最小的、2的整数次幂（调用上述构造，threshold未进行赋值，为0，肯定小于t）
            if (t > threshold)
                // 该threshold还不是最终的threshold，在put()中resize时，会重新赋值为length * loadfactor
                threshold = tableSizeFor(t);
        }
        //2.如果table不为null，并且s大于当前HashMap的threshold，进行扩容（调用上述构造，table肯定是null，不会走这里）
        else if (s > threshold)
            resize();
        //3.遍历Map，调用putVal插入键值对
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

经过上述操作，假设threshold=2^n，那么：

```
由于：threshold ≥ t > (ft-1) = s/0.75  ===>  推算出：s ≤ 2^n * 0.75
```

> threshold ≥ t是tableSizeFor()保证的，t > (ft-1)是取整保证的（不考虑ft超出MAXIMUM_CAPACITY的情况）

由于在putVal插入第一个键值对时，会进行resize，这里的threshold=2^n就变成数组length，最终的threshold = length * 0.75 = 2^n * 0.75。所以在插入第一个元素resize后，s肯定小于当前HashMap最终的threshold，即可以容纳Map中所有的键值对，而且期间不需要扩容，设计还是比较巧妙的。

> 上面多次提到JDK1.8中的put()的resize，下面就讲一下resize()

#### 注意

首先，这4个构造都对loadFactor成员变量赋了值。其次：

构造1就没有更多操作了。没有为table创建Node<K,V>[]数组，即table=null。threshold和数组长度也没指定。是在第一次put操作时调用resize()，在其中创建16长度的table，指定threshold=0.75。

构造2和3，也没有为table创建Node<K,V>[]数组，即table=null。但指定了数组长度，只不过该长度暂存到threshold成员上。第一次put时，会在resize()中会再赋值给数组长度（当然会转化成稍大的2^n），即创建数组table，而threshold的值也是在resize()中指定。

构造4中有put操作，所以也是在第一次执行putVal()时调用resize()，创建table（确定数组长度）和指定threshold。


## put(K key, V value)

put操作的流程如下：

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/hashMap_put_process.jpg)

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

//对key进行hash
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //1.tab为空则创建
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //2.计算index，并对null做处理 
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        //3.节点key存在，直接覆盖value
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //4.该链为红黑树时
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //5.该链为链表时
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //链表长度大于8时，转换为红黑树（treeifyBin中还要判断数组长度）
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //key已经存在直接覆盖value
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //6.超过最大容量 就扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    //如果数组长度小于64，进行扩容，并不会转化为红黑树
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    //如果数组长度大于64，将链表转化为红黑树
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

JDK1.8的HashMap和之前版本的不同：

||实现结构|put时的链表插入|key的哈希值的计算方式|resize的实现|
|JDK1.8|数组+链表/红黑树|尾插法|优化了高位运算，通过hashCode()的高16位异或低16位实现|resize不会产生环链，避免导致cpu使用率飙升到100%，但仍有数据丢失的可能|
|之前版本|数组+链表|头插法|-|旧链表迁移的时候，如果在新表的数组索引位置相同，则链表元素会倒置，而JDK1.8会保持原来链表的顺序|

#### 确定数组索引

上面put操作中，确定索引的过程，可以分为3步：
- 1.key.hashCode()
- 2.高位运算
- 3.取模运算

```java
// 1.取key.hashCode() + 2.高位运算
(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)
// 3.取模运算，n是数组长度
i = (n - 1) & hash
```

> 其中的取模运算，在JDK1.7中是个独立的方法indexFor()，和JDK1.8的 实现原理一样。

首先通过1、2步计算hash值。如果两个key的key.hashCode()相同，那么经过上述1、2步计算得到的hash值总是相同的，即hash冲突，此时会覆盖或链入或挂树。

然后，需要将hash值对数组长度取模运算，这样元素的分布会比较均匀。但模运算的消耗较大，**为了提高效率，HashMap采用了更优雅的方式：位与运算&。**

> 在计算数组索引上，h & (n-1)比(h % n)更高效

**这种方式要求HashMap底层数组的长度n总是2的正整数幂**。这样(n - 1)的二进制前几位都是0，后几位都是1。在hash和n-1进行位与运算时，由于高位都是0，与后还是0，只有低位的1才能决定最终的结果，就相当于对hash进行了对n的求模。

> 比如3的二进制：00000011，7的二进制：00000111，15的二进制：00001111（n是int是32位，这里只展示了后八位）。
> 以数组长度n=4为例，n-1=3的二进制为00000011，那么任何hash & 00000011的结果都是0~3。即可以均匀的分布到table[0]~table[3]

另外，在第2步中，JDK1.8优化了高位运算的算法，即将h的高16位异或低16位。主要是从速度、功效、质量来考虑的，这么做可以在桶数组table的长度较小时，也能保证高低位都参与到hash的计算中，同时不会有太大的开销。

> 当数组长度n较小时，就像上面n=4时，n-1的二进制是00000011，那么只有key.hashCode()的后2位参与到位于运算。所以使用高低16位异或，可以提高key.hashCode()高地位的运算参与度。

示例：假设HashMap的数组长度n=16，某key.hashCode()如下

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/put_cal_index.png)

#### 扩容

数组是固定长度的，当然是用新大数组代替旧小数组。

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    //table中有值（非第一次put操作）
    if (oldCap > 0) {
        // 原容量超过最大值就不再扩充，扩容阈值放到最大int值，就不会再扩容了，随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 原容量没超过最大值，就扩充为原来的2倍
        // 如果扩容后容量仍小于最大值&&原容量≥16，则将扩容阈值也增大2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    //构造2和3：oldCap=0 && oldThr>0，将暂存到threshold的数组长度归还
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    //构造1和4：oldCap=0 && oldThr=0
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的扩容阈值new threshold
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    //根据容量，创建数组table
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    //扩容后数据移动
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order（保持原有顺序，跟JDK1.7不同）
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

上面4个构造中提到，它们都没有指定threshold，就是在这里指定的。另外构造2和构造3中寄存在threshold变量中的数组长度，也是在这里还给newCap，继而完成table的创建。

#### JDK1.7中的put

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```