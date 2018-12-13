---
title: Java基础
date: 2018-01-01 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录一些基础知识。

<!-- more -->

##### 目录
+ I.计算机基础
+ II.Java简介
+ III.Java基础

---

# I.计算机基础

#### 1.硬件Hardware

> 冯.诺依曼结构

- 计算机硬件的5大组成部件：运算器、控制器、存储器、输入设备和输出设备
- 运算器和控制器是核心，合称中央处理单元（Central Processing Unit）。CPU中还有一些高速存储单元-寄存器
    - 运算器执行算术、逻辑运算
    - 控制器从存储器将指令取出，经编译后向计算机发出控制命令
    - 寄存器为CPU提供操作所需的数据
- 存储器分内部储存器和外部存储器，用来存储程序和数据
    - 内存存放正在执行的程序和使用的数据，成本高、容量小，但速度快
    - 外存用于长期保存程序和数据，成本低、容量大，但速度慢
- 输入输出设备，简称外设或I/O设备，实现人机交互

#### 2.软件Software

- 系统软件：DOS（Disk Operating System）、Windows、Linux、Unix、Mac、Android、iOS
- 应用软件：office、Eclipse...

#### 3.计算机语言

> 人和计算机交流沟通的语言，已发展三代

- 机器语言：直接用二进制代码指令表示，由0和1组成一串代码指令
- 汇编语言：比第一代稍好，使用一些特殊符号来代替机器语言中的二进制代码，如MOV等。计算机不能直接识别，需要翻译成机器语言
- 高级语言：使用阅读性高的英语等编码，适合开发。通过编译器将源码翻译成机器语言。如C、Java等

#### 4.人机交互方式

- 命令行
- 图形化界面

# II.Java简介

> JAVA技术，既可指一种编程语言，又可指Java Platform
> - 编程语言：有特定语法和风格的高级面向对象语言
> - Java平台：Java应用运行的环境（Java SE、Java EE、Java ME、JavaFX）

- James Gosling
- 前名"Oak"（橡树）
- 特点：开源、跨系统平台(write once,run anywhere! 各系统的JVM，JVM负责Java程序运行)、面向对象

> - Java的跨平台：JDK或JRE级别。即源码和字节码文件（编译）对不同平台是相同的（write once），只是JDK版本随平台不同而不同
> - C或C++的跨平台：在编译器或代码级别实现跨平台，即编码会根据系统平台不同而有差异

#### 1.平台版本

分为Java SE、Java ME、Java EE(WEB开发，包含servlet，jsp等)、JavaFX（富互联网应用）

- [所有平台版本](https://docs.oracle.com/javaee/6/firstcup/doc/gkhoy.html)都包括a virtual machine and an API
- JVM：是个虚拟机程序，针对不同硬件和软件平台
- API：core classes，用来编写应用

> 其中Java SE平台版本中包含：

- API：提供了Java语言的基础核心功能。其中定义了basic types and objects，以及用于networking、security、database access、graphical user interface (GUI) development和XML parsing的类
- JVM
- 开发工具、部署技术、其他常用的类库和工具包

> JAVASE5.0(1.5.0)/JDK5.0是里程碑版本(Tiger)，特性比较重要

> Java EE平台版本建立在Java SE基础上，额外提供了Servlet、JSP等API

JavaSE8 at a glance:

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/basic01/Javase8-at-a-glance.png)


#### 2.JVM、JRE和JDK

> Java有内存管理（经久不衰的原因），所以JVM要深入了解

- JVM(Virtual Machine)：针对不同硬件和软件平台
- JRE(Runtime Environment) = JVM + java程序所需的核心类库等（运行环境）
- JDK(Development Kit) = JRE + 开发工具(javac编译工具、jar打包工具、JavaDoc、java Debugger等)，SDK的一种
- 开发需要JDK，仅运行Java程序使用JRE即可

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/basic01/jdk-jre-jvm.jpg)

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/basic01/how-java-program-runs.jpg)

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/basic01/jvm-architecture.png)

#### 3.环境配置

- 官网安装JDK（不需要再单独安装JRE）
- 环境变量配置：Path和classpath配置
- Path指**可执行命令文件**路径，classpath指**class文件**路径

###### 3.1 Path配置

> notepad（记事本）在Windows任何目录的DOS中都能执行，就是因为该命令的可执行文件配置到了Path中

- 1.未将java和javac命令（可执行文件）添加到Path时，只能在JDK的bin目录执行javac和java命令(即只能在bin目录编译和运行)，显然不合理。（）
- 2.将bin目录(如D:\JavaDevelop\jdk\jdk1.8.0_131\bin)添加到Path中；Path有先后顺序，可将jdk的bin目录放在最前面，用;分割，重启命令窗口即可
  - 方式1：在Path中直接添加
  - 方式2(推荐)：另外创建JAVA_HOME=D:\JavaDevelop\jdk\jdk1.8.0_131\bin，将%JAVA_HOME%配置到Path中（好处：可复用、方便修改）

> Linux下的Path配置相似

###### 3.2 classpath配置

- 1.classpath推荐不做配置，因为默认classpath就是java文件的所在目录，这样class文件自动生成在java文件所在的目录中，两个文件在一起挺好。只不过不能在其他目录对该class文件执行java命令
- 2.如果配置了classpath(环境变量-系统变量中配置)，那么虽然能在任何目录对classpath中的字节码文件运行，但要求所有的class文件都必须在该classpath中，否则不能执行
- 3.如果非要配置可以配成classpath=.;D:\haha，.表示当前目录。其实相当于没配，所以**classpath不需要配置**

#### 4.JDK安装目录介绍

- bin目录：可执行程序，如javac（编译器）、java（运行工具）、jar（打包工具）、javadoc（文档生成工具）
- db目录：小型数据库。从JDK6.0开始，Java中引用新成员JavaDB，是由Java实现的开源数据库系统，轻便且支持JDBC4.0规范。所以在学习JDBC时不额外安装其他数据库也可以，可直接使用JavaDB
- jre目录：Java运行时环境的根目录，包含了JVM、运行时核心类包、Java应用启动器和一个bin目录，但不包括开发工具。这个jre目录和单独安装JRE时的目录一样，所以安了JDK，不需再安JRE
- include目录：JDK是通过C和C++实现的，启动时需要引入一些C的头文件，这些头文件就存放在此
- lib目录：Java类库，开发工具使用的归档包文件
- src.zip：JDK核心类的源码文件的压缩文件

#### 5.IDE

- notepad(记事本)、Editplus、notepad++、Eclipse、MyEclipse、IDEA等
- 注意代码格式规范美观，约定俗成

# III.Java基础

#### 1.基础

- 1.java文件名不必跟其代码中的类名一致（一般要求一致）；编译后的class文件和类名一致，不是按java文件名命名
- 2.java命令需要跟文件的java后缀，而javac命令不跟字节码文件的.class后缀

```
java Hello.java
javac Hello
```

#### 2.语法

###### 2.1 注释

- 单行//abc，可以嵌套
- 多行/*abc*/，不可嵌套
- 文档/**abc*/，用于生成帮助文档的

> 生成API文档示例

- 1.测试类

```
/**
 * 测试工具类
 * @author yuzhou
 * @since 1.0.0
 */
public class ArrayTool {
    /** 测试文档生成:私有成员变量，不会生成对应说明 */
    private String privateTest;
    /** 成员变量test */
    public String publicTest;
    
    /** 测试文档生成:私有成员方法(构造)，不会生成对应说明 */
    private ArrayTool() {}
    
    /**
     * 成员方法-打印
     * @param s
     * @return
     */
    public static String print(String s) {
        System.out.println(s);
        return s;
    }
}
```

- 2.在ArrayTool.java所在目录下，执行下列命令即可

```
# -d是当前目录下创建api文件夹
javadoc -d api -version -author ArrayTool.java
```

- 3.打开api文件夹中的index.html即可查看

###### 2.2 关键字

> 语言中被赋予特殊含义的单词

- 都是**小写**
- goto和const为保留关键字，留用（C++中有使用）

###### 2.3 标识符

> 类、接口、方法、变量等的命名字符序列

- 包含a-z，A-Z，0-9，$，_下划线
- 不能以数字开头
- 不能使用关键字（李世民-观(世)音菩萨）
- 大小写敏感

> Java中的命名：

- 包名：域名倒写
- 类或接口：Hello、HelloWorld（驼峰）
- 方法和变量：hello()、helloWorld（低头驼峰）
- 常量：HELLO、HELLO_WORLD


#### 3.进制

> 二进制、八进制、十六进制

- 二进制起源易经中的阴阳/开关信号
- 八进制和十六进制都是为了简化表示二进制，相当于二进制的缩写。所以其他进制转换成八/十六进制时，都需要先转成二进制
- 进制越大，(同一值)表现形式越短

###### 3.1 二进制

- 组成：0和1
- 表现形式：JDK1.7后，以0b/0B开头

```
# 表示二进制110
0b110
# 表示十进制整数
110
```

- 比特位：0/1
- 字节：1byte = 8bit(1字节=8比特位)，1kb = 1024b

> 硬盘厂商是以1000倍数来生产的，计算机运算中使用1024位数来运算的。厂家500g = 500 × 1000 × 1000b ÷ 1024 ÷ 1024 = 465.7g，实际只有约465.7g

###### 3.2 八进制

- 组成：0-7
- 表现形式：以0开头

###### 3.3 十进制
- 组成：0-9
- 表现形式：整数默认是十进制的

###### 3.4 十六进制
- 组成：0-9，a-f（大小写都可）
- 表现形式：以0x/0X开头

###### 3.5 其他进制转换成十进制

> 系数×基数的(权次幂)

```
syso默认输出十进制：
syso(0b100);    //输出4
syso(0100);     //输出64
syso(100);      //输出100
syso(0x100);    //输出256
```

###### 3.6 十进制转换成其他进制

> 除积，倒取余

###### 3.7 二进制转换成其他进制

> 8421码快速运算

#### 4.原码、反码、补码

> 计算机中存储、运算等用补码。反码用于两者间的转换。原码是二进制的定点表示法，最高位为符号位，0正1负，其余位是数值位

- **正数：原反补相同**
- **负数：反码 = 除符号位，对原码其他位取反；补码 = 反码末位 + 1**

> - 以8位二进制数为例：表示范围：-128~127，其中-128的补码为[1000 0000]，它没有原码和反码

> - 这里只考虑原反补的简化计算方式，深入的历史背景和设计原理不深究...
> - 原码更符合人的逻辑思维，但补码运算是为了降低计算机硬件的复杂度
> - 通过补码运算，可以把减法运算变成加法运算；而乘法可以用加法来做，除法可以转变成减法。这样加、减、乘、除四种运算，只需要CPU中有加法器就可以了