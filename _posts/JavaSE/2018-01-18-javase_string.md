---
title: String
date: 2018-01-18 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Java中String的相关知识。

<!-- more -->

##### 目录
- I.简介
- II.String

---
# I.简介

Java中的字符串，就是Unicode字符序列，每个用双引号括起来的字符串都是String类的实例。例如在[在线Unicode/中文转换](http://tools.jb51.net/transcoding/unicode_chinese)中输入"Java\u2122"，对应了5个Unicode字符J、a、v、a和TM。

---
# II.String

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence
```

> String类是final的，不可被继承。另外，String还是不可变类。

## 1.内部实现

String内部定义了一个final的char数组。

```java
private final char value[];
```

**final修饰的变量不可被改变，但这不是String是不可变类的原因**。final只能保证引用不可变，并不能保证内容不可变。String类的大多数方法都是基于该char数组实现的。

## 2.构造

String类非过时的构造如下：

```java
public String()
public String(String original)
public String(char value[])
public String(char value[], int offset, int count)
public String(int[] codePoints, int offset, int count)
public String(byte bytes[], int offset, int length, String charsetName)
public String(byte bytes[], int offset, int length, Charset charset)
public String(byte bytes[], String charsetName)
public String(byte bytes[], Charset charset)
public String(byte bytes[], int offset, int length)
public String(byte bytes[])
public String(StringBuffer buffer)
public String(StringBuilder builder)
String(char[] value, boolean share)
```

构造方法较多，稍微讲述下以char数组为参数的构造：

```java
public String(char value[]) {
    this.value = Arrays.copyOf(value, value.length);
}

public String(char value[], int offset, int count) {
    if (offset < 0) {
        throw new StringIndexOutOfBoundsException(offset);
    }
    if (count <= 0) {
        if (count < 0) {
            throw new StringIndexOutOfBoundsException(count);
        }
        if (offset <= value.length) {
            this.value = "".value;
            return;
        }
    }
    // Note: offset or count might be near -1>>>1.
    if (offset > value.length - count) {
        throw new StringIndexOutOfBoundsException(offset + count);
    }
    this.value = Arrays.copyOfRange(value, offset, offset+count);
}
```

可以看到，String类通过Arrays.copyOf创造了一个新数组，而不是直接使用传进来的value数组（直接赋值，两个引用指向同一内存），这样对value数组的修改就不会影响到String。这是Sun公司精心设计String类的一个缩影，其他方法还有类似的"谨慎"设计，为的就是，**不暴露给外部修改char value[]的机会。这就是String类不可变的重要原因，而并非仅仅依靠一个final char数组的声明**。

## 3.方法

```java
// 返回字符串中index位置的字符
public char charAt(int index)
// 返回字符串中index位置的十进制码位值
public int codePointAt(int index)
// 返回字符串中index位置前的十进制码位值
public int codePointBefore(int index)  
// 返回字符串中beginIndex到endIndex之间的码位个数
public int codePointCount(int beginIndex, int endIndex) 
// 字符串比较
public int compareTo(String anotherString)
// 忽略大小写的字符串比较
public int compareToIgnoreCase(String str) 
// 字符串拼接 
public String concat(String str)  
// 判断s是否是当前字符串的子串  
public boolean contains(CharSequence s)
// 判断cs内容跟当前字符串是否相等
public boolean contentEquals(CharSequence cs)  
// 判断sb内容跟当前字符串是否相等
public boolean contentEquals(StringBuffer sb)   
// 静态方法，char数组转String
public static String copyValueOf(char data[]) 
// 静态方法，拷贝data[]中从offset索引位置起count个字符，生成字符串  
public static String copyValueOf(char data[], int offset, int count)    
// 判断字符串是否以suffix结束
public boolean endsWith(String suffix)
// 判断字符串跟anObject内容是否相等  
public boolean equals(Object anObject) 
// 忽略大小写，判断字符串跟anObject内容是否相等 
public boolean equalsIgnoreCase(String anotherString)   
// 格式化String，l为语言环境
public static String format(Locale l, String format, Object… args)  
// 格式化String
public static String format(String format, Object… args)    
// 通过操作系统默认支持的编码方式，获取字符串编码的byte数组
public byte[] getBytes()  
// 通过指定编码方式，获取字符串编码byte数组  
public byte[] getBytes(Charset charset) 
public byte[] getBytes(String charsetName) 
// 将srcBegin到srcEnd区间内的子串，拷贝到dst[]中dstBegin开始的位置
public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin)   
// 返回字符串hashCode 
public int hashCode()   
// 返回十进制码位ch在字符串内部持有的char数组首次出现的index
public int indexOf(int ch)  
// 返回十进制码位ch在字符串内部持有的char数组从fromIndex开始首次出现的index
public int indexOf(int ch, int fromIndex)  
// 返回str在字符串内部持有的char数组首次出现的index 
public int indexOf(String str)  
// 返回str在字符串内部持有的char数组从fromIndex开始首次出现的index
public int indexOf(String str, int fromIndex)   
// 显式地将字符串放入常量池，返回常量池中改字符串常量的地址
public native String intern()  
// 判断字符串是否为空 
public boolean isEmpty()    
// 连接字符串，连接符为delimiter
public static String join(CharSequence delimiter, CharSequence… elements)
public static String join(CharSequence delimiter,
Iterable<? extends CharSequence> elements)  
// 返回十进制码位ch在字符串内部持有的char数组最后出现的index
public int lastIndexOf(int ch)  
// 返回十进制码位ch在字符串内部持有的char数组从fromIndex开始最后出现的index
public int lastIndexOf(int ch, int fromIndex)   
// 返回str在字符串内部持有的char数组最后出现的index
public int lastIndexOf(String str)  
// 返回str在字符串内部持有的char数组从fromIndex开始最后出现的index
public int lastIndexOf(String str, int fromIndex)   
// 返回字符串内部持有的char数组长度
public int length() 
// 判断字符串是否符合正则表达式
public boolean matches(String regex)    
// 字符串内部持有的char数组从给定的index处偏移codePointOffset个代码位的索引
public int offsetByCodePoints(int index, int codePointOffset)   
// 检测两个字符串在一个区域内是否相等
public boolean regionMatches(boolean ignoreCase, int toffset,String other, int ooffset, int len) 
// 不忽略大小写，检测两个字符串在一个区域内是否相等
public boolean regionMatches(int toffset, String other, int ooffset, int len) 
// 字符串替换
public String replace(char oldChar, char newChar)
public String replace(CharSequence target, CharSequence replacement)
public String replaceAll(String regex, String replacement)
public String replaceFirst(String regex, String replacement)    
// 根据regex匹配分割字符串
public String[] split(String regex) 
// 判断字符串是否以prefix开始
public boolean startsWith(String prefix)    
// 判断字符串从toffset开始的子串，是否已prefix开始
public boolean startsWith(String prefix, int toffset)   
// 截取字符串，左闭右开
public CharSequence subSequence(int beginIndex, int endIndex)   
// 截取字符串，从beginIndex到字符串结束，包括beginIndex
public String substring(int beginIndex) 
// 截取字符串，左闭右开
public String substring(int beginIndex, int endIndex)   
// 转char数组
public char[] toCharArray() 
// 转小写，Local为语言环境
public String toLowerCase()
public String toLowerCase(Locale locale)     
// 转大写，Local为语言环境
public String toUpperCase()
public String toUpperCase(Locale locale)    
// toString方法
public String toString() 
// 去空格
public String trim()    
// 转字符串
public static String valueOf(boolean b)
public static String valueOf(char data[])
public static String valueOf(char c)
public static String valueOf(char data[], int offset, int count)
public static String valueOf(double d)
public static String valueOf(float f)
public static String valueOf(int i)
public static String valueOf(long l)
public static String valueOf(Object obj)    
```

#### 3.1 charAt

返回String内部char数组中，指定index的char。

```java
public char charAt(int index) {
    if ((index < 0) || (index >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return value[index];
}
```

如果字符串中只有单字节字符，字符串的字符个数就等于内部char数组的长度。比如str="a白菜饼b"，那么str.toCharArray().length=5，charAt(1)='白'。假如str="a𤭢b"，因为字符'𤭢'在JVM中用4字节(两个char)表示，所以charAt(1)会返回"半个"𤭢字符，无法正常表示，便展示为一个?。

```java
class StringTest {
    public static void main(String[] args) {
        System.out.println(System.getProperty("file.encoding"));//系统默认编码

        String str1 = "a白菜饼b";
        System.out.println("str1: " + str1);
        System.out.println("str1.length(): " + str1.length());
        System.out.println("str1.charAt(1): " + str1.charAt(1));

        System.out.println("----------------------------------");

        String str2 = "a𤭢b";
        System.out.println("str2: " + str2);
        System.out.println("str2.length(): " + str2.length());
        System.out.println("str2.charAt(1): " + str2.charAt(1));
    }
}
```
> 输出如下：

```
UTF-8
str1: a白菜饼b
str1.length(): 5
str1.charAt(1): 白
----------------------------------
str2: a𤭢b
str2.length(): 4
str2.charAt(1): ?
```