---
title: Collection&Map
date: 2018-01-11 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Java集合：单列集合Collection、双列集合Map。

<!-- more -->

##### 目录
- [X] I.概述
- [x] II.Collection
- [x] III.Map
- [x] IV.参考


---

# I.概述

Java中的[集合](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)，基本可分为：单列集合Collection、k-v双列集合Map。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/java-collections-framework.png)

Collection和Map体系的设计，采用了经典的接口（interfaces）与实现（implementations）分离的思想，实现类都会实现一个或多个接口。另外，在接口和实现类之间，大多还使用了抽象类，在抽象类中做一些通用的实现，更方便实现接口。

## 1.示例

下面以队列（queue）为例，阐述集合体系中的【接口-抽象-实现】设计思想。

> **队列是种FIFO数据结构，队尾入队，队头出队。队列通常有两种实现方式：循环数组、链表**。

Queue接口只定义了队列的通用方法，没有体现具体的实现方式。以add()和remove()为例

```java
public interface Queue<E> extends Collection<E> {
    boolean add(E e);
    E remove();
    ...
}
```

Java中也提供了基于循环数组、链表的实现：ArrayDequeue<E>、LinkedList<E>。

```java
public class ArrayDeque<E> extends AbstractCollection<E> implements Deque<E>, Cloneable, Serializable {

    public boolean add(E e) {
        addLast(e);
        return true;
    }

    public E remove() { return removeFirst(); }
    ...
}
```
```java
public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, java.io.Serializable {

    public boolean add(E e) {
        linkLast(e);
        return true;
    }

    public E remove() { return removeFirst(); }
    ...
}
```

一般情况下，循环数组队列比链表队列效率高。但是循环数组容量有限，如果要收集的对象没有上限，可使用链表队列。

## 2.扩展

稍微扩展下Java中的数据结构和数据存储结构，方便后面内容的理解。
- 数据结构：队列、栈、树
- 数据存储结构
    + 线性表：
        + 数组：物理存储连续、固定长度。如数组、ArrayList
        + 链表：物理存储不连续、可伸缩，元素的逻辑顺序靠指针维持。分为单、双链表。如LinkedList

> 队列和栈区别

- 队列：FIFO，两端都能操作，但尾部只进不出，头部只出不进
- 栈：FILO，只能操作栈顶

**队列和栈都可以使用数组和链表实现。**

> 数组和链表的区别

||存储|长度|数据访问|
|:--|:---|:--|:------|
|数组|顺序存储|固定|随机访问快捷，数据移动麻烦|
|链表|非顺序存储，前驱和后继|可伸缩|访问麻烦，需要从头/尾顺延，数据移动方便|

数组和链表的差异决定了它们的使用场景。如果访问数据频繁，可使用数组；如果数据移位较多，可使用链表。


---
# II.Collection

数组长度是固定的，不够灵活。所以Java提供了可以存储任意对象、长度可变的集合（可变容器）。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/collection-system.png)

## 1.Collection和数组

> 和数组的区别

- 数组固定长度；集合长度可变
- 数组可以存储基本数据类型和引用数据类型；**集合只能存储引用数据类型**（自动装箱）

当元素个数固定，可使用数组，节省内存；否则使用集合，更灵活。

> 和数组的联系

集合中部分子类是基于数组实现的，如ArrayList，初始长度（第一次add时）10，超出时扩容1.5倍

## 2.Collection接口

所有单列集合都直接或间接地实现了Collection接口。接口中定义了一些通用方法，包括操作元素、操作集合等方法。

```java
public interface Collection<E> extends Iterable<E> {
    // 元素个数
    int size();
    // 判断集合是否为空
    boolean isEmpty();
    // 返回集合迭代器
    Iterator<E> iterator();
    // 集合转数组
    Object[] toArray();
    // 集合转数组
    <T> T[] toArray(T[] a);
    
    // 添加元素（List永远返回true；Set元素不能重复，可能返回false）
    boolean add(E e);
    // 删除集合中等于o的元素
    boolean remove(Object o);
    // 判断集合中是否存在一个与o相等的元素
    boolean contains(Object o);

    // 判断集合是否包含集合c中的所有元素
    boolean containsAll(Collection<?> c);
    // 将集合c中所有元素添加到集合中，如果调用改变了集合返回true
    boolean addAll(Collection<? extends E> c);
    // 从集合中删除另一个集合c中的所有元素，如果调用改变了集合返回true
    boolean removeAll(Collection<?> c);
    // 从集合中保留另一个集合c中的所有元素，如果调用改变了集合返回true
    boolean retainAll(Collection<?> c);
    // 清空集合
    void clear();
    
    // equals
    boolean equals(Object o);
    // hashCode
    int hashCode();

    /** 以下since 1.8 */
    // 如果集合中的元素满足Predicate，则删除元素，如果调用改变了集合返回true
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }

    // 返回集合的Spliterator迭代器
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }

    // 返回集合流，用于支持集合函数式编程
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }

    // 返回集合并行流
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
}
```

## 3.AbstractCollection

Java集合框架中，一个接口往往会对应一个抽象类，实现接口中的一些通用方法。Collection接口也有AbstractCollection抽象类，提供基于迭代器Iterator的contains、remove、toString等一些方法的实现，方便实现类更轻松地实现Collection接口。

```java
public abstract class AbstractCollection<E> implements Collection<E> {

    public abstract Iterator<E> iterator();

    public boolean contains(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        return false;
    }
    ...
}
```

## 3.Iterable和Iterator

通过java.util.Iterator对象，可以对实现了java.lang.Iterable接口（since 1.5）的类进行迭代。

> Collection接口继承了Iterable接口，而Map接口没有。

```java
public interface Iterable<T> {
   
    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

```java
public interface Iterator<E> {

    boolean hasNext();

    E next();

    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    // since 1.8
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```

迭代器用于集合的遍历，但由于实现类的数据结构不同，Iterator就将hasNext和next等方法抽象到接口层，由实现类自定义这两个方法的具体实现。如ArrayList的内部类Itr和ListItr。这样使用Iterator进行遍历时，风格统一。

#### 3.1 迭代器遍历

迭代器主要是用于遍历。

通过反复调用next()，可以逐个访问元素。当到达集合末尾时，next()将抛出noSuchElementException异常。因此在调用next()前，需要调用hasNext()，判断是否存在下一个元素。

```java
Collection<String> c = …;
Iterator<String> iterator = c.iterator();
while(c.hasNext()) {
    // 该方法会将指针向前移动一位
    String element = iterator.next();//如果没用泛型，就需要强转(String)it.next()，因为存入时是多态Object o = "a";
    //处理element
}
```

Java 5提供了一个更优雅的增强for循环语法糖：foreach。编译器会将foreach翻译为迭代器循环。

```java
for (String element : c) {
    //处理element
}
```

#### 3.2 迭代器删除

Iterator接口的remove()用于删除上次调用next()时返回的元素，该方法在单线程下可避免并发修改异常。

```java
Iterator<String> iterator = c.iterator();
iterator.next();
iterator.remove();
```

注意，remove()的调用对next()具有依赖性。调用remove()之前没有调用next()是不合法的，会抛IllegalStateException。比如删除两个连续的元素，不能直接这样调用：

```java
iterator.remove();
iterator.remove(); //remove前未调用next方法，异常
```

Iterator接口中没有提供add()，即普通的迭代器只能并发删除，不能并发插入。有些Iterator的实现类中提供了add()，支持并发插入，如ArrayList中的ListItr。

#### 3.3 迭代器forEachRemaining

forEachRemaining是Java8中新引入的default方法，方法的参数Consumer用于逐个消费集合中的元素，如下：

```java
List<String> list = Lists.newArrayList("this", "is", "list");
list.iterator().forEachRemaining(ele -> System.out.println("consume :" + ele));
```

## 4.Collection通用遍历

即所有的Collection都可以使用的遍历方式：
- 1.转数组遍历
- 2.迭代器遍历：iterator、foreach

```java
Collection<String> c = new ArrayList<>();
c.add("a");
c.add("b");
c.add("c");
```

#### 4.1 转数组遍历

```java
Object[] arr = c.toArray();
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);//Object arr[0] = "a";多态，调用String的toString()
}
```

循环内如果需要调用子类String特有的方法（多态的弊端），需要向下强转(String)arr[i];

#### 4.2 迭代器遍历

包括迭代器、JDK1.5提供的语法糖foreach

```java
Iterator<String> it = c.iterator();
while(it.hasNext()) {
    String s = it.next();
    // do something..
}

//增强for循环
for (String s : c) {
    System.out.println(s);
}
```


---
# III.Map

## 1.概述

双列集合的根接口。是个顶级接口（不像Collection还继承了Iterable）

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/map-system.png)

- 1.将键映射到值
- 2.键唯一，不可重复（每个键只能映射一个值）

每种Map保证键唯一的方式不同，根据它们实现唯一的方式，分为HashMap、TreeMap等

> Map键唯一，Set元素唯一。Set底层依赖Map（Set元素存到k位置，v=new Object()）

```java
public interface Map<K,V> {
    // 返回Map中EntrySet的长度
    int size();
    // Map是否为空
    boolean isEmpty();
    // 判断是否存在key
    boolean containsKey(Object key);
    // 判断是否存在value
    boolean containsValue(Object value);
    // 获取key对应的value
    V get(Object key);
    // 存入键值对，键已存在则替换旧值
    V put(K key, V value);
    // 删除键值对
    V remove(Object key);
    // 存入另一Map的所有键值对
    void putAll(Map<? extends K, ? extends V> m);
    // 清空
    void clear();
    // 返回Map中所有key组成的Set集合
    Set<K> keySet();
    // 返回Map中所有value组成的集合
    Collection<V> values();
    // 返回Map中所有键值对组成的Set集合
    Set<Map.Entry<K, V>> entrySet();

  
    interface Entry<K,V> {
        K getKey();
        V getValue();
        V setValue(V value);
        boolean equals(Object o);
        int hashCode();

        public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getKey().compareTo(c2.getKey());
        }

        public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getValue().compareTo(c2.getValue());
        }

        public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
        }

        public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
        }
    }

    // Comparison and hashing
    boolean equals(Object o);
    int hashCode();

    // Defaultable methods since 1.8

    // 获取Map中key对应的value，如果key不存在，则返回defaultValue
    default V getOrDefault(Object key, V defaultValue) {
        V v;
        return (((v = get(key)) != null) || containsKey(key))
            ? v
            : defaultValue;
    }

    // 遍历Map，通过BiConsumer处理Map中每一个键值对的key、value
    default void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
            action.accept(k, v);
        }
    }

    /* 将Map中所有的键值对的value，替换为根据BiFunction规则通过key-value
    得到的新值*/
    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        Objects.requireNonNull(function);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }

            // ise thrown from function is not a cme.
            v = function.apply(k, v);

            try {
                entry.setValue(v);
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
        }
    }

    // 如果key不存在，则添加一个key-value键值对
    default V putIfAbsent(K key, V value) {
        V v = get(key);
        if (v == null) {
            v = put(key, value);
        }

        return v;
    }

    // 删除key-value键值对
    default boolean remove(Object key, Object value) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        remove(key);
        return true;
    }

    // 将Map中key-oldValue键值对的value值替换为newValue
    default boolean replace(K key, V oldValue, V newValue) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, oldValue) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        put(key, newValue);
        return true;
    }

    // 将Map中key对应的值替换为value
    default V replace(K key, V value) {
        V curValue;
        if (((curValue = get(key)) != null) || containsKey(key)) {
            curValue = put(key, value);
        }
        return curValue;
    }

    /* 如果key不存在，则根据Function规则计算key对应的value值，
    并进行put操作(前提计算得到的value值也不为null)*/
    default V computeIfAbsent(K key,
            Function<? super K, ? extends V> mappingFunction) {
        Objects.requireNonNull(mappingFunction);
        V v;
        if ((v = get(key)) == null) {
            V newValue;
            if ((newValue = mappingFunction.apply(key)) != null) {
                put(key, newValue);
                return newValue;
            }
        }

        return v;
    }

    /* 如果key存在，则根据BiFunction规则通过key和对应的value
    计算newValue，如果newValue为null，则删除key对应的键值对，否则替换key对应的value值*/
    default V computeIfPresent(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue;
        if ((oldValue = get(key)) != null) {
            V newValue = remappingFunction.apply(key, oldValue);
            if (newValue != null) {
                put(key, newValue);
                return newValue;
            } else {
                remove(key);
                return null;
            }
        } else {
            return null;
        }
    }

    /* 相当于上述两个方法的综合，根据BiFunction，通过key和对应的value
    计算newValue，如果newValue为null，会删除对应KV映射，否则会插入或替换原KV映射*/
    default V compute(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue = get(key);

        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue == null) {
            // delete mapping
            if (oldValue != null || containsKey(key)) {
                // something to remove
                remove(key);
                return null;
            } else {
                // nothing to do. Leave things as they were.
                return null;
            }
        } else {
            // add or replace old mapping
            put(key, newValue);
            return newValue;
        }
    }

    /* 如果key不存在或对应值为null，则给其一个对应的非null值value。否则根据
    BiFunction规则通过oldValue和value计算一个newValue，替换或删除(newValue为null)原
    键值对*/
    default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if(newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }
}
```

## 2.AbstractMap

Map接口也有AbstractMap抽象类，提供一些基于Iterator<Map.Entry<K,V>>迭代器的containsKey、remove等方法的实现，方便实现Map接口。

```java
public abstract class AbstractMap<K,V> implements Map<K,V> {
    public boolean containsKey(Object key) {
        Iterator<Map.Entry<K,V>> i = entrySet().iterator();
        if (key==null) {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getKey()==null)
                    return true;
            }
        } else {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (key.equals(e.getKey()))
                    return true;
            }
        }
        return false;
    }
    ...
}
```


---
# IV.参考

- [javase collections overview](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)
- [Java编程拾遗](http://lidol.top/category/java/detail)
