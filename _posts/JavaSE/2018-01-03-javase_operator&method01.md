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

> 对常量和变量进行操作的符号。包括算数运算符、赋值运算符、比较/关系/条件运算符、逻辑运算符、位运算符、三目/元运算符

## 1.算数运算符

> +, -, *, /, %, ++, --

- 1.Java中的+有三种作用：正号、加法、字符串连接符
- 2.整数/整数只能得到整数，若想得到小数，须把其中一个数变成浮点型

```
System.out.println(10 / 3);  //3
System.out.println(10 / 3.0);//3.3333333333
```

- 3./得到的是商，%得到的是余数

```
# 余数的正负，取决于分子
System.out.println(12 % 5); //2
System.out.println(-12 % 5);//-2
System.out.println(12 % -5);//2
System.out.println(-12 % -5);//-2
System.out.println(4 % 5);  //4
System.out.println(-4 % 5); //-4
System.out.println(4 % -5); //4
System.out.println(-4 % -5);//-4
System.out.println(0 % 5);  //0
```

- 4.++和--运算符单独使用时，放在操作数的前后效果一样。如果参与其他运算，放前面：先++/--，再参与其他运算，放后面：先参与其他运算，再++/--

```
int x = 4;
int y = (x++) + (++x) + (x * 10)
syso(x);//6
syso(y);//70
```

- 5.++和--底层会有强转

```
byte b = 10;//可以
b++;        //可以，**不止是b = b + 1，而是b = (byte)(b + 1)**
b = b + 1;  //不可以，b+1结果为int，需要强转
```

## 2.赋值运算符

> =, +=, -=, *=, /=, %=

- 扩展赋值运算符+=, -=, *=, /=, %=，和++/--一样，底层有强转

```
short s = 1;
s = s + 1;  //不可以，需要强转
s += 1;     //可以，底层有强转
```

## 3.关系运算符(比较/条件运算符)

> ==, !=, >, >=, <, <=，关系运算符的结果都是boolean

## 4.逻辑运算符

> &, \|, !, ^, &&, \|\|

- 逻辑运算符操作的是布尔值
- 异或^，不同时返回true
- 其中&&、\|\|有短路效果

## 5.位运算符

> 位与&，位或\|，位取反~，位异或^，位左移<<，无符号右移>>，有符号右移>>

- **按补码运算**(先得到补码，再位运算，根据运算后的补码再去得到原码)
- 左移<<：是右边补0，
- 无符号右移>>>：左边补0
- 有符号右移>>：最高位是0/1，左边就补0/1
- 一个数异或另一个数两次，结果还是该数
- **最有效的算出乘或除2的运算：位运算**

> - int数4字节32位，是按x << N%32来运算
> - long数8字节64位，是按x << N%64来运算的

```
System.out.println(1<<2);//4
System.out.println(1<<30);//1073741824
System.out.println(1<<31);//-2147483648（int最小值、-0）
System.out.println(1<<32);//1(N%32)
System.out.println(1<<64);//1

System.out.println(-1<<2);//-4
System.out.println(-1<<31);//-2147483648
System.out.println(-1<<32);//-1
System.out.println(-1<<33);//-2
System.out.println(-1<<64);//-1

System.out.println(8L>>>3);//1
System.out.println(8L>>>4);//0
System.out.println(8L>>>32);//0
System.out.println(8L>>>50);//0
System.out.println(8L>>>64);//8

System.out.println(-8>>>2);//1073741822
System.out.println(-8>>2);//-2
System.out.println(8>>2);//2

System.out.println(8^2^2);//8
```

> 变量值交换的三种方式

- 1.第三方变量(安全，推荐)

```
int x = 10,y = 20;
int temp;
temp = x;
x = y;
y = temp;
```
    
- 2.位异或

```
x = x ^ y;
y = x ^ y;
x = x ^ x;
```

- 3.加法(可能会超出范围，不推荐)

```
x = x + y;
y = x - y;
x = x - y;
```

## 6.三元表达式

> 比if的效率好。因为后面要执行的语句已排好队列，而if还要根据逻辑结果可能需要重新编排语句的执行顺序


# II.方法

> 完成特定功能的代码块，用于提高代码的复用性

#### 1.基本内容

> 1.1 格式

```
修饰符 返回值类型 方法名(参数类型1 参数名2, ..) {
    方法体;
    return 返回值;
}
```

> 1.2 说明

+ 返回值类型：void、基本数据类型、引用数据类型
+ 方法名：合法标识符
+ 参数：
    * 实参-实际参与运算的
    * 形参-方法中定义的，用于接收实参
+ 方法不能嵌套
+ 调用方法时只传值，不能加数据类型，会报错
+ 如果返回值不是void，就必须要有return带回返回值


#### 2.形参

> **基本数据类型当做形参，传递的是值；引用数据类型当做形参，传递的是地址值**

```
public static void main(String[] args) {
    int i =1;
    print(i);               //打印2
    System.out.println(i);  //打印1（不改变原值）
}

private static void print(int i) {
    System.out.println(++i);
}
```

```
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

#### 3.overload

> 重载：**方法名相同，参数列表不同，与返回值类型无关**（其中参数列表满足以下一个或多个即可）

- 参数类型不同
- 参数个数不同
- 参数顺序不同

```
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

#### 4.overwrite

> 重写：**子父类的方法一模一样（返回值类型可以是子父类关系），多用于继承或接口实现中的方法覆写或增强**

- 子类重写，最好声明一模一样
- **父类的私有方法，不能被重写（也不能继承/访问）**
- 子类重写父类方法时，访问权限不能更低，最好一致(因为访问权限变低，有悖于功能增强的原则)(public > protected > default > private)

```
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

#### 5.overload和overwrite的区别

> - overload：方法名相同，参数列表不同，返回值类型可以相同也可以不同（返回值类型无关）
> - overwrite：方法名相同，参数列表相同，返回值类型相同或是子父类

#### 6.main方法

```
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

```
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

#### 7.Scanner键盘录入

```
public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    System.out.println("请输入数据:");
    int a = sc.nextInt();
    System.out.println(a);
}
```