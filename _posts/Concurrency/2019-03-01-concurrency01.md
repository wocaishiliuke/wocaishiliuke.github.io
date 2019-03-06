# 一、并发概述

## 1.并发优点
- 充分发挥多核CPU的计算能力（硬件设计者将摩尔定律的责任推给了软件开发者）
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

如脏读（即共享数据线程间的可见性问题，如DCL中指令重排的坑）、死锁等

> 扩展：避免死锁的方式

- 避免一个线程同时获得多个锁
- 避免一个线程在锁内部占有多个资源，尽量保证每个锁只占用一个资源
- 尝试使用定时锁，lock.tryLock(timeOut)，当超时等待时当前线程不会阻塞
- 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况

## 3.并发中的概念

- 同步：必须等待当前被调用方法执行结束，才能执行后续代码
- 异步：在当前被调用方法未完成时，也可以调用后续代码

- 并发：多任务交替切换执行
- 并行：多任务同时执行（多核CPU）

- 阻塞：一个线程占用了临界区资源（共享资源），其他线程必须等待资源的释放，才能访问
- 非阻塞：多个线程可以同时访问某资源

- 临界区：共享数据，可被多线程使用。但某线程占用时，其他线程必须等待


---

# 二、线程概述

Java程序本身就是一个多线程程序，包括了：

- main线程（程序入口）
- 分发处理发送信号到JVM的线程
- 调用对象finaliz()的线程
- 清除Reference的线程

## 1.线程的创建

3种方式

## 1.1 继承Thread类，重写run()

有单继承的弊端

```java
Thread thread = new Thread() {
    @Override
    public void run() {
        System.out.println("继承Thread");
        super.run();
    }
};
thread.start();
```

## 1.2 实现Runnable接口

最常用

```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("实现runable接口");
   }
});
thread.start();
```

## 1.3 实现Callable接口

```java
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

## 2.线程的生命周期

即线程的状态，和生命周期（转换）方法（参考图更清晰）

- NEW：初始状态。初始化后，还未调用Thread.start()
- RUNNABLE：运行状态。涵盖操作系统中的就绪（READY）和运行（RUNNING）两种状态
- BLOCKED：阻塞状态。阻塞于锁
- WAITING：等待状态。等待其他线程做出一些动作（通知或中断）
- TIME_WAITING：超时等待状态。不同于WAITING，可在指定时间自行返回
- TERMINATED：终止状态。表示线程已执行完毕

## 3.常用的线程操作

#### 3.1 interrupted

中断可以理解为线程的一个标志位，表示一个运行中的线程是否被其他线程进行了中断操作.

中断好比其他线程对该线程打了个招呼。其他线程可以调用该线程的interrupt()对其进行中断操作，同时该线程可以调用isInterrupted()来感知其他线程对自身的中断操作，从而做出响应。另外，同样可以调用Thread.interrupted()对当前线程进行中断，该方法会清除中断标志位。

在该线程执行了Object.wait()、Object.wait(long)、sleep(long)、join()、join(long)时，对其中断，会抛InterruptedException，同时会清空中断标志位。

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
    while (sleepThread.isInterrupted()) ;
    System.out.println("sleepThread isInterrupted: " + sleepThread.isInterrupted());
    System.out.println("busyThread isInterrupted: " + busyThread.isInterrupted());
}
```

输出：

```
sleepThread isInterrupted: false
busyThread isInterrupted: true
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at com.baicai.thread.InterruptDemo$1.run(InterruptDemo.java:10)
```

> 开启sleepThread和BusyThread后，sleepThread睡眠1s，busyThread执行死循环。然后分别对两个线程进行中断操作，sleepThread抛出InterruptedException后清除标志位，而busyThread不会清除标志位。while (sleepThread.isInterrupted()) ;是在main进程中监测sleepThread，一旦sleepThread的中断标志位清零，Main线程才会继续往下执行。因此，中断操作可以看做线程间一种简便的交互方式。一般在结束线程时通过中断标志位或者标志位去清理资源，相对于武断而直接的结束线程，这种方式要优雅安全。

#### 3.2 join

是线程间协作的一种方式。如果线程A执行了threadB.join()，意味着，当前线程A会等待线程B终止后才继续执行。join一共提供了如下方法:

```java
public final synchronized void join(long millis)
public final synchronized void join(long millis, int nanos)
public final void join() throws InterruptedException
```

其中join()的逻辑是：

```java
while (isAlive()) {
    wait(0);
}
```

如上，当前threadA会一直阻塞，直到threadB结束后即isAlive()返回false，才会结束while循环。当threadB退出时会调用notifyAll()通知所有等待的线程。

```java
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

上例输出（每个线程都会等待前一个线程结束才会继续运行）：

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

#### 3.3 sleep

Thread类的静态方法，让当前线程按照指定时间休眠。如果当前线程获得了锁，sleep()并不会失去锁。

```java
public static native void sleep(long millis)
```

> sleep()和wait()的区别：

- sleep()是Thread的静态方法。wait是Object的非静态方法
- wait()必须在同步方法或同步块中调用，也就是必须已经获得对象锁。sleep()可以在任何地方使用
- wait()会释放占有的对象锁，使得该线程进入等待池中，等待下一次获取资源。sleep()只会让出CPU，并不会释放掉对象锁
- sleep()在休眠时间达到后，如果再次获得CPU时间片会继续执行。wait()必须等待Object.notify()或Object.notifyAll()通知后，才会离开等待池，并且再次获得CPU时间片才会继续执行。

#### 3.4 yield

Thread类的静态方法，使当前线程让出CPU

```java
public static native void yield();
```

- 1.让出CPU不代表当前线程不再运行，如果在下次竞争中又获得了时间片，会继续运行
- 2.让出的时间片只会分配给与当前线程相同优先级的线程

sleep()和yield()，都会让当前线程会交出CPU资源。不同的是：sleep()交出来的时间片其他线程都可以去竞争，yield()只允许与当前线程具有相同优先级的线程竞争。

> 关于线程优先级

操作系统会分出一个个时间片，线程会分配到若干时间片，当前时间片用完后就会发生线程调度，并等待这下次分配。线程分配到的时间多少也就决定了线程使用处理器资源的多少，而线程优先级就是决定线程需要或多或少分配一些处理器资源的线程属性。
在Java中，通过一个整型成员变量Priority来控制优先级，范围1~10，默认值5。创建线程时可以通过setPriority(int)进行设置，**优先级高的线程相较于优先级低的线程，会优先获得CPU时间片。需要注意的是，在不同JVM和操作系统上，线程规划存在差异，有些操作系统甚至会忽略线程优先级的设定。

## 4.守护线程

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


---

# 三、JMM

Java Memory Model，Java内存模型。

线程安全：通俗的说，在多线程下代码执行的结果与预期正确的结果一致。否则就是线程不安全的。

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

- 可见性
- 有序性
- 原子性

可见性，即上述可能出的数据"脏读"现象，也就是数据可见性的问题。有序性，即需要注意重排序在多线程中也容易出现问题，比如经典的单例DCL，此时就需要禁止重排序。另外，原子性是指，在多线程下原子操作如i++不加以注意的也容易出现线程安全的问题（实际i++在Class文件中对应多条指令）。最后就是学习和使用好J.U.C包下的并发工具类和并发容器。


---

# 四、synchronized

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

## ４．锁获取和锁释放的内存语义

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