---
title: Java基础
date: 2018-06-16 19:01:09
categories:
    - JavaSE
tags:
    - JavaSE
    - Basic
    - Java
---

本文将介绍一些计算机常识和Java常识。

<!-- more -->

##### 目录
+ I.计算机基础
+ II.Java常识
+ III.快速入门
+ IV.核心知识
+ V.创建Maven web项目
+ VI.参考

---

# I.计算机基础

#### 1.硬件Hardware
冯.诺依曼结构

- 计算机硬件由5大组成部件：运算器、控制器、存储器、输入设备和输出设备
- 运算器和控制器是核心，合成中央处理单元（Central Processing Unit，CPU）。CPU中还有一些高速存储单元-寄存器。运算器执行算术&逻辑运算；控制器从存储器将指令取出，经编译后向计算机发出控制命令；寄存器为CPU提供操作所需的数据
- 存储器分内部储存器和外部存储器，用来存储程序和数据。内存存放正在执行的程序和使用的数据，成本高、容量小，但速度快；外存用于长期保存程序和数据，成本低、容量大，但速度慢
- 输入输出设备，简称外设或I/O设备，实现人机交互

#### 2.软件Software

- 系统软件：DOS（Disk Operating System）、Windows、Linux、Unix、Mac、Android、iOS
- 应用软件：office、Eclipse...

#### 3.计算机语言

> 人和计算机交流沟通的语言，已发展至三代

- 机器语言：直接用二进制代码指令表示，由0和1组成一串代码指令
- 汇编语言：比第一代稍好，使用一些特殊符号来代替机器语言中的二进制代码，如MOV等。计算机不能直接识别，需要翻译成机器语言
- 高级语言：使用阅读性高的英语等进行编码，适合开发。通过编译器将源代码翻译成机器语言。如C，C++，Java...

#### 4.人机交互方式

- 命令行
- 图形化界面

#### 5.常见DOS命令

>Windows系统使用win+R，cmd进入

- d:+回车---进入盘符
- dir 列出当前目录下的文件和文件夹
- md aaa 创建文件夹aaa
- rd aaa 删除文件夹aaa
- cd  进入目录
- cd.. 返回上一级
- cd\ 返回到根目录
- del a.txt 删除文件
- del *.txt 通配符删除
- cls 清屏
- exit

---

# II.Java常识

- James Gosling
- 前名"Oak"(橡树)
- 特点：开源&跨系统平台(write once,run anywhere! 各个系统的JVM，JVM负责Java程序运行)、面向对象

#### 1.平台版本

> J2SE、J2ME、J2EE(WEB开发，包含servlet，jsp等)

#### 2.JVM、JRE和JDK

- JVM(Virtual Machine)
- JRE(Runtime Environment) = JVM + java程序所需的核心类库等。
- JDK(Development Kit) = JRE + 开发工具(编译工具javac.exe，打包工具jar.exe，java.exe等)，SDK的一种
	- JAVASE5.0(1.5.0)/JDK5.0里程碑的版本(Tiger)，特性比较重要

#### 3.环境配置

- 官网安装JDK（不需要再单独安装JRE）
- 环境变量配置：Path和classpath配置
- Path指**可执行命令文件**路径；classpath指**class文件**路径

###### 3.1 Path配置

- 1.未将java和javac命令（可执行文件）添加到Path时，只能在JDK的bin目录执行javac和java命令(即只能在bin目录编译和运行)，显然不合理。（notepad在Windows中任何目录的DOS中都能执行，就是因为该命令的可执行文件配置到了Path中）
- 2.将bin目录(D:\JavaDevelop\jdk\jdk1.8.0_131\bin)添加到Path中(环境变量-系统变量)中；Path有先后顺序，可将jdk的bin目录放在最前面，用;分割，重启命令窗口即可
  - 方式1：在Path中直接添加
  - 方式2(推荐)：另外创建JAVA_HOME=D:\JavaDevelop\jdk\jdk1.8.0_131\bin，将%JAVA_HOME%配置到Path中（好处：可复用、方便修改）

###### 3.2 classpath配置

- 1.classpath推荐不做配置，因为默认classpath就是java文件的所在目录，这样class文件自动生成在java文件所在的目录中，两个文件在一起挺好。只不过不能在其他目录对该class文件执行java命令
- 2.如果配置了classpath(环境变量-系统变量中配置)，那么虽然能在任何目录对classpath中的字节码文件运行，但要求所有的class文件都必须在该classpath中，否则不能执行
- 3.如果非要配置可以配成classpath=.;D:\haha，.表示当前目录。其实相当于没配，所以**classpath不需要配置**


#### 4.JDK安装目录介绍

- bin目录：可执行程序，如javac.exe（编译器）、java.exe（运行工具）、jar.exe（打包工具）、javadoc.exe（文档生成工具）
- db目录：小型数据库。从JDK6.0开始，Java中引用新成员JavaDB，是由纯Java实现的开源数据库系统，轻便且支持JDBC4.0规范。所以在学习JDBC时不额外安装其他数据库也可以，直接使用JavaDB即可
- jre目录：是Java运行时环境的根目录，包含了JVM、运行时核心类包、Java应用启动器和一个bin目录，但不包括开发工具。这个jre目录和单独安装JRE时的目录一样，所以安了JDK，不需再安JRE
- include目录：JDK是通过C和C++实现的，启动时需要引入一些C的头文件，这些头文件就存放在这
- lib目录：Java类库，开发工具使用的归档包文件
- src.zip：JDK核心类的源码文件的压缩文件


#### 5.IDE

- notepad(记事本)、Editplus、Eclipse、MyEclipse、IDEA等
- 注意格式规范美观，约定俗成，使用eclipse或idea等工具可避免