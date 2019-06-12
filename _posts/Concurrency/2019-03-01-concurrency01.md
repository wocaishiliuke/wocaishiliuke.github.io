---
title: 并发基础
date: 2019-03-01 00:00:00
categories:
    - Concurrency
tags:
    - Concurrency
---

参考[CL0610/Java-concurrency](https://github.com/CL0610/Java-concurrency)，整理一些并发的基础知识。

<!-- more -->

##### 目录
+ I.并发概述
+ II.线程概述
+ III.JMM
+ IV.synchronized
+ V.volatile
+ VI.final

---

# I.并发概述

- 并发：多任务交替切换执行
- 并行：多任务同时执行（多核CPU）

## 1.并发优点
- 充分发挥CPU的高效运算能力
- 方便业务拆分，提升应用性能

## 2.并发缺点
- 频繁的上下文切换（切换时，需要保存当前状态。切换的时间比加锁解锁还高一个数量级，还会导致CPU缓存失效）
- 有线程安全问题

#### 2.1 上下文切换

减少上下文切换的措施

- 无锁并发编程：可参照concurrentHashMap锁分段思想，不同线程处理不同段的数据，这样在多线程竞争的条件下，可以减少上下文切换的时间
- CAS：利用Atomic下使用CAS算法来更新数据，使用了乐观锁，可减少部分因不必要的锁竞争带来的上下文切换
- 最少线程：避免创建不需要的线程，比如任务很少时创建很多的线程，会造成大量线程处于等待状态
- 协程：在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换

#### 2.2 线程安全

线程安全，通俗的说，在多线程下代码执行的结果与预期正确的结果一致。否则就是线程不安全的，如出现脏读（即线程间共享数据的可见性问题，如DCL中指令重排的坑）、死锁等

> 避免死锁的方式

- 避免一个线程同时获得多个锁
- 避免一个线程在锁内部占有多个资源，尽量保证每个锁只占用一个资源
- 尝试使用定时锁，lock.tryLock(timeOut)，当超时等待时当前线程不会阻塞
- 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况
- 加锁解锁按优先级

## 3.并发中的概念

- 同步：必须等待当前被调用方法执行结束，才能执行后续代码
- 异步：在当前被调用方法未完成时，也可以调用后续代码

- 阻塞：一个线程占用了临界区资源（共享资源），其他线程必须等待资源的释放，才能访问
- 非阻塞：多个线程可以同时访问某资源

- 临界区：共享数据，可被多线程使用。但某线程占用时，其他线程必须等待

## 4.上下文切换

一个CPU内核在任意时刻只能被一个线程使用。一般线程数会大于CPU核数，为了让这些线程都能够及时地执行，以及充分使用CPU的计算，CPU采取为每个线程分配时间片并轮转的策略。当一个线程的时间片用完时，就会保存当前的执行状态，并重新处于就绪状态，让出CPU给其他线程使用，该过程就是上下文切换。

每次切换都需要纳秒级的时间，频繁的切换对系统来说会消耗大量的CPU时间。

## 5.死锁

线程A持有锁1，线程B持有锁2，此时如果两个线程都想获得对方的锁资源，就会互相一直等待，出现死锁。


---

# II.线程概述

## 1.进程和线程

**进程（process）**是程序的一次执行过程，是系统运行程序的基本单位。系统运行一个程序就代表着一个进程从创建、运行到消亡的过程。启动main函数时其实就是启动了一个JVM的进程，而main()所在的线程就是该java进程中的一个线程，称为主线程。

**线程（thread）**与进程相似，但线程是比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。由于多线程共享java进程的堆和方法区资源，尽管每个线程也有独立的程序计数器、虚拟机栈和本地方法栈，线程间的切换负担还是要比进程切换小得多。线程也被称为轻量级进程。

> Java程序本身就是一个多线程程序：

```java
// 查看该java进程的线程
public static void main(String[] args) {
    // 获取Java线程管理MXBean
    ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
    // 不需要获取同步的 monitor 和 synchronizer 信息，仅获取线程和线程堆栈信息
    ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
    // 遍历线程信息，仅打印线程 ID 和线程名称信息
    for (ThreadInfo threadInfo : threadInfos) {
        System.out.println("[" + threadInfo.getThreadId() + "] " + threadInfo.getThreadName());
    }
}
```
```
[5] Monitor Ctrl-Break
[4] Signal Dispatcher   // 分发处理给JVM信号的线程
[3] Finalizer           // 调用对象finalize()的线程
[2] Reference Handler   // 清除reference的线程
[1] main                // 主线程，程序入口
```


## 2.线程的创建

- 方式1：继承Thread类，重写run()，有单继承的弊端
- 方式2：实现Runnable接口，最常用
- 方式3：实现Callable接口

```java
// 第一种
Thread thread = new Thread() {
    @Override
    public void run() {
        System.out.println("继承Thread");
        super.run();
    }
};
thread.start();

// 第二种
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("实现runable接口");
   }
});
thread.start();

// 第三种
ExecutorService service = Executors.newSingleThreadExecutor();
Future<String> future = service.submit(new Callable() {
    @Override
    public String call() throws Exception {
        return "通过实现Callable接口";
    }
});
try {
    String result = future.get();
    System.out.println(result);
} catch (Exception e) {
    e.printStackTrace();
}
```

> 为什么调用start()会执行run()，为什么不直接调用run()？

因为新建状态的thread需要先调用start()，进入就绪状态（Runnable），当分配到时间片后就可以开始运行了。start()会执行一些准备工作，然后再自动执行run()。

如果直接执行run()方法，会把run()当成一个main线程下的普通方法去执行，并不会在thread线程中执行。

## 3.线程的生命周期

即线程的6个状态，和生命周期（转换）方法（参考图更清晰）

|状态|状态名|说明|
|:--|:----|:---|
|NEW|初始状态|初始化后，还未调用Thread.start()|
|RUNNABLE|运行状态|涵盖操作系统中的就绪（READY）和运行（RUNNING）两种状态|
|BLOCKED|阻塞状态|阻塞于锁|
|WAITING|等待状态|等待其他线程做出一些动作（通知或中断）|
|TIME_WAITING|超时等待状态|不同于WAITING，可在指定时间自行返回|
|TERMINATED|终止状态|表示线程已执行完毕|

![avatar](https://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/Concurrency/thread_status.png)

- 线程被创建后处于NEW（新建状态）
- 调用start()后处于READY（可运行状态）
- 当获得了CPU时间片（timeslice）后处于RUNNING（运行状态）
- 如果执行wait()等，该线程会进入WAITING（等待状态），需要其他线程的通知才能返回到运行状态
- 如果执行wait(long millis)等，该线程会进入TIME_WAITING（超时等待状态），相当于在WAITING状态的基础上增加了超时限制，到达超时时间后线程将返回到RUNNABLE状态
- 在调用synchronzied方法时，如果没有获取到锁，该线程会进入BLOCKED（阻塞状态）
- 当run()或main()执行完后，该线程会进入TERMINATED（终止状态）
- 已终止的线程不能复生，如果再调用start()，抛IllegalThreadStateException

## 4.常见线程操作

Thread类中的常用API。

#### 4.1 interrupt

其他线程可以调用该线程的interrupt()对其进行中断操作，同时该线程可以调用isInterrupted()或interrupted()来感知其他线程对自身的中断操作，从而做出响应。

中断好比其他线程对该线程打了个招呼，体现在线程的中断标志位上。因此，中断操作可以看做是线程间的一种简单交互。一般在结束线程时通过中断标志位去清理资源，相比于直接结束线程更安全优雅。

在该线程执行了Object.wait()、Object.wait(long)、sleep(long)、join()、join(long)后，对其中断，会抛InterruptedException，同时会清空中断标志位。

```java
public static void main(String[] args) {
    //sleepThread睡眠1000ms
    final Thread sleepThread = new Thread() {
        @Override
        public void run() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            super.run();
        }
    };
    //busyThread一直执行死循环
    Thread busyThread = new Thread() {
        @Override
        public void run() {
            while (true) ;
        }
    };
    sleepThread.start();
    busyThread.start();
    sleepThread.interrupt();
    busyThread.interrupt();
    while (sleepThread.isInterrupted());
    System.out.println("sleepThread isInterrupted: " + sleepThread.isInterrupted());
    System.out.println("busyThread isInterrupted: " + busyThread.isInterrupted());
}
```
```
sleepThread isInterrupted: false
busyThread isInterrupted: true
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at com.baicai.thread.InterruptDemo$1.run(InterruptDemo.java:10)
```

> 开启sleepThread和BusyThread后，sleepThread睡眠1s，busyThread执行死循环。然后分别对两个线程进行中断操作，sleepThread抛出InterruptedException后清除标志位，而busyThread不会清除标志位。while (sleepThread.isInterrupted());是在main线程中监测sleepThread线程，一旦sleepThread的中断标志位清零，Main线程才会继续往下执行。

#### 4.2 join

```java
public final synchronized void join(long millis)
public final synchronized void join(long millis, int nanos)
public final void join() throws InterruptedException
```

线程间协作的一种方式。如果线程A执行了B.join()，线程A会一直处于WAITING状态，直到线程B终止后才继续执行。

```java
// join()的底层实现
while (isAlive()) {
    wait(0);
}
```

线程B结束，即isAlive()返回false，才会结束while循环。线程B退出时会调用notifyAll()通知所有等待的线程。

```java
// 嵌套join，每个线程都会等待前一个线程结束再继续运行
public class JoinDemo {
    public static void main(String[] args) {
        Thread previousThread = Thread.currentThread();
        for (int i = 1; i <= 10; i++) {
            Thread curThread = new JoinThread(previousThread);
            curThread.start();
            previousThread = curThread;
        }
    }

    static class JoinThread extends Thread {
        private Thread thread;
        public JoinThread(Thread thread) {
            this.thread = thread;
        }

        @Override
        public void run() {
            try {
                thread.join();
                System.out.println(thread.getName() + " terminated.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
```
main terminated.
Thread-0 terminated.
Thread-1 terminated.
Thread-2 terminated.
Thread-3 terminated.
Thread-4 terminated.
Thread-5 terminated.
Thread-6 terminated.
Thread-7 terminated.
Thread-8 terminated.
```

#### 4.3 sleep

Thread的静态本地方法，按指定时间休眠当前线程。如果当前线程获得了锁，**sleep()并不会失去已获得的锁**。

```java
public static native void sleep(long millis)
```

> sleep()和wait()都可以暂停线程的执行，区别在于：

- sleep()是Thread的方法；wait是Object的方法（Thread也继承了）
- wait()必须在同步方法或同步块中调用，也就是必须先获得该对象的锁；sleep()可以在任何地方使用
- **wait()会释放占有的锁，该线程会进入等待池，等待通知。sleep()只会让出CPU，并不会释放锁**
- sleep()在休眠时间达到后会自动苏醒，如果再次获得时间片会继续执行。wait()必须等待Object.notify()或Object.notifyAll()通知后，才会离开等待池，并且再次获得CPU时间片后才会继续执行

wait()通常被用于线程间的交互和通信，sleep()通常仅用于暂停线程。

#### 4.4 yield

Thread的静态方法，使当前线程让出CPU

```java
public static native void yield();
```

- 让出CPU不代表当前线程不再运行，如果在下次竞争中又获得了时间片，会继续运行
- 让出的时间片只会分配给与当前线程相同优先级的线程

sleep()和yield()，都会让当前线程会交出CPU资源。不同的是，sleep()交出来的时间片其他线程都可以去竞争，yield()只允许与当前线程具有相同优先级的线程竞争。

## 5.守护线程

系统的守护者，在后台默默守护一些系统服务，如垃圾回收线程、JIT线程等。与用户线程对应。当用户线程完全结束后，守护线程也会退出。

```java
public class DaemonDemo {
    public static void main(String[] args) {
        Thread daemonThread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    try {
                        System.out.println("i am alive");
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        System.out.println("finally block");
                    }
                }
            }
        });
        daemonThread.setDaemon(true);
        daemonThread.start();
        //确保main线程结束前能给daemonThread能够分到时间片
        try {
            Thread.sleep(800);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## 6.线程优先级

操作系统会分出一个个时间片，线程会分配到若干时间片，当前时间片用完后就会发生线程调度，并等待这下次分配。线程分配到的时间多少也就决定了线程占用处理器资源的多少，而线程优先级就是决定线程占用处理器资源比例的属性。

在Java中，通过一个整型成员变量Priority来控制优先级，范围1~10，默认值5。创建线程时可以通过setPriority(int)进行设置，**优先级高的线程相较于优先级低的线程，会优先获得CPU时间片**。需要注意的是，在不同JVM和操作系统上，线程规划存在差异，有些操作系统甚至会忽略线程优先级的设定。


---

# III.JMM

Java Memory Model，Java内存模型。

> Java虚拟机规范试图定义一种Java内存模型，来屏蔽各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。在此之前，主流程序语言（如C/C++等）直接使用物理硬件和OS的内存模型，由于不同平台上内存模型的差异，导致程序在一套平台上并发正常，而在另一平台上并发出错，此时就必须针对不同的平台来编写程序

**存在线程安全问题的根本原因：多线程访问共享资源时，由于主存和线程的工作内存数据不一致，或指令重排序**。如果应用中不存在多线程访问共享资源的情景，就不需要考虑多线程安全问题。

解决线程安全的问题，最重要的是找出这两种问题发生的原因，核心在于理解Java内存模型（JMM）

## 1.JMM

#### 1.1 线程间通信

两种模式：

- 共享变量（Java采用）
- 消息通知

**Java内存模型是"共享内存"的并发模型，线程间主要通过读-写共享变量来完成隐式通信**。

共享变量包括：堆内存中的实例、静态资源、数组。不包括（栈中的）局部变量、形参、异常处理器参数等。非共享变量自然不会有线程安全问题。

#### 1.2 JMM下的线程通信

也是JMM的抽象结构（主存和线程工作内存）

参考图

- 1.线程A拷贝共享变量
- 2.线程A对变量副本操作
- 3.线程A将变量写回主存
- 4.线程B读取最新值

该过程可能出现"跨线程的数据可见性"问题，即脏读：A未写回，B就读取了主存中的共享变量。可以通过2种方式避免：

- 1.同步机制（锁，控制线程间的操作顺序）
- 2.volatile（强制刷新到主存）

#### 1.3 重排序

重排序导致的线程安全问题，最常见的例子就是单例模式DCL中的问题（也是使用volatile解决）。

优秀的内存模型会放松对处理器和编译器规则的束缚，在不改变程序执行结果的前提下，尽可能提高并行度。JMM对编译器和处理器也减少了束缚。为了提高性能，编译器和处理器常常会对指令进行重排序。重排序一般可以分为2种：

- 编译器重排序
- 处理器重排序
    - 指令级并行的重排序
    - 内存系统的重排序

JMM对编译器和处理器重排序的约束：

- 对编译器重排序：JMM会禁止一些特定类型的编译器重排序
- 对处理器重排序：编译器在生成指令序列时，会通过插入内存屏障（memory barrier）指令来禁止某些特殊处理器的重排序（比如volatile关键字就是这种方式）

除了上述约束外，编译器和处理器还会遵循数据依赖性：**编译器和处理器不会改变存在数据依赖性关系的两个操作的执行顺序**

> 数据依赖性

如果两个操作访问同一变量，且其中有写操作，此时这两个操作就存在数据依赖性。包括三种情况：读后写、写后写、写后读。在这三种情况下重排序，就会影响最终执行结果。

```java
double pi = 3.14         //A
double r = 1.0           //B
double area = pi * r * r //C
```

> A和B间没有数据依赖性，可以重排。即A-B-C和B-A-C两种顺序的执行结果相同


## 2.happens-before原则

> 如果开发人员考虑各种重排序的可能，哪还有时间处理业务逻辑。于是，JMM帮助做了一些事情

在...之前发生原则

JSR-133中使用happens-before概念来指定两个操作之间的执行顺序。这两个操作可以在单线程内，也可以在不同线程之间。JMM正是通过该原则，保证了跨线程间数据可见性。（如果A线程的写操作happens-before线程B的读操作，尽管写读操作是在不同的线程中执行的，JMM也能保证写操作后的数据将对读操作可见）

JMM对开发人员的承诺：如果一个操作happens-before另一个操作，那么第一个操作的执行结果对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。但实际执行顺序并非一定如此。两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。JMM遵循一个基本原则：只要不改变程序的执行结果（指的是单线程程序和正确同步的多线程程序），编译器和处理器怎么优化重排都行。JMM这么做的原因：开发人员对于这两个操作是否真的被重排序并不关心，只关心程序的语义不能被改变（即执行结果不能被改变）。因此，happens-before关系本质上和as-if-serial语义是一回事（程序员被你们骗来骗去...）：

> as-if-serial VS happens-before

- as-if-serial保证单线程内程序的执行结果不被改变，happens-before保证正确同步的多线程程序的执行结果不被改变
- as-if-serial给编写单线程程序的开发人员创造了一个假象：单线程程序是按程序的顺序来执行的。happens-before给编写正确同步的多线程程序的开发人员创造了一个假象：正确同步的多线程程序是按happens-before指定的顺序来执行的
- as-if-serial和happens-before的目的，都是为了在不改变程序执行结果的前提下，尽可能地提高程序执行的并行度

> 上述计算圆面积示例中，A happens-before B，对于开发人员，A操作的执行顺序在B操作之前。A、B操作不存在数据依赖性，在不改变最终结果的前提下，允许A，B两个操作重排序，即happens-before关系并不代表了最终的执行顺序。

#### 2.1 具体规则

- 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作
- 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁
- volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读
- 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C
- start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作
- join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回
- 程序中断规则：对线程interrupted()的调用先行于被中断线程的代码检测到中断时间的发生
- 对象finalize规则：一个对象的初始化完成（构造函数执行结束）先行于它的finalize()的开始

## 2.JMM总结

JMM是语言级的内存模型，可以理解JMM处于中间层，包含了两个方面
- 1.内存模型
- 2.重排序以及happens-before规则

JMM设计者需要考虑两个因素：
- 1.开发人员希望内存模型易于理解和使用，即强内存模型
- 2.编译器和处理器希望内存模型对它们的束缚越少越好，即弱内存模型。

所以，JMM
- 1.对于上层有基于JMM的关键字和J.U.C包，方便开发人员的并发编程
- 2.对于下层，在基本原则下（不改变运行结果的前提下，可以重排序），又提供了禁止特定类型的重排序方式，对编译器和处理器指令序列加以控制（如volatile会插入内存屏障指令）

> 关于JMM对编译器和处理器的基本原则。比如，如果编译器经过分析后，认定某个锁只会被单线程访问，那么这个锁可以被消除。再如，如果编译器认定某个volatile变量只会被单线程访问，那么编译器可以把这个volatile变量当作一个普通变量来对待。这些优化既不会改变程序的执行结果，又能提高程序的执行效率

另外，重排序可以分为两类：
- 会改变程序执行结果的重排序（JMM禁止这种重排序）
- 不会改变程序执行结果的重排序（JMM允许这种重排序）

## 3.并发编程需要注意的问题

并发的切入点分为2个核心，3大性质：

- 2个核心：
    - JMM内存模型（主内存和工作内存）
    - happens-before原则
- 3大性质：
    - 原子性
    - 可见性
    - 有序性

对于3大性质：可见性，即上述可能出的数据"脏读"现象，也就是数据可见性的问题。有序性，即需要注意重排序在多线程中也容易出现问题，比如经典的单例DCL，此时就需要禁止重排序。另外，原子性是指，在多线程下原子操作如i++不加以注意的也容易出现线程安全的问题（实际i++在Class文件中对应多条指令）。最后就是学习和使用好J.U.C包下的并发工具类和并发容器。


---

# IV.synchronized

```java
public class SynchronizedDemo implements Runnable {
    private static int count = 0;

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(new SynchronizedDemo());
            thread.start();
        }
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("result: " + count);
    }

    @Override
    public void run() {
        for (int i = 0; i < 1000000; i++)
            count++;
    }
}
```

> 每个线程都操作共享变量count，预期返回10*1000000，但却不一定

如果多线程间没有共享的数据，即多线程间没有协作完成一项任务，那么多线程就不能发挥优势，也没意义。如果有共享数据，共享数据的线程安全问题怎样处理呢？最先想到的是：每个线程依次读写共享变量，而synchronized就具有该功能。虽然同步机制效率低，但synchronized是其他并发容器实现的基础，需要学习。

## 1.概述

synchronized可修饰**方法和代码块**

|修饰目标|具体分类|被锁的对象|伪代码|
|:------|:-----|:----|:-----|
|方法|非静态方法|该类实例|public synchronized void method() {...}|
|方法|静态方法|该类对象|public static synchronized void method() {...}|
|代码块|该类实例|该类实例|synchronized (this) {...}|
|代码块|该类Class对象|该类对象|synchronized (Demo) {...}|
|代码块|任意实例|该实例|String lock = ""; synchronized (lock) {...}|

> 如果锁的是类对象，尽管new多个实例，他们都属于该类，所以仍会被锁住，即线程之间保证同步关系。

## 2.synchronized的底层实现

**对象锁（monitor）机制**

```java
public class SynchronizedDemo {
    public static void main(String[] args) {
        synchronized (SynchronizedDemo.class) {
        }
        method();
    }

    private synchronized static void method() {
    }
}
```

> 上述同步代码块和同步静态方法，锁住的都是类对象。编译后，查看SynchronizedDemo.class

```shell
$ javac SynchronizedDemo.java 
$ javap -v SynchronizedDemo.class 
```

> 部分字节码如下

```
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #2                  // class SynchronizedDemo
         2: dup
         3: astore_1
         4: monitorenter
         5: aload_1
         6: monitorexit
         7: goto          15
        10: astore_2
        11: aload_1
        12: monitorexit
        13: aload_2
        14: athrow
        15: invokestatic  #3                  // Method method:()V
        18: return
        ...
```

上面monitor相关的3个指令，是使用Synchronized之后才有的。使用Synchronized进行同步，关键在于必须获取到对象的监视器monitor，即只有线程获取了monitor后才能继续往下执行，否则该线程只能等待。并且获取的过程是互斥的，同一时刻只有一个线程能够获取到monitor。执行同步代码块后首先需要执行monitorenter指令，退出时执行monitorexit指令。

上面demo中在执行完同步代码块后，紧接着执行同步静态方法method()。而method()的锁对象依然是该类对象，那么正在执行的线程还需要获取该锁吗？不需要！从字节码中可以看到，执行method()时只有monitorexit指令，并没有monitorenter获取锁的指令。这就是**锁的重入性**，即在同一锁程中，线程不需要再次获取同一把锁。Synchronized先天具有重入性。每个对象拥有一个计数器，当线程获取该对象锁后，计数器就会加一，释放锁后就会将计数器减一。

每个对象都有自己的监视器，当执行该对象的同步块或同步方法时，执行线程必须先获取该对象的监视器，才能执行同步块和同步方法。如果没有获取到，线程被阻塞在同步块和同步方法的入口处（同步队列），进入BLOCKED状态。当监视器占有者释放后，同步队列中的线程才有机会获取该对象的监视器。

## 3.synchronized的happens-before关系

synchronized中的happens-before规则，即监视器锁规则：对一个监视器的解锁happens-before对该监视器的加锁。

```java
public class MonitorDemo {
    private int a = 0;

    public synchronized void writer() {     // 1
        a++;                                // 2
    }                                       // 3
}
```

假设有两个线程访问上述同步方法

- 1.线程A获取锁
- 2.执行操作临界区共享变量a
- 3.线程A释放锁
- 4.线程B获取锁
- 5.执行操作临界区共享变量a
- 6.线程B释放锁

**1.线程A释放锁happens-before线程B加锁。２．线程A的执行结果对线程B可见（即线程B所读取到主存中的a＝1）**

## ４.锁获取和锁释放的内存语义

即【线程A加锁-操作临界区变量-释放锁】过程中的内存操作（以上述代码为例）：线程A会先从主内存中读取共享变量a=0，然后将该变量拷贝到自己的本地内存，进行加１操作后，再将该值刷回到主内存

线程B获取锁时，会从主内存中读取共享变量最新值a=1，拷贝到线程B的工作内存，释放锁时同样会回写到主内存中。

**线程A的执行结果（a=1）对线程B是可见的的实现原理：释放锁的时候会将值刷新到主内存中，其他线程获取锁时会强制从主内存中获取最新的值。**

从两个线程的角度来看，就像是线程A通过主内存中的共享变量和线程B进行通信，A告诉B我们俩的共享数据ａ现在为1啦，这种线程间的通信机制正是JMM共享内存的并发模型结构。

## 5.synchronized优化

Synchronized同步最大的特征：同一时刻只有一个线程能够获得对象的监视器monitor，即互斥性（排它性）。当前线程获取到锁时，会阻塞其他线程，所以是**悲观锁策略**。

> 使用锁时，会假设每一次执行临界区代码都会产生冲突，就要阻塞其他线程，所以是悲观的

synchronized的这种同步形式是不能改变的！只能让每个线程执行通过的速度变快，也就是中间的某些环节变快，比如让获取锁的过程变快，即锁优化。实际上java也是这么做的。

在学习锁优化也就是锁的几种状态前，有必要先关注：1.CAS操作，2.Java对象头

#### 5.1 CAS

###### CAS概念

Compare And Swap，比较并替换。

相比之前线程获取锁的**悲观锁策略**，CAS操作（又称为无锁操作）是一种**乐观锁策略**。它假设所有线程访问共享资源的时候不会出现冲突和阻塞。如果出现冲突，会使用**CAS操作（比较交换）**来鉴别线程是否出现冲突，出现冲突就重试当前操作直到没有冲突为止。

###### CAS操作过程

多个线程使用CAS操作一个变量时，只有一个线程会成功更新，其余都失败。失败的线程会重新尝试，当然也可以选择挂起线程。

CAS比较交换的过程可以理解为CAS(V,O,N)，其中

- V：内存地址存放的实际值
- O：预期的值（旧值）
- N：更新的新值

对于同时操作的每个线程会比较V和O：

- 当V和O相同时，旧值=内存中实际的值，表明该值还没有被其他线程修改过，此时可以将该线程计算的新值N赋值给V，完成操作
- 当V和O不相同，表明该值已经被其他线程改过了，即O不是最新值了，所以不能将自己计算的新值N赋给V，返回V即可

> CAS的实现需要硬件指令集的支撑，在JDK1.5后，虚拟机才可以使用处理器提供的CMPXCHG指令实现。

###### Synchronized VS CAS

两者主要的区别

- **之前版本的Synchronized（未优化前）最主要的问题在于：线程竞争时出现的线程阻塞和唤醒锁带来的性能问题，是一种互斥同步（阻塞同步）**
- **CAS并不是武断的将线程挂起，当CAS操作失败后会进行一定的重试，而非耗时的挂起唤醒操作，因此也叫做非阻塞同步**

###### CAS的应用场景

J.U.C包中使用CAS的实现类很多，可以说CAS支撑起了整个concurrency包的实现。Lock实现中有CAS改变state变量，atomic包中的实现类也几乎都使用CAS实现。其他使用场景稍后详述。

###### CAS的问题

- 1.ABA问题

CAS在检查旧值是否变化时，存在这样一个情形：比如，旧值A变为了成B，然后再变成A，此时做CAS时发现旧值没有变化，但是实际上的确发生了变化。解决方案可以参考数据库中常用的乐观锁方式：添加版本号。原来的变化路径A->B->A就变成了1A->2B->3A。在java1.5后的atomic包中，提供了AtomicStampedReference来解决ABA问题，解决思路就是这样的。

- 2.自旋时间过长

使用CAS时非阻塞同步，也就是说不会将线程挂起，但会自旋（无非就是一个死循环）进行下一次尝试，如果这里自旋时间过长对性能也是很大的消耗。如果JVM能支持处理器提供的pause指令，那么在效率上会有一定的提升。

- 3.只能保证一个共享变量的原子操作

当对一个共享变量执行操作时CAS能保证其原子性，如果对多个共享变量进行操作，CAS就不能保证其原子性。一个解决方案是使用对象整合多个共享变量，对该对象做CAS操作就可以保证其原子性。atomic中提供了AtomicReference来保证引用对象之间的原子性。

#### 5.2 Java对象头

同步的时候获取对象的monitor，即获取对象的锁。对象的锁类似对象的一个标志，就是存放在Java对象中的对象头。Java对象头中的Mark Word中，默认存放的是对象的Hashcode、分代年龄和锁标记位。32位的JVM Mark Word的默认存储结构为（注:java对象头以及下面的锁状态变化摘自《java并发编程的艺术》一书）：

|锁状态|25bit|4bit|1bit是否是偏向锁|2bit锁标志位|
|:----|:----|:----|:-------------|:---------|
|无锁状态|对象的hashCode|对象分代年龄|0|01|

JavaSE 1.6中，锁一共有4种状态（级别从低到高）：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态。这些状态会随着竞争情况逐渐升级。锁可以升级但不能降级，如偏向锁升级成轻量级锁后不能再降回到偏向锁。这种只升不降的策略，是为了提高获得锁和释放锁的效率。对象的MarkWord变化为下图：

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/Concurrency/makeword)

###### 1.偏向锁

HotSpot的作者发现，大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。

> 偏向锁的获取

当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里，存储锁偏向的线程ID。以后该线程在进入和退出同步块时，不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头中的Mark Word里，是否存储着指向当前线程的偏向锁。如果测试成功，表示线程已经获得了锁。如果测试失败，则需要再测试一下Mark Word中偏向锁的标识是否设置成1（1就表示当前是偏向锁）：如果没有设置，则使用CAS竞争锁；如果设置了，则尝试使用CAS将对象头的偏向锁指向当前线程

> 偏向锁的撤销

偏向锁使用了一种**出现竞争才释放锁**的机制。所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁。

- 时机
    - 1.偏向锁的撤销，需要等待全局安全点（在这个时间点上，没有正在执行的字节码）
- 步骤
    - 1.首先，暂停拥有偏向锁的线程
    - 2.然后，检查持有偏向锁的线程是否活着
        - a.如果线程不处于活动状态，则将对象头设置成无锁状态
        - b.如果线程活着，拥有偏向锁的栈会被执行，遍历偏向对象的锁记录。栈中的锁记录和对象头中的Mark Word，要么重新偏向于其他线程，要么恢复到无锁或者标记对象不适合作为偏向锁
    - 3.最后，唤醒暂停的线程

下图线程1展示了偏向锁获取的过程，线程2展示了偏向锁撤销的过程

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/Concurrency/biased-lock-revocation)

> 偏向锁的关闭

偏向锁在Java 6和Java 7里是默认启用的，但是是在应用程序启动几秒钟之后才激活。如果需要可以使用JVM参数来关闭延迟：-XX:BiasedLockingStartupDelay=0。**如果确定应用程序里所有的锁通常情况下都处于竞争状态，可以通过下面的JVM参数关闭偏向锁，那么程序默认会进入轻量级锁状态**。

```
-XX:-UseBiasedLocking=false
```

###### 2.轻量级锁

> 加锁

线程在执行同步块之前，JVM会先在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录中，官方称为Displaced Mark Word。然后线程尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。

> 解锁

轻量级解锁时，会使用原子的CAS操作将Displaced Mark Word替换回到对象头，如果成功，则表示没有竞争发生。如果失败，表示当前锁存在竞争，锁就会膨胀成重量级锁。下图是两个线程同时争夺锁，导致锁膨胀的流程图。

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/Concurrency/lightweight-lock)

因为自旋也会消耗CPU，为了避免无用的自旋（比如获得锁的线程被阻塞住了），一旦锁升级成重量级锁，就不会再恢复到轻量级锁状态。当锁处于重量级状态下，其他线程试图获取锁时，都会被阻塞住，当持有锁的线程释放锁之后会唤醒这些线程，被唤醒的线程就会进行新一轮的夺锁之争。

###### 3.各种锁的比较

|锁|优点|缺点|适用场景|
|:-|:--|:---|:-----|
|偏向锁|加锁和解锁不需要额外的消耗，和执行非同步方法相比仅存在纳秒级的差距|如果线程间存在锁竞争，会带来额外的锁撤销消耗|只有一个线程访问同步块时|
|轻量级锁|竞争的线程不会阻塞，提高了程序的响应速度|如果始终得不到锁竞争的线程，使用自旋会消耗CPU|追求响应时间、同步块执行速度非常快|
|重量级锁|线程竞争不使用自旋，不会消耗CPU|线程阻塞，响应时间缓慢|追求吞吐量、同步块执行速度较长|

## 6.改造示例

```java
public class SynchronizedDemo implements Runnable {
    private static int count = 0;

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(new SynchronizedDemo());
            thread.start();
        }
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("result: " + count);
    }

    @Override
    public void run() {
        synchronized (SynchronizedDemo.class) {
            for (int i = 0; i < 1000000; i++)
                count++;
        }
    }
}
```

如果不使用同步，可能会出现A线程累加后，B线程做累加，但使用的是原来值（A写回主存前的值），即"脏值"，结果就会出错。使用了Syncnized就可以能保证内存可见性，保证每个线程操作的都是最新值。当然还有其他改造方式


---

# V.volatile

## 1.概述

volatile可以说是和synchronized各领风骚。synchronized是阻塞式同步，在线程竞争激烈时会升级为重量级锁。而volatile可以说是java虚拟机提供的最轻量级的同步机制。

Java内存模型中，各个线程会将共享变量从主内存拷贝到工作内存，然后对工作内存中的数据进行操作。对于普通变量，何时写回主存是没有规定的，而针对volatile修饰的变量，java虚拟机有着特殊的约定：**线程对volatile变量的修改会立刻被其他线程所感知**，不会出现脏读，从而保证数据的"可见性"。

被volatile修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免数据脏读。

> volatile只能保证可见性，不能保证原子性。如对volatile int a进行a++操作

## 2.原理

在生成汇编代码时，会在volatile修饰的共享变量进行写操作时，多出Lock前缀的指令（可以使用一些工具查看）。Lock前缀的指令在多核处理器下主要有两个方面的影响：

- a.将当前处理器缓存行的数据写回系统内存
- b.该写回内存的操作，会使得其他CPU里缓存了该内存地址的数据无效

为了提高处理速度，处理器不直接和内存通信。而是先将系统内存的数据读到内部缓存（L1，L2或其他），再进行操作，但操作完成后不知道何时会写到内存。如果对volatile变量进行写操作，JVM就会向处理器发送一条Lock前缀指令，将这个变量所在缓存行的数据写回系统内存。但此时其他处理器缓存的值还是旧的，再执行计算操作仍会有问题。所以，在多处理器下，为了保证各处理器的缓存一致，会实现**缓存一致性协议**。每个处理器通过嗅探在总线上传播的数据，来检查自己缓存的值是否过期。一旦处理器发现自己缓存行对应的内存地址被修改，会将缓存行设置成无效状态。当处理器对该数据进行操作时，会重新从系统内存中把该数据读到自己的缓存。即volatile：

- a.Lock前缀的指令会引起处理器缓存写回内存
- b.一个处理器的缓存回写到内存，会导致其他处理器的缓存失效
- c.当某个处理器发现本地缓存失效后，就会从内存中重读该变量，获取最新值

对于volatile变量，通过缓存一致性协议这样的机制，使得每个线程都能获得该变量的最新值，满足数据的"可见性"。

## 3.volatile的happens-before关系

在happens-before的具体规则中，有一条关于volatile变量的：**对一个volatile域的写，happens-before于任意后续对这个volatile域的读。**

```java
public class VolatileDemo {
    private int a = 0;
    private volatile boolean flag = false;
    public void writer(){
        a = 1;          //1
        flag = true;    //2
    }
    public void reader(){
        if(flag){       //3
            int i = a;  //4
        }
    }
}
```

假设加锁线程A先执行writer方法，然后线程B执行reader方法：

- 1.线程A修改共享变量a
- 2.线程A修改共享volatile变量flag
- 3.线程B读取共享volatile变量flag
- 4.线程B读取共享变量a

其中，根据happens-before规则对volatile变量的要求可知，线程A修改flag（步骤2）happens-before线程B读取flag（步骤3），线程A修改后的flag值对线程B可见。即当线程A将volatile变量 flag更改为true后线程B就能够迅速感知。

## 4.volatile的内存语义

即volatile对应的内存操作

假设线程A先执行writer()，线程B随后执行reader()。初始时线程的本地内存中flag和a都是初始状态，下图是线程A执行写后的状态：

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/Concurrency/volatile01)

当volatile变量写回主存后，线程B中的本地内存中的flag就会置为失效状态，线程B需要重新从主内存中读取该变量值。下图是线程B读取后的状态：

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/Concurrency/volatile02)

横向看，就像线程A和线程B之间进行了一次通信。线程A在写volatile变量时，实际上就像是给B发送了一个消息，告诉线程B你现在的值过期了。线程B读取使用自己工作内存中的该变量时，就像是接收了线程A刚刚发送的消息，然后重新去主内存读取。

#### volatile的内存语义实现

为了性能优化，JMM在不改变正确语义的前提下，允许编译器和处理器对指令序列重排序。如果想阻止重排序，可以添加内存屏障。

- 内存屏障

JMM内存屏障分为4类：

|屏障类型|指令示例|说明|
|:------|:-----|:---|
|LoadLoad Barriers|Load1;LoadLoad;Load2|确保Load1数据的装载先于Load2及所有后续装载指令的装载|
|StoreStore Barriers|Store1;StoreStore;Store2|确保Store1数据对其他处理器可见（刷新到内存）先于Store2及所有后续存储指令的存储|
|LoadStore Barriers|Load1;LoadStore;Store2|确保Load1数据装载先于Store2及所有后续存储指令刷新到内存|
|StoreLoad Barriers|Store1;StoreLoad;Load2|确保Store1数据对其他处理器可见（刷新到内存）先于Load2及所有后续装载指令的装载。StoreLoad Barriers会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令|

为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中的适当位置，插入内存屏障来禁止特定类型的处理器重排序。JMM针对编译器制定的volatile重排序规则表：

|能否重排序|第二个操作|第二个操作|第二个操作|
|:--|:-----|:---|:----|
|第一个操作|普通读/写|volatile读|volatile写|
|普通读/写| | |NO|
|volatile读|NO|NO|NO|
|volatile写| |NO|NO|

上表中，"NO"表示禁止重排序。对于编译器来说，采用一个最优布置来最小化插入屏障的总数，几乎是不可能的，为此，JMM采取了保守策略：

- 1.在每个volatile写操作的前面插入1个StoreStore屏障，后面插入1个StoreLoad屏障
- 2.在每个volatile读操作的后面插入1个LoadLoad屏障和1个LoadStore屏障

> volatile写是在前后分别插入内存屏障，而volatile读是在后面插入两个内存屏障

- StoreStore屏障：禁止上面的普通写和下面的volatile写重排序；
- StoreLoad屏障：防止上面的volatile写与下面可能有的volatile读/写重排序
- LoadLoad屏障：禁止下面所有的普通读操作和上面的volatile读重排序
- LoadStore屏障：禁止下面所有的普通写操作和上面的volatile读重排序

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/Concurrency/volatile-barrier-01)

![avatar](http://blog-wocaishiliuke.oss-cn-shanghai.aliyuncs.com/images/Concurrency/volatile-barrier-02)

## 5.示例

#### 示例1：volatile的用处

```java
public class VolatileDemo {
    private static volatile boolean isOver = false;

    public static void main(String[] args) {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (!isOver) ;
            }
        });
        thread.start();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        isOver = true;
    }
}
```

上述将isOver设置成volatile变量，这样在main线程中将isOver改为了true后，thread线程的工作内存该变量值就会失效，需要重新从主内存中读取该值。读取isOver最新值后，结束掉thread线程中的死循环，从而才能够完成和停掉thread线程。

另外，单例DCL中创建实例的线程安全问题，也可以使用volatile解决。即不让instance = new Singleton()对应的多条指令重排序（按照1.分配内存空间-2.初始化对象-3.引用赋内存地址值的顺序，而非1-3-2），保证了多线程下的单例。

#### 示例2：volatile的无用之处

```java
public class VolatileDemo {
    public static volatile int a = 0;
    public static final int THREAD_COUNT = 20;
    
    public static void main(String[] args) {
        Thread[] threads = new Thread[THREAD_COUNT];
        for (int i = 0; i < THREAD_COUNT; i++) {
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int i = 0; i < 1000; i++) {
                        increase();
                    }
                }
            });
            threads[i].start();
        }
        //等待所有累加线程结束
        while(Thread.activeCount() > 1)
            Thread.yield();
        System.out.println(a);  
    }

    public static void increase() {
        a++;
    }
}
```

每次执行的结果可能都不一样，即使使用了volatile，也没能避免并发安全问题。因为volatile只能保证可见性，无法保证原子性。**自增操作并不是一个原子操作**（字节码如下所示，对应0、3、4、5指令），包括了读和写操作，在并发的情况下，putstatic指令可能把较小的a值同步回主内存之中（覆盖掉其他线程的写回值），导致结果出错。

```
public static void increase();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=0, args_size=0
         0: getstatic     #11                 // Field a:I
         3: iconst_1
         4: iadd
         5: putstatic     #11                 // Field a:I
         8: return
         ...
```

当然可以对整个increase()同步，即加上synchronized修饰，那么其他线程将会阻塞，性能稍差。此时可以使用CAS解决：使用Java并发包中的原子操作类（Atomic开头）

```java
public static AtomicInteger a = new AtomicInteger(0);
public static void increase() {
    // a++;             //非原子操作：取值-加1-写值
    a.getAndIncrement();//原子操作
}
```

> getAndIncrement()是采用CAS操作实现的自增（其中使用了基于硬件cmpxchg指令的Atomic::cmpxchg()方法），保证了操作的原子性


---

# VI.final

## 1.简介

参考[Java面向对象（3）](http://blog.wocaishiliuke.cn/javase/2018/01/08/javase_oo03/)

## 2.多线程中的final

JMM对于编译器和处理器是一种弱内存数据模型。允许它们对指令的重排序。多线程中的final讨论的就是**final变量的重排序**问题

#### 2.1 基本数据类型

即final修饰的基本数据类型的重排序问题

```java
public class FinalDemo {
    private int a;  //普通域
    private final int b; //final域
    private static FinalDemo finalDemo;

    public FinalDemo() {
        a = 1; // 1.写普通域
        b = 2; // 2.写final域（final域的显示初始化）
    }

    public static void writer() {
        finalDemo = new FinalDemo();
    }

    public static void reader() {
        FinalDemo demo = finalDemo; // 3.读对象引用
        int a = demo.a;             // 4.读普通域
        int b = demo.b;             // 5.读final域
    }
}
```

假设线程A在执行writer()，线程B在执行reader()。（如果线程B先执行了reader()，会报空指针）

##### 写final域的重排序规则

**JMM禁止编译器把final域的写，重排序到构造函数之外**。

该规则的实现方式：编译器会在final域写之后，构造函数return之前，插入一个storestore屏障。该屏障可以禁止处理器把final域的写，重排序到构造函数之外。

构造方法中，a和b之间没有数据依赖性。就存在一种执行可能：普通变量a有可能被重排序到构造函数之外，线程B也就有可能读到a的默认初始化值0，出现错误。而final域b，根据重排序规则，其写操作不会重排序到构造函数之外，线程B就能够读到final变量显示初始化后的值。描述如下：

> 线程A执行writer()，new操作大致分3步：创建对象、构造初始化、赋地址值给引用

- 1.线程A开始执行构造方法
- 2.final变量b=2写操作
- 3.storestore屏障
- 4.构造执行完成return
- 5.赋地址值给引用finalDemo
- 6.普通变量a=1写操作

- i.线程B开始执行reader()
- ii.读引用finalDemo
- iii.读普通变量a
- iiii.读final变量b

>上述两个线程执行的过程中，如果线程A执行到步骤5之后，线程B执行iii去读取普通变量a，就只能读到0。

因此，**写final域的重排序规则可以确保：在对象引用finalDemo为任意线程可见之前，对象的final域能够被正确初始化。而普通域就不具有这个保障。**

##### 读final域的重排序规则

**在一个线程中，初次读对象引用和初次读该对象包含的final域，JMM会禁止这两个操作的重排序。**

注意，该规则仅针对处理器，处理器会在读final域操作的前面，插入一个LoadLoad屏障。

> 实际上，读对象的引用和读该对象的final域存在间接依赖性，一般处理器不会重排序这两个操作。但是有一些处理器会重排序。因此，该规则就是针对这些处理器而设定的。

read()方法主要包含了三个操作：初次读引用finalDemo、初次读引用对象的普通域a、初次读引用对象的final域b。假设线程A写过程没有重排序，针对某些处理器，存在一种执行可能：

- 1.线程A开始执行构造方法
- 2..普通变量a=1写操作
- 3.final变量b=2写操作
- 4.storestore屏障
- 5.构造执行完成return
- 6.赋地址值给引用finalDemo


- i.线程B开始执行reader()
- ii.读普通变量a
- iii.读引用finalDemo
- iiii.LoadLoad屏障
- iiiii.读final变量b

上述可能中，读取对象的普通变量，被重排序到了读对象引用的前面，显然是错误的操作。而final域的读规则避免了这种情况：**在读一个对象的final域之前，一定会先读包含这个final域的对象的引用**。

#### 2.2 引用数据类型

即final修饰的引用数据类型的重排序问题

```java
public class FinalReferenceDemo {
    final int[] array;
    private FinalReferenceDemo finalReferenceDemo;

    public FinalReferenceDemo() {
        array = new int[1];  //1
        array[0] = 1;        //2
    }

    public void writerOne() {
        finalReferenceDemo = new FinalReferenceDemo(); //3
    }

    public void writerTwo() {
        array[0] = 2;  //4
    }

    public void reader() {
        if (finalReferenceDemo != null) {  //5
            int temp = finalReferenceDemo.array[0];  //6
        }
    }
}
```

##### 写final域的重排序规则

JMM可以确保线程C至少能看到写线程A对final引用的对象的成员域的写入，即能看下arrays[0] = 1，而写线程B对数组元素的写入可能看到可能看不到。JMM不保证线程B的写入对线程C可见，线程B和线程C之间存在数据竞争，此时的结果是不可预知的。如果可见的，可使用锁或者volatile。



##### 对final修饰的对象的成员域写操作

针对引用数据类型，final域写针对编译器和处理器重排序增加了这样的约束：在构造函数内对一个final修饰的对象的成员域的写入，与随后在构造函数之外把这个被构造的对象的引用赋给一个引用变量，这两个操作是不能被重排序的。注意这里的是“增加”也就说前面对final基本数据类型的重排序规则在这里还是使用。这句话是比较拗口的，下面结合实例来看。
