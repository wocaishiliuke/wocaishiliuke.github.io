---
title: 设计模式-汇总
date: 2019-01-01 19:02:09
categories:
    - DesignPatterns
tags:
    - DesignPatterns

---

本文将介绍设计模式的6大原则，并持续汇总各设计模式的具体讲解。

<!-- more -->

##### 目录
+ I.简介
+ II.汇总


# I.简介

GOF"四人帮"设计模式，分为3大类23种，详见下述。江湖上还有其他设计模式，如简单工厂、J2EE模式（更关注表示层）等。

## 1.原则

#### 0.OCP

设计模式遵循的总原则是OCP开闭原则：**Software entities like classes, modules and functions should be open for extension but closed for modifications**。要实现OCP，一般需要（抽象化）使用接口和抽象。下面举例说明OCP：

###### 不遵循OCP 

```java
// Open-Close Principle - Bad example
class GraphicEditor {
    public void drawShape(Shape s) {
        if (s.m_type==1)
            drawRectangle(s);
        else if (s.m_type==2)
            drawCircle(s);
    }
    public void drawCircle(Circle r) {...}
    public void drawRectangle(Rectangle r) {...}
}
 
class Shape {
    int m_type;
}
 
class Rectangle extends Shape {
    Rectangle() {
        super.m_type=1;
    }
}
 
class Circle extends Shape {
    Circle() {
        super.m_type=2;
    }
} 
```

上述代码，没有遵循OCP，每当增加/删除Shape时，GraphicEditor就需要修改。有以下缺点：

- 每次新增Shape，就需要重新对GraphicEditor单元测试
- 增加Shape时需要花费大量时间，因为开发人员必须熟悉GraphicEditor的代码逻辑
- 即使新增的Shape运行正常，它也可能影响已有的功能

GraphicEditor往往是个大Class，由多人维护，而Shape往往只有一个维护，理想的情况是：增加Shape，不需要修改GraphicEditor。

###### 遵循OCP 

把GraphicEditor中的drawShape()抽象出来，将方法实现转移到具体Shape类中。此时增加新Shape，GraphicEditor不需要再改动。

```java
// Open-Close Principle - Good example
class GraphicEditor {
    public void drawShape(Shape s) {
        s.draw();
    }
}
 
class Shape {
    abstract void draw();
}
 
class Rectangle extends Shape  {
    public void draw() {
        // draw the rectangle
    }
} 
```

- 增加后，不需要单元测试
- 修改前，不需要知道GraphicEditor的逻辑
- 由于方法实现转移到了下层的具体Shape类，降低了对原有功能的影响

OCP只是一个原则，它使得代码灵活的同时，也会花费更多的时间和精力。所以它适合有可能改动的代码（实际开发中，大部分都可能改动）。其他原则如装饰器模式、工厂方法模式、观察者模式等都可以解决避免修改的问题，即他们遵循了OCP

#### 1.单一职责原则

Single Responsibility Principle

一个类只负责一个功能领域中的相应职责，换言之，一个类，应该只有一个引起它变化的原因。该原则用于控制类的粒度。

类不能"太累"，越多的功能，意味着复用的可能性就越小，这些功能也越容易耦合在一起。一个职责变化就可能影响其他职责，所以需要拆分。单一职责原则是实现高内聚、低耦合的指导方针。

比如UserDao中一般只有增删改查的方法，获取数据库连接的方法放在DBUtil类中。

方法中也同样适用，比如某接口根据传入参数changeType判断，进行密码或地址修改操作。应该拆成两个接口方法。

#### 2.里氏替换原则

Liskov Substitution Principle

所有引用父类的地方，必须能透明地使用其子类的对象，并且替换为子类不会产生任何错误或异常，反之不一定。里氏替换原则包含4层含义：

- 子类必须实现父类中的所有抽象方法，但不能覆写父类非抽象方法
- 子类可以有自己的特有方法
- 当子类实现父类的抽象方法时，方法的形参要比父类的更宽松
- 当子类实现父类的抽象方法时，方法的返回值要比父类的更严格

> 跟多态类似。比如很多接口方法的形参使用父类接收，传参时传入子类实参。即遵训了里氏替换原则，又是多态的体现。

#### 3.依赖倒置原则

Dependency Inversion Principle

**面向接口/抽象类编程**，而非面向实现编程

> 倒置是指：本该依赖具体实现类的，但却依赖父接口或抽象类。**即定义依赖时使用接口或抽象类，运行传参时，才传入实现类实例。**

该原则要求，定义传参或关联关系时，尽量使用高层的接口或抽象类。即声明变量、方法形参、方法返回值时，都用接口或抽象类，不直接使用具体类型（多态），面向接口编程。可以将具体类型写入配置文件，当需要修改或增删功能时，只需要修改配置，更灵活，也满足OCP。

依赖注入(DependencyInjection, DI)是指一个对象需要其他对象时，就发生了依赖关系。（Java对象除了继承和实现关系，就是依赖关系了，否则这两个对象间没关系）

常用的依赖注入方式有3种：

- 构造注入：通过构造函数传入具体类对象
- Setter注入：通过Setter()传入具体类对象
- 接口注入：通过在接口中声明的业务方法来传入具体类的对象。

这些方法在定义时使用的是抽象类型，在运行时再传入具体类型的对象

```java
// 接口
interface Car{
    public void run();
}

//实现类
class Benz implements Car{
    public void run(){
        System.out.println("benz run...");
    } 
}

// 构造注入
class Driver{
    private Car car;
    Driver(Car car){
        this.car = car;
    }
    public void drive(){
        this.car.run();
    }
}

// setter注入
class Driver{
    private Car car;
    public void setCar(Car car){
        this.car = car;
    }
    public void drive(){
        this.car.run();
    }
}

// 接口注入
class Driver{
    public void drive(Car car){
        car.run();
    }
}
```

#### 4.接口分离原则

Interface Segregation Principle

一个接口太大时，需要分割成一些更细小的接口，使用该接口的客户端仅需知道与之相关的方法即可，不应该依赖不需要的接口。用于控制接口的粒度

该原则鼓励使用多个专门的接口，而非单一庞大的总接口。庞大的接口必然导致实现类的臃肿，灵活性差。一般而言，一个接口提供一个角色的功能即可，为客户端提供"定制服务"。（和单一职责类似）

> 接口尽量小，也不能太小。过多的接口会使得系统结构复杂，不易于维护。所以应根据业务把握度。


#### 5.迪米特法则

Law of Demeter，又称为最少知识原则(LeastKnowledge Principle，LKP) 

一个实体应当尽可能少地与其他实体发生相互作用。用于限制实体间的耦合度。

尽量减少对象之间的通信交互。当两个对象没必要直接通信时，就不必直接耦合，可以通过中间者调用。

比如，教导主任A想要点名三年级二班人数，就命令班主任B，班主任B又命令班长周某C进行点名。该例中，A不知道班长是谁，也不需要知道，就不需要直接调用C，通过中间者B进行实现。

```java
// 教导主任（A和C耦合）
class A {
    public void count() {
        B b = new B();
        C c = B.getC();
        System.out.println(c.count());
    }
}
// 班主任
class B {
    public C getC() {
        return new C();
    }
}

// 班长
class C {
    public int count() {
        return 30;
    }
}
```

解耦A和C：

```java
// 教导主任（A和C耦合）
class A {
    public void count() {
        B b = new B();
        System.out.println(b.count());
    }
}
// 班主任
class B {
    public int count() {
        return new C().count();
    }
}

// 班长
class C {
    public int count() {
        return 30;
    }
}
```

#### 6.合成复用原则

Composite Reuse Principle

尽量使用合成/聚合的方式，而非继承（聚合优先于继承）。继承实际上破坏了类的封装性，超类的方法可能会被子类修改，并且耦合度高。

> C语言中，合成是值的聚合(Aggregation by Value)，聚合是则是引用的聚合(Aggregation by Reference)

合成（Composition）和聚合（Aggregation），都是关联的特殊种类。
- 聚合表示弱拥有，体现的是A对象可以包含B对象，但B对象不是A对象的一部分。如狼和狼群（集合或数组）。
- 合成表示强拥有，体现了严格的部分和整体关系，部分和整体的生命周期一样。如狼和腿（引用型成员变量）

> 如何判断：Is-A时用继承，Has-A时用合成。不要为了部分功能而继承，认错了亲爹。

---

# II.汇总

- Creational 创建型（5种）
    + [Factory method（工厂方法模式）](http://blog.wocaishiliuke.cn/designpatterns/2019/01/03/Factory/)
    + [Abstract factory（抽象工厂模式）](http://blog.wocaishiliuke.cn/designpatterns/2019/01/03/Factory/)
    + [Singleton（单例模式）](http://blog.wocaishiliuke.cn/designpatterns/2019/01/02/Singleton/)
    + Builder（建造者模式）
    + Prototype（原型模式）
- Structural 结构型（7种）
    + Adapter（适配器模式）
    + Decorator（装饰器模式）
    + Proxy（代理模式）
    + Facade（外观模式）
    + Bridge（桥接模式）
    + Composite（组合模式）
    + Flyweight（享元模式）
- Behavioral 行为型（11种）
    + Strategy（策略模式）
    + Template method（模板方法模式）
    + Observer（观察者模式）
    + Iterator（迭代子模式）
    + Chain of responsibility（责任链模式）
    + Command（命令模式）
    + Memento（备忘录模式）
    + State（状态模式）
    + Visitor（访问者模式）
    + Mediator（中介者模式）
    + Interpreter（解释器模式）