---
title: Map
date: 2018-01-13 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Map接口的相关成员。由于Set中的成员很多依赖对应的Map，所以这里先介绍Map体系。

<!-- more -->

##### 目录
- [X] I.Map
- [X] II.HashMap
- [X] III.LinkedHashMap
- [X] IV.TreeMap
- [X] V.EnumMap
- [X] VI.参考


---
# I.Map

Map接口的方法，参见上篇[Collection&Map](http://blog.wocaishiliuke.cn/javase/2018/01/11/javase_collection&map/)


---
# II.HashMap

HashMap实现了Map接口，继承了抽象类AbstractMap，可克隆，可被序列化。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/HashMap.png)

## 1.成员变量

> 一些重要的成员变量

分清哪些是成员变量，哪些是线程内的方法局部变量，便于理解HashMap的线程安全问题。

```java
//用来存储Node节点的数组，JDK1.8之前叫Entry<K,V>[] table
transient Node<K,V>[] table;
//扩容的阈值（capacity * load factor），超过此值进行扩容，即当前实际能存的键值对数
int threshold;
//负载因子
final float loadFactor;
//存入的k-v对数
transient int size;
//修改次数
transient int modCount;
```

modCount用来记录HashMap内部结构发生变化的次数，便于迭代中的安全判断（并发修改异常）。这里的变化指的是结构变化。如果put()时key已存在，只是value值的覆盖，就不属于结构变化。

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

## 2.实现原理

讲解具体方法前，先大概阐述下HashMap的实现原理，便于理解。

#### 2.1 结构

结构实现上，HashMap采用【数组+链表/红黑树】（JDK1.8新增红黑树优化），即链地址法：

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/hashmap-structure.png)

其中，数组指Node<K,V>[] table。Node是HashMap的内部类（JDK1.8前叫Entry），实现了Map.Entry接口。Node本质就是K-V键值对，额外还有2个属性hash和next，分别表示key的哈希值和下一节点引用（链表）。

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

#### 2.2 原理

- Hash碰撞

如上图所示，HashMap使用哈希表来存储。哈希表存在hash冲突问题，可以采用开放地址法、链地址法等方式解决。**HashMap采用了链地址法，即数组+链表结合**。当数据被Hash后，得到数组下标，把数据放在对应下标的链表上。如：

```java
map.put("a", 1);
```

首先计算key的哈希值："a".hashCode()，然后根据哈希值定位该键值对在数组中的存储位置index。

当两个key定位到相同的位置，就表示发生了**Hash碰撞**。当然Hash冲突的几率跟Hash算法有关，好的Hash算法计算结果分散均匀，Hash碰撞的概率也就越小，HashMap的存取效率就会越高。

当然，如果哈希数组table很大，即使较差的Hash算法也会比较分散（mod取模），如果数组长度很小，即使好的Hash算法也会出现较多碰撞。**这就需要在空间成本和时间成本之间权衡：即根据实际情况确定数组长度，并在此基础上设计好的Hash算法减少Hash碰撞**。

> 就是因为有hash碰撞，HashMap才使用链地址法。若仅用数组就能完美实现HashMap，就不会搞这么复杂。

那么如何在减少Hash碰撞的同时，又能使得table占用较少的空间呢？答案就是**好的Hash算法和扩容机制**。

- 扩容

首先，table的初始化长度默认值是16，Load factor负载因子默认值0.75，threshold = length * Load factor是当前实际所能容纳的最大Node数，超过该threshold阈值时进行扩容resize，扩容后的HashMap容量是之前的2倍。当然，这些成员变量可以调用构造时自定义，覆盖默认配置。

**默认的负载因子0.75是对空间和时间效率的一个平衡选择**，建议不要修改，除非在时间和空间比较特殊的情况下。比如如果内存空间很多而且对时间效率要求很高，可以适当降低负载Load factor，如果内存空间紧张而对时间效率要求不高，可以适当增大负载loadFactor。另外loadFactor可以大于1。

- 数组长度设计

在HashMap中，**哈希数组table的长度必须为2的n次方**，这是一种非常规的设计。[常规的设计是把桶的大小设计为素数](https://blog.csdn.net/liuqiyao_01/article/details/14475159)，因为素数导致冲突的概率要小于合数，比如Hashtable的初始化桶大小就是11（但它扩容后不能保证还是素数）。**HashMap之所以采用这种非常规设计，主要是为了在取模和扩容时做优化。**

- 高位运算

为了减少冲突，HashMap定位哈希数组索引位置时，也加入了**高位参与运算**的过程，防止低位相同高位不同的hash值的冲突。

- 红黑树优化

即使负载因子和Hash算法设计的再合理，也可能出现拉链过长的情况，此时会严重影响HashMap的性能。在JDK1.8中，引入了红黑树进行优化：**当链表长度过长（默认超过8并且数组长度大于64）时，就将链表转换为红黑树，利用红黑树增删改查快速的特点提高HashMap的性能**。


## 3.构造方法

#### 3.1 构造1

空参构造HashMap()

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

- JDK1.8：空参构造仅对负载因子赋值0.75，不对threashold初始化。threshold的赋值放在了resize()中，在第一次put操作时，会进行resize，并指定threshold赋值
- JDK1.7：空参构造会对负载因子赋值0.75，会对threashold初始化，但只是将DEFAULT_INITIAL_CAPACITY暂存到threashold成员变量上，并不是真正的扩容阈值。也在第一次put时，在inflateTable()中将threashold的值还给数组长度，并指定真正的threashold阈值。

#### 3.2 构造2

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
- JDK1.7：初始化得到的HashMap，loadFactor=0.75，threashold=13（这里也只是指定的数组长度，在第一次put()时会进行inflateTable()，对threashold重新赋值，并对数组长度13处理到2^n，即16）

> 在这两个版本的JDK中，该构造都对threashold进行了赋值，但都不是真正的threashold阈值，而是第一次put中要使用的数组长度值，真正的阈值也都是在第一次put中指定的。另外，1.8中会提前格式化数组长度（2^n），而1.7中是直接赋值，在put操作的inflateTable()中使用roundUpToPowerOf2()完成长度格式化。

#### 3.3 构造3

HashMap(int initialCapacity)

其实调用的还是上述的HashMap(int initialCapacity, float loadFactor)，只不过loadFactor=0.75。

```java
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

#### 3.4 构造4

HashMap(Map<? extends K, ? extends V> m)

```java
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;//0.75
    putMapEntries(m, false);
}
```

通过一个Map初始化HashMap，负载因子为默认值0.75。然后调用putMapEntries()，将Map中的键值对放入HashMap。

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

由于在putVal插入第一个键值对时，会进行resize，这里的threshold=2^n就变成数组length，最终的threshold = length * 0.75 = 2^n * 0.75。所以在插入第一个元素resize后，s肯定小于当前HashMap最终的threshold，即可以容纳下Map的所有键值对，期间不需要扩容，设计比较巧妙。

#### 3.5 注意

首先，这4个构造都对负载因子loadFactor赋了值。其次：

构造1就没有更多操作了。没有为table创建Node<K,V>[]数组，**即HashMap创建初始化后table=null**。threshold和数组长度也没指定。是在第一次put操作时调用resize()，在其中创建16长度的table，指定threshold=0.75。

构造2和3，也没有为table创建Node<K,V>[]数组，即table=null。但指定了数组长度，只不过该长度暂存到threshold成员上。第一次put时，会在resize()中会再赋值给数组长度（当然会转化成稍大的2^n），即创建数组table，而threshold的值也是在resize()中指定。

构造4中有put操作，所以也是在第一次执行putVal()时调用resize()，创建table（确定数组长度）和指定threshold。


## 4.put(K key, V value)

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
                    //尾插法
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

#### 4.1 确定数组索引

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

> 其中的取模运算，在JDK1.7中是个独立的方法indexFor()，是和JDK1.8的实现方式（位于运算）一样。

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

#### 4.2 扩容

数组是固定长度的，扩容当然是用新大数组代替旧小数组。

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

#### 4.3 JDK1.7的put

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    //如果key已存在，覆盖value
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
    //没有重复的key，添加新Entry
    addEntry(hash, key, value, i);
    return null;
}

//添加新Entry
void addEntry(int hash, K key, V value, int bucketIndex) {
    //这里扩容的条件有两个：size >= threshold && 该键值对对应的index不为null
    if ((size >= threshold) && (null != table[bucketIndex])) {
        //扩容
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    createEntry(hash, key, value, bucketIndex);
}

//扩容
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    //如果原桶数组已达2^30，将阈值修改为最大值，以后就不会扩容了
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }
    //创建新数组，数组转移，修改阈值
    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}

//数据转移到新数组
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {//遍历旧数组
        while(null != e) {      //遍历链表
            //1.记录e的下一个元素（单链表转移头指针）
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            //2.rehash计算在新数组中的索引（1.8做了优化）
            int i = indexFor(e.hash, newCapacity);
            //3.将e插入对应新数组index上的链表头
            e.next = newTable[i];//头插法(尾插还要遍历，效率低)
            newTable[i] = e;
            //4.遍历下一个元素
            e = next;
        }
    }
}
```

在JDK1.7的resize()中的数据转移transfer()时，会遍历原数组，rehash后，组织到新数组中：采用单链表的头插法，即链表会顺序颠倒（1.8则保持转移后顺序不变）。

示例：假设哈希数组table长度为2，负载因子loadFactor=1，阈值=2*1=2，依次将key=3、7、5执行put:

- 1.第一次插入执行inflateTable()，确定table长度为2，threshold=2，然后插入key=3到table[1]
- 2.插入key=7，到table[1]（此时7在链头，即7--->3）
- 3.插入key=5，执行put()
    - 3.1 执行addEntry()，满足(size >= threshold) && (null != table[1])，执行resize()
    - 3.2 创建新数组，执行transfer()
    - 3.3 遍历oldTable，只有oldTable[1]不为null，开始遍历链表
    - 3.4 先移动链头的key=7，rehash得到在newTable中的索引是3，所以将7移到newTable[3]
    - 3.5 移动key=3，rehash后的新索引也是3，头插法，即newTable[3] = 3--->7
    - 3.6 插入key=5

上述Java7中的扩容和数据迁移操作图示如下：

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/hashMap7_resize.png)

#### 4.4 JDK1.8中优化

JDK1.8的HashMap和之前版本的不同（优化）：

|  |实现结构|key哈希值的处理|put时的链表插入|resize时的数据迁移|resize的实现|
|:-|:------|:-----------------|:-------------|:---------------|:----------|
|JDK1.8|数组+链表/红黑树|优化了高位运算，通过hashCode()的高16位异或低16位实现|尾插法|bit位判断（代替rehash）+平移|resize不会产生环链，避免导致cpu使用率飙升到100%，但仍有数据丢失的可能|
|之前版本|数组+链表|-|头插法|遍历+rehash+头插|旧链表迁移的时候，如果在新表的数组索引位置相同，则链表元素会倒置，而JDK1.8会保持原来链表的顺序|

- 红黑树优化
- 确定索引时，高位参与运算
- 扩容数据迁移时，使用bit位判断代替rehash，使用平移（相当于遍历+尾插）代替会倒序的头插法（不会环链）

JDK1.8中，**数据迁移时不需要rehash，而是通过直接判断对应高位（n-1对应的高位），确定在新数组中的index（原位置或原位置+cap），效率较高**。

上图中，我们使用2次幂扩展（长度×2）后，**元素要么是在原位置，要么在原位置移动2次幂个位置上**（如上的1和3）。以16扩到32为例：

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/hashMap8_indexFor.png)

元素保持index位置不变，还是移动2次幂，关键看：和(n-1)新增位（上图中的红色位）对应的hash值中的bit位，是0还是1。于是在JDK1.8中做了优化：检查原hash值中新增参与位与运算的那个bit位，是1还是0。是0索引不变，是1索引变成"原索引+oldCap"。

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/hashMap8_resize.png)

JDK1.8的这个设计非常巧妙，节省重新计算hash值的时间。同时，新增参与位与运算的bit位是0还是1，可以认为是随机的，因此resize的过程，也会把之前的冲突节点分散到新的bucket上。

另外，JDK1.7中，在旧链表迁移时，如果在新表的数组索引位置没有变化，则链表元素会倒置。但JDK1.8中不会倒置。

#### 4.5 线程安全

HashMap是线程不安全的，推荐使用线程安全的ConcurrentHashMap。JDK1.8之前，HashMap在多线程时可能造成死循环，导致cpu使用率飙升到100%，并且有可能会丢失数据。**JDk1.8之后，不会再造成死循环，但无法规避数据丢失的可能**。

之所以在put()中讲述HashMap的线程安全问题，是因为**环链（JDK1.7）和数据丢失就发生在put时的扩容、数据转移过程中**。

先来看一下**JDK1.7**中的HashMap，在**多线程**时为什么会造成死循环。

示例：map初始化长度为2，loadFactor=1，threshold=2*1=2。即put第3个key时，map会resize。

```java
public class HashMapInfiniteLoop {
    private static HashMap<Integer,String> map = new HashMap<>(2,1f);
    public static void main(String[] args) {
        map.put(1, "A");
        map.put(5, "B");//这里设置的初始元素，需保证在rehash后仍会碰撞
        new Thread("Thread1") {
            public void run() {
                map.put(7, "C");
            };
        }.start();
        new Thread("Thread2") {
            public void run() {
                map.put(11, "D");
            };
        }.start();
    }
}
```

为方便理解，这里再次贴上transfer()的逻辑

```java
//JDK1.7
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {//遍历旧数组
        while(null != e) {      //遍历链表
            //1.记录e的下一个元素（单链表转移头指针）
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            //2.rehash计算在新数组中的索引（1.8做了优化）
            int i = indexFor(e.hash, newCapacity);
            //3.将e插入对应新数组index上的链表头
            e.next = newTable[i];//头插法(尾插还要遍历，效率低)
            newTable[i] = e;
            //4.遍历下一个元素
            e = next;
        }
    }
}
```

使用IDEA的多线程断点调试：

- 1.让两个线程都执行到put-->addEntry-->resize-->transfer()的第一行

此时，由于table是共享的成员变量，Thread1和Thread2线程又各自创建了newTable

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/java7_hashMap_Infinite_loop0.png)

- 2.让Thread1执行到transfer的Entry<K,V> next = e.next

此时e指向节点5-B，next指向节点1-A（一定要让next有值，不能为null）

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/java7_hashMap_Infinite_loop1.png)

- 3.让Thread2执行完

Thread2执行完put，会使用它的newTable替换旧table

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/java7_hashMap_Infinite_loop2.png)

此时Thread1的局部变量：e仍指向5-B，next仍指向1-A

- 4.接着执行Thread1

本次循环使得Thread1的newTable[1]指向5-B，5-B的next=null

```java
int i = indexFor(e.hash, newCapacity) = 1
e.next = newTable[1]= null
newTable[1] = e = 5-B
e = next = 1-A
```

> 由于Thread2执行完后，使得1-A的next指向了5-B，所以此时的局部变量e和next的值如下

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/java7_hashMap_Infinite_loop3.png)

- 5.接着执行Thread1的循环

本次循环使得：Thread1的newTable[1]指向1-A，并将1-A的next指向5-B（原本也是指向的）。另外局部变量e和next变化如下

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/java7_hashMap_Infinite_loop4.png)

- 6.接着执行Thread1最后一个循环，此时局部变量：e指向5-B，next=null

本次循环使得：Thread1的newTable[1]指回5-B，并将5-B的next指向之前的链头1-A，**环链形成**。此时局部变量e=null，循环结束

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/java7_hashMap_Infinite_loop5.png)

- 7.最后put节点7-C，替换table，Thread1执行完成

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/java7_hashMap_Infinite_loop6.png)

**可以看到形成了环链，并且数据丢失（11-D）。之后如果遍历到环链这里，会一直在这里遍历next，导致死循环。**

> 实际在IDEA调试时，由于debug窗口中的变量，也会查询map中的元素，所以在产生环链后的操作很卡顿，使用top查看CPU爆表，控制台报错如下

```java
Exception in thread "Thread2" java.lang.OutOfMemoryError: Java heap space
```

JDK1.8中的HashMap在扩容时，不是使用JDK1.7的这种重组链表的方式，而是通过维护两个链表指针loHead、loTail、hiHead、hiTail，只是将元素添加到链表的尾部，并不需要头部链接（**平移代替头插法**），所以即使是多线程场景，也不会产生环链。但是由于table是成员变量，由Thread1和Thread2共享，所以在Thread1调度回来重组链表的时候，仍会覆盖Thread2写的数据，即HashMap是线程不安全的。

**总结：多线程下，环链（JDK1.7）是因为rehash后的头插法；数据丢失是因为table是成员变量。**


## 5.其他常用方法

#### 5.1 get(Object key)

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //获取key所在数组索引上的第一个元素first
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && //比较第一个元素first与key是否相等
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //第一个元素不相等，遍历first的后继元素
        if ((e = first.next) != null) {
            //如果是红黑树，则取遍历红黑树，获取与key相等的TreeNode节点
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                //遍历链表，获取与key相等的Node节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

#### 5.2 containsKey(Object key)

map中是否包含key。getNode如上。

```java
public boolean containsKey(Object key) {
    return getNode(hash(key), key) != null;
}
```

#### 5.3 containsValue(Object value)

双重遍历，效率较低

```java
public boolean containsValue(Object value) {
    Node<K,V>[] tab; V v;
    if ((tab = table) != null && size > 0) {
        //遍历数组
        for (int i = 0; i < tab.length; ++i) {
            //遍历数组各个index上的Node
            for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                if ((v = e.value) == value ||
                    (value != null && value.equals(v)))
                    return true;
            }
        }
    }
    return false;
}
```

#### 5.4 remove(Object key)

其中，查询Node的方式跟上述getNode()一致。

```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

/**
* 删除key对应的节点，并返回该Node
* @param hash key的哈希值
* @param key key
* @param value 如果matchValue为true，则比较value，否则忽略value
* @param matchValue 如果为true，只有当value也相等时，才删除节点
* @param movable 如果为false，在删除时，不移动其他Node，作用于红黑树节点删除
* @return 返回删除的Node，如果未删除，返回null
*/
final Node<K,V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        //与getNode(int hash, Object key)一致，获取key对应的Node
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        //判断是否能删除Node
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                //红黑树节点删除
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                //node为链表首节点时，将tab[index]指向node的后继节点
                tab[index] = node.next;
            else
                //node非链表首节点时，删除node节点
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

#### 5.5 remove(Object key, Object value)

```java
public boolean remove(Object key, Object value) {
    return removeNode(hash(key), key, value, true, true) != null;
}
```

## 6.遍历

```java
HashMap<String,Object> map = new HashMap<>(8);
map.put("1", "A");
map.put("2", "B");
```

#### 6.1 通过键集合

可以使用Iterator和foreach

```java
Set<String> keys = map.keySet();
Iterator<String> it = keys.iterator();
while(it.hasNext()){
    String key = it.next();
    System.out.println(key + " = " + map.get(key));
}
```

```java
for (String key : map.keySet()) {
    System.out.println(key + " = " + map.get(key));
}
```

#### 6.2 通过键值对集合

也可以使用Iterator和foreach两种形式，推荐语法糖更优雅

```java
Set<Map.Entry<String, Object>> entries = map.entrySet();
Iterator<Map.Entry<String, Object>> it = entries.iterator();
while (it.hasNext()) {
    Map.Entry<String, Object> entry = it.next();
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
```

```java
for (Map.Entry<String,Object> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
```

## 7.键唯一

HashMap的键唯一是通过如下比较实现的：

```java
if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
```

上述比较依赖两个方法：**先比较hash值，hash值相同时再比较equals**。

> **hash值不同，key肯定不同，但hash值相同，key也不一定是相同，还需要具体比较equals**。

hash值比较的是两个int值，效率比两个对象直接equals更高。所以先使用hash值过滤大部分情况，降低equals()的使用次数。基本数据类型的包装类、String等都重写了这两个方法。

#### 7.1 hashCode()

> 如《确定数组索引》小节所述，通过(h = key.hashCode()) ^ (h >>> 16)计算hash值，再和n-1进行位于取模。如果key.hashCode()相同，那么高位运算得到的hash值也总是相同的，即hash冲突。**所以key类中需要重写hashCode()，好的hash算法首先会规避掉很多碰撞**。

重写hashCode()的原则：
- 1.属相相同的对象，给出的哈希值必须相同
- 2.属相不同的对象，给出的哈希值尽量不同


#### 7.2 equals()

hash碰撞，并不一定表示key相同。所以在putVal()中又调用了key.equals()。**在hash值相同的时候，进一步判断key是否真的相同。所以key类中需要重写equals()**。

重写equals()的原则：
- 1.属相相同的对象，返回true
- 2.属相不同的对象，返回false

#### 7.3 Object中的hashCode和equals

- hashCode()返回对象地址的int值
- equals()其实就是==，比较地址值

```java
public class Object {
    //在native层生成的，非Java代码实现
    public native int hashCode();

    public boolean equals(Object obj) {
        return (this == obj);
    }
    ...
}
```

> A native method is a Java method whose implementation is provided by non-java code.

#### 7.4 示例

这里使用编辑器提供的模板重写

```java
public class User {
    private String name;
    private int age;

    //setter and getter

    @Override
    public String toString() { ... }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return age == user.age && Objects.equals(name, user.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

这里使用了Objects（since 1.7）类的hash方法

```java
//Objects（since 1.7）的hash()
public static int hash(Object... values) {
    return Arrays.hashCode(values);
}

//Arrays的hashCode()
public static int hashCode(Object a[]) {
    if (a == null) return 0;
    int result = 1;
    for (Object element : a)
        //31×1+地址值
        result = 31 * result + (element == null ? 0 : element.hashCode());
    return result;
}
```

**为什么是31？**

> 除了Arrays的hashCode()外，String的hashCode()也是使用了31

- a.**31是不大不小的质子数**

素数不容易冲突，而太小的素数也不合适，范围太小，太大可能超出int的最大范围。

> 国外有人做的测试：对超过50,000个英文单词进行hashCode，使用常数31, 37, 39, 41作为乘子，每个常数算出的哈希值冲突数都小于7个，那么这几个数就被作为生成hashCode的备选乘数了。

- b.**31可以被JVM优化**
　
31 * i = (i << 5) - i，JVM可以使用位运算更高效的计算。

**另外，HashMap的key可以为null（单个，多个会覆盖）**

```java
Map<String, Object> map = new HashMap<>();
map.put(null,1);
map.put(null,2);
System.out.println(map);
```


---
# III.LinkedHashMap

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/LinkedHashMap.png)

LinkedHashMap是HashMap的子类。继承了HashMap的所有特性，另外，**LinkedHashMap能保证元素顺序一致**

> LinkedHashMap中存在2个链表：
> - 单向链表：next指针，维护Map节点
> - 双向链表：before和after指针，维护节点的遍历顺序

## 1.顺序一致

和HashMap相比，LinkedHashMap增强了保持顺序的特点。为此，LinkedHashMap做了以下增强：

- 1.节点Entry增强为双向（双向链表，前驱、后继）
- 2.具体实现了HashMap中维持顺序的钩子方法

> 下面几个钩子方法在HashMap中没有具体实现，而在LinkedHashMap中重写了这些维持顺序的相关操作。

```java
// Callbacks to allow LinkedHashMap post-actions
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
void afterNodeRemoval(Node<K,V> p) { }
```

**LinkedHashMap支持2种顺序一致：存取顺序（insertion-order）、访问顺序（access-order）**。

#### 1.1 存取顺序

先插入的在前，后插入的在后，修改操作不改变顺序。

```java
Map<String,Object> map = new LinkedHashMap();
map.put("1",1);
map.put("2",2);
map.put("3",3);
map.put("4",4);
map.put("2",22);
map.remove("3");
for (Map.Entry<String,Object> entry: map.entrySet()) {
    System.out.println(entry.getKey() + "--->" + entry.getValue());
}

//输出
1--->1
2--->22
4--->4
```

使用场景：

LinkedHashMap默认就是存取顺序一致（空参构造），常用来处理需要存取一致的数据。比如从DB查询order by的数据，可以在内存中使用LinkedHashMap接收。

#### 访问顺序

通过设置accessOrder成员变量来开启。

```java
Map<String,Object> map = new LinkedHashMap(16, 0.75f, true);
map.put("1",1);
map.put("2",2);
map.put("3",3);
map.put("4",4);
//被访问的元素，会被移动到内部双向链表末尾
map.get("2");       //1 -> 3 -> 4 -> 2
map.put("1", "11"); //3 -> 4 -> 2 -> 1
map.put("5", "5");  //3 -> 4 -> 2 -> 1 -> 5

for (Map.Entry<String,Object> entry: map.entrySet()) {
    System.out.println(entry.getKey() + "--->" + entry.getValue());
}

//输出
3--->3
4--->4
2--->2
1--->11
5--->5
```

使用场景：通过LinkedHashMap的访问有序，实现LRU(Least Recently Used，最近最少使用)算法，如缓存。

> 凡是位于速度相差较大的两种实体之间，用于协调两者数据传输速度差异的结构，均可称之为Cache**。主存容量远大于CPU cache，磁盘容量远大于磁盘缓存。任何缓存都面临一个同样的问题：当缓存空间用完后，如何挑选要舍弃的内容，为新缓存的内容腾出空间。常见的解决算法有：最久未使用算法（LFU）、先进先出算法（FIFO）、最近最少使用算法（LRU）、非最近使用算法（NMRU）等。这些算法在不同层次的缓存上执行时拥有不同的效率和代价，需根据具体场合选择。本文讲的LinkedHashMap的访问有序，就可以很容易的实现LRU算法。

```java
public class LRUCache<K,V> extends LinkedHashMap<K,V> {

    private int maxSize;

    public LRUCache(int maxSize) {
        super(16,0.75f, true);  //开启访问顺序
        this.maxSize = maxSize;
    }

    //重写LinkedHashMap中的removeEldestEntry()
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return this.size() > maxSize; //超出size时，开启清除"最老节点"（该Map永远不会扩容）
    }
}
```

```java
// 测试
public void test() {
    LRUCache<String,Object> cache = new LRUCache(3);
    cache.put("1",1);
    cache.put("2",2);
    cache.put("3",3);

    //被访问的元素，会被移动到内部双向链表末尾
    cache.get("2");       //1 -> 3 -> 2
    cache.put("4", 4);    //[1] -> 3 -> 2 -> 4，1会被删除

    for (Map.Entry<String,Object> entry: cache.entrySet()) {
        System.out.println(entry.getKey() + "--->" + entry.getValue());
    }
}
// 结果
3--->3
2--->2
4--->4
```

其中removeEldestEntry是LinkedHashMap特有的方法。该方法用于插入键值对后，在afterNodeInsertion()中被调用。如果返回true，则会清除"最老"(最久未访问)的键值对。LinkedHashMap中该方法总是返回false，即插入后永远不会清除元素。所以要实现LRU，需要重写此方法。

```java
// LinkedHashMap
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

## 构造

构造都是调用HashMap的对应构造，通过accessOrder切换存取顺序、访问顺序。

```java
public LinkedHashMap() {
    super();
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity) {
    super(initialCapacity);
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;
}

// 该构造是唯一一个能够指定使用access-order的构造函数
public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}

//putMapEntries()的第二个参数控制着afterNodeInsertion()后是否进行检查，来清除最老元素
//这里遵循insertion-order，所以插入后永不删除，直接赋值false
public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);
}
```

## 成员变量

其他均继承自HashMap。

```java
//双向节点（键值对），继承自HashMap.Node
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}

// The head (eldest) of the doubly linked list
transient LinkedHashMap.Entry<K,V> head;

//The tail (youngest) of the doubly linked list
transient LinkedHashMap.Entry<K,V> tail;

//The iteration order：true for access-order,false for insertion-order
final boolean accessOrder;
```

相比于HashMap的Node，LinkedHashMap的Entry除了有next指针(继承自HashMap.Node)外，还有指向前驱、后继的before、after指针。**LinkedHashMap是通过【数组+链表/红黑树+双向链表】实现的**。

示例：某LinkedHashMap数组table的size=4，loadFactor=1，依次put四个元素（5、1、9、3为key）。假设index是简单的通过key.hashCode()取模mod数组长度来确定，则LinkedHashMap的内部结构大致为：

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/LinkedHashMap1.jpg)

LinkedHashMap = HashMap + before&after双向链。当LinkedHashMap改变时，双向链表结构也会改变，保证head始终指向“最老”节点，tail始终指向“最新”节点。当然最老和最新节点的确定跟保持顺序的2种方式有关。

## 常用方法

#### put()

LinkedHashMap的put()和putAll()完全继承自HashMap。那么

- 问题1：如何实现插入的是LinkedHashMap.Entry而非HashMap.Node
- 问题2：如何实现双向链表排序

> put()和putAll()主要通过putVal()来实现，所以这里分析putVal()的逻辑即可。

问题1：在HashMap的putVal()中，通过调用newNode()，构建一个待插入的Node对象，而LinkedHashMap重写了newNode()，所以这里会调用自己的newNode()，构建一个待插入的LinkedHashMap.Entry对象。（实际就是继承+方法重写）

问题2：在LinkedHashMap.newNode()中，调用了linkNodeLast()来维持双向链表。即新节点在创建后、插入前，会链到双向链表的尾部。源码如下：

> put()、putVal()可参考上述HashMap，在其中调用了newNode()

```java
//LinkedHashMap.newNode()
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    //创建Entry节点
    LinkedHashMap.Entry<K,V> p = new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    //维护双向链表结构，将新节点p插到双向链表尾部
    linkNodeLast(p);
    return p;
}

private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    //tail指针移到p节点
    tail = p;
    //如果插入p节点之前链表为null，则将head指针也指向新插入的节点p
    if (last == null)
        head = p;
    else {
        //将新插入的节点p插入到双向链表尾部
        p.before = last;
        last.after = p;
    }
}
```

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/LinkedHashMap_put_insert.jpg)

在putVal()中，调用了上述提到的2个钩子方法：afterNodeAccess()、afterNodeInsertion()。在HashMap中它们是空方法，在LinkedHashMap有具体的实现：

- afterNodeAccess：当按访问顺序时，维护双向链表结构，将最新访问的节点移到双向链表的尾部
- afterNodeInsertion：插入节点后，检查是否需要清除最老节点（LRU）。默认永不删除（因为removeEldestEntry返回false），如果需要清除最老节点，可以参考上述LRU示例：LRUCache

> afterNodeInsertion是在插入节点后，检查是否需要清除最老节点。linkNodeLast才是插入前，维护双向链表。

##### afterNodeAccess

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    //开启了access-order && e不是双向链表的last，则调整双向链表
    if (accessOrder && (last = tail) != e) {
        //p：刚被访问的节点，a：后继节点，b：前驱节点
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/LinkedHashMap_put_access.jpg)

除了HashMap.put()中调用外，afterNodeAccess()在replace()、merge()、compute()中也有调用。保证access-order时，最后被访问的节点会被移到双向链表尾部。（insertion-order时，该方法不做任何操作）

##### afterNodeInsertion

插入后，是否删除head节点（双向链表的头结点）

> 一般put时，前两个都为true，大部分时候只取决于removeEldestEntry()，所以上述实现LRU时需要重写removeEldestEntry()。但构造LinkedHashMap(Map m)在插入时，evict为false，即构造初始化插入时不删除。

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    //evit为ture && 双向链表不为null && removeEldestEntry()返回true时，清除最老节点
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

> removeEldestEntry只在这里被使用，即插入后判断是否删除最老元素。

#### remove(Object key)

LinkedHashMap也没有重写remove()，完全使用继承的HashMap.remove()。和put操作一样，remove操作在单向链表中删除节点后，也是通过钩子方法，完成在双向链表中该节点的删除。

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/LinkedHashMap_remove.jpg)

> HashMap的remove()和removeNode()，详见上述，这里不再赘述。

```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

final Node<K,V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        //与getNode(int hash, Object key)一致，获取key对应的Node
        ...
        //判断是否能删除Node
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            //红黑树/链表节点删除
            ...
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

> LinkedHashMap重写的afterNodeRemoval()，删除双向链表中的该节点

```java
//LinkedHashMap删除节点后，将节点也从双向链表中移除
void afterNodeRemoval(Node<K,V> e) {
    //删除的节点p，b是p的前驱节点，a是p的后继节点
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    //清空p的前驱和后继
    p.before = p.after = null;
    //处理p的前驱
    if (b == null)
        head = a;
    else
        b.after = a;
    //处理p的后驱
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

#### get(Object key)

LinkedHashMap重写了HashMap.get()。只是增加了一个判断：如果LinkedHashMap是按访问有序，则在get访问后调用afterNodeAccess()，调整双向链表结构。

```java
public V get(Object key) {
    Node<K,V> e;
    //调用HashMap.getNode()，获取key对应的Node
    if ((e = getNode(hash(key), key)) == null)
        return null;
    //维护双向链表：如果LinkedHashMap按访问有序，则将刚访问的节点e，移到双向链表的尾部
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}
```

同样，getOrDefault()的实现是一样的：

```java
public V getOrDefault(Object key, V defaultValue) {
   Node<K,V> e;
   if ((e = getNode(hash(key), key)) == null)
       return defaultValue;
   if (accessOrder)
       afterNodeAccess(e);
   return e.value;
}
```

#### containsKey(Object key)

LinkedHashMap没有重写HashMap.containsKey()，完全复用。因为判断key是否存在，不涉及双向链表的维护，且HashMap.containsKey()的时间复杂度为O(1)，效率较高，没必要重写。

#### containsValue(Object value)

虽然containsValue()不需要维护双向链表，LinkedHashMap还是重写了containsValue()，是因为HashMap.containsValue()效率太低。LinkedHashMap内部额外维护了一个双向链表，直接遍历双向链表即可访问HashMap的所有节点，依次判断即可，时间复杂度为O(n)。

> HashMap.containsValue()会遍历hash数组+遍历链表/红黑树，使用两层for循环来比较value，时间复杂度O(n)-O(n^2)，详见上述。

```java
public boolean containsValue(Object value) {
    //遍历双向链表，判断value是否存在
    for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
        V v = e.value;
        if (v == value || (value != null && value.equals(v)))
            return true;
    }
    return false;
}
```

#### 总结

**对比HashMap可以发现，LinkedHashMap其实就是比HashMap多了access-order，即双向链表和维护双向链表的逻辑。双向链表的维护是通过【linkNodeLast()+钩子方法实现的】**。

put()时：首先在newNode()时，就通过linkNodeLast()（非钩子方法，LinkedHashMap特有）将新Entry维护到双向链表的表尾。然后通过钩子方法afterNodeAccess()，完成在访问顺序时的双向链表的维护（不管是访问有序还是插入有序，此时都要维护），以及通过钩子方法afterNodeInsertion()，判断是否删除最老节点。

remove()时：在单向链表中删除节点后，通过钩子方法afterNodeRemoval()，在双向链表中也删除该节点。

get()时：如果是按访问有序，通过afterNodeAccess()来调整双向链表结构，否则按插入有序则不变。


---
# IV.TreeMap

TreeMap是基于[红黑树](https://blog.wocaishiliuke.cn/datastructure/2018/12/03/Binarytree01/)的有序Map实现。

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/DataStructure/treemap.png)

> 其中SortedMap、NavigableMap两个接口，主要是根据TreeMap有序的特点，获取一定范围内的键值对集合。

- TreeMap可以保证内部key的顺序，可以很方便的获取第一个、最后一个、或某区间内的Entry/key
- 基于红黑树，根据键查找、删除、插入效率较高，时间复杂度为O(log2_T)

## 1.有序

有序是TreeMap的显著特点。TreeMap依赖比较器comparator或key实现Comparable来实现比较排序：
- 自然排序（natural ordering）：Comparable.compareTo()
- 比较器排序：comparator.compare()

```java
final int compare(Object k1, Object k2) {
    return comparator==null ? ((Comparable<? super K>)k1).compareTo((K)k2)
        : comparator.compare((K)k1, (K)k2);
}
```

**在TreeMap中，比较器comparator不为null时，使用比较器排序；否则使用key的自然排序**。

#### 1.1 Comparable

可比较的。该接口只有一个方法：compareTo()，该方法也是该接口实现类的natural comparison method。

null不是任何类的实例。所以e.compareTo(null)会抛NullPointerException

```java
public interface Comparable<T> {
    /**
     * e.compareTo(o)：小于返回-1，等于返回0，大于返回1
     * @throws o=null时会抛NullPointerException
     * @throws 如果o类不能和e比较（一般是不同类型），抛ClassCastException
     */
    public int compareTo(T o);
}
```

#### 1.2 Comparator

比较器。

```java
/**
 * 一些集合用来排序的比较器。如Collections.sort(List,Comparator)、Arrays.sort(Object[],Comparator)，或不使用自然排序时SortedSet、SortedMap等集合中依赖的比较器。
 */
@FunctionalInterface
public interface Comparator<T> {
    /**
     * o1小于o2返回-1，o1=02返回0，o1大于o2返回1
     * @throws 当其中一个参数为null，并且该比较器不允许null时，抛NullPointerException
     * @throws 当T类型不能使用该比较器时，抛ClassCastException
     */
    int compare(T o1, T o2);

    //当obj也是一个比较器，并且和该comparator的排序规则一样时（实现类实现），返回true
    boolean equals(Object obj);

    // since1.8，有添加了很多方法
    ...
}
```

## 2.实现接口

TreeMap实现了NavigableMap接口，NavigableMap接口又实现了SortedMap接口。

```java
public class TreeMap<K,V> extends AbstractMap<K,V> implements NavigableMap<K,V>, Cloneable, java.io.Serializable {
    ...
}
```

#### 2.1 SortedMap

key排序过的Map。另外提供了一些基于key范围，获取子SortedMap视图的方法。

```java
/*
 * key基于自然排序或比较器排序后的Map（有序体现在遍历时，类似于SortedSet）
 * SortedMap中的keys，都实现了Comparable接口，或可以使用该类的comparator进行比较
*/
public interface SortedMap<K,conparV> extends Map<K,V> {
    
    //返回SortedMap中用于key之间进行比较的comparator
    Comparator<? super K> comparator();

    //返回SortedMap中，K在[fromKey,toKey)范围的所有键值对组成的子SortedMap视图
    //注意：对子SortedMap视图的改变会影响SortedMap的内容，反之亦然
    SortedMap<K,V> subMap(K fromKey, K toKey);
    //返回SortedMap中，K在(-∞,toKey)范围的所有键值对组成的子SortedMap视图
    //注意：对子SortedMap视图的改变会影响SortedMap的内容，vice-versa
    SortedMap<K,V> headMap(K toKey);
    //返回SortedMap中，K在[fromKey,+∞)范围的所有键值对组成的子SortedMap视图
    //注意：对子SortedMap视图的改变会影响SortedMap的内容，反之亦然
    SortedMap<K,V> tailMap(K fromKey);

    //返回SortedMap中第一个键，如果SortedMap为空，抛NoSuchElementException
    K firstKey();
    //返回SortedMap中最后一个键，如果SortedMap为空，抛NoSuchElementException
    K lastKey();

    //返回SortedMap中所有key组成的Set
    Set<K> keySet();
    //返回SortedMap中所有value组成的Collection
    Collection<V> values();
    //返回SortedMap中所有Entry视图
    //注意：对视图的改变会影响SortedMap的内容，反之亦然
    Set<Map.Entry<K, V>> entrySet();
}
```

#### 2.2 NavigableMap

SortedMap的增强接口，扩展了更多导航方法。

```java
public interface NavigableMap<K,V> extends SortedMap<K,V> {
    
    //返回NavigableMap中严格小于key的最大的Entry
    Map.Entry<K,V> lowerEntry(K key);
    //返回NavigableMap中严格小于key的最大的key
    K lowerKey(K key);

    //返回NavigableMap中严格大于key的最小的Entry
    Map.Entry<K,V> higherEntry(K key);
    //返回NavigableMap中严格大于key的最小的key
    K higherKey(K key);

    //返回NavigableMap中小于等于key的最大的Entry
    Map.Entry<K,V> floorEntry(K key);
    //返回NavigableMap中小于等于key的最大的key
    K floorKey(K key);

    //返回NavigableMap中大于等于key的最小的Entry
    Map.Entry<K,V> ceilingEntry(K key);
    //返回NavigableMap中大于等于key的最小的key
    K ceilingKey(K key);

    //返回NavigableMap中第一个Entry，如果NavigableMap为空返回null
    Map.Entry<K,V> firstEntry();
    //返回NavigableMap中最后一个Entry，如果NavigableMap为空返回null
    Map.Entry<K,V> lastEntry();
    //删除并返回NavigableMap中第一个Entry，如果NavigableMap为空，返回null
    Map.Entry<K,V> pollFirstEntry();
    //删除并返回NavigableMap中最后一个Entry，如果NavigableMap为空，返回null
    Map.Entry<K,V> pollLastEntry();

    //返回NavigableMap的逆序NavigableMap视图
    //注意：对视图的操作会影响原NavigableMap结构，反之亦然
    NavigableMap<K,V> descendingMap();
    //返回NavigableMap中所有key的逆序NavigableSet视图
    //注意：对视图的操作会影响原NavigableMap结构，反之亦然
    NavigableSet<K> descendingKeySet();

    //返回NavigableMap中所有key的NavigableSet视图
    //注意：对视图的操作会影响原NavigableMap结构，反之亦然
    NavigableSet<K> navigableKeySet();

    //返回NavigableMap中从fromKey到toKey的所有键值对组成的子NavigableMap视图，fromInclusive、toInclusive表示是否包含边界
    //注意：对视图的操作会影响原NavigableMap结构，反之亦然
    NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive, K toKey, boolean toInclusive);
    //返回NavigableMap中小于toKey的所有键值对组成的子NavigableMap视图，inclusive表示是否包含边界
    //注意：对视图的操作会影响原NavigableMap结构，反之亦然
    NavigableMap<K,V> headMap(K toKey, boolean inclusive);
    //返回NavigableMap中大于fromKey的所有键值对组成的子NavigableMap视图，inclusive表示是否包含边界
    //注意：对视图的操作会影响原NavigableMap结构，反之亦然
    NavigableMap<K,V> tailMap(K fromKey, boolean inclusive);

    //等价于继承自SortedMap的subMap(fromKey, true, toKey, false)
    SortedMap<K,V> subMap(K fromKey, K toKey);
    //等价于继承自SortedMap的headMap(toKey, false)
    SortedMap<K,V> headMap(K toKey);
    //等价于继承自SortedMap的tailMap(fromKey, true)
    SortedMap<K,V> tailMap(K fromKey);
}
```

## 3.成员变量

主要成员变量如下。

```java    
//用于比较器排序，natural ordering时为null
private final Comparator<? super K> comparator;
//红黑树根节点
private transient Entry<K,V> root;
//entry个数
private transient int size = 0;
//structural modification次数，用于实现fail-fast机制
private transient int modCount = 0;
```

## 4.构造

```java
//空参构造，comparator=null，自然排序，该情形下要求key实现Comparable接口
public TreeMap() {
    comparator = null;
}

//构造指定comparator比较器的TreeMap，该比较器用于key排序
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}

//通过Map构造，自然排序，要求m中的key实现Comparable接口。并使用putAll存入
public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}

//通过buildFromSorted函数，使用SortedMap构建一个红黑树TreeMap
public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```

## 5.红黑树节点

TreeMap中，使用内部类Entry表示红黑树的节点（HashMap是Node（Java8），LinkedHashMap是Entry）。

每个节点Entry的属性：
- key、value
- 左孩子(left)、右孩子(right)、父节点(parent)
- color颜色，TreeMap基于红黑树实现，每个节点非黑即红

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;    //左子节点
    Entry<K,V> right;   //右子节点
    Entry<K,V> parent;  //父节点
    boolean color = BLACK;//默认黑色

    //该构造创建的节点，默认是黑色、叶子节点
    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }
    public String toString() { return key + "=" + value; }

    //新值覆盖旧值，返回旧值
    public V setValue(V value) {
        V oldValue = this.value;
        this.value = value;
        return oldValue;
    }

    public boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>)o;
        return valEquals(key,e.getKey()) && valEquals(value,e.getValue());
    }

    public int hashCode() {
        int keyHash = (key==null ? 0 : key.hashCode());
        int valueHash = (value==null ? 0 : value.hashCode());
        return keyHash ^ valueHash;
    }
}
```

## 6.put(K key, V value)

```java
/**
 * 插入节点，key已存在时只覆盖值
 * @return key已存在时，返回原值（原值也可以是null）；key不存在时，返回null
 * @throws 当key不能和map中已存在的keys比较时，抛ClassCastException
 * @throws 当key=null，并且是自然排序或该map的比较器不允许键为null时，抛NullPointerException
 */
public V put(K key, V value) {
    Entry<K,V> t = root;
    /* 1.当插入的是空树时 */
    if (t == null) {
        //只是检查，是否抛ClassCastException或NullPointerException
        compare(key, key); // type (and possibly null) check
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    /* 2.当插入的是非空树时 */
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    // 2.1 比较器排序
    if (cpr != null) {
        //使用while循环遍历RBT，而非迭代
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);//key已存在，覆盖并返回原值
        } while (t != null);
    }
    // 2.2 自然排序
    else {
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);//key已存在，覆盖并返回原值
        } while (t != null);
    }
    //key不存在,直接插入
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    //插入后RBT的修复
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```

上述两种情况（自然排序、比较器排序）下的RBT遍历，都是为了查找key节点（值覆盖）或作为新Entry将要插入位置的parent。

```java
// RBT插入后的修复
private void fixAfterInsertion(Entry<K,V> x) {
    // 1.规定插入的新节点为红色
    x.color = RED;
    // 2.当新节点：不为null && 不是root节点 && 父节点也是红色时，才需要修复
    while (x != null && x != root && x.parent.color == RED) {
        //x的父节点是左节点（叔叔节点为null或是右节点）
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));//叔叔节点
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == rightOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateLeft(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateRight(parentOf(parentOf(x)));
            }
        //x的父节点是左节点（叔叔节点为null或是右节点）
        } else {
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));//叔叔节点
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    // 3.当x是根节点时，直接涂黑（当x为null或父节点为黑时，该操作无意义）
    root.color = BLACK;
}
```

> 关于红黑树插入后的修复，参考[二叉树](https://blog.wocaishiliuke.cn/datastructure/2018/12/03/Binarytree01/)。

## 7.putAll(Map<? extends K, ? extends V> map)

- 如果TreeMap为空，map又是SortedMap，并且map的比较器和当前TreeMap的比较器相同，就通过buildFromSorted()构建红黑树
- 否则调用AbstractMap.putAll()，依次插入map中的元素，每次插入都需要修复，效率不高

```java
/**
 * 将一个map中的所有键值对添加到TreeMap中
 * @throws ClassCastException if the class of a key or value in
 *         the specified map prevents it from being stored in this map
 * @throws NullPointerException if the specified map is null or
 *         the specified map contains a null key and this map does not
 *         permit null keys
 */
public void putAll(Map<? extends K, ? extends V> map) {
    int mapSize = map.size();
    if (size==0 && mapSize!=0 && map instanceof SortedMap) {
        Comparator<?> c = ((SortedMap<?,?>)map).comparator();
        if (c == comparator || (c != null && c.equals(comparator))) {
            ++modCount;
            try {
                buildFromSorted(mapSize, map.entrySet().iterator(), null, null);
            } catch (java.io.IOException cannotHappen) {
            } catch (ClassNotFoundException cannotHappen) {
            }
            return;
        }
    }
    super.putAll(map);
}
```

> 构造TreeMap(Map m)调用了该putAll方法。构造TreeMap(SortedMap m)则直接使用了buildFromSorted()，和该方法中的逻辑相似。

#### 7.1 buildFromSorted()

通过buildFromSorted()构建一颗新红黑树：使用迭代器从小到大，以折半的方式组装RBT。构建成的红黑树，最下方的节点是红色，其他都是黑色。

```java
/**
 * 基于已排序数据的树构建算法(线性时间). 可以从iterator或stream中接收keys和/或values
 * @param size 从iterator或stream中读取的key或k-v对的个数
 * @param it 如果非null, 将从iterator中读取的entries或keys插入TreeMap
 * @param str 如果非null, 将从str流中读取keys插入TreeMap.
 *        必须保证it和str其中一个不为null
 * @param defaultVal 如果非null, 则插入的键值对的value都为defaultVal; 否则
 *        从iterator或stream中读取value.
 * @throws java.io.IOException propagated from stream reads.
 * @throws ClassNotFoundException propagated from readObject.
 */
private void buildFromSorted(int size, Iterator<?> it, java.io.ObjectInputStream str, V defaultVal) throws  java.io.IOException, ClassNotFoundException {
    this.size = size;
    root = buildFromSorted(0, 0, size-1, computeRedLevel(size), it, str, defaultVal);
}
```

```java
//计算红黑树叶子结点的层数，在下面递归过程中，如果level=redLevel，则此时的结点为红色
private static int computeRedLevel(int sz) {
    int level = 0;
    for (int m = sz - 1; m >= 0; m = m / 2 - 1)
        level++;
    return level;
}
```

```java
/**
 * 真正创建树的方法(递归方式)
 * @param level 当前树的level，初始值为0
 * @param lo 当前子树的第一个元素的index，第一次递归为0
 * @param hi 当前子树的最后一个元素的index，第一次递归为size-1
 * @param redLevel 红节点的level
 */
@SuppressWarnings("unchecked")
private final Entry<K,V> buildFromSorted(int level, int lo, int hi, int redLevel, Iterator<?> it, java.io.ObjectInputStream str, V defaultVal)
    throws java.io.IOException, ClassNotFoundException {

    if (hi < lo) return null;

    int mid = (lo + hi) >>> 1;
    Entry<K,V> left  = null;
    if (lo < mid)
        left = buildFromSorted(level+1, lo, mid - 1, redLevel, it, str, defaultVal);

    // 从iterator或stream提取键值对
    K key;
    V value;
    if (it != null) {
        if (defaultVal==null) {
            Map.Entry<?,?> entry = (Map.Entry<?,?>)it.next();
            key = (K)entry.getKey();
            value = (V)entry.getValue();
        } else {
            key = (K)it.next();
            value = defaultVal;
        }
    } else { // use stream
        key = (K) str.readObject();
        value = (defaultVal != null ? defaultVal : (V) str.readObject());
    }
    Entry<K,V> middle =  new Entry<>(key, value, null);

    // color nodes in non-full bottommost level red
    if (level == redLevel)
        middle.color = RED;

    if (left != null) {
        middle.left = left;
        left.parent = middle;
    }

    if (mid < hi) {
        Entry<K,V> right = buildFromSorted(level+1, mid+1, hi, redLevel, it, str, defaultVal);
        middle.right = right;
        right.parent = middle;
    }

    return middle;
}
```

> 上述方法中，lo和hi参数是从迭代器或stream中取出元素的最小和最大索引（这些元素用于插入当前子树）。实际上没有索引，只是因为序列是有序的，可以通过这种折半取数的方式保证顺序。

该方法使用递归方式，将一个有序序列构建成红黑树。以序列[1,2,3,4,5,6,7,8,9,10]为例，构建过程为：
- 1.以最中间的数作为根结点，然后将序列分成两组，(1,2,3,4) (6,7,8,9,10)
- 2.递归第一组序列找出最中间的树作为根结点，建立一个子树，该子树作为整个树的左子树
- 3.递归第二组序列找出最中间的树作为根结点，建立一个子树，该子树作为整个树的右子树
- 4.最终形成的树，在叶子结点以上是一个满二叉树。并把所有的叶子节点标记为红色，满足红黑树的大致平衡要求

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/DataStructure/tree-map-build-from-sorted.jpg)

#### 7.2 putAll

不满足上述条件时（当前TreeMap为空，参数map又是SortedMap，且参数map的比较器和当前TreeMap的比较器相同），就不能使用buildFromSorted()。而是使用AbstractMap.putAll()（实际是TreeMap.put()）依次插入，但每次插入都需要修复，时间复杂度为O(n * logn)，效率低。

```java
//AbstractMap.putAll()
public void putAll(Map<? extends K, ? extends V> m) {
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
        //又调用自己覆写的TreeMap.put(),详见上述
        put(e.getKey(), e.getValue());
}
```

## 8.remove(Object key)

```java
/**
 * Removes the mapping for this key from this TreeMap if present.
 * @param  key 待删除的key
 * @return 待删除key对应的value（可以为null），如果key不存在，返回null
 * @throws 如果key和TreeMap中所有的key无法进行比较，抛ClassCastException
 * @throws 当key=null，并且是自然排序或比较器排序时该map的比较器不允许键为null时，抛NullPointerException
 */
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;

    V oldValue = p.value;
    deleteEntry(p);
    return oldValue;
}
```

getEntry(key)的分析见下述get(Object key)小节。deleteEntry()的实现分有三种情况（其实就是BST的删除）：

- 叶子节点：直接将父节点对应引用置null即可
- 只有一个孩子：直接将父亲节点和孩子节点建立链接
- 有两个孩子：需要寻找后继（右子树的最小值），替换当前节点的内容，然后删除后继节点（后继一定没有左孩子，就将两个孩子的情况转换为前两种情况）

```java
//Delete node p, and then rebalance the tree.
private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

    // 1.当p有两个孩子，使用后继节点s的内容替换当前节点p的内容。将真正要删除的p指向后继s
    if (p.left != null && p.right != null) {
        Entry<K,V> s = successor(p);
        p.key = s.key;
        p.value = s.value;
        p = s;//转化成叶子节点或单子节点的情况，从此p就是真正要删除的节点引用了
    }
    
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);//p的子节点
    // 2.p是单子节点
    if (replacement != null) {
        // 将p的父、子节点相连（两步）
        // 2.1 修改子节点的父引用
        replacement.parent = p.parent;
        // 2.2 修改父节点的子引用
        if (p.parent == null)// p是root节点，单子节点
            root = replacement;
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        else
            p.parent.right = replacement;

        // Null out links so they are OK to use by fixAfterDeletion.
        p.left = p.right = p.parent = null;
        // Fix replacement
        if (p.color == BLACK)
            fixAfterDeletion(replacement);// 修复的是最终删除的replacement，而非p
    // 3.p是叶子节点
    } else if (p.parent == null) {  // p又是root节点
        root = null;
    } else {
        if (p.color == BLACK)
            fixAfterDeletion(p);    // 修复的是最终删除的p

        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}
```

RBT删除节点后的修复逻辑，可参考[二叉树](https://blog.wocaishiliuke.cn/datastructure/2018/12/03/Binarytree01/)。

```java
private void fixAfterDeletion(Entry<K,V> x) {
    // 当真正删除的x，不是root并且是黑色时
    while (x != root && colorOf(x) == BLACK) {
        // x是左孩子
        if (x == leftOf(parentOf(x))) {
            Entry<K,V> sib = rightOf(parentOf(x));  // 兄弟节点
            // case1：左旋+变色，转化成其他case
            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                rotateLeft(parentOf(x));
                sib = rightOf(parentOf(x));
            }
            // case2：变色，并向上追溯
            if (colorOf(leftOf(sib))  == BLACK &&
                colorOf(rightOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            } else {
                // case4：右旋+变色，转化成case5
                if (colorOf(rightOf(sib)) == BLACK) {
                    setColor(leftOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateRight(sib);
                    sib = rightOf(parentOf(x));
                }
                // case5：左旋+变色
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(rightOf(sib), BLACK);
                rotateLeft(parentOf(x));
                x = root;
            }
        } else { // 对称操作（x是右孩子）
            Entry<K,V> sib = leftOf(parentOf(x));

            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                rotateRight(parentOf(x));
                sib = leftOf(parentOf(x));
            }

            if (colorOf(rightOf(sib)) == BLACK &&
                colorOf(leftOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            } else {
                if (colorOf(leftOf(sib)) == BLACK) {
                    setColor(rightOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateLeft(sib);
                    sib = leftOf(parentOf(x));
                }
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(leftOf(sib), BLACK);
                rotateRight(parentOf(x));
                x = root;
            }
        }
    }
    // 当真正删除的x，是root，或是红色时
    setColor(x, BLACK);
}
```

## 9.get(Object key)

```java
/**
 * 通过key获取value
 * @throws ClassCastException 由getEntry传递抛出
 * @throws NullPointerException 由getEntry传递抛出
 */
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}
```

```java
/**
 * 通过key获取Entry
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 */
final Entry<K,V> getEntry(Object key) {
    // 1.出于性能考虑，分离出使用比较器获取Entry的情况
    if (comparator != null)
        return getEntryUsingComparator(key);
    if (key == null)
        //comparator为null（自然排序），如果key也为null，抛NPE
        throw new NullPointerException();
    // 2.自然排序查找
    @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
    // while循环方式，遍历红黑树
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}

/** 使用comparator来getEntry（比较器排序查找） */
final Entry<K,V> getEntryUsingComparator(Object key) {
    @SuppressWarnings("unchecked")
        K k = (K) key;
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = cpr.compare(k, p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
    }
    return null;
}
```

## 10.containsKey(Object key)

TreeMap中是否存在键key。同上，因为使用getEntry()，也可能抛ClassCastException和NPE。

```java
public boolean containsKey(Object key) {
    return getEntry(key) != null;
}
```

## 11.containsValue(Object value)

判断当前TreeMap中是否存在值为value

```java
public boolean containsValue(Object value) {
    for (Entry<K,V> e = getFirstEntry(); e != null; e = successor(e))
        if (valEquals(value, e.value))
            return true;
    return false;
}
```

```java
// TreeMap中的第一个Entry（key最小的，最左最下的那个节点），空树返回null
final Entry<K,V> getFirstEntry() {
    Entry<K,V> p = root;
    if (p != null)
        while (p.left != null)
            p = p.left;
    return p;
}

// 返回t的后继节点successor（右子树的最小值）
static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.right != null) {
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    } else {
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```

其中，successor(Entry<K,V> t)的逻辑为：
- 1.先看t是否有右孩子，如果有，t的后继就是其右子树的最小节点
- 2.如果t没有右孩子，那么后继为某祖先节点，从当前节点t往上找
    + 如果t是其父节点p的左孩子，则其父节点p就是t的后继
    + 如果t是其父节点p的右孩子，则继续向上找p的父节点，直到某个节点不是右孩子或其父节点为空，那么第一个非右孩子节点的父亲节点就是后继节点
    + 如果父节点为空，则后继为null

例如下图中，2的后继是3，3的后继是4，4的后继是5...，9的后继是null。

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/DataStructure/treemap-successor.jpg)


---
# V.EnumMap

参考[EnumMap](http://lidol.top/java/1024/)


---
# VI.参考

- [Java编程拾遗『TreeMap』](http://lidol.top/java/detail/986/)