---
title: Java数组
date: 2018-01-05 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Java数组的基本知识。

<!-- more -->

##### 目录
+ I.一维数组
+ II.二维数组

---

> 数组

- **为了存储同种数据类型多个值**，可看做一个容器，弥补了变量的不足
- 既可以存基本数据类型，也可以存引用数据类型
- 一维数组、二维数组
- 数组本身属于引用数据类型

# I.一维数组

## 1.格式

> 数组名要求为合法标识符

- 格式1：动态初始化

> 数据类型[] 数组名 = new 数据类型[长度]

```
int[] arr = new int[5];
```

- 格式2：静态初始化

> 数据类型[] 数组名 = new 数据类型[]{元素1,元素2...};

```
//可以
int[] arr = new int[] {1,2,3};
//声明和赋值可以分开
int[] arr;
arr = new int[] {1,2,3};
//不可以动静结合
int[] arr = new int[3] {1,2,3};
```

- 格式3：静态初始化-简化方式
数据类型[] 数组名 = {元素1,元素2...};
```
//可以
int[] arr = {1,2,3};
//必须声明和赋值同时进行,不可以拆开
int[] arr;
arr = {1,2,3};
```

## 2.初始化

> 数组初始化：为数组开辟连续的JVM（堆）内存空间，并赋予初始值。分为动态初始化、静态初始化
   
+ 动态初始化：只指定长度，系统赋初始值，数据类型[] 数组名 = new 数据类型[长度]
+ 静态初始化：给定初始化值，由系统决定长度

- 关于初始化值

|数组类型|默认初始化值|
|:------|:--------|
|整数类型(byte、short、int、long)|0|
|浮点型(float、double)|0.0|
|布尔型boolean|false|
|字符型char|'\u0000'|

> 其中，char类型占两字节16个比特位，\u0000中的0是16进制的0，每个0代表了4个0(8421)，即共16个0.表示了char在内存中占的16位

#### 2.1 动态初始化

```
int[] arr = new int[5]
syso(arr);//[I@19bb25a
syso(arr[0]);//0
```

> - 会在堆内存中分配连续的5块空间，并赋初始化值0
> - 将arr指向整个数组的内存地址，再根据数组索引找到具体的元素
> - **[I@19bb25a中，[表示一维数组，I表示int数组类型，@固定符号，后面是十六进制的地址值**

#### 2.2 静态初始化

```
int[] arr = new int[] {1,2,3}
syso(arr);//[I@19bb25a
syso(arr[0]);//0
```

> - 会在堆内存中分配连续的3块空间，并赋初始化值0
> - 进行显示初始化，将arr[0]=1,arr[1]=2,arr[2]=3
> - 将arr指向整个数组的内存地址，再根据数组索引找到具体的元素
> - **[I@19bb25a的含义同上

## 3.常见问题

#### 3.1 数组索引越界

> ArrayIndexOutOfBoundsException

```
//长度为5，索引为0-4
int[] arr = new int[5];
System.out.println(arr[-1]);
System.out.println(arr[5]);
```

#### 3.2 空指针异常

> NullPointerException

```
//长度为5，索引为0-4
int[] arr = null;
System.out.println(arr[0]);
```

## 4.数组的遍历

#### 4.1 数组遍历

```
int[] arr = {1,2,3};
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}
```

```
int[] arr = {1,2,3};
for (int x : arr) {
    System.out.println(x);
}
```

#### 4.2 数组最值

```
int[] arr = {1,2,3};
int max = arr[0];
for (int i = 0; i < arr.length; i++) {
    if(max < arr[i])
        max = arr[i];
}
```

#### 4.3 数组反转

```
int[] arr = {1,2,3};
for (int i = 0; i < arr.length / 2; i++) {
    int temp = arr[i];
    arr[i] = arr[arr.length - 1 - i];
    arr[arr.length - 1 - i] = temp;
}
```

# II.二维数组

## 1.格式

- 1.第一种

```
//该二维数组，包含3个一维数组，每个一维数组有2个元素
int[][] arr1 = new int[3][2];
int[] arr2[] = new int[3][2];
int arr3[][] = new int[3][2];
```

```
//声明一个一维数组x
int[] x;
//声明一个二维数组y
int[] y[];
//声明一个一维数组x，一个二维数组y
int[] x, y[];
```

```
int[][] arr = new int[3][2];
//二维数组地址值
System.out.println(arr);//[[I@7852e922
//第一个一维数组地址值
System.out.println(arr[0]);//[I@4e25154f
//第一个一维数组的第一个值
System.out.println(arr[0][0]);//0
```

> - 方法进栈，在栈内存创建int[][] arr变量
> - 在堆内存分配连续的三块空间，并初始化赋值为null（一维数组的地址值，初始化值为null）
> - 在堆内存创建三个一维数组，并赋初始化值0
> - 把每个一维数组的地址值，赋值给二维数组中的arr[0],arr[1],arr[2]

- 2.第二种

```
int[][] arr = new int[3][];
arr[0] = new int[2];
arr[1] = new int[2];
System.out.println(arr[0]);//[I@7852e922
System.out.println(arr[1]);//[I@4e25154f
System.out.println(arr[2]);//null
System.out.println(arr[0][0]);//0
//System.out.println(arr[2][0]);//报空指针
```

> - 该格式只定义了二维数组的长度，即一维数组的个数，此时arr[0]，arr[1]，arr[2]内的初始值为null

- 3.第三种

>相当于静态初始化

```
int[][] arr = {\{1,2,3},{4,5},{6,7,8}\};
System.out.println(arr[0]);//[I@7852e922
System.out.println(arr[1]);//[I@4e25154f
System.out.println(arr[2]);//[I@70dea4e
System.out.println(arr[0].length);//3
System.out.println(arr[0][1]);//2
```

## 2.二维数组遍历

```
int[][] arr = {{1,2,3},{4,5},{6,7,8}};
for (int i = 0; i < arr.length; i++) {
    for (int j = 0; j < arr[i].length; j++) {
        System.out.print(arr[i][j]);
    }
    System.out.println();
}
```