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
- III.参考

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

#### 3.2 codePointAt

返回index位置上的字符，在字符集中的码位的十进制。

```java
public int codePointAt(int index) {
    if ((index < 0) || (index >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return Character.codePointAtImpl(value, index, value.length);
}
```

标准ASCII码包含128个码位(8位，1字节)，从0x01到0x7F。扩展ASCII码包含256个码位(8位，1字节)，从0x00到0xFF。而Unicode包含1,114,112个码位，范围是0(16进制)到10FFFF(16进制)。Unicode的码位使用码元来表示，有的码位使用一个码元(16位，2字节)表示，有的需要使用两个码元(32位，4字节)表示。Java中char（占2字节）是使用Unicode编码的，所以其实一个char就可以理解为一个码元。而codePointAt的作用是返回字符串内部持有的字符数组index索引位置的码元的十进制表示。这里要分双字节字符和四字节字符讨论：

```java
public static void main(String[] args) {
    //系统默认编码
    System.out.println(System.getProperty("file.encoding"));

    String str1 = "a白菜饼b";
    System.out.println("str1: " + str1);
    System.out.println("str1.length(): " + str1.length());
    System.out.println("str1.charAt(1): " + str1.charAt(1));
    //使用构造函数public String(char value[])
    System.out.println("str1.codePointAt(1): " + new String(Character.toChars(str1.codePointAt(1))));
    //使用构造函数public String(int[] codePoints, int offset, int count)
    System.out.println("str1.codePointAt(2): " + new String(new int[]{str1.codePointAt(2)}, 0, 1));

    System.out.println("----------------------------------");

    String str2 = "a𤭢菜饼b";
    System.out.println("str2: " + str2);
    System.out.println("str2.length(): " + str2.length());
    System.out.println("str2.charAt(1): " + str2.charAt(1));

    System.out.println("str2.codePointAt(1): " + new String(Character.toChars(str2.codePointAt(1))));
    System.out.println("str2.codePointAt(1): " + new String(new int[]{str2.codePointAt(1)}, 0, 1));

    //'𤭢'是双字节字符，所以'菜'的index是3不是2
    System.out.println("str2.codePointAt(3): " + new String(new int[]{str2.codePointAt(3)}, 0, 1));

    //如果使用codePointAt(2),会得到一个非法的codePoint,因为char数组index 2位置是字符'𤭢'的低16位。
    //对于四字节字符，可以获取合法codePoint的前提是index必须是字符数组该四字节字符的高16位char的index
    System.out.println("str2.codePointAt(2): " + new String(new int[]{str2.codePointAt(2)}, 0, 1));
}
```

> 控制台输出

```java
UTF-8
str1: a白菜饼b
str1.length(): 5
str1.charAt(1): 白
str1.codePointAt(1): 白
str1.codePointAt(2): 菜
----------------------------------
str2: a𤭢菜饼b
str2.length(): 6
str2.charAt(1): ?
str2.codePointAt(1): 𤭢
str2.codePointAt(1): 𤭢
str2.codePointAt(3): 菜
str2.codePointAt(2): ?
```

#### 3.3 codePointCount

返回字符串内部value组从beginIndex到endIndex(左闭右开)码位的个数(一个码位其实就等同于字符串中一个字符)

```java
public int codePointCount(int beginIndex, int endIndex) {
    if (beginIndex < 0 || endIndex > value.length || beginIndex > endIndex) {
        throw new IndexOutOfBoundsException();
    }
    return Character.codePointCountImpl(value, beginIndex, endIndex - beginIndex);
}
```

String.length()返回字符串内部value数组的length。**有些字符使用unicode编码是占4个字节（2个char），所以使用length()方法，并不一定跟字符串中字符的个数完全相同**。

```java
public static void main(String[] args) {
    //系统默认编码
    System.out.println(System.getProperty("file.encoding"));

    String str1 = "a白菜饼b";
    System.out.println("str1: " + str1);
    System.out.println("str1.length(): " + str1.length());
    System.out.println("str1.codePointCount(): " + str1.codePointCount(0, str1.length()));

    System.out.println("----------------------------------");

    String str2 = "a𤭢菜饼b";
    System.out.println("str2: " + str2);
    System.out.println("str2.length(): " + str2.length());
    System.out.println("str2.codePointCount(): " + str2.codePointCount(0, str2.length()));
}
```

> 控制台输出

```java
UTF-8
str1: a白菜饼b
str1.length(): 5
str1.codePointCount(): 5
----------------------------------
str2: a𤭢菜饼b
str2.length(): 6
str2.codePointCount(): 5
```

#### 3.4 getBytes

按照特定编码，将字符串转化为byte数组（计算机中的表示形式），不指定编码方式，将使用系统默认的编码方式。

```java
public byte[] getBytes(Charset charset) {
    if (charset == null) throw new NullPointerException();
    return StringCoding.encode(charset, value, 0, value.length);
}
```

在Unicode字符集中，每个字符都有一个对应的码位。同一个码位使用不同的编码方式会得到的不同结果，而字符在计算机中存储的形式，就是编码后的二进制串。比如汉字"中"，使用UTF-16编码结果是\u4E2D（16位，2个字节），而使用UTF-8编码结果则是\uE4B8AD（24位，3个字节）。

```java
public static void main(String[] args) {
    //系统默认编码
    System.out.println(System.getProperty("file.encoding"));
    System.out.print("UTF-8: ");
    printHexString("中".getBytes());

    System.out.print("UTF-16: ");
    printHexString("中".getBytes(StandardCharsets.UTF_16BE));
}

private static void printHexString(byte[] b) {
    StringBuilder hex = new StringBuilder();
    for (int i = 0; i < b.length; i++) {
        hex.append(Strings.padStart(Integer.toHexString(b[i] & 0xFF), 2, '0'));
    }
    String result = "\\u" + hex.toString().toUpperCase();
    System.out.println(result);
}
```

> 控制台输出

```java
UTF-8
UTF-8: \uE4B8AD
UTF-16: \u4E2D
```

同一字符串"中"，使用UTF-8编码结果是\uE4B8AD（24位，3字节），使用UTF-16编码结果是\u4E2D（16位，2字节）。Java中的char在内存中占2字节，使用Unicode编码，所以char使用的是UTF-16编码后存储的。UTF-16编码是变长2/4字节编码方式，所以这也是为什么之前"𤭢"可以占两个char的原因。

#### 3.5 intern

提供了运行期间动态地将常量放入常量池的方式。（对应class文件中常量池的静态描述）

```java
/**
 * 调用intern时，如果pool中已经包含了和该string相同的字符串（equals返回true），返回池中的字
 * 符串地址。否则先加入pool中，再返回池中的字符串地址。
 */
public native String intern();
```

在理解intern()前，先看一下下面的几个问题。

##### 3.5.1 字面量和运行时常量池

为了提高性能和减少内存开销，JVM在实例化字符串常量时进行了优化。为了减少在JVM中创建的字符串数量，String类维护了一个字符串常量池，相同内容的字符串对象，可以引用字符串常量池中同一个字符串常量（基于之前讲的字符串的不可变性）。Java6及之前，常量池维护在JVM方法区中。从Java7之后，常量池放在了JVM堆中，主要用来存储**编译期生成的各种[字面量（Literal）](https://docs.oracle.com/javase/specs/jls/se8/html/jls-3.html#jls-3.10.5)和符号引用（Symbolic References）**。

> **class文件中的Constant pool部分的内容，会在类加载时被加载到运行时常量池中**。Java中基本类型的包装类的大部分都实现了常量池技术：Byte、Short、Integer、Long、Character、Boolean，这5种包装类默认创建了数值[-128，127]的相应类型的缓存数据，但是超出此范围仍然会去创建新的对象。另外，两种浮点数类型的包装类Float、Double并没有实现常量池技术。

```java
public class Intern {
    public static void main(String[] args){
        String s1 = "123";
    }
}
```

编译后，Constant pool内容如下，\#14就是字面值常量：

```java
$ javap -v Intern.class 
...
Constant pool:
   #1 = Methodref          #4.#13         // java/lang/Object."<init>":()V
   #2 = String             #14            // 123
   #3 = Class              #15            // com/baicai/thread/Intern
   #4 = Class              #16            // java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Utf8               Code
   #8 = Utf8               LineNumberTable
   #9 = Utf8               main
  #10 = Utf8               ([Ljava/lang/String;)V
  #11 = Utf8               SourceFile
  #12 = Utf8               Intern.java
  #13 = NameAndType        #5:#6          // "<init>":()V
  #14 = Utf8               123
  #15 = Utf8               com/baicai/thread/Intern
  #16 = Utf8               java/lang/Object
...
```

上述字节码中字符串"123"在常量池中的定义方式：

```java
#2 = String             #14            // 123
#14 = Utf8              123
```

在main方法字节码指令中，0-2行对应代码String s1 = "123"，对应ldc #2和astore_1指令。

```java
public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=1, locals=2, args_size=1
         0: ldc           #2                  // String 123
         2: astore_1      
         3: return        
```

- 1.类加载时，"123"字符串在Constant pool中使用符号引用symbol表示，当调用ldc #2指令时，如果Constant pool中索引#2的symbol还未解析，则调用C++底层的StringTable::intern方法生成char数组，并将引用保存在StringTable和常量池中，当下次调用ldc #2时，可以直接从Constant pool根据索引#2获取"123"字符串的引用，避免再次到StringTable中查找。
- 2.astore_1指令将"123"字符串的引用保存在局部变量表中

> 常量池的内存分配在JDK6、7、8中有不同的实现：

- 1.JDK6及之前，常量池放在永久代PermGen中，所以其大小会受到PermGen内存大小的限制
- 2.JDK7中，常量池的内存在Java堆上进行分配，意味着常量池不受固定大小的限制了
- 3.JDK8中，虚拟机团队移除了永久代PermGen

##### 3.5.2 new String("123")到底创建了几个对象

```java
String s1 = new String(“123”)
```

在编译期，字面量123会被加入到class文件的Constant pool中，在类加载阶段，会进入字符串常量池。当然并不会直接把Constant pool中的所有常量都加载进来，而是会先做比较，如果字符串在常量池中已存在，就不需要再把字符串加进来了。

在运行期，执行到new String("123")时，因为有new操作，要在JVM堆中创建一个字符串对象，而该对象所对应的字符串字面量已经保存在常量池中（类加载时放入）。引用s1保存在JVM栈上，它指向的是堆中刚创建的字符串对象，而不是常量池中的字面值地址。如下图：

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/string/NewString.jpg)

- 1.常量池中的对象是在编译期就初步确定的，在类加载时创建的
- 2.堆中的对象是在运行期才创建的，由new操作触发

所以，**new String("123")，会创建一个或两个对象。类加载时肯定创建一个池中对象，运行时如果执行到改行代码，会再创建一个堆中对象**。

##### 3.5.3 intern()方法的作用

上面提到，编译期生成的各种字面量和符号引用是运行时常量池中比较重要的一部分来源，但是不是全部。**String.intern()可以在运行期向运行时常量池中动态地增加常量**。当调用intern()时：

- 如果常量池中没有相同Unicode的字符串常量，则在常量池中创建一个
- 返回常量池中常量的引用

```java
public static void main(String[] args){
    String s1 = "123";
    String s2 = new String("123");
    String s3 = new String("123").intern();

    System.out.println("s1 == s2: " + String.valueOf(s1 == s2));    //false
    System.out.println("s1 == s3: " + String.valueOf(s1 == s3));    //true
    System.out.println("s2 == s3: " + String.valueOf(s2 == s3));    //false
}
```

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/string/StringIntern.jpg)

对于String s3 = new String("123").intern()，在不调用intern情况，s3指向的是JVM在堆中创建的那个对象的引用的（如图中的s2）。但是当执行了intern方法时，s3将指向字符串常量池中的那个字符串常量，不会创建堆中的String对象了。

##### 3.5.4 intern()正确打开方式

上文中调用intern()好像并没有什么意义，因为即使不使用intern()，new String("123")在编译后，123作为字面量也会被放到class文件的Constant pool中，进而加入到运行时常量池中。显然这种情况是不需要使用intern()的，那么intern()的正确打开方式是什么？先来看两段代码：

```java
//示例1
String s1 = "123";
String s2 = "456";
String s3 = s1 + s2;

//示例2
String s4 = "123" + "456";
```

> 借助IDEA，将class反编译为java文件

```java
//示例1
String var1 = "123";
String var2 = "456";
(new StringBuilder()).append(var1).append(var2).toString();

//示例2
String var4 = "123456";
```

同样是字符串拼接，s3和s4在编译后的实现方式并不相同。s3被转化成StringBuilder及append，而s4被直接拼接成新的字符串。究其原因，**是编译器做了优化，对于s4而言，编译器可以确定一个字面量。而对于s3，编译器无法知道s3内容是什么，只能等到运行时才知道结果**。下面是class文件的Constant pool：

```java
Constant pool:
   #1 = Methodref          #10.#19        //  java/lang/Object."<init>":()V
   #2 = String             #20            //  123
   #3 = String             #21            //  456
   #4 = Class              #22            //  java/lang/StringBuilder
   #5 = Methodref          #4.#19         //  java/lang/StringBuilder."<init>":()V
   #6 = Methodref          #4.#23         //  java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #7 = Methodref          #4.#24         //  java/lang/StringBuilder.toString:()Ljava/lang/String;
   #8 = String             #25            //  123456
   #9 = Class              #26            //  com/baicai/thread/Intern
  #10 = Class              #27            //  java/lang/Object
  #11 = Utf8               <init>
  #12 = Utf8               ()V
  #13 = Utf8               Code
  #14 = Utf8               LineNumberTable
  #15 = Utf8               main
  #16 = Utf8               ([Ljava/lang/String;)V
  #17 = Utf8               SourceFile
  #18 = Utf8               Intern.java
  #19 = NameAndType        #11:#12        //  "<init>":()V
  #20 = Utf8               123
  #21 = Utf8               456
  #22 = Utf8               java/lang/StringBuilder
  #23 = NameAndType        #28:#29        //  append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #24 = NameAndType        #30:#31        //  toString:()Ljava/lang/String;
  #25 = Utf8               123456
  #26 = Utf8               com/baicai/thread/Intern
  #27 = Utf8               java/lang/Object
  #28 = Utf8               append
  #29 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #30 = Utf8               toString
  #31 = Utf8               ()Ljava/lang/String;
```

可以看到Constant pool中，示例1的字面量只有#20 123和#21 456，没有123456，而示例2中有123456字面量。这就意味着，类加载后，示例1并不会有123456放入常量池的，此时就可以使用intern了。

**程序中只能在运行期才能确定的String，无法在类加载时被加入到常量池中。此时，对于那些可能经常使用到的字符串，就可以使用intern进行定义，在运行时加入常量池。然后返回该字面值引用，减少字符串对象的创建**。如：

```java
static final int MAX = 1000 * 10000;
static final String[] arr = new String[MAX];

public static void main(String[] args) throws Exception {
    Integer[] DB_DATA = new Integer[10];
    Random random = new Random(10 * 10000);
    for (int i = 0; i < DB_DATA.length; i++) {
        DB_DATA[i] = random.nextInt();
    }
    long t = System.currentTimeMillis();
    for (int i = 0; i < MAX; i++) {
         arr[i] = new String(String.valueOf(DB_DATA[i % DB_DATA.length])).intern();
    }

    System.out.println((System.currentTimeMillis() - t) + "ms");
    System.gc();
}
```

上述代码，会有很多重复的字符串产生，但这些字符串只能在运行期才能确定。所以，只能通过intern显示的在运行期将其加入常量池，减少字符串的重复创建。是否使用intern()，运行相差4s左右。

#### 3.6 hashCode

**String类内部维护了一个私有成员变量hash，用于缓存hashCode()返回值**，第一次调用hashCode()时，把结果保存到hash变量，之后的调用将直接返回缓存值。

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

> 具体的计算方式：

```java
value[0]*31^(n-1) + value[1]*31^(n-2) + ... + value[n-1]
```

**该公式中，hash值与每个字符相关。不同位置乘以不同的值，所以hash值也与每个字符的位置相关关**。使用31的原因：

- 可以产生更分散的散列，尽量保证不同字符串的hash值也不同
- 计算效率比较高，31*h与32*h-h即 (h<<5)-h等价，可以用更高效的移位和减法操作代替乘法操作

> 在Java中，普遍采用以上思路来实现hashCode。

## 4.String的不可变性

#### 4.1 什么是不可变性

String的不可变性很简单：**不是在原内存地址上修改数据，而是重新指向一个新对象**。

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/string/string_unchange.png)

#### 4.2 为什么是不可变的

- 1.首先，String类是final的，不可被继承，防止子类修改
- 2.其次，成员value是个final char[]数组，**保证引用不可变**

```java
final int[] a1 = {1, 2, 3};
int[] a2 = {4, 5};
// 编译报错
a1 = a2;
// 但仍可修改内容
a1[0] = 4;
```

- 3.最后，最关键的是内部各方法的实现，都没有修改char[] value的元素，也不对外提供修改的机会

String的不可变性，关键在于SUN公司的工程师，在所有String的方法实现里，没有去动内部char数组的元素，也没有暴露内部成员字段。比如初始化时，是通过Arrays.copyOf创造一个新的数组，而非直接使用构造函数传来的数组参数。也正是因为String的不可变性，Stirng字符串常量池才可以实现。


---
# III.参考

- [深入解析String#intern](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)
- [Java编程拾遗『String类』](http://lidol.top/java/760/)
- [我终于搞清楚了和String有关的那点事儿。](https://www.hollischuang.com/archives/2517)