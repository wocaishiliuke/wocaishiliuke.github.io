---
title: Exception
date: 2018-01-20 00:00:00
categories:
    - JavaSE
tags:
    - JavaSE
---

本文记录Java中的Exception体系。

<!-- more -->

##### 目录
- I.Throwable
- II.Exception
- III.异常处理

---
# I.Throwable

Java中把异常也视为对象来处理。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/JavaSE/exception/exception-system.png)

Throwable是异常的基类，可分为Error（错误）、Exception（异常）。
- Error：系统错误，无法处理的异常，如StackOverflowError、OutOfMemoryError等。JVM会终止程序，编程时无需关心
- Exception：编译期和运行期出现的程序异常，分为checked exception和unchecked exception

## unchecked、checked异常

把error也归于unchecked exception，所以Throwable分为：

- unchecked exception：RuntimeException、error
- checked exception：其他所有非RuntimeException的Exception

**异常分类（unchecked、checked）的原因是，编译器会检查程序中的checked exception，是否有被处理**。比如使用Class.forName()创建时字节码对象时，不作处理（throws或try），编译就通不过（ClassNotFoundException是checked exception）。

```java
public static Class<?> forName(String className) throws ClassNotFoundException{
   ...
}
```

> unchecked、checked区别
- 逻辑上
    + unchecked：多由程序BUG导致，逻辑上要让这些异常走到main()或Thread.run()，终止程序或线程，防止BUG被掩盖
    + checked：多是程序依赖了不可控的外部资源，未雨绸缪，这也体现了Java的安全性
- 处理上
    + unchecked：不必须throw、throws、try，调用者也不用处理，防止掩盖代码BUG
    + checked：必须处理，有能力处理就try，否则throw（先抓后抛）/throws


---
# II.Exception

## RuntimeException

如上所述，Exception的RuntimeException子体系是unchecked的，编译器不强制对其进行处理，即：

- 不必throw（比如，程序中没必要抛出ArrayIndexOutOfBoundsException）
- 即使throw了，也不必在方法上throws
- 即使throws了，调用者也可以不处理（try或throw）

> 为什么不必须处理RuntimeException

**避免程序BUG被掩盖。RuntimeException的发生，多是代码BUG导致的**，如ArrayIndexOutOfBoundsException，表示程序无法继续执行下去，希望终止程序或当前线程，然后由开发人员修改BUG。所以不必须处理。如果抛到了最上层，由多线程的Thread.run()或单线程的main()抛出，线程或程序终止。

所以RuntimeException不容易预先避免，后果严重，其实也就是BUG的后果严重...

> 自定义RuntimeException

通常自定义业务异常，就可以继承自RuntimeException，即unchecked的。当违反业务逻辑时，就可以抛出该异常，然后使用ExceptionHandler处理，返回对应的异常信息到前端。这也是主动处理unchecked exception的情形。（不终止当前请求线程，而是捕获后返回指定响应内容）

```java
public class ProductException extends RuntimeException{
    private static final long serialVersionUID = -8798692913708977154L;
    public ProductException() { super(); }
    public ProductException(String message) { super(message); }
}
```

```java
@ControllerAdvice
public class ProductExceptionHandler extends ResponseEntityExceptionHandler{
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    
    /** 统一处理商品异常 */
    @ExceptionHandler({ ProductException.class })
    @ResponseStatus(value=HttpStatus.BAD_REQUEST)
    @ResponseBody
    public MessageDTO handleProductException(ProductException ex) {
        logger.error("=========Error========:" + ex.getMessage());
        return new MessageDTO(MessageType.ERROR, ex.getMessage());
    } 
}
```

## 其他子类

Exception的其他子类都是checked，即在编译前就需要处理的，否则编译通不过。至于throw还是try，要看具体的情况：
- 一般能try的就当场处理掉
- 自己不能处理的，向上throw/throws

> 自定义checked Exception

自定义的checked Exception，需要处理，否则编译通不过。

```java
public class CustomizeException extends Exception {
    private static final long serialVersionUID = 8526730077158440372L;
    public CustomizeException() { super(); }
    public CustomizeException(String message) { super(message); }
}
```


---
# III.异常处理

## 1.默认处理（不处理）

对于unchecked exception，编译能通过，可以不处理，即交由JVM处理

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(new Inner().div(1,0));
    }

    static class Inner{
        public int div(int a, int b) { return a / b; }
    }
}
```

```java
Exception in thread "main" java.lang.ArithmeticException: / by zero
    at com.baicai.dao.Test$Inner.div(Test.java:14)
    at com.baicai.dao.Test.main(Test.java:9)
```

ArithmeticException属于RuntimeException，unchecked

## 2.try

有3种形式：try-catch-finally、try-catch、try-finally

```java
/* 形式1 */
try {
    //可能抛出异常
} catch (Exception e) {
    //捕获异常
}finally{
    //不管有无异常都执行
}
```
```java
/* 形式2 */
try {
    //可能抛出异常
} catch (Exception e) {
    //捕获异常
}
```

```java
/* 形式3 */
try{
    //可能抛出异常，但没catch，将异常传递给上层调用方
}finally{
    //不管有无异常都执行
}
```

异常处理可能出现3种情况：
- 没有异常发生，try内的代码执行结束后，执行finally（如果有）
- 有异常发生且被catch捕获，执行catch后，执行finally（如果有）
- 有异常发生但没被捕获（没有catch，或catch没抓到），执行finally，然后抛出异常（给上层）


注意事项：
- 1.try没抛异常，不走catch，但会走finally；抛了异常才会走catch，然后继续执行
- 2.try{}内局部变量的作用范围，就只是{}内
- 3.catch可用多态接收；catch可以多个；也可以合起来（JDK1.7新特性）
- 4.当有多个catch块时，"小异常放前，大异常放后"，否则编译过不去，且只会走一个catch
- 5.finally常用于释放资源：DB连接、I/O流等

```java
// 多个catch块时，"小异常放前，大异常放后"
public static void main(String[] args) {
    try {
        int x = 1 / 0;
    } catch (ArithmeticException | NullPointerException e) { // JDK1.7新特性
        //System.out.println(x);    //超出了x的作用范围
        e.printStackTrace();
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        System.out.println("finally block...");
    }
}

// 输出
finally block...
java.lang.ArithmeticException: / by zero
```

#### finally中的return

finally中没有return时：
- 如果没有抛异常，先执行finally，然后正常return
- 如果抛出异常，先执行finally，调用方

- 1.finally先于return执行
- 2.如果finally中有return，会覆盖其他return返回值，包括try和catch中的return，也包括没有抛异常的情况

## 3.throw、throws

- throw：方法内主动抛出异常
- throws：**声明**一个方法可能抛出的异常，可以声明多个异常，以逗号分隔

> throws只是声明，声明该方法可能抛出这些异常（内部没有处理，或没有处理完），调用者必须进行处理。该声明没有说明什么情况下对应会抛出什么样的异常，通常应该将这些信息写到注释中，方便调用者能更好的处理异常。

- 对于RuntimeException（unchecked exception），不要求必须使用throws声明，即使声明了，调用者也可以无视
- 对于checked exception，必须throws声明或内部处理。如果没有声明，不能抛出异常
- 对于checked exception，也可以只throws声明，但实际并不抛出

> 实际并不抛出异常的方法，仍然使用throws声明，主要是用于父类方法中。此时虽然父类方法不会抛出异常，但子类重写增强后就有可能就抛出，而子类又不能抛出父类方法中没有声明的checked exception，所以就需要将所有可能抛出的异常都声明到父类方法上。

当一个方法调用另一个声明抛出checked exception的方法时，就必须处理这些checked exception，可以catch，也可以继续throws

```java
public void tester() throws IOException {
    try {
        test();
    }  catch (SQLException e) {
        e.printStackTrace();
    }
}
```

> 对于test()抛出的SQLException，有能力就catch。对于test()抛出的IOException，处理不了就throws声明，交由上层处理。