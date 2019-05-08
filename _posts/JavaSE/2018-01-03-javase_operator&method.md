---
title: Java运算符和方法
date: 2018-01-03 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Java运算符和方法的基本知识。

<!-- more -->

##### 目录
+ I.运算符
+ II.方法

---

# I.运算符

对常量和变量进行操作的符号。包括算数运算符、赋值运算符、比较/关系/条件运算符、逻辑运算符、位运算符、三目/元运算符

## 1.算数运算符

+，-，*，/，%，++，\-\-

- 1.Java中的+有三种作用：正号、加法、字符串连接符
- 2.整数/整数只能得到整数，若想得到小数，需要把其中一个数变成浮点型

```java
System.out.println(10 / 3);     //3
System.out.println(10 / 3.0);   //3.3333333333
```

- 3./得到的是商，%得到的是余数

```java
# 余数的正负，取决于分子
System.out.println(12 % 5);     //2
System.out.println(-12 % 5);    //-2
System.out.println(12 % -5);    //2
System.out.println(-12 % -5);   //-2
System.out.println(4 % 5);      //4
System.out.println(-4 % 5);     //-4
System.out.println(4 % -5);     //4
System.out.println(-4 % -5);    //-4
System.out.println(0 % 5);      //0
```

- 4.++和\-\-运算符单独使用时，放在操作数的前后效果一样。如果参与其他运算，放前面：先++/\-\-，再参与其他运算，放后面：先参与其他运算，再++/\-\-

```java
int x = 4;
int y = (x++) + (++x) + (x * 10)
syso(x);    //6
syso(y);    //70
```

- 5.++和\-\-底层会有强转

```java
byte b = 10;    //可以
b++;            //可以，**不止是b = b + 1，而是b = (byte)(b + 1)**
b = b + 1;      //不可以，b+1结果为int，需要强转
```

## 2.赋值运算符

=，+=，-=，*=，/=，%=

> 扩展的赋值运算符+=，-=，*=，/=，%=，和++/\-\-一样，底层有强转

```java
short s = 1;
s = s + 1;      //不可以，需要强转
s += 1;         //可以，底层有强转
```

## 3.关系运算符(比较/条件运算符)

==, !=, >, >=, <, <=，关系运算符的结果都是boolean

## 4.逻辑运算符

&，\|，!，^，&&,，\|\|

- 逻辑运算符操作的是布尔值
- 异或^，不同时返回true
- 其中&&、\|\|有短路效果

## 5.位运算符

位与&，位或\|，位取反~，位异或^，位左移<<，无符号右移>>>，有符号右移>>

- **按补码运算**(先得到补码，再位运算，根据运算后的补码再去得到原码)
- 左移<<：是右边补0，
- 无符号右移>>>：左边补0
- 有符号右移>>：最高位是0/1，左边就补0/1
- **一个数异或另一个数2次，结果还是该数**
- **最有效的算出乘或除2的运算：位运算**

> - int数4字节32位，是按x << N%32来运算
> - long数8字节64位，是按x << N%64来运算的

```java
System.out.println(1<<2);   //4
System.out.println(1<<30);  //1073741824
System.out.println(1<<31);  //-2147483648（int最小值、-0）
System.out.println(1<<32);  //1(N%32)
System.out.println(1<<64);  //1

System.out.println(-1<<2);  //-4
System.out.println(-1<<31); //-2147483648
System.out.println(-1<<32); //-1
System.out.println(-1<<33); //-2
System.out.println(-1<<64); //-1

System.out.println(8L>>>3); //1
System.out.println(8L>>>4); //0
System.out.println(8L>>>32);//0
System.out.println(8L>>>50);//0
System.out.println(8L>>>64);//8

System.out.println(-8>>>2); //1073741822
System.out.println(-8>>2);  //-2
System.out.println(8>>2);   //2

System.out.println(8^2^2);  //8
```

## 6.三元表达式

比if的效率高。因为后面要执行的语句已排好队列，而if还要根据逻辑结果可能需要重新编排语句的执行顺序


## 7.变量值交换的三种方式

- 1.第三方变量(安全，推荐)

```java
int x = 10,y = 20;
int temp;
temp = x;
x = y;
y = temp;
```
    
- 2.位异或

```java
x = x ^ y;
y = x ^ y;
x = x ^ x;
```

- 3.加法(可能会超出范围，不推荐)

```java
x = x + y;
y = x - y;
x = x - y;
```


---

# II.方法

完成特定功能的代码块，提高代码复用性

## 1.基本内容

#### 1.1 格式

```
修饰符 返回值类型 方法名(参数类型1 参数名2, ..) {
    方法体;
    return 返回值;
}
```

#### 1.2 说明

+ 返回值类型：void、基本或引用数据类型
+ 方法名：合法标识符
+ 参数：
    * 实参-实际参与运算的
    * 形参-方法上定义，用于接收实参
+ 方法不能嵌套
+ 调用方法时只传值，不能加数据类型，会报错
+ 如果返回值不是void，就必须要有return带回返回值


## 2.形参

**基本数据类型当做形参，传递的是值；引用数据类型当做形参，传递的是地址值**

```java
public static void main(String[] args) {
    int i =1;
    print(i);               //打印2
    System.out.println(i);  //打印1（不改变原值）
}

private static void print(int i) {
    System.out.println(++i);
}
```

```java
public static void main(String[] args) {
    Student s1 = new Student();
    s1.age = 18;
    s1.sex = "男";
    //s1和s是两个引用指向同一对象
    print(s1);
}

private static void print(Student s) {
    s.age = 188;            //改变原值
    System.out.println(s.age + ".." + s.sex);
}
```

## 3.overload

重载，**方法名相同，参数列表不同，与返回值类型无关**（其中参数列表满足以下一个或多个即可）

- 参数类型不同
- 参数个数不同
- 参数顺序不同

```java
public void add(int a) {
    System.out.println(a + 1);
}
public void add(double a) {
    System.out.println(a + 1);
}
public double add(int a, double b) {
    System.out.println(a + b);
    return (a + b);
}
public double add(double b, int a) {
    System.out.println(a + b);
    return (a + b);
}
```

## 4.overwrite

重写，**子父类的方法一模一样（返回值类型可以是子父类关系），多用于继承或接口实现中的方法覆写增强**

- 子类重写，最好声明与父类完全一样
- **父类的私有方法，不能被重写（也不能继承/访问）**
- 子类重写父类方法时，访问权限不能更低，最好一致(因为访问权限变低，有悖于功能增强的原则)(public > protected > default > private)

```java
class Father {
    public void print() {
        System.out.println("Father...print");
    }
}

class Son extends Father {
    //报错：不写默认是default，public > protected > default > private
    void print() {
        super.print();
        System.out.println("Son...print");
    }
}
```

- 静态方法不算真正的重写（静态只能覆盖静态）

## 5.overload和overwrite对比

- overload：方法名相同，参数列表不同，返回值类型可以相同也可以不同（返回值类型无关）
- overwrite：方法名相同，参数列表相同，返回值类型相同或是子父类

## 6.main方法

```java
public class HelloWorld {
    public static void main(String[] args) {
    }
}
```

- public：被jvm调用，所以权限要足够大，必须是public
- static：被jvm调用时，不需要创建HelloWorld对象，直接类名调用即可
- void：被jvm调用，不需要有任何的返回值
- 方法名main：固定
- String[] args：以前是用来接收键盘录入的（现在用Scanner代替）

```java
public class HelloWorld {
    public static void main(String[] args) {
        for (int i = 0; i < args.length; i++) {
            System.out.println(args[i]);
        }
    }
}

//main方法的键盘录入：在控制台执行以下命令
java HelloWorld haha xixi wowo
//输出
haha
xixi
wowo
```

## 7.构造方法

构造方法是一种特殊的方法，在对象实例化时，用来初始化对象（的成员变量）。具有以下特点：

- 构造方法名必须与类名相同，并且没有返回值（void也不行）
- 每个类可以有多个构造。当代码中没有提供构造时，编译器在编译成字节码时，会提供一个默认的空参构造，如果代码中提供了，编译器不再提供。
- 构造总伴随着new操作一起调用，不能由开发人员直接调用，必须要由系统调用。构造在实例化时被自动调用，且只运行一次
- 构造函数不能被继承，也就不能被重写，但是可以重载
- 子类可以通过super显式地调用父类的构造，当父类没有提供无参构造时，子类的构造必须显式地调用父类的有参构造。如果父类提供了无参构造，子类的构造可以不用显式地调用父类的构造，此时编译器默认调用父类提供的无参构造
- 当有父类时，在实例化对象前，会先执行父类的构造，然后才执行子类构造
- 如果父类和子类中都没有定义构造，编译器会为父类和子类都生成一个默认的构造函数
- 另外，构造的修饰符只跟当前类的修饰符有关，如果一个类定义为public，那么它的构造也是public的

## 8.静态和非静态方法

静态方法，用于描述类行为，可以直接使用类名调用。在同一个类中，静态方法中不能访问非静态方法和非静态成员变量。关于静态方法和非静态方法的一些解释：

- a.静态方法常驻内存，实例方法不是，所以静态方法效率高但占内存?

> 效率上，他们是一样的。在调用和占用内存上，静态方法和实例方法是一样的，被调用时加载到栈中，调用结束后弹栈，执行效率上基本没差。

- b.静态方法在堆上分配内存，实例方法在堆栈上?

> 所有方法都不会在堆或栈上分配内存，方法作为代码，会被加载到特殊的代码内存区域，这个内存区域是不可写的。方法占用的内存大小，跟是不是static也没关系。对于成员变量，每个实例的成员变量值不同，所以每个实例的所有字段都会在内存中有一分拷贝。和成员变量不同，不论有多少个实例，它们的方法代码都是一样的，所以只要要一份代码。因此无论是static还是非static的方法，都只存在一份代码，即只占用一份内存空间。  

- c.同一个方法，为什么运行起来表现却不一样？

> 这依赖于方法所访问的数据。包括：方法实参、该类的成员变量。

- d.实例方法需要先创建实例才可以调用，比较麻烦，静态方法不用，比较简单，都创建成static的？

> 类名调用确实简介，但方法是否定义成static，在于它的逻辑和实例是否相关。如果一个方法与实例无关，就应该定义成静态的。如果和每个实例有关，就要定义成实例方法。既然与实例有关，必然需要先创建实例，就没有麻不麻烦一说了。从面向对象的角度看，在抉择使用实例方法或静态方法时，应该根据该方法是否和实例具有逻辑上的相关性。当然了，如果从线程安全、性能、兼容性上来看 还是选用实例方法为宜。

- e.为什么要区分静态方法和实例方法？

早期的结构化编程，几乎所有的方法都是"静态方法"，引入实例化方法概念是在面向对象概念出现以后。区分静态方法和实例方法不是为了要解决性能或内存等问题，而是在概念上让开发更加模式化、面向对象化，即是为了解决模式的问题。

## 9.Scanner键盘录入

```java
public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    System.out.println("请输入数据:");
    int a = sc.nextInt();
    System.out.println(a);
}
```