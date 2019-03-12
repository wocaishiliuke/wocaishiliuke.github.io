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
- [X] I.List
- [x] II.实现类


---

# I.List

List是Collection的子接口。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/List.png)

List是有序的，即有索引（Set没有）。所以除了Collection中的通用方法外，List中还有一些基于索引的特有方法。

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

## 1.索引遍历

除了Collection中通用的2种遍历方式外，因为List有索引，所以它还有自己的遍历方式：

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}
```

## 2.并发修改异常

ConcurrentModificationException。即在遍历时，增删元素，报出的异常。使用Iterator或foreach时都会出现该异常。

> 增强for循环只是一个语法糖，编译后还是使用的迭代器Iterator

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");

Iterator<String> it = list.iterator();
while (it.hasNext()){
    String s = it.next();
    if ("a".equals(s))
        list.remove("a");
}
System.out.println(list);
```

```java
Exception in thread "main" java.util.ConcurrentModificationException
    at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
    at java.util.ArrayList$Itr.next(ArrayList.java:859)
    at com.baicai.thread.Test1.main(Test1.java:40)
```

#### 2.1 原因

根据报错查看ArrayList$Itr源码

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    
    //继承自AbstractList
    //对List的修改次数。ArrayList中的add()和remove()，都会modCount++
    protected transient int modCount = 0;
    
    //继承自AbstractList
    public Iterator<E> iterator() {
        return new Itr();
    }

    //An optimized version of AbstractList.Itr
    private class Itr implements Iterator<E> {
        int cursor;                     // 下一个要访问的元素索引，类变量，有初始值0
        int lastRet = -1;               // 上一个访问的元素索引
        int expectedModCount = modCount;// 期望对List的修改次数，初始值为modCount

        Itr() {}

        public boolean hasNext() { return cursor != size; }

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

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
        ...
    }

    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
    ...
}
```

- 循环开始时，cursor=0, lastRet=-1, expectedModCount=modCount=0
- list.iterator()，执行从AbstractList继承的方法，返回new Itr()
- 执行it.hasNext()，即cursor和size是否相等，是否到达末尾了
- 执行String s = it.next()后，cursor=1, lastRet=0, expectedModCount=modCount=0
- if条件成立，实际执行子类ArrayList的remove，调用fastRemove()：modCount加1，arraycopy后，集合size-1，最后一个元素引用置为null，方便GC回收。此时cursor=1, lastRet=0, expectedModCount=0, modCount=1
- 该次循环结束，执行下次循环，执行到it.next()时会checkForComodification()，由于expectedModCount!=modCount抛出异常

#### 2.2 解决

- 1.使用迭代器Itr提供的remove()，而非list的remove

> 在Itr.remove()中，有expectedModCount = modCount操作，所以不会出现并发修改异常。

```java
Iterator<String> it = list.iterator();
while (it.hasNext()){
    String s = it.next();
    if ("a".equals(s))
        it.remove();
}
```

> 但没有Itr.add方法，所以该方式只能解决"并发删除异常"，对于"并发增加"无计可施

- 2.使用另一个内部类ListItr

该类继承增强了Itr

```java
private class ListItr extends Itr implements ListIterator<E> {
    ...

    public void add(E e) {
        checkForComodification();

        try {
            int i = cursor;
            AbstractList.this.add(i, e);
            lastRet = -1;
            cursor = i + 1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    public boolean hasPrevious() {
            return cursor != 0;
        }

    public E previous() {
        checkForComodification();
        try {
            int i = cursor - 1;
            E previous = get(i);
            lastRet = cursor = i;
            return previous;
        } catch (IndexOutOfBoundsException e) {
            checkForComodification();
            throw new NoSuchElementException();
        }
    }
}
```

修改后代码为：

```java
ListIterator<String> lit = list.listIterator();//多态
while (lit.hasNext()){
    String s = lit.next();
    if ("a".equals(s)){
        lit.remove();
        lit.add("w");
    }
}
```

- 3.多线程下的并发修改

上面两个在单线程下可以避免并发修改异常。但在多线程下，对于共享List的并发读写出现并发修改异常的原因：**不是因为ArrayList是线程不安全的（使用Vector依然会报并发修改异常），而是因为modCount属于List，expectedModCount属于iterator（Itr）。即modCount是共享的，而每个线程都有自己的expectedModCount**。这就导致了即使使用了iterator.remove()，修改的线程没问题，但其他线程的expectedModCount可能还是0，即使modCount不是volatile的，其他线程也可能读到修改线程写回的最新值1，抛出异常。

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");

Thread t1 = new Thread(new Runnable() {
    @Override
    public void run() {
        Iterator<String> it = list.iterator();
        while (it.hasNext()){
            System.out.println("t1--->" + it.next());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
});
Thread t2 = new Thread(new Runnable() {
    @Override
    public void run() {
        Iterator<String> it = list.iterator();
        while (it.hasNext()){
            String s = it.next();
            System.out.println("t2--->" + s);
            if ("b".equals(s)){
                it.remove();
                System.out.println("t2 del " + s);
            }
        }
    }
});
t1.start();
t2.start();
```

```java
t1--->a
t2--->a
t2--->b
t2 del b
t2--->c
Exception in thread "Thread-0" java.util.ConcurrentModificationException
```

此时解决方式有两种：
> - 1.iterator迭代时，使用synchronized或者Lock进行同步，阻塞其他线程
> - 2.使用并发容器CopyOnWriteArrayList代替ArrayList和Vector

#### 2.3 其他

以下代码不会出现并发异常（删除的是倒数第二个元素）：

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");

Iterator<String> it = list.iterator();
while (it.hasNext()){
    String s = it.next();
    if ("b".equals(s))
        list.remove("b");
}
System.out.println(list);
```

执行了list.remove()后，虽然modCount自增了，但同时集合的size也减少了1。下次进行hasNext判断时，cusor=3=size，hasNext()返回false，不会进入循环体执行next()，也就不会执行checkForComodification()，即不会抛异常。

Java 8之后，还可以调用Collection接口的新方法removeIf()实现相同的效果：

```java
list.removeIf(string -> string.equals("b"));
```

## 3.双向遍历

上述ListItr还提供了previous()和hasPrevious()，与next()和hasNext()对应，即ListItr支持双向遍历

```java
ListIterator<String> lit = list.listIterator(list.size());
while (lit.hasPrevious()){
    System.out.println(lit.previous());
}
```


---

# II.实现类

这里介绍ArrayList、LinkedList、Vector。

## 对比

|对比|ArrayList|Vector|LinkedList|
|:---|:----|:-----|:---------|
|数据结构|数组|数组|链表|
|效率|查询（修改）快O(1)，增删慢O(N)|查询（修改）快，增删慢|查询（修改）慢，增删快|
|线程|线程不安全，效率高|线程安全，效率低|线程不安全，效率高|

> 这里ArrayList和Vector的查询指的是，使用索引访问。两者查找元素效率比较低，未排序时，时间复杂度为O(N)，已排序时，可以通过二分查找，时间复杂度为O(logN)

当容器需要大量的插入删除操作时，避免使用ArrayList，可选用LinkedList。

> 由于存在把线程不安全转变成线程安全的工具类，即使Vector是线程安全的，也会被ArrayList替代

## ArrayList

ArrayList是基于数组实现的动态容器，容量不足时，会进行扩容。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/ArrayList.png)

- 实现了RandomAccess接口，即支持快速随机访问（通过数组下标），时间复杂度为O(1)
- 支持序列化
- 可以被克隆

#### 源码

ArrayList有两个重要成员：存放元素的Object[] elementData、集合长度size。类中的所有方法都是基于这两个成员实现的。

```java
// 数组elementData默认容量
private static final int DEFAULT_CAPACITY = 10;
//空数组
private static final Object[] EMPTY_ELEMENTDATA = {};
//默认容量的空元素数组
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

//存放元素的集合
transient Object[] elementData; // non-private to simplify nested class access
//集合长度
private int size;
```

#### 构造

```java
public ArrayList(int initialCapacity) {
    ...
}

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

public ArrayList(Collection<? extends E> c) {
    ...
}
```

通过默认空参构造创建的ArrayList，被赋值ULTCAPACITY_EMPTY_ELEMENTDATA空数组。**即如果不进行后续操作，其实长度是0而非10（这算是种优化）。但如果添加元素，会在add()中设置默认容量10。**

#### add(E e)

这里以空参构造后，add("a")为例

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!&&确保数组容量
    elementData[size++] = e;           // 存入元素、长度+1
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

//计算容量
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

//是否需要扩容
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);//1.5倍
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

- 经过calculateCapacity({},1)后，执行ensureExplicitCapacity(10)
- modCount+1，并且if(10-0>0)成立，执行grow(10)，最终newCapacity=10

即可以理解成：**通过ArrayList的默认构造函数，（第一次执行add时）内部数组的容量默认为10**

#### add(int index, E element)

index不能大于集合size，即该方法不能跳跃添加元素，否则报索引越界：

```java
List<Integer> integerList = new ArrayList<>();
integerList.add(1, 1);
integerList.add(3, 2);  //IndexOutOfBoundsException
```

查看源码，可以看到在rangeCheckForAdd()中做校验：

```java
public void add(int index, E element) {
    //检查索引
    rangeCheckForAdd(index);
    //保障数组容量
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //移位操作，将index位置及以后的元素往后移一位
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    //对index位赋值
    elementData[index] = element;
    size++;
}

private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```


## LinkedList



## Vector

自JDK1.0，比Collection（JDK1.2）体系还早。后来归入到List接口下，但被同是基于数组的ArrayList代替。

- 在没有继承List前，使用枚举遍历：

```java
Vector<String> v = new Vector<>();
v.add("a");
v.add("b");

// 按元素顺序，返回该Vector的枚举
Enumeration<String> en = v.elements();
while (en.hasMoreElements()){
    System.out.println(en.nextElement());
}
```

- 继承List后，可以使用List的方式遍历