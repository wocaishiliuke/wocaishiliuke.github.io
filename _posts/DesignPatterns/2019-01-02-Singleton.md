---
title: 设计模式-单例模式
date: 2019-01-02 19:02:09
categories:
    - DesignPatterns
tags:
    - DesignPatterns
    
---

本文基于【芋道源码】的《深入解析单例模式的七种实现》稍作修改，记录单例模式的实现方式。

<!-- more -->

##### 目录
+ I.简介
+ II.实现方式

---

# I.简介

## 1.what

单例模式Singleton Pattern，又称单子模式，是一种常用的软件设计模式。应用该模式时，单例的类必须保证只有一个实例存在。包括以下场景

- 1.类加载时就创建实例的情况（由JVM保证）
- 2.应用运行期间，多线程访问时才创建实例的情况（懒加载）
- 3.反射和反序列化时

单例好处：

- 1.对于频繁使用的对象，节省创建对象的开销，尤其是重量级对象
- 2.少了new操作，降低内存的使用频率，减轻GC压力，缩短GC停顿时间

## 2.why

> 实际就是因为，有时候需要一个全局对象，或者一个对象就足够。并且能节省内存

很多时候，整个系统只需要拥有一个的全局对象，这样有利于协调系统整体的行为。比如在某个服务器程序中，该服务器的配置信息存放在一个文件中，这些配置数据由一个单例对象统一读取，然后服务进程中的其他对象再通过该单例对象获取这些配置信息，简化在复杂环境下的配置管理。再如常用的servlet也是单例多线程的，很多abstract factory和builder也是单例的。

## 3.how

- a.私有构造方法（不让其他类创建本类实例）
- b.提供一个public static的getInstance()，返回该类的唯一实例

下面将叙述单例模式的7种实现方式


---

# II.实现方式

## 1.懒汉式

最简单的单例模式

```java
public class Singleton1 {
    /** 引用 */
    private static Singleton1 singleton1;
    
    /** 1.私有构造 */
    private Singleton1() {}

    /** 2.获取实例的方法 */
    public static Singleton1 getInstance() {
        if (singleton1 == null) {
            singleton1 = new Singleton1();
        }
        return singleton1;
    }
}
```

与饿汉式相比，**懒汉式使用时间换取空间**：即需要该类实例时才创建，节约内存，但首次获取需要时间。另外它有并发缺陷，**多线程情况下懒汉式不能保证单例**。

- 多线程测试

```java
public class SingletonTest {
    public static void main(String[] args) throws InterruptedException {
        Set<Singleton1> set = new HashSet<>();
        // Windows每个进程最多1000个线程，Linux每个进程最多2000个线程
        for (int i = 0; i < 2000; i++) {
            new Thread(() -> {
                set.add(Singleton1.getInstance());
            }).start();
        }
        // main线程等待15秒，让2000个线程尽量都执行完毕
        Thread.sleep(15000);
        System.out.println("懒汉式单例模式测试-----");
        for (Singleton1 singleton1 : set) {
            System.out.println(singleton1);
        }
    }
}
```

> 经多次测试后，得到下述结果（多测试几次才可能出现）

```
单例模式测试-----
com.baicai.singleton.Singleton1@2cb68cf7
com.baicai.singleton.Singleton1@1588196a
```

高并发时，假设线程1进入getInstance()，判断引用为null，准备进入if块内执行实例化。此时该线程突然让出时间片，线程2进入方法，此时引用还是null。于是线程2进入if块执行实例化，线程1唤醒后也会进入if块进行实例化。结果就会出现2个实例。

> 懒汉式不能保证单例，如何优化？

## 2.同步懒汉式

在懒汉式的基础上，只需要将getInstance()同步

```java
public class Singleton3 {
    /** 引用 */
    private static Singleton3 singleton3;

    /** 1.私有构造 */
    private Singleton3() {}

    /** 2.获取实例的方法 */
    public synchronized static Singleton3 getInstance() {
        if (singleton3 == null) {
            singleton3 = new Singleton3();
        }
        return singleton3;
    }
}
```

使用synchronized可以保证getInstance线程安全，保证同时只有一个线程进入此方法。但该方式使得每个线程都要阻塞排队，包括实例初始化之后进入方法的线程们。实际上实例初始化后，就不需要做同步控制了，否则阻塞会影响性能。继续优化...

> **懒汉式的并发BUG，问题点在于第一次创建实例时的并发控制，而非每个时刻getInstance方法级的同步控制**

## 3.DCL方式

DCL（double checked locking），即双重检验锁

```java
public class Singleton4 {
    /** 引用，volatile防止编译期流程优化 */
    private static volatile Singleton4 singleton4;

    /** 1.私有构造 */
    private Singleton4() {}

    /** 2.获取实例的方法 */
    public static Singleton4 getInstance() {
        if (singleton4 == null) {
            synchronized (Singleton4.class) {
                if (singleton4 == null) {
                    singleton4 = new Singleton4();
                }
            }
        }
        return singleton4;
    }
}
```

同步范围由方法级降为代码块级，允许多线程进入getInstance()。当引用不为null时直接返回，大部分情况也是如此，取消该部分的阻塞解决了同步懒汉式的性能问题。一开始当引用为null时，多线程进入第一层if，随后只能有一个线程进入同步代码块，完成实例的创建。其他已经进入第一层if的线程们，也会先后进入同步块，所以需要第二层if避免重复创建实例。引用不为null后访问的线程，会直接返回实例，避阻塞，解决了并发下线程安全和性能的平衡。

- 为什么**同时使用volatile修饰引用**

JVM初始化一个对象，总的来说做了3件事情：

- 1.在堆空间分配内存
- 2.执行构造方法进行初始化
- 3.将引用指向内存中分配的空间地址

但编译时，编译器在生成汇编代码时会对流程进行优化（这里涉及到happen-before原则和Java内存模型和CPU流水线执行的知识，略），优化的结果可能按123顺序执行，也有可能按132执行。假设按照132顺序执行，执行完第3步（还没到第2步）时，此时其他线程可能同时访问getInstance()，走到第一层if时，发现single4已经不是null了，就直接返回了，但是此时对象还没有完成初始化。如果该线程此时立刻访问实例的某些需要初始化的参数时，就有可能报错。使用volatile关键字，能够告诉编译器不要对代码进行重排序优化，避免上述问题。

> 该方式既能保证多线程下的单例，又有懒加载，性能上也做了优化，是近乎完美的懒汉式（后面还可以对反序列化情景下进行优化）

## 4.饿汉式

饿汉式，与懒汉式相比，在类加载中就创建好实例，有两种变种：静态成员变量、静态代码块。

```java
public class Singleton2 {
    /** 引用，类加载时就创建实例 */
    private final static Singleton2 singleton2 = new Singleton2();
    
    /** 1.私有构造 */
    private Singleton2() {}

    /** 2.获取实例的方法 */
    public static Singleton2 getInstance() {
        return singleton2;
    }
}
```

```java
public class Singleton2 {

    private Singleton2 singleton2 = null;

    static {
        singleton2 = new Singleton2();
    }
    
    /** 1.私有构造 */
    private Singleton2() {}

    /** 2.获取实例的方法 */
    public static Singleton2 getInstance() {
        return singleton2;
    }
}
```

与懒汉式相比，**饿汉式使用空间换取时间**：即加载类字节码到JVM时就实例化该类对象，提前消耗内存，没有并发问题，能保证实例唯一（类加载时的单例由JVM的类加载机制保证）

#### JVM如何保证饿汉式的线程安全

在编译生成class文件时，会自动产生两个方法，类的初始化方法clinit，和实例的初始化方法init

| |作用|调用时机|
|:---|:----|:--|
|clinit方法|类的初始化方法，包括静态变量初始化语句和静态块的执行|在jvm第一次加载class文件时被调用|
|init方法|实例的初始化方法|在创建实例时被调用，发生在new操作符、调用Class或Java.lang.reflect.Constructor对象的newInstance()、调用任何现有对象的clone()、通过java.io.ObjectInputStream类的getObject()方法反序列化时|

> clinit方法是由编译器在编译期，自动收集类中的所有类变量（静态成员变量）的赋值动作和静态代码块（static{}）中的语句合并产生的。因此，private static Singleton2 singleton2 = new Singleton2();也会被放入到这个方法中

类生命周期的7个阶段：【加载】【验证】【准备】【解析】【初始化】【使用】【卸载】，前5个为类加载阶段。静态成员变量singleton2的初始化语句就发生在类Singleton2类加载的初始化阶段，初始化阶段是执行类构造器clinit方法的过程。

> Java虚拟机规范中没有约束何时进行第一个阶段加载，即交由虚拟机具体实现。但对初始化阶段，虚拟机规范严格规定了有且只有以下5种情况必须立即对类进行初始化（而加载、验证、准备自然需要在此之前完成）：
> - 1.遇到new、getstatic、putstatic、invokestatic这4条字节码指令时，如果类没有进行过初始化，需要先触发其初始化。生成这4条指令的最常见场景：使用new关键字、读取或设置类静态字段（被fina修饰的静态字段除外，其已在编译期把值放入了常量池中）、调用类静态方法
> - 2.使用java.lang.reflect包的方法对类进行反射时，如果类没有进行过初始化，需要先触发其初始化
> - 3.初始化一个类的时候，如果其父类还没有进行初始化，由先触发其父类的初始化
> - 4.当虚拟机启动时，用户需要指定一个执行主类（包含main()方法的那个类），虚拟机会先初始化这个主类
> - 5.当使用JDK 1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getstatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化

虚拟机会保证一个类的clinit方法在多线程环境中被正确的加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程执行该类的clinit方法，其他线程阻塞，直到活动线程执行完clinit方法。其他线程虽然被阻塞，但执行clinit方法的那条线程退出clinit方法后，其他线程唤醒后不会再进入clinit方法。**同一个类加载器下，一个类只会初始化clinit一次。即private static Singleton2 singleton2 = new Singleton2()只执行了一次**

> 如果对性能没有要求，可以使用饿汉式，简单方便。如果既要保证性能（不提前消耗内存），又要线程安全，向下看...

## 5.饿汉式-静态内部类

静态内部类是懒加载的，所以使用静态内部类改造饿汉式。既能不浪费内存，又保证了线程安全。

> 虚拟机的机制是，如果没有访问一个类，那么不会载入该类进入虚拟机。当使用外部类的其他属性时，是不会加载内部类到内存的，也就不会浪费内存创建内部类中的单例。

```java
public class Singleton5 {
    /** 1.私有构造 */
    private Singleton5() {}

    /** 2.获取实例的方法 */
    public static Singleton5 getInstance() {
        return Inner.singleton5;
    }

    /** 静态内部类，即能保证懒加载，又能保证线程安全 */
    private static class Inner {
        private static final Singleton5 singleton5 = new Singleton5();
    }
}
```

以上5种方式，可分为懒汉式（1\2\3）和饿汉式（4\5），以及对两者的优化。其中【3.双重检验锁】和【5.静态内部类】能保证多线程单例和懒加载。

> 上述看似完美的实现方式，在反射和反序列化时会被破坏，所以接着讨论...

## 6.反射和反序列化破坏单例

Java反射几乎什么事情都能做，私有的公有的都能破坏。而反序列化也很厉害，一旦实现了序列化接口，该类将不能保持单例，readObject()会返回一个新的实例。

> 下面的Singleton6和Singleton5一样，使用静态内部类实现。然后使用反射和反序列化进行单例破坏

```java
public class Singleton6 implements Serializable{
    /** 1.私有构造 */
    private Singleton6() {}

    /** 2.获取实例的方法 */
    public static Singleton6 getInstance() {
        return Inner.singleton6;
    }

    /** 静态内部类，即能保证懒加载，又能保证线程安全 */
    private static class Inner {
        private static final Singleton6 singleton6 = new Singleton6();
    }  
}
```

```java
/** 测试反射和反序列化破坏单例 */
public static void main(String[] args) throws IllegalAccessException, InstantiationException, IOException, ClassNotFoundException {
    // 正常获取实例
    Singleton6 singleton6 = getInstance();
    // 反射获取实例
    Singleton6 singleton6Two = Singleton6.class.newInstance();
    // 反序列化获取实例
    String file = "singleton6";
    ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream(file));
    outputStream.writeObject(singleton6);
    ObjectInputStream inputStream = new ObjectInputStream(new FileInputStream(file));
    Singleton6 singleton6Three = (Singleton6) inputStream.readObject();

    /* 比较3个实例 */
    System.out.println(singleton6 + "\n" + singleton6Two + "\n" + singleton6Three);
}
```

> 反射和反序列化得到的实例，和原实例不同，即破坏了单例

```
com.baicai.singleton.Singleton6@135fbaa4
com.baicai.singleton.Singleton6@5fd0d5ae
com.baicai.singleton.Singleton6@2d98a335
```

#### 解决反序列化的破坏

反序列化破坏单例，可以通过**重写readResolve()**遏制，即在上述Singleton6中重写：

```java
/** 重写该方法，防止反序列化破坏单例 */
public Object readResolve() {
    return Inner.singleton6;
}
```

> 再次测试，发现反序列化的实例已与原实例一致（反射得到的仍不一致）

```java
com.baicai.singleton.Singleton6@135fbaa4
com.baicai.singleton.Singleton6@5fd0d5ae
com.baicai.singleton.Singleton6@135fbaa4
```

反序列化时，如果实现了serializable或externalizable接口的类中包含readResolve()方法，会直接调用readResolve()获取实例，这样就避免了反序列化破坏单例。

> 至此，【3.双重检验锁】或【5.静态内部类】的方式+重写readResolve()是最接近完美的（除了反射场景）

## 7.枚举方式

#### 枚举

具体可参考[Enum](https://blog.wocaishiliuke.cn/javase/2018/01/19/javase_enum/)。

- enum本质是一个final的Enum子类，即默认继承了java.lang.Enum（间接继承了Object）
- 枚举类可以实现一个或多个接口
- 枚举类的构造只能是私有的

枚举可以实现单例，正是因为枚举本身的特点：
- 本质是类，可以有成员变量、成员方法等
- 构造都是私有的（默认或手动给出的）
- 创建枚举默认是线程安全的（因为是静态的）

#### 实现

```java
public enum Singleton7 {
    //实例
    SINGLETON7;

    //也可以省略该方法，使用Singleton7.SINGLETON7访问
    public Singleton7 getInstance() {
        return SINGLETON7;
    }
}
```

> 枚举值SINGLETON7就相当于：

```java
public static final Singleton7 SINGLETON7 = new Singleton7("SINGLETON7",0);
```

通过反编译可以看出，枚举的本质就是一个继承了java.lang.Enum的final类

```java
javap -c src/com/baicai/singleton/Singleton7.class 
Compiled from "Singleton7.java"
public final class com.baicai.singleton.Singleton7 extends java.lang.Enum<com.baicai.singleton.Singleton7> {
  public static final com.baicai.singleton.Singleton7 SINGLETON7;

  public static com.baicai.singleton.Singleton7[] values();
    Code:
       0: getstatic     #1                  // Field $VALUES:[Lcom/baicai/singleton/Singleton7;
       3: invokevirtual #2                  // Method "[Lcom/baicai/singleton/Singleton7;".clone:()Ljava/lang/Object;
       6: checkcast     #3                  // class "[Lcom/baicai/singleton/Singleton7;"
       9: areturn

  public static com.baicai.singleton.Singleton7 valueOf(java.lang.String);
    Code:
       0: ldc           #4                  // class com/baicai/singleton/Singleton7
       2: aload_0
       3: invokestatic  #5                  // Method java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
       6: checkcast     #4                  // class com/baicai/singleton/Singleton7
       9: areturn

  public com.baicai.singleton.Singleton7 getInstance();
    Code:
       0: getstatic     #7                  // Field SINGLETON7:Lcom/baicai/singleton/Singleton7;
       3: areturn

  static {};
    Code:
       0: new           #4                  // class com/baicai/singleton/Singleton7
       3: dup
       4: ldc           #8                  // String SINGLETON7
       6: iconst_0
       7: invokespecial #9                  // Method "<init>":(Ljava/lang/String;I)V
      10: putstatic     #7                  // Field SINGLETON7:Lcom/baicai/singleton/Singleton7;
      13: iconst_1
      14: anewarray     #4                  // class com/baicai/singleton/Singleton7
      17: dup
      18: iconst_0
      19: getstatic     #7                  // Field SINGLETON7:Lcom/baicai/singleton/Singleton7;
      22: aastore
      23: putstatic     #1                  // Field $VALUES:[Lcom/baicai/singleton/Singleton7;
      26: return
}
```

#### 测试

使用上述6中的代码进行枚举实现单例的反射和反序列化验证

```java
public static void main(String[] args) throws IllegalAccessException, InstantiationException, IOException, ClassNotFoundException {
    // 获取枚举单例
    Singleton7 singleton7 = Singleton7.SINGLETON7;
    // 查看构造
    Constructor<?>[] constructors = Singleton7.class.getDeclaredConstructors();
    /* 反射获取实例 */
    Singleton7 singleton7Two = Singleton7.class.newInstance();

    /* 反序列化获取实例 */
    String file = "singleton7";
    ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream(file));
    outputStream.writeObject(singleton7);
    ObjectInputStream inputStream = new ObjectInputStream(new FileInputStream(file));
    Singleton7 singleton7Three = (Singleton7) inputStream.readObject();

    /* 比较3个实例 */
    System.out.println(singleton7 + "\n" + singleton7Two + "\n" + singleton7Three);
}
```

> 报错，debug查看Singleton7的构造器，发现没有空参构造，只有一个有参构造，然后看下Enum源码，得知这两个参数是为了初始化父类的name和ordial两个属性

```java
Exception in thread "main" java.lang.InstantiationException: com.baicai.singleton.Singleton7
    at java.lang.Class.newInstance(Class.java:427)
    at com.baicai.singleton.Singleton7.main(Singleton7.java:23)
Caused by: java.lang.NoSuchMethodException: com.baicai.singleton.Singleton7.<init>()
    at java.lang.Class.getConstructor0(Class.java:3082)
    at java.lang.Class.newInstance(Class.java:412)
    ... 1 more
```

```java
private com.baicai.singleton.Singleton7(java.lang.String,int)
```

> 既然没有空参构造，就使用有参构造反射：

```java
/* 反射获取实例 */
//没有空参，该方式报错
//Singleton7 singleton7Two = Singleton7.class.newInstance();

Constructor<Singleton7> constructor = Singleton7.class.getDeclaredConstructor(String.class, int.class);
constructor.setAccessible(true);
Singleton7 singleton7Two = constructor.newInstance("test", 666);
```

> 结果仍然报错：**Cannot reflectively create enum objects**（不能反射创建枚举实例）

```java
Exception in thread "main" java.lang.IllegalArgumentException: Cannot reflectively create enum objects
    at java.lang.reflect.Constructor.newInstance(Constructor.java:417)
    at com.baicai.singleton.Singleton7.main(Singleton7.java:32)
```

> 追根溯源，查看Constructor类的newInstance()源码

可以看到，**反射在通过newInstance创建对象时，会检查该类是否是ENUM，并禁止使用反射创建枚举实例**。

```java
/**
 * @exception IllegalArgumentException if this constructor pertains to an enum type.
 */
@CallerSensitive
public T newInstance(Object ... initargs)
    throws InstantiationException, IllegalAccessException,
           IllegalArgumentException, InvocationTargetException
{
    if (!override) {
        if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz, null, modifiers);
        }
    }
    if ((clazz.getModifiers() & Modifier.ENUM) != 0)
        throw new IllegalArgumentException("Cannot reflectively create enum objects");
    ConstructorAccessor ca = constructorAccessor;   // read volatile
    if (ca == null) {
        ca = acquireConstructorAccessor();
    }
    @SuppressWarnings("unchecked")
    T inst = (T) ca.newInstance(initargs);
    return inst;
}
```

#### 优点

使用枚举方式实现单例的优点
- 1.线程安全，保证单例（由类加载时JVM保证）
- 2.代码简单，充分利用了枚举的特点
- 3.自由序列化。对于序列化和反序列化，因为每个枚举类和枚举变量在JVM中都是唯一的，Java在序列化和反序列化枚举时做了特殊的规定，枚举的writeObject、readObject、readObjectNoData、writeReplace和readResolve等方法是被编译器禁用的，因此不存在实现序列化接口后调用readObject破坏单例的情况
- 4.可以防止反射破坏单例

> 只是该类不那么"干净"，继承了Enum的成员，如name、ordinal、name()等。

## 8.CAS

CAS是种乐观锁技术，当多线程尝试更新同一变量时，CAS能保证只有一个线程更新成功，其他线程失败，失败的线程并不会被挂起，而是被告知失败，并可以再次尝试。

```java
public class Singleton8 {
    private static final AtomicReference<Singleton8> INSTANCE = new AtomicReference();

    private Singleton8() {}

    public static Singleton8 getInstance() {
        for (;;) {
            Singleton8 singleton8 = INSTANCE.get();
            if (null != singleton8)
                return singleton8;

            singleton8 = new Singleton8();
            if (INSTANCE.compareAndSet(null, singleton8))
                return singleton8;
        }
    }
}
```
> for (;;)和while(true)，Java编译器都会优化成goto，即编译优化后两者效果和性能一样。但对于c语言，for语句会被编译器优化成一条汇编指令，而while会生成好几条汇编指令。

> 优缺点

- 优点
    + 优于传统锁机制（如synchronized），CAS基于忙等待算法，依赖底层硬件实现，没有线程切换和阻塞带来的消耗，支持较大的并行度
- 缺点
    + 如果忙等待一直执行不成功（一直死循环），会增加CPU负担
    + 当很多线程同时执行到new Singleton8()时，会创建很多对象，容易溢出

不建议CAS方式。


