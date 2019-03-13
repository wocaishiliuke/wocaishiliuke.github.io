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
- [x] II.ArrayList
- [X] III.LinkedList
- [X] IV.Vector


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

    //返回o在list中第一次出现的index，如果不存在返-1
    int indexOf(Object o);
    //返回o在list中最后一次出现的index，如果不存在返-1
    int lastIndexOf(Object o);
    ListIterator<E> listIterator();
    ListIterator<E> listIterator(int index);
    //截取，左闭右开
    List<E> subList(int fromIndex, int toIndex);

    //since 1.8

    //根据UnaryOperator规则，将list中每一个元素映射为另一个元素
    default void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final ListIterator<E> li = this.listIterator();
        while (li.hasNext()) {
            li.set(operator.apply(li.next()));
        }
    }

    //根据Comparator排序规则，进行排序
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

    //返回集合的Spliterator迭代器
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

## 4.实现类

ArrayList、Vector、LinkedList

|对比|ArrayList|Vector|LinkedList|
|:---|:----|:-----|:---------|
|数据结构|数组|数组|链表|
|效率|按索引查询快O(1)，增删慢O(N)|按索引查询快O(1)，增删慢O(N)|不支持随机访问；按索引查询慢（从头/尾找，O(N/2)）；两端增删快O(1)，中间增删需要先定位O(N)，但不需要移位，即修改本身快O(1)|
|线程|线程不安全，效率高|线程安全，效率低|线程不安全，效率高|

> 这里ArrayList和Vector的查询指的是，使用索引访问。两者查找元素效率比较低，未排序时，时间复杂度为O(N)，已排序时，可以通过二分查找，时间复杂度为O(logN)

当容器需要大量插入删除操作时，避免使用ArrayList，可选用LinkedList。

> 由于存在把线程不安全转变成线程安全的工具类，即使Vector是线程安全的，也没能避免被ArrayList替代


---

# II.ArrayList

ArrayList是List基于数组的实现。是物理存储连续动态容器。容量不足时，会进行扩容。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/ArrayList.png)

- 实现了RandomAccess接口，支持随机访问（因为物理存储连续），时间复杂度为O(1)
- 支持序列化
- 可以被克隆

## 1.成员变量

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

## 2.构造

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

## 3.常用方法

#### 3.1 add(E e)

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

> 另外扩容的逻辑也在这里grow()

#### 3.2 add(int index, E element)

其中，index不能大于集合size，即该方法不能跳跃添加元素，否则报索引越界：

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

#### 3.3 依赖equals

ArrayList中的contains(Object o)和remove(Object o)，依赖equals()。

这里以去重为例：

> String类重写了equals()，比较的是字符串本身，而非地址值

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("b");
List<String> newList = new ArrayList<>();
Iterator<String> it = list.iterator();
while (it.hasNext()){
    String s = it.next();
    if (!newList.contains(s))
        newList.add(s);
}
System.out.println(newList);
```

如果是自定义POJO，需要重写equals()

```java
public class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // getter and setter

    @Override
    public String toString() {
        return "User{" + "name=" + name + ", age=" + age + '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return age == user.age &&
                Objects.equals(name, user.name);
    }
}
```

如果User不重写equals()，即使用继承自Object的equals()，比较的是地址值，那么newList中会有3个对象。

```java
List<User> list = new ArrayList<>();
list.add(new User("a", 1));
list.add(new User("b", 1));
list.add(new User("a", 1));
List<User> newList = new ArrayList<>();
Iterator<User> it = list.iterator();
while (it.hasNext()){
    User user = it.next();
    if (!newList.contains(user))
        newList.add(user);
}
System.out.println(newList);
```


---

# III.LinkedList

LinkedList是List基于双向链表的实现。LinkedList中的每个元素是链表的一个节点。元素逻辑上连续，物理存储上不连续。不支持RandomAccess。

> 双向链表的节点包括可前驱和后继引用（指针），单向链表的节点只有后继。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/collection/LinkedList.png)

> 抽象类AbstractSequentialList跟ArrayList继承的AbstractList作用相同，也是为了实现接口分离，实现一些通用方法，方便扩展。

另外，LinkedList实现了Deque接口（Double Ended Queues），即可以操作双端队列。（链表本身可以实现队列和栈，所以LinkedList可用作栈、队列、双向队列）

## 1.内部实现

LinkedList使用内部类Node，表示每一个元素节点。每个节点包括元素item、前驱prev和后继next

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

## 2.成员变量

```java
//链表长度，默认为0
transient int size = 0;
//双向链表头节点
transient Node<E> first;
//双向链表尾节点
transient Node<E> last;
//继承自AbstractList
protected transient int modCount = 0;
```

## 3.构造

两个构造。使用无参构造时，size为0，头节点first、尾节点last都为null。

```java
public LinkedList() {}

public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

## 4.基本方法

下面使用索引index的方法，都会依赖Node<E> node(int index)方法。即先查找节点，再进行其他操作。

#### 4.1 add(E e)

向链表尾部添加一个元素。

> ArrayList基于数组存储结构实现，在添加元素时，要保证容量足够。但LinkedList在物理上是不连续的，没有容量的定义，所以add时，也不用像ArrayList那样提前分配和保证余量。只需要新增一个Node，然后修改链接即可。

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

//尾插法
void linkLast(E e) {
    //1.创建新Node，并将prev指向之前的尾节点，next置null
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    //2.将该Node作为新尾节点
    last = newNode;
    //3.如果链中只有该newNode，将该Node也作为头节点
    if (l == null)
        first = newNode;
    //4.如果链中之前有元素，不用管first。将原尾节点的next指向该节点，完成last和last-1互指
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

#### 4.2 add(int index, E element)

在指定位置插入元素。

> index在[0,size]区间，否则报索引越界

```java
public void add(int index, E element) {
    //1.检查index索引的合法性，否则报索引越界
    checkPositionIndex(index);

    //2.在尾部插入就使用尾插法linkLast（如上），否则在某Node前插入linkBefore
    if (index == size)
        linkLast(element);
    else
        //node(index)返回索引index位上原本的Node，然后在该Node前插入element
        linkBefore(element, node(index));
}

//检查index索引的合法性
private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;
}

//尾插法
void linkLast(E e) {
    //见上面add(E e)
}

//在节点succ前链入e
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;//succ是原本在index位上的元素
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}

//返回链表中指定index位上的Node
Node<E> node(int index) {
    // assert isElementIndex(index);
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

node(int index)方法中，使用了**二分查找**。如果索引位于前半部分（index<(size>>1)），从头开始查找，否则从尾开始查找，以提高效率。同时也验证了之前的结论，ArrayList支持随机访问，查询效率比较高，LinkedList虽然有索引，但不物理内存连续，不支持随机访问，需要从头或尾遍历，效率相对较低。

#### 4.3 remove(int index)

删除索引为index的节点。index需要在区间[0,size)

```java
public E remove(int index) {
    //1.检查index合法性
    checkElementIndex(index);
    //2.删除index位上的Node
    return unlink(node(index));
}

unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    //处理x.prev指针和x前一个节点的next指针
    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }
    //处理x.next指针和x后一个节点的prev指针
    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
    //处理x的节点值
    x.item = null;
    size--;
    modCount++;
    return element;
}

//检查index合法性
private void checkElementIndex(int index) {
    if (!isElementIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
private boolean isElementIndex(int index) {
    return index >= 0 && index < size;
}
```

#### 4.4 remove(Object o)

根据节点值删除。依赖equals()，所以要求o重写equals()，否则使用继承自Object的equals()是比较地址值。

> 根据值，从头开始遍历，删除第一个匹配的节点后结束。这里也可以看到，LinkedList（List）可以存储null，而且元素可以重复。

```java
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);  //见上面
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

#### 4.5 get(int index)

根据索引获取元素。index需要在区间[0,size)

```java
public E get(int index) {
    checkElementIndex(index);//见上面remove(int index)
    return node(index).item;//见上面add(int index, E element)
}
```

## 5.实现List接口

LinkedList实现了List接口，也有继承List接口的实现类（AbstractList和AbstractSequentialList）。所以上述List接口相关的方法，它都可以用。如使用迭代器遍历等。

## 6.实现Deque接口

['dek]，Deque是个双向队列，定义了支持栈和队列的相关方法，所以可以使用LinkedList实现栈、队列的相关操作。Deque继承自单向队列Queue，先来看下Queue接口。

#### 6.1 Queue接口

[kju]，单向队列，入队只能在尾部，出队只能在头部。

```java
public interface Queue<E> extends Collection<E> {
    //向队尾添加元素，队列满时，抛IllegalStateException
    boolean add(E e);
    //向队尾添加元素，队列满时，返回false，不抛异常
    boolean offer(E e);
    //从队头删除一个元素，队列空时，抛NoSuchElementException
    E remove();
    //从队头删除一个元素，队列空时，返回null，不抛异常
    E poll();
    //返回队头元素，不改变队列，队列空时，抛NoSuchElementException
    E element();
    //返回队头元素，不改变队列，队列空时，返回null，不抛异常
    E peek();
}
```

可以发现，队列同一个操作都有两个方法实现，主要区别在于对于一些极端情况的处理上，如队空、队满。另外，对于LinkedList实现的队列而言，队列容量没有限制，可以无限大，但对于像ArrayDeque这样的队列，就有容量限制，存在队列满的情况。

#### 6.2 Deque接口

**栈**是一种FILO数据结构，只操作头部。Deque中定义了一些栈相关操作的方法，并在LinkedList中给予实现，所以使用LinkedList可以实现队列操作。

> 源自JDK 1.0的java.util.Stack（继承自Vector）就是栈这一数据结构在Java中的实现，但现在很少使用。

**队列**是一种FIFO数据结构，两端都操作。但尾部只添加、头部只查看和删除。但Deque更为通用，可以操作两端。即除了上述栈操作方法，它还支持双向队列操作。

```java
public interface Deque<E> extends Queue<E> {
    ...

    /* 栈操作方法 */
    //入栈，如果栈容量已满，抛IllegalStateException
    void push(E e);
    //出栈，如果栈已空，抛NoSuchElementException
    E pop();
    //查看栈顶元素，栈为空时，返回null，不抛异常
    E peek();

    /* 双向队列操作方法 */
    //向双向队列头部添加元素，如果队列已满，抛IllegalStateException
    void addFirst(E e);
    //向双向队列尾部添加元素，如果队列已满，抛IllegalStateException
    void addLast(E e);
    //向双向队列头部添加元素，如果队列已满，返回false，不抛异常
    boolean offerFirst(E e);
    //向双向队列尾部添加元素，如果队列已满，返回false，不抛异常
    boolean offerLast(E e);
    //删除双向队列头部元素，如果队列为空，抛NoSuchElementException
    E removeFirst();
    //删除双向队列尾部元素，如果队列为空，抛NoSuchElementException
    E removeLast();
    //删除双向队列头部元素，如果队列为空，返回null，不抛异常
    E pollFirst();
    //删除双向队列尾部元素，如果队列为空，返回null，不抛异常
    E pollLast();
    //获取双向队列头部元素，不改变结构，如果队列为空，抛NoSuchElementException
    E getFirst();
    //获取双向队列尾部元素，不改变结构，如果队列为空，抛NoSuchElementException
    E getLast();
    //获取双向队列头部元素，不改变结构，如果队列为空，返回null，不抛异常
    E peekFirst();
    //获取双向队列尾部元素，不改变结构，如果队列为空，返回null，不抛异常
    E peekLast();
    //删除双向队列中第一次出现的元素o，如果不包含o，返回false
    boolean removeFirstOccurrence(Object o);
    //删除双向队列中最后一次出现的元素o，如果不包含o，返回false
    boolean removeLastOccurrence(Object o);
    //获取双向队列逆序迭代器
    Iterator<E> descendingIterator();
}
```

## 7.迭代器

LinkedList有以下4种获取迭代器的方式。跟ArrayList相似，会有并发修改异常。

```java
//1.继承自AbstractList，返回从头向尾的单向迭代器Iterator
Iterator<E> iterator()

//2.实现自List，返回从index位置开始的的双向迭代器
public ListIterator<E> listIterator(final int index) {
    rangeCheckForAdd(index);

    return new ListItr(index);
}

//3.实现自Deque，返回从尾向头的单向Iterator迭代器
public Iterator<E> descendingIterator() {
    return new DescendingIterator();
}

//4.返回LinkedList的并行Spliterator迭代器
default Spliterator<E> spliterator() {
    return Spliterators.spliterator(this, Spliterator.ORDERED);
}
```


---

# IV.Vector

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