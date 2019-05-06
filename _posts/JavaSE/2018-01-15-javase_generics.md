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
- [X] IV.泛型代码和虚拟机
- [X] V.泛型通配符


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

首先，一个方法是不是泛型方法，跟它所处的类是不是泛型类没有关系。普通类中也可以定义泛型方法，只不过泛型类中更常见。

> 格式

```java
修饰符 <类型参数列表> 返回类型 方法名(形参列表) { 方法体 }
```

> 泛型方法声明时，类型变量要放在修饰符后、返回值前。示例：

```java
public static <T> T getFirst(T[] arr) {
    if (array == null || array.length == 0)
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

# III.类型变量的限定

泛型变量默认是Object类型。如果泛型变量要使用Object没有的方法，那么泛到Object肯定是不行的。

> 如上述的Box类，如果在其方法内调用compareTo()，肯定是不行的。

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

> 示例

假设现在Box类如下，类中提供了hasEqual()，对两个Box<T>实例的data进行比较。

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
        System.out.println(numBox.equalData(intBox));// 编译报错
    }
}
```

编译报错如下，不能将Box<Integer>对象赋值给Box<Number>：

```sql
equalData(com.baicai.generic.Box<java.lang.Number>) in Box cannot be applied to (com.baicai.generic.Box<java.lang.Integer>)
```

原因：使用numBox调用equalData()时，具体调用的是equalData(Box<Number> other)，故传参编译报错。**虽然Integer是Number的子类，但是Box<Integer>并不是Box<Number>的子类**，也就不能把Pair<Integer>对象赋值给Pair<Number>类型的引用other，此时就需要将上界定义成类型变量。

> 解决

```java
// 修改equalData()，使用其他类型变量E，T是E的上界
public class Box<T> {
    ...
    public <E extends T> boolean equalData(Box<E> other) {
        return data.equals(other.getData());
    }
}
```

此时再测试，输出true。


---
# IV.泛型代码和虚拟机

**泛型只是对于Java编译器的概念，compiler会把泛型代码转换为普通的非泛型代码**。

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

JVM中加载的肯定是Comparable类型的，这就需要compiler在编译期做很多强转,如Comparable tmp = (Comparable)box.getData()。

## 1.泛型表达式

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

由于存在泛型擦除机制，当调用getData()时，由于返回类型是类型变量T，JVM只能给出被擦除后的Object类型的结果，而调用处可以使用String data接收。之所以能这么赋值，是因为对于泛型类中的方法，如果返回擦除后的类型，编译器还会插入强转逻辑。所以box.getData()调用时，其实执行了对应两条虚拟机指令：
- 1.调用擦除后的原始方法getData()
- 2.将返回的Object类型的结果强转为String类型

## 2.类型擦除与多态(运行时)

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
        box.setData(new Date()); // 这里IDEA只提示setData(Date data)，没有提示setData(T data)
    }
}
```

考虑泛型擦除，DateBox在继承中重写的setData(Date data)，和父类Box<Date>的setData(Object data)理应是"重载"的关系，而非重写。但在编写测试代码时，IDEA确实只提示了setData(Date data)，没有提示setData(T data)，说明确实是重写。从"擦除后的重载关系"到"最终的重写关系"，肯定存在特殊处理：**Java编译器提供的桥方法**。

```java
public void setData(Object data) {
    setData((Date) data);
}
```

当执行box.setData(new Date())时，根据多态的特性，会执行子类的setData(Object second)（继承父类Box<Date>的），在该方法中又调用了子类特有的setSecond(Date date)，达到执行子类重写方法的多态效果。所以，**泛型擦除有可能会与多态冲突，此时Java编译器会自动生成一个桥方法，实现多态**

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

在这两个java文件下执行:

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

可以看到class文件中有两个setSecond()，其中一个就是编译器生成的桥方法。另外，对于getSecond()，编译器也生成了一个桥方法，但导致了一个类中有两个方法，除返回值类型外都一样。在编码时，是不允许这种情况出现的，会编译报错。但在JVM中，用参数类型和返回值类型确定一个方法，因此编译器可能产生两个仅返回值类型不同的方法字节码，但是JVM能够正确的处理这种情况。

```java
public void setData(java.util.Date);
public void setData(java.lang.Object);
public Date getSecond();
public Object getSecond();
```

## 3.泛型和虚拟机小节

- Java虚拟机中没有泛型，只有普通类和方法
- 擦除时，所有的类型参数，都用它们的限定类型替换（没有限定类型时替换为Object）
- Java编译器会生成桥方法用于保持多态的特性
- 使用泛型，并不需要显示地强制类型转换，因为这一动作，由Java编译器实现


---
# 参考

- [The Java™ Tutorials generics](https://docs.oracle.com/javase/tutorial/java/generics/index.html)
- [Java编程拾遗『泛型——基本概念』](http://lidol.top/java/809/)