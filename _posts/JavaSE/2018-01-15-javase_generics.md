---
title: Generics
date: 2018-01-15 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录泛型的相关知识。

<!-- more -->

##### 目录
- [X] I.简介
- [X] II.泛型的使用
- [X] III.类型变量的限定
- [X] IV.泛型通配符
- [X] V.泛型和虚拟机
- [X] VI.泛型的使用问题
- [X] VII.参考


---

# I.简介

Java 5引入了泛型特性，是JSR 14规范的实现。常用在Java容器类和一些工具类中，甚至可以说，Java泛型的引入是为了更好的使用Java集合容器。Java 7又引入了"菱形泛型"，可以省略构造函数中的泛型。

> 数组和集合都是由一组类型相同的元素组成。数组在声明时可以直接指定元素类型，而集合就是依靠泛型来限定元素类型。

```java
int[] arr = new int[5];
List list = new ArrayList<>();  // 默认Object，什么都能放
List<String> list = new ArrayList<>();
```

## 1.泛型和形参的类比 

泛型可以认为是类、接口、方法定义时的参数，只不过参数只能接收class和interface。将方法形参看做一般参数formal parameters的话，泛型可看做类型参数type parameters。

和方法形参类比：
- 相似点：在不同input时，泛型和方法形参都能使得代码重用
- 不同点：方法形参的input是具体的值，泛型的input是类型

## 2.Why Use Generics

Code that uses generics has many benefits over non-generic code:

- 1.更安全

Java是强类型语言，compiler提供了严格的检查机制。而泛型将原本运行期错误，提前暴露在编译期，提高程序安全性。

- 2.更简单

泛型消除了类型转化（强转）。

```java
// non-generic code
List list = new ArrayList();
list.add("hello");
String s = (String) list.get(0);

// uses generics
List<String> list = new ArrayList<String>();
list.add("hello");
String s = list.get(0);   // no cast
```

- 3.更便捷

使用泛型，开发人员可以只开发一套逻辑代码，而适用不同的类型。提高代码重用性、可读性。


---
# II.泛型的使用

## 1.泛型接口

如Java中自然排序时，需要实现的Comparable接口。在实现类中完成具体类型的指定。

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

```java
public final class Integer extends Number implements Comparable<Integer> {
    public int compareTo(Integer anotherInteger) {
        return compare(this.value, anotherInteger.value);
    }
    //...
}
```

## 2.泛型类

```java
public class Box<T> {
    private T data;

    public Box() {}
    public Box(T data) {
        this.data = data;
    }

    public T getData() {
        return data;
    }
    public void setData(T data) {
        this.data = data;
    }
}
```

与普通类相比，Box类引入了类型参数T。T在泛型类中可用来指定成员变量类型、方法返回值类型、局部变量类型等。泛型类也可以使用多个类型参数变量，如：

```java
public class Box<T, U> {
    private T data;
    private U color;
    ...
}
```

> 不能在普通静态方法、静态代码块等静态内容中，使用泛型的类型参数，编译通不过

```java
public class Box<T> {
    ...
    public static T getInput(T t) { // 编译报错
        return t;
    }

    static {
        T t1 = null;                // 编译报错
    }

    // 该方法不是泛型方法，只是使用了泛型类Box的类型变量T，用于限定方法返回值和形参的类型
    public T print(T t) {           // 编译通过
        System.out.println(t);
        return t;
    }
}
```

对于上述静态方法无法使用Box的类型变量T，以及非泛型类中方法的泛型问题（更常见），就需要使用泛型方法。

## 3.泛型方法

一个方法是不是泛型方法，跟它所处的类是不是泛型类没有关系。普通类中也可以定义泛型方法，只是泛型类中更常见而已。

> 格式

```java
修饰符 <类型参数列表> 返回类型 方法名(形参列表) { 方法体 }
```

> 泛型方法声明时，类型变量要放在修饰符后、返回值前。示例：

```java
public static <T> T getFirst(T[] arr) {
    if (arr == null || arr.length == 0)
        return null;
    return arr[0];
}
```

> 泛型方法的调用

```java
public class Box<T> {
    ...
    // 泛型方法
    public static <T, S> int func(List<T> list, Map<Integer, S> map) {
        return 0;
    }

    public static void main(String[] args) {
        Box<String> box = new Box();
        List<Integer> list = new ArrayList<>();
        Map<Integer, Double> map = new HashMap<>();
        box.<Integer,Double>func(list,map);//这里T泛List的Integer，而非Box的String
        //box.func(list,map);   //调用时省略泛型，更常用
    }
}
```

## 4.泛型类的继承

#### 4.1 Box<S>和Box<T>

Integer是Number的子类，但是Box<Integer>并不是Box<Number>的子类，也就不能把Boxr<Integer>对象赋值给Box<Number>类型的引用。即无论S和T是什么关系，Box<S>和Box<T>都是没有联系的。

#### 4.2 转化

可以将参数化类型Box<T>转化为一个原始类型Box。

```java
Box<Number> numBox = new Box<>(10);
Box box = numBox;
box.setData("hello");               //编译通过
Number data = (Number)box.getData();//报ClassCastException
```

Box<Number>是Box的一个子类型。因为泛型是Java 5引入的新特性，所以在兼容遗留代码时，这种转化比较实用，但是会失去泛型的安全性。

Java中的ArrayList<T>实现了List<T>。所以ArrayList<Integer>可以转化为List<Integer>。但ArrayList<Integer>不是ArrayList<Number>的子类，也不是List<Number>的子类。

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/generics/generic.jpg)


---
# III.类型变量的限定

泛型变量默认是Object类型。如果泛型变量要使用Object类没有的方法，那么泛到Object肯定是不行的。

> 如上述Box<T>，如果在其某方法内调用compareTo()，肯定是不行的：

```java
public class Box<T> {
    private T data;
    
    ...

    public void test(T t) {
        data.compareTo(t);// 编译报错：T默认是Object类型，不能调用自己没有的方法
    }
}
```

所以Java中支持给泛型变量指定上界，对类型变量加以约束，这样在使用泛型变量时，就会把它当作指定的上界类型，而不再是Object类型。上界通过extends关键字来限定，并且上界可以是类、接口，也可以是其他类型参数。

## 1.上界是接口

限定后，T就不再被当做Object类型，而是上限Comparable类型。

```java
// 在泛型类上限定
public class Box<T extends Comparable<T>> {
    private T data;

    public Box() {}
    public Box(T data) {this.data = data;}

    public T getData() {return data;}
    public void setData(T data) {this.data = data;}

    public void test(T t) {
        data.compareTo(t);// T默认是Comparable类型，可以调用自己的compareTo()
    }
}
```

```java
// 在泛型方法上限定
public <T extends Comparable<T>> Box<T> print(T[] arr) {
    // do sth...
    arr[0].compareTo(arr[1]);
    return new Box<>(arr[0]);
}
```

> <T extends Comparable<T>>也称为递归类型限制。

## 2.上界是具体类

```java
public class Box<T extends Number> {
    private T data;

    public Box() {}
    public Box(T data) {this.data = data;}

    public T getData() {return data;}
    public void setData(T data) {this.data = data;}

    public void print() {
        System.out.println(data.intValue());// T默认是Number类型
    }
}
```

```java
// 在泛型方法上限定
public <T extends Number> Box<T> print(T t) {
    // do sth...
    System.out.println(t.intValue());
    return new Box<>(t);
}
```

## 3.上界是类型变量

除了类或接口，上界也可以是其他的类型变量。

> 假设Box<T>如下，类中提供了equalData()，对两个Box<T>实例的data进行比较。

```java
// Box class
public class Box<T> {
    private T data;

    public Box() {}
    public Box(T data) {this.data = data;}

    public T getData() {return data;}
    public void setData(T data) {this.data = data;}

    public boolean equalData(Box<T> other) {
        return data.equals(other.getData());
    }
}
```

```java
public class BoxTest {
    public static void main(String[] args) {
        Box<Number> numBox = new Box<>(1);
        Box<Integer> intBox = new Box<>(1);
        System.out.println(numBox.equalData(intBox));  // 编译报错
    }
}
```

编译报错提示如下，即不能将Box<Integer>对象赋值给Box<Number>：

```sql
equalData(com.baicai.generic.Box<java.lang.Number>) in Box cannot be applied to (com.baicai.generic.Box<java.lang.Integer>)
```

原因：使用numBox调用equalData()时，具体调用的是equalData(Box<Number> other)，故传参编译报错。**虽然Integer是Number的子类，但是Box<Integer>并不是Box<Number>的子类**，也就不能把Box<Integer>对象赋值给Box<Number>类型的引用other，此时可以将上界定义成类型变量。

> 解决如下。此时再测试，输出true。

```java
// 修改equalData()，使用其他类型变量E，T是E的上界
public class Box<T> {
    ...
    public <E extends T> boolean equalData(Box<E> other) {
        return data.equals(other.getData());
    }
}
```


---
# IV.泛型通配符

固定的泛型类型变量使用起来不是一直那么方便，Java泛型中引入了通配符（仍是安全的）。

## 1.通配符的子类型限定

#### 1.1 示例

上述Box<Integer>和Box<Number>的示例，使用了<E extends T>来解决。通过声明新的类型变量E，达到非固定泛型类型的效果。这种方式较繁琐，还可以使用通配符实现非固定泛型类型：

```java
public boolean equalData(Box<? extends T> other) {
    return data.equals(other.getData());
}

// test code
public static void main(String[] args) {
    Box<Number> numBox = new Box<>(1);
    Box<Integer> intBox = new Box<>(1);
    System.out.println(numBox.equalData(intBox));
}
```

当numBox调用equalData()时，它的T为Number，此时形参other的类型为Box<? extends Number> other，就可以接收Box<Integer>类型的intBox了，Box<Double>、Box<Long>等类型的实参也都可以。

> <E extends T>和<? extends T>的区别
- <E extends T>是**定义类型参数**。声明了新的类型变量E，E可用于泛型类、泛型方法的定义中
- <? extends T>是**实例化类型参数**。?是个具体的类型，只不过这个具体类型是未知的，只知道它是T或T的子类

虽然两种写法不同，但效果相同。一般能使用通配符解决的，通过类型变量也能实现，但是通配符更加简洁，可读性更好：

```java
public boolean equalData(Box<? extends T> other);
public <E extends T> boolean equalData(Box<E> other);
```

#### 1.2 子类型限定通配符的读写特性

如下，两个引用intBox和numBox，分别是Box<Integer>和Box<? extends Number>类型的，但指向同一内存空间（实例）。numBox.setData()理应可以接收Number或其子类型的实参，但不行。

```java
public static void main(String[] args) {
    Box<Integer> intBox = new Box<>(1);
    Box<? extends Number> numBox = intBox;
    intBox.setData(2);             // 编译不报错
    numBox.setData(3);             // 编译报错
    Number data = numBox.getData();// 编译不报错
}
```

如果允许numBox.setData(3)，就相当于可以把data设置为任何Number的子类型。由于两个引用指向同一内存空间，就可能使得intBox中出现非Integer类型的成员，破坏了泛型的安全性，所以需要**禁止对通配符泛型的写操作**。

但getData操作不会出现这个问题，因为getData的返回值赋值给Number类型的引用是完全合法的。也就造成一个现象：**子类型限定通配符是只读的**。

## 2.通配符的超类型限定

上面使用通配符的子类型限定，达到和声明新类型变量一样的非固定泛型效果。对于通配符，还可以指定超类型限定。

```java
// 匹配E的某个父类型
? super E
```

#### 2.1 示例

上面通配符的子类型限定中，解决了接收子类型参数的问题：

> Box<? extends Number>可以接收Box<Integer>，所以numBox.equalData(intBox)没问题，但intBox.equalData(numBox)依然编译报错。

```java
public boolean equalData(Box<? extends T> other) {
    return data.equals(other.getData());
}

// test code
public static void main(String[] args) {
    Box<Number> numBox = new Box<>(1);
    Box<Integer> intBox = new Box<>(2);
    System.out.println(numBox.equalData(intBox));// 使用通配符子类型限定，已解决
    System.out.println(intBox.equalData(numBox));// 仍编译报错
}
```

因为intBox.equalData(numBox)调用时，Box<? extends Integer>无法接收Box<Number>，Number是Integer的父类而非子类。

> 解决

```java
public boolean equalSuperData(Box<? super T> other) {
    return data.equals(other.getData());
}

// test code
public static void main(String[] args) {
    Box<Number> numBox = new Box<>(1);
    Box<Integer> intBox = new Box<>(2);
    System.out.println(numBox.equalData(intBox));// 使用通配符子类型限定，已解决
    System.out.println(intBox.equalSuperData(numBox));// 编译不报错
}
```

#### 2.2 超类型限定通配符的读写特性

上面讲过，子类型限定通配符是只读的，下面看一下超类型限定通配符的读写特性。

```java
/* 子类型限定通配符 */
Box<Integer> intBox = new Box<>(1);
// 也可以直接Box<? extends Number> extendsBox = new Box<>(1);
Box<? extends Number> extendsBox = intBox;
extendsBox.setData(2);                        // 编译报错，子类型限定通配符是只读的
Number extendsBoxData = extendsBox.getData(); // 可读，可以使用Number接收

/* 超类型限定通配符 */
Box<Number> numBox = new Box<>(10);
// 也可以直接Box<? super Integer> superBox = new Box<>(10);
Box<? super Integer> superBox = numBox;
superBox.setData(11);                    // 编译不报错，超类型限定通配符可写
Object superBoxData = superBox.getData();// 也可读，但只能用Object接收，不能用Number
```

numBox和superBox也指向同一内存地址。superBox.setData()接收Integer或其父类型的实参，那么该实参肯定可以安全地转换为Number（不会报ClassCastException），不会破坏泛型安全，所以**超类型限定通配符是可写的**。

在superBox.getData()读取时，由于返回值类型为<? super Number>，无法具体确认使用什么类型接收，就只能使用最顶层的Object接收。所以**超类型限定通配符也是"可读"的，只是返回值类型有限制**。

> 超类型限定通配符和子类型限定通配符的读写特性，体现了泛型的安全设计，其实就是向上类型转换和向下强转的安全问题。

#### 2.3 应用示例

超类型限定通配符的一个常用方式：让父类的比较方法可以应用于子类对象。

> 工具类：用于查找arr中的最大值

```java
public class ArrayUtil {
    public static <T extends Comparable<T>> T max(T[] arr) {
        if (arr == null || arr.length == 0) return null;

        T max = arr[0];
        for (int i = 1; i < arr.length; i++) {
            if (max.compareTo(arr[i]) < 0)
                max = arr[i];
        }
        return max;
    }
}
```

> Parent实现了Comparable<Parent>，而Child继承了Parent，也就间接实现了Comparable<Parent>，而非Comparable<Child>。

```java
// 父类
public class Parent implements Comparable<Parent> {
    private Integer score;
    // 有参构造
    public Parent(Integer score) {this.score = score;}

    public Integer getScore() {return score;}

    @Override
    public int compareTo(Parent o) {
        return this.score.compareTo(o.getScore());
    }
}

// 子类
public class Child extends Parent {
    // 有参构造
    public Child(Integer score) {super(score);}
}
```

> 测试代码：有编译报错，Incompatible types

```java
// 测试
public static void main(String[] args) {
    Parent p1 = new Parent(1);
    Parent p2 = new Parent(2);
    Parent p3 = new Parent(3);
    Parent maxParent = ArrayUtil.max(new Parent[]{p1, p2, p3});
    System.out.println(maxParent.getScore());

    Child c1 = new Child(4);
    Child c2 = new Child(5);
    Child c3 = new Child(6);
    Child maxChild = ArrayUtil.max(new Child[]{c1, c2, c3});    // 编译报错
    System.out.println(maxChild.getScore());
}
```

> 原因

类型不匹配。因为通过泛型方法max()接收实参Child[]后，编译器推断出T为Child类型，而该方法声明时要求T extends Comparable<T>。Child并没有实现Comparable<Child>，它实现的是Comparable<Parent>，所以编译报错。

> 解决

此时就可以通过超类型限定来解决。修改泛型方法对T的约束：

```java
public static <T extends Comparable<? super T>> T max(T[] arr) {
    ...
}
```

这样，虽然Child没有实现Comparable<Child>，而是间接实现Comparable<Parent>，也能编译通过。子类不需要重复实现Comparable<Child>接口，重复重写compareTo()，使用父类的即可。

## 3.无限定通配符

不对通配符做上限（子类限定）或下限（超类限定）。

#### 3.1 示例

```java
// 无限定通配符
public boolean equal(Box<?> other) {    // 非泛型方法
    //? data = other.getData();         // 编译报错
    return data.equals(other.getData());
}

// test code
public static void main(String[] args) {
    Box<Number> numBox = new Box<>(1);
    Box<Integer> intBox = new Box<>(2);
    System.out.println(numBox.equal(intBox));// 编译不报错
    System.out.println(intBox.equal(numBox));// 编译不报错
}
```

这种通配符使用方式，不能使用?作为一种类型。如果想使用泛型声明变量，可以转换成泛型方法，避免使用通配符：

```java
public <T> boolean equal(Box<T> other) { // 泛型方法
    T data = other.getData();
    ...
    return this.data.equals(other.getData());
}
```

> Java容器类中有类似这样的用法。

#### 3.1 无限定通配符的读写特性

无限定通配符是只读的。

```java
Box<Number> numBox = new Box<>(10);
Box<?> superBox = numBox;   
superBox.setData(11);   // 编译报错
Object superBoxData = superBox.getData();
```

> superBox和numBox指定同一内存空间，分别是Box<?>和Box<Number>类型的，如果允许superBox.setData()，可能存入非Number类型的数据，所以为了安全，禁止Box<?>类型引用的写。

## 4.通配符总结

- <?>和<? extends E>用于实现更为灵活的读取，它们可以用类型参数的形式替代，但通配符更简洁
- <? super E>用于实现更为灵活的写入和比较，不能被类型参数形式替代


---
# V.泛型和虚拟机

**泛型只是对于Java编译器的概念，compiler会把泛型代码转换为普通的非泛型代码**。

## 1.泛型擦除

对于JVM而言，所有类都是普通类，JVM感知不到类型变量的存在。所有的泛型类，对于JVM来讲都有一个相应的原始类型，原始类型相当于，擦除类型参数并替换为限定类型（无限定的类型变量替换为Object）之后的形式。

比如泛型类Box<T>、Box<T extends Comparable<T>>的原始类型如下：

```java
// Box<T>
public class Box<T> {
    private T data;

    public Box() {}
    public Box(T data) {this.data = data;}

    public T getData() {return data;}
    public void setData(T data) {this.data = data;}
}

// Box<T>的原始类型Box。把T都换成Object，擦除后就是一个普通类
public class Box {
    private Object data;

    public Box() {}
    public Box(Object data) {this.data = data;}

    public Object getData() {return data;}
    public void setData(Object data) {this.data = data;}
}
```

```java
// Box<T extends Comparable<T>>
public class Box<T extends Comparable<T>> {
    private T data;

    public Box() {}
    public Box(T data) {this.data = data;}

    public T getData() {return data;}
    public void setData(T data) {this.data = data;}
}

// Box<T extends Comparable<T>>的原始类型Box
public class Box {
    private Comparable data;

    public Box() {}
    public Box(Comparable data) {this.data = data;}

    public Comparable getData() {return data;}
    public void setData(Comparable data) {this.data = data;}
}
```

例如Box<T extends Comparable<T>>，JVM中加载的肯定是Comparable类型的，这就需要compiler在编译期做很多替换。

## 2.泛型底层的强转

在调用返回值类型是类型变量时，如上述Box<T>中的T getData()：

```java
// 擦除前
public T getData() {
    return data;
}
// 擦除后
public Object getData() {
    return data;
}

// 调用
Box<String> box = new Box<>();
String data = box.getData();
```

由于存在泛型擦除机制，当调用getData()时，由于返回类型是类型变量T，JVM只能给出被擦除后的Object类型的结果，而调用处却可以使用String data接收。之所以能这么赋值，是因为对于泛型类中的方法，如果返回擦除后的类型，编译器还会插入强转逻辑（调用处的强转）。所以box.getData()调用时，其实执行了对应两条虚拟机指令：
- 1.调用擦除后的原始方法getData()
- 2.将返回的Object类型的结果强转为String类型

## 3.类型变量擦除与多态

假设存在下列多态情形：

```java
// Box<T>
public class Box<T> {
    private T data;

    public Box() {}
    public Box(T data) {this.data = data;}

    public T getData() {return data;}
    public void setData(T data) {this.data = data;}
}
// Box<T>擦除后，对于JVM，Box<Date>是这样的
public class Box {
    private Object data;

    public Box() {}
    public Box(Object data) {this.data = data;}

    public Object getData() {return data;}
    public void setData(Object data) {this.data = data;}
}
// DateBox
public class DateBox extends Box<Date> {
    @Override
    public void setData(Date data) {
        super.setData(data);
    }
}
// test
public class BoxTest {
    public static void main(String[] args) {
        Box<Date> box = new DateBox();
        box.setData(new Date()); // 这里IDEA只提示了setData(Date data)，没有提示setData(T data)
    }
}
```

考虑泛型擦除，DateBox在继承中重写的setData(Date data)，和父类Box<Date>的setData(Object data)理应是"重载"的关系，而非重写。但在编写测试代码时，IDEA确实只提示了setData(Date data)，没有提示setData(T data)，说明确实是重写。从"擦除后的重载关系"到"最终的重写关系"，肯定存在某种特殊处理，即**Java编译器提供的桥方法**：

```java
public void setData(Object data) {
    setData((Date) data);
}
```

> 编译器为子类生成的桥方法，重写了父类的setData(Object data)。

当执行box.setData(new Date())时，由于子类的setData((Date) data)只是重载，所以执行的还是setData(Object second)，根据多态特性，执行的是子类中重写的桥方法setData(Object second)（而非父类的setData(Object second)），在桥方法中又调用了子类特有的setSecond(Date date)，达到最终执行子类重写方法的多态效果。所以，**泛型擦除有可能会与多态冲突，此时Java编译器会自动生成一个桥方法，实现多态效果**

> 下面通过反编译class文件，来证实桥方法的存在。

```java
// Box<T>
public class Box<T> {
    private T data;

    public Box() {}
    public Box(T data) {this.data = data;}

    public T getData() {return data;}
    public void setData(T data) {this.data = data;}
}
// 子类DateBox
public class DateBox extends Box<Date> {
    @Override
    public void setData(Date data) {
        if (data.before(new Date())) {
            super.setData(data);
        }
    }

    @Override
    public Date getData() {
        return super.getData();
    }
}
```

在这两个java文件所在目录下执行:

```shell
$ javac *.java
$ javap -l DateBox.class 
Compiled from "DateBox.java"
public class com.baicai.generic.DateBox extends com.baicai.generic.Box<java.util.Date> {
  public com.baicai.generic.DateBox();
    LineNumberTable:
      line 9: 0

  public void setData(java.util.Date);
    LineNumberTable:
      line 12: 0
      line 13: 14
      line 15: 19

  public java.util.Date getData();
    LineNumberTable:
      line 19: 0

  public void setData(java.lang.Object);
    LineNumberTable:
      line 9: 0

  public java.lang.Object getData();
    LineNumberTable:
      line 9: 0
}
```

可以看到class文件中有两个setSecond()，其中一个就是编译器生成的桥方法。另外，对于getSecond()，编译器也生成了一个桥方法，但导致了一个类中有两个方法除返回值类型外都一样。在编码时，是不允许这种情况出现的，会编译报错。但在JVM中，用参数类型和返回值类型确定一个方法，因此编译器可能产生两个仅返回值类型不同的方法字节码，JVM能够正确的处理这种情况。

```java
public void setData(java.util.Date);
public void setData(java.lang.Object);
public Date getSecond();
public Object getSecond();
```

## 3.泛型和虚拟机小节

- Java虚拟机中没有泛型，只有普通类和方法
- 泛型擦除时，所有的类型参数，都用它们的限定类型替换（没有限定类型时替换为Object）
- Java编译器会生成桥方法用于保持多态的特性
- 使用泛型，并不需要显示地强制类型转换，因为这一动作，由Java编译器实现


---
# VI.泛型的使用问题

参考[Java编程拾遗『泛型——基本概念』](http://lidol.top/java/809/)。


---
# VII.参考

- [The Java™ Tutorials generics](https://docs.oracle.com/javase/tutorial/java/generics/index.html)
- [Java编程拾遗『泛型——基本概念』](http://lidol.top/java/809/)