---
title: Java运算符
date: 2018-01-03 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
    - Operator
    - Java
---

本文记录Java运算符的基本知识。

<!-- more -->

##### 目录
+ I.运算符
+ II.变量
+ III.数据类型
+ IV.转义字符
+ V.值传递

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

> &, |, !, ^, &&, ||

- 逻辑运算符操作的是布尔值
- 异或^，不同时返回true
- 其中&&、||有短路效果

## 5.位运算符

> 位与&，位或|，位取反~，位异或^，位左移<<，无符号右移>>，有符号右移>>

- **按补码运算**(先得到补码，再位运算，根据运算后的补码再去得到原码)
- 左移<<：是右边补0，
- 无符号右移>>>：左边补0
- 有符号右移>>：最高位是0/1，左边就补0/1
- 一个数异或另一个数两次，结果还是该数
- **最有效的算乘/除2-使用位运算**

> - TODO 还不是很明白
> - 1.int数4字节32位，是按x << N%32来运算
> - 2.long数8字节64位，是按x << N%64来运算的

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


## 7.键盘录入

```
public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    System.out.println("请输入数据:");
    int a = sc.nextInt();
    System.out.println(a);
}
```

