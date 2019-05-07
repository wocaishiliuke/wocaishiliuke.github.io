---
title: Enum
date: 2018-01-19 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Java中枚举的相关知识。

<!-- more -->

##### 目录
- I.简介
- II.实现原理
- III.应用场景
- IV.枚举容器
- V.参考

---
# I.简介

## 1.背景

在Java 5引入枚举类之前，通常用一组int常量表示枚举（int枚举模式）。

```java
public class Sex {
    public static final int MALE = 1;
    public static final int FEMALE = 2;
    // 获取code对应的描述
    public static String getDesc(int sexCode) {
        String desc = null;
        switch (sexCode) {
            case 1:
                desc = "MALE";
                break;
            case 2:
                desc = "FEMALE";
                break;
        }
        return desc;
    }
}
```

```java
// 调用
String desc0 = Sex.getDesc(Sex.MALE); // MALE
String desc1 = Sex.getDesc(3);        // null
```

该方式的缺点：
- 安全性：无法保证传入值为合法，不安全
- 可读性：差

枚举应运而生。

## 2.定义

枚举是一组固定常量组成的数据类型，比如性别、星期、四季等。Java中使用键字enum来定义枚举。

```java
public enum SexEnum {
    MALE,FEMALE
}
```

> 枚举可以单独定义成一个类文件，也可以定义成内部类

## 3.特性

枚举是一个特殊的类型，既有普通类的性质，又有自己特性。

- 枚举类的name()和toString()返回值相同
- 枚举类的equals()和==效果一样
- 枚举类的ordinal()用于获取枚举值的声明顺序
- 枚举类实现了Comparable，枚举值之间可以通过compareTo()比较，比较的就是ordinal()声明顺序

```java
// 调用
public static void main(String[] args) {
    SexEnum sex = SexEnum.MALE;
    //枚举类中，name和toString返回内容一样
    System.out.println(sex.toString()); //MALE
    System.out.println(sex.name());     //MALE

    //枚举类中 == 和 equals效果一样，直接使用 == 即可
    System.out.println(sex == SexEnum.MALE);        //true
    System.out.println(sex.equals(SexEnum.MALE));   //true
    System.out.println(sex == SexEnum.FEMALE);      //false

    //ordinal()方法，表示枚举值声明的顺序，从0开始
    System.out.println(sex.ordinal());              //0
    System.out.println(SexEnum.FEMALE.ordinal());   //1

    //枚举类中实现了Comparable接口，枚举值之间可以通过compareTo比较，比较的就是ordinal()
    System.out.println(sex.compareTo(SexEnum.FEMALE)); //-1
}
```

## 4.隐藏方法

编译器会为枚举类自动生成的两个比较重要的方法：values、valueOf。

```java
// values()：获取所有枚举值
for (SexEnum s : SexEnum.values()) {
    System.out.println(s.name());
}

// valueOf(String)：根据枚举名称反查枚举值。名称不存在时，报IllegalArgumentException
System.out.println(SexEnum.valueOf("MALE")); //MALE
```

## 5.优点

- 枚举类是类型安全的，一个枚举类型的变量，要么为null，要么为枚举值之一，不可能为其他值
- 枚举类自带很多便利方法，如values, valueOf, toString等，方便使用


---
# II.实现原理

## 1.Enum

枚举类也是一种类，只不过声明方式跟普通类不同。枚举类会被compiler编译为一个普通类，该类继承了java.lang.Enum类。

```java
public abstract class Enum<E extends Enum<E>> implements Comparable<E>, Serializable {
    // 枚举名
    private final String name;
    // 声明顺序
    private final int ordinal;

    public final String name() {
        return name;
    }

    public final int ordinal() {
        return ordinal;
    }

    protected Enum(String name, int ordinal) {
        this.name = name;
        this.ordinal = ordinal;
    }

    public String toString() {
        return name;
    }

    public final boolean equals(Object other) {
        return this==other;
    }

    public final int hashCode() {
        return super.hashCode();
    }
    // 枚举不能被克隆
    protected final Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }

    public final int compareTo(E o) {
        Enum<?> other = (Enum<?>)o;
        Enum<E> self = this;
        if (self.getClass() != other.getClass() && // optimization
            self.getDeclaringClass() != other.getDeclaringClass())
            throw new ClassCastException();
        return self.ordinal - other.ordinal;
    }

    public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                                String name) {
        T result = enumType.enumConstantDirectory().get(name);
        if (result != null)
            return result;
        if (name == null)
            throw new NullPointerException("Name is null");
        throw new IllegalArgumentException(
            "No enum constant " + enumType.getCanonicalName() + "." + name);
    }

    //……
}
```

Enum中有两个重要的成员变量name、ordinal，类中的方法name()、toString()、ordinal()、compareTo()、 equals()都是操作这两个成员变量的。

Enum类因为要实现Comparable接口，所以需要定义成泛型类。那为什么不定义成如下形式呢？
```java
public abstract class Enum<E> extends Object implements Comparable<E>,Serializable
```

因为Comparable.compareTo()是这样声明的：public final int compareTo(E o)，如果定义成上述那样，泛型擦除后参数就会变为Object，枚举值调用compareTo就可以任意对象进行比较，没有意义。所以需要上界限定，这样被擦除之后，E被替换成Enum，compareTo()也就只能和枚举比较了。

## 2.反编译

枚举类被编译后生成的class继承了 Enum类，反编译SexEnum的字节码：

```java
public final class SexEnum extends Enum<SexEnum> {
    public static final SexEnum MALE = new SexEnum("MALE",0);
    public static final SexEnum FEMALE = new SexEnum("FEMALE",1);
    
    private static SexEnum[] VALUES = new SexEnum[]{MALE,FEMALE};
    
    private SexEnum(String name, int ordinal){
        super(name, ordinal);
    }
    
    public static SexEnum[] values(){
        SexEnum[] values = new SexEnum[VALUES.length];
        System.arraycopy(VALUES, 0, values, 0, VALUES.length);
        return values;
    }
    
    public static SexEnum valueOf(String name){
        return Enum.valueOf(SexEnum.class, name);
    }
}
```

- 枚举类私有了构造（有参），不能在外部创建实例。该构造将name和ordinal传递给父类
- 枚举值实际是final和static的，不能被修改
- values()是编译器添加的，内部有一个values数组，存有所有枚举值
- valueOf()也是编译器添加的，调用了父类Enum的valueOf()，SexEnum.class参数用于反射


---
# III.应用场景

## 1.表示常量

最基本的用法

```java
public enum Size {  
  SAMLL, MEDIUM, LARGE, EXTRA_LARGE
}
```

## 2.自定义枚举属性

上面SexEnum中，只使用了父类的name、ordinal属性，但声明顺序ordinal的值是0,1,2...。通常不符合业务场景，此时就需要自定义属性。

```java
public enum Season {
    SPRING(1, "春天"),
    SUMMER(2, "夏天"),
    AUTUMN(3, "秋天"),
    WINTER(4, "冬天");

    // 成员变量  
    private int code;
    private String desc;

    // 构造方法，默认是private的
    Season(int code, String desc) {
        this.code = code;
        this.desc = desc;
    }

    // 获取描述  
    public static String getDesc(int code) {
        for (Season s : Season.values()) {
            if (s.getCode() == code)
                return s.getDesc();
        }
        return null;
    }

    // getter
    public int getCode() { return code; }
    public String getDesc() { return desc; }
}
```

这里构造器是加修饰符，默认是private的。编译器会默认生成一个私有的构造函数，如下：

```java
private Season(String name, int ordinal, int code, String desc) {
    super(name, ordinal);
    this.code = code;
    this.desc = desc;
}
```

## 3.switch语句

枚举变量可以和普通类型的变量一样使用，可作为类变量、成员变量、方法形参等，还可以用于switch语句。

```java
public static String getSexDesc(SexEnum sex) {
    String sexDesc = null;
    switch (sex) {
        case MALE:
            sexDesc = "Male";
            break;
        case FEMALE:
            sexDesc = "Female";
            break;
        default:
            sexDesc = "Not Exist";
            break;
    }
    return sexDesc;
}
```

和上述《背景》小节中的getDesc()相比，使用了枚举的getSexDesc()明显更安全。因为入参限定了只能是Sex.MALE或者Sex.Female，永远走不到default逻辑。另外，在switch语句内部，枚举值不能带枚举类型前缀，例如直接使用MALE，不能使用SexEnum.MALE。

## 4.枚举类中定义抽象方法

枚举类中可以声明抽象方法，每个枚举值中都需要实现该方法。常用来实现不同枚举值的特有行为。

```java
public enum SexEnum {
    MALE {
        @Override
        public void doSth() {
            System.out.println("打猎");
        }
    },FEMALE {
        @Override
        public void doSth() {
            System.out.println("织布");
        }
    };
    public abstract void doSth();
}
```

> 使用

```java
SexEnum sex = SexEnum.FEMALE;
switch (sex) {
    case MALE:
        sex.doSth();
        break;
    case FEMALE:
        sex.doSth();
        break;
}
```

## 5.接口组织枚举

```java
public interface Food {
    enum Coffee implements Food{  
        BLACK_COFFEE,DECAF_COFFEE,LATTE,CAPPUCCINO  
    }  
    enum Dessert implements Food{  
        FRUIT, CAKE, GELATO  
    }  
}
```

> 使用

```java
Food.Coffee coffee = Food.Coffee.BLACK_COFFEE;
Food.Dessert dessert = Food.Dessert.CAKE;
```

## 6.单例

```java
public enum Singleton {
    //实例
    INSTANCE;

    //也可以省略该方法，使用Singleton.INSTANCE访问
    public Singleton getInstance() {
        return INSTANCE;
    }
}
```

关于单例，更多可参考[设计模式-单例模式](https://blog.wocaishiliuke.cn/designpatterns/2019/01/02/Singleton/)。


---
# IV.枚举容器

枚举类存在两种容器，EnumSet和EnumMap。EnumSet 是一个枚举集合，EnumMap是一个使用枚举值做key的Map实现。相比于普通Set集合和Map，EnumSet和EnumMap处理枚举类型数据更高效。

## 1.EnumSet

EnumSet枚举集合，是一个抽象类。继承了JumboEnumSet和RegularEnumSet。

与HashSet基于HashMap实现不同，EnumSet的实现与EnumMap没有关系。EnumSet使用极简和高效的位向量实现，效率高于HashSet。EnumSet内部有很多静态工厂方法可以用来构造EnumSet对象（具体可查看源码）：

```java
// 初始集合包括指定枚举类型的所有枚举值
public static <E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType)
// 初始集合包括枚举值中指定范围的元素
public static <E extends Enum<E>> EnumSet<E> range(E from, E to)
// 初始集合包括指定集合的补集
public static <E extends Enum<E>> EnumSet<E> complementOf(EnumSet<E> s)
// 初始集合包括参数中的所有元素
public static <E extends Enum<E>> EnumSet<E> of(E e)
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2)
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3)
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4)
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4, E e5)
public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest)
// 初始集合包括参数容器中的所有元素
public static <E extends Enum<E>> EnumSet<E> copyOf(EnumSet<E> s)
public static <E extends Enum<E>> EnumSet<E> copyOf(Collection<E> c)
```

> 使用

```java
//创建一个包含指定元素类型的所有元素的枚举 set
EnumSet<SexEnum> setAll = EnumSet.allOf(SexEnum.class);
//创建一个指定范围的Set
EnumSet<SexEnum> setRange = EnumSet.range(SexEnum.MALE,SexEnum.FEMALE);
//创建一个指定枚举类型的空set
EnumSet<SexEnum> setEmpty = EnumSet.noneOf(SexEnum.class);
//复制一个set
EnumSet<SexEnum> setNew = EnumSet.copyOf(setRange);
另外Set集合的一些基础操作，EnumSet也是支持的，比如交集、差集操作。
```

## 2.EnumMap

EnumMap是一个使用枚举值做key的Map实现，可以通过如下方式构建EnumMap实例（具体可查看源码）：

```java
//创建一个具有指定键类型的空枚举映射
EnumMap<SexEnum, String> map1 = new EnumMap<>(SexEnum.class);
//从一个 EnumMap 创建
EnumMap<SexEnum, String> map2 = new EnumMap<SexEnum, String>(map1);
//从一个 Map 创建
Map<SexEnum, ? extends String> map3 = new HashMap<>();
EnumMap<SexEnum, String> map4 = new EnumMap<SexEnum, String>(map3);
```


---
# V.参考

-[Enum Types](https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html)
-[Java编程拾遗『枚举类』](http://lidol.top/java/864/)