---
title: Java流程控制语句
date: 2018-01-04 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Java流程控制语句的基本知识。

<!-- more -->

##### 目录
+ I.顺序结构
+ II.选择结构
+ III.循环结构
+ IV.跳转控制

---

流程控制语句：顺序结构语句、选择结构语句、循环结构语句

# I.顺序结构

从上往下，依次执行


---

# II.选择结构

## 1.if语句

- if语句可以嵌套
- 条件表达式的结果必须是boolean型
- 如果语句体只有一条语句，可以省略{}，但一般都不省
- 三元运算符能实现的，if都能实现，反之不一定
- 最后一个else可以省略，但不建议省略，可以做范围外的错误提示（e.g.抛异常）

> **int x = 1;不是一条语句，是两条**

```java
int x = 0;
if (true) 
   //x = 1;//可以
   int y = 1;//不可以
```

```java
if (condition) {
    
}else if (condition) {
    
}else {
    
}
```

## 2.switch语句

- 表达式可以接收的类型：
    + 基本数据类型：byte、short、char、int
    + 引用数据类型：枚举（JDK1.5）、String（JDK1.7）
- case后只能是常量，不能是变量，且多个case后的值不能相同
- 最后一个break可以省略，其他break不能省略，可能出现case穿透（可能造成错误，但也可以加以利用）
- default和if语句的else相似，可以省略，但一般不省略，用于范围外的提示
- default不是一定要放在最后，可在任意位置，不管放在哪，都是最后才执行，但建议放最后
- switch结束的条件：
    + 遇到break;
    + 执行到switch的右大括号

#### 2.1 case穿透

> 进入case1后没有break，**不会再判断case2**，直接执行后面的语句。同时打印1和2

```java
int x = 1;
switch (x) {
    case 1:
        System.out.println(1);
        //break;
    case 2:
        System.out.println(2);
        break;
    default:
        System.out.println("超出范围");
        break;
}
```

> 利用case穿透

```java
int x = 4;
switch (x) {
    case 3:
    case 4:
    case 5:
        System.out.println("春季");
        break;
    case 6:
    case 7:
    case 8:
        System.out.println("夏季");
        break;
    default:
        System.out.println("秋季或冬季");
        //break;//可以省略
}
```

#### 2.2 default位置

> 虽然放在了最前面，但不会直接走default，还是走的case1。结果打印1，而不是“超出范围”

```java
int x = 1;
switch (x) {
    default:
        System.out.println("超出范围");
        break;
    case 1:
        System.out.println(1);
        break;
    case 2:
        System.out.println(2);
        break;
}
```

> case3和4都不满足，走default，但有case穿透，最终y=6，遇到右大括号结束的。**所以，如果default不是放在最后，最好使用break，否则很可能会case穿透**

```java
int x = 2;
int y = 3;
switch (x) {
    default:
        y++;
    case 3:
        y++;
    case 4:
        y++;
}
```

> 走case2，但有case穿透，最终y=5，遇到右大括号结束的（只要走了case就不会走default，除非default前有case穿透）

```java
int x = 2;
int y = 3;
switch (x) {
    default:
        y++;
    case 2:
        y++;
    case 4:
        y++;
}
sout(y);    //5
```

```java
int x = 2;
int y = 3;
switch (x) {
    case 2:
        y++;
    case 4:
        y++;
    default:
        y++;
}
sout(y);    //6
```

## 3.if和switch的区别

- 1.使用场景
    - switch常用在判断固定值的时候，效率更高，但有数据类型限制
    - if常用在区间或范围的判断
- 2.执行过程
    - switch走了一个case后，如果没有break，其他case不再判断，但其后的语句会继续执行（穿透）（不是{代码块}）
    - if走了一个后，else if不会再判断，else也不会再判断，其中的代码块不会执行
- 3.能用switch做的，if肯定也能做，反之不一定


---

# III.循环结构

## 1.for循环

初始表达式，条件表达式，循环后操作表达式，循环体

```java
for (int i = 0; i < 9; i++) {
    循环体;      
}
```

#### 1.1 流程

- 1.执行初始表达式（只执行一次）
- 2.执行条件表达式
    + A.条件表达式=true-->执行循环体-->循环后操作表达式-->条件表达式
    + B.条件表达式=false-->结束循环

> - **i是for循环中的局部变量，循环结束后会被释放，可再次声明使用**
> - 循环体只有一条语句时，可省略{}，但不建议省略

#### 1.2 案例

- 案例1

```java
//报错：for之后有;，表示执行了紧跟的空语句，循环结束后i被释放，syso处就会报错没有声明i
for (int i = 0; i < 9; i++);
    syso(i);      
```

- 案例2：水仙花数

各位上数的立方和=该数：153、370、371、407

```java
for (int i = 100; i < 1000; i++) {
    /*
    int x = i / 100;//百位
    int y = (i - 100 * x) / 10;//十位
    int z = (i - 100 * x - 10 * y);//个位
    */
    int z = i % 10;//个位
    int y = i / 10 % 10;//十位
    int x = i / 10 / 10;//百位
    if (i == x * x * x + y * y * y + z * z * z) {
        System.out.println(i);
    }
}
```

- 案例3：九九乘法表

```java
for (int i = 1; i < 10; i++) {
    for (int j = 1; j <= i; j++) {
        System.out.print(j + " * " + i + " = " + j * i + '\t');
    }
    System.out.println();
}
```

> Java5中引入了主要用于数组或者实现了Iterable接口的类的增强for循环，依次处理数组中的元素，而不需要指定数组索引。
> Java8中结合Lambda表达式，引入一种新的for循环，用于实现了Iterable接口的类的遍历，forEach方法是集合父接口java.lang.Iterable中新增的一个default实现方法。

## 2.while循环

组成：初始表达式，条件表达式，循环控制语句=循环后操作表达式，循环体

```java
初始化表达式;
while (条件表达式) {
    循环体; 
    循环控制语句;      
}
```

#### 2.1 流程

- 1.执行初始表达式（只执行一次）
- 2.执行条件表达式
    + A.条件表达式:true-->执行循环体-->循环后操作表达式-->条件表达式
    + B.条件表达式:false-->结束循环

> 如果没有循环控制语句，可能会一直在循环

## 3.do while循环

组成：初始表达式，条件表达式，循环控制语句=循环后操作表达式，循环体

```java
初始表达式;
do {
    循环体;
    循环控制语句;        
} while (条件表达式);
```

#### 3.1 流程
- 1.执行初始表达式（只执行一次）
- 2.先执行一次循环体和循环控制语句
- 3.执行条件表达式
    + A.条件表达式:true-->执行循环体-->循环后操作表达式-->条件表达式
    + B.条件表达式:false-->结束循环

## 4.三种循环的区别

- do while至少执行一次循环体：先执行一次循环体和循环控制语句后，再进行判断。其他两种不会先执行
- 如果想继续使用循环中的循环变量，用while，因为是声明在循环体外的；如果不再使用就用for，循环结束后释放变量，减少内存压力

## 5.死循环

```java
for (;;) {
    System.out.println(11);
}

while (true) {
    System.out.println(11);
}
```


---

# IV.跳转控制

## 1.break

**只能用在循环和switch中**

## 2.continue

终止本次循环，继续下一个循环，**只能在循环中使用**

## 3.标号

命名使用合法标识符。一般用于多层循环中

- break label

```java
outer: for (int i = 1; i < 10; i++) {
    System.out.println(i);
    inner: for (int j = 1; j < 10; j++) {
        System.out.println(j);
        break outer;//跳出外层循环(不加标号的break;只能跳出内层循环)
    }
}
```

```java
//不会报错http是个合法标号，"//www.baidu.com"是注释
System.out.println(i);
http://www.baidu.com
System.out.println(i);
```

- continue label

```java
public void test(){
    CONTINUE_LABEL:
    for (int i = 0; i < 3; i++) {
        System.out.println(String.format("start outer for loop index %d", i));
        for (int k = 0; k < 3; k++) {
            if (k == 1){
                continue CONTINUE_LABEL;
            }
            System.out.println(String.format("inner loop index %d", k));
        }
        System.out.println(String.format("end outer for loop index %d", i));
    }
    System.out.println("loop end");
}
```

输出结果：

```
start outer for loop index 0
inner loop index 0
start outer for loop index 1
inner loop index 0
start outer for loop index 2
inner loop index 0
loop end
```

## 4.return

结束方法
