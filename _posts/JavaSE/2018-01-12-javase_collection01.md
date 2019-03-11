---
title: List
date: 2018-01-12 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Collection的子接口List的相关成员。

<!-- more -->

##### 目录
- [X] I.概述
- [x] II.


---

# I.概述

![avatar](TODO)

List是Collection的子接口。由于List是有序的，即有索引（Set没有），所以除了Collection中的通用方法外，List中还有基于索引的特有方法

```java
public interface List<E> extends Collection<E> {
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
    boolean addAll(int index, Collection<? extends E> c);
    boolean removeAll(Collection<?> c);
    boolean retainAll(Collection<?> c);

    void clear();
    boolean equals(Object o);
    int hashCode();
    E get(int index);
    E set(int index, E element);
    void add(int index, E element);
    E remove(int index);

    int indexOf(Object o);
    int lastIndexOf(Object o);
    ListIterator<E> listIterator();
    ListIterator<E> listIterator(int index);
    List<E> subList(int fromIndex, int toIndex);

    //since 1.8

    default void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final ListIterator<E> li = this.listIterator();
        while (li.hasNext()) {
            li.set(operator.apply(li.next()));
        }
    }

    @SuppressWarnings({"unchecked", "rawtypes"})
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }

    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.ORDERED);
    }
}
```

## 1.遍历

除了Collection中通用的2种遍历方式外，因为List有索引，所以它还有特有的遍历方式：

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}
```

## 1.并发修改异常

