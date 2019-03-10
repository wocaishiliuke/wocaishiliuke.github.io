---
title: Collection体系
date: 2018-01-11 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Java Collection家族的相关成员。

<!-- more -->

##### 目录
- [x] I.集合
- [x] II.Collection接口

---

# I.集合

## 1.概述

数组长度是固定的（固定容器），不够灵活。Java提供了可以存储任意对象、长度可变的集合（可变容器）。

## 2.和数组的区别

- 1.数组固定长度；集合长度可变
- 2.数组可以存储基本数据类型和引用数据类型；**集合只能存储引用数据类型**（自动装箱）

> 当元素个数固定，可使用数组，节省内存；否则使用集合，更灵活。

> 集合中部分子类是基于数组实现的，如ArrayList，初始长度10，超出时扩容1.5倍

## 3.继承体系

Java中的常用容器，基本可以分为两类：表示集合的Collection、表示K-V的Map。其中所有集合都实现了Collection接口，所有的Map都实现了Map接口。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/java-collections-framework.png)

集合体系的设计，采用了经典的接口(interfaces)与实现(implementations)分离的思想，具体集合类都会实现一个或多个接口。另外，在接口和具体实现类之间，大多还使用了抽象类，在抽象类中做了一些通用的实现，是为了更方便的实现接口。以队列（queue）为例： 

> **队列是种FIFO的数据结构，入队在队尾，出队在队头**。Queue接口中只定义了队列的通用方法，没有说明队列的具体实现方式。这里以add()和remove()为例

```java
public interface Queue<E> extends Collection<E> {
    boolean add(E e);
    E remove();
    ...
}
```

> **队列通常有两种实现方式：循环数组和链表**。Java提供了对应实现：ArrayDequeue<E>和LinkedList<E>。一般情况下，循环数组队列比链表队列效率高，但是循环数组是个有界集合，容量有限。如果程序中要收集的对象没有上限，需要使用链表队列。

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


---

# II.Collection接口

```java
package java.util;

import java.util.function.Predicate;
import java.util.stream.Stream;
import java.util.stream.StreamSupport;

public interface Collection<E> extends Iterable<E> {
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean remove(Object o);
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean removeAll(Collection<?> c);
    boolean retainAll(Collection<?> c);
    void clear();
    boolean equals(Object o);
    int hashCode();

    /** 以下since 1.8 */

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

    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }

    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }

    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
}
```