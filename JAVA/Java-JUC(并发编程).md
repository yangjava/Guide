# Java并发编程

## 并发编程概念

### 程序(programm)

概念：是为完成特定任务、用某种语言编写的一组指令的集合。即指一段静态的代码。

### 进程(process)

概念：程序的一次执行过程，或是正在运行的一个程序。
说明：进程作为资源分配的单位，系统在运行时会为每个进程分配不同的内存区域

### 线程(thread)

概念：进程可进一步细化为线程，是一个程序内部的一条执行路径。
说明：线程作为CPU调度和执行的单位，每个线程拥独立的运行栈和程序计数器(pc)，线程切换的开销小。

进程可以细化为多个线程。
每个线程，拥有自己独立的：栈、程序计数器
多个线程，共享同一个进程中的结构：方法区、堆。

### 并发和并行

　　**并行**：指两个或多个时间在同一时刻发生（同时发生）；指多个线程同时执行来使用不同的资源

　　**并发**：指两个或多个事件在一个时间段内发生。指多个线程同时执行来抢占相同的资源

　　在操作系统中，安装了多个程序，并发指的是在一段时间内宏观上有多个程序同时运行，这在单 CPU 系统中，每一时刻只能有一道程序执行，即微观上这些程序是分时的交替运行，只不过是给人的感觉是同时运行，那是因为分时交替运行的时间是非常短的。

　　而在多个 CPU 系统中，则这些可以并发执行的程序便可以分配到多个处理器上（CPU），实现多任务并行执行，即利用每个处理器来处理一个可以并发执行的程序，这样多个程序便可以同时执行。

　　目前电脑市场上说的多核 CPU，便是多核处理器，核 越多，并行处理的程序越多，能大大的提高电脑运行的效率。

注意：单核处理器的计算机肯定不能并行的处理多个任务，只能是多个任务交替的在单个 CPU 上运行。

> 方便理解，举个栗子：
>
> - 并发：很多人同时在同一个窗口打饭
> - 并行：很多人同时在不同的窗口打饭

### 单核CPU与多核CPU的理解

- 单核CPU，其实是一种假的多线程，因为在一个时间单元内，也只能执行一个线程的任务。例如：虽然有多车道，但是收费站只有一个工作人员在收费，只有收了费才能通过，那么CPU就好比收费人员。如果某个人不想交钱，那么收费人员可以把他“挂起”（晾着他，等他想通了，准备好了钱，再去收费。）但是因为CPU时间单元特别短，因此感觉不出来。
- 如果是多核的话，才能更好的发挥多线程的效率。（现在的服务器都是多核的）
- 一个Java应用程序java.exe，其实至少三个线程：main()主线程，gc()垃圾回收线程，异常处理线程。当然如果发生异常，会影响主线程。

### 并行与并发的理解

并行：多个CPU同时执行多个任务。比如：多个人同时做不同的事。
并发：一个CPU(采用时间片)同时执行多个任务。比如：秒杀、多个人做同一件事

### **同步（Synchronous）和异步（Asynchronous）**

同步和异步通常来形容一次方法调用，**同步方法调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为**。**异步方法调用更像一个消息传递，一旦开始，方法调用就会立即返回，调用者就可以继续后续的操作**。而异步方法通常会在另外一个线程中“真实”地执行。整个过程，不会阻碍调用者的工作。

如图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicEJhWlK06CGcWVldNgyN4eYU11dSFHE6YTnvmXOp8cmGGcyPePMxNiba2RWLHaoX0jMaDiaqzPEFErWXHhsPowA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图中显示了同步方法调用和异步方法调用的区别。对于调用者来说，异步调用似乎是一瞬间就完成的。如果异步调用需要返回结果，那么当这个异步调用真实完成时，则会通知调用者。

打个比方，比如购物，如果你去商场买空调，当你到了商场看重了一款空调，你就向售货员下单。售货员去仓库帮你调配物品。这天你热的是在不行了，就催着商家赶紧给你送货，于是你就在商店里面候着他们，直到商家把你和空调一起送回家，一次愉快的购物就结束了。这就是同步调用。

不过，如果我们赶时髦，就坐在家里打开电脑，在电脑上订购了一台空调。当你完成网上支付的时候，对你来说购物过程已经结束了。虽然空调还没有送到家，但是你的任务已经完成了。商家接到你的订单后，就会加紧安平送货，当然这一切已经跟你无关了。你已经支付完成，想干什么就能去干什么，出去溜几圈都不成问题，等送货上门的时候，接到商家的电话，回家一趟签收就完事了。这就是异步调用。

### **并发（Concurrency）和并行（Parallelism）**

并发和并行是两个非常容易被混淆的概念。他们都可以表示两个或者多个任务一起执行，但是侧重点有所不同。并发偏重于多个任务**交替**执行，而多个任务之间有可能还是串行的，而并行是真正意义上的“同时执行”，下图很好地诠释了这点。

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicEJhWlK06CGcWVldNgyN4eYU11dSFHECabJNHebs5nc0S5aENpggzuzTMoeI54ZfhZvYAw4HNfdthicGHuk9hg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

大家排队在一个咖啡机上接咖啡，交替执行，是并发；两台咖啡机上面接咖啡，是并行。

从严格意义上来说，并行的多任务是真的同时执行，而对于并发来说，这个过程只是交替的，一会执行任务A，一会执行任务B，系统会不停地在两者之间切换。但对于外部观察者来说，即使多个任务之间是串行并发的，也会造成多任务间并行执行的错觉。

**并发**说的是在**一个时间段内**，多件事情在这个时间段内**交替执行**。

**并行**说的是多件事情在**同一个时刻**同事发生。

实际上，如果系统内只有一个CPU，而使用多进程或者多线程任务，那么真实环境中这些任务不可能是真实并行的，毕竟一个CPU一次只能执行一条指令，在这种情况下多进程或者多线程就是并发的，而不是并行的（操作系统会不停地切换多任务）。真实的并行也只可能出现在拥有多个CPU的系统中（比如多核CPU）。

### **临界区**

临界区用来表示一种公共资源或者说共享数据，可以被多个线程使用，但是每一次只能有一个线程使用它，一旦临界区资源被占用，其他线程要想使用这个资源就必须等待。

比如，一个办公室里有一台打印机，打印机一次只能执行一个任务。如果小王和小明同时需要打印文件，很明显，如果小王先发了打印任务，打印机就开始打印小王的文件，小明的任务就只能等待小王打印结束后才能打印，这里的打印机就是一个临界区的例子。

在并行程序中，临界区资源是保护的对象，如果意外出现打印机同时执行两个任务的情况，那么最有可能的结果就是打印出来的文件是损坏的文件，它既不是小王想要的，也不是小明想要的。

### **阻塞（Blocking）和非阻塞（Non-Blocking）**

阻塞和非阻塞通常用来形容很多线程间的相互影响。比如一个线程占用了临界区资源，那么其他所有需要这个资源的线程就必须在这个临界区中等待。等待会导致线程挂起，这种情况就是阻塞。此时，如果占用资源的线程一直不愿意释放资源，那么其他线程阻塞在这个临界区上的线程都不能工作。

非阻塞的意思与之相反，它强调没有一个线程可以妨碍其他线程执行，所有的线程都会尝试不断向前执行。

### **死锁（Deadlock）、饥饿（Starvation）和活锁（Livelock）**

**死锁**、**饥饿**和**活锁**都属于多线程的活跃性问题。如果发现上述几种情况，那么相关线程就不再活跃，也就是说它可能很难再继续往下执行了。

**死锁**应该是最糟糕的一种情况了（当然，其他几种情况也好不到哪里去），如下图显示了一个死锁的发生：

![图片](https://mmbiz.qpic.cn/mmbiz_png/xicEJhWlK06CGcWVldNgyN4eYU11dSFHEu2ib1PN1cnaHmnJHUpHiayibZJjZibKgUcMup53nsADiabpCdicM1GlCiaZiaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

A、B、C、D四辆小车都在这种情况下都无法继续行驶了。他们彼此之间相互占用了其他车辆的车道，如果大家都不愿意释放自己的车道，那么这个状况将永远持续下去，谁都不可能通过，死锁是一个很严重的并且应该避免和实时小心的问题，后面的文章中会做更详细的讨论。

**饥饿**是指某一个或者多个线程因为种种原因无法获得所要的资源，导致一直无法执行。比如它的优先级可能太低，而高优先级的线程不断抢占它需要的资源，导致低优先级线程无法工作。在自然界中，母鸡给雏鸟喂食很容易出现这种情况：由于雏鸟很多，食物有限，雏鸟之间的事务竞争可能非常厉害，经常抢不到事务的雏鸟有可能被饿死。线程的饥饿非常类似这种情况。此外，某一个线程一直占着关键资源不放，导致其他需要这个资源的线程无法正常执行，这种情况也是饥饿的一种。于死锁想必，饥饿还是有可能在未来一段时间内解决的（比如，高优先级的线程已经完成任务，不再疯狂执行）。

**活锁**是一种非常有趣的情况。不知道大家是否遇到过这么一种场景，当你要做电梯下楼时，电梯到了，门开了，这是你正准备出去。但很不巧的是，门外一个人当着你的去路，他想进来。于是，你很礼貌地靠左走，礼让对方。同时，对方也非常礼貌的靠右走，希望礼让你。结果，你们俩就又撞上了。于是乎，你们都意识到了问题，希望尽快避让对方，你立即向右边走，同时，他立即向左边走。结果，又撞上了！不过介于人类的智慧，我相信这个动作重复两三次后，你应该可以顺利解决这个问题。因为这个时候，大家都会本能地对视，进行交流，保证这种情况不再发生。但如果这种情况发生在两个线程之间可能就不那么幸运了。如果线程智力不够。且都秉承着“谦让”的原则，主动将资源释放给他人使用，那么久会导致资源不断地在两个线程间跳动，而没有一个线程可以同时拿到所有资源正常执行。这种情况就是活锁。

**死锁的例子**

```
package com.concurrency.dead;

public class DeadlockTest {

    public static void main(String[] args) {
        Obj1 obj1 = new Obj1();
        Obj2 obj2 = new Obj2();
        Thread thread1 = new Thread(new SynAddRunalbe(obj1, obj2, 1, 2, true));
        thread1.setName("thread1");
        thread1.start();
        Thread thread2 = new Thread(new SynAddRunalbe(obj1, obj2, 2, 1, false));
        thread2.setName("thread2");
        thread2.start();
    }

    /**
     * 线程死锁等待演示
     */
    public static class SynAddRunalbe implements Runnable {
        Obj1 obj1;
        Obj2 obj2;
        int a, b;
        boolean flag;

        public SynAddRunalbe(Obj1 obj1, Obj2 obj2, int a, int b, boolean flag) {
            this.obj1 = obj1;
            this.obj2 = obj2;
            this.a = a;
            this.b = b;
            this.flag = flag;
        }

        @Override
        public void run() {
            try {
                if (flag) {
                    synchronized (obj1) {
                        Thread.sleep(100);
                        synchronized (obj2) {
                            System.out.println(a + b);
                        }
                    }
                } else {
                    synchronized (obj2) {
                        Thread.sleep(100);
                        synchronized (obj1) {
                            System.out.println(a + b);
                        }
                    }
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static class Obj1 {
    }

    public static class Obj2 {
    }
}
```

运行上面代码，可以通过jstack查看到死锁信息:

```
"thread2" #15 prio=5 os_prio=0 tid=0x000000001a503800 nid=0x3cf0 waiting for monitor entry [0x000000001b6cf000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.concurrency.dead.DeadlockTest$SynAddRunalbe.run(DeadlockTest.java:47)
        - waiting to lock <0x00000000d67abc70> (a com.concurrency.dead.DeadlockTest$Obj1)
        - locked <0x00000000d67ae298> (a com.concurrency.dead.DeadlockTest$Obj2)
        at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
        - None

"thread1" #14 prio=5 os_prio=0 tid=0x000000001a503000 nid=0x24e0 waiting for monitor entry [0x000000001b5cf000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.concurrency.dead.DeadlockTest$SynAddRunalbe.run(DeadlockTest.java:40)
        - waiting to lock <0x00000000d67ae298> (a com.concurrency.dead.DeadlockTest$Obj2)
        - locked <0x00000000d67abc70> (a com.concurrency.dead.DeadlockTest$Obj1)
        at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
        - None


```

thread1持有com.jvm.visualvm.Demo4$Obj1的锁，等待获取com.jvm.visualvm.Demo4$Obj2的锁
thread2持有com.jvm.visualvm.Demo4$Obj2的锁，等待获取com.jvm.visualvm.Demo4$Obj1的锁，两个线程相互等待获取对方持有的锁，出现死锁。

**饥饿死锁的例子**

```
package com.concurrency.dead;

import java.util.concurrent.*;

public class Starvation {

    private static ExecutorService single = Executors.newSingleThreadExecutor();

    public static class AnotherCallable implements Callable<String> {
        @Override
        public String call() throws Exception {
            System.out.println("in AnotherCallable");
            return "annother success";
        }
    }


    public static class MyCallable implements Callable<String> {
        @Override
        public String call() throws Exception {
            System.out.println("in MyCallable");
            Future<String> submit = single.submit(new AnotherCallable());
            return "success:" + submit.get();
        }
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyCallable task = new MyCallable();
        Future<String> submit = single.submit(task);
        System.out.println(submit.get());
        System.out.println("over");
        single.shutdown();
    }
}

```

线程日志

```

"pool-1-thread-1" #14 prio=5 os_prio=0 tid=0x0000000019fe5800 nid=0x2814 waiting on condition [0x000000001b00e000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000d6a25b10> (a java.util.concurrent.FutureTask)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.FutureTask.awaitDone(FutureTask.java:429)
        at java.util.concurrent.FutureTask.get(FutureTask.java:191)
        at com.concurrency.dead.Starvation$MyCallable.call(Starvation.java:23)
        at com.concurrency.dead.Starvation$MyCallable.call(Starvation.java:18)
        at java.util.concurrent.FutureTask.run$$$capture(FutureTask.java:266)
        at java.util.concurrent.FutureTask.run(FutureTask.java)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
        - <0x00000000d699b2f8> (a java.util.concurrent.ThreadPoolExecutor$Worker)

"Service Thread" #13 daemon prio=9 os_prio=0 tid=0x0000000019ecb000 nid=0x3cf4 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C1 CompilerThread3" #12 daemon prio=9 os_prio=2 tid=0x0000000019e55000 nid=0x151c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C2 CompilerThread2" #11 daemon prio=9 os_prio=2 tid=0x0000000019e3d800 nid=0x2f00 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C2 CompilerThread1" #10 daemon prio=9 os_prio=2 tid=0x0000000019e3d000 nid=0xdc0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C2 CompilerThread0" #9 daemon prio=9 os_prio=2 tid=0x0000000019e3a000 nid=0xae4 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"JDWP Command Reader" #8 daemon prio=10 os_prio=0 tid=0x0000000019db8800 nid=0x2e88 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"JDWP Event Helper Thread" #7 daemon prio=10 os_prio=0 tid=0x0000000019db5000 nid=0xc9c runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"JDWP Transport Listener: dt_socket" #6 daemon prio=10 os_prio=0 tid=0x0000000019da4000 nid=0x3ecc runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x0000000019d9d800 nid=0x1050 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x0000000019d4a800 nid=0x22f0 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x0000000017ece000 nid=0x12a4 in Object.wait() [0x000000001a30f000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000d6508ed0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
        - locked <0x00000000d6508ed0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

   Locked ownable synchronizers:
        - None

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x0000000019d30800 nid=0xe08 in Object.wait() [0x000000001a20f000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000d6506bf8> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x00000000d6506bf8> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

   Locked ownable synchronizers:
        - None

"main" #1 prio=5 os_prio=0 tid=0x0000000002f13800 nid=0x1428 waiting on condition [0x000000000286f000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000d69980e8> (a java.util.concurrent.FutureTask)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.FutureTask.awaitDone(FutureTask.java:429)
        at java.util.concurrent.FutureTask.get(FutureTask.java:191)
        at com.concurrency.dead.Starvation.main(Starvation.java:30)

   Locked ownable synchronizers:
        - None

```

堆栈信息结合图中的代码，可以看出主线程在

```
System.out.println(submit.get());
```

处于等待中，线程池中的工作线程在

```
return "success:" + submit.get();
```

处于等待中，等待获取结果。由于线程池是一个线程，AnotherCallable得不到执行，而被饿死，最终导致了程序死锁的现象。

### 进程和线程

　　**进程**：是指一个内存中运行的应用程序，每个进程都有一个独立的内存空间，一个应用程序可以同时运行多个进程；进程也是程序的一次执行过程，是系统运行程序的基本单位；系统运行一个程序即是一个进程从创建、运行到消亡的过程。

　　**线程**：进程内部的一个独立执行单元；一个进程可以同时并发的运行多个线程，可以理解为一个进程便相当于一个单 CPU 操作系统，而线程便是这个系统中运行的多个任务。

 

**注意：**1、因为一个进程中的多个线程是并发运行的，那么从微观角度看也是有先后顺序的，哪个线程执行完全取决于 CPU 的调度，程序员是干涉不了的。而这也就造成的多线程的随机性。

　　　2、Java 程序的进程里面至少包含两个线程，主进程也就是 main()方法线程，另外一个是垃圾回收机制线程。每当使用 java 命令执行一个类时，实际上都会启动一个 JVM，每一个 JVM 实际上就是在操作系统中启动了一个线程，java 本身具备了垃圾的收集机制，所以在 Java 运行时至少会启动两个线程。

　　　3、由于创建一个线程的开销比创建一个进程的开销小的多，那么我们在开发多任务运行的时候，通常考虑创建多线程，而不是创建多进程。

 　 4、多线程是为了同步完成多个任务，不是为了提高程序运行效率，而是通过提高资源使用效率来提高系统的效率。比如做一个添加功能，需要向很多用户发送短信和邮件，如果需要一分钟，不能能让用户等一分钟才响应，可以使用多线程做后台发送，如果是单核，则多个任务交替的在单个 CPU 上运行，理论上比不使用多线程耗时多，因为CPU切换任务需要耗费时间，如果是多核，则可以提高资源的使用效率，多核可以充分利用上。

### 并发级别

由于临界区的存在，多线程之间的并发必须受到控制。根据控制并发的策略，我们可以把并发的级别分为**阻塞**、**无饥饿**、**无障碍**、**无锁**、**无等待**几种。

#### 阻塞

一个线程是阻塞的，那么在其他线程释放资源之前，当前线程无法继续执行。当我们使用synchronized关键字或者重入锁时，我们得到的就是阻塞的线程。

synchronize关键字和重入锁都试图在执行后续代码前，得到临界区的锁，如果得不到，线程就会被挂起等待，直到占有了所需资源为止。

#### 无饥饿(Starvation-Free)

如果线程之间是有优先级的，那么线程调度的时候总是会倾向于先满足高优先级的线程。也就是说，对于同一个资源的分配，是不公平的！图1.7中显示了非公平锁与公平锁两种情况(五角星表示高优先级线程)。对于非公平锁来说，系统允许高优先级的线程插队。这样有可能导致低优先级线程产生饥饿。但如果锁是公平的，按照先来后到的规则，那么饥饿就不会产生，不管新来的线程优先级多高，要想获得资源，就必须乖乖排队，这样所有的线程都有机会执行。

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715171755904-1855306497.png)

#### 无障碍(Obstruction-Free)

无障碍是一种最弱的非阻塞调度。两个线程如果无障碍地执行，那么不会因为临界区的问题导致一方被挂起。换言之，大家都可以大摇大摆地进入临界区了。那么大家一起修改共享数据，把数据改坏了怎么办呢？对于无障碍的线程来说，一旦检测到这种情况，它就会立即对自己所做的修改进行回滚，确保数据安全。但如果没有数据竞争发生，那么线程就可以顺利完成自己的工作，走出临界区。

如果说阻塞的控制方式是悲观策略，也就是说，系统认为两个线程之间很有可能发生不幸的冲突，因此以保护共享数据为第一优先级，相对来说，非阻塞的调度就是一种乐观的策略。它认为多个线程之间很有可能不会发生冲突，或者说这种概率不大。因此大家都应该无障碍地执行，但是一旦检测到冲突，就应该进行回滚。

从这个策略中也可以看到，无障碍的多线程程序并不一定能顺畅运行。因为当临界区中存在严重的冲突时，所有的线程可能都会不断地回滚自己的操作，而没有一个线程可以走出临界区。这种情况会影响系统的正常执行。所以，我们可能会非常希望在这一堆线程中，至少可以有一个线程能够在有限的时间内完成自己的操作，而退出临界区。至少这样可以保证系统不会在临界区中进行无限的等待。

一种可行的无障碍实现可以依赖一个"一致性标记"来实现。线程在操作之前，先读取并保存这个标记，在操作完成后，再次读取，检查这个标记是否被更改过，如果两者是一致的，则说明资源访问没有冲突。如果不一致，则说明资源可能在操作过程中与其他线程冲突，需要重试操作。而任何对资源有修改操作的线程，在修改数据前，都需要更新这个一致性标记，表示数据不再安全。

数据库中乐观锁，应该比较熟悉，表中需要一个字段version(版本号)，每次更新数据version+1，更新的时候将版本号作为条件进行更新，根据更新影响的行数判断更新是否成功，伪代码如下：

```java
1.查询数据，此时版本号为w_v
2.打开事务
3.做一些业务操作
4.update t set version = version+1 where id = 记录id and version = w_v;//此行会返回影响的行数c
5.if(c>0){
		//提交事务
	}else{
		//回滚事务
	}
```

多个线程更新同一条数据的时候，数据库会对当前数据加锁，同一时刻只有一个线程可以执行更新语句。

#### 无锁(Lock-Free)

无锁的并行都是无障碍的。在无锁的情况下，所有的线程都能尝试对临界区进行访问，但不同的是，无锁的并发保证必然有一个线程能够在有限步内完成操作离开临界区。

在无锁的调用中，一个典型的特点是可能会包含一个无穷循环。在这个循环中，线程会不断尝试修改共享变量。如果没有冲突，修改成功，那么程序退出，否则继续尝试修改。但无论如何，无锁的并行总能保证有一个线程是可以胜出的，不至于全军覆没。至于临界区中竞争失败的线程，他们必须不断重试，直到自己获胜。如果运气很不好，总是尝试不成功，则会出现类似饥饿的先写，线程会停止。

下面就是一段无锁的示意代码，如果修改不成功，那么循环永远不会停止。

```java
while(!atomicVar.compareAndSet(localVar, localVar+1)){
		localVal = atomicVar.get();
}
```

#### 无等待

无锁只要求有一个线程可以在有限步内完成操作，而无等待则在无锁的基础上更进一步扩展。它要求所有线程都必须在有限步内完成，这样不会引起饥饿问题。如果限制这个步骤的上限，还可以进一步分解为有界无等待和线程数无关的无等待等几种，他们之间的区别只是对循环次数的限制不同。

一种典型的无等待结果就是RCU(Read Copy Update)。它的基本思想是，对数据的读可以不加控制。因此，所有的读线程都是无等待的，它们既不会被锁定等待也不会引起任何冲突。但在写数据的时候，先获取原始数据的副本，接着只修改副本数据(这就是为什么读可以不加控制)，修改完成后，在合适的时机回写数据。

## 带着BAT大厂的面试问题去理解

- 多线程的出现是要解决什么问题的?
- 线程不安全是指什么? 举例说明
- 并发出现线程不安全的本质什么? 可见性，原子性和有序性。
- Java是怎么解决并发问题的? 3个关键字，JMM和8个Happens-Before
- 线程安全是不是非真即假? 不是
- 线程安全有哪些实现思路?
- 如何理解并发和并行的区别?

### 为什么需要多线程

众所周知，CPU、内存、I/O 设备的速度是有极大差异的，为了合理利用 CPU 的高性能，平衡这三者的速度差异，计算机体系结构、操作系统、编译程序都做出了贡献，主要体现为:

- CPU 增加了缓存，以均衡与内存的速度差异；// 导致 `可见性`问题
- 操作系统增加了进程、线程，以分时复用 CPU，进而均衡 CPU 与 I/O 设备的速度差异；// 导致 `原子性`问题
- 编译程序优化指令执行次序，使得缓存能够得到更加合理地利用。// 导致 `有序性`问题

### 线程不安全示例

如果多个线程对同一个共享数据进行访问而不采取同步操作的话，那么操作的结果是不一致的。

以下代码演示了 1000 个线程同时对 cnt 执行自增操作，操作结束之后它的值有可能小于 1000。

```
public class ThreadUnsafeExample {

    private int cnt = 0;

    public void add() {
        cnt++;
    }

    public int get() {
        return cnt;
    }
}
```

测试

```
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    ThreadUnsafeExample example = new ThreadUnsafeExample();
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}

//返回结果
// 997  结果总是小于1000
```

## 并发出现问题的根源: 并发三要素

上述代码输出为什么不是1000? 并发出现问题的根源是什么?

### 可见性: CPU缓存引起

可见性：一个线程对共享变量的修改，另外一个线程能够立刻看到。

举个简单的例子，看下面这段代码：

```java
//线程1执行的代码
int i = 0;
i = 10;
 
//线程2执行的代码
j = i;
```

假若执行线程1的是CPU1，执行线程2的是CPU2。由上面的分析可知，当线程1执行 i =10这句时，会先把i的初始值加载到CPU1的高速缓存中，然后赋值为10，那么在CPU1的高速缓存当中i的值变为10了，却没有立即写入到主存当中。

此时线程2执行 j = i，它会先去主存读取i的值并加载到CPU2的缓存当中，注意此时内存当中i的值还是0，那么就会使得j的值为0，而不是10.

这就是可见性问题，线程1对变量i修改了之后，线程2没有立即看到线程1修改的值。

### 原子性: 分时复用引起

原子性：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

经典的**转账问题**：比如从账户A向账户B转1000元，那么必然包括2个操作：从账户A减去1000元，往账户B加上1000元。

试想一下，如果这2个操作不具备原子性，会造成什么样的后果。假如从账户A减去1000元之后，操作突然中止。然后又从B取出了500元，取出500元之后，再执行 往账户B加上1000元 的操作。这样就会导致账户A虽然减去了1000元，但是账户B没有收到这个转过来的1000元。

所以这2个操作必须要具备原子性才能保证不出现一些意外的问题。

### 有序性: 重排序引起

有序性：即程序执行的顺序按照代码的先后顺序执行。举个简单的例子，看下面这段代码：

```java
int i = 0;              
boolean flag = false;
i = 1;                //语句1  
flag = true;          //语句2
    
```

上面代码定义了一个int型变量，定义了一个boolean类型变量，然后分别对两个变量进行赋值操作。从代码顺序上看，语句1是在语句2前面的，那么JVM在真正执行这段代码的时候会保证语句1一定会在语句2前面执行吗? 不一定，为什么呢? 这里可能会发生指令重排序（Instruction Reorder）。

在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。重排序分三种类型：

- 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
- 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
- 内存系统的重排序。由于处理器使用缓存和读 / 写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

从 java 源代码到最终实际执行的指令序列，会分别经历下面三种重排序：

![img](png/juc/java-jmm-3.png)

上述的 1 属于编译器重排序，2 和 3 属于处理器重排序。这些重排序都可能会导致多线程程序出现内存可见性问题。对于编译器，JMM 的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器重排序都要禁止）。对于处理器重排序，JMM 的处理器重排序规则会要求 java 编译器在生成指令序列时，插入特定类型的内存屏障（memory barriers，intel 称之为 memory fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序（不是所有的处理器重排序都要禁止）。







？？

## 深入理解进程和线程

### 进程

进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础。程序是指令、数据及其组织形式的描述，进程是程序的实体。

**进程具有的特征：**

- **动态性**：进程是程序的一次执行过程，是临时的，有生命期的，是动态产生，动态消亡的
- **并发性**：任何进程都可以同其他进行一起并发执行
- **独立性**：进程是系统进行资源分配和调度的一个独立单位
- **结构性**：进程由程序，数据和进程控制块三部分组成

我们经常使用windows系统，经常会看见.exe后缀的文件，双击这个.exe文件的时候，这个文件中的指令就会被系统加载，那么我们就能得到一个关于这个.exe程序的进程。进程是**“活”**的，或者说是正在被执行的。

window中打开任务管理器，可以看到当前系统中正在运行的进程，如下图：

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173208647-1285618678.png)

### 线程

线程是轻量级的进程，是程序执行的最小单元，使用多线程而不是多进程去进行并发程序的设计，是因为线程间的切换和调度的成本远远小于进程。

我们用一张图来看一下线程的状态图：

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173221102-740428709.png)

线程的所有状态在**java.lang.Thread中的State**枚举中有定义，如：

```java
public enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;
}
```

线程几个状态的介绍：

- **New**：表示刚刚创建的线程，这种线程还没有开始执行
- **RUNNABLE**：运行状态，线程的start()方法调用后，线程会处于这种状态
- **BLOCKED**：阻塞状态。当线程在执行的过程中遇到了synchronized同步块，但这个同步块被其他线程已获取还未释放时，当前线程将进入阻塞状态，会暂停执行，直到获取到锁。当线程获取到锁之后，又会进入到运行状态（RUNNABLE）
- **WAITING**：等待状态。和TIME_WAITING都表示等待状态，区别是WAITING会进入一个无时间限制的等，而TIME_WAITING会进入一个有限的时间等待，那么等待的线程究竟在等什么呢？一般来说，WAITING的线程正式在等待一些特殊的事件，比如，通过wait()方法等待的线程在等待notify()方法，而通过join()方法等待的线程则会等待目标线程的终止。一旦等到期望的事件，线程就会再次进入RUNNABLE运行状态。
- **TERMINATED**：表示结束状态，线程执行完毕之后进入结束状态。

**注意：从NEW状态出发后，线程不能在回到NEW状态，同理，处理TERMINATED状态的线程也不能在回到RUNNABLE状态**

### 进程与线程的一个简单解释

进程（process）和线程（thread）是操作系统的基本概念，但是它们比较抽象，不容易掌握。

1.计算机的核心是CPU，它承担了所有的计算任务。它就像一座工厂，时刻在运行。
![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173234911-1877518755.png)

2.假定工厂的电力有限，一次只能供给一个车间使用。也就是说，一个车间开工的时候，其他车间都必须停工。背后的含义就是，单个CPU一次只能运行一个任务。
![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173245739-2006345581.png)

3.进程就好比工厂的车间，它代表CPU所能处理的单个任务。任一时刻，CPU总是运行一个进程，其他进程处于非运行状态。
![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173255722-1145592188.png)

4.一个车间里，可以有很多工人。他们协同完成一个任务。
![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173307546-1422198190.png)

5.线程就好比车间里的工人。一个进程可以包括多个线程。
![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173320273-1643817671.png)

6.车间的空间是工人们共享的，比如许多房间是每个工人都可以进出的。这象征一个进程的内存空间是共享的，每个线程都可以使用这些共享内存。
![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173331184-277572295.png)

7.可是，每间房间的大小不同，有些房间最多只能容纳一个人，比如厕所。里面有人的时候，其他人就不能进去了。这代表一个线程使用某些共享内存时，其他线程必须等它结束，才能使用这一块内存。
![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173341721-316431201.png)

8.一个防止他人进入的简单方法，就是门口加一把锁。先到的人锁上门，后到的人看到上锁，就在门口排队，等锁打开再进去。这就叫"互斥锁"（Mutual exclusion，缩写 Mutex），防止多个线程同时读写某一块内存区域。
![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173355970-1228502153.png)

9.还有些房间，可以同时容纳n个人，比如厨房。也就是说，如果人数大于n，多出来的人只能在外面等着。这好比某些内存区域，只能供给固定数目的线程使用。
![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173408814-1115142474.png)

10.这时的解决方法，就是在门口挂n把钥匙。进去的人就取一把钥匙，出来时再把钥匙挂回原处。后到的人发现钥匙架空了，就知道必须在门口排队等着了。这种做法叫做"信号量"（Semaphore），用来保证多个线程不会互相冲突。
![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173418761-532901489.png)

11.操作系统的设计，因此可以归结为三点：
（1）以多进程形式，允许多个任务同时运行；
（2）以多线程形式，允许单个任务分成不同的部分运行；
（3）提供协调机制，一方面防止进程之间和线程之间产生冲突，另一方面允许进程之间和线程之间共享资源。

## 如何创建多线程

使用线程的方法:

- 继承 Thread 类。
- 实现 Runnable 接口
- 使用匿名内部类创建线程
- 使用Callable和Future接口创建线程

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

### ①**继承 Thread 类**

同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口。

当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

```
package com.concurrency.thread;

/**
 * 步骤：1、定义一个线程类 A 继承于 java.lang.Thread 类
 * 　　　2、在 A 类中覆盖 Thread 类的 run() 方法
 * 　　　3、在 run() 方法中编写需要执行的操作
 * 　　　4、在 main 方法（线程）中，创建线程对象，并启动线程
 * 　　　　　创建线程类：A类 a = new A()类；
 * 　　　　　调用 start() 方法启动线程：a.start();
 */
public class ThreadExtends {

    public static void main(String[] args) {
        SubThread subThread1 = new SubThread();
        subThread1.start();
        SubThread subThread2 = new SubThread();
        subThread2.start();
    }
}

class SubThread extends Thread {
    //2.重写run方法，方法内实现此子线程要完成的功能
    @Override
    public void run() {
        for (int i = 0; i <= 5; i++) {
            System.out.println(Thread.currentThread().getName() + ":" + i);
        }
    }
}
```

**注意：**1、不能通过Thread实现类对象的run()去启动一个线程，此时只是主线程调用方法而已，并没有启动线程，要启动线程，必须通过Start()方法

　　　2、一个线程只能够执行一次start()，start()中会判断threadStatus的状态是否为0，不为0则抛出异常，所以一个线程调用两次start()会报异常

```
Exception in thread "main" Disconnected from the target VM, address: '127.0.0.1:57072', transport: 'socket'
java.lang.IllegalThreadStateException
	at java.lang.Thread.start(Thread.java:708)
	at com.concurrency.thread.ThreadExtends.main(ThreadExtends.java:16) 
```

### ②**实现 Runnable 接口**

需要实现 run() 方法。

通过 Thread 调用 start() 方法来启动线程。

1、Runnable接口应由任何类实现，其实例将由线程执行。 该类必须定义一个无参数的方法，称为run 。 
2、该接口旨在为希望在活动时执行代码的对象提供一个通用协议。此类整个只有一个 run() 抽象方法

```
package com.concurrency.thread;

/**
 * 步骤：1、定义一个线程类 A 实现于 java.lang.Runnable 接口（注意：A类不是线程类,没有 start()方法，不能直接 new A 的实例启动线程）
 * 　　　2、在 A 类中覆盖 Runnable 接口的 run() 方法
 * 　　　3、在 run() 方法中编写需要执行的操作
 * 　　　4、在 main 方法（线程）中，创建线程对象，并启动线程
 * 　　　　　　创建线程类：Thread t = new Thread( new A类() ) ；
 * 　　　　　　调用 start() 方法启动线程：t.start();
 */
public class ThreadInterface {

    public static void main(String [] args){
        //此程序存在线程的安全问题，打印车票时，会出现重票、错票，后面线程同步会讲到
        Window window=new Window();
        Thread thread1=new Thread(window,"窗口一");
        Thread thread2=new Thread(window,"窗口二");
        Thread thread3=new Thread(window,"窗口三");
        thread1.start();
        thread2.start();
        thread3.start();
    }
}
class Window implements  Runnable{
    int ticket=10;
    @Override
    public void run(){
        while (true){
            if(ticket > 0){
                try {
                    Thread.currentThread().sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"售票，票号为："+ticket--);
            }else {
                break;
            }
        }
    }
}

```

**哪个方式好？实现的方式优于继承的方式
**

　　why? ①避免java单继承的局限性
　　　　　②如果多个线程要操作同一份资源，更适合使用实现的方式

**注意：**此程序存在线程的安全问题，打印车票时，会出现重票、错票，下一篇线程同步会讲到



### ③**使用匿名内部类创建线程**

```
public static void main(String[] args) {
    for(int i = 0 ; i < 10 ; i++){
        System.out.println("玩游戏"+i);
        if(i==5){
            new Thread(new Runnable() {
                @Override
                public void run() {
                    for(int i = 0 ; i < 10 ;i++){
                        System.out.println("播放音乐"+i);
                    }
                }
            }).start();
        }
    }
}
```

### ④使用Callable和Future接口创建线程

使用Callable和Future接口创建线程。

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

```
public class ThreadTest {
    public static void main(String[] args) {
        Callable<Integer> myCallable = new MyCallable();    // 创建MyCallable对象
        FutureTask<Integer> ft = new FutureTask<Integer>(myCallable); //使用FutureTask来包装MyCallable对象
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 30) {
                Thread thread = new Thread(ft);   //FutureTask对象作为Thread对象的target创建新的线程
                thread.start();                      //线程进入到就绪状态
            }
        }
        System.out.println("主线程for循环执行完毕..");
        try {
            int sum = ft.get();            //取得新创建的新线程中的call()方法返回的结果
            System.out.println("sum = " + sum);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}

class MyCallable implements Callable<Integer> {
    private int i = 0;

    // 与run()方法不同的是，call()方法具有返回值
    @Override
    public Integer call() {
        int sum = 0;
        for (; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            sum += i;
        }
        return sum;
    }
}
```

### 实现接口 VS 继承 Thread

实现接口会更好一些，因为:

- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

## 线程的状态

在 Thread 类中，有一个枚举内部类：

```
    /**
     * A thread state.  A thread can be in one of the following states:
     * <ul>
     * <li>{@link #NEW}<br>
     *     A thread that has not yet started is in this state.
     *     </li>
     * <li>{@link #RUNNABLE}<br>
     *     A thread executing in the Java virtual machine is in this state.
     *     </li>
     * <li>{@link #BLOCKED}<br>
     *     A thread that is blocked waiting for a monitor lock
     *     is in this state.
     *     </li>
     * <li>{@link #WAITING}<br>
     *     A thread that is waiting indefinitely for another thread to
     *     perform a particular action is in this state.
     *     </li>
     * <li>{@link #TIMED_WAITING}<br>
     *     A thread that is waiting for another thread to perform an action
     *     for up to a specified waiting time is in this state.
     *     </li>
     * <li>{@link #TERMINATED}<br>
     *     A thread that has exited is in this state.
     *     </li>
     * </ul>
     *
     * <p>
     * A thread can be in only one state at a given point in time.
     * These states are virtual machine states which do not reflect
     * any operating system thread states.
     *
     * @since   1.5
     * @see #getState
     */
    public enum State {
        /**
         * Thread state for a thread which has not yet started.
         */
        NEW,

        /**
         * Thread state for a runnable thread.  A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         */
        RUNNABLE,

        /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * {@link Object#wait() Object.wait}.
         */
        BLOCKED,

        /**
         * Thread state for a waiting thread.
         * A thread is in the waiting state due to calling one of the
         * following methods:
         * <ul>
         *   <li>{@link Object#wait() Object.wait} with no timeout</li>
         *   <li>{@link #join() Thread.join} with no timeout</li>
         *   <li>{@link LockSupport#park() LockSupport.park}</li>
         * </ul>
         *
         * <p>A thread in the waiting state is waiting for another thread to
         * perform a particular action.
         *
         * For example, a thread that has called <tt>Object.wait()</tt>
         * on an object is waiting for another thread to call
         * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
         * that object. A thread that has called <tt>Thread.join()</tt>
         * is waiting for a specified thread to terminate.
         */
        WAITING,

        /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * the following methods with a specified positive waiting time:
         * <ul>
         *   <li>{@link #sleep Thread.sleep}</li>
         *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
         *   <li>{@link #join(long) Thread.join} with timeout</li>
         *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
         *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
         * </ul>
         */
        TIMED_WAITING,

        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         */
        TERMINATED;
    }
```



![ThreadState](png\JUC\ThreadState.png)

#### 新建状态（**NEW**）

**使用 new 创建一个线程，仅仅只是在堆中分配了内存空间**

　　　新建状态下，线程还没有调用 start()方法启动，只是存在一个线程对象而已

　　　Thread t = new Thread();//这就是t线程的新建状态

#### **可运行状态（runnable）**

**新建状态调用 start() 方法，进入可运行状态。而这个又分成两种状态，ready 和 running，分别表示就绪状态和运行状态**

　　就绪状态（Ready）

​				线程对象调用了 start() 方法，等待 JVM 的调度，（此时该线程并没有运行）

　　运行状态（Running）

​				线程对象获得 JVM 调度，如果存在多个 CPU，那么运行多个线程并行运行

　　**注意：线程对象只能调用一次 start() 方法，否则报错：illegaThreadStateExecptiong**

#### **阻塞状态（Blocked）**

**正在运行的线程因为某种原因放弃 CPU，暂时停止运行，就会进入阻塞状态。此时 JVM 不会给线程分配 CPU，知道线程重新进入就绪状态，才有机会转到 运行状态。**

　　注意：阻塞状态只能先进入就绪状态，不能直接进入运行状态

　　阻塞状态分为两种情况：

　　　　①、当线程 A 处于可运行状态中，试图获取同步锁时，却被 B 线程获取，此时 JVM 把当前 A 线程放入锁池中，A线程进入阻塞状态

　　　　②、当线程处于运行状态时，发出了 IO 请求，此时进入阻塞状态

    1）等待阻塞：通过调用线程的wait()方法，让线程等待某工作的完成。
    
    2）同步阻塞：线程在获取synchronized同步锁失败（因为锁被其他线程占用），它会进入同步阻塞状态。
    
    3）其他阻塞：通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或超时、或者I/O处理完毕时，线程重新转入就绪状态。
#### ****无限期等待**（Waiting）**

**等待状态只能被其他线程唤醒，此时使用的是无参数的 wait() 方法**

　　①、当线程处于运行状态时，调用了 wait() 方法，此时 JVM 把该线程放入等待池中

等待其它线程显式地唤醒，否则不会被分配 CPU 时间片。

| 进入方法                                   | 退出方法                             |
| ------------------------------------------ | ------------------------------------ |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法                    | -                                    |

#### ****限期等待**（Timed waiting）**

**调用了带参数的 wait（long time）或 sleep(long time) 方法**

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

　　①、当线程处于运行状态时，调用 Object.wait(long time) 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述，此时 JVM 把该线程放入等待池中。

　　②、当前线程调用了 sleep(long time) 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。



睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。

| 进入方法                                 | 退出方法                                        |
| ---------------------------------------- | ----------------------------------------------- |
| Thread.sleep() 方法                      | 时间结束                                        |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
| LockSupport.parkNanos() 方法             | -                                               |
| LockSupport.parkUntil() 方法             | -                                               |

#### **终止状态（**Terminated**）**

**通常称为死亡状态，表示线程终止**

　　①、正常终止，执行完 run() 方法，正常结束

　　②、强制终止，如调用 stop() 方法或 destory() 方法

　　③、异常终止，执行过程中发生异常

## 线程的基本操作

### 新建线程

新建线程很简单。只需要使用new关键字创建一个线程对象，然后调用它的start()启动线程即可。

```java
Thread thread1 = new Thread();
t1.start();
```

那么线程start()之后，会干什么呢？线程有个run()方法，start()会创建一个新的线程并让这个线程执行run()方法。

这里需要注意，下面代码也能通过编译，也能正常执行。但是，却不能新建一个线程，而是在当前线程中调用run()方法，将run方法只是作为一个普通的方法调用。

```java
Thread thread1 = new Thread1();
thread1.run();
```

所以，希望大家注意，调用start方法和直接调用run方法的区别。

**start方法是启动一个线程，run方法只会在当前线程中串行的执行run方法中的代码。**

默认情况下， 线程的run方法什么都没有，启动一个线程之后马上就结束了，所以如果你需要线程做点什么，需要把您的代码写到run方法中，所以必须重写run方法。

```java
Thread thread1 = new Thread() {
            @Override
            public void run() {
                System.out.println("hello,我是一个线程!");
            }
        };
thread1.start();
```

上面是使用匿名内部类实现的，重写了Thread的run方法，并且打印了一条信息。**我们可以通过继承Thread类，然后重写run方法，来自定义一个线程。**但考虑java是单继承的，从扩展性上来说，我们实现一个接口来自定义一个线程更好一些，java中刚好提供了Runnable接口来自定义一个线程。

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

Thread类有一个非常重要的构造方法：

```java
public Thread(Runnable target)
```

我们在看一下Thread的run方法：

```java
public void run() {
        if (target != null) {
            target.run();
        }
    }
```

当我们启动线程的start方法之后，线程会执行run方法，run方法中会调用Thread构造方法传入的target的run方法。

**实现Runnable接口是比较常见的做法，也是推荐的做法。**

### 终止线程（stop）

一般来说线程执行完毕就会结束，无需手动关闭。但是如果我们想关闭一个正在运行的线程，有什么方法呢？可以看一下Thread类中提供了一个stop()方法，调用这个方法，就可以立即将一个线程终止，非常方便。

```java
package com.concurrency.life;

import java.util.concurrent.TimeUnit;

public class StopThread {

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread() {
            @Override
            public void run() {
                System.out.println("start");
                boolean flag = true;
                while (flag) {
                    ;
                }
                System.out.println("end");
            }
        };
        thread1.setName("thread1");
        thread1.start();
        //当前线程休眠1秒
        TimeUnit.SECONDS.sleep(1);
        //关闭线程thread1
        thread1.stop();
        //输出线程thread1的状态
        System.out.println("线程状态"+thread1.getState());
        //当前线程休眠1秒
        TimeUnit.SECONDS.sleep(1);
        //输出线程thread1的状态
        System.out.println("线程状态"+thread1.getState());
    }
}

```

运行代码，输出：

```txt
start
线程状态RUNNABLE
线程状态TERMINATED
```

代码中有个死循环，调用stop方法之后，线程thread1的状态变为TERMINATED（结束状态），线程停止了。

这个方法是一个废弃的方法，也就是说，在将来，jdk可能就会移除该方法。

stop方法为何会被废弃而不推荐使用？stop方法过于暴力，强制把正在执行的方法停止了。

大家是否遇到过这样的场景：**电力系统需要维修，此时咱们正在写代码，维修人员直接将电源关闭了，代码还没保存的，是不是很崩溃，这种方式就像直接调用线程的stop方法类似。线程正在运行过程中，被强制结束了，可能会导致一些意想不到的后果。可以给大家发送一个通知，告诉大家保存一下手头的工作，将电脑关闭。**

### 线程中断（interrupt）

在java中，线程中断是一种重要的线程写作机制，从表面上理解，中断就是让目标线程停止执行的意思，实际上并非完全如此。在上面中，我们已经详细讨论了stop方法停止线程的坏处，jdk中提供了更好的中断线程的方法。严格的说，线程中断并不会使线程立即退出，而是给线程发送一个通知，告知目标线程，有人希望你退出了！至于目标线程接收到通知之后如何处理，则完全由目标线程自己决定，这点很重要，如果中断后，线程立即无条件退出，我们又会到stop方法的老问题。

Thread提供了3个与线程中断有关的方法，这3个方法容易混淆，大家注意下：

```java
public void interrupt() //中断线程
public boolean isInterrupted() //判断线程是否被中断
public static boolean interrupted()  //判断线程是否被中断，并清除当前中断状态
```

**interrupt()**方法是一个**实例方法**，它通知目标线程中断，也就是设置中断标志位为true，中断标志位表示当前线程已经被中断了。**isInterrupted()**方法也是一个**实例方法**，它判断当前线程是否被中断（通过检查中断标志位）。最后一个方法**interrupted()**是一个**静态方法**，返回boolean类型，也是用来判断当前线程是否被中断，但是同时会清除当前线程的中断标志位的状态。

```java
while (true) {
            if (this.isInterrupted()) {
                System.out.println("我要退出了!");
                break;
            }
        }
    }
};
thread1.setName("thread1");
thread1.start();
TimeUnit.SECONDS.sleep(1);
thread1.interrupt();
```

上面代码中有个死循环，interrupt()方法被调用之后，线程的中断标志将被置为true，循环体中通过检查线程的中断标志是否为ture（`this.isInterrupted()`）来判断线程是否需要退出了。

再看一种中断的方法：

```java
static volatile boolean isStop = false;

public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new Thread() {
        @Override
        public void run() {
            while (true) {
                if (isStop) {
                    System.out.println("我要退出了!");
                    break;
                }
            }
        }
    };
    thread1.setName("thread1");
    thread1.start();
    TimeUnit.SECONDS.sleep(1);
    isStop = true;
}
```

代码中通过一个变量isStop来控制线程是否停止。

通过变量控制和线程自带的interrupt方法来中断线程有什么区别呢？

如果一个线程调用了sleep方法，一直处于休眠状态，通过变量控制，还可以中断线程么？大家可以思考一下。

此时只能使用线程提供的interrupt方法来中断线程了。

```java
public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new Thread() {
        @Override
        public void run() {
            while (true) {
                //休眠100秒
                try {
                    TimeUnit.SECONDS.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("我要退出了!");
                break;
            }
        }
    };
    thread1.setName("thread1");
    thread1.start();
    TimeUnit.SECONDS.sleep(1);
    thread1.interrupt();
}
```

调用interrupt()方法之后，线程的sleep方法将会抛出`InterruptedException`异常。

```java
Thread thread1 = new Thread() {
    @Override
    public void run() {
        while (true) {
            //休眠100秒
            try {
                TimeUnit.SECONDS.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (this.isInterrupted()) {
                System.out.println("我要退出了!");
                break;
            }
        }
    }
};
```

运行上面的代码，发现程序无法终止。为什么？

代码需要改为：

```java
Thread thread1 = new Thread() {
    @Override
    public void run() {
        while (true) {
            //休眠100秒
            try {
                TimeUnit.SECONDS.sleep(100);
            } catch (InterruptedException e) {
                this.interrupt();
                e.printStackTrace();
            }
            if (this.isInterrupted()) {
                System.out.println("我要退出了!");
                break;
            }
        }
    }
};
```

上面代码可以终止。

**注意：sleep方法由于中断而抛出异常之后，线程的中断标志会被清除（置为false），所以在异常中需要执行this.interrupt()方法，将中断标志位置为true**

### 等待（wait）和通知（notify）

为了支持多线程之间的协作，JDK提供了两个非常重要的方法：等待wait()方法和通知notify()方法。这2个方法并不是在Thread类中的，而是在Object类中定义的。这意味着所有的对象都可以调用者两个方法。

```java
public final void wait() throws InterruptedException;
public final native void notify();
```

当在一个对象实例上调用wait()方法后，当前线程就会在这个对象上等待。这是什么意思？比如在线程A中，调用了obj.wait()方法，那么线程A就会停止继续执行，转为等待状态。等待到什么时候结束呢？线程A会一直等到其他线程调用obj.notify()方法为止，这时，obj对象成为了多个线程之间的有效通信手段。

那么wait()方法和notify()方法是如何工作的呢？如图2.5展示了两者的工作过程。如果一个线程调用了object.wait()方法，那么它就会进出object对象的等待队列。这个队列中，可能会有多个线程，因为系统可能运行多个线程同时等待某一个对象。当object.notify()方法被调用时，它就会从这个队列中随机选择一个线程，并将其唤醒。这里希望大家注意一下，这个选择是不公平的，并不是先等待线程就会优先被选择，这个选择完全是随机的。

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173833735-1878337611.png)

除notify()方法外，Object独享还有一个nofiyAll()方法，它和notify()方法的功能类似，不同的是，它会唤醒在这个等待队列中所有等待的线程，而不是随机选择一个。

这里强调一点，Object.wait()方法并不能随便调用。它必须包含在对应的synchronize语句汇总，无论是wait()方法或者notify()方法都需要首先获取目标独享的一个监视器。图2.6显示了wait()方法和nofiy()方法的工作流程细节。其中T1和T2表示两个线程。T1在正确执行wait()方法钱，必须获得object对象的监视器。而wait()方法在执行后，会释放这个监视器。这样做的目的是使其他等待在object对象上的线程不至于因为T1的休眠而全部无法正常执行。

线程T2在notify()方法调用前，也必须获得object对象的监视器。所幸，此时T1已经释放了这个监视器，因此，T2可以顺利获得object对象的监视器。接着，T2执行了notify()方法尝试唤醒一个等待线程，这里假设唤醒了T1。T1在被唤醒后，要做的第一件事并不是执行后续代码，而是要尝试重新获得object对象的监视器，而这个监视器也正是T1在wait()方法执行前所持有的那个。如果暂时无法获得，则T1还必须等待这个监视器。当监视器顺利获得后，T1才可以在真正意义上继续执行。

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173844786-262740242.png)

给大家上个例子：

```java
package com.concurrency.life;

public class WaitThread {

    static Object object = new Object();

    public static class T1 extends Thread {
        @Override
        public void run() {
            synchronized (object) {
                System.out.println(System.currentTimeMillis() + ":T1 start!");
                try {
                    System.out.println(System.currentTimeMillis() + ":T1 wait for object");
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(System.currentTimeMillis() + ":T1 end!");
            }
        }
    }

    public static class T2 extends Thread {
        @Override
        public void run() {
            synchronized (object) {
                System.out.println(System.currentTimeMillis() + ":T2 start，notify one thread! ");
                object.notify();
                System.out.println(System.currentTimeMillis() + ":T2 end!");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new T1().start();
        new T2().start();
    }
}

```

运行结果：

```txt
1648102278177:T1 start!
1648102278177:T1 wait for object
1648102278193:T2 start，notify one thread! 
1648102278193:T2 end!
1648102280205:T1 end!
```

注意下打印结果，T2调用notify方法之后，T1并不能立即继续执行，而是要等待T2释放objec投递锁之后，T1重新成功获取锁后，才能继续执行。因此最后2行日志相差了2秒（因为T2调用notify方法后休眠了2秒）。

**注意：Object.wait()方法和Thread.sleeep()方法都可以让现场等待若干时间。除wait()方法可以被唤醒外，另外一个主要的区别就是wait()方法会释放目标对象的锁，而Thread.sleep()方法不会释放锁。**

再给大家讲解一下wait()，notify()，notifyAll()，加深一下理解：

可以这么理解，obj对象上有2个队列，如图1，**q1：等待队列，q2：准备获取锁的队列**；两个队列都为空。

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173900128-1828775374.png)

**obj.wait()过程：**

```java
synchronize(obj){
	obj.wait();
}
```

假如有3个线程，t1、t2、t3同时执行上面代码，t1、t2、t3会进入q2队列，如图2，进入q2的队列的这些线程才有资格去争抢obj的锁，假设t1争抢到了，那么t2、t3机型在q2中等待着获取锁，t1进入代码块执行wait()方法，此时t1会进入q1队列，然后系统会通知q2队列中的t2、t3去争抢obj的锁，抢到之后过程如t1的过程。最后t1、t2、t3都进入了q1队列，如图3。

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173912452-1395878692.png)

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173918104-1131227097.png)

上面过程之后，又来了线程t4执行了notify()方法，如下：**

```java
synchronize(obj){
	obj.notify();
}
```

t4会获取到obj的锁，然后执行notify()方法，系统会从q1队列中随机取一个线程，将其加入到q2队列，假如t2运气比较好，被随机到了，然后t2进入了q2队列，如图4，进入q2的队列的锁才有资格争抢obj的锁，t4线程执行完毕之后，会释放obj的锁，此时队列q2中的t2会获取到obj的锁，然后继续执行，执行完毕之后，q1中包含t1、t3，q2队列为空，如图5

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173928921-719249626.png)

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173935887-257977950.png)

**接着又来了个t5队列，执行了notifyAll()方法，如下：**

```scss
synchronize(obj){
	obj.notifyAll();
}
```

2.调用obj.wait()方法，当前线程会加入队列queue1，然后会释放obj对象的锁

t5会获取到obj的锁，然后执行notifyAll()方法，系统会将队列q1中的线程都移到q2中，如图6，t5线程执行完毕之后，会释放obj的锁，此时队列q2中的t1、t3会争抢obj的锁，争抢到的继续执行，未增强到的带锁释放之后，系统会通知q2中的线程继续争抢索，然后继续执行，最后两个队列中都为空了。

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173946785-477399765.png)

### 挂起（suspend）和继续执行（resume）线程

Thread类中还有2个方法，即**线程挂起(suspend)**和**继续执行(resume)**，这2个操作是一对相反的操作，被挂起的线程，必须要等到resume()方法操作后，才能继续执行。系统中已经标注着2个方法过时了，不推荐使用。

系统不推荐使用suspend()方法去挂起线程是因为suspend()方法导致线程暂停的同时，并不会释放任何锁资源。此时，其他任何线程想要访问被它占用的锁时，都会被牵连，导致无法正常运行（如图2.7所示）。直到在对应的线程上进行了resume()方法操作，被挂起的线程才能继续，从而其他所有阻塞在相关锁上的线程也可以继续执行。但是，如果resume()方法操作意外地在suspend()方法前就被执行了，那么被挂起的线程可能很难有机会被继续执行了。并且，更严重的是：它所占用的锁不会被释放，因此可能会导致整个系统工作不正常。而且，对于被挂起的线程，从它线程的状态上看，居然还是**Runnable**状态，这也会影响我们队系统当前状态的判断。

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715173958232-90412485.png)

上个例子：

```java

public class SuspendThread {
    static Object object = new Object();

    public static class T1 extends Thread {
        public T1(String name) {
            super(name);
        }

        @Override
        public void run() {
            synchronized (object) {
                System.out.println("in " + this.getName());
                Thread.currentThread().suspend();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T1 t1 = new T1("t1");
        t1.start();
        Thread.sleep(100);
        T1 t2 = new T1("t2");
        t2.start();
        t1.resume();
        t2.resume();
        t1.join();
        t2.join();
    }
}
```

运行代码输出：

```txt
in t1
in t2
```

我们会发现程序不会结束，线程t2被挂起了，导致程序无法结束，使用jstack命令查看线程堆栈信息可以看到：

```txt
"t2" #13 prio=5 os_prio=0 tid=0x000000002796c000 nid=0xa3c runnable [0x000000002867f000]
   java.lang.Thread.State: RUNNABLE
        at java.lang.Thread.suspend0(Native Method)
        at java.lang.Thread.suspend(Thread.java:1029)
        at com.concurrency.SuspendThread$T1.run(SuspendThread.java:20)
        - locked <0x0000000717372fc0> (a java.lang.Object)
```

发现t2线程在**suspend0**处被挂起了，t2的状态竟然还是RUNNABLE状态，线程明明被挂起了，状态还是运行中容易导致我们队当前系统进行误判，代码中已经调用resume()方法了，但是由于时间先后顺序的缘故，resume并没有生效，这导致了t2永远滴被挂起了，并且永远占用了object的锁，这对于系统来说可能是致命的。

### 等待线程结束（join）和谦让（yeild）

很多时候，一个线程的输入可能非常依赖于另外一个或者多个线程的输出，此时，这个线程就需要等待依赖的线程执行完毕，才能继续执行。jdk提供了join()操作来实现这个功能。如下所示，显示了2个join()方法：

```java
public final void join() throws InterruptedException;
public final synchronized void join(long millis) throws InterruptedException;
```

第1个方法表示无限等待，它会一直只是当前线程。知道目标线程执行完毕。

第2个方法有个参数，用于指定等待时间，如果超过了给定的时间目标线程还在执行，当前线程也会停止等待，而继续往下执行。

比如：线程T1需要等待T2、T3完成之后才能继续执行，那么在T1线程中需要分别调用T2和T3的join()方法。

上个示例：

```java

public class JoinThread {
    static int num = 0;

    public static class T1 extends Thread {
        public T1(String name) {
            super(name);
        }

        @Override
        public void run() {
            System.out.println(System.currentTimeMillis() + ",start " + this.getName());
            for (int i = 0; i < 10; i++) {
                num++;
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(System.currentTimeMillis() + ",end " + this.getName());
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T1 t1 = new T1("t1");
        t1.start();
        t1.join();
        System.out.println(System.currentTimeMillis() + ",num = " + num);
    }
}
```

执行结果：

```txt
1562939889129,start t1
1562939891134,end t1
1562939891134,num = 10
```

num的结果为10，1、3行的时间戳相差2秒左右，说明主线程等待t1完成之后才继续执行的。

看一下jdk1.8中Thread.join()方法的实现：

```java
public final synchronized void join(long millis) throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

从join的代码中可以看出，在被等待的线程上使用了synchronize，调用了它的wait()方法，线程最后执行完毕之后，**系统会自动调用它的notifyAll()方法**，唤醒所有在此线程上等待的其他线程。

**注意：被等待的线程执行完毕之后，系统自动会调用该线程的notifyAll()方法。所以一般情况下，我们不要去在线程对象上使用wait()、notify()、notifyAll()方法。**

另外一个方法是**Thread.yield()**，他的定义如下：

```java
public static native void yield();
```

yield是谦让的意思，这是一个静态方法，一旦执行，它会让当前线程出让CPU，但需要注意的是，出让CPU并不是说不让当前线程执行了，当前线程在出让CPU后，还会进行CPU资源的争夺，但是能否再抢到CPU的执行权就不一定了。因此，对Thread.yield()方法的调用好像就是在说：我已经完成了一些主要的工作，我可以休息一下了，可以让CPU给其他线程一些工作机会了。

如果觉得一个线程不太重要，或者优先级比较低，而又担心此线程会过多的占用CPU资源，那么可以在适当的时候调用一下Thread.yield()方法，给与其他线程更多的机会。

### 总结

1. 创建线程的方式
2. 启动线程：调用线程的start()方法
3. 终止线程：调用线程的stop()方法，方法已过时，建议不要使用
4. 线程中断相关的方法：调用线程**实例interrupt()方法**将中断标志置为true；使用**线程实例方法isInterrupted()**获取中断标志；调用**Thread的静态方法interrupted()**获取线程是否被中断，此方法调用之后会清除中断标志（将中断标志置为false了）
5. wait、notify、notifyAll方法，这块比较难理解，可以回过头去再理理
6. 线程挂起使用**线程实例方法suspend()**，恢复线程使用**线程实例方法resume()**，这2个方法都过时了，不建议使用
7. 等待线程结束：调用**线程实例方法join()**
8. 出让cpu资源：调用**线程静态方法yeild()**

## **JMM-Java内存模型**

JMM(java内存模型)，由于并发程序要比串行程序复杂很多，其中一个重要原因是并发程序中数据访问**一致性**和**安全性**将会受到严重挑战。**如何保证一个线程可以看到正确的数据呢？**这个问题看起来很白痴。对于串行程序来说，根本就是小菜一碟，如果你读取一个变量，这个变量的值是1，那么你读取到的一定是1，就是这么简单的问题在并行程序中居然变得复杂起来。事实上，如果不加控制地任由线程胡乱并行，即使原本是1的数值，你也可能读到2。因此我们需要在深入了解并行机制的前提下，再定义一种规则，保证多个线程间可以有小弟，正确地协同工作。而JMM也就是为此而生的。

JMM关键技术点都是围绕着多线程的原子性、可见性、有序性来建立的。我们需要先了解这些概念。

### 原子性

原子性是指**操作是不可分的**，要么全部一起执行，要么不执行。在java中，其表现在对于共享变量的某些操作，是不可分的，必须连续的完成。比如a++，对于共享变量a的操作，实际上会执行3个步骤：

1.读取变量a的值，假如a=1
2.a的值+1，为2
3.将2值赋值给变量a，此时a的值应该为2

这三个操作中任意一个操作，a的值如果被其他线程篡改了，那么都会出现我们不希望出现的结果。所以必须保证这3个操作是原子性的，在操作a++的过程中，其他线程不会改变a的值，如果在上面的过程中出现其他线程修改了a的值，在满足原子性的原则下，上面的操作应该失败。

java中实现原子操作的方法大致有2种：**锁机制**、**无锁CAS机制**，后面的章节中会有介绍。

### 可见性

**可见性是指一个线程对共享变量的修改，对于另一个线程来说是否是可以看到的。**有些同学会说修改同一个变量，那肯定是可以看到的，难道线程眼盲了？

举个简单的例子，看下面这段代码：

```
1 //线程1执行的代码
2 int i = 0;
3 i = 10;
4  
5 //线程2执行的代码
6 j = i;
```

假若执行线程1的是CPU1，执行线程2的是CPU2。由上面的分析可知，当线程1执行 i =10这句时，会先把i的初始值加载到CPU1的高速缓存中，然后赋值为10，那么在CPU1的高速缓存当中i的值变为10了，却没有立即写入到主存当中。

此时线程2执行 j = i，它会先去主存读取i的值并加载到CPU2的缓存当中，注意此时内存当中i的值还是0，那么就会使得j的值为0，而不是10。

这就是可见性问题，线程1对变量i修改了之后，线程2没有立即看到线程1修改的值。

**为什么会出现这种问题呢？**

Java虚拟机有自己的内存模型（Java Memory Model，JMM），JMM可以屏蔽掉各种硬件和操作系统的内存访问差异，以实现让java程序在各种平台下都能达到一致的内存访问效果。

　　JMM决定一个线程对共享变量的写入何时对另一个线程可见，JMM定义了线程和主内存之间的抽象关系：共享变量存储在主内存(Main Memory)中，每个线程都有一个私有的本地内存（Local Memory），本地内存保存了被该线程使用到的主内存的副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存中的变量。

看一下java线程内存模型：

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715172330000-1460123585.png)

- 我们定义的所有变量都储存在`主内存`中
- 每个线程都有自己`独立的工作内存`，里面保存该线程使用到的变量的副本（主内存中该变量的一份拷贝）
- 线程对共享变量所有的操作都必须在自己的工作内存中进行，不能直接从主内存中读写（不能越级）
- 不同线程之间也无法直接访问其他线程的工作内存中的变量，线程间变量值的传递需要通过主内存来进行。（同级不能相互访问）

线程需要修改一个共享变量X，需要先把X从主内存复制一份到线程的工作内存，在自己的工作内存中修改完毕之后，再从工作内存中回写到主内存。
如果线程对变量的操作没有刷写回主内存的话，仅仅改变了自己的工作内存的变量的副本，那么对于其他线程来说是不可见的。
而如果另一个变量没有读取主内存中的新的值，而是使用旧的值的话，同样的也可以列为不可见。

**共享变量可见性的实现原理：**

线程A对共享变量的修改要被线程B及时看到的话，需要进过以下步骤：

1.线程A在自己的工作内存中修改变量之后，需要将变量的值刷新到主内存中
2.线程B要把主内存中变量的值更新到工作内存中

关于线程可见性的控制，可以使用**volatile**、**synchronized**、**锁**来实现，后面章节会有详细介绍。

计算机在执行程序时，每条指令都是在CPU中执行的，而执行指令过程中，势必涉及到数据的读取和写入。由于程序运行过程中的临时数据是存放在主存（物理内存）当中的，这时就存在一个问题，由于CPU执行速度很快，而从内存读取数据和向内存写入数据的过程跟CPU执行指令的速度比起来要慢的多，因此如果任何时候对数据的操作都要通过和内存的交互来进行，会大大降低指令执行的速度。因此在CPU里面就有了高速缓存。

　　也就是，当程序在运行过程中，会将运算需要的数据从主存复制一份到CPU的高速缓存当中，那么CPU进行计算时就可以直接从它的高速缓存读取数据和向其中写入数据，当运算结束之后，再将高速缓存中的数据刷新到主存当中。举个简单的例子，比如下面的这段代码：

```
i = i + 1;
```

当线程执行这个语句时，会先从主存当中读取i的值，然后复制一份到高速缓存当中，然后CPU执行指令对i进行加1操作，然后将数据写入高速缓存，最后将高速缓存中i最新的值刷新到主存当中。

　　这个代码在单线程中运行是没有任何问题的，但是在多线程中运行就会有问题了。在多核CPU中，每条线程可能运行于不同的CPU中，因此每个线程运行时有自己的高速缓存（对单核CPU来说，其实也会出现这种问题，只不过是以线程调度的形式来分别执行的）。本文我们以多核CPU为例。

　　比如同时有2个线程执行这段代码，假如初始时i的值为0，那么我们希望两个线程执行完之后i的值变为2。但是事实会是这样吗？

　　可能存在下面一种情况：初始时，两个线程分别读取i的值存入各自所在的CPU的高速缓存当中，然后线程1进行加1操作，然后把i的最新值1写入到内存。此时线程2的高速缓存当中i的值还是0，进行加1操作之后，i的值为1，然后线程2把i的值写入内存。

　　最终结果i的值是1，而不是2。这就是著名的缓存一致性问题。通常称这种被多个线程访问的变量为共享变量。

### 有序性

有序性指的是程序按照代码的先后顺序执行。

为了性能优化，编译器和处理器会进行指令冲排序，有时候会改变程序语句的先后顺序，比如程序。

```java
int a = 1;  //1
int b = 20; //2
int c = a + b; //3
```

编译器优化后可能变成

```java
int b = 20;  //1
int a = 1; //2
int c = a + b; //3
```

上面这个例子中，编译器调整了语句的顺序，但是不影响程序的最终结果。

**但是重排序也需要遵守一定规则：**

　　**1.重排序操作不会对存在数据依赖关系的操作进行重排序。**

　　　　比如：a=1;b=a; 这个指令序列，由于第二个操作依赖于第一个操作，所以在编译时和处理器运行时这两个操作不会被重排序。

　　**2.重排序是为了优化性能，但是不管怎么重排序，单线程下程序的执行结果不能被改变**

　　　　比如：a=1;b=2;c=a+b这三个操作，第一步（a=1)和第二步(b=2)由于不存在数据依赖关系，所以可能会发生重排序，但是c=a+b这个操作是不会被重排序的，因为需要保证最终的结果一定是c=a+b=3。

在单例模式的实现上有一种双重检验锁定的方式，代码如下：

```java
public class Singleton {
  static Singleton instance;
  static Singleton getInstance(){
    if (instance == null) {
      synchronized(Singleton.class) {
        if (instance == null)
          instance = new Singleton();
        }
    }
    return instance;
  }
}
```

我们先看`instance = new Singleton();`

**未被编译器优化的操作：**

1. 指令1：分配一款内存M
2. 指令2：在内存M上初始化Singleton对象
3. 指令3：将M的地址赋值给instance变量

**编译器优化后的操作指令：**

1. 指令1：分配一块内存S
2. 指令2：将M的地址赋值给instance变量
3. 指令3：在内存M上初始化Singleton对象

现在有2个线程，刚好执行的代码被编译器优化过，过程如下：

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715172343814-2043850393.png)

最终线程B获取的instance是没有初始化的，此时去使用instance可能会产生一些意想不到的错误。

现在比较好的做法就是采用静态内部类的方式实现：

```java
public class SingletonDemo {
    private SingletonDemo() {
    }
    private static class SingletonDemoHandler{
        private static SingletonDemo instance = new SingletonDemo();
    }
    public static SingletonDemo getInstance() {
        return SingletonDemoHandler.instance;
    }
}
```

## volatile关键字

　 volatile是Java提供的一种轻量级的同步机制。同synchronized相比（synchronized通常称为重量级锁），volatile更轻量级。

　　一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

　　**1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。**

　　**2）禁止进行指令重排序。**

### **1、共享变量的可见性**

```
public class TestVolatile {
    
    public static void main(String[] args) {
        ThreadDemo td = new ThreadDemo();
        new Thread(td).start();
        while(true){
            if(td.isFlag()){
                System.out.println("------------------");
                break;
            }
        }
    }

}

class ThreadDemo implements Runnable {
    private  boolean flag = false;
    @Override
    public void run() {
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
        }
        flag = true;
        System.out.println("flag=" + isFlag());
    }

    public boolean isFlag() {
        return flag;
    }
}
```

上面这个例子，开启一个多线程去改变flag为true，main 主线程中可以输出"------------------"吗？

　　**答案是NO!** 

　　这个结论会让人有些疑惑，可以理解。开启的线程虽然修改了flag 的值为true,但是还没来得及写入主存当中，此时main里面的 td.isFlag()还是false,但是由于 while(true) 是底层的指令来实现，速度非常之快，一直循环都没有时间去主存中更新td的值，所以这里会造成死循环！运行结果如下：

![img](https://img2018.cnblogs.com/blog/1168971/201811/1168971-20181116112911017-1818746206.png)

此时线程是没有停止的，一直在循环。

如何解决呢？只需将 flag 声明为volatile，即可保证在开启的线程A将其修改为true时，main主线程可以立刻得知：

　　第一：使用volatile关键字会强制将修改的值立即写入主存；

　　第二：使用volatile关键字的话，当开启的线程进行修改时，会导致main线程的工作内存中缓存变量flag的缓存行无效（反映到硬件层的话，就是CPU的L1缓存中对应的缓存行无效）；

　　第三：由于线程main的工作内存中缓存变量flag的缓存行无效，所以线程main再次读取变量flag的值时会去主存读取。

volatile具备两种特性，第一就是保证共享变量对所有线程的可见性。将一个共享变量声明为volatile后，会有以下效应：

　　**1.当写一个volatile变量时，JMM会把该线程对应的本地内存中的变量强制刷新到主内存中去；**

　　**2.这个写会操作会导致其他线程中的缓存无效。**



### 2、**禁止进行指令重排序**

**这里我们引用上篇文章单例里面的例子**

```
 1 class Singleton{
 2     private volatile static Singleton instance = null;
 3 
 4     private Singleton() {
 5     }
 6      
 7     public static Singleton getInstance() {
 8         if(instance==null) {
 9             synchronized (Singleton.class) {
10                 if(instance==null)
11                     instance = new Singleton();
12             }
13         }
14         return instance;
15     }
16 }
```

instance = new Singleton(); 这段代码可以分为三个步骤：
1、memory = allocate() 分配对象的内存空间
2、ctorInstance() 初始化对象
3、instance = memory 设置instance指向刚分配的内存

***但是此时有可能发生指令重排，CPU 的执行顺序可能为：***

1、memory = allocate() 分配对象的内存空间
3、instance = memory 设置instance指向刚分配的内存
2、ctorInstance() 初始化对象

在单线程的情况下，1->3->2这种顺序执行是没有问题的，但是如果是多线程的情况则有可能出现问题，线程A执行到11行代码，执行了指令1和3，此时instance已经有值了，值为第一步分配的内存空间地址，但是还没有进行对象的初始化；

此时线程B执行到了第8行代码处，此时instance已经有值了则return instance，线程B 使用instance的时候，就会出现异常。

这里可以**使用 volatile 来禁止指令重排序。**

 

**从上面知道volatile关键字保证了操作的可见性和有序性，但是volatile能保证对变量的操作是原子性吗？**

下面看一个例子：

```
package com.mmall.concurrency.example.count;
import java.util.concurrent.CountDownLatch;

/**
 * @author: ChenHao
 * @Description:
 * @Date: Created in 15:05 2018/11/16
 * @Modified by:
 */
public class CountTest {
    // 请求总数
    public static int clientTotal = 5000;
    public static volatile int count = 0;

    public static void main(String[] args) throws Exception {
        //使用CountDownLatch来等待计算线程执行完
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        //开启clientTotal个线程进行累加操作
        for(int i=0;i<clientTotal;i++){
            new Thread(){
                public void run(){
                    count++;//自加操作
                    countDownLatch.countDown();
                }
            }.start();
        }
        //等待计算线程执行完
        countDownLatch.await();
        System.out.println(count);
    }
}
```

执行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201811/1168971-20181116150856988-873369249.png)

针对这个示例，一些同学可能会觉得疑惑，如果用volatile修饰的共享变量可以保证可见性，那么结果不应该是5000么?

问题就出在count++这个操作上，**因为count++不是个原子性的操作，而是个复合操作**。我们可以简单讲这个操作理解为由这三步组成:

　　1.读取count

　　2.count 加 1

　　3.将count 写到主存

　　**所以，在多线程环境下，有可能线程A将count读取到本地内存中，此时其他线程可能已经将count增大了很多，线程A依然对过期的本地缓存count进行自加，重新写到主存中，最终导致了count的结果不合预期，而是小于5000。**

那么如何来解决这个问题呢？下面我们来看看

## 线程组 

### 线程组

我们可以把线程归属到某个线程组中，线程组可以包含多个**线程**以及**线程组**，线程和线程组组成了父子关系，是个树形结构，如下图：

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715174510093-1713299512.png)

使用线程组可以方便管理线程，线程组提供了一些方法方便方便我们管理线程。

### 创建线程关联线程组

创建线程的时候，可以给线程指定一个线程组，代码如下：

```java
package com.concurrent;

import java.util.concurrent.TimeUnit;


public class ThreadGroup {
    public static class R1 implements Runnable {
        @Override
        public void run() {
            System.out.println("threadName:" + Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ThreadGroup threadGroup = new ThreadGroup("thread-group-1");
        Thread t1 = new Thread(threadGroup, new R1(), "t1");
        Thread t2 = new Thread(threadGroup, new R1(), "t2");
        t1.start();
        t2.start();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("活动线程数:" + threadGroup.activeCount());
        System.out.println("活动线程组:" + threadGroup.activeGroupCount());
        System.out.println("线程组名称:" + threadGroup.getName());
    }
}
```

输出结果：

```txt
threadName:t1
threadName:t2
活动线程数:2
活动线程组:0
线程组名称:thread-group-1
```

**activeCount()**方法可以返回线程组中的所有活动线程数，包含下面的所有子孙节点的线程，由于线程组中的线程是动态变化的，这个值只能是一个估算值。

### 为线程组指定父线程组

创建线程组的时候，可以给其指定一个父线程组，也可以不指定，如果不指定父线程组，则父线程组为当前线程的线程组，java api有2个常用的构造方法用来创建线程组：

```java
public ThreadGroup(String name)
public ThreadGroup(ThreadGroup parent, String name)
```

第一个构造方法未指定父线程组，看一下内部的实现：

```java
public ThreadGroup(String name) {
        this(Thread.currentThread().getThreadGroup(), name);
    }
```

系统自动获取当前线程的线程组作为默认父线程组。

上一段示例代码：

```java
package com.itsoku.chat02;

import java.util.concurrent.TimeUnit;

/**
 * <b>description</b>： <br>
 * <b>time</b>：2019/7/13 17:53 <br>
 * <b>author</b>：微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo2 {
    public static class R1 implements Runnable {
        @Override
        public void run() {
            Thread thread = Thread.currentThread();
            System.out.println("所属线程组:" + thread.getThreadGroup().getName() + ",线程名称:" + thread.getName());
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ThreadGroup threadGroup1 = new ThreadGroup("thread-group-1");
        Thread t1 = new Thread(threadGroup1, new R1(), "t1");
        Thread t2 = new Thread(threadGroup1, new R1(), "t2");
        t1.start();
        t2.start();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("threadGroup1活动线程数:" + threadGroup1.activeCount());
        System.out.println("threadGroup1活动线程组:" + threadGroup1.activeGroupCount());
        System.out.println("threadGroup1线程组名称:" + threadGroup1.getName());
        System.out.println("threadGroup1父线程组名称:" + threadGroup1.getParent().getName());
        System.out.println("----------------------");
        ThreadGroup threadGroup2 = new ThreadGroup(threadGroup1, "thread-group-2");
        Thread t3 = new Thread(threadGroup2, new R1(), "t3");
        Thread t4 = new Thread(threadGroup2, new R1(), "t4");
        t3.start();
        t4.start();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("threadGroup2活动线程数:" + threadGroup2.activeCount());
        System.out.println("threadGroup2活动线程组:" + threadGroup2.activeGroupCount());
        System.out.println("threadGroup2线程组名称:" + threadGroup2.getName());
        System.out.println("threadGroup2父线程组名称:" + threadGroup2.getParent().getName());

        System.out.println("----------------------");
        System.out.println("threadGroup1活动线程数:" + threadGroup1.activeCount());
        System.out.println("threadGroup1活动线程组:" + threadGroup1.activeGroupCount());

        System.out.println("----------------------");
        threadGroup1.list();
    }
}
```

输出结果：

```txt
所属线程组:thread-group-1,线程名称:t1
所属线程组:thread-group-1,线程名称:t2
threadGroup1活动线程数:2
threadGroup1活动线程组:0
threadGroup1线程组名称:thread-group-1
threadGroup1父线程组名称:main
----------------------
所属线程组:thread-group-2,线程名称:t4
所属线程组:thread-group-2,线程名称:t3
threadGroup2活动线程数:2
threadGroup2活动线程组:0
threadGroup2线程组名称:thread-group-2
threadGroup2父线程组名称:thread-group-1
----------------------
threadGroup1活动线程数:4
threadGroup1活动线程组:1
----------------------
java.lang.ThreadGroup[name=thread-group-1,maxpri=10]
    Thread[t1,5,thread-group-1]
    Thread[t2,5,thread-group-1]
    java.lang.ThreadGroup[name=thread-group-2,maxpri=10]
        Thread[t3,5,thread-group-2]
        Thread[t4,5,thread-group-2]
```

代码解释：

1. **threadGroup1未指定父线程组，系统获取了主线程的线程组作为threadGroup1的父线程组，输出结果中是：main**
2. **threadGroup1为threadGroup2的父线程组**
3. **threadGroup1活动线程数为4，包含了threadGroup1线程组中的t1、t2，以及子线程组threadGroup2中的t3、t4**
4. **线程组的list()方法，将线程组中的所有子孙节点信息输出到控制台，用于调试使用**

### 根线程组

获取根线程组

```java
package com.itsoku.chat02;

/**
 * <b>description</b>： <br>
 * <b>time</b>：2019/7/13 17:53 <br>
 * <b>author</b>：微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo3 {

    public static void main(String[] args) {
        System.out.println(Thread.currentThread());
        System.out.println(Thread.currentThread().getThreadGroup());
        System.out.println(Thread.currentThread().getThreadGroup().getParent());
        System.out.println(Thread.currentThread().getThreadGroup().getParent().getParent());
    }
}
```

运行上面代码，输出：

```java
Thread[main,5,main]
java.lang.ThreadGroup[name=main,maxpri=10]
java.lang.ThreadGroup[name=system,maxpri=10]
null
```

从上面代码可以看出：

1. **主线程的线程组为main**
2. **根线程组为system**

看一下ThreadGroup的源码：

```java
private ThreadGroup() {     // called from C code
        this.name = "system";
        this.maxPriority = Thread.MAX_PRIORITY;
        this.parent = null;
    }
```

发现ThreadGroup默认构造方法是private的，是由c调用的，创建的正是system线程组。

### 批量停止线程

调用线程组**interrupt()**，会将线程组树下的所有子孙线程中断标志置为true，可以用来批量中断线程。

示例代码：

```java
package com.itsoku.chat02;

import java.util.concurrent.TimeUnit;

/**
 * <b>description</b>： <br>
 * <b>time</b>：2019/7/13 17:53 <br>
 * <b>author</b>：微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo4 {
    public static class R1 implements Runnable {
        @Override
        public void run() {
            Thread thread = Thread.currentThread();
            System.out.println("所属线程组:" + thread.getThreadGroup().getName() + ",线程名称:" + thread.getName());
            while (!thread.isInterrupted()) {
                ;
            }
            System.out.println("线程:" + thread.getName() + "停止了！");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ThreadGroup threadGroup1 = new ThreadGroup("thread-group-1");
        Thread t1 = new Thread(threadGroup1, new R1(), "t1");
        Thread t2 = new Thread(threadGroup1, new R1(), "t2");
        t1.start();
        t2.start();

        ThreadGroup threadGroup2 = new ThreadGroup(threadGroup1, "thread-group-2");
        Thread t3 = new Thread(threadGroup2, new R1(), "t3");
        Thread t4 = new Thread(threadGroup2, new R1(), "t4");
        t3.start();
        t4.start();
        TimeUnit.SECONDS.sleep(1);

        System.out.println("-----------threadGroup1信息-----------");
        threadGroup1.list();

        System.out.println("----------------------");
        System.out.println("停止线程组：" + threadGroup1.getName() + "中的所有子孙线程");
        threadGroup1.interrupt();
        TimeUnit.SECONDS.sleep(2);

        System.out.println("----------threadGroup1停止后，输出信息------------");
        threadGroup1.list();
    }
}
```

输出：

```txt
所属线程组:thread-group-1,线程名称:t1
所属线程组:thread-group-1,线程名称:t2
所属线程组:thread-group-2,线程名称:t3
所属线程组:thread-group-2,线程名称:t4
-----------threadGroup1信息-----------
java.lang.ThreadGroup[name=thread-group-1,maxpri=10]
    Thread[t1,5,thread-group-1]
    Thread[t2,5,thread-group-1]
    java.lang.ThreadGroup[name=thread-group-2,maxpri=10]
        Thread[t3,5,thread-group-2]
        Thread[t4,5,thread-group-2]
----------------------
停止线程组：thread-group-1中的所有子孙线程
线程:t4停止了！
线程:t2停止了！
线程:t1停止了！
线程:t3停止了！
----------threadGroup1停止后，输出信息------------
java.lang.ThreadGroup[name=thread-group-1,maxpri=10]
    java.lang.ThreadGroup[name=thread-group-2,maxpri=10]
```

停止线程之后，通过**list()**方法可以看出输出的信息中不包含已结束的线程了。

多说几句，建议大家再创建线程或者线程组的时候，给他们取一个有意义的名字，对于计算机来说，可能名字并不重要，但是在系统出问题的时候，你可能会去查看线程堆栈信息，如果你看到的都是t1、t2、t3，估计自己也比较崩溃，如果看到的是httpAccpHandler、dubboHandler类似的名字，应该会好很多。

## 用户线程和守护线程

**守护线程**是一种特殊的线程，在后台默默地完成一些系统性的服务，比如**垃圾回收线程**、**JIT线程**都是**守护线程**。与之对应的是**用户线程**，用户线程可以理解为是系统的工作线程，它会完成这个程序需要完成的业务操作。如果用户线程全部结束了，意味着程序需要完成的业务操作已经结束了，系统可以退出了。**所以当系统只剩下守护进程的时候，java虚拟机会自动退出**。

java线程分为用户线程和守护线程，线程的daemon属性为true表示是守护线程，false表示是用户线程。

下面我们来看一下守护线程的一些特性。

### 程序只有守护线程时，系统会自动退出

```java
package com.itsoku.chat03;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo1 {

    public static class T1 extends Thread {
        public T1(String name) {
            super(name);
        }

        @Override
        public void run() {
            System.out.println(this.getName() + "开始执行," + (this.isDaemon() ? "我是守护线程" : "我是用户线程"));
            while (true) ;
        }
    }

    public static void main(String[] args) {
        T1 t1 = new T1("子线程1");
        t1.start();
        System.out.println("主线程结束");
    }
}
```

运行上面代码，结果如下：

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715174714905-1252155469.png)

可以看到主线程已经结束了，但是程序无法退出，原因：子线程1是用户线程，内部有个死循环，一直处于运行状态，无法结束。

再看下面的代码：

```java
package com.itsoku.chat03;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo2 {

    public static class T1 extends Thread {
        public T1(String name) {
            super(name);
        }

        @Override
        public void run() {
            System.out.println(this.getName() + "开始执行," + (this.isDaemon() ? "我是守护线程" : "我是用户线程"));
            while (true) ;
        }
    }

    public static void main(String[] args) {
        T1 t1 = new T1("子线程1");
        t1.setDaemon(true);
        t1.start();
        System.out.println("主线程结束");
    }
}
```

运行结果：

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715174726701-380700292.png)

程序可以正常结束了，代码中通过`t1.setDaemon(true);`将t1线程设置为守护线程，main方法所在的主线程执行完毕之后，程序就退出了。

**结论：当程序中所有的用户线程执行完毕之后，不管守护线程是否结束，系统都会自动退出。**

### 设置守护线程，需要在start()方法之前进行

```java
package com.itsoku.chat03;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo3 {

    public static void main(String[] args) {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        t1.start();
        t1.setDaemon(true);
    }
}
```

`t1.setDaemon(true);`是在t1的start()方法之后执行的，执行会报异常，运行结果如下：

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190715174738847-183628539.png)

### 线程daemon的默认值

我们看一下创建线程源码，位于**Thread类的init()**方法中：

```java
Thread parent = currentThread();
this.daemon = parent.isDaemon();
```

dameon的默认值为为父线程的daemon，也就是说，父线程如果为用户线程，子线程默认也是用户现场，父线程如果是守护线程，子线程默认也是守护线程。

示例代码：

```java
package com.itsoku.chat03;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo4 {
    public static class T1 extends Thread {
        public T1(String name) {
            super(name);
        }

        @Override
        public void run() {
            System.out.println(this.getName() + ".daemon:" + this.isDaemon());
        }
    }

    public static void main(String[] args) throws InterruptedException {

        System.out.println(Thread.currentThread().getName() + ".daemon:" + Thread.currentThread().isDaemon());

        T1 t1 = new T1("t1");
        t1.start();

        Thread t2 = new Thread() {
            @Override
            public void run() {
                System.out.println(this.getName() + ".daemon:" + this.isDaemon());
                T1 t3 = new T1("t3");
                t3.start();
            }
        };

        t2.setName("t2");
        t2.setDaemon(true);
        t2.start();

        TimeUnit.SECONDS.sleep(2);
    }
}
```

运行代码，输出：

```txt
main.daemon:false
t1.daemon:false
t2.daemon:true
t3.daemon:true
```

t1是由主线程(main方法所在的线程)创建的，main线程是t1的父线程，所以t1.daemon为false，说明t1是用户线程。

t2线程调用了`setDaemon(true);`将其设为守护线程，t3是由t2创建的，所以t3默认线程类型和t2一样，t2.daemon为true。

### 总结

1. java中的线程分为**用户线程**和**守护线程**
2. 程序中的所有的用户线程结束之后，不管守护线程处于什么状态，java虚拟机都会自动退出
3. 调用线程的实例方法setDaemon()来设置线程是否是守护线程
4. setDaemon()方法必须在线程的start()方法之前调用，在后面调用会报异常，并且不起效
5. 线程的daemon默认值和其父线程一样

## 线程安全和synchronized关键字

### 什么是线程安全？

如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。

或者说当多个线程去访问同一个类（对象或方法）的时候，该类都能表现出正常的行为（与自己预想的结果一致），那我们就可以所这个类是线程安全的。

线程安全问题都是由全局变量及静态变量引起的。
 若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作，一般都需要考虑线程同步，否则就可能影响线程安全。

看一段代码：

```java
package com.itsoku.chat04;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo1 {
    static int num = 0;

    public static void m1() {
        for (int i = 0; i < 10000; i++) {
            num++;
        }
    }

    public static class T1 extends Thread {
        @Override
        public void run() {
            Demo1.m1();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T1 t1 = new T1();
        T1 t2 = new T1();
        T1 t3 = new T1();
        t1.start();
        t2.start();
        t3.start();

        //等待3个线程结束打印num
        t1.join();
        t2.join();
        t3.join();

        System.out.println(Demo1.num);
        /**
         * 打印结果：
         * 25572
         */
    }
}
```

Demo1中有个静态变量num，默认值是0，m1()方法中对num++执行10000次，main方法中创建了3个线程用来调用m1()方法，然后调用3个线程的join()方法，用来等待3个线程执行完毕之后，打印num的值。我们期望的结果是30000，运行一下，但真实的结果却不是30000。上面的程序在多线程中表现出来的结果和预想的结果不一致，说明上面的程序不是线程安全的。

线程安全是并发编程中的重要关注点，应该注意到的是，造成线程安全问题的主要诱因有两点：

1. 一是存在共享数据(也称临界资源)
2. 二是存在多条线程共同操作共享数据

因此为了解决这个问题，我们可能需要这样一个方案，当存在多个线程操作共享数据时，**需要保证同一时刻有且只有一个线程在操作共享数据**，其他线程必须等到该线程处理完数据后再进行，这种方式有个高尚的名称叫**互斥锁**，即能达到互斥访问目的的锁，也就是说当一个共享数据被当前正在访问的线程加上互斥锁后，在同一个时刻，其他线程只能处于等待的状态，直到当前线程处理完毕释放该锁。在 Java 中，**关键字 synchronized可以保证在同一个时刻，只有一个线程可以执行某个方法或者某个代码块(主要是对方法或者代码块中存在共享数据的操作)**，**同时我们还应该注意到synchronized另外一个重要的作用，synchronized可保证一个线程的变化(主要是共享数据的变化)被其他线程所看到（保证可见性，完全可以替代volatile功能）**，这点确实也是很重要的。

那么我们把上面的程序做一下调整，在m1()方法上面使用关键字synchronized，如下：

```java
public static synchronized void m1() {
    for (int i = 0; i < 10000; i++) {
        num++;
    }
}
```

然后执行代码，输出30000，和期望结果一致。

### synchronized使用方式

1. 修饰实例方法，作用于当前实例，进入同步代码前需要先获取实例的锁
2. 修饰静态方法，作用于类的Class对象，进入修饰的静态方法前需要先获取类的Class对象的锁
3. 修饰代码块，需要指定加锁对象(记做lockobj)，在进入同步代码块前需要先获取lockobj的锁

#### synchronized作用于实例对象

所谓实例对象锁就是用synchronized修饰实例对象的实例方法，注意是**实例方法**，不是**静态方法**，如：

```java
package com.itsoku.chat04;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo2 {
    int num = 0;

    public synchronized void add() {
        num++;
    }

    public static class T extends Thread {
        private Demo2 demo2;

        public T(Demo2 demo2) {
            this.demo2 = demo2;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                this.demo2.add();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Demo2 demo2 = new Demo2();
        T t1 = new T(demo2);
        T t2 = new T(demo2);
        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println(demo2.num);
    }
}
```

main()方法中创建了一个对象demo2和2个线程t1、t2，t1、t2中调用demo2的add()方法10000次，add()方法中执行了num++，num++实际上是分3步，获取num，然后将num+1，然后将结果赋值给num，如果t2在t1读取num和num+1之间获取了num的值，那么t1和t2会读取到同样的值，然后执行num++，两次操作之后num是相同的值，最终和期望的结果不一致，造成了线程安全失败，因此我们对add方法加了synchronized来保证线程安全。

注意：m1()方法是实例方法，两个线程操作m1()时，需要先获取demo2的锁，没有获取到锁的，将等待，直到其他线程释放锁为止。

synchronize作用于实例方法需要注意：

1. 实例方法上加synchronized，线程安全的前提是，多个线程操作的是**同一个实例**，如果多个线程作用于不同的实例，那么线程安全是无法保证的
2. 同一个实例的多个实例方法上有synchronized，这些方法都是互斥的，同一时间只允许一个线程操作**同一个实例的其中的一个synchronized方法**

#### synchronized作用于静态方法

当synchronized作用于静态方法时，锁的对象就是当前类的Class对象。如：

```java
package com.itsoku.chat04;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo3 {
    static int num = 0;

    public static synchronized void m1() {
        for (int i = 0; i < 10000; i++) {
            num++;
        }
    }

    public static class T1 extends Thread {
        @Override
        public void run() {
            Demo3.m1();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T1 t1 = new T1();
        T1 t2 = new T1();
        T1 t3 = new T1();
        t1.start();
        t2.start();
        t3.start();

        //等待3个线程结束打印num
        t1.join();
        t2.join();
        t3.join();

        System.out.println(Demo3.num);
        /**
         * 打印结果：
         * 30000
         */
    }
}
```

上面代码打印30000，和期望结果一致。m1()方法是静态方法，有synchronized修饰，锁用于与Demo3.class对象，和下面的写法类似：

```java
public static void m1() {
    synchronized (Demo4.class) {
        for (int i = 0; i < 10000; i++) {
            num++;
        }
    }
}
```

#### synchronized同步代码块

除了使用关键字修饰实例方法和静态方法外，还可以使用同步代码块，在某些情况下，我们编写的方法体可能比较大，同时存在一些比较耗时的操作，而需要同步的代码又只有一小部分，如果直接对整个方法进行同步操作，可能会得不偿失，此时我们可以使用同步代码块的方式对需要同步的代码进行包裹，这样就无需对整个方法进行同步操作了，同步代码块的使用示例如下：

```java
package com.itsoku.chat04;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo5 implements Runnable {
    static Demo5 instance = new Demo5();
    static int i = 0;

    @Override
    public void run() {
        //省略其他耗时操作....
        //使用同步代码块对变量i进行同步操作,锁对象为instance
        synchronized (instance) {
            for (int j = 0; j < 10000; j++) {
                i++;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println(i);
    }
}
```

从代码看出，将synchronized作用于一个给定的实例对象instance，即当前实例对象就是锁对象，每次当线程进入synchronized包裹的代码块时就会要求当前线程持有instance实例对象锁，如果当前有其他线程正持有该对象锁，那么新到的线程就必须等待，这样也就保证了每次只有一个线程执行i++;操作。当然除了instance作为对象外，我们还可以使用this对象(代表当前实例)或者当前类的class对象作为锁，如下代码：

```java
//this,当前实例对象锁
synchronized(this){
    for(int j=0;j<1000000;j++){
        i++;
    }
}

//class对象锁
synchronized(Demo5.class){
    for(int j=0;j<1000000;j++){
        i++;
    }
}
```

分析代码是否互斥的方法，先找出synchronized作用的对象是谁，如果多个线程操作的方法中synchronized作用的锁对象一样，那么这些线程同时异步执行这些方法就是互斥的。如下代码:

```java
package com.itsoku.chat04;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo6 {
    //作用于当前类的实例对象
    public synchronized void m1() {
    }

    //作用于当前类的实例对象
    public synchronized void m2() {
    }

    //作用于当前类的实例对象
    public void m3() {
        synchronized (this) {
        }
    }

    //作用于当前类Class对象
    public static synchronized void m4() {
    }

    //作用于当前类Class对象
    public static void m5() {
        synchronized (Demo6.class) {
        }
    }

    public static class T extends Thread{
        Demo6 demo6;

        public T(Demo6 demo6) {
            this.demo6 = demo6;
        }

        @Override
        public void run() {
            super.run();
        }
    }

    public static void main(String[] args) {
        Demo6 d1 = new Demo6();
        Thread t1 = new Thread(() -> {
            d1.m1();
        });
        t1.start();
        Thread t2 = new Thread(() -> {
            d1.m2();
        });
        t2.start();

        Thread t3 = new Thread(() -> {
            d1.m2();
        });
        t3.start();

        Demo6 d2 = new Demo6();
        Thread t4 = new Thread(() -> {
            d2.m2();
        });
        t4.start();

        Thread t5 = new Thread(() -> {
            Demo6.m4();
        });
        t5.start();

        Thread t6 = new Thread(() -> {
            Demo6.m5();
        });
        t6.start();
    }

}
```

分析上面代码：

1. 线程t1、t2、t3中调用的方法都需要获取d1的锁，所以他们是互斥的
2. t1/t2/t3这3个线程和t4不互斥，他们可以同时运行，因为前面三个线程依赖于d1的锁，t4依赖于d2的锁
3. t5、t6都作用于当前类的Class对象锁，所以这两个线程是互斥的，和其他几个线程不互斥

### synchronized可以确保变量的可见性

synchronized除了用于线程同步、确保线程安全外，还可以保证线程间的可见性和有序性。从可见性的角度上将，关键字synchronized可以完全替代关键字volatile的功能，只是使用上没有那么方便。就有序性而言，由于关键字synchronized限制每次只有一个线程可以访问同步块，因此，无论同步块内的代码如何被乱序执行，只要保证串行语义一致，那么执行结果总是一样的。而其他访问线程，又必须在获得锁后方能进入代码块读取数据，因此，他们看到的最终结果并不取决于代码的执行过程，有序性问题自然得到了解决（换言之，被关键字synchronized限制的多个线程是串行执行的）。

线程进入synchronized修饰的代码中时，synchronized代码块内部使用到的共享变量在当前线程的工作内存中都会被清空，会从主内存中获取，当synchronized代码块结束的时候，代码块内部修改的共享变量都会强制刷新到主存储中，所以是可见的。

关于synchronized可以保证可见性的，上个例子：

```java
package com.itsoku.chat05;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo4 {
    static boolean flag = false;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                System.out.println(this.getName() + " start");
                while (true) {
                    synchronized (this) {
                        if (flag) {
                            break;
                        }
                    }
                }
                System.out.println(this.getName() + " exit");
            }
        };
        t1.setName("t1");
        t1.start();
        Thread t2 = new Thread() {
            @Override
            public void run() {
                System.out.println(this.getName() + " start");
                synchronized (this) {
                    while (true) {
                        if (flag) {
                            break;
                        }
                    }
                }
                System.out.println(this.getName() + " exit");
            }
        };
        t2.setName("t2");
        t2.start();
        TimeUnit.SECONDS.sleep(2);
        flag = true;
    }
}
```

运行结果：
![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190717094128450-1132515455.png)

t1线程可以正常结束，t2线程无法结束，说明主线程中flag修改之后已经被刷新到了主内存了，t1可以看到主内存中中flag最新的值。
t1线程中有个while循环，循环内部有个synchronized块，前面提到过，进入synchronized时，块内部用到的变量在当前线程的工作内存中都会被清空，所以每次进入块中第一次访问flag的时候，都会从主内存中获取，然后复制到工作内存中，所以t1可以正常结束。
t2线程中while循环在synchronized内部，循环内部第一次访问flag的时候会从主内存中获取最新的值，后面再次访问的时候会从工作内存中获取，所以获取到flag一直未false，程序无法结束。

## 线程中断的方式

本文主要探讨一下中断线程的几种方式。

### 通过一个变量控制线程中断

代码：

```java
package com.itsoku.chat05;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo1 {

    public volatile static boolean exit = false;

    public static class T extends Thread {
        @Override
        public void run() {
            while (true) {
                //循环处理业务
                if (exit) {
                    break;
                }
            }
        }
    }

    public static void setExit() {
        exit = true;
    }

    public static void main(String[] args) throws InterruptedException {
        T t = new T();
        t.start();
        TimeUnit.SECONDS.sleep(3);
        setExit();
    }
}
```

代码中启动了一个线程，线程的run方法中有个死循环，内部通过exit变量的值来控制是否退出。`TimeUnit.SECONDS.sleep(3);`让主线程休眠3秒，此处为什么使用TimeUnit？TimeUnit使用更方便一些，能够很清晰的控制休眠时间，底层还是转换为Thread.sleep实现的。程序有个重点：**volatile**关键字，exit变量必须通过这个修饰，如果把这个去掉，程序无法正常退出。volatile控制了变量在多线程中的可见性，关于volatile前面的文章中有介绍，此处就不再说了。

### 通过线程自带的中断标志控制

示例代码：

```java
package com.itsoku.chat05;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo2 {

    public static class T extends Thread {
        @Override
        public void run() {
            while (true) {
                //循环处理业务
                if (this.isInterrupted()) {
                    break;
                }
            }
        }
    }


    public static void main(String[] args) throws InterruptedException {
        T t = new T();
        t.start();
        TimeUnit.SECONDS.sleep(3);
        t.interrupt();
    }
}
```

运行上面的程序，程序可以正常结束。线程内部有个中断标志，当调用线程的interrupt()实例方法之后，线程的中断标志会被置为true，可以通过线程的实例方法isInterrupted()获取线程的中断标志。

### 线程阻塞状态中如何中断

示例代码：

```java
package com.itsoku.chat05;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo3 {

    public static class T extends Thread {
        @Override
        public void run() {
            while (true) {
                //循环处理业务
                //下面模拟阻塞代码
                try {
                    TimeUnit.SECONDS.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }


    public static void main(String[] args) throws InterruptedException {
        T t = new T();
        t.start();
    }
}
```

运行上面代码，发现程序无法结束。

在此先补充几点知识：

1. **调用线程的interrupt()实例方法，线程的中断标志会被置为true**
2. **当线程处于阻塞状态时，调用线程的interrupt()实例方法，线程内部会触发InterruptedException异常，并且会清除线程内部的中断标志（即将中断标志置为false）**

那么上面代码可以调用线程的interrupt()方法来引发InterruptedException异常，来中断sleep方法导致的阻塞，调整一下代码，如下：

```java
package com.itsoku.chat05;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo3 {

    public static class T extends Thread {
        @Override
        public void run() {
            while (true) {
                //循环处理业务
                //下面模拟阻塞代码
                try {
                    TimeUnit.SECONDS.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    this.interrupt();
                }
                if (this.isInterrupted()) {
                    break;
                }
            }
        }
    }


    public static void main(String[] args) throws InterruptedException {
        T t = new T();
        t.start();
        TimeUnit.SECONDS.sleep(3);
        t.interrupt();
    }
}
```

运行结果：

```java
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at java.lang.Thread.sleep(Thread.java:340)
	at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
	at com.itsoku.chat05.Demo3$T.run(Demo3.java:17)
```

程序可以正常结束了，分析一下上面代码，注意几点：

1. main方法中调用了t.interrupt()方法，此时线程t内部的中断标志会置为true
2. 然后会触发run()方法内部的InterruptedException异常，所以运行结果中有异常输出，上面说了，当触发InterruptedException异常时候，线程内部的中断标志又会被清除（变为false），**所以在catch中又调用了this.interrupt();一次**，将中断标志置为false
3. run()方法中通过this.isInterrupted()来获取线程的中断标志，退出循环（break）

### 总结

1. 当一个线程处于被阻塞状态或者试图执行一个阻塞操作时，可以使用`Thread.interrupt()`方式中断该线程，注意此时将会抛出一个**InterruptedException**的异常，同时中断状态将会被复位(由中断状态改为非中断状态)
2. 内部有循环体，可以通过一个变量来作为一个信号控制线程是否中断，注意变量需要volatile修饰
3. 文中的几种方式可以结合起来灵活使用控制线程的中断

## 公平锁和可重入锁的原理

### 原理介绍

最经典的分布式锁是可重入的公平锁。什么是可重入的公平锁呢？直接讲解的概念和原理，会比较抽象难懂，还是从具体的实例入手吧！这里用一个简单的故事来类比，估计就简单多了。

故事发生在一个没有自来水的古代，在一个村子有一口井，水质非常的好，村民们都抢着取井里的水。井就那么一口，村里的人很多，村民为争抢取水打架斗殴，甚至头破血流。

问题总是要解决，于是村长绞尽脑汁，最终想出了一个凭号取水的方案。井边安排一个看井人，维护取水的秩序。取水秩序很简单：

（1）取水之前，先取号；

（2）号排在前面的，就可以先取水；

（3）先到的排在前面，那些后到的，一个一个挨着，在井边排成一队。

这种排队取水模型，就是一种锁的模型。排在最前面的号，拥有取水权，就是一种典型的独占锁。另外，先到先得，号排在前面的人先取到水，取水之后就轮到下一个号取水，挺公平的，说明它是一种公平锁。

什么是可重入锁呢？
假定，取水时以家庭为单位，家庭的某人拿到号，其他的家庭成员过来打水，这时候不用再取号

排在1号的家庭，老公取号，假设其老婆来了，直接排第一个，正所谓妻凭夫贵。再看上图的2号，父亲正在打水，假设其儿子和女儿也到井边了，直接排第二个，所谓子凭父贵。总之，如果取水时以家庭为单位，则同一个家庭，可以直接复用排号，不用从后面排起重新取号。

以上这个故事模型中，取号一次，可以用来多次取水，其原理为可重入锁的模型。在重入锁模型中，一把独占锁，可以被多次锁定，这就叫做可重入锁。

## ReentrantLock重入锁

### synchronized的局限性

synchronized是java内置的关键字，它提供了一种独占的加锁方式。synchronized的获取和释放锁由jvm实现，用户不需要显示的释放锁，非常方便，然而synchronized也有一定的局限性，例如：

1. 当线程尝试获取锁的时候，如果获取不到锁会一直阻塞，这个阻塞的过程，用户无法控制
2. 如果获取锁的线程进入休眠或者阻塞，除非当前线程异常，否则其他线程尝试获取锁必须一直等待

JDK1.5之后发布，加入了Doug Lea实现的java.util.concurrent包。包内提供了Lock类，用来提供更多扩展的加锁功能。Lock弥补了synchronized的局限，提供了更加细粒度的加锁功能。

### ReentrantLock

ReentrantLock是Lock的默认实现，在聊ReentranLock之前，我们需要先弄清楚一些概念：

1. 可重入锁：可重入锁是指同一个线程可以多次获得同一把锁；ReentrantLock和关键字Synchronized都是可重入锁
2. 可中断锁：可中断锁时只线程在获取锁的过程中，是否可以相应线程中断操作。synchronized是不可中断的，ReentrantLock是可中断的
3. 公平锁和非公平锁：公平锁是指多个线程尝试获取同一把锁的时候，获取锁的顺序按照线程到达的先后顺序获取，而不是随机插队的方式获取。synchronized是非公平锁，而ReentrantLock是两种都可以实现，不过默认是非公平锁

### ReentrantLock基本使用

我们使用3个线程来对一个共享变量++操作，先使用**synchronized**实现，然后使用**ReentrantLock**实现。

**synchronized方式**：

```java
package com.itsoku.chat06;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo2 {

    private static int num = 0;

    private static synchronized void add() {
        num++;
    }

    public static class T extends Thread {
        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                Demo2.add();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T t1 = new T();
        T t2 = new T();
        T t3 = new T();

        t1.start();
        t2.start();
        t3.start();

        t1.join();
        t2.join();
        t3.join();

        System.out.println(Demo2.num);
    }
}
```

ReentrantLock方式：

```java
package com.itsoku.chat06;

import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo3 {

    private static int num = 0;

    private static ReentrantLock lock = new ReentrantLock();

    private static void add() {
        lock.lock();
        try {
            num++;
        } finally {
            lock.unlock();
        }

    }

    public static class T extends Thread {
        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                Demo3.add();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T t1 = new T();
        T t2 = new T();
        T t3 = new T();

        t1.start();
        t2.start();
        t3.start();

        t1.join();
        t2.join();
        t3.join();

        System.out.println(Demo3.num);
    }
}
```

**ReentrantLock的使用过程：**

1. **创建锁：ReentrantLock lock = new ReentrantLock();**
2. **获取锁：lock.lock()**
3. **释放锁：lock.unlock();**

对比上面的代码，与关键字synchronized相比，ReentrantLock锁有明显的操作过程，开发人员必须手动的指定何时加锁，何时释放锁，正是因为这样手动控制，ReentrantLock对逻辑控制的灵活度要远远胜于关键字synchronized，上面代码需要注意**lock.unlock()**一定要放在finally中，否则，若程序出现了异常，锁没有释放，那么其他线程就再也没有机会获取这个锁了。

### ReentrantLock是可重入锁

来验证一下ReentrantLock是可重入锁，实例代码：

```java
package com.itsoku.chat06;

import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo4 {
    private static int num = 0;
    private static ReentrantLock lock = new ReentrantLock();

    private static void add() {
        lock.lock();
        lock.lock();
        try {
            num++;
        } finally {
            lock.unlock();
            lock.unlock();
        }
    }

    public static class T extends Thread {
        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                Demo4.add();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T t1 = new T();
        T t2 = new T();
        T t3 = new T();

        t1.start();
        t2.start();
        t3.start();

        t1.join();
        t2.join();
        t3.join();

        System.out.println(Demo4.num);
    }
}
```

上面代码中add()方法中，当一个线程进入的时候，会执行2次获取锁的操作，运行程序可以正常结束，并输出和期望值一样的30000，假如ReentrantLock是不可重入的锁，那么同一个线程第2次获取锁的时候由于前面的锁还未释放而导致死锁，程序是无法正常结束的。ReentrantLock命名也挺好的Reentrant Lock，和其名字一样，可重入锁。

代码中还有几点需要注意：

1. **lock()方法和unlock()方法需要成对出现，锁了几次，也要释放几次，否则后面的线程无法获取锁了；可以将add中的unlock删除一个事实，上面代码运行将无法结束**
2. **unlock()方法放在finally中执行，保证不管程序是否有异常，锁必定会释放**

## ReentrantLock实现公平锁

在大多数情况下，锁的申请都是非公平的，也就是说，线程1首先请求锁A，接着线程2也请求了锁A。那么当锁A可用时，是线程1可获得锁还是线程2可获得锁呢？这是不一定的，系统只是会从这个锁的等待队列中随机挑选一个，因此不能保证其公平性。这就好比买票不排队，大家都围在售票窗口前，售票员忙的焦头烂额，也顾及不上谁先谁后，随便找个人出票就完事了，最终导致的结果是，有些人可能一直买不到票。而公平锁，则不是这样，它会按照到达的先后顺序获得资源。公平锁的一大特点是：它不会产生饥饿现象，只要你排队，最终还是可以等到资源的；synchronized关键字默认是有jvm内部实现控制的，是非公平锁。而ReentrantLock运行开发者自己设置锁的公平性。

看一下jdk中ReentrantLock的源码，2个构造方法：

```java
public ReentrantLock() {
	sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
	sync = fair ? new FairSync() : new NonfairSync();
}
```

默认构造方法创建的是非公平锁。

第2个构造方法，有个fair参数，当fair为true的时候创建的是公平锁，公平锁看起来很不错，不过要实现公平锁，系统内部肯定需要维护一个有序队列，因此公平锁的实现成本比较高，性能相对于非公平锁来说相对低一些。因此，在默认情况下，锁是非公平的，如果没有特别要求，则不建议使用公平锁。

公平锁和非公平锁在程序调度上是很不一样，来一个公平锁示例看一下：

```java
package com.itsoku.chat06;

import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo5 {
    private static int num = 0;
    private static ReentrantLock fairLock = new ReentrantLock(true);

    public static class T extends Thread {
        public T(String name) {
            super(name);
        }

        @Override
        public void run() {
            for (int i = 0; i < 5; i++) {
                fairLock.lock();
                try {
                    System.out.println(this.getName() + "获得锁!");
                } finally {
                    fairLock.unlock();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T t1 = new T("t1");
        T t2 = new T("t2");
        T t3 = new T("t3");

        t1.start();
        t2.start();
        t3.start();

        t1.join();
        t2.join();
        t3.join();
    }
}
```

运行结果输出：

```txt
t1获得锁!
t2获得锁!
t3获得锁!
t1获得锁!
t2获得锁!
t3获得锁!
t1获得锁!
t2获得锁!
t3获得锁!
t1获得锁!
t2获得锁!
t3获得锁!
t1获得锁!
t2获得锁!
t3获得锁!
```

看一下输出的结果，锁时按照先后顺序获得的。

修改一下上面代码，改为非公平锁试试，如下：

```java
ReentrantLock fairLock = new ReentrantLock(false);
```

运行结果如下：

```java
t1获得锁!
t3获得锁!
t3获得锁!
t3获得锁!
t3获得锁!
t1获得锁!
t1获得锁!
t1获得锁!
t1获得锁!
t2获得锁!
t2获得锁!
t2获得锁!
t2获得锁!
t2获得锁!
t3获得锁!
```

可以看到t3可能会连续获得锁，结果是比较随机的，不公平的。

### ReentrantLock获取锁的过程是可中断的

对于synchronized关键字，如果一个线程在等待获取锁，最终只有2种结果：

1. 要么获取到锁然后继续后面的操作
2. 要么一直等待，直到其他线程释放锁为止

而ReentrantLock提供了另外一种可能，就是在等的获取锁的过程中（**发起获取锁请求到还未获取到锁这段时间内**）是可以被中断的，也就是说在等待锁的过程中，程序可以根据需要取消获取锁的请求。有些使用这个操作是非常有必要的。比如：你和好朋友越好一起去打球，如果你等了半小时朋友还没到，突然你接到一个电话，朋友由于突发状况，不能来了，那么你一定达到回府。中断操作正是提供了一套类似的机制，如果一个线程正在等待获取锁，那么它依然可以收到一个通知，被告知无需等待，可以停止工作了。

示例代码：

```java
package com.itsoku.chat06;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo6 {
    private static ReentrantLock lock1 = new ReentrantLock(false);
    private static ReentrantLock lock2 = new ReentrantLock(false);

    public static class T extends Thread {
        int lock;

        public T(String name, int lock) {
            super(name);
            this.lock = lock;
        }

        @Override
        public void run() {
            try {
                if (this.lock == 1) {
                    lock1.lockInterruptibly();
                    TimeUnit.SECONDS.sleep(1);
                    lock2.lockInterruptibly();
                } else {
                    lock2.lockInterruptibly();
                    TimeUnit.SECONDS.sleep(1);
                    lock1.lockInterruptibly();
                }
            } catch (InterruptedException e) {
                System.out.println("中断标志:" + this.isInterrupted());
                e.printStackTrace();
            } finally {
                if (lock1.isHeldByCurrentThread()) {
                    lock1.unlock();
                }
                if (lock2.isHeldByCurrentThread()) {
                    lock2.unlock();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T t1 = new T("t1", 1);
        T t2 = new T("t2", 2);

        t1.start();
        t2.start();
    }
}
```

先运行一下上面代码，发现程序无法结束，使用jstack查看线程堆栈信息，发现2个线程死锁了。

```java
Found one Java-level deadlock:
=============================
"t2":
  waiting for ownable synchronizer 0x0000000717380c20, (a java.util.concurrent.locks.ReentrantLock$NonfairSync),
  which is held by "t1"
"t1":
  waiting for ownable synchronizer 0x0000000717380c50, (a java.util.concurrent.locks.ReentrantLock$NonfairSync),
  which is held by "t2"
```

lock1被线程t1占用，lock2倍线程t2占用，线程t1在等待获取lock2，线程t2在等待获取lock1，都在相互等待获取对方持有的锁，最终产生了死锁，如果是在synchronized关键字情况下发生了死锁现象，程序是无法结束的。

我们队上面代码改造一下，线程t2一直无法获取到lock1，那么等待5秒之后，我们中断获取锁的操作。主要修改一下main方法，如下：

```java
T t1 = new T("t1", 1);
T t2 = new T("t2", 2);

t1.start();
t2.start();

TimeUnit.SECONDS.sleep(5);
t2.interrupt();
```

新增了2行代码`TimeUnit.SECONDS.sleep(5);t2.interrupt();`，程序可以结束了，运行结果：

```java
java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireInterruptibly(AbstractQueuedSynchronizer.java:898)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireInterruptibly(AbstractQueuedSynchronizer.java:1222)
	at java.util.concurrent.locks.ReentrantLock.lockInterruptibly(ReentrantLock.java:335)
	at com.itsoku.chat06.Demo6$T.run(Demo6.java:31)
中断标志:false
```

从上面信息中可以看出，代码的31行触发了异常，**中断标志输出：false**

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190717191731394-781205156.png)

t2在31行一直获取不到lock1的锁，主线程中等待了5秒之后，t2线程调用了`interrupt()`方法，将线程的中断标志置为true，此时31行会触发`InterruptedException`异常，然后线程t2可以继续向下执行，释放了lock2的锁，然后线程t1可以正常获取锁，程序得以继续进行。线程发送中断信号触发InterruptedException异常之后，中断标志将被清空。

关于获取锁的过程中被中断，注意几点:

1. **ReentrankLock中必须使用实例方法`lockInterruptibly()`获取锁时，在线程调用interrupt()方法之后，才会引发`InterruptedException`异常**
2. **线程调用interrupt()之后，线程的中断标志会被置为true**
3. **触发InterruptedException异常之后，线程的中断标志有会被清空，即置为false**
4. **所以当线程调用interrupt()引发InterruptedException异常，中断标志的变化是:false->true->false**

### ReentrantLock锁申请等待限时

申请锁等待限时是什么意思？一般情况下，获取锁的时间我们是不知道的，synchronized关键字获取锁的过程中，只能等待其他线程把锁释放之后才能够有机会获取到锁。所以获取锁的时间有长有短。如果获取锁的时间能够设置超时时间，那就非常好了。

ReentrantLock刚好提供了这样功能，给我们提供了获取锁限时等待的方法`tryLock()`，可以选择传入时间参数，表示等待指定的时间，无参则表示立即返回锁申请的结果：true表示获取锁成功，false表示获取锁失败。

### tryLock无参方法

看一下源码中tryLock方法：

```java
public boolean tryLock()
```

返回boolean类型的值，此方法会立即返回，结果表示获取锁是否成功，示例：

```java
package com.itsoku.chat06;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo8 {
    private static ReentrantLock lock1 = new ReentrantLock(false);

    public static class T extends Thread {

        public T(String name) {
            super(name);
        }

        @Override
        public void run() {
            try {
                System.out.println(System.currentTimeMillis() + ":" + this.getName() + "开始获取锁!");
                //获取锁超时时间设置为3秒，3秒内是否能否获取锁都会返回
                if (lock1.tryLock()) {
                    System.out.println(System.currentTimeMillis() + ":" + this.getName() + "获取到了锁!");
                    //获取到锁之后，休眠5秒
                    TimeUnit.SECONDS.sleep(5);
                } else {
                    System.out.println(System.currentTimeMillis() + ":" + this.getName() + "未能获取到锁!");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                if (lock1.isHeldByCurrentThread()) {
                    lock1.unlock();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T t1 = new T("t1");
        T t2 = new T("t2");

        t1.start();
        t2.start();
    }
}
```

代码中获取锁成功之后，休眠5秒，会导致另外一个线程获取锁失败，运行代码，输出：

```java
1563356291081:t2开始获取锁!
1563356291081:t2获取到了锁!
1563356291081:t1开始获取锁!
1563356291081:t1未能获取到锁!
```

可以看到t2获取成功，t1获取失败了，tryLock()是立即响应的，中间不会有阻塞。

### tryLock有参方法

可以明确设置获取锁的超时时间，该方法签名：

```java
public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException
```

该方法在指定的时间内不管是否可以获取锁，都会返回结果，返回true，表示获取锁成功，返回false表示获取失败。此方法由2个参数，第一个参数是时间类型，是一个枚举，可以表示时、分、秒、毫秒等待，使用比较方便，第1个参数表示在时间类型上的时间长短。此方法在执行的过程中，如果调用了线程的中断interrupt()方法，会触发InterruptedException异常。

示例：

```java
package com.itsoku.chat06;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo7 {
    private static ReentrantLock lock1 = new ReentrantLock(false);

    public static class T extends Thread {

        public T(String name) {
            super(name);
        }

        @Override
        public void run() {
            try {
                System.out.println(System.currentTimeMillis() + ":" + this.getName() + "开始获取锁!");
                //获取锁超时时间设置为3秒，3秒内是否能否获取锁都会返回
                if (lock1.tryLock(3, TimeUnit.SECONDS)) {
                    System.out.println(System.currentTimeMillis() + ":" + this.getName() + "获取到了锁!");
                    //获取到锁之后，休眠5秒
                    TimeUnit.SECONDS.sleep(5);
                } else {
                    System.out.println(System.currentTimeMillis() + ":" + this.getName() + "未能获取到锁!");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                if (lock1.isHeldByCurrentThread()) {
                    lock1.unlock();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T t1 = new T("t1");
        T t2 = new T("t2");

        t1.start();
        t2.start();
    }
}
```

程序中调用了ReentrantLock的实例方法`tryLock(3, TimeUnit.SECONDS)`，表示获取锁的超时时间是3秒，3秒后不管是否能否获取锁，该方法都会有返回值，获取到锁之后，内部休眠了5秒，会导致另外一个线程获取锁失败。

运行程序，输出：

```java
1563355512901:t2开始获取锁!
1563355512901:t1开始获取锁!
1563355512902:t2获取到了锁!
1563355515904:t1未能获取到锁!
```

输出结果中分析，t2获取到锁了，然后休眠了5秒，t1获取锁失败，t1打印了2条信息，时间相差3秒左右。

**关于tryLock()方法和tryLock(long timeout, TimeUnit unit)方法，说明一下：**

1. 都会返回boolean值，结果表示获取锁是否成功
2. tryLock()方法，不管是否获取成功，都会立即返回；而有参的tryLock方法会尝试在指定的时间内去获取锁，中间会阻塞的现象，在指定的时间之后会不管是否能够获取锁都会返回结果
3. tryLock()方法不会响应线程的中断方法；而有参的tryLock方法会响应线程的中断方法，而出发`InterruptedException`异常，这个从2个方法的声明上可以可以看出来

## ReentrantLock其他常用的方法

1. isHeldByCurrentThread：实例方法，判断当前线程是否持有ReentrantLock的锁，上面代码中有使用过。

## 获取锁的4中方法对比

| 获取锁的方法                         | 是否立即响应(不会阻塞) | 是否响应中断 |
| ------------------------------------ | ---------------------- | ------------ |
| lock()                               | ×                      | ×            |
| lockInterruptibly()                  | ×                      | √            |
| tryLock()                            | √                      | ×            |
| tryLock(long timeout, TimeUnit unit) | ×                      | √            |

## 总结

1. ReentrantLock可以实现公平锁和非公平锁
2. ReentrantLock默认实现的是非公平锁
3. ReentrantLock的获取锁和释放锁必须成对出现，锁了几次，也要释放几次
4. 释放锁的操作必须放在finally中执行
5. lockInterruptibly()实例方法可以相应线程的中断方法，调用线程的interrupt()方法时，lockInterruptibly()方法会触发`InterruptedException`异常
6. 关于`InterruptedException`异常说一下，看到方法声明上带有 `throws InterruptedException`，表示该方法可以相应线程中断，调用线程的interrupt()方法时，这些方法会触发`InterruptedException`异常，触发InterruptedException时，线程的中断中断状态会被清除。所以如果程序由于调用`interrupt()`方法而触发`InterruptedException`异常，线程的标志由默认的false变为ture，然后又变为false
7. 实例方法tryLock()获会尝试获取锁，会立即返回，返回值表示是否获取成功
8. 实例方法tryLock(long timeout, TimeUnit unit)会在指定的时间内尝试获取锁，指定的时间内是否能够获取锁，都会返回，返回值表示是否获取锁成功，该方法会响应线程的中断

# Condition条件对象

本文目标：

1. synchronized中实现线程等待和唤醒
2. Condition简介及常用方法介绍及相关示例
3. 使用Condition实现生产者消费者
4. 使用Condition实现同步阻塞队列

## synchronized中等待和唤醒线程示例

```java
package com.itsoku.chat09;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo1 {
    static Object lock = new Object();

    public static class T1 extends Thread {
        @Override
        public void run() {
            System.out.println(System.currentTimeMillis() + "," + this.getName() + "准备获取锁!");
            synchronized (lock) {
                System.out.println(System.currentTimeMillis() + "," + this.getName() + "获取锁成功!");
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(System.currentTimeMillis() + "," + this.getName() + "释放锁成功!");
        }
    }

    public static class T2 extends Thread {
        @Override
        public void run() {
            System.out.println(System.currentTimeMillis() + "," + this.getName() + "准备获取锁!");
            synchronized (lock) {
                System.out.println(System.currentTimeMillis() + "," + this.getName() + "获取锁成功!");
                lock.notify();
                System.out.println(System.currentTimeMillis() + "," + this.getName() + " notify!");
                try {
                    TimeUnit.SECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(System.currentTimeMillis() + "," + this.getName() + "准备释放锁!");
            }
            System.out.println(System.currentTimeMillis() + "," + this.getName() + "释放锁成功!");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T1 t1 = new T1();
        t1.setName("t1");
        t1.start();
        TimeUnit.SECONDS.sleep(5);
        T2 t2 = new T2();
        t2.setName("t2");
        t2.start();
    }
}
```

输出：

```txt
1：1563530109234,t1准备获取锁!
2：1563530109234,t1获取锁成功!
3：1563530114236,t2准备获取锁!
4：1563530114236,t2获取锁成功!
5：1563530114236,t2 notify!
6：1563530119237,t2准备释放锁!
7：1563530119237,t2释放锁成功!
8：1563530119237,t1释放锁成功!
```

代码结合输出的结果我们分析一下：

1. 线程t1先获取锁，然后调用了wait()方法将线程置为等待状态，然后会释放lock的锁
2. 主线程等待5秒之后，启动线程t2，t2获取到了锁，结果中1、3行时间相差5秒左右
3. t2调用lock.notify()方法，准备将等待在lock上的线程t1唤醒，notify()方法之后又休眠了5秒，看一下输出的5、8可知，notify()方法之后，t1并不能立即被唤醒，需要等到t2将synchronized块执行完毕，释放锁之后，t1才被唤醒
4. wait()方法和notify()方法必须放在同步块内调用（synchronized块内），否则会报错

## Condition使用简介

在了解Condition之前，需要先了解一下重入锁ReentrantLock。

任何一个java对象都天然继承于Object类，在线程间实现通信的往往会应用到Object的几个方法，比如wait()、wait(long timeout)、wait(long timeout, int nanos)与notify()、notifyAll()几个方法实现等待/通知机制，同样的， 在java Lock体系下依然会有同样的方法实现等待/通知机制。

从整体上来看**Object的wait和notify/notify是与对象监视器配合完成线程间的等待/通知机制，而Condition与Lock配合完成等待通知机制，前者是java底层级别的，后者是语言级别的，具有更高的可控制性和扩展性**。两者除了在使用方式上不同外，在**功能特性**上还是有很多的不同：

1. Condition能够支持不响应中断，而通过使用Object方式不支持
2. Condition能够支持多个等待队列（new 多个Condition对象），而Object方式只能支持一个
3. Condition能够支持超时时间的设置，而Object不支持

Condition由ReentrantLock对象创建，并且可以同时创建多个，Condition接口在使用前必须先调用ReentrantLock的lock()方法获得锁，之后调用Condition接口的await()将释放锁，并且在该Condition上等待，直到有其他线程调用Condition的signal()方法唤醒线程，使用方式和wait()、notify()类似。

示例代码：

```java
package com.itsoku.chat09;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo2 {
    static ReentrantLock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();

    public static class T1 extends Thread {
        @Override
        public void run() {
            System.out.println(System.currentTimeMillis() + "," + this.getName() + "准备获取锁!");
            lock.lock();
            try {
                System.out.println(System.currentTimeMillis() + "," + this.getName() + "获取锁成功!");
                condition.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
            System.out.println(System.currentTimeMillis() + "," + this.getName() + "释放锁成功!");
        }
    }

    public static class T2 extends Thread {
        @Override
        public void run() {
            System.out.println(System.currentTimeMillis() + "," + this.getName() + "准备获取锁!");
            lock.lock();
            try {
                System.out.println(System.currentTimeMillis() + "," + this.getName() + "获取锁成功!");
                condition.signal();
                System.out.println(System.currentTimeMillis() + "," + this.getName() + " signal!");
                try {
                    TimeUnit.SECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(System.currentTimeMillis() + "," + this.getName() + "准备释放锁!");
            } finally {
                lock.unlock();
            }
            System.out.println(System.currentTimeMillis() + "," + this.getName() + "释放锁成功!");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T1 t1 = new T1();
        t1.setName("t1");
        t1.start();
        TimeUnit.SECONDS.sleep(5);
        T2 t2 = new T2();
        t2.setName("t2");
        t2.start();
    }
}
```

输出：

```java
1563532185827,t1准备获取锁!
1563532185827,t1获取锁成功!
1563532190829,t2准备获取锁!
1563532190829,t2获取锁成功!
1563532190829,t2 signal!
1563532195829,t2准备释放锁!
1563532195829,t2释放锁成功!
1563532195829,t1释放锁成功!
```

输出的结果和使用synchronized关键字的实例类似。

Condition.await()方法和Object.wait()方法类似，当使用Condition.await()方法时，需要先获取Condition对象关联的ReentrantLock的锁，在Condition.await()方法被调用时，当前线程会释放这个锁，并且当前线程会进行等待（处于阻塞状态）。在signal()方法被调用后，系统会从Condition对象的等待队列中唤醒一个线程，一旦线程被唤醒，被唤醒的线程会尝试重新获取锁，一旦获取成功，就可以继续执行了。因此，在signal被调用后，一般需要释放相关的锁，让给其他被唤醒的线程，让他可以继续执行。

## Condition常用方法

Condition接口提供的常用方法有：

> **和Object中wait类似的方法**

1. void await() throws InterruptedException:当前线程进入等待状态，如果其他线程调用condition的signal或者signalAll方法并且当前线程获取Lock从await方法返回，如果在等待状态中被中断会抛出被中断异常；
2. long awaitNanos(long nanosTimeout)：当前线程进入等待状态直到被通知，中断或者**超时**；
3. boolean await(long time, TimeUnit unit) throws InterruptedException：同第二种，支持自定义时间单位，false：表示方法超时之后自动返回的，true：表示等待还未超时时，await方法就返回了（超时之前，被其他线程唤醒了）
4. boolean awaitUntil(Date deadline) throws InterruptedException：当前线程进入等待状态直到被通知，中断或者**到了某个时间**
5. void awaitUninterruptibly();：当前线程进入等待状态，不会响应线程中断操作，只能通过唤醒的方式让线程继续

> **和Object的notify/notifyAll类似的方法**

1. void signal()：唤醒一个等待在condition上的线程，将该线程从**等待队列**中转移到**同步队列**中，如果在同步队列中能够竞争到Lock则可以从等待方法中返回。
2. void signalAll()：与1的区别在于能够唤醒所有等待在condition上的线程

## Condition.await()过程中被打断

```java
package com.itsoku.chat09;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo4 {
    static ReentrantLock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();

    public static class T1 extends Thread {
        @Override
        public void run() {
            lock.lock();
            try {
                condition.await();
            } catch (InterruptedException e) {
                System.out.println("中断标志：" + this.isInterrupted());
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T1 t1 = new T1();
        t1.setName("t1");
        t1.start();
        TimeUnit.SECONDS.sleep(2);
        //给t1线程发送中断信号
        System.out.println("1、t1中断标志：" + t1.isInterrupted());
        t1.interrupt();
        System.out.println("2、t1中断标志：" + t1.isInterrupted());
    }
}
```

输出：

```java
1、t1中断标志：false
2、t1中断标志：true
中断标志：false
java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.reportInterruptAfterWait(AbstractQueuedSynchronizer.java:2014)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2048)
	at com.itsoku.chat09.Demo4$T1.run(Demo4.java:19)
```

调用condition.await()之后，线程进入阻塞中，调用t1.interrupt()，给t1线程发送中断信号，await()方法内部会检测到线程中断信号，然后触发`InterruptedException`异常，线程中断标志被清除。从输出结果中可以看出，线程t1中断标志的变换过程：false->true->false

## await(long time, TimeUnit unit)超时之后自动返回

```java
package com.itsoku.chat09;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo5 {
    static ReentrantLock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();

    public static class T1 extends Thread {
        @Override
        public void run() {
            lock.lock();
            try {
                System.out.println(System.currentTimeMillis() + "," + this.getName() + ",start");
                boolean r = condition.await(2, TimeUnit.SECONDS);
                System.out.println(r);
                System.out.println(System.currentTimeMillis() + "," + this.getName() + ",end");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T1 t1 = new T1();
        t1.setName("t1");
        t1.start();
    }
}
```

输出：

```java
1563541624082,t1,start
false
1563541626085,t1,end
```

t1线程等待2秒之后，自动返回继续执行，最后await方法返回false，**await返回false表示超时之后自动返回**

## await(long time, TimeUnit unit)超时之前被唤醒

```java
package com.itsoku.chat09;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo6 {
    static ReentrantLock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();

    public static class T1 extends Thread {
        @Override
        public void run() {
            lock.lock();
            try {
                System.out.println(System.currentTimeMillis() + "," + this.getName() + ",start");
                boolean r = condition.await(5, TimeUnit.SECONDS);
                System.out.println(r);
                System.out.println(System.currentTimeMillis() + "," + this.getName() + ",end");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T1 t1 = new T1();
        t1.setName("t1");
        t1.start();
        //休眠1秒之后，唤醒t1线程
        TimeUnit.SECONDS.sleep(1);
        lock.lock();
        try {
            condition.signal();
        } finally {
            lock.unlock();
        }
    }
}
```

输出：

```txt
1563542046046,t1,start
true
1563542047048,t1,end
```

t1线程中调用`condition.await(5, TimeUnit.SECONDS);`方法会释放锁，等待5秒，主线程休眠1秒，然后获取锁，之后调用signal()方法唤醒t1，输出结果中发现await后过了1秒（1、3行输出结果的时间差），await方法就返回了，并且返回值是true。**true表示await方法超时之前被其他线程唤醒了。**

## long awaitNanos(long nanosTimeout)超时返回

```java
package com.itsoku.chat09;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo7 {
    static ReentrantLock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();

    public static class T1 extends Thread {
        @Override
        public void run() {
            lock.lock();
            try {
                System.out.println(System.currentTimeMillis() + "," + this.getName() + ",start");
                long r = condition.awaitNanos(TimeUnit.SECONDS.toNanos(5));
                System.out.println(r);
                System.out.println(System.currentTimeMillis() + "," + this.getName() + ",end");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T1 t1 = new T1();
        t1.setName("t1");
        t1.start();
    }
}
```

输出：

```sql
1563542547302,t1,start
-258200
1563542552304,t1,end
```

**awaitNanos参数为纳秒，可以调用TimeUnit中的一些方法将时间转换为纳秒。**

t1调用await方法等待5秒超时返回，返回结果为负数，表示超时之后返回的。

## waitNanos(long nanosTimeout)超时之前被唤醒

```java
package com.itsoku.chat09;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo8 {
    static ReentrantLock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();

    public static class T1 extends Thread {
        @Override
        public void run() {
            lock.lock();
            try {
                System.out.println(System.currentTimeMillis() + "," + this.getName() + ",start");
                long r = condition.awaitNanos(TimeUnit.SECONDS.toNanos(5));
                System.out.println(r);
                System.out.println(System.currentTimeMillis() + "," + this.getName() + ",end");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T1 t1 = new T1();
        t1.setName("t1");
        t1.start();
        //休眠1秒之后，唤醒t1线程
        TimeUnit.SECONDS.sleep(1);
        lock.lock();
        try {
            condition.signal();
        } finally {
            lock.unlock();
        }
    }
}
```

输出：

```sql
1563542915991,t1,start
3999988500
1563542916992,t1,end
```

t1中调用await休眠5秒，主线程休眠1秒之后，调用signal()唤醒线程t1，await方法返回正数，表示返回时距离超时时间还有多久，将近4秒，返回正数表示，线程在超时之前被唤醒了。

**其他几个有参的await方法和无参的await方法一样，线程调用interrupt()方法时，这些方法都会触发InterruptedException异常，并且线程的中断标志会被清除。**

## 同一个锁支持创建多个Condition

使用两个Condition来实现一个阻塞队列的例子：

```java
package com.itsoku.chat09;

import java.util.LinkedList;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class BlockingQueueDemo<E> {
    int size;//阻塞队列最大容量

    ReentrantLock lock = new ReentrantLock();

    LinkedList<E> list = new LinkedList<>();//队列底层实现

    Condition notFull = lock.newCondition();//队列满时的等待条件
    Condition notEmpty = lock.newCondition();//队列空时的等待条件

    public BlockingQueueDemo(int size) {
        this.size = size;
    }

    public void enqueue(E e) throws InterruptedException {
        lock.lock();
        try {
            while (list.size() == size)//队列已满,在notFull条件上等待
                notFull.await();
            list.add(e);//入队:加入链表末尾
            System.out.println("入队：" + e);
            notEmpty.signal(); //通知在notEmpty条件上等待的线程
        } finally {
            lock.unlock();
        }
    }

    public E dequeue() throws InterruptedException {
        E e;
        lock.lock();
        try {
            while (list.size() == 0)//队列为空,在notEmpty条件上等待
                notEmpty.await();
            e = list.removeFirst();//出队:移除链表首元素
            System.out.println("出队：" + e);
            notFull.signal();//通知在notFull条件上等待的线程
            return e;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        BlockingQueueDemo<Integer> queue = new BlockingQueueDemo<>(2);
        for (int i = 0; i < 10; i++) {
            int data = i;
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        queue.enqueue(data);
                    } catch (InterruptedException e) {

                    }
                }
            }).start();
        }
        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Integer data = queue.dequeue();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }
}
```

代码非常容易理解，创建了一个阻塞队列，大小为3，队列满的时候，会被阻塞，等待其他线程去消费，队列中的元素被消费之后，会唤醒生产者，生产数据进入队列。上面代码将队列大小置为1，可以实现同步阻塞队列，生产1个元素之后，生产者会被阻塞，待消费者消费队列中的元素之后，生产者才能继续工作。

## Object的监视器方法与Condition接口的对比

| 对比项                                     | Object 监视器方法           | Condition                                                    |
| ------------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| 前置条件                                   | 获取对象的锁                | 调用Lock.lock获取锁，调用Lock.newCondition()获取Condition对象 |
| 调用方式                                   | 直接调用，如：object.wait() | 直接调用，如：condition.await()                              |
| 等待队列个数                               | 一个                        | 多个，使用多个condition实现                                  |
| 当前线程释放锁并进入等待状态               | 支持                        | 支持                                                         |
| 当前线程释放锁进入等待状态中不响应中断     | 不支持                      | 支持                                                         |
| 当前线程释放锁并进入超时等待状态           | 支持                        | 支持                                                         |
| 当前线程释放锁并进入等待状态到将来某个时间 | 不支持                      | 支持                                                         |
| 唤醒等待队列中的一个线程                   | 支持                        | 支持                                                         |
| 唤醒等待队列中的全部线程                   | 支持                        | 支持                                                         |

## 总结

1. 使用condition的步骤：创建condition对象，获取锁，然后调用condition的方法
2. 一个ReentrantLock支持创建多个condition对象
3. `void await() throws InterruptedException;`方法会释放锁，让当前线程等待，支持唤醒，支持线程中断
4. `void awaitUninterruptibly();`方法会释放锁，让当前线程等待，支持唤醒，不支持线程中断
5. `long awaitNanos(long nanosTimeout) throws InterruptedException;`参数为纳秒，此方法会释放锁，让当前线程等待，支持唤醒，支持中断。超时之后返回的，结果为负数；超时之前返回的，结果为正数（表示返回时距离超时时间相差的纳秒数）
6. `boolean await(long time, TimeUnit unit) throws InterruptedException;`方法会释放锁，让当前线程等待，支持唤醒，支持中断。超时之后返回的，结果为false；超时之前返回的，结果为true
7. `boolean awaitUntil(Date deadline) throws InterruptedException;`参数表示超时的截止时间点，方法会释放锁，让当前线程等待，支持唤醒，支持中断。超时之后返回的，结果为false；超时之前返回的，结果为true
8. `void signal();`会唤醒一个等待中的线程，然后被唤醒的线程会被加入同步队列，去尝试获取锁
9. `void signalAll();`会唤醒所有等待中的线程，将所有等待中的线程加入同步队列，然后去尝试获取锁

# LockSupport（线程工具类）

**本文主要内容：**

1. **讲解3种让线程等待和唤醒的方法，每种方法配合具体的示例**
2. **介绍LockSupport主要用法**
3. **对比3种方式，了解他们之间的区别**

**LockSupport位于java.util.concurrent（简称juc）包中，算是juc中一个基础类，juc中很多地方都会使用LockSupport，非常重要，希望大家一定要掌握。**

关于线程等待/唤醒的方法，前面的文章中我们已经讲过2种了：

1. 方式1：使用Object中的wait()方法让线程等待，使用Object中的notify()方法唤醒线程
2. 方式2：使用juc包中Condition的await()方法让线程等待，使用signal()方法唤醒线程

这2种方式，我们先来看一下示例。

## 使用Object类中的方法实现线程等待和唤醒

### 示例1：

```java
package com.itsoku.chat10;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo1 {

    static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            synchronized (lock) {
                System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " start!");
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " 被唤醒!");
            }
        });
        t1.setName("t1");
        t1.start();
        //休眠5秒
        TimeUnit.SECONDS.sleep(5);
        synchronized (lock) {
            lock.notify();
        }
    }
}
```

输出：

```txt
1563592938744,t1 start!
1563592943745,t1 被唤醒!
```

t1线程中调用`lock.wait()`方法让t1线程等待，主线程中休眠5秒之后，调用`lock.notify()`方法唤醒了t1线程，输出的结果中，两行结果相差5秒左右，程序正常退出。

### 示例2

我们把上面代码中main方法内部改一下，删除了`synchronized`关键字，看看有什么效果：

```java
package com.itsoku.chat10;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo2 {

    static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " start!");
            try {
                lock.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " 被唤醒!");
        });
        t1.setName("t1");
        t1.start();
        //休眠5秒
        TimeUnit.SECONDS.sleep(5);
        lock.notify();
    }
}
```

运行结果：

```java
Exception in thread "t1" java.lang.IllegalMonitorStateException
1563593178811,t1 start!
	at java.lang.Object.wait(Native Method)
	at java.lang.Object.wait(Object.java:502)
	at com.itsoku.chat10.Demo2.lambda$main$0(Demo2.java:16)
	at java.lang.Thread.run(Thread.java:745)
Exception in thread "main" java.lang.IllegalMonitorStateException
	at java.lang.Object.notify(Native Method)
	at com.itsoku.chat10.Demo2.main(Demo2.java:26)
```

上面代码中将**synchronized**去掉了，发现调用wait()方法和调用notify()方法都抛出了`IllegalMonitorStateException`异常，原因：**Object类中的wait、notify、notifyAll用于线程等待和唤醒的方法，都必须在同步代码中运行（必须用到关键字synchronized）**。

### 示例3

唤醒方法在等待方法之前执行，线程能够被唤醒么？代码如下：

```java
package com.itsoku.chat10;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo3 {

    static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (lock) {
                System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " start!");
                try {
                    //休眠3秒
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " 被唤醒!");
            }
        });
        t1.setName("t1");
        t1.start();
        //休眠1秒之后唤醒lock对象上等待的线程
        TimeUnit.SECONDS.sleep(1);
        synchronized (lock) {
            lock.notify();
        }
        System.out.println("lock.notify()执行完毕");
    }
}
```

运行代码，输出结果：

```java
lock.notify()执行完毕
1563593869797,t1 start!
```

输出了上面2行之后，程序一直无法结束，t1线程调用wait()方法之后无法被唤醒了，从输出中可见，`notify()`方法在`wait()`方法之前执行了，等待的线程无法被唤醒了。说明：唤醒方法在等待方法之前执行，线程无法被唤醒。

**关于Object类中的用户线程等待和唤醒的方法，总结一下：**

1. **wait()/notify()/notifyAll()方法都必须放在同步代码（必须在synchronized内部执行）中执行，需要先获取锁**
2. **线程唤醒的方法（notify、notifyAll）需要在等待的方法（wait）之后执行，等待中的线程才可能会被唤醒，否则无法唤醒**

## 使用Condition实现线程的等待和唤醒

Condition的使用，前面的文章讲过，对这块不熟悉的可以移步[JUC中Condition的使用](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933120&idx=1&sn=63ffe3ff64dcaf0418816febfd1e129a&chksm=88621b3ebf159228df5f5a501160fafa5d87412a4f03298867ec9325c0be57cd8e329f3b5ad1&token=476165288&lang=zh_CN#rd)，关于Condition我们准备了3个示例。

### 示例1

```java
package com.itsoku.chat10;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo4 {

    static ReentrantLock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            lock.lock();
            try {
                System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " start!");
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " 被唤醒!");
            } finally {
                lock.unlock();
            }
        });
        t1.setName("t1");
        t1.start();
        //休眠5秒
        TimeUnit.SECONDS.sleep(5);
        lock.lock();
        try {
            condition.signal();
        } finally {
            lock.unlock();
        }

    }
}
```

输出：

```java
1563594349632,t1 start!
1563594354634,t1 被唤醒!
```

t1线程启动之后调用`condition.await()`方法将线程处于等待中，主线程休眠5秒之后调用`condition.signal()`方法将t1线程唤醒成功，输出结果中2个时间戳相差5秒。

### 示例2

我们将上面代码中的lock.lock()、lock.unlock()去掉，看看会发生什么。代码：

```java
package com.itsoku.chat10;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo5 {

    static ReentrantLock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " start!");
            try {
                condition.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " 被唤醒!");
        });
        t1.setName("t1");
        t1.start();
        //休眠5秒
        TimeUnit.SECONDS.sleep(5);
        condition.signal();
    }
}
```

输出：

```java
Exception in thread "t1" java.lang.IllegalMonitorStateException
1563594654865,t1 start!
	at java.util.concurrent.locks.ReentrantLock$Sync.tryRelease(ReentrantLock.java:151)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.release(AbstractQueuedSynchronizer.java:1261)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.fullyRelease(AbstractQueuedSynchronizer.java:1723)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2036)
	at com.itsoku.chat10.Demo5.lambda$main$0(Demo5.java:19)
	at java.lang.Thread.run(Thread.java:745)
Exception in thread "main" java.lang.IllegalMonitorStateException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.signal(AbstractQueuedSynchronizer.java:1939)
	at com.itsoku.chat10.Demo5.main(Demo5.java:29)
```

有异常发生，`condition.await();`和`condition.signal();`都触发了`IllegalMonitorStateException`异常。原因：**调用condition中线程等待和唤醒的方法的前提是必须要先获取lock的锁**。

### 示例3

唤醒代码在等待之前执行，线程能够被唤醒么？代码如下：

```java
package com.itsoku.chat10;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo6 {

    static ReentrantLock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            lock.lock();
            try {
                System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " start!");
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " 被唤醒!");
            } finally {
                lock.unlock();
            }
        });
        t1.setName("t1");
        t1.start();
        //休眠5秒
        TimeUnit.SECONDS.sleep(1);
        lock.lock();
        try {
            condition.signal();
        } finally {
            lock.unlock();
        }
        System.out.println(System.currentTimeMillis() + ",condition.signal();执行完毕");
    }
}
```

运行结果:

```java
1563594886532,condition.signal();执行完毕
1563594890532,t1 start!
```

输出上面2行之后，程序无法结束，代码结合输出可以看出signal()方法在await()方法之前执行的，最终t1线程无法被唤醒，导致程序无法结束。

**关于Condition中方法使用总结：**

1. **使用Condtion中的线程等待和唤醒方法之前，需要先获取锁。否者会报`IllegalMonitorStateException`异常**
2. **signal()方法先于await()方法之前调用，线程无法被唤醒**

## Object和Condition的局限性

关于Object和Condtion中线程等待和唤醒的局限性，有以下几点：

1. **2中方式中的让线程等待和唤醒的方法能够执行的先决条件是：线程需要先获取锁**
2. **唤醒方法需要在等待方法之后调用，线程才能够被唤醒**

关于这2点，LockSupport都不需要，就能实现线程的等待和唤醒。下面我们来说一下LockSupport类。

## LockSupport类介绍

LockSupport类可以阻塞当前线程以及唤醒指定被阻塞的线程。LockSupport是一个线程工具类，所有的方法都是静态方法，可以让线程在任意位置阻塞，也可以在任意位置唤醒。

它的内部其实两类主要的方法：park（停车阻塞线程）和unpark（启动唤醒线程）。

主要是通过**park()**和**unpark(thread)**方法来实现阻塞和唤醒线程的操作的。

> 每个线程都有一个许可(permit)，**permit只有两个值1和0**，默认是0。
>
> 1. 当调用unpark(thread)方法，就会将thread线程的许可permit设置成1(**注意多次调用unpark方法，不会累加，permit值还是1**)。
> 2. 当调用park()方法，如果当前线程的permit是1，那么将permit设置为0，并立即返回。如果当前线程的permit是0，那么当前线程就会阻塞，直到别的线程将当前线程的permit设置为1时，park方法会被唤醒，然后会将permit再次设置为0，并返回。
>
> 注意：因为permit默认是0，所以一开始调用park()方法，线程必定会被阻塞。调用unpark(thread)方法后，会自动唤醒thread线程，即park方法立即返回。

### LockSupport中常用的方法

**阻塞线程**

- void park()：阻塞当前线程，如果调用**unpark方法**或者**当前线程被中断**，从能从park()方法中返回
- void park(Object blocker)：功能同方法1，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查
- void parkNanos(long nanos)：阻塞当前线程，最长不超过nanos纳秒，增加了超时返回的特性
- void parkNanos(Object blocker, long nanos)：功能同方法3，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查
- void parkUntil(long deadline)：阻塞当前线程，直到deadline，deadline是一个绝对时间，表示某个时间的毫秒格式
- void parkUntil(Object blocker, long deadline)：功能同方法5，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；

**唤醒线程**

- void unpark(Thread thread):唤醒处于阻塞状态的指定线程

### 示例1

主线程线程等待5秒之后，唤醒t1线程，代码如下：

```java
package com.itsoku.chat10;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.LockSupport;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo7 {

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " start!");
            LockSupport.park();
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " 被唤醒!");
        });
        t1.setName("t1");
        t1.start();
        //休眠5秒
        TimeUnit.SECONDS.sleep(5);
        LockSupport.unpark(t1);
        System.out.println(System.currentTimeMillis() + ",LockSupport.unpark();执行完毕");
    }
}
```

输出：

```java
1563597664321,t1 start!
1563597669323,LockSupport.unpark();执行完毕
1563597669323,t1 被唤醒!
```

t1中调用`LockSupport.park();`让当前线程t1等待，主线程休眠了5秒之后，调用`LockSupport.unpark(t1);`将t1线程唤醒，输出结果中1、3行结果相差5秒左右，说明t1线程等待5秒之后，被唤醒了。

`LockSupport.park();`无参数，内部直接会让当前线程处于等待中；unpark方法传递了一个线程对象作为参数，表示将对应的线程唤醒。

### 示例2

唤醒方法放在等待方法之前执行，看一下线程是否能够被唤醒呢？代码如下：

```java
package com.itsoku.chat10;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.LockSupport;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo8 {

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " start!");
            LockSupport.park();
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " 被唤醒!");
        });
        t1.setName("t1");
        t1.start();
        //休眠1秒
        TimeUnit.SECONDS.sleep(1);
        LockSupport.unpark(t1);
        System.out.println(System.currentTimeMillis() + ",LockSupport.unpark();执行完毕");
    }
}
```

输出：

```java
1563597994295,LockSupport.unpark();执行完毕
1563597998296,t1 start!
1563597998296,t1 被唤醒!
```

代码中启动t1线程，t1线程内部休眠了5秒，然后主线程休眠1秒之后，调用了`LockSupport.unpark(t1);`唤醒线程t1，此时`LockSupport.park();`方法还未执行，说明唤醒方法在等待方法之前执行的；输出结果中2、3行结果时间一样，表示`LockSupport.park();`没有阻塞了，是立即返回的。

说明：**唤醒方法在等待方法之前执行，线程也能够被唤醒，这点是另外2中方法无法做到的。Object和Condition中的唤醒必须在等待之后调用，线程才能被唤醒。而LockSupport中，唤醒的方法不管是在等待之前还是在等待之后调用，线程都能够被唤醒。**

### 示例3

park()让线程等待之后，是否能够响应线程中断？代码如下：

```java
package com.itsoku.chat10;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.LockSupport;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo9 {

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " start!");
            System.out.println(Thread.currentThread().getName() + ",park()之前中断标志：" + Thread.currentThread().isInterrupted());
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + ",park()之后中断标志：" + Thread.currentThread().isInterrupted());
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + " 被唤醒!");
        });
        t1.setName("t1");
        t1.start();
        //休眠5秒
        TimeUnit.SECONDS.sleep(5);
        t1.interrupt();

    }
}
```

输出：

```java
1563598536736,t1 start!
t1,park()之前中断标志：false
t1,park()之后中断标志：true
1563598541736,t1 被唤醒!
```

t1线程中调用了park()方法让线程等待，主线程休眠了5秒之后，调用`t1.interrupt();`给线程t1发送中断信号，然后线程t1从等待中被唤醒了，输出结果中的1、4行结果相差5秒左右，刚好是主线程休眠了5秒之后将t1唤醒了。**结论：park方法可以相应线程中断。**

**LockSupport.park方法让线程等待之后，唤醒方式有2种：**

1. **调用LockSupport.unpark方法**
2. **调用等待线程的`interrupt()`方法，给等待的线程发送中断信号，可以唤醒线程**

### 示例4

LockSupport有几个阻塞放有一个blocker参数，这个参数什么意思，上一个实例代码，大家一看就懂了：

```java
package com.itsoku.chat10;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.LockSupport;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo10 {

    static class BlockerDemo {
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            LockSupport.park();
        });
        t1.setName("t1");
        t1.start();

        Thread t2 = new Thread(() -> {
            LockSupport.park(new BlockerDemo());
        });
        t2.setName("t2");
        t2.start();
    }
}
```

运行上面代码，然后用jstack查看一下线程的堆栈信息：

```java
"t2" #13 prio=5 os_prio=0 tid=0x00000000293ea800 nid=0x91e0 waiting on condition [0x0000000029c3f000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000007180bfeb0> (a com.itsoku.chat10.Demo10$BlockerDemo)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at com.itsoku.chat10.Demo10.lambda$main$1(Demo10.java:22)
        at com.itsoku.chat10.Demo10$$Lambda$2/824909230.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:745)

"t1" #12 prio=5 os_prio=0 tid=0x00000000293ea000 nid=0x9d4 waiting on condition [0x0000000029b3f000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:304)
        at com.itsoku.chat10.Demo10.lambda$main$0(Demo10.java:16)
        at com.itsoku.chat10.Demo10$$Lambda$1/1389133897.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:745)
```

代码中，线程t1和t2的不同点是，t2中调用park方法传入了一个BlockerDemo对象，从上面的线程堆栈信息中，发现t2线程的堆栈信息中多了一行`- parking to wait for <0x00000007180bfeb0> (a com.itsoku.chat10.Demo10$BlockerDemo)`，刚好是传入的BlockerDemo对象，park传入的这个参数可以让我们在线程堆栈信息中方便排查问题，其他暂无他用。

**LockSupport的其他等待方法，包含有超时时间了，过了超时时间，等待方法会自动返回，让线程继续运行，这些方法在此就不提供示例了，有兴趣的朋友可以自己动动手，练一练。**

## 线程等待和唤醒的3种方式做个对比

到目前为止，已经说了3种让线程等待和唤醒的方法了

1. 方式1：Object中的wait、notify、notifyAll方法
2. 方式2：juc中Condition接口提供的await、signal、signalAll方法
3. 方式3：juc中的LockSupport提供的park、unpark方法

**3种方式对比：**

|                                        | Object                   | Condtion           | LockSupport |
| -------------------------------------- | ------------------------ | ------------------ | ----------- |
| 前置条件                               | 需要在synchronized中运行 | 需要先获取Lock的锁 | 无          |
| 无限等待                               | 支持                     | 支持               | 支持        |
| 超时等待                               | 支持                     | 支持               | 支持        |
| 等待到将来某个时间返回                 | 不支持                   | 支持               | 支持        |
| 等待状态中释放锁                       | 会释放                   | 会释放             | 不会释放    |
| 唤醒方法先于等待方法执行，能否唤醒线程 | 否                       | 否                 | 可以        |
| 是否能响应线程中断                     | 是                       | 是                 | 是          |
| 线程中断是否会清除中断标志             | 是                       | 是                 | 否          |
| 是否支持等待状态中不响应中断           | 不支持                   | 支持               | 不支持      |

## Semaphore（信号量）

Semaphore（信号量）为多线程协作提供了更为强大的控制方法，synchronized和重入锁ReentrantLock，这2种锁一次都只能允许一个线程访问一个资源，而信号量可以控制有多少个线程可以同时访问特定的资源。

**Semaphore常用场景：限流**

举个例子：

比如有个停车场，有5个空位，门口有个门卫，手中5把钥匙分别对应5个车位上面的锁，来一辆车，门卫会给司机一把钥匙，然后进去找到对应的车位停下来，出去的时候司机将钥匙归还给门卫。停车场生意比较好，同时来了100辆车，门卫手中只有5把钥匙，同时只能放5辆车进入，其他车只能等待，等有人将钥匙归还给门卫之后，才能让其他车辆进入。

上面的例子中门卫就相当于Semaphore，车钥匙就相当于许可证，车就相当于线程。

## Semaphore**常用方法**

- **Semaphore(int permits)**：构造方法，参数表示许可证数量，用来创建信号量
- **Semaphore(int permits,boolean fair)**：构造方法，当fair等于true时，创建具有给定许可数的计数信号量并设置为公平信号量
- **void acquire() throws InterruptedException**：从此信号量获取1个许可前线程将一直阻塞，相当于一辆车占了一个车位，此方法会响应线程中断，表示调用线程的interrupt方法，会使该方法抛出InterruptedException异常
- **void acquire(int permits) throws InterruptedException** ：和acquire()方法类似，参数表示需要获取许可的数量；比如一个大卡车要入停车场，由于车比较大，需要申请3个车位才可以停放
- **void acquireUninterruptibly(int permits)** ：和acquire(int permits) 方法类似，只是不会响应线程中断
- **boolean tryAcquire()**：尝试获取1个许可，不管是否能够获取成功，都立即返回，true表示获取成功，false表示获取失败
- **boolean tryAcquire(int permits)**：和tryAcquire()，表示尝试获取permits个许可
- **boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException**：尝试在指定的时间内获取1个许可，获取成功返回true，指定的时间过后还是无法获取许可，返回false
- **boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException**：和tryAcquire(long timeout, TimeUnit unit)类似，多了一个permits参数，表示尝试获取permits个许可
- **void release()**：释放一个许可，将其返回给信号量，相当于车从停车场出去时将钥匙归还给门卫
- **void release(int n)**：释放n个许可
- **int availablePermits()**：当前可用的许可数

## Semaphore简单的使用

```java
package com.itsoku.chat12;

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo1 {
    static Semaphore semaphore = new Semaphore(2);

    public static class T extends Thread {
        public T(String name) {
            super(name);
        }

        @Override
        public void run() {
            Thread thread = Thread.currentThread();
            try {
                semaphore.acquire();
                System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",获取许可!");
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release();
                System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",释放许可!");
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            new T("t-" + i).start();
        }
    }
}
```

输出：

```java
1563715791327,t-0,获取许可!
1563715791327,t-1,获取许可!
1563715794328,t-0,释放许可!
1563715794328,t-5,获取许可!
1563715794328,t-1,释放许可!
1563715794328,t-2,获取许可!
1563715797328,t-2,释放许可!
1563715797328,t-6,获取许可!
1563715797328,t-5,释放许可!
1563715797328,t-3,获取许可!
1563715800329,t-6,释放许可!
1563715800329,t-9,获取许可!
1563715800329,t-3,释放许可!
1563715800329,t-7,获取许可!
1563715803330,t-7,释放许可!
1563715803330,t-8,获取许可!
1563715803330,t-9,释放许可!
1563715803330,t-4,获取许可!
1563715806330,t-8,释放许可!
1563715806330,t-4,释放许可!
```

代码中`new Semaphore(2)`创建了许可数量为2的信号量，每个线程获取1个许可，同时允许两个线程获取许可，从输出中也可以看出，同时有两个线程可以获取许可，其他线程需要等待已获取许可的线程释放许可之后才能运行。为获取到许可的线程会阻塞在`acquire()`方法上，直到获取到许可才能继续。

## 获取许可之后不释放

门卫（Semaphore）有点呆，司机进去的时候给了钥匙，出来的时候不归还，门卫也不会说什么。最终结果就是其他车辆都无法进入了。

如下代码：

```java
package com.itsoku.chat12;

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo2 {
    static Semaphore semaphore = new Semaphore(2);

    public static class T extends Thread {
        public T(String name) {
            super(name);
        }

        @Override
        public void run() {
            Thread thread = Thread.currentThread();
            try {
                semaphore.acquire();
                System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",获取许可!");
                TimeUnit.SECONDS.sleep(3);
                System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",运行结束!");
                System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",当前可用许可数量:" + semaphore.availablePermits());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            new T("t-" + i).start();
        }
    }
}
```

输出：

```txt
1563716603924,t-0,获取许可!
1563716603924,t-1,获取许可!
1563716606925,t-0,运行结束!
1563716606925,t-0,当前可用许可数量:0
1563716606925,t-1,运行结束!
1563716606925,t-1,当前可用许可数量:0
```

上面程序运行后一直无法结束，观察一下代码，代码中获取许可后，没有释放许可的代码，最终导致，可用许可数量为0，其他线程无法获取许可，会在`semaphore.acquire();`处等待，导致程序无法结束。

## 释放许可正确的姿势

示例1中，在finally里面释放锁，会有问题么？

如果获取锁的过程中发生异常，导致获取锁失败，最后finally里面也释放了许可，最终会怎么样，导致许可数量凭空增长了。

示例代码：

```java
package com.itsoku.chat12;

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo3 {
    static Semaphore semaphore = new Semaphore(1);

    public static class T extends Thread {
        public T(String name) {
            super(name);
        }

        @Override
        public void run() {
            Thread thread = Thread.currentThread();
            try {
                semaphore.acquire();
                System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",获取许可,当前可用许可数量:" + semaphore.availablePermits());
                //休眠100秒
                TimeUnit.SECONDS.sleep(100);
                System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",运行结束!");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release();
            }
            System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",当前可用许可数量:" + semaphore.availablePermits());
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T t1 = new T("t1");
        t1.start();
        //休眠1秒
        TimeUnit.SECONDS.sleep(1);
        T t2 = new T("t2");
        t2.start();
        //休眠1秒
        TimeUnit.SECONDS.sleep(1);
        T t3 = new T("t3");
        t3.start();

        //给t2和t3发送中断信号
        t2.interrupt();
        t3.interrupt();
    }
}
```

输出：

```java
1563717279058,t1,获取许可,当前可用许可数量:0
java.lang.InterruptedException
1563717281060,t2,当前可用许可数量:1
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:998)
1563717281060,t3,当前可用许可数量:2
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1304)
	at java.util.concurrent.Semaphore.acquire(Semaphore.java:312)
	at com.itsoku.chat12.Demo3$T.run(Demo3.java:21)
java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1302)
	at java.util.concurrent.Semaphore.acquire(Semaphore.java:312)
	at com.itsoku.chat12.Demo3$T.run(Demo3.java:21)
```

程序中信号量许可数量为1，创建了3个线程获取许可，线程t1获取成功了，然后休眠100秒。其他两个线程阻塞在`semaphore.acquire();`方法处，代码中对线程t2、t3发送中断信号，我们看一下Semaphore中acquire的源码：

```java
public void acquire() throws InterruptedException
```

这个方法会响应线程中断，主线程中对t2、t3发送中断信号之后，`acquire()`方法会触发`InterruptedException`异常，t2、t3最终没有获取到许可，但是他们都执行了finally中的释放许可的操作，最后导致许可数量变为了2，导致许可数量增加了。所以程序中释放许可的方式有问题。需要改进一下，获取许可成功才去释放锁。

**正确的释放锁的方式，如下：**

```java
package com.itsoku.chat12;

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo4 {
    static Semaphore semaphore = new Semaphore(1);

    public static class T extends Thread {
        public T(String name) {
            super(name);
        }

        @Override
        public void run() {
            Thread thread = Thread.currentThread();
            //获取许可是否成功
            boolean acquireSuccess = false;
            try {
                semaphore.acquire();
                acquireSuccess = true;
                System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",获取许可,当前可用许可数量:" + semaphore.availablePermits());
                //休眠100秒
                TimeUnit.SECONDS.sleep(5);
                System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",运行结束!");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                if (acquireSuccess) {
                    semaphore.release();
                }
            }
            System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",当前可用许可数量:" + semaphore.availablePermits());
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T t1 = new T("t1");
        t1.start();
        //休眠1秒
        TimeUnit.SECONDS.sleep(1);
        T t2 = new T("t2");
        t2.start();
        //休眠1秒
        TimeUnit.SECONDS.sleep(1);
        T t3 = new T("t3");
        t3.start();

        //给t2和t3发送中断信号
        t2.interrupt();
        t3.interrupt();
    }
}
```

输出：

```java
1563717751655,t1,获取许可,当前可用许可数量:0
1563717753657,t3,当前可用许可数量:0
java.lang.InterruptedException
1563717753657,t2,当前可用许可数量:0
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1302)
	at java.util.concurrent.Semaphore.acquire(Semaphore.java:312)
	at com.itsoku.chat12.Demo4$T.run(Demo4.java:23)
java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:998)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1304)
	at java.util.concurrent.Semaphore.acquire(Semaphore.java:312)
	at com.itsoku.chat12.Demo4$T.run(Demo4.java:23)
1563717756656,t1,运行结束!
1563717756656,t1,当前可用许可数量:1
```

程序中增加了一个变量`acquireSuccess`用来标记获取许可是否成功，在finally中根据这个变量是否为true，来确定是否释放许可。

## 在规定的时间内希望获取许可

司机来到停车场，发现停车场已经满了，只能在外等待内部的车出来之后才能进去，但是要等多久，他自己也不知道，他希望等10分钟，如果还是无法进去，就不到这里停车了。

Semaphore内部2个方法可以提供超时获取许可的功能：

```java
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException
public boolean tryAcquire(int permits, long timeout, TimeUnit unit)
        throws InterruptedException 
```

在指定的时间内去尝试获取许可，如果能够获取到，返回true，获取不到返回false。

**示例代码：**

```java
package com.itsoku.chat12;

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：路人甲Java，专注于java技术分享（带你玩转 爬虫、分布式事务、异步消息服务、任务调度、分库分表、大数据等），喜欢请关注！
 */
public class Demo5 {
    static Semaphore semaphore = new Semaphore(1);

    public static class T extends Thread {
        public T(String name) {
            super(name);
        }

        @Override
        public void run() {
            Thread thread = Thread.currentThread();
            //获取许可是否成功
            boolean acquireSuccess = false;
            try {
                //尝试在1秒内获取许可，获取成功返回true，否则返回false
                System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",尝试获取许可,当前可用许可数量:" + semaphore.availablePermits());
                acquireSuccess = semaphore.tryAcquire(1, TimeUnit.SECONDS);
                //获取成功执行业务代码
                if (acquireSuccess) {
                    System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",获取许可成功,当前可用许可数量:" + semaphore.availablePermits());
                    //休眠5秒
                    TimeUnit.SECONDS.sleep(5);
                } else {
                    System.out.println(System.currentTimeMillis() + "," + thread.getName() + ",获取许可失败,当前可用许可数量:" + semaphore.availablePermits());
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                if (acquireSuccess) {
                    semaphore.release();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T t1 = new T("t1");
        t1.start();
        //休眠1秒
        TimeUnit.SECONDS.sleep(1);
        T t2 = new T("t2");
        t2.start();
        //休眠1秒
        TimeUnit.SECONDS.sleep(1);
        T t3 = new T("t3");
        t3.start();
    }
}
```

输出：

```java
1563718410202,t1,尝试获取许可,当前可用许可数量:1
1563718410202,t1,获取许可成功,当前可用许可数量:0
1563718411203,t2,尝试获取许可,当前可用许可数量:0
1563718412203,t3,尝试获取许可,当前可用许可数量:0
1563718412204,t2,获取许可失败,当前可用许可数量:0
1563718413204,t3,获取许可失败,当前可用许可数量:0
```

代码中许可数量为1，`semaphore.tryAcquire(1, TimeUnit.SECONDS);`：表示尝试在1秒内获取许可，获取成功立即返回true，超过1秒还是获取不到，返回false。线程t1获取许可成功，之后休眠了5秒，从输出中可以看出t2和t3都尝试了1秒，获取失败。

## 其他一些使用说明

1. Semaphore默认创建的是非公平的信号量，什么意思呢？这个涉及到公平与非公平。举个例子：5个车位，允许5个车辆进去，来了100辆车，只能进去5辆，其他95在外面排队等着。里面刚好出来了1辆，此时刚好又来了10辆车，这10辆车是直接插队到其他95辆前面去，还是到95辆后面去排队呢？排队就表示公平，直接去插队争抢第一个，就表示不公平。对于停车场，排队肯定更好一些咯。不过对于信号量来说不公平的效率更高一些，所以默认是不公平的。
2. 建议阅读以下Semaphore的源码，对常用的方法有个了解，不需要都记住，用的时候也方便查询就好。
3. 方法中带有`throws InterruptedException`声明的，表示这个方法会响应线程中断信号，什么意思？表示调用线程的`interrupt()`方法，会让这些方法触发`InterruptedException`异常，即使这些方法处于阻塞状态，也会立即返回，并抛出`InterruptedException`异常，线程中断信号也会被清除。

## CountDownLatch（闭锁）

### 需求

假如有这样一个需求，当我们需要解析一个Excel里多个sheet的数据时，可以考虑使用多线程，每个线程解析一个sheet里的数据，等到所有的sheet都解析完之后，程序需要统计解析总耗时。分析一下：解析每个sheet耗时可能不一样，总耗时就是最长耗时的那个操作。

我们能够想到的最简单的做法是使用join，代码如下：

```java
package com.itsoku.chat13;

import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：javacode2018，获取年薪50万课程
 */
public class Demo1 {

    public static class T extends Thread {
        //休眠时间（秒）
        int sleepSeconds;

        public T(String name, int sleepSeconds) {
            super(name);
            this.sleepSeconds = sleepSeconds;
        }

        @Override
        public void run() {
            Thread ct = Thread.currentThread();
            long startTime = System.currentTimeMillis();
            System.out.println(startTime + "," + ct.getName() + ",开始处理!");
            try {
                //模拟耗时操作，休眠sleepSeconds秒
                TimeUnit.SECONDS.sleep(this.sleepSeconds);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            long endTime = System.currentTimeMillis();
            System.out.println(endTime + "," + ct.getName() + ",处理完毕,耗时:" + (endTime - startTime));
        }
    }

    public static void main(String[] args) throws InterruptedException {
        long starTime = System.currentTimeMillis();
        T t1 = new T("解析sheet1线程", 2);
        t1.start();

        T t2 = new T("解析sheet2线程", 5);
        t2.start();

        t1.join();
        t2.join();
        long endTime = System.currentTimeMillis();
        System.out.println("总耗时:" + (endTime - starTime));

    }
}
```

输出：

```txt
1563767560271,解析sheet1线程,开始处理!
1563767560272,解析sheet2线程,开始处理!
1563767562273,解析sheet1线程,处理完毕,耗时:2002
1563767565274,解析sheet2线程,处理完毕,耗时:5002
总耗时:5005
```

代码中启动了2个解析sheet的线程，第一个耗时2秒，第二个耗时5秒，最终结果中总耗时：5秒。上面的关键技术点是线程的`join()`方法，此方法会让当前线程等待被调用的线程完成之后才能继续。可以看一下join的源码，内部其实是在synchronized方法中调用了线程的wait方法，最后被调用的线程执行完毕之后，由jvm自动调用其notifyAll()方法，唤醒所有等待中的线程。这个notifyAll()方法是由jvm内部自动调用的，jdk源码中是看不到的，需要看jvm源码，有兴趣的同学可以去查一下。所以JDK不推荐在线程上调用wait、notify、notifyAll方法。

而在JDK1.5之后的并发包中提供的CountDownLatch也可以实现join的这个功能。

### 概述

**闭锁（Latch）**

一种同步方法，可以延迟线程的进度直到线程到达某个终点状态。通俗的讲就是，一个闭锁相当于一扇大门，在大门打开之前所有线程都被阻断，一旦大门打开所有线程都将通过，但是一旦大门打开，所有线程都通过了，那么这个闭锁的状态就失效了，门的状态也就不能变了，只能是打开状态。也就是说闭锁的状态是一次性的，它确保在闭锁打开之前所有特定的活动都需要在闭锁打开之后才能完成。
**应用场景**

确保某个计算在其需要的所有资源都被初始化之后才继续执行。二元闭锁（包括两个状态）可以用来表示“资源R已经被初始化”，而所有需要R的操作都必须先在这个闭锁上等待。
确保某个服务在其依赖的所有其他服务都已经启动之后才启动。
等待直到某个操作的所有参与者都就绪在继续执行。（例如：多人游戏中需要所有玩家准备才能开始）
CountDownLatch是JDK 5+里面闭锁的一个实现，允许一个或者多个线程等待某个事件的发生。

CountDownLatch称之为闭锁，它可以使一个或一批线程在闭锁上等待，等到其他线程执行完相应操作后，闭锁打开，这些等待的线程才可以继续执行。确切的说，闭锁在内部维护了一个倒计数器。通过该计数器的值来决定闭锁的状态，从而决定是否允许等待的线程继续执行。

### 常用方法：

**public CountDownLatch(int count)**：构造方法，count表示计数器的值，不能小于0，否者会报异常。

**public void await() throws InterruptedException**：调用await()会让当前线程等待，直到计数器为0的时候，方法才会返回，此方法会响应线程中断操作。

**public boolean await(long timeout, TimeUnit unit) throws InterruptedException**：限时等待，在超时之前，计数器变为了0，方法返回true，否者直到超时，返回false，此方法会响应线程中断操作。

**public void countDown()**：让计数器减1

CountDownLatch使用步骤：

1. 创建CountDownLatch对象
2. 调用其实例方法`await()`，让当前线程等待
3. 调用`countDown()`方法，让计数器减1
4. 当计数器变为0的时候，`await()`方法会返回

### CountDownLatch：使用示例

我们使用CountDownLatch来完成上面示例中使用join实现的功能，代码如下：

```java
package com.itsoku.chat13;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：javacode2018，获取年薪50万课程
 */
public class Demo2 {

    public static class T extends Thread {
        //休眠时间（秒）
        int sleepSeconds;
        CountDownLatch countDownLatch;

        public T(String name, int sleepSeconds, CountDownLatch countDownLatch) {
            super(name);
            this.sleepSeconds = sleepSeconds;
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            Thread ct = Thread.currentThread();
            long startTime = System.currentTimeMillis();
            System.out.println(startTime + "," + ct.getName() + ",开始处理!");
            try {
                //模拟耗时操作，休眠sleepSeconds秒
                TimeUnit.SECONDS.sleep(this.sleepSeconds);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                countDownLatch.countDown();
            }
            long endTime = System.currentTimeMillis();
            System.out.println(endTime + "," + ct.getName() + ",处理完毕,耗时:" + (endTime - startTime));
        }
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "线程 start!");
        CountDownLatch countDownLatch = new CountDownLatch(2);

        long starTime = System.currentTimeMillis();
        T t1 = new T("解析sheet1线程", 2, countDownLatch);
        t1.start();

        T t2 = new T("解析sheet2线程", 5, countDownLatch);
        t2.start();

        countDownLatch.await();
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "线程 end!");
        long endTime = System.currentTimeMillis();
        System.out.println("总耗时:" + (endTime - starTime));

    }
}
```

输出：

```java
1563767580511,main线程 start!
1563767580513,解析sheet1线程,开始处理!
1563767580513,解析sheet2线程,开始处理!
1563767582515,解析sheet1线程,处理完毕,耗时:2002
1563767585515,解析sheet2线程,处理完毕,耗时:5002
1563767585515,main线程 end!
总耗时:5003
```

从结果中看出，效果和join实现的效果一样，代码中创建了计数器为2的`CountDownLatch`，主线程中调用`countDownLatch.await();`会让主线程等待，t1、t2线程中模拟执行耗时操作，最终在finally中调用了`countDownLatch.countDown();`,此方法每调用一次，CountDownLatch内部计数器会减1，当计数器变为0的时候，主线程中的await()会返回，然后继续执行。注意：上面的`countDown()`这个是必须要执行的方法，所以放在finally中执行。

### CountDownLatch：等待指定的时间

还是上面的示例，2个线程解析2个sheet，主线程等待2个sheet解析完成。主线程说，我等待2秒，你们还是无法处理完成，就不等待了，直接返回。如下代码：

```java
package com.itsoku.chat13;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：javacode2018，获取年薪50万课程
 */
public class Demo3 {

    public static class T extends Thread {
        //休眠时间（秒）
        int sleepSeconds;
        CountDownLatch countDownLatch;

        public T(String name, int sleepSeconds, CountDownLatch countDownLatch) {
            super(name);
            this.sleepSeconds = sleepSeconds;
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            Thread ct = Thread.currentThread();
            long startTime = System.currentTimeMillis();
            System.out.println(startTime + "," + ct.getName() + ",开始处理!");
            try {
                //模拟耗时操作，休眠sleepSeconds秒
                TimeUnit.SECONDS.sleep(this.sleepSeconds);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                countDownLatch.countDown();
            }
            long endTime = System.currentTimeMillis();
            System.out.println(endTime + "," + ct.getName() + ",处理完毕,耗时:" + (endTime - startTime));
        }
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "线程 start!");
        CountDownLatch countDownLatch = new CountDownLatch(2);

        long starTime = System.currentTimeMillis();
        T t1 = new T("解析sheet1线程", 2, countDownLatch);
        t1.start();

        T t2 = new T("解析sheet2线程", 5, countDownLatch);
        t2.start();

        boolean result = countDownLatch.await(2, TimeUnit.SECONDS);

        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "线程 end!");
        long endTime = System.currentTimeMillis();
        System.out.println("主线程耗时:" + (endTime - starTime) + ",result:" + result);

    }
}
```

输出：

```java
1563767637316,main线程 start!
1563767637320,解析sheet1线程,开始处理!
1563767637320,解析sheet2线程,开始处理!
1563767639321,解析sheet1线程,处理完毕,耗时:2001
1563767639322,main线程 end!
主线程耗时:2004,result:false
1563767642322,解析sheet2线程,处理完毕,耗时:5002
```

从输出结果中可以看出，线程2耗时了5秒，主线程耗时了2秒，主线程中调用`countDownLatch.await(2, TimeUnit.SECONDS);`，表示最多等2秒，不管计数器是否为0，await方法都会返回，若等待时间内，计数器变为0了，立即返回true，否则超时后返回false。

### CountDown：结合使用示例

有3个人参见跑步比赛，需要先等指令员发指令枪后才能开跑，所有人都跑完之后，指令员喊一声，大家跑完了。

示例代码：

```java
package com.itsoku.chat13;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：javacode2018，获取年薪50万课程
 */
public class Demo4 {

    public static class T extends Thread {
        //跑步耗时（秒）
        int runCostSeconds;
        CountDownLatch commanderCd;
        CountDownLatch countDown;

        public T(String name, int runCostSeconds, CountDownLatch commanderCd, CountDownLatch countDown) {
            super(name);
            this.runCostSeconds = runCostSeconds;
            this.commanderCd = commanderCd;
            this.countDown = countDown;
        }

        @Override
        public void run() {
            //等待指令员枪响
            try {
                commanderCd.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Thread ct = Thread.currentThread();
            long startTime = System.currentTimeMillis();
            System.out.println(startTime + "," + ct.getName() + ",开始跑!");
            try {
                //模拟耗时操作，休眠runCostSeconds秒
                TimeUnit.SECONDS.sleep(this.runCostSeconds);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                countDown.countDown();
            }
            long endTime = System.currentTimeMillis();
            System.out.println(endTime + "," + ct.getName() + ",跑步结束,耗时:" + (endTime - startTime));
        }
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "线程 start!");
        CountDownLatch commanderCd = new CountDownLatch(1);
        CountDownLatch countDownLatch = new CountDownLatch(3);

        long starTime = System.currentTimeMillis();
        T t1 = new T("小张", 2, commanderCd, countDownLatch);
        t1.start();

        T t2 = new T("小李", 5, commanderCd, countDownLatch);
        t2.start();

        T t3 = new T("路人甲", 10, commanderCd, countDownLatch);
        t3.start();

        //主线程休眠5秒,模拟指令员准备发枪耗时操作
        TimeUnit.SECONDS.sleep(5);
        System.out.println(System.currentTimeMillis() + ",枪响了，大家开始跑");
        commanderCd.countDown();

        countDownLatch.await();
        long endTime = System.currentTimeMillis();
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "所有人跑完了，主线程耗时:" + (endTime - starTime));

    }
}
```

输出：

```txt
1563767691087,main线程 start!
1563767696092,枪响了，大家开始跑
1563767696092,小张,开始跑!
1563767696092,小李,开始跑!
1563767696092,路人甲,开始跑!
1563767698093,小张,跑步结束,耗时:2001
1563767701093,小李,跑步结束,耗时:5001
1563767706093,路人甲,跑步结束,耗时:10001
1563767706093,main所有人跑完了，主线程耗时:15004
```

代码中，t1、t2、t3启动之后，都阻塞在`commanderCd.await();`，主线程模拟发枪准备操作耗时5秒，然后调用`commanderCd.countDown();`模拟发枪操作，此方法被调用以后，阻塞在`commanderCd.await();`的3个线程会向下执行。主线程调用`countDownLatch.await();`之后进行等待，每个人跑完之后，调用`countDown.countDown();`通知一下`countDownLatch`让计数器减1，最后3个人都跑完了，主线程从`countDownLatch.await();`返回继续向下执行。

### 手写一个并行处理任务的工具类

```java
package com.itsoku.chat13;

import org.springframework.util.CollectionUtils;

import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.function.Consumer;
import java.util.stream.Collectors;
import java.util.stream.Stream;

/**
 * 微信公众号：javacode2018，获取年薪50万课程
 */
public class TaskDisposeUtils {
    //并行线程数
    public static final int POOL_SIZE;

    static {
        POOL_SIZE = Integer.max(Runtime.getRuntime().availableProcessors(), 5);
    }

    /**
     * 并行处理，并等待结束
     *
     * @param taskList 任务列表
     * @param consumer 消费者
     * @param <T>
     * @throws InterruptedException
     */
    public static <T> void dispose(List<T> taskList, Consumer<T> consumer) throws InterruptedException {
        dispose(true, POOL_SIZE, taskList, consumer);
    }

    /**
     * 并行处理，并等待结束
     *
     * @param moreThread 是否多线程执行
     * @param poolSize   线程池大小
     * @param taskList   任务列表
     * @param consumer   消费者
     * @param <T>
     * @throws InterruptedException
     */
    public static <T> void dispose(boolean moreThread, int poolSize, List<T> taskList, Consumer<T> consumer) throws InterruptedException {
        if (CollectionUtils.isEmpty(taskList)) {
            return;
        }
        if (moreThread && poolSize > 1) {
            poolSize = Math.min(poolSize, taskList.size());
            ExecutorService executorService = null;
            try {
                executorService = Executors.newFixedThreadPool(poolSize);
                CountDownLatch countDownLatch = new CountDownLatch(taskList.size());
                for (T item : taskList) {
                    executorService.execute(() -> {
                        try {
                            consumer.accept(item);
                        } finally {
                            countDownLatch.countDown();
                        }
                    });
                }
                countDownLatch.await();
            } finally {
                if (executorService != null) {
                    executorService.shutdown();
                }
            }
        } else {
            for (T item : taskList) {
                consumer.accept(item);
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        //生成1-10的10个数字，放在list中，相当于10个任务
        List<Integer> list = Stream.iterate(1, a -> a + 1).limit(10).collect(Collectors.toList());
        //启动多线程处理list中的数据，每个任务休眠时间为list中的数值
        TaskDisposeUtils.dispose(list, item -> {
            try {
                long startTime = System.currentTimeMillis();
                TimeUnit.SECONDS.sleep(item);
                long endTime = System.currentTimeMillis();

                System.out.println(System.currentTimeMillis() + ",任务" + item + "执行完毕，耗时:" + (endTime - startTime));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        //上面所有任务处理完毕完毕之后，程序才能继续
        System.out.println(list + "中的任务都处理完毕!");
    }
}
```

运行代码输出：

```txt
1563769828130,任务1执行完毕，耗时:1000
1563769829130,任务2执行完毕，耗时:2000
1563769830131,任务3执行完毕，耗时:3001
1563769831131,任务4执行完毕，耗时:4001
1563769832131,任务5执行完毕，耗时:5001
1563769833130,任务6执行完毕，耗时:6000
1563769834131,任务7执行完毕，耗时:7001
1563769835131,任务8执行完毕，耗时:8001
1563769837131,任务9执行完毕，耗时:9001
1563769839131,任务10执行完毕，耗时:10001
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]中的任务都处理完毕!
```

**TaskDisposeUtils是一个并行处理的工具类，可以传入n个任务内部使用线程池进行处理，等待所有任务都处理完成之后，方法才会返回。比如我们发送短信，系统中有1万条短信，我们使用上面的工具，每次取100条并行发送，待100个都处理完毕之后，再取一批按照同样的逻辑发送。**

# CyclicBarrier（循环栅栏）

## CyclicBarrier简介

CyclicBarrier通常称为循环屏障。它和CountDownLatch很相似，都可以使线程先等待然后再执行。不过CountDownLatch是使一批线程等待另一批线程执行完后再执行；而CyclicBarrier只是使等待的线程达到一定数目后再让它们继续执行。故而CyclicBarrier内部也有一个计数器,计数器的初始值在创建对象时通过构造参数指定,如下所示：

```java
public CyclicBarrier(int parties) {
    this(parties, null);
}
```

每调用一次await()方法都将使阻塞的线程数+1，只有阻塞的线程数达到设定值时屏障才会打开，允许阻塞的所有线程继续执行。除此之外，CyclicBarrier还有几点需要注意的地方:

- CyclicBarrier的计数器可以重置而CountDownLatch不行，这意味着CyclicBarrier实例可以被重复使用而CountDownLatch只能被使用一次。而这也是循环屏障循环二字的语义所在。
- CyclicBarrier允许用户自定义barrierAction操作，这是个可选操作，可以在创建CyclicBarrier对象时指定

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}
```

一旦用户在创建CyclicBarrier对象时设置了barrierAction参数，则在阻塞线程数达到设定值屏障打开前，会调用barrierAction的run()方法完成用户自定义的操作。

## CyclicBarrier：简单使用

公司组织旅游，大家都有经历过，10个人，中午到饭点了，需要等到10个人都到了才能开饭，先到的人坐那等着，代码如下：

```java
package com.itsoku.chat15;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：javacode2018，获取年薪50万java课程
 */
public class Demo1 {
    public static CyclicBarrier cyclicBarrier = new CyclicBarrier(10);

    public static class T extends Thread {
        int sleep;

        public T(String name, int sleep) {
            super(name);
            this.sleep = sleep;
        }

        @Override
        public void run() {
            try {
                //模拟休眠
                TimeUnit.SECONDS.sleep(sleep);
                long starTime = System.currentTimeMillis();
                //调用await()的时候，当前线程将会被阻塞，需要等待其他员工都到达await了才能继续
                cyclicBarrier.await();
                long endTime = System.currentTimeMillis();
                System.out.println(this.getName() + ",sleep:" + this.sleep + " 等待了" + (endTime - starTime) + "(ms),开始吃饭了！");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 1; i <= 10; i++) {
            new T("员工" + i, i).start();
        }
    }
}
```

输出：

```java
员工1,sleep:1 等待了9000(ms),开始吃饭了！
员工9,sleep:9 等待了1000(ms),开始吃饭了！
员工8,sleep:8 等待了2001(ms),开始吃饭了！
员工7,sleep:7 等待了3001(ms),开始吃饭了！
员工6,sleep:6 等待了4001(ms),开始吃饭了！
员工4,sleep:4 等待了6000(ms),开始吃饭了！
员工5,sleep:5 等待了5000(ms),开始吃饭了！
员工10,sleep:10 等待了0(ms),开始吃饭了！
员工2,sleep:2 等待了7999(ms),开始吃饭了！
员工3,sleep:3 等待了7000(ms),开始吃饭了！
```

代码中模拟了10个员工上桌吃饭的场景，等待所有员工都到齐了才能开发，可以看到第10个员工最慢，前面的都在等待第10个员工，员工1等待了9秒，上面代码中调用`cyclicBarrier.await();`会让当前线程等待。当10个员工都调用了`cyclicBarrier.await();`之后，所有处于等待中的员工都会被唤醒，然后继续运行。

## 示例2：重复使用CyclicBarrier

对示例1进行改造一下，吃饭完毕之后，所有人都去车上，待所有人都到车上之后，驱车去下一景点玩。

```java
package com.itsoku.chat15;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：javacode2018，获取年薪50万java课程
 */
public class Demo2 {
    public static CyclicBarrier cyclicBarrier = new CyclicBarrier(10);

    public static class T extends Thread {
        int sleep;

        public T(String name, int sleep) {
            super(name);
            this.sleep = sleep;
        }

        //等待吃饭
        void eat() {
            try {
                //模拟休眠
                TimeUnit.SECONDS.sleep(sleep);
                long starTime = System.currentTimeMillis();
                //调用await()的时候，当前线程将会被阻塞，需要等待其他员工都到达await了才能继续
                cyclicBarrier.await();
                long endTime = System.currentTimeMillis();
                System.out.println(this.getName() + ",sleep:" + this.sleep + " 等待了" + (endTime - starTime) + "(ms),开始吃饭了！");

                //休眠sleep时间，模拟当前员工吃饭耗时
                TimeUnit.SECONDS.sleep(sleep);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }

        //等待所有人到齐之后，开车去下一站
        void drive() {
            try {
                long starTime = System.currentTimeMillis();
                //调用await()的时候，当前线程将会被阻塞，需要等待其他员工都到达await了才能继续
                cyclicBarrier.await();
                long endTime = System.currentTimeMillis();
                System.out.println(this.getName() + ",sleep:" + this.sleep + " 等待了" + (endTime - starTime) + "(ms),去下一景点的路上！");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void run() {
            //等待所有人到齐之后吃饭，先到的人坐那等着，什么事情不要干
            this.eat();
            //等待所有人到齐之后开车去下一景点，先到的人坐那等着，什么事情不要干
            this.drive();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 1; i <= 10; i++) {
            new T("员工" + i, i).start();
        }
    }
}
```

输出：

```makefile
员工10,sleep:10 等待了0(ms),开始吃饭了！
员工5,sleep:5 等待了5000(ms),开始吃饭了！
员工6,sleep:6 等待了4000(ms),开始吃饭了！
员工9,sleep:9 等待了1001(ms),开始吃饭了！
员工4,sleep:4 等待了6000(ms),开始吃饭了！
员工3,sleep:3 等待了7000(ms),开始吃饭了！
员工1,sleep:1 等待了9001(ms),开始吃饭了！
员工2,sleep:2 等待了8000(ms),开始吃饭了！
员工8,sleep:8 等待了2001(ms),开始吃饭了！
员工7,sleep:7 等待了3000(ms),开始吃饭了！
员工10,sleep:10 等待了0(ms),去下一景点的路上！
员工1,sleep:1 等待了8998(ms),去下一景点的路上！
员工5,sleep:5 等待了4999(ms),去下一景点的路上！
员工4,sleep:4 等待了5999(ms),去下一景点的路上！
员工3,sleep:3 等待了6998(ms),去下一景点的路上！
员工2,sleep:2 等待了7998(ms),去下一景点的路上！
员工9,sleep:9 等待了999(ms),去下一景点的路上！
员工8,sleep:8 等待了1999(ms),去下一景点的路上！
员工7,sleep:7 等待了2999(ms),去下一景点的路上！
员工6,sleep:6 等待了3999(ms),去下一景点的路上！
```

坑，又是员工10最慢，要提升效率了，不能吃的太多，得减肥。

代码中CyclicBarrier相当于使用了2次，第一次用于等待所有人到达后开饭，第二次用于等待所有人上车后驱车去下一景点。注意一些先到的员工会在其他人到达之前，都处于等待状态（`cyclicBarrier.await();`会让当前线程阻塞），无法干其他事情，等到最后一个人到了会唤醒所有人，然后继续。

> CyclicBarrier内部相当于有个计数器（构造方法传入的），每次调用`await();`后，计数器会减1，并且await()方法会让当前线程阻塞，等待计数器减为0的时候，所有在await()上等待的线程被唤醒，然后继续向下执行，此时计数器又会被还原为创建时的值，然后可以继续再次使用。

## 示例3：最后到的人给大家上酒，然后开饭

还是示例1中的例子，员工10是最后到达的，让所有人都久等了，那怎么办，得给所有人倒酒，然后开饭，代码如下：

```java
package com.itsoku.chat15;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：javacode2018，获取年薪50万java课程
 */
public class Demo3 {
    public static CyclicBarrier cyclicBarrier = new CyclicBarrier(10, () -> {
        //模拟倒酒，花了2秒，又得让其他9个人等2秒
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "说，不好意思，让大家久等了，给大家倒酒赔罪!");
    });

    public static class T extends Thread {
        int sleep;

        public T(String name, int sleep) {
            super(name);
            this.sleep = sleep;
        }

        @Override
        public void run() {
            try {
                //模拟休眠
                TimeUnit.SECONDS.sleep(sleep);
                long starTime = System.currentTimeMillis();
                //调用await()的时候，当前线程将会被阻塞，需要等待其他员工都到达await了才能继续
                cyclicBarrier.await();
                long endTime = System.currentTimeMillis();
                System.out.println(this.getName() + ",sleep:" + this.sleep + " 等待了" + (endTime - starTime) + "(ms),开始吃饭了！");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 1; i <= 10; i++) {
            new T("员工" + i, i).start();
        }
    }
}
```

输出：

```java
员工10说，不好意思，让大家久等了，给大家倒酒赔罪!
员工10,sleep:10 等待了2000(ms),开始吃饭了！
员工1,sleep:1 等待了11000(ms),开始吃饭了！
员工2,sleep:2 等待了10000(ms),开始吃饭了！
员工5,sleep:5 等待了7000(ms),开始吃饭了！
员工7,sleep:7 等待了5000(ms),开始吃饭了！
员工9,sleep:9 等待了3000(ms),开始吃饭了！
员工4,sleep:4 等待了8000(ms),开始吃饭了！
员工3,sleep:3 等待了9001(ms),开始吃饭了！
员工8,sleep:8 等待了4001(ms),开始吃饭了！
员工6,sleep:6 等待了6001(ms),开始吃饭了！
```

代码中创建`CyclicBarrier`对象时，多传入了一个参数（内部是倒酒操作），先到的人先等待，待所有人都到齐之后，需要先给大家倒酒，然后唤醒所有等待中的人让大家开饭。从输出结果中我们发现，倒酒操作是由最后一个人操作的，最后一个人倒酒完毕之后，才唤醒所有等待中的其他员工，让大家开饭。

## 示例4：其中一个人等待中被打断了

员工5等待中，突然接了个电话，有点急事，然后就拿起筷子开吃了，其他人会怎么样呢？看着他吃么？

代码如下：

```java
package com.itsoku.chat15;

import java.sql.Time;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;

/**
 * 微信公众号：javacode2018，获取年薪50万java课程
 */
public class Demo4 {
    public static CyclicBarrier cyclicBarrier = new CyclicBarrier(10);

    public static class T extends Thread {
        int sleep;

        public T(String name, int sleep) {
            super(name);
            this.sleep = sleep;
        }

        @Override
        public void run() {
            long starTime = 0, endTime = 0;
            try {
                //模拟休眠
                TimeUnit.SECONDS.sleep(sleep);
                starTime = System.currentTimeMillis();
                //调用await()的时候，当前线程将会被阻塞，需要等待其他员工都到达await了才能继续
                System.out.println(this.getName() + "到了！");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            endTime = System.currentTimeMillis();
            System.out.println(this.getName() + ",sleep:" + this.sleep + " 等待了" + (endTime - starTime) + "(ms),开始吃饭了！");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 1; i <= 10; i++) {
            int sleep = 0;
            if (i == 10) {
                sleep = 10;
            }
            T t = new T("员工" + i, sleep);
            t.start();
            if (i == 5) {
                //模拟员工5接了个电话，将自己等待吃饭给打断了
                TimeUnit.SECONDS.sleep(1);
                System.out.println(t.getName() + ",有点急事，我先开干了！");
                t.interrupt();
                TimeUnit.SECONDS.sleep(2);
            }
        }
    }
}
```

输出：

```java
员工4到了！
员工3到了！
员工5到了！
员工1到了！
员工2到了！
员工5,有点急事，我先开干了！
java.util.concurrent.BrokenBarrierException
员工1,sleep:0 等待了1001(ms),开始吃饭了！
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
员工3,sleep:0 等待了1001(ms),开始吃饭了！
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
员工4,sleep:0 等待了1001(ms),开始吃饭了！
    at com.itsoku.chat15.Demo4$T.run(Demo4.java:31)
员工2,sleep:0 等待了1001(ms),开始吃饭了！
员工5,sleep:0 等待了1002(ms),开始吃饭了！
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo4$T.run(Demo4.java:31)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo4$T.run(Demo4.java:31)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo4$T.run(Demo4.java:31)
java.lang.InterruptedException
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.reportInterruptAfterWait(AbstractQueuedSynchronizer.java:2014)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2048)
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:234)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo4$T.run(Demo4.java:31)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo4$T.run(Demo4.java:31)
java.util.concurrent.BrokenBarrierException
员工6到了！
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
员工9到了！
    at com.itsoku.chat15.Demo4$T.run(Demo4.java:31)
员工8到了！
员工7到了！
员工6,sleep:0 等待了0(ms),开始吃饭了！
员工7,sleep:0 等待了1(ms),开始吃饭了！
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo4$T.run(Demo4.java:31)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo4$T.run(Demo4.java:31)
员工8,sleep:0 等待了1(ms),开始吃饭了！
员工9,sleep:0 等待了1(ms),开始吃饭了！
Disconnected from the target VM, address: '127.0.0.1:64413', transport: 'socket'
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo4$T.run(Demo4.java:31)
员工10到了！
员工10,sleep:10 等待了0(ms),开始吃饭了！
```

输出的信息看着有点乱，给大家理一理，员工5遇到急事，拿起筷子就是吃，这样好么，当然不好，他这么做了，后面看他这么做了都跟着这么做（这种场景是不是很熟悉，有一个人拿起筷子先吃起来，其他人都跟着上了），直接不等其他人了，拿起筷子就开吃了。CyclicBarrier遇到这种情况就是这么处理的。前面4个员工都在`await()`处等待着，员工5也在`await()`上等待着，等了1秒（`TimeUnit.SECONDS.sleep(1);`），接了个电话，然后给员工5发送中断信号后（`t.interrupt();`），员工5的await()方法会触发`InterruptedException`异常，此时其他等待中的前4个员工，看着5开吃了，自己立即也不等了，内部从`await()`方法中触发`BrokenBarrierException`异常，然后也开吃了，后面的6/7/8/9/10员工来了以后发现大家都开吃了，自己也不等了，6-10员工调用`await()`直接抛出了`BrokenBarrierException`异常，然后继续向下。

**结论：**

1. **内部有一个人把规则破坏了（接收到中断信号），其他人都不按规则来了，不会等待了**
2. **接收到中断信号的线程，await方法会触发InterruptedException异常，然后被唤醒向下运行**
3. **其他等待中 或者后面到达的线程，会在await()方法上触发`BrokenBarrierException`异常，然后继续执行**

## 示例5：其中一个人只愿意等待5秒

基于示例1，员工1只愿意等的5秒，5s后如果大家还没到期，自己要开吃了，员工1开吃了，其他人会怎么样呢？

```java
package com.itsoku.chat15;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

/**
 * 微信公众号：javacode2018，获取年薪50万java课程
 */
public class Demo5 {
    public static CyclicBarrier cyclicBarrier = new CyclicBarrier(10);

    public static class T extends Thread {
        int sleep;

        public T(String name, int sleep) {
            super(name);
            this.sleep = sleep;
        }

        @Override
        public void run() {
            long starTime = 0, endTime = 0;
            try {
                //模拟休眠
                TimeUnit.SECONDS.sleep(sleep);
                starTime = System.currentTimeMillis();
                //调用await()的时候，当前线程将会被阻塞，需要等待其他员工都到达await了才能继续
                System.out.println(this.getName() + "到了！");
                if (this.getName().equals("员工1")) {
                    cyclicBarrier.await(5, TimeUnit.SECONDS);
                } else {
                    cyclicBarrier.await();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            } catch (TimeoutException e) {
                e.printStackTrace();
            }
            endTime = System.currentTimeMillis();
            System.out.println(this.getName() + ",sleep:" + this.sleep + " 等待了" + (endTime - starTime) + "(ms),开始吃饭了！");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 1; i <= 10; i++) {
            T t = new T("员工" + i, i);
            t.start();
        }
    }
}
```

输出：

```java
员工1到了！
员工2到了！
员工3到了！
员工4到了！
员工5到了！
员工6到了！
员工1,sleep:1 等待了5001(ms),开始吃饭了！
员工5,sleep:5 等待了1001(ms),开始吃饭了！
java.util.concurrent.TimeoutException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:257)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:435)
    at com.itsoku.chat15.Demo5$T.run(Demo5.java:32)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo5$T.run(Demo5.java:34)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo5$T.run(Demo5.java:34)
员工6,sleep:6 等待了2(ms),开始吃饭了！
java.util.concurrent.BrokenBarrierException
员工2,sleep:2 等待了4002(ms),开始吃饭了！
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
员工3,sleep:3 等待了3001(ms),开始吃饭了！
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
员工4,sleep:4 等待了2001(ms),开始吃饭了！
    at com.itsoku.chat15.Demo5$T.run(Demo5.java:34)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo5$T.run(Demo5.java:34)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo5$T.run(Demo5.java:34)
java.util.concurrent.BrokenBarrierException
员工7到了！
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
员工7,sleep:7 等待了0(ms),开始吃饭了！
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo5$T.run(Demo5.java:34)
员工8到了！
员工8,sleep:8 等待了0(ms),开始吃饭了！
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo5$T.run(Demo5.java:34)
员工9到了！
java.util.concurrent.BrokenBarrierException
员工9,sleep:9 等待了0(ms),开始吃饭了！
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo5$T.run(Demo5.java:34)
java.util.concurrent.BrokenBarrierException
员工10到了！
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
员工10,sleep:10 等待了0(ms),开始吃饭了！
    at com.itsoku.chat15.Demo5$T.run(Demo5.java:34)
```

从输出结果中我们可以看到：1等待5秒之后，开吃了，其他等待人都开吃了，后面来的人不等待，直接开吃了。

员工1调用有参`await`方法等待5秒之后，触发了`TimeoutException`异常，然后继续向下运行，其他的在5开吃之前已经等了一会的的几个员工，他们看到5开吃了，自己立即不等待了，也开吃了（他们的`await`抛出了`BrokenBarrierException`异常）；还有几个员工在5开吃之后到达的，他们直接不等待了，直接抛出`BrokenBarrierException`异常，然后也开吃了。

**结论：**

1. 等待超时的方法

   ```java
   public int await(long timeout, TimeUnit unit) throws InterruptedException,BrokenBarrierException,TimeoutException
   ```

2. **内部有一个人把规则破坏了（等待超时），其他人都不按规则来了，不会等待了**

3. **等待超时的线程，await方法会触发TimeoutException异常，然后被唤醒向下运行**

4. **其他等待中或者后面到达的线程，会在await()方法上触发`BrokenBarrierException`异常，然后继续执行**

## 示例6：重建规则

示例5中改造一下，员工1等待5秒超时之后，开吃了，打破了规则，先前等待中的以及后面到达的都不按规则来了，都拿起筷子开吃。过了一会，导游重新告知大家，要按规则来，然后重建了规则，大家都按规则来了。

代码如下：

```java
package com.itsoku.chat15;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

/**
 * 微信公众号：javacode2018，获取年薪50万java课程
 */
public class Demo6 {
    public static CyclicBarrier cyclicBarrier = new CyclicBarrier(10);

    //规则是否已重建
    public static boolean guizhe = false;

    public static class T extends Thread {
        int sleep;

        public T(String name, int sleep) {
            super(name);
            this.sleep = sleep;
        }

        @Override
        public void run() {
            long starTime = 0, endTime = 0;
            try {
                //模拟休眠
                TimeUnit.SECONDS.sleep(sleep);
                starTime = System.currentTimeMillis();
                //调用await()的时候，当前线程将会被阻塞，需要等待其他员工都到达await了才能继续
                System.out.println(this.getName() + "到了！");
                if (!guizhe) {
                    if (this.getName().equals("员工1")) {
                        cyclicBarrier.await(5, TimeUnit.SECONDS);
                    } else {
                        cyclicBarrier.await();
                    }
                } else {
                    cyclicBarrier.await();

                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            } catch (TimeoutException e) {
                e.printStackTrace();
            }
            endTime = System.currentTimeMillis();
            System.out.println(this.getName() + ",sleep:" + this.sleep + " 等待了" + (endTime - starTime) + "(ms),开始吃饭了！");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 1; i <= 10; i++) {
            T t = new T("员工" + i, i);
            t.start();
        }

        //等待10秒之后，重置，重建规则
        TimeUnit.SECONDS.sleep(15);
        cyclicBarrier.reset();
        guizhe = true;
        System.out.println("---------------大家太皮了，请大家按规则来------------------");
        //再来一次
        for (int i = 1; i <= 10; i++) {
            T t = new T("员工" + i, i);
            t.start();
        }
    }
}
```

输出：

```java
员工1到了！
员工2到了！
员工3到了！
员工4到了！
员工5到了！
java.util.concurrent.TimeoutException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:257)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:435)
    at com.itsoku.chat15.Demo6$T.run(Demo6.java:36)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo6$T.run(Demo6.java:38)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo6$T.run(Demo6.java:38)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo6$T.run(Demo6.java:38)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo6$T.run(Demo6.java:38)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo6$T.run(Demo6.java:38)
员工6到了！
员工1,sleep:1 等待了5002(ms),开始吃饭了！
员工6,sleep:6 等待了4(ms),开始吃饭了！
员工4,sleep:4 等待了2004(ms),开始吃饭了！
员工5,sleep:5 等待了1004(ms),开始吃饭了！
员工3,sleep:3 等待了3002(ms),开始吃饭了！
员工2,sleep:2 等待了4004(ms),开始吃饭了！
员工7到了！
员工7,sleep:7 等待了0(ms),开始吃饭了！
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo6$T.run(Demo6.java:38)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo6$T.run(Demo6.java:38)
员工8到了！
员工8,sleep:8 等待了0(ms),开始吃饭了！
java.util.concurrent.BrokenBarrierException
员工9到了！
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
员工9,sleep:9 等待了0(ms),开始吃饭了！
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo6$T.run(Demo6.java:38)
java.util.concurrent.BrokenBarrierException
员工10到了！
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
员工10,sleep:10 等待了0(ms),开始吃饭了！
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at com.itsoku.chat15.Demo6$T.run(Demo6.java:38)
---------------大家太皮了，请大家按规则来------------------
员工1到了！
员工2到了！
员工3到了！
员工4到了！
员工5到了！
员工6到了！
员工7到了！
员工8到了！
员工9到了！
员工10到了！
员工10,sleep:10 等待了0(ms),开始吃饭了！
员工1,sleep:1 等待了9000(ms),开始吃饭了！
员工2,sleep:2 等待了8000(ms),开始吃饭了！
员工3,sleep:3 等待了6999(ms),开始吃饭了！
员工7,sleep:7 等待了3000(ms),开始吃饭了！
员工6,sleep:6 等待了4000(ms),开始吃饭了！
员工5,sleep:5 等待了5000(ms),开始吃饭了！
员工4,sleep:4 等待了6000(ms),开始吃饭了！
员工9,sleep:9 等待了999(ms),开始吃饭了！
员工8,sleep:8 等待了1999(ms),开始吃饭了！
```

第一次规则被打乱了，过了一会导游重建了规则（`cyclicBarrier.reset();`），接着又重来来了一次模拟等待吃饭的操作，正常了。

## CountDownLatch和CyclicBarrier的区别

还是举例子说明一下：

**CountDownLatch示例**

主管相当于 **CountDownLatch**，干活的小弟相当于做事情的线程。

老板交给主管了一个任务，让主管搞完之后立即上报给老板。主管下面有10个小弟，接到任务之后将任务划分为10个小任务分给每个小弟去干，主管一直处于等待状态（主管会调用`await()`方法，此方法会阻塞当前线程），让每个小弟干完之后通知一下主管（调用`countDown()`方法通知主管，此方法会立即返回），主管等到所有的小弟都做完了，会被唤醒，从await()方法上苏醒，然后将结果反馈给老板。期间主管会等待，会等待所有小弟将结果汇报给自己。

**而CyclicBarrier是一批线程让自己等待，等待所有的线程都准备好了，自己才能继续。**

# ThreadPoolExecutor（线程池）

## 什么是线程池

大家用jdbc操作过数据库应该知道，操作数据库需要和数据库建立连接，拿到连接之后才能操作数据库，用完之后销毁。数据库连接的创建和销毁其实是比较耗时的，真正和业务相关的操作耗时是比较短的。每个数据库操作之前都需要创建连接，为了提升系统性能，后来出现了数据库连接池，系统启动的时候，先创建很多连接放在池子里面，使用的时候，直接从连接池中获取一个，使用完毕之后返回到池子里面，继续给其他需要者使用，这其中就省去创建连接的时间，从而提升了系统整体的性能。

线程池和数据库连接池的原理也差不多，创建线程去处理业务，可能创建线程的时间比处理业务的时间还长一些，如果系统能够提前为我们创建好线程，我们需要的时候直接拿来使用，用完之后不是直接将其关闭，而是将其返回到线程中，给其他需要这使用，这样直接节省了创建和销毁的时间，提升了系统的性能。

简单的说，在使用了线程池之后，创建线程变成了从线程池中获取一个空闲的线程，然后使用，关闭线程变成了将线程归还到线程池。

## 为什么要有线程池

线程池能够对线程进行统一分配，调优和监控:

- 降低资源消耗(线程无限制地创建，然后使用完毕后销毁)
- 提高响应速度(无须创建线程)
- 提高线程的可管理性

## 线程池实现原理

当向线程池提交一个任务之后，线程池的处理流程如下：

1. 判断是否达到核心线程数，若未达到，则直接创建新的线程处理当前传入的任务，否则进入下个流程
2. 线程池中的工作队列是否已满，若未满，则将任务丢入工作队列中先存着等待处理，否则进入下个流程
3. 是否达到最大线程数，若未达到，则创建新的线程处理当前传入的任务，否则交给线程池中的饱和策略进行处理。

流程如下图：

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190729085957398-1576485684.png)

**举个例子，加深理解：**

咱们作为开发者，上面都有开发主管，主管下面带领几个小弟干活，CTO给主管授权说，你可以招聘5个小弟干活，新来任务，如果小弟还不到5个，立即去招聘一个来干这个新来的任务，当5个小弟都招来了，再来任务之后，将任务记录到一个表格中，表格中最多记录100个，小弟们会主动去表格中获取任务执行，如果5个小弟都在干活，并且表格中也记录满了，那你可以将小弟扩充到20个，如果20个小弟都在干活，并且存放任务的表也满了，产品经理再来任务后，是直接拒绝，还是让产品自己干，这个由你自己决定，小弟们都尽心尽力在干活，任务都被处理完了，突然公司业绩下滑，几个员工没事干，打酱油，为了节约成本，CTO主管把小弟控制到5人，其他15个人直接被干掉了。所以作为小弟们，别让自己闲着，多干活。

**原理：**先找几个人干活，大家都忙于干活，任务太多可以排期，排期的任务太多了，再招一些人来干活，最后干活的和排期都达到上层领导要求的上限了，那需要采取一些其他策略进行处理了。对于长时间不干活的人，考虑将其开掉，节约资源和成本。

## java中的线程池

jdk中提供了线程池的具体实现，实现类是：`java.util.concurrent.ThreadPoolExecutor`，主要构造方法：

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

**corePoolSize**：线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize, 即使有其他空闲线程能够执行新来的任务, 也会继续创建线程；等到工作的线程数大于核心线程数时就不会在创建了。如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果执行了线程池的**prestartAllCoreThreads()**方法，线程池会提前创建并启动所有核心线程。

**maximumPoolSize**：线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；当阻塞队列是无界队列, 则maximumPoolSize则不起作用, 因为无法提交至核心线程池的线程会一直持续地放入workQueue。

**keepAliveTime**：线程空闲时的存活时间，即当线程没有任务执行时，该线程继续存活的时间；默认情况下，该参数只在线程数大于corePoolSize时才有用, 超过这个时间的空闲线程将被终止。如果没有任务处理了，有些线程会空闲，空闲的时间超过了这个值，会被回收掉。如果任务很多，并且每个任务的执行时间比较短，避免线程重复创建和回收，可以调大这个时间，提高线程的利用率

unit：keepAliveTIme的时间单位，可以选择的单位有天、小时、分钟、毫秒、微妙、千分之一毫秒和纳秒。类型是一个枚举**java.util.concurrent.TimeUnit**，这个枚举也经常使用，有兴趣的可以看一下其源码

**workQueue**：工作队列，用于缓存待处理任务的阻塞队列。在JDK中提供了如下阻塞队列:

- ArrayBlockingQueue: 基于数组结构的有界阻塞队列，按FIFO排序任务；

- LinkedBlockingQuene: 基于链表结构的阻塞队列，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQuene；

- SynchronousQuene: 一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene；

- PriorityBlockingQuene: 具有优先级的无界阻塞队列

LinkedBlockingQueue比ArrayBlockingQueue在插入删除节点性能方面更优，但是二者在put(), take()任务的时均需要加锁，SynchronousQueue使用无锁算法，根据节点的状态判断执行，而不需要用到锁，其核心是Transfer.transfer()

**threadFactory**：创建线程的工厂，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。默认为DefaultThreadFactory

**handler**：线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了4种策略

- AbortPolicy: 直接抛出异常，默认策略；
- CallerRunsPolicy: 用调用者所在的线程来执行任务；
- DiscardOldestPolicy: 丢弃阻塞队列中靠最前的任务，并执行当前任务；
- DiscardPolicy: 直接丢弃任务；

当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务。



**调用线程池的execute方法处理任务，执行execute方法的过程：**

1. 判断线程池中运行的线程数是否小于corepoolsize，是：则创建新的线程来处理任务，否：执行下一步
2. 试图将任务添加到workQueue指定的队列中，如果无法添加到队列，进入下一步
3. 判断线程池中运行的线程数是否小于`maximumPoolSize`，是：则新增线程处理当前传入的任务，否：将任务传递给`handler`对象`rejectedExecution`方法处理

**线程池的使用步骤：**

1. 调用构造方法创建线程池
2. 调用线程池的方法处理任务
3. 关闭线程池

## 四种策略、五种状态、六个核心参数、七种阻塞队列

三种阻塞队列：

```
 BlockingQueue<Runnable> workQueue = null;
 workQueue = new ArrayBlockingQueue<>(5);//基于数组的先进先出队列，有界
 workQueue = new LinkedBlockingQueue<>();//基于链表的先进先出队列，无界
 workQueue = new SynchronousQueue<>();//无缓冲的等待队列，无界
 workQueue = new SynchronousQueue<>();//无缓冲的等待队列，无界
 workQueue = new SynchronousQueue<>();//无缓冲的等待队列，无界
 workQueue = new SynchronousQueue<>();//无缓冲的等待队列，无界
 workQueue = new SynchronousQueue<>();//无缓冲的等待队列，无界
```


四种拒绝策略：

```
  RejectedExecutionHandler rejected = null;
  rejected = new ThreadPoolExecutor.AbortPolicy();//默认，队列满了丢任务抛出异常
  rejected = new ThreadPoolExecutor.DiscardPolicy();//队列满了丢任务不异常
  rejected = new ThreadPoolExecutor.DiscardOldestPolicy();//将最早进入队列的任务删，之后再尝试加入队列
  rejected = new ThreadPoolExecutor.CallerRunsPolicy();//如果添加到线程池失败，那么主线程会自己去执行该任务
```

五种线程池：

```
  ExecutorService threadPool = null;
  threadPool = Executors.newCachedThreadPool();//有缓冲的线程池，线程数 JVM 控制
  threadPool = Executors.newFixedThreadPool(3);//固定大小的线程池
  threadPool = Executors.newScheduledThreadPool(2);
  threadPool = Executors.newSingleThreadExecutor();//单线程的线程池，只有一个线程在工作
  threadPool = new ThreadPoolExecutor();//默认线程池，可控制参数比较多  
```

## ThreadPoolExecutor：简单示例

```java
package com.concurrency.threadpoolexecutor;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class ThreadPoolExecutorDemo {

    static ThreadPoolExecutor executor = new ThreadPoolExecutor(3,
            5,
            10,
            TimeUnit.SECONDS,
            new ArrayBlockingQueue<Runnable>(10),
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.AbortPolicy());

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            int j = i;
            String taskName = "任务" + j;
            executor.execute(() -> {
                //模拟任务内部处理耗时
                try {
                    TimeUnit.SECONDS.sleep(j);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + taskName + "处理完毕");
            });
        }
        //关闭线程池
        executor.shutdown();
    }
}

```

输出：

```java
pool-1-thread-1任务0处理完毕
pool-1-thread-2任务1处理完毕
pool-1-thread-3任务2处理完毕
pool-1-thread-1任务3处理完毕
pool-1-thread-2任务4处理完毕
pool-1-thread-3任务5处理完毕
pool-1-thread-1任务6处理完毕
pool-1-thread-2任务7处理完毕
pool-1-thread-3任务8处理完毕
pool-1-thread-1任务9处理完毕
```

## 线程池中常见4种工作队列

任务太多的时候，工作队列用于暂时缓存待处理的任务，jdk中常见的5种阻塞队列：

**ArrayBlockingQueue**：是一个基于数组结构的有界阻塞队列，此队列按照先进先出原则对元素进行排序

**LinkedBlockingQueue**：是一个基于链表结构的阻塞队列，此队列按照先进先出排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法`Executors.newFixedThreadPool`使用了这个队列。

**SynchronousQueue** ：一个不存储元素的阻塞队列，每个插入操作必须等到另外一个线程调用移除操作，否则插入操作一直处理阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法`Executors.newCachedThreadPool`使用这个队列

**PriorityBlockingQueue**：优先级队列，进入队列的元素按照优先级会进行排序

前2种队列相关示例就不说了，主要说一下后面2种队列的使用示例。

## SynchronousQueue队列的线程池

```java
package com.itsoku.chat16;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo2 {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        for (int i = 0; i < 50; i++) {
            int j = i;
            String taskName = "任务" + j;
            executor.execute(() -> {
                System.out.println(Thread.currentThread().getName() + "处理" + taskName);
                //模拟任务内部处理耗时
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        executor.shutdown();
    }
}
pool-1-thread-1处理任务0
pool-1-thread-2处理任务1
pool-1-thread-3处理任务2
pool-1-thread-6处理任务5
pool-1-thread-7处理任务6
pool-1-thread-4处理任务3
pool-1-thread-5处理任务4
pool-1-thread-8处理任务7
pool-1-thread-9处理任务8
pool-1-thread-10处理任务9
pool-1-thread-11处理任务10
pool-1-thread-12处理任务11
pool-1-thread-13处理任务12
pool-1-thread-14处理任务13
pool-1-thread-15处理任务14
pool-1-thread-16处理任务15
pool-1-thread-17处理任务16
pool-1-thread-18处理任务17
pool-1-thread-19处理任务18
pool-1-thread-20处理任务19
pool-1-thread-21处理任务20
pool-1-thread-25处理任务24
pool-1-thread-24处理任务23
pool-1-thread-23处理任务22
pool-1-thread-22处理任务21
pool-1-thread-26处理任务25
pool-1-thread-27处理任务26
pool-1-thread-28处理任务27
pool-1-thread-30处理任务29
pool-1-thread-29处理任务28
pool-1-thread-31处理任务30
pool-1-thread-32处理任务31
pool-1-thread-33处理任务32
pool-1-thread-38处理任务37
pool-1-thread-35处理任务34
pool-1-thread-36处理任务35
pool-1-thread-41处理任务40
pool-1-thread-34处理任务33
pool-1-thread-39处理任务38
pool-1-thread-40处理任务39
pool-1-thread-37处理任务36
pool-1-thread-42处理任务41
pool-1-thread-43处理任务42
pool-1-thread-45处理任务44
pool-1-thread-46处理任务45
pool-1-thread-44处理任务43
pool-1-thread-47处理任务46
pool-1-thread-50处理任务49
pool-1-thread-48处理任务47
pool-1-thread-49处理任务48
```

代码中使用`Executors.newCachedThreadPool()`创建线程池，看一下的源码：

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

从输出中可以看出，系统创建了50个线程处理任务，代码中使用了`SynchronousQueue`同步队列，这种队列比较特殊，放入元素必须要有另外一个线程去获取这个元素，否则放入元素会失败或者一直阻塞在那里直到有线程取走，示例中任务处理休眠了指定的时间，导致已创建的工作线程都忙于处理任务，所以新来任务之后，将任务丢入同步队列会失败，丢入队列失败之后，会尝试新建线程处理任务。使用上面的方式创建线程池需要注意，如果需要处理的任务比较耗时，会导致新来的任务都会创建新的线程进行处理，可能会导致创建非常多的线程，最终耗尽系统资源，触发OOM。

## PriorityBlockingQueue优先级队列的线程池

```java
package com.itsoku.chat16;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo3 {

    static class Task implements Runnable, Comparable<Task> {

        private int i;
        private String name;

        public Task(int i, String name) {
            this.i = i;
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "处理" + this.name);
        }

        @Override
        public int compareTo(Task o) {
            return Integer.compare(o.i, this.i);
        }
    }

    public static void main(String[] args) {
        ExecutorService executor = new ThreadPoolExecutor(1, 1,
                60L, TimeUnit.SECONDS,
                new PriorityBlockingQueue());
        for (int i = 0; i < 10; i++) {
            String taskName = "任务" + i;
            executor.execute(new Task(i, taskName));
        }
        for (int i = 100; i >= 90; i--) {
            String taskName = "任务" + i;
            executor.execute(new Task(i, taskName));
        }
        executor.shutdown();
    }
}
```

输出：

```java
pool-1-thread-1处理任务0
pool-1-thread-1处理任务100
pool-1-thread-1处理任务99
pool-1-thread-1处理任务98
pool-1-thread-1处理任务97
pool-1-thread-1处理任务96
pool-1-thread-1处理任务95
pool-1-thread-1处理任务94
pool-1-thread-1处理任务93
pool-1-thread-1处理任务92
pool-1-thread-1处理任务91
pool-1-thread-1处理任务90
pool-1-thread-1处理任务9
pool-1-thread-1处理任务8
pool-1-thread-1处理任务7
pool-1-thread-1处理任务6
pool-1-thread-1处理任务5
pool-1-thread-1处理任务4
pool-1-thread-1处理任务3
pool-1-thread-1处理任务2
pool-1-thread-1处理任务1
```

输出中，除了第一个任务，其他任务按照优先级高低按顺序处理。原因在于：创建线程池的时候使用了优先级队列，进入队列中的任务会进行排序，任务的先后顺序由Task中的i变量决定。向`PriorityBlockingQueue`加入元素的时候，内部会调用代码中Task的`compareTo`方法决定元素的先后顺序。

## 自定义创建线程的工厂

给线程池中线程起一个有意义的名字，在系统出现问题的时候，通过线程堆栈信息可以更容易发现系统中问题所在。自定义创建工厂需要实现`java.util.concurrent.ThreadFactory`接口中的`Thread newThread(Runnable r)`方法，参数为传入的任务，需要返回一个工作线程。

示例代码：

```java
package com.itsoku.chat16;

import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo4 {
    static AtomicInteger threadNum = new AtomicInteger(1);

    public static void main(String[] args) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 5,
                60L, TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(10), r -> {
            Thread thread = new Thread(r);
            thread.setName("自定义线程-" + threadNum.getAndIncrement());
            return thread;
        });
        for (int i = 0; i < 5; i++) {
            String taskName = "任务-" + i;
            executor.execute(() -> {
                System.out.println(Thread.currentThread().getName() + "处理" + taskName);
            });
        }
        executor.shutdown();
    }
}
```

输出：

```java
自定义线程-1处理任务-0
自定义线程-3处理任务-2
自定义线程-2处理任务-1
自定义线程-4处理任务-3
自定义线程-5处理任务-4
```

代码中在任务中输出了当前线程的名称，可以看到是我们自定义的名称。

通过jstack查看线程的堆栈信息，也可以看到我们自定义的名称，我们可以将代码中`executor.shutdown();`先给注释掉让程序先不退出，然后通过jstack查看，如下：

![img](https://img2018.cnblogs.com/blog/687624/201907/687624-20190729090012127-75511662.png)

## 4种常见饱和策略

当线程池中队列已满，并且线程池已达到最大线程数，线程池会将任务传递给饱和策略进行处理。这些策略都实现了`RejectedExecutionHandler`接口。接口中有个方法：

```java
void rejectedExecution(Runnable r, ThreadPoolExecutor executor)
```

> 参数说明：
>
> **r**：需要执行的任务
>
> **executor**：当前线程池对象

JDK中提供了4种常见的饱和策略:

**AbortPolicy**：直接抛出异常

**CallerRunsPolicy**：在当前调用者的线程中运行任务，即随丢来的任务，由他自己去处理

**DiscardOldestPolicy**：丢弃队列中最老的一个任务，即丢弃队列头部的一个任务，然后执行当前传入的任务

**DiscardPolicy**：不处理，直接丢弃掉，方法内部为空

## 自定义饱和策略

需要实现`RejectedExecutionHandler`接口。任务无法处理的时候，我们想记录一下日志，我们需要自定义一个饱和策略，示例代码：

```java
package com.itsoku.chat16;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo5 {
    static class Task implements Runnable {
        String name;

        public Task(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "处理" + this.name);
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        @Override
        public String toString() {
            return "Task{" +
                    "name='" + name + '\'' +
                    '}';
        }
    }

    public static void main(String[] args) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(1,
                1,
                60L,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(1),
                Executors.defaultThreadFactory(),
                (r, executors) -> {
                    //自定义饱和策略
                    //记录一下无法处理的任务
                    System.out.println("无法处理的任务：" + r.toString());
                });
        for (int i = 0; i < 5; i++) {
            executor.execute(new Task("任务-" + i));
        }
        executor.shutdown();
    }
}
```

输出：

```java
无法处理的任务：Task{name='任务-2'}
无法处理的任务：Task{name='任务-3'}
pool-1-thread-1处理任务-0
无法处理的任务：Task{name='任务-4'}
pool-1-thread-1处理任务-1
```

输出结果中可以看到有3个任务进入了饱和策略中，记录了任务的日志，对于无法处理多任务，我们最好能够记录一下，让开发人员能够知道。任务进入了饱和策略，说明线程池的配置可能不是太合理，或者机器的性能有限，需要做一些优化调整。

## 线程池状态

![ThreadPoolState](png\ThreadPoolState.png)

**RUNNING**

(1) 状态说明：线程池处在RUNNING状态时，能够接收新任务，以及对已添加的任务进行处理。 
(2) 状态切换：线程池的初始化状态是RUNNING。换句话说，线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0！

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

**SHUTDOWN**

(1) 状态说明：线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务。 
(2) 状态切换：调用线程池的shutdown()接口时，线程池由RUNNING -> SHUTDOWN。

**STOP**

(1) 状态说明：线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。 
(2) 状态切换：调用线程池的shutdownNow()接口时，线程池由(RUNNING or SHUTDOWN ) -> STOP。

**TIDYING**

(1) 状态说明：当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。 
(2) 状态切换：当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。 
当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。

 **TERMINATED**

(1) 状态说明：线程池彻底终止，就变成TERMINATED状态。 
(2) 状态切换：线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。



## 线程池中的2个关闭方法

线程池提供了2个关闭方法：`shutdown`和`shutdownNow`，当调用者两个方法之后，线程池会遍历内部的工作线程，然后调用每个工作线程的interrrupt方法给线程发送中断信号，内部如果无法响应中断信号的可能永远无法终止，所以如果内部有无线循环的，最好在循环内部检测一下线程的中断信号，合理的退出。调用者两个方法中任意一个，线程池的`isShutdown`方法就会返回true，当所有的任务线程都关闭之后，才表示线程池关闭成功，这时调用`isTerminaed`方法会返回true。

调用`shutdown`方法之后，线程池将不再接口新任务，内部会将所有已提交的任务处理完毕，处理完毕之后，工作线程自动退出。

而调用`shutdownNow`方法后，线程池会将还未处理的（在队里等待处理的任务）任务移除，将正在处理中的处理完毕之后，工作线程自动退出。

至于调用哪个方法来关闭线程，应该由提交到线程池的任务特性决定，多数情况下调用`shutdown`方法来关闭线程池，如果任务不一定要执行完，则可以调用`shutdownNow`方法。

## 扩展线程池

虽然jdk提供了`ThreadPoolExecutor`这个高性能线程池，但是如果我们自己想在这个线程池上面做一些扩展，比如，监控每个任务执行的开始时间，结束时间，或者一些其他自定义的功能，我们应该怎么办？

这个jdk已经帮我们想到了，`ThreadPoolExecutor`内部提供了几个方法`beforeExecute`、`afterExecute`、`terminated`，可以由开发人员自己去这些方法。看一下线程池内部的源码：

```java
try {
    beforeExecute(wt, task);//任务执行之前调用的方法
    Throwable thrown = null;
    try {
        task.run();
    } catch (RuntimeException x) {
        thrown = x;
        throw x;
    } catch (Error x) {
        thrown = x;
        throw x;
    } catch (Throwable x) {
        thrown = x;
        throw new Error(x);
    } finally {
        afterExecute(task, thrown);//任务执行完毕之后调用的方法
    }
} finally {
    task = null;
    w.completedTasks++;
    w.unlock();
}
```

**beforeExecute：任务执行之前调用的方法，有2个参数，第1个参数是执行任务的线程，第2个参数是任务**

```java
protected void beforeExecute(Thread t, Runnable r) { }
```

**afterExecute：任务执行完成之后调用的方法，2个参数，第1个参数表示任务，第2个参数表示任务执行时的异常信息，如果无异常，第二个参数为null**

```java
protected void afterExecute(Runnable r, Throwable t) { }
```

**terminated：线程池最终关闭之后调用的方法。所有的工作线程都退出了，最终线程池会退出，退出时调用该方法**

示例代码：

```java
package com.itsoku.chat16;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo6 {
    static class Task implements Runnable {
        String name;

        public Task(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "处理" + this.name);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        @Override
        public String toString() {
            return "Task{" +
                    "name='" + name + '\'' +
                    '}';
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(10,
                10,
                60L,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(1),
                Executors.defaultThreadFactory(),
                (r, executors) -> {
                    //自定义饱和策略
                    //记录一下无法处理的任务
                    System.out.println("无法处理的任务：" + r.toString());
                }) {
            @Override
            protected void beforeExecute(Thread t, Runnable r) {
                System.out.println(System.currentTimeMillis() + "," + t.getName() + ",开始执行任务:" + r.toString());
            }

            @Override
            protected void afterExecute(Runnable r, Throwable t) {
                System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ",任务:" + r.toString() + "，执行完毕!");
            }

            @Override
            protected void terminated() {
                System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "，关闭线程池!");
            }
        };
        for (int i = 0; i < 10; i++) {
            executor.execute(new Task("任务-" + i));
        }
        TimeUnit.SECONDS.sleep(1);
        executor.shutdown();
    }
}
```

输出：

```java
1564324574847,pool-1-thread-1,开始执行任务:Task{name='任务-0'}
1564324574850,pool-1-thread-3,开始执行任务:Task{name='任务-2'}
pool-1-thread-3处理任务-2
1564324574849,pool-1-thread-2,开始执行任务:Task{name='任务-1'}
pool-1-thread-2处理任务-1
1564324574848,pool-1-thread-5,开始执行任务:Task{name='任务-4'}
pool-1-thread-5处理任务-4
1564324574848,pool-1-thread-4,开始执行任务:Task{name='任务-3'}
pool-1-thread-4处理任务-3
1564324574850,pool-1-thread-7,开始执行任务:Task{name='任务-6'}
pool-1-thread-7处理任务-6
1564324574850,pool-1-thread-6,开始执行任务:Task{name='任务-5'}
1564324574851,pool-1-thread-8,开始执行任务:Task{name='任务-7'}
pool-1-thread-8处理任务-7
pool-1-thread-1处理任务-0
pool-1-thread-6处理任务-5
1564324574851,pool-1-thread-10,开始执行任务:Task{name='任务-9'}
pool-1-thread-10处理任务-9
1564324574852,pool-1-thread-9,开始执行任务:Task{name='任务-8'}
pool-1-thread-9处理任务-8
1564324576851,pool-1-thread-2,任务:Task{name='任务-1'}，执行完毕!
1564324576851,pool-1-thread-3,任务:Task{name='任务-2'}，执行完毕!
1564324576852,pool-1-thread-1,任务:Task{name='任务-0'}，执行完毕!
1564324576852,pool-1-thread-4,任务:Task{name='任务-3'}，执行完毕!
1564324576852,pool-1-thread-8,任务:Task{name='任务-7'}，执行完毕!
1564324576852,pool-1-thread-7,任务:Task{name='任务-6'}，执行完毕!
1564324576852,pool-1-thread-5,任务:Task{name='任务-4'}，执行完毕!
1564324576853,pool-1-thread-6,任务:Task{name='任务-5'}，执行完毕!
1564324576853,pool-1-thread-10,任务:Task{name='任务-9'}，执行完毕!
1564324576853,pool-1-thread-9,任务:Task{name='任务-8'}，执行完毕!
1564324576853,pool-1-thread-9，关闭线程池!
```

从输出结果中可以看到，每个需要执行的任务打印了3行日志，执行前由线程池的`beforeExecute`打印，执行时会调用任务的`run`方法，任务执行完毕之后，会调用线程池的`afterExecute`方法，从每个任务的首尾2条日志中可以看到每个任务耗时2秒左右。线程池最终关闭之后调用了`terminated`方法。

## 合理地配置线程池

要想合理的配置线程池，需要先分析任务的特性，可以冲一下几个角度分析：

- 任务的性质：CPU密集型任务、IO密集型任务和混合型任务
- 任务的优先级：高、中、低
- 任务的执行时间：长、中、短
- 任务的依赖性：是否依赖其他的系统资源，如数据库连接。

性质不同任务可以用不同规模的线程池分开处理。CPU密集型任务应该尽可能小的线程，如配置cpu数量+1个线程的线程池。由于IO密集型任务并不是一直在执行任务，不能让cpu闲着，则应配置尽可能多的线程，如：cup数量*2。混合型的任务，如果可以拆分，将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这2个任务执行的时间相差不是太大，那么分解后执行的吞吐量将高于串行执行的吞吐量。可以通过`Runtime.getRuntime().availableProcessors()`方法获取cpu数量。优先级不同任务可以对线程池采用优先级队列来处理，让优先级高的先执行。

使用队列的时候建议使用有界队列，有界队列增加了系统的稳定性，如果采用无解队列，任务太多的时候可能导致系统OOM，直接让系统宕机。

## 线程池中线程数量的配置

线程池汇总线程大小对系统的性能有一定的影响，我们的目标是希望系统能够发挥最好的性能，过多或者过小的线程数量无法有消息的使用机器的性能。Java Concurrency inPractice书中给出了估算线程池大小的公式：

```java
Ncpu = CUP的数量
Ucpu = 目标CPU的使用率，0<=Ucpu<=1
W/C = 等待时间与计算时间的比例
为保存处理器达到期望的使用率，最有的线程池的大小等于：
Nthreads = Ncpu × Ucpu × (1+W/C)
```

## 一些使用建议

在《阿里巴巴java开发手册》中指出了线程资源必须通过线程池提供，不允许在应用中自行显示的创建线程，这样一方面是线程的创建更加规范，可以合理控制开辟线程的数量；另一方面线程的细节管理交给线程池处理，优化了资源的开销。而线程池不允许使用Executors去创建，而要通过ThreadPoolExecutor方式，这一方面是由于jdk中Executor框架虽然提供了如newFixedThreadPool()、newSingleThreadExecutor()、newCachedThreadPool()等创建线程池的方法，但都有其局限性，不够灵活；另外由于前面几种方法内部也是通过ThreadPoolExecutor方式实现，使用ThreadPoolExecutor有助于大家明确线程池的运行规则，创建符合自己的业务场景需要的线程池，避免资源耗尽的风险。

## ThreadPoolTaskExecutor 其他知识点汇总(待补充)

1. **线程池中的所有线程超过了空闲时间都会被销毁么？**

   如果allowCoreThreadTimeOut为true，超过了空闲时间的所有线程都会被回收，不过这个值默认是false，系统会保留核心线程，其他的会被回收

2. **空闲线程是如何被销毁的？**

   所有运行的工作线程会尝试从队列中获取任务去执行，超过一定时间（keepAliveTime）还没有拿到任务，自己主动退出

3. **核心线程在线程池创建的时候会初始化好么？**

   默认情况下，核心线程不会进行初始化，在刚开始调用线程池执行任务的时候，传入一个任务会创建一个线程，直到达到核心线程数。不过可以在创建线程池之后，调用其`prestartAllCoreThreads`提前将核心线程创建好。

# Executor框架

### Executors框架介绍

Executors框架是Doug Lea的神作，通过这个框架，可以很容易的使用线程池高效地处理并行任务。

**Excecutor框架主要包含3部分的内容：**

1. 任务相关的：包含被执行的任务要实现的接口：**Runnable**接口或**Callable**接口
2. 任务的执行相关的：包含任务执行机制的**核心接口Executor**，以及继承自`Executor`的`ExecutorService`接口。Executor框架中有两个关键的类实现了ExecutorService接口（`ThreadPoolExecutor`和`ScheduleThreadPoolExecutor`）
3. 异步计算结果相关的：包含**接口Future**和**实现Future接口的FutureTask类**

**Executors框架包括：**

- Executor
- ExecutorService
- ThreadPoolExecutor
- Executors
- Future
- Callable
- FutureTask
- CompletableFuture
- CompletionService
- ExecutorCompletionService

下面我们来一个个介绍其用途和使用方法。

## Executor接口

Executor接口中定义了方法execute(Runable able)接口，该方法接受一个Runable实例，他来执行一个任务，任务即实现一个Runable接口的类。

## ExecutorService接口

ExecutorService继承于Executor接口，他提供了更为丰富的线程实现方法，比如ExecutorService提供关闭自己的方法，以及为跟踪一个或多个异步任务执行状况而生成Future的方法。

ExecutorService有三种状态：运行、关闭、终止。创建后便进入运行状态，当调用了shutdown()方法时，便进入了关闭状态，此时意味着ExecutorService不再接受新的任务，但是他还是会执行已经提交的任务，当所有已经提交了的任务执行完后，便达到终止状态。如果不调用shutdown方法，ExecutorService方法会一直运行下去，系统一般不会主动关闭。

## ThreadPoolExecutor类

线程池类，实现了`ExecutorService`接口中所有方法，该类也是我们经常要用到的，非常重要，关于此类有详细的介绍，可以移步：[玩转java中的线程池](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933151&idx=1&sn=2020066b974b5f4c0823abd419e8adae&chksm=88621b21bf159237bdacfb47bd1a344f7123aabc25e3607e78d936dd554412edce5dd825003d&token=995072421&lang=zh_CN#rd)

## ScheduleThreadPoolExecutor定时器

ScheduleThreadPoolExecutor继承自`ScheduleThreadPoolExecutor`，他主要用来延迟执行任务，或者定时执行任务。功能和Timer类似，但是ScheduleThreadPoolExecutor更强大、更灵活一些。Timer后台是单个线程，而ScheduleThreadPoolExecutor可以在创建的时候指定多个线程。

常用方法介绍：

### schedule:延迟执行任务1次

使用`ScheduleThreadPoolExecutor的schedule方法`，看一下这个方法的声明：

```java
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit)
```

> 3个参数：
>
> command：需要执行的任务
>
> delay：需要延迟的时间
>
> unit：参数2的时间单位，是个枚举，可以是天、小时、分钟、秒、毫秒、纳秒等

**示例代码：**

```java
package com.itsoku.chat18;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo1 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println(System.currentTimeMillis());
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(10);
        scheduledExecutorService.schedule(() -> {
            System.out.println(System.currentTimeMillis() + "开始执行");
            //模拟任务耗时
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(System.currentTimeMillis() + "执行结束");
        }, 2, TimeUnit.SECONDS);
    }
}
```

输出：

```java
1564575180457
1564575185525开始执行
1564575188530执行结束
```

### scheduleAtFixedRate:固定的频率执行任务

使用`ScheduleThreadPoolExecutor的scheduleAtFixedRate`方法，该方法设置了执行周期，下一次执行时间相当于是上一次的执行时间加上period，任务每次执行完毕之后才会计算下次的执行时间。

看一下这个方法的声明：

```java
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);
```

> 4个参数：
>
> command：表示要执行的任务
>
> initialDelay：表示延迟多久执行第一次
>
> period：连续执行之间的时间间隔
>
> unit：参数2和参数3的时间单位，是个枚举，可以是天、小时、分钟、秒、毫秒、纳秒等

假设系统调用scheduleAtFixedRate的时间是T1，那么执行时间如下：

第1次：T1+initialDelay

第2次：T1+initialDelay+period

第3次：T1+initialDelay+2*period

第n次：T1+initialDelay+(n-1)*period

**示例代码：**

```java
package com.itsoku.chat18;

import java.sql.Time;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo2 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println(System.currentTimeMillis());
        //任务执行计数器
        AtomicInteger count = new AtomicInteger(1);
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(10);
        scheduledExecutorService.scheduleAtFixedRate(() -> {
            int currCount = count.getAndIncrement();
            System.out.println(Thread.currentThread().getName());
            System.out.println(System.currentTimeMillis() + "第" + currCount + "次" + "开始执行");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(System.currentTimeMillis() + "第" + currCount + "次" + "执行结束");
        }, 1, 1, TimeUnit.SECONDS);
    }
}
```

前面6次输出结果：

```java
1564576404181
pool-1-thread-1
1564576405247第1次开始执行
1564576407251第1次执行结束
pool-1-thread-1
1564576407251第2次开始执行
1564576409252第2次执行结束
pool-1-thread-2
1564576409252第3次开始执行
1564576411255第3次执行结束
pool-1-thread-1
1564576411256第4次开始执行
1564576413260第4次执行结束
pool-1-thread-3
1564576413260第5次开始执行
1564576415265第5次执行结束
pool-1-thread-2
1564576415266第6次开始执行
1564576417269第6次执行结束
```

代码中设置的任务第一次执行时间是系统启动之后延迟一秒执行。后面每次时间间隔1秒，从输出中可以看出系统启动之后过了1秒任务第一次执行（1、3行输出），输出的结果中可以看到任务第一次执行结束时间和第二次的结束时间一样，为什么会这样？前面有介绍，任务当前执行完毕之后会计算下次执行时间，下次执行时间为上次执行的开始时间+period，第一次开始执行时间是1564576405247，加1秒为1564576406247，这个时间小于第一次结束的时间了，说明小于系统当前时间了，会立即执行。

### scheduleWithFixedDelay:固定的间隔执行任务

使用`ScheduleThreadPoolExecutor的scheduleWithFixedDelay`方法，该方法设置了执行周期，与scheduleAtFixedRate方法不同的是，下一次执行时间是上一次任务执行完的系统时间加上period，因而具体执行时间不是固定的，但周期是固定的，是采用相对固定的延迟来执行任务。看一下这个方法的声明：

```java
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);
```

> 4个参数：
>
> command：表示要执行的任务
>
> initialDelay：表示延迟多久执行第一次
>
> period：表示下次执行时间和上次执行结束时间之间的间隔时间
>
> unit：参数2和参数3的时间单位，是个枚举，可以是天、小时、分钟、秒、毫秒、纳秒等

假设系统调用scheduleAtFixedRate的时间是T1，那么执行时间如下：

第1次：T1+initialDelay，执行结束时间：E1

第2次：E1+period，执行结束时间：E2

第3次：E2+period，执行结束时间：E3

第4次：E3+period，执行结束时间：E4

第n次：上次执行结束时间+period

**示例代码：**

```java
package com.itsoku.chat18;

import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo3 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println(System.currentTimeMillis());
        //任务执行计数器
        AtomicInteger count = new AtomicInteger(1);
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(10);
        scheduledExecutorService.scheduleWithFixedDelay(() -> {
            int currCount = count.getAndIncrement();
            System.out.println(Thread.currentThread().getName());
            System.out.println(System.currentTimeMillis() + "第" + currCount + "次" + "开始执行");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(System.currentTimeMillis() + "第" + currCount + "次" + "执行结束");
        }, 1, 3, TimeUnit.SECONDS);
    }
}
```

前几次输出如下：

```java
1564578510983
pool-1-thread-1
1564578512087第1次开始执行
1564578514091第1次执行结束
pool-1-thread-1
1564578517096第2次开始执行
1564578519100第2次执行结束
pool-1-thread-2
1564578522103第3次开始执行
1564578524105第3次执行结束
pool-1-thread-1
1564578527106第4次开始执行
1564578529106第4次执行结束
```

延迟1秒之后执行第1次，后面每次的执行时间和上次执行结束时间间隔3秒。

`scheduleAtFixedRate`和`scheduleWithFixedDelay`示例建议多看2遍。

### 定时任务有异常会怎么样？

示例代码：

```java
package com.itsoku.chat18;

import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo4 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println(System.currentTimeMillis());
        //任务执行计数器
        AtomicInteger count = new AtomicInteger(1);
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(10);
        ScheduledFuture<?> scheduledFuture = scheduledExecutorService.scheduleWithFixedDelay(() -> {
            int currCount = count.getAndIncrement();
            System.out.println(Thread.currentThread().getName());
            System.out.println(System.currentTimeMillis() + "第" + currCount + "次" + "开始执行");
            System.out.println(10 / 0);
            System.out.println(System.currentTimeMillis() + "第" + currCount + "次" + "执行结束");
        }, 1, 1, TimeUnit.SECONDS);

        TimeUnit.SECONDS.sleep(5);
        System.out.println(scheduledFuture.isCancelled());
        System.out.println(scheduledFuture.isDone());

    }
}
```

系统输出如下内容就再也没有输出了：

```java
1564578848143
pool-1-thread-1
1564578849226第1次开始执行
false
true
```

**先说补充点知识**：schedule、scheduleAtFixedRate、scheduleWithFixedDelay这几个方法有个返回值ScheduledFuture，通过`ScheduledFuture`可以对执行的任务做一些操作，如判断任务是否被取消、是否执行完成。

再回到上面代码，任务中有个10/0的操作，会触发异常，发生异常之后没有任何现象，被ScheduledExecutorService内部给吞掉了，然后这个任务再也不会执行了，`scheduledFuture.isDone()`输出true，表示这个任务已经结束了，再也不会被执行了。**所以如果程序有异常，开发者自己注意处理一下，不然跑着跑着发现任务怎么不跑了，也没有异常输出。**

### 取消定时任务的执行

可能任务执行一会，想取消执行，可以调用`ScheduledFuture`的`cancel`方法，参数表示是否给任务发送中断信号。

```java
package com.itsoku.chat18;

import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo5 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println(System.currentTimeMillis());
        //任务执行计数器
        AtomicInteger count = new AtomicInteger(1);
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);
        ScheduledFuture<?> scheduledFuture = scheduledExecutorService.scheduleWithFixedDelay(() -> {
            int currCount = count.getAndIncrement();
            System.out.println(Thread.currentThread().getName());
            System.out.println(System.currentTimeMillis() + "第" + currCount + "次" + "开始执行");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(System.currentTimeMillis() + "第" + currCount + "次" + "执行结束");
        }, 1, 1, TimeUnit.SECONDS);

        TimeUnit.SECONDS.sleep(5);
        scheduledFuture.cancel(false);
        TimeUnit.SECONDS.sleep(1);
        System.out.println("任务是否被取消："+scheduledFuture.isCancelled());
        System.out.println("任务是否已完成："+scheduledFuture.isDone());
    }
}
```

输出：

```java
1564579843190
pool-1-thread-1
1564579844255第1次开始执行
1564579846260第1次执行结束
pool-1-thread-1
1564579847263第2次开始执行
任务是否被取消：true
任务是否已完成：true
1564579849267第2次执行结束
```

输出中可以看到任务被取消成功了。

## Executors类

Executors类，提供了一系列工厂方法用于创建线程池，返回的线程池都实现了ExecutorService接口。常用的方法有：

**newSingleThreadExecutor**

```java
public static ExecutorService newSingleThreadExecutor()
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory)
```

> 创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。内部使用了无限容量的LinkedBlockingQueue阻塞队列来缓存任务，任务如果比较多，单线程如果处理不过来，会导致队列堆满，引发OOM。

**newFixedThreadPool**

```java
public static ExecutorService newFixedThreadPool(int nThreads)
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory)
```

> 创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，在提交新任务，任务将会进入等待队列中等待。如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。内部使用了无限容量的LinkedBlockingQueue阻塞队列来缓存任务，任务如果比较多，如果处理不过来，会导致队列堆满，引发OOM。

**newCachedThreadPool**

```java
public static ExecutorService newCachedThreadPool()
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory)
```

> 创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，
>
> 那么就会回收部分空闲（60秒处于等待任务到来）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池的最大值是Integer的最大值(2^31-1)。内部使用了SynchronousQueue同步队列来缓存任务，此队列的特性是放入任务时必须要有对应的线程获取任务，任务才可以放入成功。如果处理的任务比较耗时，任务来的速度也比较快，会创建太多的线程引发OOM。

**newScheduledThreadPool**

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory)
```

> 创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。

在《阿里巴巴java开发手册》中指出了线程资源必须通过线程池提供，不允许在应用中自行显示的创建线程，这样一方面是线程的创建更加规范，可以合理控制开辟线程的数量；另一方面线程的细节管理交给线程池处理，优化了资源的开销。而线程池不允许使用Executors去创建，而要通过ThreadPoolExecutor方式，这一方面是由于jdk中Executor框架虽然提供了如newFixedThreadPool()、newSingleThreadExecutor()、newCachedThreadPool()等创建线程池的方法，但都有其局限性，不够灵活；另外由于前面几种方法内部也是通过ThreadPoolExecutor方式实现，使用ThreadPoolExecutor有助于大家明确线程池的运行规则，创建符合自己的业务场景需要的线程池，避免资源耗尽的风险。

## Future、Callable接口

`Future`接口定义了操作异步异步任务执行一些方法，**如获取异步任务的执行结果、取消任务的执行、判断任务是否被取消、判断任务执行是否完毕**等。

`Callable`接口中定义了需要有返回的任务需要实现的方法。

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

比如主线程让一个子线程去执行任务，子线程可能比较耗时，启动子线程开始执行任务后，主线程就去做其他事情了，过了一会才去获取子任务的执行结果。

### 获取异步任务执行结果

**示例代码：**

```java
package com.itsoku.chat18;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo6 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        Future<Integer> result = executorService.submit(() -> {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",start!");
            TimeUnit.SECONDS.sleep(5);
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",end!");
            return 10;
        });
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName());
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ",结果：" + result.get());
    }
}
```

输出：

```java
1564581941442,main
1564581941442,pool-1-thread-1,start!
1564581946447,pool-1-thread-1,end!
1564581941442,main,结果：10
```

代码中创建了一个线程池，调用线程池的`submit`方法执行任务，submit参数为`Callable`接口：表示需要执行的任务有返回值，submit方法返回一个`Future`对象，Future相当于一个凭证，可以在任意时间拿着这个凭证去获取对应任务的执行结果（调用其`get`方法），代码中调用了`result.get()`方法之后，此方法会阻塞当前线程直到任务执行结束。

### 超时获取异步任务执行结果

可能任务执行比较耗时，比如耗时1分钟，我最多只能等待10秒，如果10秒还没返回，我就去做其他事情了。

刚好get有个超时的方法，声明如下：

```java
V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
```

**示例代码：**

```java
package com.itsoku.chat18;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo8 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        Future<Integer> result = executorService.submit(() -> {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",start!");
            TimeUnit.SECONDS.sleep(5);
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",end!");
            return 10;
        });
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName());
        try {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ",结果：" + result.get(3,TimeUnit.SECONDS));
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        executorService.shutdown();
    }
}
```

输出：

```mipsasm
1564583177139,main
1564583177139,pool-1-thread-1,start!
java.util.concurrent.TimeoutException
	at java.util.concurrent.FutureTask.get(FutureTask.java:205)
	at com.itsoku.chat18.Demo8.main(Demo8.java:19)
1564583182142,pool-1-thread-1,end!
```

任务执行中休眠了5秒，get方法获取执行结果，超时时间是3秒，3秒还未获取到结果，get触发了`TimeoutException`异常，当前线程从阻塞状态苏醒了。

**`Future`其他方法介绍一下**

**cancel**：取消在执行的任务，参数表示是否对执行的任务发送中断信号，方法声明如下：

```java
boolean cancel(boolean mayInterruptIfRunning);
```

**isCancelled**：用来判断任务是否被取消

**isDone**：判断任务是否执行完毕。

**cancel方法来个示例：**

```java
package com.itsoku.chat18;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo7 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        Future<Integer> result = executorService.submit(() -> {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",start!");
            TimeUnit.SECONDS.sleep(5);
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",end!");
            return 10;
        });

        executorService.shutdown();
        
        TimeUnit.SECONDS.sleep(1);
        result.cancel(false);
        System.out.println(result.isCancelled());
        System.out.println(result.isDone());

        TimeUnit.SECONDS.sleep(5);
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName());
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ",结果：" + result.get());
        executorService.shutdown();
    }
}
```

输出：

```java
1564583031646,pool-1-thread-1,start!
true
true
1564583036649,pool-1-thread-1,end!
1564583037653,main
Exception in thread "main" java.util.concurrent.CancellationException
	at java.util.concurrent.FutureTask.report(FutureTask.java:121)
	at java.util.concurrent.FutureTask.get(FutureTask.java:192)
	at com.itsoku.chat18.Demo7.main(Demo7.java:24)
```

输出2个true，表示任务已被取消，已完成，最后调用get方法会触发`CancellationException`异常。

**总结：从上面可以看出Future、Callable接口需要结合ExecutorService来使用，需要有线程池的支持。**

## FutureTask类

FutureTask除了实现Future接口，还实现了Runnable接口，因此FutureTask可以交给Executor执行，也可以交给线程执行执行（**Thread有个Runnable的构造方法**），**FutureTask**表示带返回值结果的任务。

上面我们演示的是通过线程池执行任务然后获取执行结果。

这次我们通过FutureTask类，自己启动一个线程来获取执行结果，示例如下：

```java
package com.itsoku.chat18;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo9 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> futureTask = new FutureTask<Integer>(()->{
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",start!");
            TimeUnit.SECONDS.sleep(5);
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",end!");
            return 10;
        });
        System.out.println(System.currentTimeMillis()+","+Thread.currentThread().getName());
        new Thread(futureTask).start();
        System.out.println(System.currentTimeMillis()+","+Thread.currentThread().getName());
        System.out.println(System.currentTimeMillis()+","+Thread.currentThread().getName()+",结果:"+futureTask.get());
    }
}
```

输出：

```java
1564585122547,main
1564585122547,main
1564585122547,Thread-0,start!
1564585127549,Thread-0,end!
1564585122547,main,结果:10
```

**大家可以回过头去看一下上面用线程池的submit方法返回的Future实际类型正是FutureTask对象，有兴趣的可以设置个断点去看看。**

**FutureTask类还是相当重要的，标记一下。**

## 下面3个类，下一篇文章进行详解

1. 介绍CompletableFuture
2. 介绍CompletionService
3. 介绍ExecutorCompletionService

# JUC中的Executor框架详解2之ExecutorCompletionService

## 本文内容

1. ExecutorCompletionService出现的背景
2. 介绍CompletionService接口及常用的方法
3. 介绍ExecutorCompletionService类及其原理
4. 示例：执行一批任务，然后消费执行结果
5. 示例【2种方式】：异步执行一批任务，有一个完成立即返回，其他取消

## 需要解决的问题

还是举个例子说明更好理解一些。

买新房了，然后在网上下单买冰箱、洗衣机，电器商家不同，所以送货耗时不一样，然后等他们送货，快递只愿送到楼下，然后我们自己将其搬到楼上的家中。

用程序来模拟上面的实现。示例代码如下：

```java
package com.itsoku.chat18;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo12 {
    static class GoodsModel {
        //商品名称
        String name;
        //购物开始时间
        long startime;
        //送到的时间
        long endtime;

        public GoodsModel(String name, long startime, long endtime) {
            this.name = name;
            this.startime = startime;
            this.endtime = endtime;
        }

        @Override
        public String toString() {
            return name + "，下单时间[" + this.startime + "," + endtime + "]，耗时:" + (this.endtime - this.startime);
        }
    }

    /**
     * 将商品搬上楼
     *
     * @param goodsModel
     * @throws InterruptedException
     */
    static void moveUp(GoodsModel goodsModel) throws InterruptedException {
        //休眠5秒，模拟搬上楼耗时
        TimeUnit.SECONDS.sleep(5);
        System.out.println("将商品搬上楼，商品信息:" + goodsModel);
    }

    /**
     * 模拟下单
     *
     * @param name     商品名称
     * @param costTime 耗时
     * @return
     */
    static Callable<GoodsModel> buyGoods(String name, long costTime) {
        return () -> {
            long startTime = System.currentTimeMillis();
            System.out.println(startTime + "购买" + name + "下单!");
            //模拟送货耗时
            try {
                TimeUnit.SECONDS.sleep(costTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            long endTime = System.currentTimeMillis();
            System.out.println(startTime + name + "送到了!");
            return new GoodsModel(name, startTime, endTime);
        };
    }

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        long st = System.currentTimeMillis();
        System.out.println(st + "开始购物!");

        //创建一个线程池，用来异步下单
        ExecutorService executor = Executors.newFixedThreadPool(5);
        //异步下单购买冰箱
        Future<GoodsModel> bxFuture = executor.submit(buyGoods("冰箱", 5));
        //异步下单购买洗衣机
        Future<GoodsModel> xyjFuture = executor.submit(buyGoods("洗衣机", 2));
        //关闭线程池
        executor.shutdown();

        //等待冰箱送到
        GoodsModel bxGoodModel = bxFuture.get();
        //将冰箱搬上楼
        moveUp(bxGoodModel);
        //等待洗衣机送到
        GoodsModel xyjGooldModel = xyjFuture.get();
        //将洗衣机搬上楼
        moveUp(xyjGooldModel);
        long et = System.currentTimeMillis();
        System.out.println(et + "货物已送到家里咯，哈哈哈！");
        System.out.println("总耗时:" + (et - st));
    }
}
```

输出：

```java
1564653121515开始购物!
1564653121588购买冰箱下单!
1564653121588购买洗衣机下单!
1564653121588洗衣机送到了!
1564653121588冰箱送到了!
将商品搬上楼，商品信息:冰箱，下单时间[1564653121588,1564653126590]，耗时:5002
将商品搬上楼，商品信息:洗衣机，下单时间[1564653121588,1564653123590]，耗时:2002
1564653136591货物已送到家里咯，哈哈哈！
总耗时:15076
```

从输出中我们可以看出几个时间：

1. 购买冰箱耗时5秒
2. 购买洗衣机耗时2秒
3. 将冰箱送上楼耗时5秒
4. 将洗衣机送上楼耗时5秒
5. 共计耗时15秒

购买洗衣机、冰箱都是异步执行的，我们先把冰箱送上楼了，然后再把冰箱送上楼了。上面大家应该发现了一个问题，洗衣机先到的，洗衣机到了，我们并没有去把洗衣机送上楼，而是在等待冰箱到货（`bxFuture.get();`），然后将冰箱送上楼，中间导致浪费了3秒，现实中应该是这样的，先到的先送上楼，修改一下代码，洗衣机先到的，先送洗衣机上楼，代码如下：

```java
package com.itsoku.chat18;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo13 {
    static class GoodsModel {
        //商品名称
        String name;
        //购物开始时间
        long startime;
        //送到的时间
        long endtime;

        public GoodsModel(String name, long startime, long endtime) {
            this.name = name;
            this.startime = startime;
            this.endtime = endtime;
        }

        @Override
        public String toString() {
            return name + "，下单时间[" + this.startime + "," + endtime + "]，耗时:" + (this.endtime - this.startime);
        }
    }

    /**
     * 将商品搬上楼
     *
     * @param goodsModel
     * @throws InterruptedException
     */
    static void moveUp(GoodsModel goodsModel) throws InterruptedException {
        //休眠5秒，模拟搬上楼耗时
        TimeUnit.SECONDS.sleep(5);
        System.out.println("将商品搬上楼，商品信息:" + goodsModel);
    }

    /**
     * 模拟下单
     *
     * @param name     商品名称
     * @param costTime 耗时
     * @return
     */
    static Callable<GoodsModel> buyGoods(String name, long costTime) {
        return () -> {
            long startTime = System.currentTimeMillis();
            System.out.println(startTime + "购买" + name + "下单!");
            //模拟送货耗时
            try {
                TimeUnit.SECONDS.sleep(costTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            long endTime = System.currentTimeMillis();
            System.out.println(endTime + name + "送到了!");
            return new GoodsModel(name, startTime, endTime);
        };
    }

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        long st = System.currentTimeMillis();
        System.out.println(st + "开始购物!");

        //创建一个线程池，用来异步下单
        ExecutorService executor = Executors.newFixedThreadPool(5);
        //异步下单购买冰箱
        Future<GoodsModel> bxFuture = executor.submit(buyGoods("冰箱", 5));
        //异步下单购买洗衣机
        Future<GoodsModel> xyjFuture = executor.submit(buyGoods("洗衣机", 2));
        //关闭线程池
        executor.shutdown();

        //等待洗衣机送到
        GoodsModel xyjGooldModel = xyjFuture.get();
        //将洗衣机搬上楼
        moveUp(xyjGooldModel);
        //等待冰箱送到
        GoodsModel bxGoodModel = bxFuture.get();
        //将冰箱搬上楼
        moveUp(bxGoodModel);
        long et = System.currentTimeMillis();
        System.out.println(et + "货物已送到家里咯，哈哈哈！");
        System.out.println("总耗时:" + (et - st));
    }
}
```

输出：

```java
1564653153393开始购物!
1564653153466购买洗衣机下单!
1564653153466购买冰箱下单!
1564653155467洗衣机送到了!
1564653158467冰箱送到了!
将商品搬上楼，商品信息:洗衣机，下单时间[1564653153466,1564653155467]，耗时:2001
将商品搬上楼，商品信息:冰箱，下单时间[1564653153466,1564653158467]，耗时:5001
1564653165469货物已送到家里咯，哈哈哈！
总耗时:12076
```

耗时12秒，比第一种少了3秒。

问题来了，上面是我们通过调整代码达到了最优效果，实际上，购买冰箱和洗衣机具体哪个耗时时间长我们是不知道的，怎么办呢，有没有什么解决办法？

## CompletionService接口

CompletionService相当于一个执行任务的服务，通过submit丢任务给这个服务，服务内部去执行任务，可以通过服务提供的一些方法获取服务中已经完成的任务。

**接口内的几个方法：**

```java
Future<V> submit(Callable<V> task);
```

> 用于向服务中提交有返回结果的任务，并返回Future对象

```java
Future<V> submit(Runnable task, V result);
```

> 用户向服务中提交有返回值的任务去执行，并返回Future对象

```java
Future<V> take() throws InterruptedException;
```

> 从服务中返回并移除一个已经完成的任务，如果获取不到，会一致阻塞到有返回值为止。此方法会响应线程中断。

```java
Future<V> poll();
```

> 从服务中返回并移除一个已经完成的任务，如果内部没有已经完成的任务，则返回空，此方法会立即响应。

```java
Future<V> poll(long timeout, TimeUnit unit) throws InterruptedException;
```

> 尝试在指定的时间内从服务中返回并移除一个已经完成的任务，等待的时间超时还是没有获取到已完成的任务，则返回空。此方法会响应线程中断

通过submit向内部提交任意多个任务，通过take方法可以获取已经执行完成的任务，如果获取不到将等待。

## ExecutorCompletionService类

ExecutorCompletionService类是CompletionService接口的具体实现。

说一下其内部原理，ExecutorCompletionService创建的时候会传入一个线程池，调用submit方法传入需要执行的任务，任务由内部的线程池来处理；ExecutorCompletionService内部有个阻塞队列，任意一个任务完成之后，会将任务的执行结果（Future类型）放入阻塞队列中，然后其他线程可以调用它take、poll方法从这个阻塞队列中获取一个已经完成的任务，获取任务返回结果的顺序和任务执行完成的先后顺序一致，所以最先完成的任务会先返回。

**关于阻塞队列的知识后面会专门抽几篇来讲，大家可以关注一下后面的文章。**

看一下构造方法：

```java
public ExecutorCompletionService(Executor executor) {
        if (executor == null)
            throw new NullPointerException();
        this.executor = executor;
        this.aes = (executor instanceof AbstractExecutorService) ?
            (AbstractExecutorService) executor : null;
        this.completionQueue = new LinkedBlockingQueue<Future<V>>();
    }
```

构造方法需要传入一个Executor对象，这个对象表示任务执行器，所有传入的任务会被这个执行器执行。

`completionQueue`是用来存储任务结果的阻塞队列，默认用采用的是`LinkedBlockingQueue`，也支持开发自己设置。通过submit传入需要执行的任务，任务执行完成之后，会放入`completionQueue`中，有兴趣的可以看一下原码，还是很好理解的。

## 使用ExecutorCompletionService解决文章开头的问题

代码如下：

```java
package com.itsoku.chat18;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo14 {
    static class GoodsModel {
        //商品名称
        String name;
        //购物开始时间
        long startime;
        //送到的时间
        long endtime;

        public GoodsModel(String name, long startime, long endtime) {
            this.name = name;
            this.startime = startime;
            this.endtime = endtime;
        }

        @Override
        public String toString() {
            return name + "，下单时间[" + this.startime + "," + endtime + "]，耗时:" + (this.endtime - this.startime);
        }
    }

    /**
     * 将商品搬上楼
     *
     * @param goodsModel
     * @throws InterruptedException
     */
    static void moveUp(GoodsModel goodsModel) throws InterruptedException {
        //休眠5秒，模拟搬上楼耗时
        TimeUnit.SECONDS.sleep(5);
        System.out.println("将商品搬上楼，商品信息:" + goodsModel);
    }

    /**
     * 模拟下单
     *
     * @param name     商品名称
     * @param costTime 耗时
     * @return
     */
    static Callable<GoodsModel> buyGoods(String name, long costTime) {
        return () -> {
            long startTime = System.currentTimeMillis();
            System.out.println(startTime + "购买" + name + "下单!");
            //模拟送货耗时
            try {
                TimeUnit.SECONDS.sleep(costTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            long endTime = System.currentTimeMillis();
            System.out.println(endTime + name + "送到了!");
            return new GoodsModel(name, startTime, endTime);
        };
    }

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        long st = System.currentTimeMillis();
        System.out.println(st + "开始购物!");
        ExecutorService executor = Executors.newFixedThreadPool(5);

        //创建ExecutorCompletionService对象
        ExecutorCompletionService<GoodsModel> executorCompletionService = new ExecutorCompletionService<>(executor);
        //异步下单购买冰箱
        executorCompletionService.submit(buyGoods("冰箱", 5));
        //异步下单购买洗衣机
        executorCompletionService.submit(buyGoods("洗衣机", 2));
        executor.shutdown();

        //购买商品的数量
        int goodsCount = 2;
        for (int i = 0; i < goodsCount; i++) {
            //可以获取到最先到的商品
            GoodsModel goodsModel = executorCompletionService.take().get();
            //将最先到的商品送上楼
            moveUp(goodsModel);
        }

        long et = System.currentTimeMillis();
        System.out.println(et + "货物已送到家里咯，哈哈哈！");
        System.out.println("总耗时:" + (et - st));
    }
}
```

输出：

```java
1564653208284开始购物!
1564653208349购买冰箱下单!
1564653208349购买洗衣机下单!
1564653210349洗衣机送到了!
1564653213350冰箱送到了!
将商品搬上楼，商品信息:洗衣机，下单时间[1564653208349,1564653210349]，耗时:2000
将商品搬上楼，商品信息:冰箱，下单时间[1564653208349,1564653213350]，耗时:5001
1564653220350货物已送到家里咯，哈哈哈！
总耗时:12066
```

从输出中可以看出和我们希望的结果一致，代码中下单顺序是：冰箱、洗衣机，冰箱送货耗时5秒，洗衣机送货耗时2秒，洗衣机先到的，然后被送上楼了，冰箱后到被送上楼，总共耗时12秒，和期望的方案一样。

## 示例：执行一批任务，然后消费执行结果

代码如下：

```java
package com.itsoku.chat18;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.concurrent.*;
import java.util.function.Consumer;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo15 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        List<Callable<Integer>> list = new ArrayList<>();
        int taskCount = 5;
        for (int i = taskCount; i > 0; i--) {
            int j = i * 2;
            list.add(() -> {
                TimeUnit.SECONDS.sleep(j);
                return j;
            });
        }
        solve(executorService, list, a -> {
            System.out.println(System.currentTimeMillis() + ":" + a);
        });
        executorService.shutdown();
    }


    public static <T> void solve(Executor e, Collection<Callable<T>> solvers, Consumer<T> use) throws InterruptedException, ExecutionException {
        CompletionService<T> ecs = new ExecutorCompletionService<T>(e);
        for (Callable<T> s : solvers) {
            ecs.submit(s);
        }
        int n = solvers.size();
        for (int i = 0; i < n; ++i) {
            T r = ecs.take().get();
            if (r != null) {
                use.accept(r);
            }
        }
    }
}
```

输出：

```java
1564667625648:2
1564667627652:4
1564667629649:6
1564667631652:8
1564667633651:10
```

代码中传入了一批任务进行处理，最终将所有处理完成的按任务完成的先后顺序传递给`Consumer`进行消费了。

## 示例：异步执行一批任务，有一个完成立即返回，其他取消

这个给大家讲解2种方式。

### 方式1

使用ExecutorCompletionService实现，ExecutorCompletionService提供了获取一批任务中最先完成的任务结果的能力。

**代码如下：**

```java
package com.itsoku.chat18;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.concurrent.*;
import java.util.function.Consumer;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo16 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long startime = System.currentTimeMillis();
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        List<Callable<Integer>> list = new ArrayList<>();
        int taskCount = 5;
        for (int i = taskCount; i > 0; i--) {
            int j = i * 2;
            String taskName = "任务"+i;
            list.add(() -> {
                TimeUnit.SECONDS.sleep(j);
                System.out.println(taskName+"执行完毕!");
                return j;
            });
        }
        Integer integer = invokeAny(executorService, list);
        System.out.println("耗时:" + (System.currentTimeMillis() - startime) + ",执行结果:" + integer);
        executorService.shutdown();
    }


    public static <T> T invokeAny(Executor e, Collection<Callable<T>> solvers) throws InterruptedException, ExecutionException {
        CompletionService<T> ecs = new ExecutorCompletionService<T>(e);
        List<Future<T>> futureList = new ArrayList<>();
        for (Callable<T> s : solvers) {
            futureList.add(ecs.submit(s));
        }
        int n = solvers.size();
        try {
            for (int i = 0; i < n; ++i) {
                T r = ecs.take().get();
                if (r != null) {
                    return r;
                }
            }
        } finally {
            for (Future<T> future : futureList) {
                future.cancel(true);
            }
        }
        return null;
    }
}
```

程序输出下面结果然后停止了：

```java
任务1执行完毕!
耗时:2072,执行结果:2
```

代码中执行了5个任务，使用CompletionService执行任务，调用take方法获取最先执行完成的任务，然后返回。在finally中对所有任务发送取消操作（`future.cancel(true);`），从输出中可以看出只有任务1执行成功，其他任务被成功取消了，符合预期结果。

### 方式2

其实`ExecutorService`已经为我们提供了这样的方法，方法声明如下：

```java
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
```

**示例代码：**

```java
package com.itsoku.chat18;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo17 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long startime = System.currentTimeMillis();
        ExecutorService executorService = Executors.newFixedThreadPool(5);

        List<Callable<Integer>> list = new ArrayList<>();
        int taskCount = 5;
        for (int i = taskCount; i > 0; i--) {
            int j = i * 2;
            String taskName = "任务" + i;
            list.add(() -> {
                TimeUnit.SECONDS.sleep(j);
                System.out.println(taskName + "执行完毕!");
                return j;
            });
        }
        Integer integer = executorService.invokeAny(list);
        System.out.println("耗时:" + (System.currentTimeMillis() - startime) + ",执行结果:" + integer);
        executorService.shutdown();
    }
}
```

输出下面结果之后停止：

```java
任务1执行完毕!
耗时:2061,执行结果:2
```

输出结果和方式1中结果类似。

# CAS（Java并发的基石）

## 需求

**需求：我们开发了一个网站，需要对访问量进行统计，用户每次发一次请求，访问量+1，如何实现呢？**

下面我们来模仿有100个人同时访问，并且每个人对咱们的网站发起10次请求，最后总访问次数应该是1000次。实现访问如下。

### 计数器并发案例

代码如下：

```java
package com.concurrency.cas;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class UvDemo {

    //访问次数
    static int count = 0;

    //模拟访问一次
    public static void request() throws InterruptedException {
        //模拟耗时5毫秒
        TimeUnit.MILLISECONDS.sleep(5);
        count++;
    }

    public static void main(String[] args) throws InterruptedException {
        long starTime = System.currentTimeMillis();
        int threadSize = 100;
        CountDownLatch countDownLatch = new CountDownLatch(threadSize);
        for (int i = 0; i < threadSize; i++) {
            Thread thread = new Thread(() -> {
                try {
                    for (int j = 0; j < 10; j++) {
                        request();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();
                }
            });
            thread.start();
        }

        countDownLatch.await();
        long endTime = System.currentTimeMillis();
        System.out.println(Thread.currentThread().getName() + "，耗时：" + (endTime - starTime) + ",count=" + count);
    }
}

```

输出：

```java
main，耗时：138,count=975
```

代码中的count用来记录总访问次数，`request()`方法表示访问一次，内部休眠5毫秒模拟内部耗时，request方法内部对count++操作。程序最终耗时1秒多，执行还是挺快的，但是count和我们期望的结果不一致，我们期望的是1000，实际输出的是973（每次运行结果可能都不一样）。

**分析一下问题出在哪呢？**

**代码中采用的是多线程的方式来操作count，count++会有线程安全问题，count++操作实际上是由以下三步操作完成的：**

1. 获取count的值，记做A：A=count
2. 将A的值+1，得到B：B = A+1
3. 让B赋值给count：count = B

如果有A、B两个线程同时执行count++，他们同时执行到上面步骤的第1步，得到的count是一样的，3步操作完成之后，count只会+1，导致count只加了一次，从而导致结果不准确。

**那么我们应该怎么做的呢？**

对count++操作的时候，我们让多个线程排队处理，多个线程同时到达request()方法的时候，只能允许一个线程可以进去操作，其他的线程在外面候着，等里面的处理完毕出来之后，外面等着的再进去一个，这样操作count++就是排队进行的，结果一定是正确的。

**我们前面学了synchronized、ReentrantLock可以对资源加锁，保证并发的正确性，多线程情况下可以保证被锁的资源被串行访问，那么我们用synchronized来实现一下。**

### 计数器并发Synchronized案例

代码如下：

```java
package com.concurrency.cas;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class UvSynchronizedDemo {
    //访问次数
    static int count = 0;

    //模拟访问一次
    public static synchronized void request() throws InterruptedException {
        //模拟耗时5毫秒
        TimeUnit.MILLISECONDS.sleep(5);
        count++;
    }

    public static void main(String[] args) throws InterruptedException {
        long starTime = System.currentTimeMillis();
        int threadSize = 100;
        CountDownLatch countDownLatch = new CountDownLatch(threadSize);
        for (int i = 0; i < threadSize; i++) {
            Thread thread = new Thread(() -> {
                try {
                    for (int j = 0; j < 10; j++) {
                        request();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();
                }
            });
            thread.start();
        }

        countDownLatch.await();
        long endTime = System.currentTimeMillis();
        System.out.println(Thread.currentThread().getName() + "，耗时：" + (endTime - starTime) + ",count=" + count);
    }
}

```

输出：

```java
main，耗时：5563,count=1000
```

程序中request方法使用synchronized关键字，保证了并发情况下，request方法同一时刻只允许一个线程访问，request加锁了相当于串行执行了，count的结果和我们预期的结果一致，只是耗时比较长，5秒多。

### 计数器并发CAS案例

我们在看一下count++操作，count++操作实际上是被拆分为3步骤执行：

```java
1. 获取count的值，记做A：A=count
2. 将A的值+1，得到B：B = A+1
3. 让B赋值给count：count = B
```

方式2中我们通过加锁的方式让上面3步骤同时只能被一个线程操作，从而保证结果的正确性。

我们是否可以只在第3步加锁，减少加锁的范围，对第3步做以下处理：

```java
获取锁
第三步获取一下count最新的值，记做LV
判断LV是否等于A，如果相等，则将B的值赋给count，并返回true，否者返回false
释放锁
```

如果我们发现第3步返回的是false，我们就再次去获取count，将count赋值给A，对A+1赋值给B，然后再将A、B的值带入到上面的过程中执行，直到上面的结果返回true为止。

我们用代码来实现，如下：

```java
package com.concurrency.cas;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class UvCompareAndSwapDemo {

    //访问次数
    volatile static int count = 0;

    //模拟访问一次
    public static void request() throws InterruptedException {
        //模拟耗时5毫秒
        TimeUnit.MILLISECONDS.sleep(5);
        int expectCount;
        do {
            expectCount = getCount();
        } while (!compareAndSwap(expectCount, expectCount + 1));
    }

    /**
     * 获取count当前的值
     *
     * @return
     */
    public static int getCount() {
        return count;
    }

    /**
     * @param expectCount 期望count的值
     * @param newCount    需要给count赋的新值
     * @return
     */
    public static synchronized boolean compareAndSwap(int expectCount, int newCount) {
        //判断count当前值是否和期望的expectCount一样，如果一样将newCount赋值给count
        if (getCount() == expectCount) {
            count = newCount;
            return true;
        }
        return false;
    }

    public static void main(String[] args) throws InterruptedException {
        long starTime = System.currentTimeMillis();
        int threadSize = 100;
        CountDownLatch countDownLatch = new CountDownLatch(threadSize);
        for (int i = 0; i < threadSize; i++) {
            Thread thread = new Thread(() -> {
                try {
                    for (int j = 0; j < 10; j++) {
                        request();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();
                }
            });
            thread.start();
        }

        countDownLatch.await();
        long endTime = System.currentTimeMillis();
        System.out.println(Thread.currentThread().getName() + "，耗时：" + (endTime - starTime) + ",count=" + count);
    }
}

```

输出：

```java
main，耗时：116,count=1000
```

代码中用了`volatile`关键字修饰了count，可以保证count在多线程情况下的可见性。**关于volatile关键字的使用，也是非常非常重要的**

咱们再看一下代码，`compareAndSwap`方法，我们给起个简称吧叫`CAS`，这个方法有什么作用呢？这个方法使用`synchronized`修饰了，能保证此方法是线程安全的，多线程情况下此方法是串行执行的。方法由两个参数，expectCount：表示期望的值，newCount：表示要给count设置的新值。方法内部通过`getCount()`获取count当前的值，然后与期望的值expectCount比较，如果期望的值和count当前的值一致，则将新值newCount赋值给count。

再看一下request()方法，方法中有个do-while循环，循环内部获取count当前值赋值给了expectCount，循环结束的条件是`compareAndSwap`返回true，也就是说如果compareAndSwap如果不成功，循环再次获取count的最新值，然后+1，再次调用compareAndSwap方法，直到`compareAndSwap`返回成功为止。

代码中相当于将count++拆分开了，只对最后一步加锁了，减少了锁的范围，此代码的性能是不是比方式2快不少，还能保证结果的正确性。大家是不是感觉这个`compareAndSwap`方法挺好的，这东西确实很好，java中已经给我们提供了CAS的操作，功能非常强大，我们继续向下看。

## CAS（**Compare and Swap**）

### 概述

CAS（compare and swap）的缩写，中文翻译成比较并交换。

![CAS](png\CAS.png)

**CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。 如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值 。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该 位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前 值。）CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。”**

通常将 CAS 用于同步的方式是从地址 V 读取值 A，执行多步计算来获得新 值 B，然后使用 CAS 将 V 的值从 A 改为 B。如果 V 处的值尚未同时更改，则 CAS 操作成功。

> 系统底层进行CAS操作的时候，会判断当前系统是否为多核系统，如果是就给总线加锁，只有一个线程会对总线加锁成功，加锁成功之后会执行cas操作，也就是说CAS的原子性实际上是CPU实现的， 其实在这一点上还是有排他锁的.，只是比起用synchronized， 这里的排他时间要短的多， 所以在多线程情况下性能会比较好。

### 常用方法

java中提供了对CAS操作的支持，具体在`sun.misc.Unsafe`类中，声明如下：

```java
//第一个参数o为给定对象，offset为对象内存的偏移量，通过这个偏移量迅速定位字段并设置或获取该字段的值，
//expected表示期望值，x表示要设置的值，下面3个方法都通过CAS原子指令执行操作。
public final native boolean compareAndSwapObject(Object o, long offset,Object expected, Object x);                                                                                                  
 
public final native boolean compareAndSwapInt(Object o, long offset,int expected,int x);
 
public final native boolean compareAndSwapLong(Object o, long offset,long expected,long x);
```

上面三个方法都是类似的，主要对4个参数做一下说明。

> o：表示要操作的对象
>
> offset：表示要操作对象中属性地址的偏移量
>
> expected：表示需要修改数据的期望的值
>
> x：表示需要修改为的新值

JUC包中大部分功能都是依靠CAS操作完成的，所以这块也是非常重要的，有关Unsafe类，下篇文章会具体讲解。

### CAS详解

`synchronized`、`ReentrantLock`这种独占锁属于**悲观锁**，它是在假设需要操作的代码一定会发生冲突的，执行代码的时候先对代码加锁，让其他线程在外面等候排队获取锁。悲观锁如果锁的时间比较长，会导致其他线程一直处于等待状态，像我们部署的web应用，一般部署在tomcat中，内部通过线程池来处理用户的请求，如果很多请求都处于等待获取锁的状态，可能会耗尽tomcat线程池，从而导致系统无法处理后面的请求，导致服务器处于不可用状态。

除此之外，还有**乐观锁**，乐观锁的含义就是假设系统没有发生并发冲突，先按无锁方式执行业务，到最后了检查执行业务期间是否有并发导致数据被修改了，如果有并发导致数据被修改了 ，就快速返回失败，这样的操作使系统并发性能更高一些。cas中就使用了这样的操作。

关于乐观锁这块，想必大家在数据库中也有用到过，给大家举个例子，可能以后会用到。

如果你们的网站中有调用支付宝充值接口的，支付宝那边充值成功了会回调商户系统，商户系统接收到请求之后怎么处理呢？假设用户通过支付宝在商户系统中充值100，支付宝那边会从用户账户中扣除100，商户系统接收到支付宝请求之后应该在商户系统中给用户账户增加100，并且把订单状态置为成功。

处理过程如下：

```java
开启事务
获取订单信息
if(订单状态==待处理){
	给用户账户增加100
	将订单状态更新为成功
}
返回订单处理成功
提交事务
```

由于网络等各种问题，可能支付宝回调商户系统的时候，回调超时了，支付宝又发起了一笔回调请求，刚好这2笔请求同时到达上面代码，最终结果是给用户账户增加了200，这样事情就搞大了，公司蒙受损失，严重点可能让公司就此倒闭了。

那我们可以用乐观锁来实现，给订单表加个版本号version，要求每次更新订单数据，将版本号+1，那么上面的过程可以改为：

```java
获取订单信息,将version的值赋值给V_A
if(订单状态==待处理){
	开启事务
	给用户账户增加100
	update影响行数 = update 订单表 set version = version + 1 where id = 订单号 and version = V_A;
	if(update影响行数==1){
		提交事务
	}else{
		回滚事务
	}
}
返回订单处理成功
```

上面的update语句相当于我们说的CAS操作，执行这个update语句的时候，多线程情况下，数据库会对当前订单记录加锁，保证只有一条执行成功，执行成功的，影响行数为1，执行失败的影响行数为0，根据影响行数来决定提交还是回滚事务。上面操作还有一点是将事务范围缩小了，也提升了系统并发处理的性能。这个知识点希望你们能get到。

## CAS 的问题

cas这么好用，那么有没有什么问题呢？还真有

#### **ABA问题**

**CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了**。这就是CAS的ABA问题。 常见的解决思路是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么`A-B-A` 就会变成`1A-2B-3A`。 目前在JDK的atomic包里提供了一个类`AtomicStampedReference`来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

#### 自旋开销问题

上面我们说过如果CAS不成功，则会原地循环（自旋操作），如果长时间自旋会给CPU带来非常大的执行开销。并发量比较大的情况下，CAS成功概率可能比较低，可能会重试很多次才会成功。

解决方案：可以考虑限制自旋的次数，避免过度消耗 CPU；另外还可以考虑延迟执行。

#### 只能保证单个变量的原子性

当对一个共享变量执行操作时，可以使用 CAS 来保证原子性，但是如果要对多个共享变量进行操作时，CAS 是无法保证原子性的，比如需要将 i 和 j 同时加 1：

```
i++；j++；
```

这个时候可以使用 synchronized 进行加锁，有没有其他办法呢？有，将多个变量操作合成一个变量操作。从 JDK1.5 开始提供了`AtomicReference` 类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。

## CAS实现计数器

juc框架中提供了一些原子操作，底层是通过Unsafe类中的cas操作实现的。通过原子操作可以保证数据在并发情况下的正确性。

此处我们使用`java.util.concurrent.atomic.AtomicInteger`类来实现计数器功能，AtomicInteger内部是采用cas操作来保证对int类型数据增减操作在多线程情况下的正确性。

计数器代码如下：

```java
package com.concurrency.cas;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class UvAtomicIntegerDemo {

    //访问次数
    static AtomicInteger count = new AtomicInteger();

    //模拟访问一次
    public static void request() throws InterruptedException {
        //模拟耗时5毫秒
        TimeUnit.MILLISECONDS.sleep(5);
        //对count原子+1
        count.incrementAndGet();
    }

    public static void main(String[] args) throws InterruptedException {
        long starTime = System.currentTimeMillis();
        int threadSize = 100;
        CountDownLatch countDownLatch = new CountDownLatch(threadSize);
        for (int i = 0; i < threadSize; i++) {
            Thread thread = new Thread(() -> {
                try {
                    for (int j = 0; j < 10; j++) {
                        request();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();
                }
            });
            thread.start();
        }

        countDownLatch.await();
        long endTime = System.currentTimeMillis();
        System.out.println(Thread.currentThread().getName() + "，耗时：" + (endTime - starTime) + ",count=" + count);
    }
}

```

输出：

```java
main，耗时：119,count=1000
```

耗时很短，并且结果和期望的一致。

关于原子类操作，都位于`java.util.concurrent.atomic`包中，下篇文章我们主要来介绍一下这些常用的类及各自的使用场景。

# Unsafe（底层工具类）

## 概述

Java高并发中主要涉及到类位于java.util.concurrent包中，简称**juc**，juc中大部分类都是依赖于Unsafe来实现的，主要用到了Unsafe中的CAS、线程挂起、线程恢复等相关功能。所以如果打算深入了解JUC原理的，必须先了解一下Unsafe类。

先上一幅Unsafe类的功能图：

![img](png/unsafe.png)

Unsafe是位于sun.misc包下的一个类，主要提供一些用于执行低级别、不安全操作的方法，如直接访问系统内存资源、自主管理内存资源等，这些方法在提升Java运行效率、增强Java语言底层资源操作能力方面起到了很大的作用。但由于Unsafe类使Java语言拥有了类似C语言指针一样操作内存空间的能力，这无疑也增加了程序发生相关指针问题的风险。在程序中过度、不正确使用Unsafe类会使得程序出错的概率变大，使得Java这种安全的语言变得不再“安全”，因此对Unsafe的使用一定要慎重。

从Unsafe功能图上看出，Unsafe提供的API大致可分为**内存操作**、**CAS**、**Class相关**、**对象操作**、**线程调度**、**系统信息获取**、**内存屏障**、**数组操作**等几类。**

看一下UnSafe的原码部分：

```java
public final class Unsafe {
  // 单例对象
  private static final Unsafe theUnsafe;

  private Unsafe() {
  }
  @CallerSensitive
  public static Unsafe getUnsafe() {
    Class var0 = Reflection.getCallerClass();
    // 仅在引导类加载器`BootstrapClassLoader`加载时才合法
    if(!VM.isSystemDomainLoader(var0.getClassLoader())) {    
      throw new SecurityException("Unsafe");
    } else {
      return theUnsafe;
    }
  }
}
```

从代码中可以看出，Unsafe类为单例实现，提供静态方法getUnsafe获取Unsafe实例，内部会判断当前调用者是否是由系统类加载器加载的，如果不是系统类加载器加载的，会抛出`SecurityException`异常。

那我们想使用这个类，如何获取呢？

可以把我们的类放在jdk的lib目录下，那么启动的时候会自动加载，这种方式不是很好。

我们学过反射，通过反射可以获取到`Unsafe`中的`theUnsafe`字段的值，这样可以获取到Unsafe对象的实例。

## 通过反射获取Unsafe实例

代码如下：

```java
package com.concurrency.unsafe;

import sun.misc.Unsafe;

import java.lang.reflect.Field;

public class UnsafeDemo {

    static Unsafe unsafe;

    static {
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            unsafe = (Unsafe) field.get(null);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        System.out.println(unsafe);
    }
}

```

输出：

```css
sun.misc.Unsafe@136432db
```

## CAS 操作相关

看一下Unsafe中CAS相关方法定义：

```java
/**
 * CAS 操作
 *
 * @param o        包含要修改field的对象
 * @param offset   对象中某field的偏移量
 * @param expected 期望值
 * @param update   更新值
 * @return true | false
 */
public final native boolean compareAndSwapObject(Object o, long offset, Object expected, Object update);

public final native boolean compareAndSwapInt(Object o, long offset, int expected,int update);
  
public final native boolean compareAndSwapLong(Object o, long offset, long expected, long update);
```

什么是CAS? 即比较并替换，实现并发算法时常用到的一种技术。CAS操作包含三个操作数——**内存位置、预期原值及新值**。**执行CAS操作的时候，将内存位置的值与预期原值比较，如果相匹配，那么处理器会自动将该位置值更新为新值，否则，处理器不做任何操作，多个线程同时执行cas操作，只有一个会成功**。我们都知道，CAS是一条CPU的原子指令（cmpxchg指令），不会造成所谓的数据不一致问题，Unsafe提供的CAS方法（如compareAndSwapXXX）底层实现即为CPU指令cmpxchg。执行cmpxchg指令的时候，会判断当前系统是否为多核系统，如果是就给总线加锁，只有一个线程会对总线加锁成功，加锁成功之后会执行cas操作，也就是说CAS的原子性实际上是CPU实现的， 其实在这一点上还是有排他锁的，只是比起用synchronized， 这里的排他时间要短的多， 所以在多线程情况下性能会比较好。

> 说一下offset，offeset为字段的偏移量，每个对象有个地址，offset是字段相对于对象地址的偏移量，对象地址记为baseAddress，字段偏移量记为offeset，那么字段对应的实际地址就是baseAddress+offeset，所以cas通过对象、偏移量就可以去操作字段对应的值了。

CAS在java.util.concurrent.atomic相关类、Java AQS、JUC中并发集合等实现上有非常广泛的应用，我们看一下`java.util.concurrent.atomic.AtomicInteger`类，这个类可以在多线程环境中对int类型的数据执行高效的原子修改操作，并保证数据的正确性，看一下此类中用到Unsafe cas的地方：

java.util.concurrent.atomic包下的原子操作类都是基于CAS实现的，下面拿AtomicInteger分析一下，首先是AtomicInteger类变量的定义：

```
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
 try {
    valueOffset = unsafe.objectFieldOffset
        (AtomicInteger.class.getDeclaredField("value"));
  } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;
```

关于这段代码中出现的几个成员属性：

1、Unsafe是CAS的核心类，前面已经讲过了

2、valueOffset表示的是变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的原值的

3、value是用volatile修饰的，这是非常关键的

下面找一个方法getAndIncrement来研究一下AtomicInteger是如何实现的，比如我们常用的addAndGet方法：

```
public final int addAndGet(int delta) {
    for (;;) {
        int current = get();
        int next = current + delta;
        if (compareAndSet(current, next))
            return next;
    }
}
```

这段代码如何在不加锁的情况下通过CAS实现线程安全，我们不妨考虑一下方法的执行：

1、AtomicInteger里面的value原始值为3，即主内存中AtomicInteger的value为3，根据Java内存模型，线程1和线程2各自持有一份value的副本，值为3

2、线程1运行获取到当前的value为3，线程切换

3、线程2开始运行，获取到value为3，利用CAS对比内存中的值也为3，比较成功，修改内存，此时内存中的value改变比方说是4，线程切换

4、线程1恢复运行，利用CAS比较发现自己的value为3，内存中的value为4，得到一个重要的结论-->**此时value正在被另外一个线程修改，所以我不能去修改它**

5、线程1的compareAndSet失败，循环判断，因为value是volatile修饰的，所以它具备可见性的特性，线程2对于value的改变能被线程1看到，只要线程1发现当前获取的value是4，内存中的value也是4，说明**线程2对于value的修改已经完毕并且线程1可以尝试去修改它**

6、最后说一点，比如说此时线程3也准备修改value了，没关系，因为比较-交换是一个原子操作不可被打断，线程3修改了value，线程1进行compareAndSet的时候必然返回的false，这样线程1会继续循环去获取最新的value并进行compareAndSet，直至获取的value和内存中的value一致为止

整个过程中，利用CAS机制保证了对于value的修改的线程安全性。

JUC中其他地方使用到CAS的地方就不列举了，有兴趣的可以去看一下源码。

#### CAS操作实例

5个方法，看一下实现：

```java
/**
 * int类型值原子操作，对var2地址对应的值做原子增加操作(增加var4)
 *
 * @param var1 操作的对象
 * @param var2 var2字段内存地址偏移量
 * @param var4 需要加的值
 * @return
 */
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while (!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}

/**
 * long类型值原子操作，对var2地址对应的值做原子增加操作(增加var4)
 *
 * @param var1 操作的对象
 * @param var2 var2字段内存地址偏移量
 * @param var4 需要加的值
 * @return 返回旧值
 */
public final long getAndAddLong(Object var1, long var2, long var4) {
    long var6;
    do {
        var6 = this.getLongVolatile(var1, var2);
    } while (!this.compareAndSwapLong(var1, var2, var6, var6 + var4));

    return var6;
}

/**
 * int类型值原子操作方法，将var2地址对应的值置为var4
 *
 * @param var1 操作的对象
 * @param var2 var2字段内存地址偏移量
 * @param var4 新值
 * @return 返回旧值
 */
public final int getAndSetInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while (!this.compareAndSwapInt(var1, var2, var5, var4));

    return var5;
}

/**
 * long类型值原子操作方法，将var2地址对应的值置为var4
 *
 * @param var1 操作的对象
 * @param var2 var2字段内存地址偏移量
 * @param var4 新值
 * @return 返回旧值
 */
public final long getAndSetLong(Object var1, long var2, long var4) {
    long var6;
    do {
        var6 = this.getLongVolatile(var1, var2);
    } while (!this.compareAndSwapLong(var1, var2, var6, var4));

    return var6;
}

/**
 * Object类型值原子操作方法，将var2地址对应的值置为var4
 *
 * @param var1 操作的对象
 * @param var2 var2字段内存地址偏移量
 * @param var4 新值
 * @return 返回旧值
 */
public final Object getAndSetObject(Object var1, long var2, Object var4) {
    Object var5;
    do {
        var5 = this.getObjectVolatile(var1, var2);
    } while (!this.compareAndSwapObject(var1, var2, var5, var4));

    return var5;
}
```

看一下上面的方法，内部通过自旋的CAS操作实现的，这些方法都可以保证操作的数据在多线程环境中的原子性，正确性。

来个示例，我们还是来实现一个网站计数功能，同时有100个人发起对网站的请求，每个人发起10次请求，每次请求算一次，最终结果是1000次，代码如下：

```java
package com.itsoku.chat21;

import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo2 {
    static Unsafe unsafe;
    //用来记录网站访问量，每次访问+1
    static int count;
    //count在Demo.class对象中的地址偏移量
    static long countOffset;

    static {
        try {
            //获取Unsafe对象
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            unsafe = (Unsafe) field.get(null);

            Field countField = Demo2.class.getDeclaredField("count");
            //获取count字段在Demo2中的内存地址的偏移量
            countOffset = unsafe.staticFieldOffset(countField);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //模拟访问一次
    public static void request() throws InterruptedException {
        //模拟耗时5毫秒
        TimeUnit.MILLISECONDS.sleep(5);
        //对count原子加1
        unsafe.getAndAddInt(Demo2.class, countOffset, 1);
    }

    public static void main(String[] args) throws InterruptedException {
        long starTime = System.currentTimeMillis();
        int threadSize = 100;
        CountDownLatch countDownLatch = new CountDownLatch(threadSize);
        for (int i = 0; i < threadSize; i++) {
            Thread thread = new Thread(() -> {
                try {
                    for (int j = 0; j < 10; j++) {
                        request();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();
                }
            });
            thread.start();
        }

        countDownLatch.await();
        long endTime = System.currentTimeMillis();
        System.out.println(Thread.currentThread().getName() + "，耗时：" + (endTime - starTime) + ",count=" + count);
    }
}
```

输出：

```java
main，耗时：114,count=1000
```

代码中我们在静态块中通过反射获取到了Unsafe类的实例，然后获取Demo2中count字段内存地址偏移量`countOffset`，main方法中模拟了100个人，每人发起10次请求，等到所有请求完毕之后，输出count的结果。

代码中用到了`CountDownLatch`，通过`countDownLatch.await()`让主线程等待，等待100个子线程都执行完毕之后，主线程在进行运行。

## 线程调度

这部分，包括线程挂起、恢复、锁机制等方法。

```java
//取消阻塞线程
public native void unpark(Object thread);
//阻塞线程,isAbsolute：是否是绝对时间，如果为true，time是一个绝对时间，如果为false，time是一个相对时间，time表示纳秒
public native void park(boolean isAbsolute, long time);
//获得对象锁（可重入锁）
@Deprecated
public native void monitorEnter(Object o);
//释放对象锁
@Deprecated
public native void monitorExit(Object o);
//尝试获取对象锁
@Deprecated
public native boolean tryMonitorEnter(Object o);
```

调用`park`后，线程将被阻塞，直到`unpark`调用或者超时，如果之前调用过`unpark`,不会进行阻塞，即`park`和`unpark`不区分先后顺序。**monitorEnter、monitorExit、tryMonitorEnter** 3个方法**已过期**，不建议使用了。

将一个线程进行挂起是通过park方法实现的，调用 park后，线程将一直阻塞直到超时或者中断等条件出现。unpark可以终止一个挂起的线程，使其恢复正常。Java对线程的挂起操作被封装在 LockSupport类中，LockSupport类中有各种版本pack方法，其底层实现最终还是使用Unsafe.park()方法和Unsafe.unpark()方法

#### park和unpark示例

代码如下：

```java
package com.concurrency.unsafe;

import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.TimeUnit;

public class UnsafeParkDemo {

    static Unsafe unsafe;

    static {
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            unsafe = (Unsafe) field.get(null);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 调用park和unpark，模拟线程的挂起和唤醒
     *
     * @throws InterruptedException
     */
    public static void m1() throws InterruptedException {
        Thread thread = new Thread(() -> {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ",start");
            unsafe.park(false, 0);
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ",end");
        });
        thread.setName("thread1");
        thread.start();

        TimeUnit.SECONDS.sleep(5);
        unsafe.unpark(thread);
    }

    /**
     * 阻塞指定的时间
     */
    public static void m2() {
        Thread thread = new Thread(() -> {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ",start");
            //线程挂起3秒
            unsafe.park(false, TimeUnit.SECONDS.toNanos(3));
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ",end");
        });
        thread.setName("thread2");
        thread.start();
    }

    public static void main(String[] args) throws InterruptedException {
        m1();
        m2();
    }
}

```

输出：

```java
1565000238474,thread1,start
1565000243475,thread1,end
1565000243475,thread2,start
1565000246476,thread2,end
```

m1()中thread1调用park方法，park方法会将**当前线程阻塞**，被阻塞了5秒之后，被主线程调用unpark方法给唤醒了，unpark方法参数表示需要唤醒的线程。

线程中相当于有个许可，许可默认是0，调用park的时候，发现是0会阻塞当前线程，调用unpark之后，许可会被置为1，并会唤醒当前线程。如果在park之前先调用了unpark方法，执行park方法的时候，不会阻塞。park方法被唤醒之后，许可又会被置为0。多次调用unpark的效果是一样的，许可还是1。

juc中的`LockSupport`类是通过unpark和park方法实现的。

代码如下：

```java
package com.concurrency.unsafe;

import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.CountDownLatch;

public class UnsafeMonitorDemo {

    static Unsafe unsafe;
    //用来记录网站访问量，每次访问+1
    static int count;

    static {
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            unsafe = (Unsafe) field.get(null);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //模拟访问一次
    public static void request() {
        unsafe.monitorEnter(UnsafeMonitorDemo.class);
        try {
            count++;
        } finally {
            unsafe.monitorExit(UnsafeMonitorDemo.class);
        }
    }


    public static void main(String[] args) throws InterruptedException {
        long starTime = System.currentTimeMillis();
        int threadSize = 100;
        CountDownLatch countDownLatch = new CountDownLatch(threadSize);
        for (int i = 0; i < threadSize; i++) {
            Thread thread = new Thread(() -> {
                try {
                    for (int j = 0; j < 10; j++) {
                        request();
                    }
                } finally {
                    countDownLatch.countDown();
                }
            });
            thread.start();
        }

        countDownLatch.await();
        long endTime = System.currentTimeMillis();
        System.out.println(Thread.currentThread().getName() + "，耗时：" + (endTime - starTime) + ",count=" + count);
    }
}

```

输出：

```java
main，耗时：64,count=1000
```

monitorEnter、monitorExit都有1个参数，表示上锁的对象。用法和synchronized关键字语义类似。

注意：

1. **monitorEnter、monitorExit、tryMonitorEnter 3个方法已过期，不建议使用了**
2. **monitorEnter、monitorExit必须成对出现，出现的次数必须一致，也就是说锁了n次，也必须释放n次，否则会造成死锁**

## 普通读写

## volatile读写

java中操作内存分为主内存和工作内存，共享数据在主内存中，线程如果需要操作主内存的数据，需要先将主内存的数据复制到线程独有的工作内存中，操作完成之后再将其刷新到主内存中。如线程A要想看到线程B修改后的数据，需要满足：线程B修改数据之后，需要将数据从自己的工作内存中刷新到主内存中，并且A需要去主内存中读取数据。

被关键字volatile修饰的数据，有2点语义：

1. 如果一个变量被volatile修饰，读取这个变量时候，会强制从主内存中读取，然后将其复制到当前线程的工作内存中使用
2. 给volatile修饰的变量赋值的时候，会强制将赋值的结果从工作内存刷新到主内存

上面2点语义保证了被volatile修饰的数据在多线程中的可见性。

Unsafe中提供了和volatile语义一样的功能的方法，如下：

```java
//设置给定对象的int值，使用volatile语义，即设置后立马更新到内存对其他线程可见
public native void  putIntVolatile(Object o, long offset, int x);
//获得给定对象的指定偏移量offset的int值，使用volatile语义，总能获取到最新的int值。
public native int getIntVolatile(Object o, long offset);
```

putIntVolatile方法，2个参数：

> o：表示需要操作的对象
>
> offset：表示操作对象中的某个字段地址偏移量
>
> x：将offset对应的字段的值修改为x，并且立即刷新到主存中
>
> 调用这个方法，会强制将工作内存中修改的数据刷新到主内存中。

getIntVolatile方法，2个参数

> o：表示需要操作的对象
>
> offset：表示操作对象中的某个字段地址偏移量
>
> 每次调用这个方法都会强制从主内存读取值，将其复制到工作内存中使用。

其他的还有几个putXXXVolatile、getXXXVolatile方法和上面2个类似。

## Unsafe中Class相关方法

此部分主要提供Class和它的静态字段的操作相关方法，包含静态字段内存定位、定义类、定义匿名类、检验&确保初始化等。

```java
//获取给定静态字段的内存地址偏移量，这个值对于给定的字段是唯一且固定不变的
public native long staticFieldOffset(Field f);
//获取一个静态类中给定字段的对象指针
public native Object staticFieldBase(Field f);
//判断是否需要初始化一个类，通常在获取一个类的静态属性的时候（因为一个类如果没初始化，它的静态属性也不会初始化）使用。 当且仅当ensureClassInitialized方法不生效时返回false。
public native boolean shouldBeInitialized(Class<?> c);
//检测给定的类是否已经初始化。通常在获取一个类的静态属性的时候（因为一个类如果没初始化，它的静态属性也不会初始化）使用。
public native void ensureClassInitialized(Class<?> c);
//定义一个类，此方法会跳过JVM的所有安全检查，默认情况下，ClassLoader（类加载器）和ProtectionDomain（保护域）实例来源于调用者
public native Class<?> defineClass(String name, byte[] b, int off, int len, ClassLoader loader, ProtectionDomain protectionDomain);
//定义一个匿名类
public native Class<?> defineAnonymousClass(Class<?> hostClass, byte[] data, Object[] cpPatches);
```

### 示例：staticFieldOffset、staticFieldBase、staticFieldBase

```java
package com.itsoku.chat21;

import lombok.extern.slf4j.Slf4j;
import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.TimeUnit;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
@Slf4j
public class Demo7 {

    static Unsafe unsafe;
    //静态属性
    private static Object v1;
    //实例属性
    private Object v2;

    static {
        //获取Unsafe对象
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            unsafe = (Unsafe) field.get(null);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws NoSuchFieldException {
        Field v1Field = Demo7.class.getDeclaredField("v1");
        Field v2Field = Demo7.class.getDeclaredField("v2");

        System.out.println(unsafe.staticFieldOffset(v1Field));
        System.out.println(unsafe.objectFieldOffset(v2Field));

        System.out.println(unsafe.staticFieldBase(v1Field)==Demo7.class);
    }
}
```

输出：

```java
112
12
true
```

可以看出`staticFieldBase`返回的就是Demo2的class对象。

### 示例：shouldBeInitialized、ensureClassInitialized

```java
package com.itsoku.chat21;

import lombok.extern.slf4j.Slf4j;
import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.TimeUnit;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo8 {

    static Unsafe unsafe;

    static {
        //获取Unsafe对象
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            unsafe = (Unsafe) field.get(null);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static class C1 {
        private static int count;

        static {
            count = 10;
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "，C1 static init.");
        }
    }

    static class C2 {
        private static int count;

        static {
            count = 11;
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "，C2 static init.");
        }
    }

    public static void main(String[] args) throws NoSuchFieldException {
        //判断C1类是需要需要初始化，如果已经初始化了，会返回false，如果此类没有被初始化过，返回true
        if (unsafe.shouldBeInitialized(C1.class)) {
            System.out.println("C1需要进行初始化");
            //对C1进行初始化
            unsafe.ensureClassInitialized(C1.class);
        }

        System.out.println(C2.count);
        System.out.println(unsafe.shouldBeInitialized(C1.class));
    }
}
```

输出：

```csharp
C1需要进行初始化
1565069660679,main，C1 static init.
1565069660680,main，C2 static init.
11
false
```

代码中C1未被初始化过，所以`unsafe.shouldBeInitialized(C1.class)`返回true，然后调用`unsafe.ensureClassInitialized(C1.class)`进行初始化。

代码中执行`C2.count`会触发C2进行初始化，所以`shouldBeInitialized(C1.class)`返回false

## 对象操作的其他方法

```java
//返回对象成员属性在内存地址相对于此对象的内存地址的偏移量
public native long objectFieldOffset(Field f);
//获得给定对象的指定地址偏移量的值，与此类似操作还有：getInt，getDouble，getLong，getChar等
public native Object getObject(Object o, long offset);
//给定对象的指定地址偏移量设值，与此类似操作还有：putInt，putDouble，putLong，putChar等
public native void putObject(Object o, long offset, Object x);
//从对象的指定偏移量处获取变量的引用，使用volatile的加载语义
public native Object getObjectVolatile(Object o, long offset);
//存储变量的引用到对象的指定的偏移量处，使用volatile的存储语义
public native void putObjectVolatile(Object o, long offset, Object x);
//有序、延迟版本的putObjectVolatile方法，不保证值的改变被其他线程立即看到，只有在field被volatile修饰符修饰时有效
public native void putOrderedObject(Object o, long offset, Object x);
//绕过构造方法、初始化代码来创建对象
public native Object allocateInstance(Class<?> cls) throws InstantiationException;
```

`getObject`相当于获取对象中字段的值，`putObject`相当于给字段赋值，有兴趣的可以自己写个例子看看效果。

## 类加载

```
public native Class<?> defineClass(String var1, byte[] var2, int var3, int var4, ClassLoader var5, ProtectionDomain var6);

public native Class<?> defineAnonymousClass(Class<?> var1, byte[] var2, Object[] var3);

public native Object allocateInstance(Class<?> var1) throws InstantiationException;

public native boolean shouldBeInitialized(Class<?> var1);

public native void ensureClassInitialized(Class<?> var1);
```



介绍一下`allocateInstance`，这个方法可以绕过构造方法来创建对象，示例代码如下：

```java
package com.itsoku.chat21;

import sun.misc.Unsafe;

import java.lang.reflect.Field;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo9 {

    static Unsafe unsafe;

    static {
        //获取Unsafe对象
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            unsafe = (Unsafe) field.get(null);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static class C1 {
        private String name;

        private C1() {
            System.out.println("C1 default constructor!");
        }

        private C1(String name) {
            this.name = name;
            System.out.println("C1 有参 constructor!");
        }
    }

    public static void main(String[] args) throws InstantiationException {
        System.out.println(unsafe.allocateInstance(C1.class));
    }
}
```

输出：

```java
com.itsoku.chat21.Demo9$C1@782830e
```

看一下类C1中有两个构造方法，都是private的，通过new、反射的方式都无法创建对象。但是可以通过Unsafe的allocateInstance方法绕过构造函数来创建C1的实例，输出的结果中可以看出创建成功了，并且没有调用构造方法。

#### 典型应用

- **常规对象实例化方式**：我们通常所用到的创建对象的方式，从本质上来讲，都是通过new机制来实现对象的创建。但是，new机制有个特点就是当类只提供有参的构造函数且无显示声明无参构造函数时，则必须使用有参构造函数进行对象构造，而使用有参构造函数时，必须传递相应个数的参数才能完成对象实例化。
- **非常规的实例化方式**：而Unsafe中提供allocateInstance方法，仅通过Class对象就可以创建此类的实例对象，而且不需要调用其构造函数、初始化代码、JVM安全检查等。它抑制修饰符检测，也就是即使构造器是private修饰的也能通过此方法实例化，只需提类对象即可创建相应的对象。由于这种特性，allocateInstance在java.lang.invoke、Objenesis（提供绕过类构造器的对象生成方式）、Gson（反序列化时用到）中都有相应的应用。

## 数组相关的一些方法

这部分主要介绍与数据操作相关的arrayBaseOffset与arrayIndexScale这两个方法，两者配合起来使用，即可定位数组中每个元素在内存中的位置。

```java
//返回数组中第一个元素的偏移地址
public native int arrayBaseOffset(Class<?> arrayClass);
//返回数组中一个元素占用的大小
public native int arrayIndexScale(Class<?> arrayClass);
```

这两个与数据操作相关的方法，在java.util.concurrent.atomic 包下的AtomicIntegerArray（可以实现对Integer数组中每个元素的原子性操作）中有典型的应用，如下图AtomicIntegerArray源码所示，通过Unsafe的arrayBaseOffset、arrayIndexScale分别获取数组首元素的偏移地址base及单个元素大小因子scale。后续相关原子性操作，均依赖于这两个值进行数组中元素的定位，如下图二所示的getAndAdd方法即通过checkedByteOffset方法获取某数组元素的偏移地址，而后通过CAS实现原子性操作。

**数组元素定位：**

Unsafe类中有很多以BASE_OFFSET结尾的常量，比如ARRAY_INT_BASE_OFFSET，ARRAY_BYTE_BASE_OFFSET等，这些常量值是通过arrayBaseOffset方法得到的。arrayBaseOffset方法是一个本地方法，可以获取数组第一个元素的偏移地址。Unsafe类中还有很多以INDEX_SCALE结尾的常量，比如 ARRAY_INT_INDEX_SCALE ， ARRAY_BYTE_INDEX_SCALE等，这些常量值是通过arrayIndexScale方法得到的。arrayIndexScale方法也是一个本地方法，可以获取数组的转换因子，也就是数组中元素的增量地址。将arrayBaseOffset与arrayIndexScale配合使用，可以定位数组中每个元素在内存中的位置。

## 内存管理

Unsafe类中存在直接操作内存的方法

```
//分配内存指定大小的内存
public native long allocateMemory(long bytes);
//根据给定的内存地址address设置重新分配指定大小的内存
public native long reallocateMemory(long address, long bytes);
//用于释放allocateMemory和reallocateMemory申请的内存
public native void freeMemory(long address);
//将指定对象的给定offset偏移量内存块中的所有字节设置为固定值
public native void setMemory(Object o, long offset, long bytes, byte value);
//设置给定内存地址的值
public native void putAddress(long address, long x);
//获取指定内存地址的值
public native long getAddress(long address);

//设置给定内存地址的long值
public native void putLong(long address, long x);
//获取指定内存地址的long值
public native long getLong(long address);
//设置或获取指定内存的byte值
public native byte  getByte(long address);
public native void  putByte(long address, byte x);
//其他基本数据类型(long,char,float,double,short等)的操作与putByte及getByte相同

//操作系统的内存页大小
public native int pageSize();

```



## 内存屏障

在Java 8中引入，用于定义内存屏障（也称内存栅栏，内存栅障，屏障指令等，是一类同步屏障指令，是CPU或编译器在对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作），避免代码重排序。

```java
//内存屏障，禁止load操作重排序。屏障前的load操作不能被重排序到屏障后，屏障后的load操作不能被重排序到屏障前
public native void loadFence();
//内存屏障，禁止store操作重排序。屏障前的store操作不能被重排序到屏障后，屏障后的store操作不能被重排序到屏障前
public native void storeFence();
//内存屏障，禁止load、store操作重排序
public native void fullFence();
```

Unsafe相关的就介绍这么多！

# Atomic（JUC中原子类）

## 本文主要内容

1. JUC中的原子类介绍
2. 介绍基本类型原子类
3. 介绍数组类型原子类
4. 介绍引用类型原子类
5. 介绍对象属性修改相关原子类

## 预备知识

JUC中的原子类都是都是依靠**volatile**、**CAS**、**Unsafe**类配合来实现的，需要了解的请移步：

[volatile与Java内存模型](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933088&idx=1&sn=f1d666dd799664b1989c77441b9d12c5&chksm=88621adebf1593c83501ac33d6a0e0de075f2b2e30caf986cf276cbb1c8dff0eac2a0a648b1d&token=2041017112&lang=zh_CN&scene=21#wechat_redirect)

[java中的CAS](http://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933166&idx=1&sn=15e614500676170b76a329efd3255c12&chksm=88621b10bf1592064befc5c9f0d78c56cda25c6d003e1711b85e5bfeb56c9fd30d892178db87&scene=21#wechat_redirect)

[JUC底层工具类Unsafe](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933173&idx=1&sn=80eb550294677b0042fc030f90cce109&chksm=88621b0bbf15921d2274a7bf6afde912fec02a4c3ade9cfb50d03cdce73e07e33d08d35a3b27&token=1771071046&lang=zh_CN#rd)

## JUC中原子类介绍

**什么是原子操作？**

**atomic** 翻译成中文是原子的意思。在化学上，我们知道原子是构成一般物质的最小单位，在化学反应中是不可分割的。**在我们这里 atomic 是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰，所以，所谓原子类说简单点就是具有原子操作特征的类，原子操作类提供了一些修改数据的方法，这些方法都是原子操作的，在多线程情况下可以确保被修改数据的正确性**。

JUC中对原子操作提供了强大的支持，这些类位于**java.util.concurrent.atomic**包中，如下图：

![img](https://img2018.cnblogs.com/blog/687624/201908/687624-20190807151213130-1588807953.png)

## JUC中原子类思维导图

![img](https://img2018.cnblogs.com/blog/687624/201908/687624-20190807151234004-748322976.png)

## 基本类型原子类

使用原子的方式更新基本类型

- AtomicInteger：int类型原子类
- AtomicLong：long类型原子类
- AtomicBoolean ：boolean类型原子类

上面三个类提供的方法几乎相同，这里以 AtomicInteger 为例子来介绍。

**AtomicInteger 类常用方法**

```java
public final int get() //获取当前的值
public final int getAndSet(int newValue)//获取当前的值，并设置新的值
public final int getAndIncrement()//获取当前的值，并自增
public final int getAndDecrement() //获取当前的值，并自减
public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
public final void lazySet(int newValue)//最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
```

**部分源码**

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;
```

> 2个关键字段说明：
> **value**：使用volatile修饰，可以确保value在多线程中的可见性。
> **valueOffset**：value属性在AtomicInteger中的偏移量，通过这个偏移量可以快速定位到value字段，这个是实现AtomicInteger的关键。

**getAndIncrement源码：**

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

内部调用的是**Unsafe**类中的**getAndAddInt**方法，我们看一下**getAndAddInt**源码：

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

> 说明：
> this.getIntVolatile：可以确保从主内存中获取变量最新的值。
>
> compareAndSwapInt：CAS操作，CAS的原理是拿期望的值和原本的值作比较，如果相同则更新成新的值，可以确保在多线程情况下只有一个线程会操作成功，不成功的返回false。
>
> 上面有个do-while循环，compareAndSwapInt返回false之后，会再次从主内存中获取变量的值，继续做CAS操作，直到成功为止。
>
> getAndAddInt操作相当于线程安全的count++操作，如同：
> synchronize(lock){
> count++;
> }
> count++操作实际上是被拆分为3步骤执行：
>
> 1. 获取count的值，记做A：A=count
> 2. 将A的值+1，得到B：B = A+1
> 3. 让B赋值给count：count = B
>    多线程情况下会出现线程安全的问题，导致数据不准确。
>
> synchronize的方式会导致占时无法获取锁的线程处于阻塞状态，性能比较低。CAS的性能比synchronize要快很多。

**示例**

> 使用AtomicInteger实现网站访问量计数器功能，模拟100人同时访问网站，每个人访问10次，代码如下：

```java
package com.itsoku.chat23;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo1 {
    //访问次数
    static AtomicInteger count = new AtomicInteger();

    //模拟访问一次
    public static void request() throws InterruptedException {
        //模拟耗时5毫秒
        TimeUnit.MILLISECONDS.sleep(5);
        //对count原子+1
        count.incrementAndGet();
    }

    public static void main(String[] args) throws InterruptedException {
        long starTime = System.currentTimeMillis();
        int threadSize = 100;
        CountDownLatch countDownLatch = new CountDownLatch(threadSize);
        for (int i = 0; i < threadSize; i++) {
            Thread thread = new Thread(() -> {
                try {
                    for (int j = 0; j < 10; j++) {
                        request();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();
                }
            });
            thread.start();
        }

        countDownLatch.await();
        long endTime = System.currentTimeMillis();
        System.out.println(Thread.currentThread().getName() + "，耗时：" + (endTime - starTime) + ",count=" + count);
    }
}
```

输出：

```java
main，耗时：158,count=1000
```

通过输出中可以看出`incrementAndGet`在多线程情况下能确保数据的正确性。

## 数组类型原子类介绍

使用原子的方式更新数组里的某个元素，可以确保修改数组中数据的线程安全性。

- AtomicIntegerArray：整形数组原子操作类
- AtomicLongArray：长整形数组原子操作类
- AtomicReferenceArray ：引用类型数组原子操作类

上面三个类提供的方法几乎相同，所以我们这里以 AtomicIntegerArray 为例子来介绍。

**AtomicIntegerArray 类常用方法**

```java
public final int get(int i) //获取 index=i 位置元素的值
public final int getAndSet(int i, int newValue)//返回 index=i 位置的当前的值，并将其设置为新值：newValue
public final int getAndIncrement(int i)//获取 index=i 位置元素的值，并让该位置的元素自增
public final int getAndDecrement(int i) //获取 index=i 位置元素的值，并让该位置的元素自减
public final int getAndAdd(int delta) //获取 index=i 位置元素的值，并加上预期的值
boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将 index=i 位置的元素值设置为输入值（update）
public final void lazySet(int i, int newValue)//最终 将index=i 位置的元素设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
```

**示例**

> 统计网站页面访问量，假设网站有10个页面，现在模拟100个人并行访问每个页面10次，然后将每个页面访问量输出，应该每个页面都是1000次，代码如下：

```java
package com.itsoku.chat23;

import java.util.Arrays;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicIntegerArray;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo2 {

    static AtomicIntegerArray pageRequest = new AtomicIntegerArray(new int[10]);

    /**
     * 模拟访问一次
     *
     * @param page 访问第几个页面
     * @throws InterruptedException
     */
    public static void request(int page) throws InterruptedException {
        //模拟耗时5毫秒
        TimeUnit.MILLISECONDS.sleep(5);
        //pageCountIndex为pageCount数组的下标，表示页面对应数组中的位置
        int pageCountIndex = page - 1;
        pageRequest.incrementAndGet(pageCountIndex);
    }

    public static void main(String[] args) throws InterruptedException {
        long starTime = System.currentTimeMillis();
        int threadSize = 100;
        CountDownLatch countDownLatch = new CountDownLatch(threadSize);
        for (int i = 0; i < threadSize; i++) {
            Thread thread = new Thread(() -> {
                try {

                    for (int page = 1; page <= 10; page++) {
                        for (int j = 0; j < 10; j++) {
                            request(page);
                        }
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();
                }
            });
            thread.start();
        }

        countDownLatch.await();
        long endTime = System.currentTimeMillis();
        System.out.println(Thread.currentThread().getName() + "，耗时：" + (endTime - starTime));

        for (int pageIndex = 0; pageIndex < 10; pageIndex++) {
            System.out.println("第" + (pageIndex + 1) + "个页面访问次数为" + pageRequest.get(pageIndex));
        }
    }
}
```

输出：

```java
main，耗时：635
第1个页面访问次数为1000
第2个页面访问次数为1000
第3个页面访问次数为1000
第4个页面访问次数为1000
第5个页面访问次数为1000
第6个页面访问次数为1000
第7个页面访问次数为1000
第8个页面访问次数为1000
第9个页面访问次数为1000
第10个页面访问次数为1000
```

说明：

> 代码中将10个面的访问量放在了一个int类型的数组中，数组大小为10，然后通过`AtomicIntegerArray`来操作数组中的每个元素，可以确保操作数据的原子性，每次访问会调用`incrementAndGet`，此方法需要传入数组的下标，然后对指定的元素做原子+1操作。输出结果都是1000，可以看出对于数组中元素的并发修改是线程安全的。如果线程不安全，则部分数据可能会小于1000。

其他的一些方法可以自行操作一下，都非常简单。

## 引用类型原子类介绍

基本类型原子类只能更新一个变量，如果需要原子更新多个变量，需要使用 引用类型原子类。

- **AtomicReference**：引用类型原子类
- **AtomicStampedRerence**：原子更新引用类型里的字段原子类
- **AtomicMarkableReference** ：原子更新带有标记位的引用类型

**AtomicReference** 和 **AtomicInteger** 非常类似，不同之处在于 **AtomicInteger**是对整数的封装，而**AtomicReference**则是对应普通的对象引用，它可以确保你在修改对象引用时的线程安全性。在介绍**AtomicReference**的同时，我们先来了解一个有关原子操作逻辑上的不足。

### ABA问题

之前我们说过，线程判断被修改对象是否可以正确写入的条件是对象的当前值和期望值是否一致。这个逻辑从一般意义上来说是正确的，但是可能出现一个小小的例外，就是当你获得当前数据后，在准备修改为新值钱，对象的值被其他线程连续修改了两次，而经过这2次修改后，对象的值又恢复为旧值，这样，当前线程就无法正确判断这个对象究竟是否被修改过，这就是所谓的ABA问题，可能会引发一些问题。

**举个例子**

有一家蛋糕店，为了挽留客户，决定为贵宾卡客户一次性赠送20元，刺激客户充值和消费，但条件是，每一位客户只能被赠送一次，现在我们用`AtomicReference`来实现这个功能，代码如下：

```java
package com.itsoku.chat22;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo3 {
    //账户原始余额
    static int accountMoney = 19;
    //用于对账户余额做原子操作
    static AtomicReference<Integer> money = new AtomicReference<>(accountMoney);

    /**
     * 模拟2个线程同时更新后台数据库，为用户充值
     */
    static void recharge() {
        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                for (int j = 0; j < 5; j++) {
                    Integer m = money.get();
                    if (m == accountMoney) {
                        if (money.compareAndSet(m, m + 20)) {
                            System.out.println("当前余额：" + m + "，小于20，充值20元成功，余额：" + money.get() + "元");
                        }
                    }
                    //休眠100ms
                    try {
                        TimeUnit.MILLISECONDS.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }

    /**
     * 模拟用户消费
     */
    static void consume() throws InterruptedException {
        for (int i = 0; i < 5; i++) {
            Integer m = money.get();
            if (m > 20) {
                if (money.compareAndSet(m, m - 20)) {
                    System.out.println("当前余额：" + m + "，大于10，成功消费10元，余额：" + money.get() + "元");
                }
            }
            //休眠50ms
            TimeUnit.MILLISECONDS.sleep(50);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        recharge();
        consume();
    }

}
```

输出：

```undefined
当前余额：19，小于20，充值20元成功，余额：39元
当前余额：39，大于10，成功消费10元，余额：19元
当前余额：19，小于20，充值20元成功，余额：39元
当前余额：39，大于10，成功消费10元，余额：19元
当前余额：19，小于20，充值20元成功，余额：39元
当前余额：39，大于10，成功消费10元，余额：19元
当前余额：19，小于20，充值20元成功，余额：39元
```

> 从输出中可以看到，这个账户被先后反复多次充值。其原因是账户余额被反复修改，修改后的值和原有的数值19一样，使得CAS操作无法正确判断当前数据是否被修改过（是否被加过20）。虽然这种情况出现的概率不大，但是依然是有可能出现的，因此，当业务上确实可能出现这种情况时，我们必须多加防范。JDK也为我们考虑到了这种情况，使用`AtomicStampedReference`可以很好地解决这个问题。

### 使用AtomicStampedRerence解决ABA的问题

`AtomicReference`无法解决上述问题的根本原因是，对象在被修改过程中丢失了状态信息，比如充值20元的时候，需要同时标记一个状态，用来标注用户被充值过。因此我们只要能够记录对象在修改过程中的状态值，就可以很好地解决对象被反复修改导致线程无法正确判断对象状态的问题。

`AtomicStampedRerence`正是这么做的，他内部不仅维护了对象的值，还维护了一个时间戳（我们这里把他称为时间戳，实际上它可以使用任何一个整形来表示状态值），当AtomicStampedRerence对应的数值被修改时，除了更新数据本身外，还必须要更新时间戳。当AtomicStampedRerence设置对象值时，对象值及时间戳都必须满足期望值，写入才会成功。因此，即使对象值被反复读写，写回原值，只要时间戳发生变量，就能防止不恰当的写入。

`AtomicStampedRerence`的几个Api在`AtomicReference`的基础上新增了有关时间戳的信息。

```java
//比较设置，参数依次为：期望值、写入新值、期望时间戳、新时间戳
public boolean compareAndSet(V expectedReference, V newReference, int expectedStamp, int newStamp);
//获得当前对象引用
public V getReference();
//获得当前时间戳
public int getStamp();
//设置当前对象引用和时间戳
public void set(V newReference, int newStamp);
```

现在我们使用`AtomicStampedRerence`来修改一下上面充值的问题，代码如下：

```java
package com.itsoku.chat22;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;
import java.util.concurrent.atomic.AtomicStampedReference;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo4 {
    //账户原始余额
    static int accountMoney = 19;
    //用于对账户余额做原子操作
    static AtomicStampedReference<Integer> money = new AtomicStampedReference<>(accountMoney, 0);

    /**
     * 模拟2个线程同时更新后台数据库，为用户充值
     */
    static void recharge() {
        for (int i = 0; i < 2; i++) {
            int stamp = money.getStamp();
            new Thread(() -> {
                for (int j = 0; j < 50; j++) {
                    Integer m = money.getReference();
                    if (m == accountMoney) {
                        if (money.compareAndSet(m, m + 20, stamp, stamp + 1)) {
                            System.out.println("当前时间戳：" + money.getStamp() + ",当前余额：" + m + "，小于20，充值20元成功，余额：" + money.getReference() + "元");
                        }
                    }
                    //休眠100ms
                    try {
                        TimeUnit.MILLISECONDS.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }

    /**
     * 模拟用户消费
     */
    static void consume() throws InterruptedException {
        for (int i = 0; i < 50; i++) {
            Integer m = money.getReference();
            int stamp = money.getStamp();
            if (m > 20) {
                if (money.compareAndSet(m, m - 20, stamp, stamp + 1)) {
                    System.out.println("当前时间戳：" + money.getStamp() + ",当前余额：" + m + "，大于10，成功消费10元，余额：" + money.getReference() + "元");
                }
            }
            //休眠50ms
            TimeUnit.MILLISECONDS.sleep(50);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        recharge();
        consume();
    }

}
```

输出：

```java
当前时间戳：1,当前余额：19，小于20，充值20元成功，余额：39元
当前时间戳：2,当前余额：39，大于10，成功消费10元，余额：19元
```

结果正常了。

**关于这个时间戳的，在数据库修改数据中也有类似的用法，比如2个编辑同时编辑一篇文章，同时提交，只允许一个用户提交成功，提示另外一个用户：博客已被其他人修改，如何实现呢？**

> 博客表：t_blog（id,content,stamp)，stamp默认值为0，每次更新+1
>
> A、B 二个编辑同时对一篇文章进行编辑，stamp都为0，当点击提交的时候，将stamp和id作为条件更新博客内容，执行的sql如下：
>
> **update t_blog set content = 更新的内容,stamp = stamp+1 where id = 博客id and stamp = 0;**
>
> 这条update会返回影响的行数，只有一个会返回1，表示更新成功，另外一个提交者返回0，表示需要修改的数据已经不满足条件了，被其他用户给修改了。这种修改数据的方式也叫乐观锁。

## 对象的属性修改原子类介绍

如果需要原子更新某个类里的某个字段时，需要用到对象的属性修改原子类。

- AtomicIntegerFieldUpdater：原子更新整形字段的值
- AtomicLongFieldUpdater：原子更新长整形字段的值
- AtomicReferenceFieldUpdater ：原子更新应用类型字段的值

要想原子地更新对象的属性需要两步：

1. 第一步，因为对象的属性修改类型原子类都是抽象类，所以每次使用都必须使用静态方法 newUpdater()创建一个更新器，并且需要设置想要更新的类和属性。
2. 第二步，更新的对象属性必须使用 public volatile 修饰符。

上面三个类提供的方法几乎相同，所以我们这里以`AtomicReferenceFieldUpdater`为例子来介绍。

调用`AtomicReferenceFieldUpdater`静态方法`newUpdater`创建`AtomicReferenceFieldUpdater`对象

```java
public static <U,W> AtomicReferenceFieldUpdater<U,W> newUpdater(Class<U> tclass,
                                                                    Class<W> vclass,
                                                                    String fieldName) 
```

说明:

> 三个参数
>
> tclass：需要操作的字段所在的类
> vclass：操作字段的类型
> fieldName：字段名称

**示例**

> 多线程并发调用一个类的初始化方法，如果未被初始化过，将执行初始化工作，要求只能初始化一次

代码如下：

```java
package com.itsoku.chat22;

import com.sun.org.apache.xpath.internal.operations.Bool;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReferenceFieldUpdater;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo5 {

    static Demo5 demo5 = new Demo5();
    //isInit用来标注是否被初始化过
    volatile Boolean isInit = Boolean.FALSE;
    AtomicReferenceFieldUpdater<Demo5, Boolean> updater = AtomicReferenceFieldUpdater.newUpdater(Demo5.class, Boolean.class, "isInit");

    /**
     * 模拟初始化工作
     *
     * @throws InterruptedException
     */
    public void init() throws InterruptedException {
        //isInit为false的时候，才进行初始化，并将isInit采用原子操作置为true
        if (updater.compareAndSet(demo5, Boolean.FALSE, Boolean.TRUE)) {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "，开始初始化!");
            //模拟休眠3秒
            TimeUnit.SECONDS.sleep(3);
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "，初始化完毕!");
        } else {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + "，有其他线程已经执行了初始化!");
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                try {
                    demo5.init();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

输出：

```java
1565159962098,Thread-0，开始初始化!
1565159962098,Thread-3，有其他线程已经执行了初始化!
1565159962098,Thread-4，有其他线程已经执行了初始化!
1565159962098,Thread-2，有其他线程已经执行了初始化!
1565159962098,Thread-1，有其他线程已经执行了初始化!
1565159965100,Thread-0，初始化完毕!
```

说明：

> 1. isInit属性必须要volatille修饰，可以确保变量的可见性
> 2. 可以看出多线程同时执行`init()`方法，只有一个线程执行了初始化的操作，其他线程跳过了。多个线程同时到达`updater.compareAndSet`，只有一个会成功。

# ThreadLocal、InheritableThreadLocal（通俗易懂）

## 本文内容

1. 需要解决的问题
2. 介绍ThreadLocal
3. 介绍InheritableThreadLocal

## 需要解决的问题

> 我们还是以解决问题的方式来引出`ThreadLocal`、`InheritableThreadLocal`，这样印象会深刻一些。

目前java开发web系统一般有3层，controller、service、dao，请求到达controller，controller调用service，service调用dao，然后进行处理。

我们写一个简单的例子，有3个方法分别模拟controller、service、dao。代码如下：

```java
package com.itsoku.chat24;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo1 {

    static AtomicInteger threadIndex = new AtomicInteger(1);
    //创建处理请求的线程池子
    static ThreadPoolExecutor disposeRequestExecutor = new ThreadPoolExecutor(3,
            3,
            60,
            TimeUnit.SECONDS,
            new LinkedBlockingDeque<>(),
            r -> {
                Thread thread = new Thread(r);
                thread.setName("disposeRequestThread-" + threadIndex.getAndIncrement());
                return thread;
            });

    //记录日志
    public static void log(String msg) {
        StackTraceElement stack[] = (new Throwable()).getStackTrace();
        System.out.println("****" + System.currentTimeMillis() + ",[线程:" + Thread.currentThread().getName() + "]," + stack[1] + ":" + msg);
    }

    //模拟controller
    public static void controller(List<String> dataList) {
        log("接受请求");
        service(dataList);
    }

    //模拟service
    public static void service(List<String> dataList) {
        log("执行业务");
        dao(dataList);
    }

    //模拟dao
    public static void dao(List<String> dataList) {
        log("执行数据库操作");
        //模拟插入数据
        for (String s : dataList) {
            log("插入数据" + s + "成功");
        }
    }

    public static void main(String[] args) {
        //需要插入的数据
        List<String> dataList = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            dataList.add("数据" + i);
        }

        //模拟5个请求
        int requestCount = 5;
        for (int i = 0; i < requestCount; i++) {
            disposeRequestExecutor.execute(() -> {
                controller(dataList);
            });
        }

        disposeRequestExecutor.shutdown();
    }
}
```

运行结果：

```java
****1565338891286,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.controller(Demo1.java:36):接受请求
****1565338891286,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.controller(Demo1.java:36):接受请求
****1565338891287,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.service(Demo1.java:42):执行业务
****1565338891287,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.service(Demo1.java:42):执行业务
****1565338891287,[线程:disposeRequestThread-3],com.itsoku.chat24.Demo1.controller(Demo1.java:36):接受请求
****1565338891287,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.dao(Demo1.java:48):执行数据库操作
****1565338891287,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据0成功
****1565338891287,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据1成功
****1565338891287,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.dao(Demo1.java:48):执行数据库操作
****1565338891287,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据2成功
****1565338891287,[线程:disposeRequestThread-3],com.itsoku.chat24.Demo1.service(Demo1.java:42):执行业务
****1565338891288,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.controller(Demo1.java:36):接受请求
****1565338891287,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据0成功
****1565338891288,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.service(Demo1.java:42):执行业务
****1565338891288,[线程:disposeRequestThread-3],com.itsoku.chat24.Demo1.dao(Demo1.java:48):执行数据库操作
****1565338891288,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.dao(Demo1.java:48):执行数据库操作
****1565338891288,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据1成功
****1565338891288,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据0成功
****1565338891288,[线程:disposeRequestThread-3],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据0成功
****1565338891288,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据1成功
****1565338891288,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据2成功
****1565338891288,[线程:disposeRequestThread-1],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据2成功
****1565338891288,[线程:disposeRequestThread-3],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据1成功
****1565338891288,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.controller(Demo1.java:36):接受请求
****1565338891288,[线程:disposeRequestThread-3],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据2成功
****1565338891288,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.service(Demo1.java:42):执行业务
****1565338891289,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.dao(Demo1.java:48):执行数据库操作
****1565338891289,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据0成功
****1565338891289,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据1成功
****1565338891289,[线程:disposeRequestThread-2],com.itsoku.chat24.Demo1.dao(Demo1.java:51):插入数据数据2成功
```

代码中调用controller、service、dao 3个方法时，来模拟处理一个请求。main方法中循环了5次模拟发起5次请求，然后交给线程池去处理请求，dao中模拟循环插入传入的dataList数据。

**问题来了：开发者想看一下哪些地方耗时比较多，想通过日志来分析耗时情况，想追踪某个请求的完整日志，怎么搞？**

上面的请求采用线程池的方式处理的，多个请求可能会被一个线程处理，通过日志很难看出那些日志是同一个请求，我们能不能给请求加一个唯一标志，日志中输出这个唯一标志，当然可以。

如果我们的代码就只有上面示例这么简单，我想还是很容易的，上面就3个方法，给每个方法加个traceId参数，log方法也加个traceId参数，就解决了，代码如下：

```java
package com.itsoku.chat24;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo2 {

    static AtomicInteger threadIndex = new AtomicInteger(1);
    //创建处理请求的线程池子
    static ThreadPoolExecutor disposeRequestExecutor = new ThreadPoolExecutor(3,
            3,
            60,
            TimeUnit.SECONDS,
            new LinkedBlockingDeque<>(),
            r -> {
                Thread thread = new Thread(r);
                thread.setName("disposeRequestThread-" + threadIndex.getAndIncrement());
                return thread;
            });

    //记录日志
    public static void log(String msg, String traceId) {
        StackTraceElement stack[] = (new Throwable()).getStackTrace();
        System.out.println("****" + System.currentTimeMillis() + "[traceId:" + traceId + "],[线程:" + Thread.currentThread().getName() + "]," + stack[1] + ":" + msg);
    }

    //模拟controller
    public static void controller(List<String> dataList, String traceId) {
        log("接受请求", traceId);
        service(dataList, traceId);
    }

    //模拟service
    public static void service(List<String> dataList, String traceId) {
        log("执行业务", traceId);
        dao(dataList, traceId);
    }

    //模拟dao
    public static void dao(List<String> dataList, String traceId) {
        log("执行数据库操作", traceId);
        //模拟插入数据
        for (String s : dataList) {
            log("插入数据" + s + "成功", traceId);
        }
    }

    public static void main(String[] args) {
        //需要插入的数据
        List<String> dataList = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            dataList.add("数据" + i);
        }

        //模拟5个请求
        int requestCount = 5;
        for (int i = 0; i < requestCount; i++) {
            String traceId = String.valueOf(i);
            disposeRequestExecutor.execute(() -> {
                controller(dataList, traceId);
            });
        }

        disposeRequestExecutor.shutdown();
    }
}
```

输出：

```java
****1565339559773[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.controller(Demo2.java:36):接受请求
****1565339559773[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.controller(Demo2.java:36):接受请求
****1565339559773[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo2.controller(Demo2.java:36):接受请求
****1565339559774[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.service(Demo2.java:42):执行业务
****1565339559774[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.service(Demo2.java:42):执行业务
****1565339559774[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.dao(Demo2.java:48):执行数据库操作
****1565339559774[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo2.service(Demo2.java:42):执行业务
****1565339559774[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据0成功
****1565339559774[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.dao(Demo2.java:48):执行数据库操作
****1565339559774[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据1成功
****1565339559774[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo2.dao(Demo2.java:48):执行数据库操作
****1565339559774[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据2成功
****1565339559774[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据0成功
****1565339559775[traceId:3],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.controller(Demo2.java:36):接受请求
****1565339559775[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据0成功
****1565339559775[traceId:3],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.service(Demo2.java:42):执行业务
****1565339559775[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据1成功
****1565339559775[traceId:3],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.dao(Demo2.java:48):执行数据库操作
****1565339559775[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据1成功
****1565339559775[traceId:3],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据0成功
****1565339559775[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据2成功
****1565339559775[traceId:3],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据1成功
****1565339559775[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据2成功
****1565339559775[traceId:3],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据2成功
****1565339559775[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.controller(Demo2.java:36):接受请求
****1565339559776[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.service(Demo2.java:42):执行业务
****1565339559776[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.dao(Demo2.java:48):执行数据库操作
****1565339559776[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据0成功
****1565339559776[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据1成功
****1565339559776[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo2.dao(Demo2.java:51):插入数据数据2成功
```

上面我们通过修改代码的方式，把问题解决了，但前提是你们的系统都像上面这么简单，功能很少，需要改的代码很少，可以这么去改。但事与愿违，我们的系统一般功能都是比较多的，如果我们都一个个去改，岂不是要疯掉，改代码还涉及到重新测试，风险也不可控。那有什么好办法么？

## ThreadLocal

还是拿上面的问题，我们来分析一下，每个请求都是由一个线程处理的，线程就相当于一个人一样，每个请求相当于一个任务，任务来了，人来处理，处理完毕之后，再处理下一个请求任务。人身上是不是有很多口袋，人刚开始准备处理任务的时候，我们把任务的编号放在处理者的口袋中，然后处理中一路携带者，处理过程中如果需要用到这个编号，直接从口袋中获取就可以了。那么刚好java中线程设计的时候也考虑到了这些问题，Thread对象中就有很多口袋，用来放东西。Thread类中有这么一个变量：

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
```

这个就是用来操作Thread中所有口袋的东西，`ThreadLocalMap`源码中有一个数组（有兴趣的可以去看一下源码），对应处理者身上很多口袋一样，数组中的每个元素对应一个口袋。

如何来操作Thread中的这些口袋呢，java为我们提供了一个类`ThreadLocal`，ThreadLocal对象用来操作Thread中的某一个口袋，可以向这个口袋中放东西、获取里面的东西、清除里面的东西，这个口袋一次性只能放一个东西，重复放东西会将里面已经存在的东西覆盖掉。

常用的3个方法：

```java
//向Thread中某个口袋中放东西
public void set(T value);
//获取这个口袋中目前放的东西
public T get();
//清空这个口袋中放的东西
public void remove()
```

我们使用ThreadLocal来改造一下上面的代码，如下：

```java
package com.itsoku.chat24;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo3 {

    //创建一个操作Thread中存放请求任务追踪id口袋的对象
    static ThreadLocal<String> traceIdKD = new ThreadLocal<>();

    static AtomicInteger threadIndex = new AtomicInteger(1);
    //创建处理请求的线程池子
    static ThreadPoolExecutor disposeRequestExecutor = new ThreadPoolExecutor(3,
            3,
            60,
            TimeUnit.SECONDS,
            new LinkedBlockingDeque<>(),
            r -> {
                Thread thread = new Thread(r);
                thread.setName("disposeRequestThread-" + threadIndex.getAndIncrement());
                return thread;
            });

    //记录日志
    public static void log(String msg) {
        StackTraceElement stack[] = (new Throwable()).getStackTrace();
        //获取当前线程存放tranceId口袋中的内容
        String traceId = traceIdKD.get();
        System.out.println("****" + System.currentTimeMillis() + "[traceId:" + traceId + "],[线程:" + Thread.currentThread().getName() + "]," + stack[1] + ":" + msg);
    }

    //模拟controller
    public static void controller(List<String> dataList) {
        log("接受请求");
        service(dataList);
    }

    //模拟service
    public static void service(List<String> dataList) {
        log("执行业务");
        dao(dataList);
    }

    //模拟dao
    public static void dao(List<String> dataList) {
        log("执行数据库操作");
        //模拟插入数据
        for (String s : dataList) {
            log("插入数据" + s + "成功");
        }
    }

    public static void main(String[] args) {
        //需要插入的数据
        List<String> dataList = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            dataList.add("数据" + i);
        }

        //模拟5个请求
        int requestCount = 5;
        for (int i = 0; i < requestCount; i++) {
            String traceId = String.valueOf(i);
            disposeRequestExecutor.execute(() -> {
                //把traceId放入口袋中
                traceIdKD.set(traceId);
                try {
                    controller(dataList);
                } finally {
                    //将tranceId从口袋中移除
                    traceIdKD.remove();
                }
            });
        }

        disposeRequestExecutor.shutdown();
    }
}
```

输出：

```java
****1565339644214[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo3.controller(Demo3.java:41):接受请求
****1565339644214[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.controller(Demo3.java:41):接受请求
****1565339644214[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.controller(Demo3.java:41):接受请求
****1565339644214[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.service(Demo3.java:47):执行业务
****1565339644214[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo3.service(Demo3.java:47):执行业务
****1565339644214[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.dao(Demo3.java:53):执行数据库操作
****1565339644214[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.service(Demo3.java:47):执行业务
****1565339644214[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据0成功
****1565339644214[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.dao(Demo3.java:53):执行数据库操作
****1565339644214[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo3.dao(Demo3.java:53):执行数据库操作
****1565339644215[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据0成功
****1565339644215[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据1成功
****1565339644215[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据1成功
****1565339644215[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据0成功
****1565339644215[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据2成功
****1565339644215[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据2成功
****1565339644215[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据1成功
****1565339644215[traceId:4],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.controller(Demo3.java:41):接受请求
****1565339644215[traceId:3],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.controller(Demo3.java:41):接受请求
****1565339644215[traceId:4],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.service(Demo3.java:47):执行业务
****1565339644215[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据2成功
****1565339644215[traceId:4],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.dao(Demo3.java:53):执行数据库操作
****1565339644215[traceId:3],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.service(Demo3.java:47):执行业务
****1565339644215[traceId:4],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据0成功
****1565339644215[traceId:3],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.dao(Demo3.java:53):执行数据库操作
****1565339644215[traceId:4],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据1成功
****1565339644215[traceId:3],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据0成功
****1565339644215[traceId:4],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据2成功
****1565339644215[traceId:3],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据1成功
****1565339644215[traceId:3],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo3.dao(Demo3.java:56):插入数据数据2成功
```

可以看出输出和刚才使用traceId参数的方式结果一致，但是却简单了很多。不用去修改controller、service、dao代码了，风险也减少了很多。

代码中创建了一个`ThreadLocal traceIdKD`，这个对象用来操作Thread中一个口袋，用这个口袋来存放tranceId。在main方法中通过`traceIdKD.set(traceId)`方法将traceId放入口袋，log方法中通`traceIdKD.get()`获取口袋中的traceId，最后任务处理完之后，main中的finally中调用`traceIdKD.remove();`将口袋中的traceId清除。

**ThreadLocal的官方API解释为：**

> “该类提供了线程局部 (thread-local) 变量。这些变量不同于它们的普通对应物，因为访问某个变量（通过其 get 或 set 方法）的每个线程都有自己的局部变量，它独立于变量的初始化副本。ThreadLocal 实例通常是类中的 private static 字段，它们希望将状态与某一个线程（例如，用户 ID 或事务 ID）相关联。”

## InheritableThreadLocal

继续上面的实例，dao中循环处理dataList的内容，假如dataList处理比较耗时，我们想加快处理速度有什么办法么？大家已经想到了，用多线程并行处理`dataList`，那么我们把代码改一下，如下：

```java
package com.itsoku.chat24;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo4 {

    //创建一个操作Thread中存放请求任务追踪id口袋的对象
    static ThreadLocal<String> traceIdKD = new ThreadLocal<>();

    static AtomicInteger threadIndex = new AtomicInteger(1);
    //创建处理请求的线程池子
    static ThreadPoolExecutor disposeRequestExecutor = new ThreadPoolExecutor(3,
            3,
            60,
            TimeUnit.SECONDS,
            new LinkedBlockingDeque<>(),
            r -> {
                Thread thread = new Thread(r);
                thread.setName("disposeRequestThread-" + threadIndex.getAndIncrement());
                return thread;
            });

    //记录日志
    public static void log(String msg) {
        StackTraceElement stack[] = (new Throwable()).getStackTrace();
        //获取当前线程存放tranceId口袋中的内容
        String traceId = traceIdKD.get();
        System.out.println("****" + System.currentTimeMillis() + "[traceId:" + traceId + "],[线程:" + Thread.currentThread().getName() + "]," + stack[1] + ":" + msg);
    }

    //模拟controller
    public static void controller(List<String> dataList) {
        log("接受请求");
        service(dataList);
    }

    //模拟service
    public static void service(List<String> dataList) {
        log("执行业务");
        dao(dataList);
    }

    //模拟dao
    public static void dao(List<String> dataList) {
        CountDownLatch countDownLatch = new CountDownLatch(dataList.size());

        log("执行数据库操作");
        String threadName = Thread.currentThread().getName();
        //模拟插入数据
        for (String s : dataList) {
            new Thread(() -> {
                try {
                    //模拟数据库操作耗时100毫秒
                    TimeUnit.MILLISECONDS.sleep(100);
                    log("插入数据" + s + "成功,主线程：" + threadName);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();
                }
            }).start();
        }
        //等待上面的dataList处理完毕
        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        //需要插入的数据
        List<String> dataList = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            dataList.add("数据" + i);
        }

        //模拟5个请求
        int requestCount = 5;
        for (int i = 0; i < requestCount; i++) {
            String traceId = String.valueOf(i);
            disposeRequestExecutor.execute(() -> {
                //把traceId放入口袋中
                traceIdKD.set(traceId);
                try {
                    controller(dataList);
                } finally {
                    //将tranceId从口袋中移除
                    traceIdKD.remove();
                }
            });
        }

        disposeRequestExecutor.shutdown();
    }
}
```

输出：

```java
****1565339904279[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.controller(Demo4.java:42):接受请求
****1565339904279[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.service(Demo4.java:48):执行业务
****1565339904279[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo4.controller(Demo4.java:42):接受请求
****1565339904279[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo4.service(Demo4.java:48):执行业务
****1565339904279[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo4.controller(Demo4.java:42):接受请求
****1565339904279[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo4.service(Demo4.java:48):执行业务
****1565339904279[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.dao(Demo4.java:56):执行数据库操作
****1565339904279[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo4.dao(Demo4.java:56):执行数据库操作
****1565339904279[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo4.dao(Demo4.java:56):执行数据库操作
****1565339904281[traceId:null],[线程:Thread-3],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据0成功,主线程：disposeRequestThread-1
****1565339904281[traceId:null],[线程:Thread-5],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据0成功,主线程：disposeRequestThread-2
****1565339904281[traceId:null],[线程:Thread-4],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据0成功,主线程：disposeRequestThread-3
****1565339904281[traceId:null],[线程:Thread-6],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据1成功,主线程：disposeRequestThread-3
****1565339904281[traceId:null],[线程:Thread-9],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据2成功,主线程：disposeRequestThread-3
****1565339904282[traceId:null],[线程:Thread-8],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据1成功,主线程：disposeRequestThread-1
****1565339904282[traceId:null],[线程:Thread-10],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据2成功,主线程：disposeRequestThread-1
****1565339904282[traceId:3],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo4.controller(Demo4.java:42):接受请求
****1565339904282[traceId:null],[线程:Thread-7],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据1成功,主线程：disposeRequestThread-2
****1565339904282[traceId:null],[线程:Thread-11],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据2成功,主线程：disposeRequestThread-2
****1565339904282[traceId:3],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo4.service(Demo4.java:48):执行业务
****1565339904282[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.controller(Demo4.java:42):接受请求
****1565339904283[traceId:3],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo4.dao(Demo4.java:56):执行数据库操作
****1565339904283[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.service(Demo4.java:48):执行业务
****1565339904283[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.dao(Demo4.java:56):执行数据库操作
****1565339904283[traceId:null],[线程:Thread-12],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据0成功,主线程：disposeRequestThread-3
****1565339904283[traceId:null],[线程:Thread-13],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据1成功,主线程：disposeRequestThread-3
****1565339904283[traceId:null],[线程:Thread-14],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据0成功,主线程：disposeRequestThread-1
****1565339904284[traceId:null],[线程:Thread-15],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据2成功,主线程：disposeRequestThread-3
****1565339904284[traceId:null],[线程:Thread-17],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据2成功,主线程：disposeRequestThread-1
****1565339904284[traceId:null],[线程:Thread-16],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:62):插入数据数据1成功,主线程：disposeRequestThread-1
```

看一下上面的输出，有些traceId为null，这是为什么呢？这是因为dao中为了提升处理速度，创建了子线程来并行处理，子线程调用log的时候，去自己的存放traceId的口袋中拿去东西，肯定是空的了。

那有什么办法么？可不可以这样？

父线程相当于主管，子线程相当于干活的小弟，主管让小弟们干活的时候，将自己兜里面的东西复制一份给小弟们使用，主管兜里面可能有很多牛逼的工具，为了提升小弟们的工作效率，给小弟们都复制一个，丢到小弟们的兜里，然后小弟就可以从自己的兜里拿去这些东西使用了，也可以清空自己兜里面的东西。

`Thread`对象中有个`inheritableThreadLocals`变量，代码如下：

```java
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

inheritableThreadLocals相当于线程中另外一种兜，这种兜有什么特征呢，当创建子线程的时候，子线程会将父线程这种类型兜的东西全部复制一份放到自己的`inheritableThreadLocals`兜中，使用`InheritableThreadLocal`对象可以操作线程中的`inheritableThreadLocals`兜。

`InheritableThreadLocal`常用的方法也有3个：

```java
//向Thread中某个口袋中放东西
public void set(T value);
//获取这个口袋中目前放的东西
public T get();
//清空这个口袋中放的东西
public void remove()
```

使用`InheritableThreadLocal`解决上面子线程中无法输出traceId的问题，只需要将上一个示例代码中的`ThreadLocal`替换成`InheritableThreadLocal`即可，代码如下：

```java
package com.itsoku.chat24;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo4 {

    //创建一个操作Thread中存放请求任务追踪id口袋的对象,子线程可以继承父线程中内容
    static InheritableThreadLocal<String> traceIdKD = new InheritableThreadLocal<>();

    static AtomicInteger threadIndex = new AtomicInteger(1);
    //创建处理请求的线程池子
    static ThreadPoolExecutor disposeRequestExecutor = new ThreadPoolExecutor(3,
            3,
            60,
            TimeUnit.SECONDS,
            new LinkedBlockingDeque<>(),
            r -> {
                Thread thread = new Thread(r);
                thread.setName("disposeRequestThread-" + threadIndex.getAndIncrement());
                return thread;
            });

    //记录日志
    public static void log(String msg) {
        StackTraceElement stack[] = (new Throwable()).getStackTrace();
        //获取当前线程存放tranceId口袋中的内容
        String traceId = traceIdKD.get();
        System.out.println("****" + System.currentTimeMillis() + "[traceId:" + traceId + "],[线程:" + Thread.currentThread().getName() + "]," + stack[1] + ":" + msg);
    }

    //模拟controller
    public static void controller(List<String> dataList) {
        log("接受请求");
        service(dataList);
    }

    //模拟service
    public static void service(List<String> dataList) {
        log("执行业务");
        dao(dataList);
    }

    //模拟dao
    public static void dao(List<String> dataList) {
        CountDownLatch countDownLatch = new CountDownLatch(dataList.size());

        log("执行数据库操作");
        String threadName = Thread.currentThread().getName();
        //模拟插入数据
        for (String s : dataList) {
            new Thread(() -> {
                try {
                    //模拟数据库操作耗时100毫秒
                    TimeUnit.MILLISECONDS.sleep(100);
                    log("插入数据" + s + "成功,主线程：" + threadName);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();
                }
            }).start();
        }
        //等待上面的dataList处理完毕
        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        //需要插入的数据
        List<String> dataList = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            dataList.add("数据" + i);
        }

        //模拟5个请求
        int requestCount = 5;
        for (int i = 0; i < requestCount; i++) {
            String traceId = String.valueOf(i);
            disposeRequestExecutor.execute(() -> {
                //把traceId放入口袋中
                traceIdKD.set(traceId);
                try {
                    controller(dataList);
                } finally {
                    //将tranceId从口袋中移除
                    traceIdKD.remove();
                }
            });
        }

        disposeRequestExecutor.shutdown();
    }
}
```

输出：

```java
****1565341611454[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo4.controller(Demo4.java:42):接受请求
****1565341611454[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo4.controller(Demo4.java:42):接受请求
****1565341611454[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.controller(Demo4.java:42):接受请求
****1565341611454[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo4.service(Demo4.java:48):执行业务
****1565341611454[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo4.service(Demo4.java:48):执行业务
****1565341611454[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.service(Demo4.java:48):执行业务
****1565341611454[traceId:2],[线程:disposeRequestThread-3],com.itsoku.chat24.Demo4.dao(Demo4.java:56):执行数据库操作
****1565341611454[traceId:1],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo4.dao(Demo4.java:56):执行数据库操作
****1565341611454[traceId:0],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.dao(Demo4.java:56):执行数据库操作
****1565341611557[traceId:2],[线程:Thread-5],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据0成功,主线程：disposeRequestThread-3
****1565341611557[traceId:0],[线程:Thread-4],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据0成功,主线程：disposeRequestThread-1
****1565341611557[traceId:1],[线程:Thread-11],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据2成功,主线程：disposeRequestThread-2
****1565341611557[traceId:1],[线程:Thread-3],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据0成功,主线程：disposeRequestThread-2
****1565341611557[traceId:1],[线程:Thread-8],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据1成功,主线程：disposeRequestThread-2
****1565341611557[traceId:0],[线程:Thread-6],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据1成功,主线程：disposeRequestThread-1
****1565341611557[traceId:0],[线程:Thread-10],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据2成功,主线程：disposeRequestThread-1
****1565341611557[traceId:3],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo4.controller(Demo4.java:42):接受请求
****1565341611557[traceId:2],[线程:Thread-9],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据2成功,主线程：disposeRequestThread-3
****1565341611558[traceId:2],[线程:Thread-7],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据1成功,主线程：disposeRequestThread-3
****1565341611557[traceId:3],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo4.service(Demo4.java:48):执行业务
****1565341611557[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.controller(Demo4.java:42):接受请求
****1565341611558[traceId:3],[线程:disposeRequestThread-2],com.itsoku.chat24.Demo4.dao(Demo4.java:56):执行数据库操作
****1565341611558[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.service(Demo4.java:48):执行业务
****1565341611558[traceId:4],[线程:disposeRequestThread-1],com.itsoku.chat24.Demo4.dao(Demo4.java:56):执行数据库操作
****1565341611659[traceId:3],[线程:Thread-15],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据2成功,主线程：disposeRequestThread-2
****1565341611659[traceId:4],[线程:Thread-14],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据0成功,主线程：disposeRequestThread-1
****1565341611659[traceId:3],[线程:Thread-13],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据1成功,主线程：disposeRequestThread-2
****1565341611659[traceId:3],[线程:Thread-12],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据0成功,主线程：disposeRequestThread-2
****1565341611660[traceId:4],[线程:Thread-16],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据1成功,主线程：disposeRequestThread-1
****1565341611660[traceId:4],[线程:Thread-17],com.itsoku.chat24.Demo4.lambda$dao$1(Demo4.java:64):插入数据数据2成功,主线程：disposeRequestThread-1
```

输出中都有traceId了，和期望的结果一致。

希望通过这篇文章可以学会使用`InheritableThreadLocal`和`InheritableThreadLocal`。有问题可以加我微信`itsoku`交流，也可以留言，谢谢。

# Queue（阻塞队列）

## 本文内容

1. 掌握Queue、BlockingQueue接口中常用的方法
2. 介绍6中阻塞队列，及相关场景示例
3. 重点掌握4种常用的阻塞队列

## Queue接口

队列是一种先进先出（FIFO）的数据结构，java中用`Queue`接口来表示队列。

`Queue`接口中定义了6个方法：

```java
public interface Queue<E> extends Collection<E> {
    boolean add(e);
    boolean offer(E e);
    E remove();
    E poll();
    E element();
    E peek();
}
```

每个`Queue`方法都有两种形式：

（1）如果操作失败则抛出异常，

（2）如果操作失败，则返回特殊值（`null`或`false`，具体取决于操作），接口的常规结构如下表所示。

| 操作类型 | 抛出异常    | 返回特殊值 |
| :------- | :---------- | :--------- |
| 插入     | `add(e)`    | `offer(e)` |
| 移除     | `remove()`  | `poll()`   |
| 检查     | `element()` | `peek()`   |

`Queue`从`Collection`继承的`add`方法插入一个元素，除非它违反了队列的容量限制，在这种情况下它会抛出`IllegalStateException`；`offer`方法与`add`不同之处仅在于它通过返回`false`来表示插入元素失败。

`remove`和`poll`方法都移除并返回队列的头部，确切地移除哪个元素是由具体的实现来决定的，仅当队列为空时，`remove`和`poll`方法的行为才有所不同，在这些情况下，`remove`抛出`NoSuchElementException`，而`poll`返回`null`。

`element`和`peek`方法返回队列头部的元素，但不移除，它们之间的差异与`remove`和`poll`的方式完全相同，如果队列为空，则`element`抛出`NoSuchElementException`，而`peek`返回`null`。

队列一般不要插入空元素。

## BlockingQueue接口

`BlockingQueue`位于juc中，熟称阻塞队列， 阻塞队列首先它是一个队列，继承`Queue`接口，是队列就会遵循先进先出（FIFO）的原则，又因为它是阻塞的，故与普通的队列有两点区别：

1. 当一个线程向队列里面添加数据时，如果队列是满的，那么将阻塞该线程，暂停添加数据
2. 当一个线程从队列里面取出数据时，如果队列是空的，那么将阻塞该线程，暂停取出数据

`BlockingQueue`相关方法：

| 操作类型 | 抛出异常    | 返回特殊值 | 一直阻塞 | 超时退出               |
| :------- | :---------- | :--------- | -------- | ---------------------- |
| 插入     | `add(e)`    | `offer(e)` | put(e)   | offer(e,timeuout,unit) |
| 移除     | `remove()`  | `poll()`   | take()   | poll(timeout,unit)     |
| 检查     | `element()` | `peek()`   | 不支持   | 不支持                 |

**重点，再来解释一下，加深印象：**

1. 3个可能会有异常的方法，add、remove、element；这3个方法不会阻塞（是说队列满或者空的情况下是否会阻塞）；队列满的情况下，add抛出异常；队列为空情况下，remove、element抛出异常
2. offer、poll、peek 也不会阻塞（是说队列满或者空的情况下是否会阻塞）；队列满的情况下，offer返回false；队列为空的情况下，pool、peek返回null
3. 队列满的情况下，调用put方法会导致当前线程阻塞
4. 队列为空的情况下，调用take方法会导致当前线程阻塞
5. `offer(e,timeuout,unit)`，超时之前，插入成功返回true，否者返回false
6. `poll(timeout,unit)`，超时之前，获取到头部元素并将其移除，返回true，否者返回false
7. **以上一些方法希望大家都记住，方便以后使用**

## BlockingQueue常见的实现类

看一下相关类图

![img](https://img2018.cnblogs.com/blog/687624/201908/687624-20190815162502143-70167989.png)

**ArrayBlockingQueue**

> 基于数组的阻塞队列实现，其内部维护一个定长的数组，用于存储队列元素。线程阻塞的实现是通过ReentrantLock来完成的，数据的插入与取出共用同一个锁，因此ArrayBlockingQueue并不能实现生产、消费同时进行。而且在创建ArrayBlockingQueue时，我们还可以控制对象的内部锁是否采用公平锁，默认采用非公平锁。

**LinkedBlockingQueue**

> 基于单向链表的阻塞队列实现，在初始化LinkedBlockingQueue的时候可以指定大小，也可以不指定，默认类似一个无限大小的容量（Integer.MAX_VALUE），不指队列容量大小也是会有风险的，一旦数据生产速度大于消费速度，系统内存将有可能被消耗殆尽，因此要谨慎操作。另外LinkedBlockingQueue中用于阻塞生产者、消费者的锁是两个（锁分离），因此生产与消费是可以同时进行的。

**PriorityBlockingQueue**

> 一个支持优先级排序的无界阻塞队列，进入队列的元素会按照优先级进行排序

**SynchronousQueue**

> 同步阻塞队列，SynchronousQueue没有容量，与其他BlockingQueue不同，SynchronousQueue是一个不存储元素的BlockingQueue，每一个put操作必须要等待一个take操作，否则不能继续添加元素，反之亦然

**DelayQueue**

> DelayQueue是一个支持延时获取元素的无界阻塞队列，里面的元素全部都是“可延期”的元素，列头的元素是最先“到期”的元素，如果队列里面没有元素到期，是不能从列头获取元素的，哪怕有元素也不行，也就是说只有在延迟期到时才能够从队列中取元素

**LinkedTransferQueue**

> LinkedTransferQueue是基于链表的FIFO无界阻塞队列，它出现在JDK7中，Doug Lea 大神说LinkedTransferQueue是一个聪明的队列，它是ConcurrentLinkedQueue、SynchronousQueue(公平模式下)、无界的LinkedBlockingQueues等的超集，`LinkedTransferQueue`包含了`ConcurrentLinkedQueue、SynchronousQueue、LinkedBlockingQueues`三种队列的功能

下面我们来介绍每种阻塞队列的使用。

## ArrayBlockingQueue

有界阻塞队列，内部使用数组存储元素，有2个常用构造方法：

```java
//capacity表示容量大小，默认内部采用非公平锁
public ArrayBlockingQueue(int capacity)
//capacity：容量大小，fair：内部是否是使用公平锁
public ArrayBlockingQueue(int capacity, boolean fair)
```

**需求：**业务系统中有很多地方需要推送通知，由于需要推送的数据太多，我们将需要推送的信息先丢到阻塞队列中，然后开一个线程进行处理真实发送，代码如下：

```java
package com.itsoku.chat25;

import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import sun.text.normalizer.NormalizerBase;

import java.util.Calendar;
import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo1 {
    //推送队列
    static ArrayBlockingQueue<String> pushQueue = new ArrayBlockingQueue<String>(10000);

    static {
        //启动一个线程做真实推送
        new Thread(() -> {
            while (true) {
                String msg;
                try {
                    long starTime = System.currentTimeMillis();
                    //获取一条推送消息，此方法会进行阻塞，直到返回结果
                    msg = pushQueue.take();
                    long endTime = System.currentTimeMillis();
                    //模拟推送耗时
                    TimeUnit.MILLISECONDS.sleep(500);

                    System.out.println(String.format("[%s,%s,take耗时:%s],%s,发送消息:%s", starTime, endTime, (endTime - starTime), Thread.currentThread().getName(), msg));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    //推送消息，需要发送推送消息的调用该方法，会将推送信息先加入推送队列
    public static void pushMsg(String msg) throws InterruptedException {
        pushQueue.put(msg);
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 1; i <= 5; i++) {
            String msg = "一起来学java高并发,第" + i + "天";
            //模拟耗时
            TimeUnit.SECONDS.sleep(i);
            Demo1.pushMsg(msg);
        }
    }
}
```

输出：

```java
[1565595629206,1565595630207,take耗时:1001],Thread-0,发送消息:一起来学java高并发,第1天
[1565595630208,1565595632208,take耗时:2000],Thread-0,发送消息:一起来学java高并发,第2天
[1565595632208,1565595635208,take耗时:3000],Thread-0,发送消息:一起来学java高并发,第3天
[1565595635208,1565595639209,take耗时:4001],Thread-0,发送消息:一起来学java高并发,第4天
[1565595639209,1565595644209,take耗时:5000],Thread-0,发送消息:一起来学java高并发,第5天
```

代码中我们使用了有界队列`ArrayBlockingQueue`，创建`ArrayBlockingQueue`时候需要制定容量大小，调用`pushQueue.put`将推送信息放入队列中，如果队列已满，此方法会阻塞。代码中在静态块中启动了一个线程，调用`pushQueue.take();`从队列中获取待推送的信息进行推送处理。

**注意：**`ArrayBlockingQueue`如果队列容量设置的太小，消费者发送的太快，消费者消费的太慢的情况下，会导致队列空间满，调用put方法会导致发送者线程阻塞，所以注意设置合理的大小，协调好消费者的速度。

## LinkedBlockingQueue

内部使用单向链表实现的阻塞队列，3个构造方法：

```java
//默认构造方法，容量大小为Integer.MAX_VALUE
public LinkedBlockingQueue();
//创建指定容量大小的LinkedBlockingQueue
public LinkedBlockingQueue(int capacity);
//容量为Integer.MAX_VALUE,并将传入的集合丢入队列中
public LinkedBlockingQueue(Collection<? extends E> c);
```

`LinkedBlockingQueue`的用法和`ArrayBlockingQueue`类似，建议使用的时候指定容量，如果不指定容量，插入的太快，移除的太慢，可能会产生OOM。

## PriorityBlockingQueue

**无界的优先级**阻塞队列，内部使用数组存储数据，达到容量时，会自动进行扩容，放入的元素会按照优先级进行排序，4个构造方法：

```java
//默认构造方法，默认初始化容量是11
public PriorityBlockingQueue();
//指定队列的初始化容量
public PriorityBlockingQueue(int initialCapacity);
//指定队列的初始化容量和放入元素的比较器
public PriorityBlockingQueue(int initialCapacity,Comparator<? super E> comparator);
//传入集合放入来初始化队列，传入的集合可以实现SortedSet接口或者PriorityQueue接口进行排序，如果没有实现这2个接口，按正常顺序放入队列
public PriorityBlockingQueue(Collection<? extends E> c);
```

优先级队列放入元素的时候，会进行排序，所以我们需要指定排序规则，有2种方式：

1. 创建`PriorityBlockingQueue`指定比较器`Comparator`
2. 放入的元素需要实现`Comparable`接口

上面2种方式必须选一个，如果2个都有，则走第一个规则排序。

**需求：**还是上面的推送业务，目前推送是按照放入的先后顺序进行发送的，比如有些公告比较紧急，优先级比较高，需要快点发送，怎么搞？此时`PriorityBlockingQueue`就派上用场了，代码如下：

```java
package com.itsoku.chat25;

import java.util.concurrent.PriorityBlockingQueue;
import java.util.concurrent.TimeUnit;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo2 {

    //推送信息封装
    static class Msg implements Comparable<Msg> {
        //优先级，越小优先级越高
        private int priority;
        //推送的信息
        private String msg;

        public Msg(int priority, String msg) {
            this.priority = priority;
            this.msg = msg;
        }

        @Override
        public int compareTo(Msg o) {
            return Integer.compare(this.priority, o.priority);
        }

        @Override
        public String toString() {
            return "Msg{" +
                    "priority=" + priority +
                    ", msg='" + msg + '\'' +
                    '}';
        }
    }

    //推送队列
    static PriorityBlockingQueue<Msg> pushQueue = new PriorityBlockingQueue<Msg>();

    static {
        //启动一个线程做真实推送
        new Thread(() -> {
            while (true) {
                Msg msg;
                try {
                    long starTime = System.currentTimeMillis();
                    //获取一条推送消息，此方法会进行阻塞，直到返回结果
                    msg = pushQueue.take();
                    //模拟推送耗时
                    TimeUnit.MILLISECONDS.sleep(100);
                    long endTime = System.currentTimeMillis();
                    System.out.println(String.format("[%s,%s,take耗时:%s],%s,发送消息:%s", starTime, endTime, (endTime - starTime), Thread.currentThread().getName(), msg));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    //推送消息，需要发送推送消息的调用该方法，会将推送信息先加入推送队列
    public static void pushMsg(int priority, String msg) throws InterruptedException {
        pushQueue.put(new Msg(priority, msg));
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 5; i >= 1; i--) {
            String msg = "一起来学java高并发,第" + i + "天";
            Demo2.pushMsg(i, msg);
        }
    }
}
```

输出：

```java
[1565598857028,1565598857129,take耗时:101],Thread-0,发送消息:Msg{priority=1, msg='一起来学java高并发,第1天'}
[1565598857162,1565598857263,take耗时:101],Thread-0,发送消息:Msg{priority=2, msg='一起来学java高并发,第2天'}
[1565598857263,1565598857363,take耗时:100],Thread-0,发送消息:Msg{priority=3, msg='一起来学java高并发,第3天'}
[1565598857363,1565598857463,take耗时:100],Thread-0,发送消息:Msg{priority=4, msg='一起来学java高并发,第4天'}
[1565598857463,1565598857563,take耗时:100],Thread-0,发送消息:Msg{priority=5, msg='一起来学java高并发,第5天'}
```

main中放入了5条推送信息，i作为消息的优先级按倒叙放入的，最终输出结果中按照优先级由小到大输出。注意Msg实现了`Comparable`接口，具有了比较功能。

## SynchronousQueue

> 同步阻塞队列，SynchronousQueue没有容量，与其他BlockingQueue不同，SynchronousQueue是一个不存储元素的BlockingQueue，每一个put操作必须要等待一个take操作，否则不能继续添加元素，反之亦然。SynchronousQueue 在现实中用的不多，线程池中有用到过，`Executors.newCachedThreadPool()`实现中用到了这个队列，当有任务丢入线程池的时候，如果已创建的工作线程都在忙于处理任务，则会新建一个线程来处理丢入队列的任务。

来个示例代码：

```java
package com.itsoku.chat25;

import java.util.concurrent.PriorityBlockingQueue;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo3 {

    static SynchronousQueue<String> queue = new SynchronousQueue<>();

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            try {
                long starTime = System.currentTimeMillis();
                queue.put("java高并发系列，路人甲Java!");
                long endTime = System.currentTimeMillis();
                System.out.println(String.format("[%s,%s,take耗时:%s],%s", starTime, endTime, (endTime - starTime), Thread.currentThread().getName()));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        //休眠5秒之后，从队列中take一个元素
        TimeUnit.SECONDS.sleep(5);
        System.out.println(System.currentTimeMillis() + "调用take获取并移除元素," + queue.take());
    }
}
```

输出：

```java
1565600421645调用take获取并移除元素,java高并发系列，路人甲Java!
[1565600416645,1565600421645,take耗时:5000],Thread-0
```

main方法中启动了一个线程，调用`queue.put`方法向队列中丢入一条数据，调用的时候产生了阻塞，从输出结果中可以看出，直到take方法被调用时，put方法才从阻塞状态恢复正常。

## DelayQueue

> DelayQueue是一个支持延时获取元素的无界阻塞队列，里面的元素全部都是“可延期”的元素，列头的元素是最先“到期”的元素，如果队列里面没有元素到期，是不能从列头获取元素的，哪怕有元素也不行，也就是说只有在延迟期到时才能够从队列中取元素。

**需求：**还是推送的业务，有时候我们希望早上9点或者其他指定的时间进行推送，如何实现呢？此时`DelayQueue`就派上用场了。

我们先看一下`DelayQueue`类的声明：

```java
public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
    implements BlockingQueue<E>
```

元素E需要实现接口`Delayed`，我们看一下这个接口的代码：

```java
public interface Delayed extends Comparable<Delayed> {
    long getDelay(TimeUnit unit);
}
```

`Delayed`继承了`Comparable`接口，这个接口是用来做比较用的，`DelayQueue`内部使用`PriorityQueue`来存储数据的，`PriorityQueue`是一个优先级队列，丢入的数据会进行排序，排序方法调用的是`Comparable`接口中的方法。下面主要说一下`Delayed`接口中的`getDelay`方法：此方法在给定的时间单位内返回与此对象关联的剩余延迟时间。

**对推送我们再做一下处理，让其支持定时发送（定时在将来某个时间也可以说是延迟发送），代码如下：**

```java
package com.itsoku.chat25;

import java.util.Calendar;
import java.util.concurrent.DelayQueue;
import java.util.concurrent.Delayed;
import java.util.concurrent.PriorityBlockingQueue;
import java.util.concurrent.TimeUnit;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo4 {

    //推送信息封装
    static class Msg implements Delayed {
        //优先级，越小优先级越高
        private int priority;
        //推送的信息
        private String msg;
        //定时发送时间，毫秒格式
        private long sendTimeMs;

        public Msg(int priority, String msg, long sendTimeMs) {
            this.priority = priority;
            this.msg = msg;
            this.sendTimeMs = sendTimeMs;
        }

        @Override
        public String toString() {
            return "Msg{" +
                    "priority=" + priority +
                    ", msg='" + msg + '\'' +
                    ", sendTimeMs=" + sendTimeMs +
                    '}';
        }

        @Override
        public long getDelay(TimeUnit unit) {
            return unit.convert(this.sendTimeMs - Calendar.getInstance().getTimeInMillis(), TimeUnit.MILLISECONDS);
        }

        @Override
        public int compareTo(Delayed o) {
            if (o instanceof Msg) {
                Msg c2 = (Msg) o;
                return Integer.compare(this.priority, c2.priority);
            }
            return 0;
        }
    }

    //推送队列
    static DelayQueue<Msg> pushQueue = new DelayQueue<Msg>();

    static {
        //启动一个线程做真实推送
        new Thread(() -> {
            while (true) {
                Msg msg;
                try {
                    //获取一条推送消息，此方法会进行阻塞，直到返回结果
                    msg = pushQueue.take();
                    //此处可以做真实推送
                    long endTime = System.currentTimeMillis();
                    System.out.println(String.format("定时发送时间：%s,实际发送时间：%s,发送消息:%s", msg.sendTimeMs, endTime, msg));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    //推送消息，需要发送推送消息的调用该方法，会将推送信息先加入推送队列
    public static void pushMsg(int priority, String msg, long sendTimeMs) throws InterruptedException {
        pushQueue.put(new Msg(priority, msg, sendTimeMs));
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 5; i >= 1; i--) {
            String msg = "一起来学java高并发,第" + i + "天";
            Demo4.pushMsg(i, msg, Calendar.getInstance().getTimeInMillis() + i * 2000);
        }
    }
}
```

输出：

```java
定时发送时间：1565603357198,实际发送时间：1565603357198,发送消息:Msg{priority=1, msg='一起来学java高并发,第1天', sendTimeMs=1565603357198}
定时发送时间：1565603359198,实际发送时间：1565603359198,发送消息:Msg{priority=2, msg='一起来学java高并发,第2天', sendTimeMs=1565603359198}
定时发送时间：1565603361198,实际发送时间：1565603361199,发送消息:Msg{priority=3, msg='一起来学java高并发,第3天', sendTimeMs=1565603361198}
定时发送时间：1565603363198,实际发送时间：1565603363199,发送消息:Msg{priority=4, msg='一起来学java高并发,第4天', sendTimeMs=1565603363198}
定时发送时间：1565603365182,实际发送时间：1565603365183,发送消息:Msg{priority=5, msg='一起来学java高并发,第5天', sendTimeMs=1565603365182}
```

可以看出时间发送时间，和定时发送时间基本一致，代码中`Msg`需要实现`Delayed接口`，重点在于`getDelay`方法，这个方法返回剩余的延迟时间，代码中使用`this.sendTimeMs`减去当前时间的毫秒格式时间，得到剩余延迟时间。

## LinkedTransferQueue

> LinkedTransferQueue是一个由链表结构组成的无界阻塞TransferQueue队列。相对于其他阻塞队列，LinkedTransferQueue多了tryTransfer和transfer方法。

LinkedTransferQueue类继承自AbstractQueue抽象类，并且实现了TransferQueue接口：

```java
public interface TransferQueue<E> extends BlockingQueue<E> {
    // 如果存在一个消费者已经等待接收它，则立即传送指定的元素，否则返回false，并且不进入队列。
    boolean tryTransfer(E e);
    // 如果存在一个消费者已经等待接收它，则立即传送指定的元素，否则等待直到元素被消费者接收。
    void transfer(E e) throws InterruptedException;
    // 在上述方法的基础上设置超时时间
    boolean tryTransfer(E e, long timeout, TimeUnit unit)
        throws InterruptedException;
    // 如果至少有一位消费者在等待，则返回true
    boolean hasWaitingConsumer();
    // 获取所有等待获取元素的消费线程数量
    int getWaitingConsumerCount();
}
```

再看一下上面的这些方法，`transfer(E e)`方法和`SynchronousQueue的put方法`类似，都需要等待消费者取走元素，否者一直等待。其他方法和`ArrayBlockingQueue、LinkedBlockingQueue`中的方法类似。

## 总结

1. 重点需要了解`BlockingQueue`中的所有方法，以及他们的区别
2. 重点掌握`ArrayBlockingQueue`、`LinkedBlockingQueue`、`PriorityBlockingQueue`、`DelayQueue`的使用场景
3. 需要处理的任务有优先级的，使用`PriorityBlockingQueue`
4. 处理的任务需要延时处理的，使用`DelayQueue`

# [实战篇，接口性能成倍提升，让同事刮目相看，现学现用](https://www.cnblogs.com/itsoku123/p/11364035.html)

这是java高并发系列第27篇文章。

开发环境：jdk1.8。

### 案例讲解

电商app都有用过吧，商品详情页，需要给他们提供一个接口获取商品相关信息：

1. 商品基本信息（名称、价格、库存、会员价格等）
2. 商品图片列表
3. 商品描述信息（描述信息一般是由富文本编辑的大文本信息）

数据库中我们用了3张表存储上面的信息：

1. 商品基本信息表：t_goods（字段：id【商品id】、名称、价格、库存、会员价格等）
2. 商品图片信息表：t_goods_imgs（字段：id、goods_id【商品id】、图片路径），一个商品会有多张图片
3. 商品描述信息表：t_goods_ext（字段：id，goods_id【商品id】、商品描述信息【大字段】）

这需求对于大家来说很简单吧，伪代码如下：

```java
public Map<String,Object> detail(long goodsId){
	//创建一个map
	//step1：查询商品基本信息，放入map
	map.put("goodsModel",(select * from t_goods where id = #gooldsId#));
	//step2：查询商品图片列表，返回一个集合放入map
	map.put("goodsImgsModelList",(select * from t_goods_imgs where goods_id = #gooldsId#));
	//step3：查询商品描述信息，放入map
	map.put("goodsExtModel",(select * from t_goods_ext where goods_id = #gooldsId#));
    return map;
}
```

上面这种写法应该很常见，代码很简单，假设上面每个步骤耗时200ms，此接口总共耗时>=600毫秒，其他还涉及到网络传输耗时，估计总共会在700ms左右，此接口有没有优化的空间，性能能够提升多少？我们一起来挑战一下。

在看一下上面的逻辑，整个过程是按顺序执行的，实际上3个查询之间是没有任何依赖关系，所以说3个查询可以同时执行，那我们对这3个步骤采用多线程并行执行，看一下最后什么情况，代码如下：

```java
package com.itsoku.chat26;

import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo1 {

    /**
     * 获取商品基本信息
     *
     * @param goodsId 商品id
     * @return 商品基本信息
     * @throws InterruptedException
     */
    public String goodsDetailModel(long goodsId) throws InterruptedException {
        //模拟耗时，休眠200ms
        TimeUnit.MILLISECONDS.sleep(200);
        return "商品id:" + goodsId + ",商品基本信息....";
    }

    /**
     * 获取商品图片列表
     *
     * @param goodsId 商品id
     * @return 商品图片列表
     * @throws InterruptedException
     */
    public List<String> goodsImgsModelList(long goodsId) throws InterruptedException {
        //模拟耗时，休眠200ms
        TimeUnit.MILLISECONDS.sleep(200);
        return Arrays.asList("图1", "图2", "图3");
    }

    /**
     * 获取商品描述信息
     *
     * @param goodsId 商品id
     * @return 商品描述信息
     * @throws InterruptedException
     */
    public String goodsExtModel(long goodsId) throws InterruptedException {
        //模拟耗时，休眠200ms
        TimeUnit.MILLISECONDS.sleep(200);
        return "商品id:" + goodsId + ",商品描述信息......";
    }

    //创建个线程池
    ExecutorService executorService = Executors.newFixedThreadPool(10);

    /**
     * 获取商品详情
     *
     * @param goodsId 商品id
     * @return
     * @throws ExecutionException
     * @throws InterruptedException
     */
    public Map<String, Object> goodsDetail(long goodsId) throws ExecutionException, InterruptedException {
        Map<String, Object> result = new HashMap<>();

        //异步获取商品基本信息
        Future<String> gooldsDetailModelFuture = executorService.submit(() -> goodsDetailModel(goodsId));
        //异步获取商品图片列表
        Future<List<String>> goodsImgsModelListFuture = executorService.submit(() -> goodsImgsModelList(goodsId));
        //异步获取商品描述信息
        Future<String> goodsExtModelFuture = executorService.submit(() -> goodsExtModel(goodsId));

        result.put("gooldsDetailModel", gooldsDetailModelFuture.get());
        result.put("goodsImgsModelList", goodsImgsModelListFuture.get());
        result.put("goodsExtModel", goodsExtModelFuture.get());
        return result;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long starTime = System.currentTimeMillis();
        Map<String, Object> map = new Demo1().goodsDetail(1L);
        System.out.println(map);
        System.out.println("耗时(ms):" + (System.currentTimeMillis() - starTime));
    }
}
```

输出：

```java
{goodsImgsModelList=[图1, 图2, 图3], gooldsDetailModel=商品id:1,商品基本信息...., goodsExtModel=商品id:1,商品描述信息......}
耗时(ms):208
```

可以看出耗时200毫秒左右，性能提升了2倍，假如这个接口中还存在其他无依赖的操作，性能提升将更加显著，上面使用了线程池并行去执行3次查询的任务，最后通过Future获取异步执行结果。

**整个优化过程：**

1. 先列出无依赖的一些操作
2. 将这些操作改为并行的方式

**用到的技术有：**

1. **[线程池相关知识](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933151&idx=1&sn=2020066b974b5f4c0823abd419e8adae&chksm=88621b21bf159237bdacfb47bd1a344f7123aabc25e3607e78d936dd554412edce5dd825003d&token=995072421&lang=zh_CN#rd)**
2. **[Executors、Future相关知识](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933156&idx=1&sn=30f7d67b44a952eae98e688bc6035fbd&chksm=88621b1abf15920c7a0705fbe34c4ce92b94b88e08f8ecbcad3827a0950cfe4d95814b61f538&token=995072421&lang=zh_CN#rd)**

### 总结

1. **对于无依赖的操作尽量采用并行方式去执行，可以很好的提升接口的性能**
2. 大家可以在你们的系统中试试这种方法，感受一下效果，会让你感觉很爽

# JUC中常见的集合

## 本文内容

1. 了解JUC常见集合，学会使用
2. ConcurrentHashMap
3. ConcurrentSkipListMap
4. ConcurrentSkipListSet
5. CopyOnWriteArraySet
6. 介绍Queue接口
7. ConcurrentLinkedQueue
8. CopyOnWriteArrayList
9. 介绍Deque接口
10. ConcurrentLinkedDeque

## JUC集合框架图

![img](https://img2018.cnblogs.com/blog/687624/201908/687624-20190816151047652-375019204.png)

> 图可以看到，JUC的集合框架也是从Map、List、Set、Queue、Collection等超级接口中继承而来的。所以，大概可以知道JUC下的集合包含了一一些基本操作，并且变得线程安全。

## Map

### ConcurrentHashMap

功能和HashMap基本一致，内部使用红黑树实现的。

**特性：**

1. 迭代结果和存入顺序不一致
2. key和value都不能为空
3. 线程安全的

### ConcurrentSkipListMap

内部使用跳表实现的，放入的元素会进行排序，排序算法支持2种方式来指定：

1. 通过构造方法传入一个`Comparator`
2. 放入的元素实现`Comparable`接口

上面2种方式必选一个，如果2种都有，走规则1。

**特性：**

1. 迭代结果和存入顺序不一致
2. 放入的元素会排序
3. key和value都不能为空
4. 线程安全的

## List

### CopyOnWriteArrayList

实现List的接口的，一般我们使用`ArrayList、LinkedList、Vector`，其中只有Vector是线程安全的，可以使用Collections静态类的synchronizedList方法对ArrayList、LinkedList包装为线程安全的List，不过这些方式在保证线程安全的情况下性能都不高。

CopyOnWriteArrayList是线程安全的List，内部使用数组存储数据，`集合中多线程并行操作一般存在4种情况：读读、读写、写写、写读，这个只有在写写操作过程中会导致其他线程阻塞，其他3种情况均不会阻塞`，所以读取的效率非常高。

可以看一下这个类的名称：CopyOnWrite，意思是在写入操作的时候，进行一次自我复制，换句话说，当这个List需要修改时，并不修改原有内容（这对于保证当前在读线程的数据一致性非常重要），而是在原有存放数据的数组上产生一个副本，在副本上修改数据，修改完毕之后，用副本替换原来的数组，这样也保证了写操作不会影响读。

**特性：**

1. 迭代结果和存入顺序一致
2. 元素不重复
3. 元素可以为空
4. 线程安全的
5. 读读、读写、写读3种情况不会阻塞；写写会阻塞
6. 无界的

## Set

### ConcurrentSkipListSet

有序的Set，内部基于ConcurrentSkipListMap实现的，放入的元素会进行排序，排序算法支持2种方式来指定：

1. 通过构造方法传入一个`Comparator`
2. 放入的元素实现`Comparable`接口

上面2种方式需要实现一个，如果2种都有，走规则1

**特性：**

1. 迭代结果和存入顺序不一致
2. 放入的元素会排序
3. 元素不重复
4. 元素不能为空
5. 线程安全的
6. 无界的

### CopyOnWriteArraySet

内部使用CopyOnWriteArrayList实现的，将所有的操作都会转发给CopyOnWriteArrayList。

**特性：**

1. 迭代结果和存入顺序不一致
2. 元素不重复
3. 元素可以为空
4. 线程安全的
5. 读读、读写、写读 不会阻塞；写写会阻塞
6. 无界的

## Queue

Queue接口中的方法，我们再回顾一下：

| 操作类型 | 抛出异常    | 返回特殊值 |
| :------- | :---------- | :--------- |
| 插入     | `add(e)`    | `offer(e)` |
| 移除     | `remove()`  | `poll()`   |
| 检查     | `element()` | `peek()`   |

3种操作，每种操作有2个方法，不同点是队列为空或者满载时，调用方法是抛出异常还是返回特殊值，大家按照表格中的多看几遍，加深记忆。

### ConcurrentLinkedQueue

高效并发队列，内部使用链表实现的。

**特性：**

1. 线程安全的
2. 迭代结果和存入顺序一致
3. 元素可以重复
4. 元素不能为空
5. 线程安全的
6. 无界队列

## Deque

先介绍一下Deque接口，双向队列(Deque)是Queue的一个子接口，双向队列是指该队列两端的元素既能入队(offer)也能出队(poll)，如果将Deque限制为只能从一端入队和出队，则可实现栈的数据结构。对于栈而言，有入栈(push)和出栈(pop)，遵循先进后出原则。

一个线性 collection，支持在两端插入和移除元素。名称 *deque* 是“double ended queue（双端队列）”的缩写，通常读为“deck”。大多数 `Deque` 实现对于它们能够包含的元素数没有固定限制，但此接口既支持有容量限制的双端队列，也支持没有固定大小限制的双端队列。

此接口定义在双端队列两端访问元素的方法。提供插入、移除和检查元素的方法。每种方法都存在两种形式：一种形式在操作失败时抛出异常，另一种形式返回一个特殊值（`null` 或 `false`，具体取决于操作）。插入操作的后一种形式是专为使用有容量限制的 `Deque` 实现设计的；在大多数实现中，插入操作不能失败。

下表总结了上述 12 种方法：

|          | **第一个元素（头部）** | 第一个元素（头部） | 最后一个元素（尾部） | 最后一个元素（尾部） |
| -------- | ---------------------- | ------------------ | -------------------- | -------------------- |
|          | *抛出异常*             | *特殊值*           | *抛出异常*           | *特殊值*             |
| **插入** | `addFirst(e)`          | `offerFirst(e)`    | `addLast(e)`         | `offerLast(e)`       |
| **移除** | `removeFirst()`        | `pollFirst()`      | `removeLast()`       | `pollLast()`         |
| **检查** | `getFirst()`           | `peekFirst()`      | `getLast()`          | `peekLast()`         |

此接口扩展了 `Queue`接口。在将双端队列用作队列时，将得到 FIFO（先进先出）行为。将元素添加到双端队列的末尾，从双端队列的开头移除元素。从 `Queue` 接口继承的方法完全等效于 `Deque` 方法，如下表所示：

此接口扩展了 `Queue`接口。在将双端队列用作队列时，将得到 FIFO（先进先出）行为。将元素添加到双端队列的末尾，从双端队列的开头移除元素。从 `Queue` 接口继承的方法完全等效于 `Deque` 方法，如下表所示：

| **Queue 方法** | **等效 Deque 方法** |
| -------------- | ------------------- |
| `add(e)`       | `addLast(e)`        |
| `offer(e)`     | `offerLast(e)`      |
| `remove()`     | `removeFirst()`     |
| `poll()`       | `pollFirst()`       |
| `element()`    | `getFirst()`        |
| `peek()`       | `peekFirst()`       |

## ConcurrentLinkedDeque

实现了Deque接口，内部使用链表实现的高效的并发双端队列。

**特性：**

1. 线程安全的
2. 迭代结果和存入顺序一致
3. 元素可以重复
4. 元素不能为空
5. 线程安全的
6. 无界队列

## BlockingQueue

关于阻塞队列，上一篇有详细介绍，可以看看：**[掌握JUC中的阻塞队列](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933190&idx=1&sn=916f539cb1e695948169a358549227d3&chksm=88621b78bf15926e0a94e50a43651dab0ceb14a1fb6b1d8b9b75e38c6d8ac908e31dd2131ded&token=1963100670&lang=zh_CN#rd)**



# 实战篇，微服务日志的伤痛，一并帮你解决掉

### 本文内容

1. 日志有什么用？
2. 日志存在的痛点？
3. 构建日志系统

## 日志有什么用？

1. 系统出现故障的时候，可以通过日志信息快速定位问题，修复bug，恢复业务
2. 提取有用数据，做数据分析使用

本文主要讨论通过日志来快速定位并解决问题。

## 日志存在的痛点

先介绍一下多数公司采用的方式：目前比较流行的是采用springcloud（或者dubbo）做微服务，按照业拆分为多个独立的服务，服务采用集群的方式部署在不同的机器上，当一个请求过来的时候，可能会调用到很多服务进行处理，springcloud一般采用logback（或者log4j）输出日志到文件中。当系统出问题的时候，按照系统故障的严重程度，严重的会回退版本，然后排查bug，轻的，找运维去线上拉日志，然后排查问题。

这个过程中存在一些问题：

1. 日志文件太大太多，不方便查找
2. 日志分散在不同的机器上，也不方便查找
3. 一个请求可能会调用多个服务，完整的日志难以追踪
4. 系统出现了问题，只能等到用户发现了，自己才知道

本文要解决上面的几个痛点，构建我们的日志系统，达到以下要求：

1. 方便追踪一个请求完整的日志
2. 方便快速检索日志
3. 系统出现问题自动报警，通知相关人员

## 构建日志系统

按照上面我们定的要求，一个个解决。

### 方便追踪一个请求完整的日志

当一个请求过来的时候，可能会调用多个服务，多个服务内部可能又会产生子线程处理业务，所以这里面有两个问题需要解决：

1. 多个服务之间日志的追踪
2. 服务内部子线程和主线程日志的追踪，这个地方举个例子，比如一个请求内部需要给10000人发送推送，内部开启10个线程并行处理，处理完毕之后响应操作者，这里面有父子线程，我们要能够找到这个里面所有的日志

需要追踪一个请求完整日志，我们需要给每个请求设置一个全局唯一编号，可以使用UUID或者其他方式也行。

多个服务之间日志追踪的问题：当一个请求过来的时候，在入口处生成一个trace_id，然后放在ThreadLocal中，如果内部设计到多个服务之间相互调用，调用其他服务的时，将trace_id顺便携带过去。

父子线程日志追踪的问题：可以采用InheritableThreadLocal来存放trace_id，这样可以在线程中获取到父线程中的trace_id。

所以此处我们需要使用`InheritableThreadLocal`来存储trace_id。

关于ThreadLocal和InheritableThreadLocal可以参考：[ThreadLocal、InheritableThreadLocal（通俗易懂）](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933186&idx=1&sn=079567e8799e43cb734b833c44055c01&chksm=88621b7cbf15926aace88777445822314d6eed2c1f5559b36cb6a6e181f0e543ee14d832ebc2&token=2027319240&lang=zh_CN#rd)

如果自己使用了线程池处理请求的，由于线程池中的线程采用的是复用的方式，所以需要对执行的任务Runable做一些改造，如代码：

```java
public class TraceRunnable implements Runnable {
    private String tranceId;
    private Runnable target;

    public TraceRunnable(Runnable target) {
        this.tranceId = TraceUtil.get();
        this.target = target;
    }

    @Override
    public void run() {
        try {
            TraceUtil.set(this.tranceId);
            MDC.put(TraceUtil.MDC_TRACE_ID, TraceUtil.get());
            this.target.run();
        } finally {
            MDC.remove(TraceUtil.MDC_TRACE_ID);
            TraceUtil.remove();
        }
    }

    public static Runnable trace(Runnable target) {
        return new TraceRunnable(target);
    }
}
```

需要用线程池执行的任务使用`TraceRunnable`封装一下就可以了。

TraceUtil代码：

```java
public class TraceUtil {

    public static final String REQUEST_HEADER_TRACE_ID = "com.ms.header.trace.id";
    public static final String MDC_TRACE_ID = "trace_id";

    private static InheritableThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();

    /**
     * 获取traceid
     *
     * @return
     */
    public static String get() {
        String traceId = inheritableThreadLocal.get();
        if (traceId == null) {
            traceId = IDUtil.getId();
            inheritableThreadLocal.set(traceId);
        }
        return traceId;
    }

    public static void set(String trace_id) {
        inheritableThreadLocal.set(trace_id);
    }

    public static void remove() {
        inheritableThreadLocal.remove();
    }

}
```

日志输出中携带上trace_id，这样最终我们就可以通过trace_id找到一个请求的完整日志了。

### 方便快速检索日志

日志分散在不同的机器上，如果要快速检索，需要将所有服务产生的日志汇集到一个地方。

关于检索日志的，列一下需求：

1. 我们将收集日志发送到消息中间件中（可以是kafka、rocketmq），消息中间件这块不介绍，选择玩的比较溜的就可以了
2. 系统产生日志尽量不要影响接口的效率
3. 带宽有限的情况下，发送日志也尽量不要去影响业务
4. 日志尽量低延次，产生的日志，尽量在生成之后1分钟后可以检索到
5. 检索日志功能要能够快速响应

关于上面几点，我们需要做的：日志发送的地方进行改造，引入消息中间件，将日志异步发送到消息中间件中，查询的地方采用elasticsearch，日志系统需要订阅消息中间件中的日志，然后丢给elasticsearch建索引，方便快速检索，咱们来一点点的介绍。

**日志发送端的改造**

日志是有业务系统产生的，一个请求过来的时候会产生很多日志，日志产生时，我们尽量减少日志输出对业务耗时的影响，我们的过程如下：

1. 业务系统内部引用一个线程池来异步处理日志，线程池内部可以使用一个容量稍微大一点的阻塞队列
2. 业务系统将日志丢给线程池进行处理
3. 线程池中将需要处理的日志先压缩一下，然后发送至mq

线程池的使用可以参考：[JAVA线程池，这一篇就够了](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933151&idx=1&sn=2020066b974b5f4c0823abd419e8adae&chksm=88621b21bf159237bdacfb47bd1a344f7123aabc25e3607e78d936dd554412edce5dd825003d&token=2027319240&lang=zh_CN#rd)

**引入mq存储日志**

业务系统将日志先发送到mq中，后面由其他消费者订阅进行消费。日志量比较大的，对mq的要求也比较高，可以选择kafka，业务量小的，也可以选取activemq。

**使用elasticsearch来检索日志**

elasticsearch（以下简称es）是一个全文检索工具，具体详情可以参考其官网相关文档。使用它来检索数据效率非常高。日志系统中需要我们开发一个消费端来拉取mq中的消息，将其存储到es中方便快速检索，关于这块有几点说一下：

1. 建议按天在es中建立数据库，日质量非常大的，也可以按小时建立数据库。查询的时候，时间就是必选条件了，这样可以快速让es定位到日志库进行检索，提升检索效率
2. 日志常见的需要收集的信息：trace_id、时间、日志级别、类、方法、url、调用的接口开始时间、调用接口的结束时间、接口耗时、接口状态码、异常信息、日志信息等等，可以按照这些在es中建立索引，方便检索。

### 日志监控报警

日志监控报警是非常重要的，这个必须要有，日志系统中需要开发监控报警功能，这块我们可以做成通过页面配置的方式，支持报警规则的配置，如日志中产生了某些异常、接口响应时间大于多少、接口返回状态码404等异常信息的时候能够报警，具体的报警可以是语音电话、短信通知、钉钉机器人报警等等，这些也做成可以配置的。

日志监控模块从mq中拉取日志，然后去匹配我们启用的一些规则进行报警。

### 结构图如下

![img](https://img2018.cnblogs.com/blog/687624/201908/687624-20190819160614349-1714858769.png)

关于搭建日志中遇到的一些痛点，可以加我微信itsoku交流。

## 构建日志系统需要用到的知识点

1. [java中线程池的使用](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933151&idx=1&sn=2020066b974b5f4c0823abd419e8adae&chksm=88621b21bf159237bdacfb47bd1a344f7123aabc25e3607e78d936dd554412edce5dd825003d&token=2027319240&lang=zh_CN#rd)
2. [ThreadLocal、InheritableThreadLocal（通俗易懂）](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933186&idx=1&sn=079567e8799e43cb734b833c44055c01&chksm=88621b7cbf15926aace88777445822314d6eed2c1f5559b36cb6a6e181f0e543ee14d832ebc2&token=2027319240&lang=zh_CN#rd)
3. elasticsearch，可以参考其官方文档
4. mq

# 高并发中常见的限流方式

## 本文内容

1. 介绍常见的限流算法
2. 通过控制最大并发数来进行限流
3. 通过漏桶算法来进行限流
4. 通过令牌桶算法来进行限流
5. 限流工具类RateLimiter

## 常见的限流的场景

1. 秒杀活动，数量有限，访问量巨大，为了防止系统宕机，需要做限流处理
2. 国庆期间，一般的旅游景点人口太多，采用排队方式做限流处理
3. 医院看病通过发放排队号的方式来做限流处理。

## 常见的限流算法

1. 通过控制最大并发数来进行限流
2. 使用漏桶算法来进行限流
3. 使用令牌桶算法来进行限流

## 通过控制最大并发数来进行限流

以秒杀业务为例，10个iphone，100万人抢购，100万人同时发起请求，最终能够抢到的人也就是前面几个人，后面的基本上都没有希望了，那么我们可以通过控制并发数来实现，比如并发数控制在10个，其他超过并发数的请求全部拒绝，提示：秒杀失败，请稍后重试。

并发控制的，通俗解释：一大波人去商场购物，必须经过一个门口，门口有个门卫，兜里面有指定数量的门禁卡，来的人先去门卫那边拿取门禁卡，拿到卡的人才可以刷卡进入商场，拿不到的可以继续等待。进去的人出来之后会把卡归还给门卫，门卫可以把归还来的卡继续发放给其他排队的顾客使用。

JUC中提供了这样的工具类：Semaphore，示例代码：

```java
package com.itsoku.chat29;

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo1 {

    static Semaphore semaphore = new Semaphore(5);

    public static void main(String[] args) {
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                boolean flag = false;
                try {
                    flag = semaphore.tryAcquire(100, TimeUnit.MICROSECONDS);
                    if (flag) {
                        //休眠2秒，模拟下单操作
                        System.out.println(Thread.currentThread() + "，尝试下单中。。。。。");
                        TimeUnit.SECONDS.sleep(2);
                    } else {
                        System.out.println(Thread.currentThread() + "，秒杀失败，请稍微重试！");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    if (flag) {
                        semaphore.release();
                    }
                }
            }).start();
        }
    }

}
```

输出：

```java
Thread[Thread-10,5,main]，尝试下单中。。。。。
Thread[Thread-8,5,main]，尝试下单中。。。。。
Thread[Thread-9,5,main]，尝试下单中。。。。。
Thread[Thread-12,5,main]，尝试下单中。。。。。
Thread[Thread-11,5,main]，尝试下单中。。。。。
Thread[Thread-2,5,main]，秒杀失败，请稍微重试！
Thread[Thread-1,5,main]，秒杀失败，请稍微重试！
Thread[Thread-18,5,main]，秒杀失败，请稍微重试！
Thread[Thread-16,5,main]，秒杀失败，请稍微重试！
Thread[Thread-0,5,main]，秒杀失败，请稍微重试！
Thread[Thread-3,5,main]，秒杀失败，请稍微重试！
Thread[Thread-14,5,main]，秒杀失败，请稍微重试！
Thread[Thread-6,5,main]，秒杀失败，请稍微重试！
Thread[Thread-13,5,main]，秒杀失败，请稍微重试！
Thread[Thread-17,5,main]，秒杀失败，请稍微重试！
Thread[Thread-7,5,main]，秒杀失败，请稍微重试！
Thread[Thread-19,5,main]，秒杀失败，请稍微重试！
Thread[Thread-15,5,main]，秒杀失败，请稍微重试！
Thread[Thread-4,5,main]，秒杀失败，请稍微重试！
Thread[Thread-5,5,main]，秒杀失败，请稍微重试！
```

关于`Semaphore`的使用，可以移步：[JUC中的Semaphore（信号量）](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933130&idx=1&sn=cecc6bd906e79a86510c1fbb0e66cd21&chksm=88621b34bf159222042da8ed4b633e94ca04a614d290d54a952a668459a339ebec0c754d562d&token=702505185&lang=zh_CN&scene=21#wechat_redirect)

## 使用漏桶算法来进行限流

国庆期间比较火爆的景点，人流量巨大，一般入口处会有限流的弯道，让游客进去进行排队，排在前面的人，每隔一段时间会放一拨进入景区。排队人数超过了指定的限制，后面再来的人会被告知今天已经游客量已经达到峰值，会被拒绝排队，让其明天或者以后再来，这种玩法采用漏桶限流的方式。

漏桶算法思路很简单，水（请求）先进入到漏桶里，漏桶以一定的速度出水，当水流入速度过大会直接溢出，可以看出漏桶算法能强行限制数据的传输速率。

漏桶算法示意图：

![img](https://img2018.cnblogs.com/blog/687624/201908/687624-20190820162244719-2104161385.png)

简陋版的实现，代码如下：

```java
package com.itsoku.chat29;

import java.util.Objects;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.locks.LockSupport;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo2 {

    public static class BucketLimit {
        static AtomicInteger threadNum = new AtomicInteger(1);
        //容量
        private int capcity;
        //流速
        private int flowRate;
        //流速时间单位
        private TimeUnit flowRateUnit;
        private BlockingQueue<Node> queue;
        //漏桶流出的任务时间间隔（纳秒）
        private long flowRateNanosTime;

        public BucketLimit(int capcity, int flowRate, TimeUnit flowRateUnit) {
            this.capcity = capcity;
            this.flowRate = flowRate;
            this.flowRateUnit = flowRateUnit;
            this.bucketThreadWork();
        }

        //漏桶线程
        public void bucketThreadWork() {
            this.queue = new ArrayBlockingQueue<Node>(capcity);
            //漏桶流出的任务时间间隔（纳秒）
            this.flowRateNanosTime = flowRateUnit.toNanos(1) / flowRate;
            Thread thread = new Thread(this::bucketWork);
            thread.setName("漏桶线程-" + threadNum.getAndIncrement());
            thread.start();
        }

        //漏桶线程开始工作
        public void bucketWork() {
            while (true) {
                Node node = this.queue.poll();
                if (Objects.nonNull(node)) {
                    //唤醒任务线程
                    LockSupport.unpark(node.thread);
                }
                //休眠flowRateNanosTime
                LockSupport.parkNanos(this.flowRateNanosTime);
            }
        }

        //返回一个漏桶
        public static BucketLimit build(int capcity, int flowRate, TimeUnit flowRateUnit) {
            if (capcity < 0 || flowRate < 0) {
                throw new IllegalArgumentException("capcity、flowRate必须大于0！");
            }
            return new BucketLimit(capcity, flowRate, flowRateUnit);
        }

        //当前线程加入漏桶，返回false，表示漏桶已满；true：表示被漏桶限流成功，可以继续处理任务
        public boolean acquire() {
            Thread thread = Thread.currentThread();
            Node node = new Node(thread);
            if (this.queue.offer(node)) {
                LockSupport.park();
                return true;
            }
            return false;
        }

        //漏桶中存放的元素
        class Node {
            private Thread thread;

            public Node(Thread thread) {
                this.thread = thread;
            }
        }
    }

    public static void main(String[] args) {
        BucketLimit bucketLimit = BucketLimit.build(10, 60, TimeUnit.MINUTES);
        for (int i = 0; i < 15; i++) {
            new Thread(() -> {
                boolean acquire = bucketLimit.acquire();
                System.out.println(System.currentTimeMillis() + " " + acquire);
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }

}
```

代码中`BucketLimit.build(10, 60, TimeUnit.MINUTES);`创建了一个容量为10，流水为60/分钟的漏桶。

代码中用到的技术有：

1. [BlockingQueue阻塞队列](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933190&idx=1&sn=916f539cb1e695948169a358549227d3&chksm=88621b78bf15926e0a94e50a43651dab0ceb14a1fb6b1d8b9b75e38c6d8ac908e31dd2131ded&token=1963100670&lang=zh_CN&scene=21#wechat_redirect)
2. [JUC中的LockSupport工具类，必备技能](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933125&idx=1&sn=382528aeb341727bafb02bb784ff3d4f&chksm=88621b3bbf15922d93bfba11d700724f1e59ef8a74f44adb7e131a4c3d1465f0dc539297f7f3&token=1338873010&lang=zh_CN&scene=21#wechat_redirect)

## 使用令牌桶算法来进行限流

令牌桶算法的原理是系统以恒定的速率产生令牌，然后把令牌放到令牌桶中，令牌桶有一个容量，当令牌桶满了的时候，再向其中放令牌，那么多余的令牌会被丢弃；当想要处理一个请求的时候，需要从令牌桶中取出一个令牌，如果此时令牌桶中没有令牌，那么则拒绝该请求。从原理上看，令牌桶算法和漏桶算法是相反的，一个“进水”，一个是“漏水”。这种算法可以应对突发程度的请求，因此比漏桶算法好。

令牌桶算法示意图：

![img](https://img2018.cnblogs.com/blog/687624/201908/687624-20190820162301308-1812063535.png)

有兴趣的可以自己去实现一个。

## 限流工具类RateLimiter

Google开源工具包Guava提供了限流工具类RateLimiter，可以非常方便的控制系统每秒吞吐量，示例代码如下：

```java
package com.itsoku.chat29;

import com.google.common.util.concurrent.RateLimiter;

import java.util.Calendar;
import java.util.Date;
import java.util.Objects;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.locks.LockSupport;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo3 {

    public static void main(String[] args) throws InterruptedException {
        RateLimiter rateLimiter = RateLimiter.create(5);//设置QPS为5
        for (int i = 0; i < 10; i++) {
            rateLimiter.acquire();
            System.out.println(System.currentTimeMillis());
        }
        System.out.println("----------");
        //可以随时调整速率，我们将qps调整为10
        rateLimiter.setRate(10);
        for (int i = 0; i < 10; i++) {
            rateLimiter.acquire();
            System.out.println(System.currentTimeMillis());
        }
    }
}
```

输出：

```java
1566284028725
1566284028922
1566284029121
1566284029322
1566284029522
1566284029721
1566284029921
1566284030122
1566284030322
1566284030522
----------
1566284030722
1566284030822
1566284030921
1566284031022
1566284031121
1566284031221
1566284031321
1566284031422
1566284031522
1566284031622
```

代码中`RateLimiter.create(5)`创建QPS为5的限流对象，后面又调用`rateLimiter.setRate(10);`将速率设为10，输出中分2段，第一段每次输出相隔200毫秒，第二段每次输出相隔100毫秒，可以非常精准的控制系统的QPS。

上面介绍的这些，业务中可能会用到，也可以用来应对面试。

# 获取线程执行结果，这6种方法你都知道？

这是java高并发系列第31篇。

环境：jdk1.8。

java高并发系列已经学了不少东西了，本篇文章，我们用前面学的知识来实现一个需求：

**在一个线程中需要获取其他线程的执行结果，能想到几种方式？各有什么优缺点？**

结合这个需求，我们使用**6种方式**，来对之前学过的知识点做一个回顾，加深记忆。

## 方式1：Thread的join()方法实现

代码：

```java
package com.itsoku.chat31;

import java.sql.Time;
import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo1 {
    //用于封装结果
    static class Result<T> {
        T result;

        public T getResult() {
            return result;
        }

        public void setResult(T result) {
            this.result = result;
        }
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println(System.currentTimeMillis());
        //用于存放子线程执行的结果
        Result<Integer> result = new Result<>();
        //创建一个子线程
        Thread thread = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
                result.setResult(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        thread.start();
        //让主线程等待thread线程执行完毕之后再继续，join方法会让当前线程阻塞
        thread.join();

        //获取thread线程的执行结果
        Integer rs = result.getResult();
        System.out.println(System.currentTimeMillis());
        System.out.println(System.currentTimeMillis() + ":" + rs);
    }
}
```

输出：

```java
1566733162636
1566733165692
1566733165692:10
```

代码中通过join方式阻塞了当前主线程，当thread线程执行完毕之后，join方法才会继续执行。

join的方式，只能阻塞一个线程，如果其他线程中也需要获取thread线程的执行结果，join方法无能为力了。

关于join()方法和线程更详细的使用，可以参考：[线程的基本操作](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933082&idx=1&sn=e940c4f94a8c1527b6107930eefdcd00&chksm=88621ae4bf1593f270991e6f6bac5769ea850fa02f11552d1aa91725f4512d4f1ff8f18fcdf3&token=2041017112&lang=zh_CN&scene=21#wechat_redirect)

## 方式2：CountDownLatch实现

代码：

```java
package com.itsoku.chat31;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo2 {
    //用于封装结果
    static class Result<T> {
        T result;

        public T getResult() {
            return result;
        }

        public void setResult(T result) {
            this.result = result;
        }
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println(System.currentTimeMillis());
        CountDownLatch countDownLatch = new CountDownLatch(1);
        //用于存放子线程执行的结果
        Demo1.Result<Integer> result = new Demo1.Result<>();
        //创建一个子线程
        Thread thread = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
                result.setResult(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                countDownLatch.countDown();
            }
        });
        thread.start();
        //countDownLatch.await()会让当前线程阻塞，当countDownLatch中的计数器变为0的时候，await方法会返回
        countDownLatch.await();

        //获取thread线程的执行结果
        Integer rs = result.getResult();
        System.out.println(System.currentTimeMillis());
        System.out.println(System.currentTimeMillis() + ":" + rs);
    }
}
```

输出：

```java
1566733720406
1566733723453
1566733723453:10
```

上面代码也达到了预期效果，使用`CountDownLatch`可以让一个或者多个线程等待一批线程完成之后，自己再继续；`CountDownLatch`更详细的介绍见：[JUC中等待多线程完成的工具类CountDownLatch，必备技能](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933134&idx=1&sn=65c2b9982bb6935c54ff33082f9c111f&chksm=88621b30bf159226d41607292a1dc83186f8928744dbc44acfda381266fa2cdc006177b44095&token=773938509&lang=zh_CN&scene=21#wechat_redirect)

## 方式3：ExecutorService.submit方法实现

代码：

```java
package com.itsoku.chat31;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo3 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //创建一个线程池
        ExecutorService executorService = Executors.newCachedThreadPool();
        System.out.println(System.currentTimeMillis());
        Future<Integer> future = executorService.submit(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 10;
        });
        //关闭线程池
        executorService.shutdown();
        System.out.println(System.currentTimeMillis());
        Integer result = future.get();
        System.out.println(System.currentTimeMillis() + ":" + result);
    }
}
```

输出：

```java
1566734119938
1566734119989
1566734122989:10
```

使用`ExecutorService.submit`方法实现的，此方法返回一个`Future`，`future.get()`会让当前线程阻塞，直到Future关联的任务执行完毕。

相关知识：

1. [JAVA线程池，这一篇就够了](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933151&idx=1&sn=2020066b974b5f4c0823abd419e8adae&chksm=88621b21bf159237bdacfb47bd1a344f7123aabc25e3607e78d936dd554412edce5dd825003d&token=995072421&lang=zh_CN&scene=21#wechat_redirect)
2. [JUC中的Executor框架详解1](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933156&idx=1&sn=30f7d67b44a952eae98e688bc6035fbd&chksm=88621b1abf15920c7a0705fbe34c4ce92b94b88e08f8ecbcad3827a0950cfe4d95814b61f538&token=995072421&lang=zh_CN&scene=21#wechat_redirect)
3. [JUC中的Executor框架详解2](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933160&idx=1&sn=62649485b065f68c0fc59bb502ed42df&chksm=88621b16bf159200d5e25d11ab7036c60e3f923da3212ae4dd148753d02593a45ce0e9b886c4&token=42900009&lang=zh_CN&scene=21#wechat_redirect)

## 方式4：FutureTask方式1

代码：

```java
package com.itsoku.chat31;

import java.util.concurrent.*;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo4 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println(System.currentTimeMillis());
        //创建一个FutureTask
        FutureTask<Integer> futureTask = new FutureTask<>(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 10;
        });
        //将futureTask传递一个线程运行
        new Thread(futureTask).start();
        System.out.println(System.currentTimeMillis());
        //futureTask.get()会阻塞当前线程，直到futureTask执行完毕
        Integer result = futureTask.get();
        System.out.println(System.currentTimeMillis() + ":" + result);
    }
}
```

输出：

```java
1566736350314
1566736350358
1566736353360:10
```

代码中使用`FutureTask`实现的，FutureTask实现了`Runnable`接口，并且内部带返回值，所以可以传递给Thread直接运行，`futureTask.get()`会阻塞当前线程，直到`FutureTask`构造方法传递的任务执行完毕，get方法才会返回。关于`FutureTask`详细使用，请参考：[JUC中的Executor框架详解1](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933156&idx=1&sn=30f7d67b44a952eae98e688bc6035fbd&chksm=88621b1abf15920c7a0705fbe34c4ce92b94b88e08f8ecbcad3827a0950cfe4d95814b61f538&token=995072421&lang=zh_CN&scene=21#wechat_redirect)

## 方式5：FutureTask方式2

代码：

```java
package com.itsoku.chat31;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
import java.util.concurrent.TimeUnit;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo5 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println(System.currentTimeMillis());
        //创建一个FutureTask
        FutureTask<Integer> futureTask = new FutureTask<>(() -> 10);
        //将futureTask传递一个线程运行
        new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            futureTask.run();
        }).start();
        System.out.println(System.currentTimeMillis());
        //futureTask.get()会阻塞当前线程，直到futureTask执行完毕
        Integer result = futureTask.get();
        System.out.println(System.currentTimeMillis() + ":" + result);
    }
}
```

输出：

```java
1566736319925
1566736319970
1566736322972:10
```

创建了一个`FutureTask`对象，调用`futureTask.get()`会阻塞当前线程，子线程中休眠了3秒，然后调用`futureTask.run();`当futureTask的run()方法执行完毕之后，`futureTask.get()`会从阻塞中返回。

注意：这种方式和方式4的不同点。

关于`FutureTask`详细使用，请参考：[JUC中的Executor框架详解1](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933156&idx=1&sn=30f7d67b44a952eae98e688bc6035fbd&chksm=88621b1abf15920c7a0705fbe34c4ce92b94b88e08f8ecbcad3827a0950cfe4d95814b61f538&token=995072421&lang=zh_CN&scene=21#wechat_redirect)

## 方式6：CompletableFuture方式实现

代码：

```java
package com.itsoku.chat31;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
import java.util.concurrent.TimeUnit;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo6 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println(System.currentTimeMillis());
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 10;
        });
        System.out.println(System.currentTimeMillis());
        //futureTask.get()会阻塞当前线程，直到futureTask执行完毕
        Integer result = completableFuture.get();
        System.out.println(System.currentTimeMillis() + ":" + result);
    }
}
```

输出：

```java
1566736205348
1566736205428
1566736208429:10
CompletableFuture.supplyAsync`可以用来异步执行一个带返回值的任务，调用`completableFuture.get()
```

会阻塞当前线程，直到任务执行完毕，get方法才会返回。

关于`CompletableFuture`更详细的使用见：[JUC中工具类CompletableFuture，必备技能](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933221&idx=1&sn=1af60b8917df6494b7c6b05c9eaebfe7&chksm=88621b5bbf15924d403e66e6d442d6b5897757471368b8d3a28c5de6e264cef104338dba1811&token=2098378399&lang=zh_CN#rd)

# 高并发中计数器的实现方式有哪些？

## 本文主要内容

1. 4种方式实现计数器功能，对比其性能
2. 介绍LongAdder
3. 介绍LongAccumulator

**需求：一个jvm中实现一个计数器功能，需保证多线程情况下数据正确性。**

我们来模拟50个线程，每个线程对计数器递增100万次，最终结果应该是5000万。

我们使用4种方式实现，看一下其性能，然后引出为什么需要使用`LongAdder`、`LongAccumulator`。

## 方式一：synchronized方式实现

```java
package com.itsoku.chat32;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.atomic.LongAccumulator;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo1 {
    static int count = 0;

    public static synchronized void incr() {
        count++;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        for (int i = 0; i < 10; i++) {
            count = 0;
            m1();
        }
    }

    private static void m1() throws InterruptedException {
        long t1 = System.currentTimeMillis();
        int threadCount = 50;
        CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                try {
                    for (int j = 0; j < 1000000; j++) {
                        incr();
                    }
                } finally {
                    countDownLatch.countDown();
                }
            }).start();
        }
        countDownLatch.await();
        long t2 = System.currentTimeMillis();
        System.out.println(String.format("结果：%s,耗时(ms)：%s", count, (t2 - t1)));
    }
}
```

输出：

```java
结果：50000000,耗时(ms)：1437
结果：50000000,耗时(ms)：1913
结果：50000000,耗时(ms)：386
结果：50000000,耗时(ms)：383
结果：50000000,耗时(ms)：381
结果：50000000,耗时(ms)：382
结果：50000000,耗时(ms)：379
结果：50000000,耗时(ms)：379
结果：50000000,耗时(ms)：392
结果：50000000,耗时(ms)：384
```

**平均耗时：390毫秒**

## 方式2：AtomicLong实现

```java
package com.itsoku.chat32;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.atomic.AtomicLong;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo2 {
    static AtomicLong count = new AtomicLong(0);

    public static void incr() {
        count.incrementAndGet();
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        for (int i = 0; i < 10; i++) {
            count.set(0);
            m1();
        }
    }

    private static void m1() throws InterruptedException {
        long t1 = System.currentTimeMillis();
        int threadCount = 50;
        CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                try {
                    for (int j = 0; j < 1000000; j++) {
                        incr();
                    }
                } finally {
                    countDownLatch.countDown();
                }
            }).start();
        }
        countDownLatch.await();
        long t2 = System.currentTimeMillis();
        System.out.println(String.format("结果：%s,耗时(ms)：%s", count, (t2 - t1)));
    }
}
```

输出：

```java
结果：50000000,耗时(ms)：971
结果：50000000,耗时(ms)：915
结果：50000000,耗时(ms)：920
结果：50000000,耗时(ms)：923
结果：50000000,耗时(ms)：910
结果：50000000,耗时(ms)：916
结果：50000000,耗时(ms)：923
结果：50000000,耗时(ms)：916
结果：50000000,耗时(ms)：912
结果：50000000,耗时(ms)：908
```

**平均耗时：920毫秒**

`AtomicLong`内部采用CAS的方式实现，并发量大的情况下，CAS失败率比较高，导致性能比synchronized还低一些。并发量不是太大的情况下，CAS性能还是可以的。

`AtomicLong`属于JUC中的原子类，还不是很熟悉的可以看一下：[JUC中原子类，一篇就够了](https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933181&idx=1&sn=a1e254365d405cdc2e3b8372ecda65ee&chksm=88621b03bf159215ca696c9f81e228d0544a7598b03fe30436babc95c6a95e848161f61b868c&token=743622661&lang=zh_CN&scene=21#wechat_redirect)

## 方式3：LongAdder实现

先介绍一下`LongAdder`，说到LongAdder，不得不提的就是AtomicLong，AtomicLong是JDK1.5开始出现的，里面主要使用了一个long类型的value作为成员变量，然后使用循环的CAS操作去操作value的值，并发量比较大的情况下，CAS操作失败的概率较高，内部失败了会重试，导致耗时可能会增加。

**LongAdder是JDK1.8开始出现的，所提供的API基本上可以替换掉原先的AtomicLong**。LongAdder在并发量比较大的情况下，操作数据的时候，相当于把这个数字分成了很多份数字，然后交给多个人去管控，每个管控者负责保证部分数字在多线程情况下操作的正确性。当多线程访问的时，通过hash算法映射到具体管控者去操作数据，最后再汇总所有的管控者的数据，得到最终结果。相当于降低了并发情况下锁的粒度，所以效率比较高，看一下下面的图，方便理解：

![img](https://img2018.cnblogs.com/blog/687624/201908/687624-20190828154414438-24350125.png)

代码：

```java
package com.itsoku.chat32;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.atomic.LongAdder;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo3 {
    static LongAdder count = new LongAdder();

    public static void incr() {
        count.increment();
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        for (int i = 0; i < 10; i++) {
            count.reset();
            m1();
        }
    }

    private static void m1() throws ExecutionException, InterruptedException {
        long t1 = System.currentTimeMillis();
        int threadCount = 50;
        CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                try {
                    for (int j = 0; j < 1000000; j++) {
                        incr();
                    }
                } finally {
                    countDownLatch.countDown();
                }
            }).start();
        }
        countDownLatch.await();
        long t2 = System.currentTimeMillis();
        System.out.println(String.format("结果：%s,耗时(ms)：%s", count.sum(), (t2 - t1)));
    }
}
```

输出：

```java
结果：50000000,耗时(ms)：206
结果：50000000,耗时(ms)：105
结果：50000000,耗时(ms)：107
结果：50000000,耗时(ms)：107
结果：50000000,耗时(ms)：105
结果：50000000,耗时(ms)：99
结果：50000000,耗时(ms)：106
结果：50000000,耗时(ms)：102
结果：50000000,耗时(ms)：106
结果：50000000,耗时(ms)：102
```

**平均耗时：100毫秒**

代码中`new LongAdder()`创建一个LongAdder对象，内部数字初始值是0，调用`increment()`方法可以对LongAdder内部的值原子递增1。`reset()`方法可以重置`LongAdder`的值，使其归0。

## 方式4：LongAccumulator实现

**LongAccumulator介绍**

LongAccumulator是LongAdder的功能增强版。LongAdder的API只有对数值的加减，而LongAccumulator提供了自定义的函数操作，其构造函数如下：

```java
/**
  * accumulatorFunction：需要执行的二元函数（接收2个long作为形参，并返回1个long）
  * identity：初始值
 **/
public LongAccumulator(LongBinaryOperator accumulatorFunction, long identity) {
	this.function = accumulatorFunction;
	base = this.identity = identity;
}
```

示例代码：

```java
package com.itsoku.chat32;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.atomic.LongAccumulator;
import java.util.concurrent.atomic.LongAdder;

/**
 * 跟着阿里p7学并发，微信公众号：javacode2018
 */
public class Demo4 {
    static LongAccumulator count = new LongAccumulator((x, y) -> x + y, 0L);

    public static void incr() {
        count.accumulate(1);
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        for (int i = 0; i < 10; i++) {
            count.reset();
            m1();
        }
    }

    private static void m1() throws ExecutionException, InterruptedException {
        long t1 = System.currentTimeMillis();
        int threadCount = 50;
        CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                try {
                    for (int j = 0; j < 1000000; j++) {
                        incr();
                    }
                } finally {
                    countDownLatch.countDown();
                }
            }).start();
        }
        countDownLatch.await();
        long t2 = System.currentTimeMillis();
        System.out.println(String.format("结果：%s,耗时(ms)：%s", count.longValue(), (t2 - t1)));
    }
}
```

输出：

```java
结果：50000000,耗时(ms)：138
结果：50000000,耗时(ms)：111
结果：50000000,耗时(ms)：111
结果：50000000,耗时(ms)：103
结果：50000000,耗时(ms)：103
结果：50000000,耗时(ms)：105
结果：50000000,耗时(ms)：101
结果：50000000,耗时(ms)：106
结果：50000000,耗时(ms)：102
结果：50000000,耗时(ms)：103
```

**平均耗时：100毫秒**

`LongAccumulator`的效率和`LongAdder`差不多，不过更灵活一些。

调用`new LongAdder()`等价于`new LongAccumulator((x, y) -> x + y, 0L)`。

从上面4个示例的结果来看，`LongAdder、LongAccumulator`全面超越同步锁及`AtomicLong`的方式，建议在使用`AtomicLong`的地方可以直接替换为`LongAdder、LongAccumulator`，吞吐量更高一些。

# [Java中保证线程安全的三板斧](https://my.oschina.net/onexstone/blog/5272662)

### 前言

现在，如果要使用 Java 实现一段线程安全的代码，大致有 synchronized 、 java.util.concurrent 包等手段。虽然大家都会用，但却不一定真正清楚其在 JVM 层面上的实现原理，因此，笔者在查阅了一些资料后，希望把自己对此的一些见解分享给大家。

### 测试环境

```
- JDK：
    - java version "1.8.0_202"
    - Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
    - Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
- OS：Windows 10
- IDE：
    - IntelliJ IDEA 2021.1.3 (Ultimate Edition)
```

### 三板斧之一：互斥同步

- 互斥同步：使用互斥的手段来保证同步操作。互斥是方法，同步是目的。
- 在 Java 的世界里，最基本的互斥同步手段就是使用 synchronized 关键字。

#### synchronized 关键字

1. synchronized 能实现同步的理论基础是：Java 中的每一个对象都可以作为锁。
2. synchronized 关键字在不同的使用场景下，作为锁的对象有所不同，主要分为以下三种情况：
   - 对于同步代码块，锁就是声明 synchronized 同步块时指定的对象（synchronized 括号中配置的对象）；
   - 对于普通对象方法，锁就是当前的实例对象；
   - 对于静态同步块，锁就是当前类的 Class 对象。
3. 我们可以通过一段代码来进一步说明 synchronized 是如何实现互斥同步的。

- 示例代码

```Java
public class SynchronizedTest {
    public void test() {
        synchronized (this) {
            try {
                System.out.println("SynchronizedTest.test() method start!");
            } catch (Exception e) {

            }
        }
    }
}
```

- 对上述代码生成的字节码使用 Javap 进行反编译，结果如下：

```Java
Compiled from "SynchronizedTest.java"
public class com.xxx.JVMTest.SynchronizedTest {
  public com.xxx.JVMTest.SynchronizedTest();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public void test();
    Code:
       0: aload_0
       1: dup
       2: astore_1
       3: monitorenter
       4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       7: ldc           #3                  // String SynchronizedTest.test() method start!
       9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      12: aload_1
      13: monitorexit
      14: goto          22
      17: astore_2
      18: aload_1
      19: monitorexit
      20: aload_2
      21: athrow
      22: return
    Exception table:
       from    to  target type
           4    14    17   any
          17    20    17   any
}
```

- 我们可以看到反编译的代码中，存在两个由 Javac 编译器加入的指令，分别是插入到同步代码块开始位置的 monitorenter 指令和插入到同步代码块结束位置以及异常处的 monitorexit 指令。
- 根据《Java 虚拟机规范》可知，每个 Java 对象都有一个监视器锁（monitor）。在执行 monitorenter 指令时，首先要去尝试获取对象的锁，如果这个对象没被锁定，或者当前线程已经持有了该对象的锁，就把锁的计数器的值加一，而在执行 monitorexit 指令时会将锁计数器的值减一。一旦锁计数器的值为零，锁随即被释放。如果其他线程已经占用了该对象的锁，则该线程进入阻塞状态，直到锁的计数器为零时，再重新尝试获取该对象的所有权。
- 因此，**本质上 JVM 就是通过进入 Monitor 对象（monitorenter）以及退出 Monitor 对象（monitorexit）来实现方法和代码块的同步操作。**

1. 通过对 monitorenter 指令和 monitorexit 指令的分析，我们可以推出 synchronized 的三条结论：

- 被 synchronized 声明的同步代码块对同一线程而言是可重入的，所以同一线程重复进入同步块也不会出现被自己锁死的情况；
- 被 synchronized 声明的同步块在持有锁的线程执行完毕并释放锁之前，会无条件地阻塞后面其他线程的进入。因此无法实现对已经获得锁的线程强制释放锁的操作，以及对等待锁的线程实现中断等待或超时退出的机制。
- 由于 Java 线程是映射到操作系统的原生内核线程之上的，如果要阻塞或者唤醒一条线程，则需要操作系统来帮忙完成，这不可避免地陷入用户态到核心态的转变之中，因此在一些经典的 Java 并发编程资料中，synchronized 被形象地称为重量级锁。但它相对于利用 java.util.concurrent 包中 Lock 接口实现的锁机制仍有一个先天的优势，就是 synchronized 的锁信息是被 JVM 记录在线程和对象的元数据中的，可以很轻易的知道当前哪些锁对象是被哪些特定的线程所持有，从而更容易进行锁优化。

1. 在这里需要补充一点的就是，同步方法虽然也可以使用 monitorenter 指令和 monitorexit 指令实现同步操作，但实际上目前的实现中并没有采用这种方案

- 我们可以具体分析下面的代码

```Java
public class SynchronizedTest {
    public synchronized void testTwo() {
        System.out.println("SynchronizedTest.testTwo() method start!");
    }
}
```

- 对上述代码生成的字节码使用 Javap 进行反编译，结果如下：

```Java
  public synchronized void testTwo();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #6                  // String SynchronizedTest.testTwo() method start!
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 23: 0
        line 24: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/xxx/JVMTest/SynchronizedTest;
```

- 从反编译的结果来看，方法的同步并没有通过指令 monitorenter 和 monitorexit 来完成。相对于普通方法，其常量池中多了 ACC_SYNCHRONIZED 标示符。实质上 JVM 是根据该标示符来实现方法的同步的，当方法被调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取 Monitor 锁，获取成功之后才去执行方法体，并在方法执行完后释放 Monitor 锁。同时，在方法执行期间，其他任何线程都无法再获得同一个 Monitor 锁对象。
- 方法的同步和代码块的同步没有本质区别，只是其用一种隐式的方式来实现，无需通过字节码来完成。

### 三板斧之二：非阻塞同步

- 根据上一小节我们可以知道，在进行互斥同步时，无论共享的数据是否真的存在竞争，它都会进行加锁操作，从而导致用户态与核心态的转换、维护锁计数器以及检查是否有等待锁的线程需要被唤醒等额外开销，因此互斥同步属于一种悲观的并发策略。
- 那么是否存在一种乐观的并发策略呢？答案是有的，目前在 Java 中实现了一种基于冲突检测的加锁策略 ———— CAS 操作。
- 通俗的说就是先不管是否存在竞争，先进行操作，一旦产生了冲突，再通过其他补偿手段进行修正。最常见的就是通过不断地重试，直到没有竞争为止。
- 这种策略地好处在于全程是处于用户态中进行操作，从而避免了频繁地用户态与核心态之间的切换操作。

1. 直到 JDK 5 ，在 java.util.concurrent.atomic 包中才提供了一些类支持原子级别的 CAS 操作，包括 AtomicBoolean、AtomicInteger、AtomicLong 等，而这些类的方法大多数又是调用的 sun.misc.Unsafe 类里面的 compareAndSwapInt() 和 compareAndSwapLong() 等几个保证原子操作的方法。

- 以 java.util.concurrent.atomic.AtomicInteger 类的 getAndIncrement() 方法为例：

```Java
public class AtomicInteger extends Number implements java.io.Serializable {

    static {
        try {
            //获取 value 变量的偏移量, 赋值给 valueOffset
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    /**
     * Atomically increments by one the current value.
     *
     * @return the previous value
     */
    public final int getAndIncrement() {
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }

    ...other methods...
}

/*==========================================*/

public final class Unsafe {
    public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            //通过对象和偏移量获取变量的值
    	    //由于 volatile 的修饰, 因此所有线程看到的 var5 都是一样的
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }

    ...other methods...
}
```

- 我们可以看到 Unsafe 类的 getAndAddInt() 方法中存在一个 do while 循环，而循环条件中的 compareAndSwapInt() 方法会以原子的方式尝试修改 var5 的值。
- 具体而言，该方法通过 obj 和 valueOffset 获取变量的值，如果这个值和 var5 不一样，说明其他线程已经先一步修改了 obj + valueOffset 地址处的值，此时 compareAndSwapInt() 返回 false，继续循环；如果这个值和 var5 一样，说明没有其他线程修改 obj + valueOffset 地址处的值，此时可以将 obj + valueOffset 地址处的值改为 var5 + var4 ，compareAndSwapInt() 返回 true，退出循环。由于 compareAndSwapInt() 方法是原子操作, 所以compareAndSwapInt() 修改 obj + valueOffset 地址处的值时不会被其他线程中断。

1. 通过上面的例子我们可以发现，使用 CAS 来实现同步操作也引发了一些新的问题：

- 如果自旋 CAS 长时间不成功，就会白白浪费本来就宝贵的 CPU 时间；
- 理论上而言，CAS 也只能保证一个共享变量的原子操作，功能上并没有 synchronized 同步代码块丰富；
- ABA问题：我们可以假设这样一种场景，如果一个值原来是A，变成了B，之后又变回了A，那么在使用 CAS 操作进行检查时会出现以为它的值没有发生变化，而实际上已经变化了的情况。不过实际上即使出现了 ABA 问题在大部分并发情况下也不会影响程序的并发正确性，如果证实确实存在影响，那么最好改用 synchronized 同步代码块来实现同步操作。

### 三板斧之三：无同步线程安全

- 其实，同步与否与是否线程安全没有必然联系，同步只是实现线程安全的一种手段，如果存在有竞争的共享数据那么使用同步手段来保证线程安全也不失为一种好的方案，但如果本来就不存在竞争的可能，那它本身就有隐式的线程安全保证。

1. 可重入代码（纯代码）

> 是一种允许多个进程同时访问的代码。程序在运行过程中可以被打断，并由开始处再次执行，并且在合理的范围内（多次重入，而不造成堆栈溢出等其他问题），程序可以在被打断处继续执行，且执行结果不受影响。（[可重入代码 | 百度百科](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E5%8F%AF%E9%87%8D%E5%85%A5%E4%BB%A3%E7%A0%81%2F1955592%3Ffr%3Daladdin)）

1. 可重入代码拥有一些共同的特征：

- 不依赖全局变量；
- 不依赖存储在堆上的数据和公用的系统资源；
- 使用到的状态量都由参数传入；
- 不调用其他非可重入的方法； ......

1. 因此，如果一段代码中存在与其他代码的共享变量，只要能保证这些变量的可见范围只在同一个线程内，那么无需同步也能保证线程之间的数据安全性。
2. 在 Java 中，使用了 java.lang.ThreadLocal 类来实现线程本地存储的功能，每个线程的 Thread 对象中都有一个 ThreadLocalMap 对象，这个对象存储了一组以 ThreadLocal.threadLocalHashCode 为键，以本地线程变量为值的 K - v 键值对。由于每个线程的 ThreadLocal.threadLocalHashCode 的值都是独一无二的，因此所映射的值也只能该线程自己才能访问到，也就实现了线程安全。

### 总结

1. 可以使用互斥同步（阻塞同步）的方式，实现共享变量的线程安全，典型例子包括：synchronized 等;
2. 可以使用自旋 CAS 的方式，实现共享变量的线程安全，典型例子包括：sun.misc.Unsafe 类、java.util.concurrent.atomic 包中的 AtomicBoolean、AtomicInteger、AtomicLong 等；
3. 如果可以保证共享变量的可见范围均在同一个线程之内，那么其本身就带有隐式的线程安全性，不需要再做其他显式的同步操作。

### 参考文献

1. 方腾飞, 魏鹏, 程晓明．Java并发编程的艺术 [M]. 北京：机械工业出版社，2015：11-20.
2. 周志明．深入理解Java虚拟机: JVM高级特性与最佳实践（3 版）[M]. 北京：机械工业出版社，2019：471-478.

- 与文章相关的疑问都可在评论区中留言，或者 [笔者的个人博客](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.onexstone.online%2Fposts%2Fcf2f473d) 下进行评论。

# [Java 多线程（五）—— 线程池基础 之 FutureTask源码解析](https://www.cnblogs.com/java-chen-hao/p/10243509.html)



**目录**

- [示例](https://www.cnblogs.com/java-chen-hao/p/10243509.html#_label0)
- FutureTask 源码分析
  - [FutureTask 类结构](https://www.cnblogs.com/java-chen-hao/p/10243509.html#_label1_0)
  - [RunnableFuture 接口](https://www.cnblogs.com/java-chen-hao/p/10243509.html#_label1_1)
  - [Future 接口](https://www.cnblogs.com/java-chen-hao/p/10243509.html#_label1_2)
  - [run 方法](https://www.cnblogs.com/java-chen-hao/p/10243509.html#_label1_3)
- [总结](https://www.cnblogs.com/java-chen-hao/p/10243509.html#_label2)
- [ ](https://www.cnblogs.com/java-chen-hao/p/10243509.html#_label3)

 

**正文**

FutureTask是一个支持取消行为的异步任务执行器。该类实现了Future接口的方法。
如：

1. 取消任务执行
2. 查询任务是否执行完成
3. 获取任务执行结果（”get“任务必须得执行完成才能获取结果，否则会阻塞直至任务完成）。
   注意：一旦任务执行完成或取消任务，则不能执行取消任务或者重新启动任务。(除非一开始就使用runAndReset模式运行任务)

FutureTask实现了Runnable接口和Future接口，因此FutureTask可以传递到线程对象Thread或Excutor(线程池)来执行。

如果在当前线程中需要执行比较耗时的操作，但又不想阻塞当前线程时，可以把这些作业交给FutureTask，另开一个线程在后台完成，当当前线程将来需要时，就可以通过FutureTask对象获得后台作业的计算结果或者执行状态。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10243509.html#_labelTop)

## 示例

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class FutureTaskDemo {
 2     public static void main(String[]args)throws InterruptedException {
 3         FutureTask < Integer > ft = new FutureTask <  > (new Callable < Integer > () {
 4                  @Override 
 5                  public Integer call()throws Exception {
 6                     int num = new Random().nextInt(10);
 7                     TimeUnit.SECONDS.sleep(num);
 8                     return num;
 9                 }
10             });
11         Thread t = new Thread(ft);
12         t.start(); 
13         //这里可以做一些其它的事情，跟futureTask任务并行，等需要futureTask的运行结果时，可以调用get方法获取
14         try { 
15             //等待任务执行完成，获取返回值
16             Integer num = ft.get();
17             System.out.println(num);
18         } catch (Exception e) {
19             e.printStackTrace();
20         }
21     }
22 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10243509.html#_labelTop)

## FutureTask 源码分析

JDK1.8自己实现了一个同步等待队列，在结果返回之前，所有的线程都被阻塞，存放到等待队列中。

下面我们来分析下JDK1.8的FutureTask 源码



### FutureTask 类结构

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class FutureTask<V> implements RunnableFuture<V> { 
 2 /** * 当前任务的运行状态。 
 3 * 
 4 * 可能存在的状态转换 
 5 * NEW -> COMPLETING -> NORMAL（有正常结果） 
 6 * NEW -> COMPLETING -> EXCEPTIONAL（结果为异常） 
 7 * NEW -> CANCELLED（无结果） 
 8 * NEW -> INTERRUPTING -> INTERRUPTED（无结果） 
 9 */ 
10 private volatile int state; 
11 private static final int NEW = 0; //初始状态 
12 private static final int COMPLETING = 1; //结果计算完成或响应中断到赋值给返回值之间的状态。 
13 private static final int NORMAL = 2; //任务正常完成，结果被set 
14 private static final int EXCEPTIONAL = 3; //任务抛出异常 
15 private static final int CANCELLED = 4; //任务已被取消 
16 private static final int INTERRUPTING = 5; //线程中断状态被设置ture，但线程未响应中断 
17 private static final int INTERRUPTED = 6; //线程已被中断 
18 
19 //将要执行的任务 
20 private Callable<V> callable; //用于get()返回的结果，也可能是用于get()方法抛出的异常 
21 private Object outcome; // non-volatile, protected by state reads/writes //执行callable的线程，调用FutureTask.run()方法通过CAS设置 
22 private volatile Thread runner; //栈结构的等待队列，该节点是栈中的最顶层节点。 
23 private volatile WaitNode waiters; 
24 .... 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

FutureTask实现的接口信息如下：



### RunnableFuture 接口

```
1 public interface RunnableFuture<V> extends Runnable, Future<V> {
2     void run();
3 }
```

RunnableFuture 接口基础了Runnable和Future接口



### Future 接口

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public interface Future<V> { 
 2     //取消任务 
 3     boolean cancel(boolean mayInterruptIfRunning); 
 4     //判断任务是否已经取消 
 5     boolean isCancelled(); 
 6     //判断任务是否结束（执行完成或取消） 
 7     boolean isDone(); 
 8     //阻塞式获取任务执行结果 
 9     V get() throws InterruptedException, ExecutionException; 
10     //支持超时获取任务执行结果 
11     V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException; 
12 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### run 方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void run() {
 2     //保证callable任务只被运行一次
 3     if (state != NEW || !UNSAFE.compareAndSwapObject(this, runnerOffset, null, Thread.currentThread()))
 4         return;
 5     try {
 6         Callable < V > c = callable;
 7         if (c != null && state == NEW) {
 8             V result;
 9             boolean ran;
10             try { 
11                 //执行任务，上面的例子我们可以看出，call()里面可能是一个耗时的操作，不过这里是同步的
12                 result = c.call();
13                 //上面的call()是同步的，只有上面的result有了结果才会继续执行
14                 ran = true;
15             } catch (Throwable ex) {
16                 result = null;
17                 ran = false;
18                 setException(ex);
19             }
20             if (ran)
21                 //执行完了，设置result
22                 set(result);
23         }
24     }
25     finally {
26         runner = null;
27         int s = state;
28         //判断该任务是否正在响应中断，如果中断没有完成，则等待中断操作完成
29         if (s >= INTERRUPTING)
30             handlePossibleCancellationInterrupt(s);
31     }
32 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1.如果state状态不为New或者设置运行线程runner失败则直接返回false，说明线程已经启动过，保证任务在同一时刻只被一个线程执行。
2.调用callable.call()方法,如果调用成功则执行set(result)方法，将state状态设置成NORMAL。如果调用失败抛出异常则执行setException(ex)方法，将state状态设置成EXCEPTIONAL，唤醒所有在get()方法上等待的线程。
3.如果当前状态为INTERRUPTING(步骤2已CAS失败)，则一直调用Thread.yield()直至状态不为INTERRUPTING

#### set方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 protected void set(V v) {
2     if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
3         outcome = v;
4         UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
5         finishCompletion();
6     }
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

1. 首先通过CAS把state的NEW状态修改成COMPLETING状态。
2. 修改成功则把v值赋给outcome变量。然后再把state状态修改成NORMAL，表示现在可以获取返回值。
3. 最后调用finishCompletion()方法，唤醒等待队列中的所有节点。

#### finishCompletion方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void finishCompletion() {
 2     for (WaitNode q; (q = waiters) != null; ) { 
 3         //通过CAS把栈顶的元素置为null，相当于弹出栈顶元素
 4         if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
 5             for (; ; ) {
 6                 Thread t = q.thread;
 7                 if (t != null) {
 8                     q.thread = null;
 9                     LockSupport.unpark(t);
10                 }
11                 WaitNode next = q.next;
12                 if (next == null)
13                     break;
14                 q.next = null; // unlink to help gc
15                 q = next;
16             }
17             break;
18         }
19     }
20     done();
21     callable = null; // to reduce footprint
22 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

把栈中的元素一个一个弹出，并通过 LockSupport.unpark(t)唤醒每一个节点，通知每个线程，该任务执行完成（可能是执行完成，也可能cancel，异常等）

### runAndReset 方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 protected boolean runAndReset() {
 2     if (state != NEW ||
 3         !UNSAFE.compareAndSwapObject(this, runnerOffset,
 4                                      null, Thread.currentThread()))
 5         return false;
 6     boolean ran = false;
 7     int s = state;
 8     try {
 9         Callable<V> c = callable;
10         if (c != null && s == NEW) {
11             try {
12                 // 执行任务，和run方法不同的是这里不需要设置返回值
13                 c.call(); // don't set result
14                 ran = true;
15             } catch (Throwable ex) {
16                 setException(ex);
17             }
18         }
19     } finally {
20         // runner must be non-null until state is settled to
21         // prevent concurrent calls to run()
22         runner = null;
23         // state must be re-read after nulling runner to prevent
24         // leaked interrupts
25         s = state;
26         if (s >= INTERRUPTING)
27             handlePossibleCancellationInterrupt(s);
28     }
29     //这里并没有改变state的状态，还是NEW状态
30     return ran && s == NEW;
31 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
runAndReset()和run()方法最大的区别是 runAndReset 不需要设置返回值，并且不需要改变任务的状态，也就是不改变state的状态，一直是NEW状态。
```

### get方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public V get()throws InterruptedException, ExecutionException {
2     int s = state;
3     if (s <= COMPLETING)
4         s = awaitDone(false, 0L);
5     return report(s);
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果state状态小于等于COMPLETING，说明任务还没开始执行或还未执行完成，然后调用awaitDone方法阻塞该调用线程。

如果state的状态大于COMPLETING，则说明任务执行完成，或发生异常、中断、取消状态。直接通过report方法返回执行结果。

#### awaitDone 方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private int awaitDone(boolean timed, long nanos)throws InterruptedException {
 2     final long deadline = timed ? System.nanoTime() + nanos : 0L;
 3     WaitNode q = null;
 4     boolean queued = false;
 5     for (; ; ) { 
 6         //如果该线程执行interrupt()方法，则从队列中移除该节点，并抛出异常
 7         if (Thread.interrupted()) {
 8             removeWaiter(q);
 9             throw new InterruptedException();
10         }
11         int s = state; 
12         //如果state状态大于COMPLETING 则说明任务执行完成，或取消
13         if (s > COMPLETING) {
14             if (q != null)
15                 q.thread = null;
16             return s;
17         } 
18         //如果state=COMPLETING，则使用yield，因为此状态的时间特别短，通过yield比挂起响应更快。
19         else if (s == COMPLETING) // cannot time out yet
20             Thread.yield(); 
21         //构建节点
22         else if (q == null)
23             q = new WaitNode();
24         //把当前节点入栈
25         else if (!queued)
26             queued = UNSAFE.compareAndSwapObject(this, waitersOffset, q.next = waiters, q);
27         //如果需要阻塞指定时间，则使用LockSupport.parkNanos阻塞指定时间
28         //如果到指定时间还没执行完，则从队列中移除该节点，并返回当前状态
29         else if (timed) {
30             nanos = deadline - System.nanoTime();
31             if (nanos <= 0L) {
32                 removeWaiter(q);
33                 return state;
34             }
35             LockSupport.parkNanos(this, nanos);
36         }
37         //阻塞当前线程
38         else
39             LockSupport.park(this);
40     }
41 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

构建栈链表的节点元素，并将该节点入栈，同时阻塞当前线程等待运行主任务的线程唤醒该节点。

#### report方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 private V report(int s)throws ExecutionException {
2     Object x = outcome;
3     if (s == NORMAL)
4         return (V)x;
5     if (s >= CANCELLED)
6         throw new CancellationException();
7     throw new ExecutionException((Throwable)x);
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果state的状态为NORMAL，说明任务正确执行完成，直接返回计算后的值。
如果state的状态大于等于CANCELLED，说明任务被成功取消执行、或响应中断，直接返回CancellationException异常
否则返回ExecutionException异常。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10243509.html#_labelTop)

## 总结

　　1.任务开始运行后，不能在次运行，保证只运行一次（runAndReset 方法除外）
　　2.任务还未开始，或者任务已被运行，但未结束，这两种情况下都可以取消； 如果任务已经结束，则不可以被取消 。

# [并发编程（一）—— volatile关键字和 atomic包](https://www.cnblogs.com/java-chen-hao/p/9968544.html)



**目录**

- [Java内存模型](https://www.cnblogs.com/java-chen-hao/p/9968544.html#_label0)
- 并发编程中的三个概念
  - [1.原子性](https://www.cnblogs.com/java-chen-hao/p/9968544.html#_label1_0)
  - [2.可见性](https://www.cnblogs.com/java-chen-hao/p/9968544.html#_label1_1)
  - [3.有序性](https://www.cnblogs.com/java-chen-hao/p/9968544.html#_label1_2)
- volatile关键字
  - [1、共享变量的可见性](https://www.cnblogs.com/java-chen-hao/p/9968544.html#_label2_0)
  - [2、禁止进行指令重排序](https://www.cnblogs.com/java-chen-hao/p/9968544.html#_label2_1)
- [Atomic包](https://www.cnblogs.com/java-chen-hao/p/9968544.html#_label3)

 

**正文**

**本文将讲解volatile关键字和 atomic包，为什么放到一起讲呢，主要是因为这两个可以解决并发编程中的原子性、可见性、有序性，让我们一起来看看吧。**





 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/9968544.html#_labelTop)

## **Atomic包**

**在java 1.5的java.util.concurrent.atomic包下提供了一些原子操作类，即对基本数据类型的 自增（加1操作），自减（减1操作）、以及加法操作（加一个数），减法操作（减一个数）进行了封装，保证这些操作是原子性操作。atomic是利用CAS来实现原子性操作的（Compare And Swap）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package com.mmall.concurrency.example.count;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author: ChenHao
 * @Description:
 * @Date: Created in 15:05 2018/11/16
 * @Modified by:
 */
public class CountTest {
    // 请求总数
    public static int clientTotal = 5000;
    public static AtomicInteger count = new AtomicInteger(0);

    public static void main(String[] args) throws Exception {
        //使用CountDownLatch来等待计算线程执行完
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        //开启clientTotal个线程进行累加操作
        for(int i=0;i<clientTotal;i++){
            new Thread(){
                public void run(){
                    count.incrementAndGet();//先加1，再get到值
                    countDownLatch.countDown();
                }
            }.start();
        }
        //等待计算线程执行完
        countDownLatch.await();
        System.out.println(count);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

执行结果:

![img](https://img2018.cnblogs.com/blog/1168971/201811/1168971-20181116153150280-1858052321.png)

下面我们来看看原子类操作的基本原理

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public final int incrementAndGet() {
 2      return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
 3 }
 4 
 5 public final int getAndAddInt(Object var1, long var2, int var4) {
 6     int var5;
 7     do {
 8         var5 = this.getIntVolatile(var1, var2);
 9     } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
10 
11     return var5;
12 }
13 
14 /***
15 * 获取obj对象中offset偏移地址对应的整型field的值。
16 * @param obj 包含需要去读取的field的对象
17 * @param obj中整型field的偏移量
18 */
19 public native int getIntVolatile(Object obj, long offset);
20 
21 /**
22 * 比较obj的offset处内存位置中的值和期望的值，如果相同则更新。此更新是不可中断的。
23 * 
24 * @param obj 需要更新的对象
25 * @param offset obj中整型field的偏移量
26 * @param expect 希望field中存在的值
27 * @param update 如果期望值expect与field的当前值相同，设置filed的值为这个新值
28 * @return 如果field的值被更改返回true
29 */
30 public native boolean compareAndSwapInt(Object obj, long offset, int expect, int update);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**首先介绍一下什么是Compare And Swap(CAS)？简单的说就是比较并交换。**

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。” Java并发包(java.util.concurrent)中大量使用了CAS操作,涉及到并发的地方都调用了sun.misc.Unsafe类方法进行CAS操作。

 我们来分析下incrementAndGet的逻辑：

　　1.先获取当前的value值

　　2.调用compareAndSet方法来来进行原子更新操作，这个方法的语义是：

　　　　**先检查当前value是否等于obj中整型field的偏移量处的值，如果相等，则意味着\**obj中整型field的偏移量处的值\** 没被其他线程修改过，更新并返回true。如果不相等，compareAndSet则会返回false，然后循环继续尝试更新。**

**第一次count 为0时**线程A调用incrementAndGet时，传参为 var1=AtomicInteger(0)，var2为var1 里面 0 的偏移量，比如为8090，var4为需要加的数值1,var5为线程工作内存值，do里面会先执行一次，通过getIntVolatile 获取obj对象中offset偏移地址对应的整型field的值此时var5=0;while 里面compareAndSwapInt 比较obj的8090处内存位置中的值和期望的值var5，如果相同则更新obj的值为(var5+var4=1)，此时更新成功，返回true,则 while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));结束循环，return var5。

**当count 为0时，线程B 和线程A 同时读取到 count** ,进入到第 8 行代码处，线程B 也是取到的var5=0,当线程B 执行到compareAndSwapInt时，线程A已经执行完compareAndSwapInt，已经将内存地址为8090处的值修改为1，此时线程B 执行compareAndSwapInt返回false,则继续循环执行do里面的语句，再次取内存地址偏移量为8090处的值为1，再去执行compareAndSwapInt，更新obj的值为(var5+var4=2)，返回为true,结束循环，return var5。

 

**CAS的ABA问题**

　　当然CAS也并不完美，它存在"ABA"问题，假若一个变量初次读取是A，在compare阶段依然是A，但其实可能在此过程中，它先被改为B，再被改回A，而CAS是无法意识到这个问题的。CAS只关注了比较前后的值是否改变，而无法清楚在此过程中变量的变更明细，这就是所谓的ABA漏洞。 

# [并发编程（二）—— CountDownLatch、CyclicBarrier和Semaphore](https://www.cnblogs.com/java-chen-hao/p/9970581.html)



**目录**

- CountDownLatch
  - [CountDownLatch是什么?](https://www.cnblogs.com/java-chen-hao/p/9970581.html#_label0_0)
- CyclicBarrier
  - [CyclicBarrier是什么?](https://www.cnblogs.com/java-chen-hao/p/9970581.html#_label1_0)
  - [CyclicBarrier使用例子](https://www.cnblogs.com/java-chen-hao/p/9970581.html#_label1_1)
  - [CyclicBarrier的应用场景](https://www.cnblogs.com/java-chen-hao/p/9970581.html#_label1_2)
  - [CyclicBarrier和CountDownLatch的区别](https://www.cnblogs.com/java-chen-hao/p/9970581.html#_label1_3)
- Semaphore
  - [Semaphore是什么?](https://www.cnblogs.com/java-chen-hao/p/9970581.html#_label2_0)

 

**正文**

**本文将讲解CountDownLatch，CyclicBarrier和Semaphore这三个并发包里面的辅助类。**

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/9970581.html#_labelTop)

## ***\*CountDownLatch\****

   正如每个Java文档所描述的那样，CountDownLatch 是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程的操作执行完后再执行。



### CountDownLatch是什么?

　　CountDownLatch是在java1.5被引入的，跟它一起被引入的并发工具类还有CyclicBarrier、Semaphore、ConcurrentHashMap和BlockingQueue，它们都存在于java.util.concurrent包下。CountDownLatch这个类能够使一个线程等待其他线程完成各自的工作后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。

　　CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

![img](https://img2018.cnblogs.com/blog/1168971/201811/1168971-20181123110911033-1163155530.png)

CountDownLatch类只提供了一个构造器：

```
public CountDownLatch(int count) {  };  //参数count为计数值
```

然后下面这3个方法是CountDownLatch类中最重要的方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void await() throws InterruptedException { };   //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public void countDown() { };  //将count值减1
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

构造器中的**计数值（count）实际上就是闭锁需要等待的线程数量**。这个值只能被设置一次，而且CountDownLatch**没有提供任何机制去重新设置这个计数值**。

与CountDownLatch的第一次交互是主线程等待其他线程。主线程必须在启动其他线程后立即调用**CountDownLatch.await()**方法。这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。

其他N 个线程必须引用闭锁对象，因为他们需要通知CountDownLatch对象，他们已经完成了各自的任务。这种通知机制是通过 **CountDownLatch.countDown()**方法来完成的；每调用一次这个方法，在构造函数中初始化的count值就减1。所以当N个线程都调 用了这个方法，count的值等于0，然后主线程就能通过await()方法，恢复执行自己的任务。

### CountDownLatch使用例子

　　比如对于马拉松比赛，进行排名计算，参赛者的排名，肯定是跑完比赛之后，进行计算得出的，翻译成Java识别的预发，就是N个线程执行操作，主线程等到N个子线程执行完毕之后，再继续往下执行。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
/**
 * @author: ChenHao
 * @Description:
 * @Date: Created in 11:05 2018/11/23
 * @Modified by:马拉松比赛
 */
public class CountdownLatchTest {

    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
        final CountDownLatch cdOrder = new CountDownLatch(1);
        final CountDownLatch cdAnswer = new CountDownLatch(3);
        for(int i=0;i<3;i++){
            Runnable runnable = new Runnable(){
                @Override
                public void run(){
                    try {
                        System.out.println("运动员" + Thread.currentThread().getName() + "等待信号枪");
                        cdOrder.await();
                        System.out.println("运动员" + Thread.currentThread().getName() + "开跑");
                        Thread.sleep((long)(Math.random()*10000));
                        System.out.println("运动员" + Thread.currentThread().getName() + "到达终点！");
                        cdAnswer.countDown();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            };
            service.execute(runnable);
        }
        try {
            Thread.sleep(5000);

            System.out.println("裁判" + Thread.currentThread().getName() + "即将鸣信号枪");
            cdOrder.countDown();
            System.out.println("裁判" + Thread.currentThread().getName() + "已经鸣枪，等待运动员跑完");
            cdAnswer.await();
            System.out.println("三个运动员都跑到了终点，裁判"+ Thread.currentThread().getName() +"统计名次" );
        } catch (Exception e) {
            e.printStackTrace();
        }
        service.shutdown();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201811/1168971-20181123112033498-1890644557.png)

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/9970581.html#_labelTop)

## **CyclicBarrier**

   字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。



### **CyclicBarrier**是什么?

　　CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。



### **CyclicBarrier**使用例子

 实例代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class CyclicBarrierTest {

    static CyclicBarrier c = new CyclicBarrier(2);

    public static void main(String[] args) {
        new Thread(new Runnable() {

            @Override
            public void run() {
                try {
                    c.await();
                } catch (Exception e) {

                }
                System.out.println(1);
            }
        }).start();

        try {
            c.await();
        } catch (Exception e) {

        }
        System.out.println(2);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

输出

```
2
1
```

或者输出

```
1
2
```

如果把new CyclicBarrier(2)修改成new CyclicBarrier(3)则主线程和子线程会永远等待，因为没有第三个线程执行await方法，即没有第三个线程到达屏障，所以之前到达屏障的两个线程都不会继续执行。

CyclicBarrier还提供一个更高级的构造函数CyclicBarrier(int parties, Runnable barrierAction)，用于在线程到达屏障时，优先执行barrierAction，方便处理更复杂的业务场景。代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class CyclicBarrierTest2 {

    static CyclicBarrier c = new CyclicBarrier(2, new A());

    public static void main(String[] args) {
        new Thread(new Runnable() {

            @Override
            public void run() {
                try {
                    c.await();
                } catch (Exception e) {

                }
                System.out.println(1);
            }
        }).start();

        try {
            c.await();
        } catch (Exception e) {

        }
        System.out.println(2);
    }

    static class A implements Runnable {

        @Override
        public void run() {
            System.out.println(3);
        }

    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

输出

```
3
1
2
```

下面我们来看看Barrier循环使用的例子，下面例子中getNumberWaiting方法可以获得CyclicBarrier阻塞的线程数量

周末公司组织大巴去旅游，总共有三个景点，每个景点约定好游玩时间，一个景点结束后需要集中一起出发到下一个景点。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 import java.util.concurrent.CyclicBarrier;
 2 import java.util.concurrent.ExecutorService;
 3 import java.util.concurrent.Executors;
 4 
 5 public class CyclicBarrierTest {
 6 
 7     public static void main(String[] args) {
 8         ExecutorService service = Executors.newCachedThreadPool();
 9         final  CyclicBarrier cb = new CyclicBarrier(3);
10         for(int i=0;i<3;i++){
11             Runnable runnable = new Runnable(){
12                 public void run(){
13                     try {
14                         Thread.sleep((long)(Math.random()*10000));
15                         System.out.println("线程" + Thread.currentThread().getName() + "即将到达集合地点1，当前已有" + (cb.getNumberWaiting()+1) + "个已经到达，" + (cb.getNumberWaiting()==2?"都到齐了，继续走啊":"正在等候"));
16                         cb.await();
17 
18                         Thread.sleep((long)(Math.random()*10000));
19                         System.out.println("线程" + Thread.currentThread().getName() + "即将到达集合地点2，当前已有" + (cb.getNumberWaiting()+1) + "个已经到达，" + (cb.getNumberWaiting()==2?"都到齐了，继续走啊":"正在等候"));
20                         cb.await();
21                         Thread.sleep((long)(Math.random()*10000));
22                         System.out.println("线程" + Thread.currentThread().getName() + "即将到达集合地点3，当前已有" + (cb.getNumberWaiting() + 1) + "个已经到达，" + (cb.getNumberWaiting()==2?"都到齐了，继续走啊":"正在等候"));
23                         cb.await();
24                     } catch (Exception e) {
25                         e.printStackTrace();
26                     }
27                 }
28             };
29             service.execute(runnable);
30         }
31         service.shutdown();
32     }
33 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201811/1168971-20181123120735616-1737363979.png)

结果分析：第9行设置需要拦截的线程数为3，三个人一起出发先到第一个景点游玩，第一个景点游玩结束后，第一个到达集合地点一的时候，cb.getNumberWaiting()为0，所以当前有1个已经到达，到16行代码cb.await()处第一个到达的人开始等待剩余两人到达；

第二人到达后等待第三人，第三人到达时，cb.getNumberWaiting()为2，表示前面等待的人数为2，此时三个人都到达了集合地点一，同时出发前往集合地点二，此时接着执行第18行代码；同理三个人都到达集合地点二后再前往集合地点三，也就是执行第21行代码。



### CyclicBarrier的应用场景

CyclicBarrier可以用于多线程计算数据，最后合并计算结果的应用场景。比如我们用一个Excel保存了用户所有银行流水，每个Sheet保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，都执行完之后，得到每个sheet的日均银行流水，最后，再用barrierAction用这些线程的计算结果，计算出整个Excel的日均银行流水。



### CyclicBarrier和CountDownLatch的区别

- CountDownLatch的计数器只能使用一次。而CyclicBarrier的计数器可以使用reset() 方法重置。所以CyclicBarrier能处理更为复杂的业务场景，比如如果计算发生错误，可以重置计数器，并让线程们重新执行一次。
- CyclicBarrier还提供其他有用的方法，比如getNumberWaiting方法可以获得CyclicBarrier阻塞的线程数量。isBroken方法用来知道阻塞的线程是否被中断。

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/9970581.html#_labelTop)

## Semaphore

   Semaphore翻译成字面意思为 信号量，Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。



### Semaphore是什么?

　　emaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源。把它比作是控制流量的红绿灯，比如一条马路要限制流量，只允许同时有一百辆车在这条路上行使，其他的都必须在路口等待，所以前一百辆车会看到绿灯，可以开进这条马路，后面的车会看到红灯，不能驶入马路，但是如果前一百辆中有五辆车已经离开了马路，那么后面就允许有5辆车驶入马路，这个例子里说的车就是线程，驶入马路就表示线程在执行，离开马路就表示线程执行完成，看见红灯就表示线程被阻塞，不能执行。

Semaphore类位于java.util.concurrent包下，它提供了2个构造器：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Semaphore(int permits) {          //参数permits表示许可数目，即同时可以允许多少线程进行访问
    sync = new NonfairSync(permits);
}
public Semaphore(int permits, boolean fair) {    //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

下面说一下Semaphore类中比较重要的几个方法，首先是acquire()、release()方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void acquire() throws InterruptedException {  }     //获取一个许可
public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
public void release() { }          //释放一个许可
public void release(int permits) { }    //释放permits个许可
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　acquire()用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。

　　release()用来释放许可。注意，在释放许可之前，必须先获获得许可。　

  这4个方法都会被阻塞，如果想立即得到执行结果，可以使用下面几个方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public boolean tryAcquire() { };    //尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { };  //尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(int permits) { }; //尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { }; //尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### Semaphore使用例子

　　假若一个工厂有5台机器，但是有8个工人，一台机器同时只能被一个工人使用，只有使用完了，其他工人才能继续使用。那么我们就可以通过Semaphore来实现：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Test {
    public static void main(String[] args) {
        int N = 8;            //工人数
        Semaphore semaphore = new Semaphore(5); //机器数目
        for(int i=0;i<N;i++)
            new Worker(i,semaphore).start();
    }
     
    static class Worker extends Thread{
        private int num;
        private Semaphore semaphore;
        public Worker(int num,Semaphore semaphore){
            this.num = num;
            this.semaphore = semaphore;
        }
         
        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("工人"+this.num+"占用一个机器在生产...");
                Thread.sleep(2000);
                System.out.println("工人"+this.num+"释放出机器");
                semaphore.release();           
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

执行结果：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
工人0占用一个机器在生产...
工人1占用一个机器在生产...
工人2占用一个机器在生产...
工人4占用一个机器在生产...
工人5占用一个机器在生产...
工人0释放出机器
工人2释放出机器
工人3占用一个机器在生产...
工人7占用一个机器在生产...
工人4释放出机器
工人5释放出机器
工人1释放出机器
工人6占用一个机器在生产...
工人3释放出机器
工人7释放出机器
工人6释放出机器
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

# [并发编程（三）—— ReentrantLock的用法](https://www.cnblogs.com/java-chen-hao/p/10037209.html)



**目录**

- 可重入性/公平锁/非公平锁
  - [可重入性](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label0_0)
  - [公平锁/非公平锁](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label0_1)
- 源码分析
  - [1、无参构造器（默认为非公平锁）](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label1_0)
  - [2、带布尔值的构造器（是否公平）](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label1_1)
  - [3、lock()](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label1_2)
  - [4、unlock()](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label1_3)
  - [5、tryLock()](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label1_4)
  - [6、newCondition()](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label1_5)
  - [7、await（）](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label1_6)
  - [8、signal（）](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label1_7)
  - [9、创建多个condition对象](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label1_8)
- [ABC循环打印20遍](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label2)
- [ 总结](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_label3)

 

**正文**

　　ReentrantLock是Java并发包中提供的一个**可重入的互斥锁**。**ReentrantLock**和**synchronized**在基本用法，行为语义上都是类似的，同样都具有可重入性。只不过相比原生的Synchronized，ReentrantLock增加了一些高级的扩展功能，比如它可以实现**公平锁，**同时也可以绑定**多个Conditon**。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_labelTop)

## 可重入性/公平锁/非公平锁



### **可重入性**

   所谓的可重入性，就是**可以支持一个线程对锁的重复获取**，原生的synchronized就具有可重入性，一个用synchronized修饰的递归方法，当线程在执行期间，它是可以反复获取到锁的，而不会出现自己把自己锁死的情况。ReentrantLock也是如此，在调用lock()方法时，已经获取到锁的线程，能够再次调用lock()方法获取锁而不被阻塞。



### **公平锁/非公平锁**

　　所谓公平锁,顾名思义，意指锁的获取策略相对公平，当多个线程在获取同一个锁时，必须按照锁的申请时间来依次获得锁，排排队，不能插队；非公平锁则不同，当锁被释放时，等待中的线程均有机会获得锁。synchronized是非公平锁，ReentrantLock默认也是非公平的，但是可以通过带boolean参数的构造方法指定使用公平锁，但**非公平锁的性能一般要优于公平锁。**

　　synchronized是Java原生的互斥同步锁，使用方便，对于synchronized修饰的方法或同步块，无需再显式释放锁。而ReentrantLock做为API层面的互斥锁，需要显式地去加锁解锁。采用Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。因此一般来说，使用Lock必须在try{}catch{}块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class X {
    private final ReentrantLock lock = new ReentrantLock();
    // ...
 
    public void m() {
      lock.lock();  // 加锁
      try {
        // ... 函数主题
      } finally {
        lock.unlock() //解锁
      }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_labelTop)

## 源码分析

　　接下来我们从源码角度来看看ReentrantLock的实现原理，它是如何保证可重入性，又是如何实现公平锁的。



### **1、无参构造器（默认为非公平锁）**

```
public ReentrantLock() {
     sync = new NonfairSync();//默认是非公平的
}
```

sync是ReentrantLock内部实现的一个同步组件，它是Reentrantlock的一个静态内部类，继承于AQS。



### **2、带布尔值的构造器（是否公平）**

```
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();//fair为true，公平锁；反之，非公平锁
}
```

此处可以指定是否采用公平锁，**FailSync和NonFailSync亦为Reentrantlock的静态内部类，都继承于Sync**。



### 3、**lock()**

```
public void lock() {
        sync.lock();//代理到Sync的lock方法上
}
```

Sync的lock方法是抽象的，实际的lock会代理到FairSync或是NonFairSync上（根据用户的选择来决定，公平锁还是非公平锁）



### 4、**unlock()**

```
public void unlock() {
        sync.release(1);//释放锁
}
```

释放锁，调用sync的release方法。



### 5、tryLock()

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){
         
     }finally{
         lock.unlock();   //释放锁
     } 
}else {
    //如果不能获取锁，则直接做其他事情
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

tryLock()方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false。



### 6、**newCondition()**

```
public Condition newCondition() {
        return sync.newCondition();
}
```

获取一个conditon，ReentrantLock支持多个Condition



### 7、await（）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MyService {

    private Lock lock = new ReentrantLock();
    private Condition condition=lock.newCondition();
    public void testMethod() {
        
        try {
            lock.lock();
            System.out.println("开始wait");
            condition.await();
            for (int i = 0; i < 5; i++) {
                System.out.println("ThreadName=" + Thread.currentThread().getName()
                        + (" " + (i + 1)));
            }
        } catch (InterruptedException e) {
            // TODO 自动生成的 catch 块
            e.printStackTrace();
        }
        finally
        {
            lock.unlock();
        }
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过创建Condition对象来使线程wait，必须先执行lock.lock方法获得锁



### 8、signal（）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void signal() {
        try {
            lock.lock();
            condition.signal();
        } finally {
            lock.unlock();
        }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

condition对象的signal方法可以唤醒wait线程



### 9、创建多个condition对象

　　一个condition对象的signal（signalAll）方法和该对象的await方法是一一对应的，也就是一个condition对象的signal（signalAll）方法不能唤醒其他condition对象的await方法

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_labelTop)

## ABC循环打印20遍

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
  1 package main.java.Juc;
  2 
  3 import java.util.concurrent.locks.Condition;
  4 import java.util.concurrent.locks.Lock;
  5 import java.util.concurrent.locks.ReentrantLock;
  6 
  7 /*
  8  * 编写一个程序，开启 3 个线程，这三个线程的 ID 分别为 A、B、C，每个线程将自己的 ID 在屏幕上打印 10 遍，要求输出的结果必须按顺序显示。
  9  *    如：ABCABCABC…… 依次递归
 10  */
 11 public class TestABCAlternate {
 12     
 13     public static void main(String[] args) {
 14         AlternateDemo ad = new AlternateDemo();
 15         
 16         new Thread(new Runnable() {
 17             @Override
 18             public void run() {
 19                 for (int i = 1; i <= 20; i++) {
 20                     ad.loopA(i);
 21                 }
 22             }
 23         }, "A").start();
 24         
 25         new Thread(new Runnable() {
 26             @Override
 27             public void run() {
 28                 for (int i = 1; i <= 20; i++) {
 29                     ad.loopB(i);
 30                 }
 31             }
 32         }, "B").start();
 33         
 34         new Thread(new Runnable() {
 35             @Override
 36             public void run() {
 37                 for (int i = 1; i <= 20; i++) {
 38                     ad.loopC(i);
 39                     System.out.println("-----------------------------------");
 40                 }
 41             }
 42         }, "C").start();
 43     }
 44 
 45 }
 46 
 47 class AlternateDemo{
 48     
 49     private int number = 1; //当前正在执行线程的标记
 50     
 51     private Lock lock = new ReentrantLock();
 52     private Condition condition1 = lock.newCondition();
 53     private Condition condition2 = lock.newCondition();
 54     private Condition condition3 = lock.newCondition();
 55     
 56     /**
 57      * @param totalLoop : 循环第几轮
 58      */
 59     public void loopA(int totalLoop){
 60         lock.lock();
 61         try {
 62             //1. 判断
 63             if(number != 1){
 64                 condition1.await();
 65             }
 66             //2. 打印
 67             for (int i = 1; i <= 1; i++) {
 68                 System.out.println(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
 69             }
 70             //3. 唤醒
 71             number = 2;
 72             condition2.signal();
 73         } catch (Exception e) {
 74             e.printStackTrace();
 75         } finally {
 76             lock.unlock();
 77         }
 78     }
 79     
 80     public void loopB(int totalLoop){
 81         lock.lock();
 82         try {
 83             //1. 判断
 84             if(number != 2){
 85                 condition2.await();
 86             }
 87             //2. 打印
 88             for (int i = 1; i <= 1; i++) {
 89                 System.out.println(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
 90             }
 91             //3. 唤醒
 92             number = 3;
 93             condition3.signal();
 94         } catch (Exception e) {
 95             e.printStackTrace();
 96         } finally {
 97             lock.unlock();
 98         }
 99     }
100     
101     public void loopC(int totalLoop){
102         lock.lock();
103         try {
104             //1. 判断
105             if(number != 3){
106                 condition3.await();
107             }
108             //2. 打印
109             for (int i = 1; i <= 1; i++) {
110                 System.out.println(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
111             }
112             //3. 唤醒
113             number = 1;
114             condition1.signal();
115         } catch (Exception e) {
116             e.printStackTrace();
117         } finally {
118             lock.unlock();
119         }
120     }
121     
122 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201811/1168971-20181129113045958-1548802925.png)

代码分析：

　　三个线程分别循环20次调用loopA、loopB、loopC打印，但是不确定是哪个方法先被调用到,如果是loopB先调用，则loopB方法先获取到锁，loopA和loopC等待锁，此时线程执行标记number=1，代码84行处为true,则condition2.await();如果需要唤醒此线程，则需要用condition2来唤醒，此时线程交出锁；

　　如果loopA获取了锁，loopB和loopC等待锁，此时线程执行标记number=1，代码63行处为false,则执行67行打印，打印完则用condition2.signal()唤醒打印loopB的线程，接着loopB的线程去打印B，线程loopB打印完毕去唤醒打印loopC的线程，打印完loopC再唤醒loopA，如此循环20次。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10037209.html#_labelTop)

##  总结

 1、Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；

 2、synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；

 3、Lock类可以创建Condition对象，Condition对象用来是线程等待和唤醒线程，需要注意的是Condition对象的唤醒的是用同一个Condition执行await方法的线程，所以也就可以实现唤醒指定类的线程

# [并发编程（四）—— ThreadLocal源码分析及内存泄露预防](https://www.cnblogs.com/java-chen-hao/p/10059735.html)



**目录**

- [ThreadLocal是什么？](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label0)
- API说明
  - [1、ThreadLocal()](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label1_0)
  - [2、T get()](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label1_1)
  - [3、protected T initialValue()](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label1_2)
  - [4、void remove()](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label1_3)
  - [5、void set(T value)](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label1_4)
- [ThreadLocal使用示例](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label2)
- [ThreadLocal源码分析](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label3)
- [ThreadLocal的应用场景](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label4)
- [ ThreadLocal使用的一般步骤](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label5)
- ThreadLocal为什么会内存泄漏
  - [为什么使用弱引用](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label6_0)
  - [ThreadLocal 最佳实践](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label6_1)
- [总结](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_label7)

 

**正文**

今天我们一起探讨下ThreadLocal的实现原理和源码分析。首先，本文先谈一下对ThreadLocal的理解，然后根据ThreadLocal类的源码分析了其实现原理和使用需要注意的地方，最后给出了两个应用场景。相信本文一定能让大家完全了解ThreadLocal。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_labelTop)

## ThreadLocal是什么？

　　ThreadLocal是啥？以前面试别人时就喜欢问这个，有些伙伴喜欢把它和线程同步机制混为一谈，事实上ThreadLocal与线程同步无关。ThreadLocal虽然提供了一种解决多线程环境下成员变量的问题，但是它并不是解决多线程共享变量的问题。那么ThreadLocal到底是什么呢？

　　ThreadLocal很容易让人望文生义，想当然地认为是一个“本地线程”。其实，ThreadLocal并不是一个Thread，而是Thread的**局部变量**，也许把它命名为ThreadLocalVariable更容易让人理解一些。线程局部变量(ThreadLocal)其实的功用非常简单，就是为每一个使用该变量的线程都提供一个变量值的副本，是Java中一种较为特殊的线程绑定机制，是每一个线程都可以独立地改变自己的副本，而不会和其它线程的副本冲突。

　　通过ThreadLocal存取的数据，总是与当前线程相关，也就是说，JVM 为每个运行的线程，绑定了私有的本地实例存取空间，从而为多线程环境常出现的并发访问问题提供了一种隔离机制。ThreadLocal是如何做到为每一个线程维护变量的副本的呢？其实实现的思路很简单，在ThreadLocal类中有一个Map，用于存储每一个线程的变量的副本。概括起来说，ThreadLocal为每一个线程都提供了一份变量，因此可以同时访问而互不影响。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_labelTop)

## API说明



### **1、ThreadLocal()**

 创建一个线程本地变量。



### **2、T get()**

  返回此线程局部变量的当前线程副本中的值，如果这是线程第一次调用该方法，则创建并初始化此副本。



### 3、protected T initialValue()

  返回此线程局部变量的当前线程的初始值。最多在每次访问线程来获得每个线程局部变量时调用此方法一次，即线程第一次使用 get() 方法访问变量的时候。如果线程先于 get 方法调用 set(T) 方法，则不会在线程中再调用 initialValue 方法。

  若该实现只返回 null；如果程序员希望将线程局部变量初始化为 null 以外的某个值，则必须为 ThreadLocal 创建子类，并重写此方法。通常，将使用匿名内部类。initialValue 的典型实现将调用一个适当的构造方法，并返回新构造的对象。



### 4、void remove()

 移除此线程局部变量的值。这可能有助于减少线程局部变量的存储需求。



### 5、void set(T value)

 将此线程局部变量的当前线程副本中的值设置为指定值。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_labelTop)

## ThreadLocal使用示例

假设我们要为每个线程关联一个唯一的序号，在每个线程周期内，我们需要多次访问这个序号，这时我们就可以使用ThreadLocal了

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 package concurrent;
 2 
 3 import java.util.concurrent.atomic.AtomicInteger;
 4 
 5 /**
 6  * Created by chenhao on 2018/12/03.
 7  */
 8 public class ThreadLocalDemo {
 9     public static void main(String []args){
10         for(int i=0;i<5;i++){
11             final Thread t = new Thread(){
12                 @Override
13                 public void run(){
14                     System.out.println("当前线程:"+Thread.currentThread().getName()+",已分配ID:"+ThreadId.get());
15                 }
16             };
17             t.start();
18         }
19     }
20     static   class ThreadId{
21         //一个递增的序列，使用AtomicInger原子变量保证线程安全
22         private static final AtomicInteger nextId = new AtomicInteger(0);
23         //线程本地变量，为每个线程关联一个唯一的序号
24         private static final ThreadLocal<Integer> threadId =
25                 new ThreadLocal<Integer>() {
26                     @Override
27                     protected Integer initialValue() {
28                         return nextId.getAndIncrement();//相当于nextId++,由于nextId++这种操作是个复合操作而非原子操作，会有线程安全问题(可能在初始化时就获取到相同的ID，所以使用原子变量
29                     }
30                 };
31 
32        //返回当前线程的唯一的序列，如果第一次get，会先调用initialValue，后面看源码就了解了
33         public static int get() {
34             return threadId.get();
35         }
36     }
37 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
当前线程:Thread-4,已分配ID:1
当前线程:Thread-0,已分配ID:0
当前线程:Thread-2,已分配ID:3
当前线程:Thread-1,已分配ID:4
当前线程:Thread-3,已分配ID:2
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_labelTop)

## ThreadLocal源码分析

　　ThreadLocal最常见的操作就是set、get、remove三个动作，下面来看看这三个动作到底做了什么事情。首先看set操作，源码片段

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public void set(T value) {
2     Thread t = Thread.currentThread();
3     ThreadLocalMap map = getMap(t);
4     if (map != null)
5         map.set(this, value);
6     else
7         createMap(t, value);
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

第 2 行代码取出了当前线程 t，然后调用getMap(t)方法时传入了当前线程，换句话说，该方法返回的ThreadLocalMap和当前线程有点关系，我们先记录下来。进一步判定如果这个map不为空，那么设置到Map中的Key就是this，值就是外部传入的参数。这个this是什么呢？就是定义的ThreadLocal对象。
代码中有两条路径需要追踪，分别是getMap(Thread)和createMap(Thread , T)。首先来看看getMap(t)操作

```
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

在这里，我们看到ThreadLocalMap其实就是线程里面的一个属性，它在Thread类中的定义是：

```
ThreadLocal.ThreadLocalMap threadLocals = null;
```

即:每个Thread对象都有一个ThreadLocal.ThreadLocalMap成员变量,ThreadLocal.ThreadLocalMap是一个ThreadLocal类的静态内部类(如下所示),所以Thread类可以进行引用.所以每个线程都会有一个ThreadLocal.ThreadLocalMap对象的引用

```
static class ThreadLocalMap {
```

 

首先获取当前线程的引用,然后获取当前线程的ThreadLocal.ThreadLocalMap对象,如果该对象为空就创建一个,如下所示:

```
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

这个this变量就是ThreadLocal的引用,对于同一个ThreadLocal对象每个线程都是相同的,但是每个线程各自有一个ThreadLocal.ThreadLocalMap对象保存着各自ThreadLocal引用为key的值,所以互不影响,而且:如果你新建一个ThreadLocal的对象,这个对象还是保存在每个线程同一个ThreadLocal.ThreadLocalMap对象之中,因为一个线程只有一个ThreadLocal.ThreadLocalMap对象,这个对象是在第一个ThreadLocal第一次设值的时候进行创建,如上所述的createMap方法.

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ThreadLocalMap(ThreadLocal firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

至此，ThreadLocal的原理我们应该已经清楚了，简单来讲，就是每个Thread里面有一个ThreadLocal.ThreadLocalMap threadLocals作为私有的变量而存在，所以是线程安全的。ThreadLocal通过Thread.currentThread()获取当前的线程就能得到这个Map对象，同时将自身（ThreadLocal对象）作为Key发起写入和读取，由于将自身作为Key，所以一个ThreadLocal对象就能存放一个线程中对应的Java对象，通过get也自然能找到这个对象。

最后来看看get()、remove()代码，或许看到这里就可以认定我们的理论是正确的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public T get() {
 2     Thread t = Thread.currentThread();
 3     ThreadLocalMap map = getMap(t);
 4     if (map != null) {
 5         ThreadLocalMap.Entry e = map.getEntry(this);
 6         if (e != null) {
 7             @SuppressWarnings("unchecked")
 8             T result = (T)e.value;
 9             return result;
10         }
11     }
12     return setInitialValue();
13 }
14 
15 public void remove() {
16      ThreadLocalMap m = getMap(Thread.currentThread());
17      if (m != null)
18          m.remove(this);
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　第一句是取得当前线程，然后通过getMap(t)方法获取到一个map，map的类型为ThreadLocalMap。然后接着下面获取到<key,value>键值对，注意这里获取键值对传进去的是 this，而不是当前线程t。

　　如果获取成功，则返回value值。

　　如果map为空，则调用setInitialValue方法返回value。

　　可以看出第12行处的方法setInitialValue()只有在线程第一次使用 get() 方法访问变量的时候调用。如果线程先于 get 方法调用 set(T) 方法，则不会在线程中再调用 initialValue 方法。

```
protected T initialValue() {
    return null;
}
```

该方法定义为protected级别且返回为null，很明显是要子类实现它的，所以我们在使用ThreadLocal的时候一般都应该覆盖该方法,创建匿名内部类重写此方法。该方法不能显示调用，只有在第一次调用get()或者set()方法时才会被执行，并且仅执行1次。

 

**对于ThreadLocal需要注意的有两点：**
\1. ThreadLocal实例本身是不存储值，它只是提供了一个在当前线程中找到副本值得key。
\2. 是ThreadLocal包含在Thread中，而不是Thread包含在ThreadLocal中，有些小伙伴会弄错他们的关系。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_labelTop)

## ThreadLocal的应用场景

　　最常见的ThreadLocal使用场景为 用来解决 数据库连接、Session管理等。如：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * 数据库连接管理类
 */
public class ConnectionManager {
 
    /** 线程内共享Connection，ThreadLocal通常是全局的，支持泛型 */
    private static ThreadLocal<Connection> threadLocal = new ThreadLocal<Connection>();
    
    public static Connection getCurrConnection() {
        // 获取当前线程内共享的Connection
        Connection conn = threadLocal.get();
        try {
            // 判断连接是否可用
            if(conn == null || conn.isClosed()) {
                // 创建新的Connection赋值给conn(略)
                // 保存Connection
                threadLocal.set(conn);
            }
        } catch (SQLException e) {
            // 异常处理
        }
        return conn;
    }
    
    /**
     * 关闭当前数据库连接
     */
    public static void close() {
        // 获取当前线程内共享的Connection
        Connection conn = threadLocal.get();
        try {
            // 判断是否已经关闭
            if(conn != null && !conn.isClosed()) {
                // 关闭资源
                conn.close();
                // 移除Connection
                threadLocal.remove();
                conn = null;
            }
        } catch (SQLException e) {
            // 异常处理
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

也可以重写initialValue方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private static ThreadLocal<Connection> connectionHolder= new ThreadLocal<Connection>() {
    public Connection initialValue() {
        return DriverManager.getConnection(DB_URL);
    }
};
 
public static Connection getConnection() {
    return connectionHolder.get();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Hiberante的Session 工具类HibernateUtil

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class HibernateUtil {
private static Log log = LogFactory.getLog(HibernateUtil.class);
private static final SessionFactory sessionFactory; //定义SessionFactory
static {
try {
// 通过默认配置文件hibernate.cfg.xml创建SessionFactory
sessionFactory = new Configuration().configure().buildSessionFactory();
} catch (Throwable ex) {
log.error("初始化SessionFactory失败！", ex);
throw new ExceptionInInitializerError(ex);
}
}
//创建线程局部变量session，用来保存Hibernate的Session
public static final ThreadLocal session = new ThreadLocal();
/**
* 获取当前线程中的Session
* @return Session
* @throws HibernateException
*/
public static Session currentSession() throws HibernateException {
Session s = (Session) session.get();
// 如果Session还没有打开，则新开一个Session
if (s == null) {
s = sessionFactory.openSession();
session.set(s); //将新开的Session保存到线程局部变量中
}
return s;
}
public static void closeSession() throws HibernateException {
//获取线程局部变量，并强制转换为Session类型
Session s = (Session) session.get();
session.set(null);
if (s != null)
s.close();
}
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在这个类中，由于没有重写ThreadLocal的initialValue()方法，则首次创建线程局部变量session其初始值为null，第一次调用currentSession()的时候，线程局部变量的get()方法也为null。因此，对session做了判断，如果为null，则新开一个Session，并保存到线程局部变量session中

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_labelTop)

##  ThreadLocal使用的一般步骤

1、在多线程的类（如ThreadDemo类）中，创建一个ThreadLocal对象threadXxx，用来保存线程间需要隔离处理的对象xxx。

2、在ThreadDemo类中，创建一个获取要隔离访问的数据的方法getXxx()，在方法中判断，若ThreadLocal对象为null时候，应该new()一个隔离访问类型的对象，并强制转换为要应用的类型。

3、在ThreadDemo类的run()方法中，通过getXxx()方法获取要操作的数据，这样可以保证每个线程对应一个数据对象，在任何时刻都操作的是这个对象。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_labelTop)

## ThreadLocal为什么会内存泄漏

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　上面代码中Entry 继承了WeakReference，说明该map的key为一个弱引用，我们知道弱引用有利于GC回收。

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181204130755546-2031692808.png)

`　　ThreadLocalMap`使用`ThreadLocal`的弱引用作为`key`，如果一个`ThreadLocal`没有外部强引用来引用它，那么系统 GC 的时候，这个`ThreadLocal`势必会被回收，这样一来，`ThreadLocalMap`中就会出现`key`为`null`的`Entry`，就没有办法访问这些`key`为`null`的`Entry`的`value`，如果当前线程再迟迟不结束的话，这些`key`为`null`的`Entry`的`value`就会一直存在一条强引用链：`Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value`永远无法回收，造成内存泄漏。其实，`ThreadLocalMap`的设计中已经考虑到这种情况，也加上了一些防护措施：在`ThreadLocal`的`get()`,`set()`,`remove()`的时候都会清除线程`ThreadLocalMap`里所有`key`为`null`的`value`。但是这些被动的预防措施并不能保证不会内存泄漏：

- 使用`static`的`ThreadLocal`，延长了`ThreadLocal`的生命周期，可能导致的内存泄漏。
- 分配使用了`ThreadLocal`又不再调用`get()`,`set()`,`remove()`方法，那么就会导致内存泄漏。

[![img](https://img-blog.csdn.net/20171005154856389?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbnNzeQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)](https://img-blog.csdn.net/20171005154856389?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbnNzeQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



### 为什么使用弱引用

- **key 使用强引用**：引用的`ThreadLocal`的对象被回收了，但是`ThreadLocalMap`还持有`ThreadLocal`的强引用，如果没有手动删除，`ThreadLocal`不会被回收，导致`Entry`内存泄漏。
- **key 使用弱引用**：引用的`ThreadLocal`的对象被回收了，由于`ThreadLocalMap`持有`ThreadLocal`的弱引用，即使没有手动删除，`ThreadLocal`也会被回收。`value`在下一次`ThreadLocalMap`调用`set`,`get`，`remove`的时候会被清除。

　　**1、可以知道使用弱引用可以多一层保障：理论上\**弱引用`ThreadLocal`不会内存泄漏，对应的`value`在下一次`ThreadLocalMap`调用`set`,`get`,`remove`的时候会被清除；但是如果分配使用了`ThreadLocal`又不再调用`get()`,`set()`,`remove()`方法，那么就有可能导致内存泄漏\**
**

　　**2、通常，我们需要\*保证作为key的\*ThreadLocal\*类型能够被全局访问到\*，同时也必须\*保证其为单例\*，因此，在一个类中将其设为static类型便成为了惯用做法，如上面例子中都是用了Static修饰。使用static修饰\**ThreadLocal对象的引用后，\*\*\*\*ThreadLocal\*\*\*\*\**的生命周期跟`Thread`一样长，因此ThreadLocalMap的Key也不会被GC回收，弱引用形同虚设，此时就极容易造成\**\*\*`ThreadLocalMap内存泄露。`\*\**\***

**关键在于threadLocal如果用Static修饰，如果是多线程操作threadlocal，当前线程结束后，ThreadLocal对象作为GCRoot还在其他线程中，这是弱引用就不能被回收，也就是当前Thread中的Map中的key还不会被回收，也就是很多线程中都有threadlocal为key的map不会被回收，那就会出现内存泄露。**



### ThreadLocal 最佳实践

综合上面的分析，我们可以理解`ThreadLocal`内存泄漏的前因后果，那么怎么避免内存泄漏呢？

- 每次使用完`ThreadLocal`，都调用它的`remove()`方法，清除数据。

在使用线程池的情况下，没有及时清理`ThreadLocal`，不仅是内存泄漏的问题，更严重的是可能导致业务逻辑出现问题。所以，使用`ThreadLocal`就跟加锁完要解锁一样，用完就清理。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10059735.html#_labelTop)

## **总结**

- ThreadLocal 不是用于解决共享变量的问题的，也不是为了协调线程同步而存在，而是为了方便每个线程处理自己的状态而引入的一个机制。这点至关重要。
- 每个Thread内部都有一个ThreadLocal.ThreadLocalMap类型的成员变量，该成员变量用来存储实际的ThreadLocal变量副本。
- ThreadLocal并不是为线程保存对象的副本，它仅仅只起到一个索引的作用。它的主要木得视为每一个线程隔离一个类的实例，这个实例的作用范围仅限于线程内部。
- 每次使用完`ThreadLocal`，都调用它的`remove()`方法，清除数据，避免造成内存泄露。

# [并发编程（五）——AbstractQueuedSynchronizer 之 ReentrantLock源码分析](https://www.cnblogs.com/java-chen-hao/p/10172910.html)



**目录**

- [AQS 结构](https://www.cnblogs.com/java-chen-hao/p/10172910.html#_label0)
- [线程抢锁](https://www.cnblogs.com/java-chen-hao/p/10172910.html#_label1)
- [解锁操作](https://www.cnblogs.com/java-chen-hao/p/10172910.html#_label2)
- [总结](https://www.cnblogs.com/java-chen-hao/p/10172910.html#_label3)

 

**正文**

本文将从 ReentrantLock 的公平锁源码出发，分析下 AbstractQueuedSynchronizer 这个类是怎么工作的，希望能给大家提供一些简单的帮助。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10172910.html#_labelTop)

## AQS 结构

先来看看 AQS 有哪些属性，搞清楚这些基本就知道 AQS 是什么套路了！

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 头结点，你直接把它当做 当前持有锁的线程 
private transient volatile Node head;
// 阻塞的尾节点，每个新的节点进来，都插入到最后，也就形成了一个隐视的链表
private transient volatile Node tail;
// 这个是最重要的，不过也是最简单的，代表当前锁的状态，0代表没有被占用，大于0代表有线程持有当前锁
// 之所以说大于0，而不是等于1，是因为锁可以重入嘛，每次重入都加上1
private volatile int state;
// 代表当前持有独占锁的线程，举个最重要的使用例子，因为锁可以重入
// reentrantLock.lock()可以嵌套调用多次，所以每次用这个来判断当前线程是否已经拥有了锁
// if (currentThread == getExclusiveOwnerThread()) {state++}
private transient Thread exclusiveOwnerThread; //继承自AbstractOwnableSynchronizer
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

AbstractQueuedSynchronizer 的等待队列示意如下所示，注意了，之后分析过程中所说的 queue，也就是阻塞队列不包含 head，因为head表示当前持有锁的线程，并没有在等待获取锁。

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181225110438900-203061568.png)

等待队列中每个线程被包装成一个 node，数据结构是链表，一起看看源码吧：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static final class Node {
    /** Marker to indicate a node is waiting in shared mode */
    // 标识节点当前在共享模式下
    static final Node SHARED = new Node();
    /** Marker to indicate a node is waiting in exclusive mode */
    // 标识节点当前在独占模式下
    static final Node EXCLUSIVE = null;

    // ======== 下面的几个int常量是给waitStatus用的 ===========
    /** waitStatus value to indicate thread has cancelled */
    // 代码此线程取消了争抢这个锁
    static final int CANCELLED =  1;
    /** waitStatus value to indicate successor's thread needs unparking */
    // 官方的描述是，其表示当前node的后继节点对应的线程需要被唤醒
    static final int SIGNAL    = -1;
    /** waitStatus value to indicate thread is waiting on condition */
    // 本文不分析condition，所以略过吧，下一篇文章会介绍这个
    static final int CONDITION = -2;
    /**
     * waitStatus value to indicate the next acquireShared should
     * unconditionally propagate
     */
    // 同样的不分析，略过吧
    static final int PROPAGATE = -3;
    // =====================================================

    // 取值为上面的1、-1、-2、-3，或者0(以后会讲到)
    // 这么理解，暂时只需要知道如果这个值 大于0 代表此线程取消了等待，
    // 也许就是说半天抢不到锁，不抢了，ReentrantLock是可以指定timeouot的。。。
    volatile int waitStatus;
    // 前驱节点的引用
    volatile Node prev;
    // 后继节点的引用
    volatile Node next;
    // 这个就是线程本尊
    volatile Thread thread;
    // 这个是在condition中用来构建单向链表，同样下一篇文章中介绍
    Node nextWaiter;

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Node 的数据结构其实也挺简单的，就是 thread + waitStatus + pre + next 四个属性而已，如果大家对LinkedList熟悉的话，那就更简单了，如果想了解LinkedList，可以看看我前面的文章[JDK1.8源码(二)——java.util.LinkedList](https://www.cnblogs.com/java-chen-hao/p/9600078.html)。

下面，我们开始说 ReentrantLock 的公平锁，首先，我们先看下 ReentrantLock 的使用方式。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 我用个web开发中的service概念吧
public class OrderService {
    // 使用static，这样每个线程拿到的是同一把锁
    private static ReentrantLock reentrantLock = new ReentrantLock(true);

    public void createOrder() {
        // 比如我们同一时间，只允许一个线程创建订单
        reentrantLock.lock();
        // 通常，lock 之后紧跟着 try 语句
        try {
            // 这块代码同一时间只能有一个线程进来(获取到锁的线程)，
            // 其他的线程在lock()方法上阻塞，等待获取到锁，再进来
            // 执行代码...
        } finally {
            // 释放锁
            // 释放锁必须要在finally里，确保锁一定会被释放，如果写在try里面，发生异常，则有可能不会执行，就会发生死锁
            reentrantLock.unlock();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ReentrantLock 在内部用了内部类 Sync 来管理锁，所以真正的获取锁和释放锁是由 Sync 的实现类来控制的。

```
abstract static class Sync extends AbstractQueuedSynchronizer {

}
```

Sync 有两个实现，分别为 NonfairSync（非公平锁）和 FairSync（公平锁）

公平锁：每个线程抢占锁的顺序为先后调用lock方法的顺序依次获取锁，类似于排队吃饭。

非公平锁：每个线程抢占锁的顺序不定，谁运气好，谁就获取到锁，和调用lock方法的先后顺序无关。

我们看 FairSync 部分。

```
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10172910.html#_labelTop)

## 线程抢锁

我们来看看lock方法的实现

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 static final class FairSync extends Sync {
 2     private static final long serialVersionUID = -3000897897090466540L;
 3       // 争锁
 4     final void lock() {
 5         acquire(1);
 6     }
 7     // 来自父类AQS，我直接贴过来这边
 8     // 如果tryAcquire(arg) 返回true,表示尝试获取锁成功，获取到锁，也就结束了。
 9     // 否则，acquireQueued方法会将线程压到队列中
10     public final void acquire(int arg) { // 此时 arg == 1
11         // 首先调用tryAcquire(1)一下，名字上就知道，这个只是试一试
12         // 因为有可能直接就成功了呢，也就不需要进队列排队了
13         if (!tryAcquire(arg) &&
14             // tryAcquire(arg)没有成功，这个时候需要把当前线程挂起，放到阻塞队列中。
15             acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) {
16               selfInterrupt();
17         }
18     }
19     //先看下tryAcquire方法:使用protected修饰，留空了，是想留给子类去实现
20     protected boolean tryAcquire(int arg) {
21         throw new UnsupportedOperationException();
22     }
23     
24     //看FairSync的tryAcquire方法：
25     // 尝试直接获取锁，返回值是boolean，代表是否获取到锁
26     // 返回true：1.没有线程在等待锁；2.重入锁，线程本来就持有锁，也就可以理所当然可以直接获取
27     protected final boolean tryAcquire(int acquires) {
28         final Thread current = Thread.currentThread();
29         int c = getState();
30         // state == 0 此时此刻没有线程持有锁
31         if (c == 0) {
32             // 虽然此时此刻锁是可以用的，但是这是公平锁，既然是公平，就得讲究先来后到，
33             // 看看有没有别人在队列中等了半天了，如果在队列中有等待的线程，则这里就不能获取到锁
34             if (!hasQueuedPredecessors() &&
35                 // 如果没有线程在等待，那就用CAS尝试一下，尝试将state的状态从0改成1，成功了就获取到锁了，
36                 // 不成功的话，只能说明一个问题，就在刚刚几乎同一时刻有个线程抢先了
37                 // 有其他线程同时进入到了这一步，并且执行CAS改变state状态成功
38                 compareAndSetState(0, acquires)) {
39                 // 到这里就是获取到锁了，标记一下，告诉大家，现在是我占用了锁
40                 setExclusiveOwnerThread(current);
41                 return true;
42             }
43         }
44           // 会进入这个else if分支，说明是重入了，需要操作：state=state+1
45         else if (current == getExclusiveOwnerThread()) {
46             int nextc = c + acquires;
47             if (nextc < 0)
48                 throw new Error("Maximum lock count exceeded");
49             setState(nextc);
50             return true;
51         }
52         // 如果到这里，说明前面的if和else if都没有返回true，说明没有获取到锁
53         return false;
54     }
55 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 我们来看看 tryAcquire 里面的几个方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    //如果队列中有等待的线程，则返回true,否则返回false
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
//我们看看第38行CAS改变state状态的方法 
//compareAndSetState(0, acquires)) acquires=1

protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    //此处调用了sun.misc.Unsafe类方法进行CAS操作
    //this表示AbstractQueuedSynchronizer对象，stateOffset表示state的偏移量，expect此时为0，update为1
    //此方法表示比较state的stateOffset处内存位置中的值和期望的值，如果相同则更新。
    //此时表示把state的值从0改为1，成功返回true;但是同时有可能其他线程也来修改了state的值，已经不为0了，则返回false
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}

private static final Unsafe unsafe = Unsafe.getUnsafe();
//AbstractQueuedSynchronizer中state属性的偏移量
private static final long stateOffset;
//AbstractQueuedSynchronizer中head属性的偏移量，后面以此类推
private static final long headOffset;
private static final long tailOffset;
private static final long waitStatusOffset;
private static final long nextOffset;

static {
    try {
        //在类加载的时候会通过sun.misc.Unsafe类方法获取AbstractQueuedSynchronizer中各个属性的偏移量，方便后面各种CAS操作
        stateOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("state"));
        headOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("head"));
        tailOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
        waitStatusOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("waitStatus"));
        nextOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("next"));

    } catch (Exception ex) { throw new Error(ex); }
}
    
/**
* 比较obj的offset处内存位置中的值和期望的值，如果相同则更新。此更新是不可中断的。
* 
* @param obj 需要更新的对象
* @param offset obj中整型field的偏移量
* @param expect 希望field中存在的值
* @param update 如果期望值expect与field的当前值相同，设置filed的值为这个新值
* @return 如果field的值被更改返回true
*/
public native boolean compareAndSwapInt(Object obj, long offset, int expect, int update);

//等待队列里没有线程等待，并且CAS改变state成功，则进入第40行代码 setExclusiveOwnerThread(current);
protected final void setExclusiveOwnerThread(Thread thread) {
    //就是对exclusiveOwnerThread赋值
    exclusiveOwnerThread = thread;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

由此我们清楚了tryAcquire(arg)方法的作用，就是改变把state的状态改为1或者加1，并将 exclusiveOwnerThread 赋值为当前线程，如果获取锁成功，则lock（）方法结束，主线程里面的业务代码继续往下执行。

如果不tryAcquire(arg)返回false，则要执行 acquireQueued(addWaiter(Node.EXCLUSIVE), arg))，将当前线程挂起，放到阻塞队列中

这个方法，首先需要执行：addWaiter(Node.EXCLUSIVE)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
  1 /**
  2  * Creates and enqueues node for current thread and given mode.
  3  *
  4  * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
  5  * @return the new node
  6  */
  7 // 此方法的作用是把线程包装成node，同时进入到队列中
  8 // 参数mode此时是Node.EXCLUSIVE，代表独占模式
  9 private Node addWaiter(Node mode) {
 10     Node node = new Node(Thread.currentThread(), mode);
 11     // Try the fast path of enq; backup to full enq on failure
 12     // 以下几行代码想把当前node加到链表的最后面去，也就是进到阻塞队列的最后
 13     Node pred = tail;
 14 
 15     // tail!=null => 队列不为空
 16     if (pred != null) { 
 17         // 设置自己的前驱 为当前的队尾节点
 18         node.prev = pred; 
 19         // 用CAS把自己设置为队尾,就是更新tail的值，pred表示tail原始值，node表示期望更新的值， 如果成功后，tail == node了
 20         if (compareAndSetTail(pred, node)) { 
 21             // 进到这里说明设置成功，当前node==tail, 将自己与之前的队尾相连
 22             // 上面已经有 node.prev = pred
 23             // 加上下面这句，也就实现了和之前的尾节点双向连接了
 24             // pred为临时变量，表示之前的队尾节点，现在将队尾节点的next指向node，则将node添加到队尾了
 25             pred.next = node;
 26             // 线程入队了，可以返回了
 27             return node;
 28         }
 29     }
 30     // 仔细看看上面的代码，如果会到这里，
 31     // 说明 pred==null(队列是空的) 或者 CAS失败(有线程在竞争入队)
 32     enq(node);
 33     return node;
 34 }
 35 
 36 /**
 37  * Inserts node into queue, initializing if necessary. See picture above.
 38  * @param node the node to insert
 39  * @return node's predecessor
 40  */
 41 // 采用自旋的方式入队
 42 // 之前说过，到这个方法只有两种可能：等待队列为空，或者有线程竞争入队，
 43 // 自旋在这边的语义是：CAS设置tail过程中，竞争一次竞争不到，我就多次竞争，总会排到的
 44 private Node enq(final Node node) {
 45     for (;;) {
 46         Node t = tail;
 47         // 之前说过，队列为空也会进来这里
 48         if (t == null) { // Must initialize
 49             // 初始化head节点
 50             // 还是一步CAS，你懂的，现在可能是很多线程同时进来呢
 51             if (compareAndSetHead(new Node()))
 52                 // 给后面用：这个时候head节点的waitStatus==0
 53                 // 这个时候有了head，但是tail还是null，设置一下，
 54                 // 注意：这里只是设置了tail=head，这里还没return
 55                 // 所以，设置完了以后，继续for循环，下次就到下面的else分支了
 56                 tail = head;
 57         } else {
 58             // 下面几行，和上一个方法 addWaiter 是一样的，
 59             // 只是这个套在无限循环里，反正就是将当前线程排到队尾，有线程竞争的话排不上重复排
 60             node.prev = t;
 61             if (compareAndSetTail(t, node)) {
 62                 t.next = node;
 63                 return t;
 64             }
 65         }
 66     }
 67 }
 68 
 69 
 70 // 现在，又回到这段代码了
 71 // if (!tryAcquire(arg) 
 72 //        && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) 
 73 //     selfInterrupt();
 74 
 75 // 下面这个方法，参数node，经过addWaiter(Node.EXCLUSIVE)，此时已经进入阻塞队列
 76 // 注意一下：如果acquireQueued(addWaiter(Node.EXCLUSIVE), arg))返回true的话，
 77 // 意味着上面这段代码将进入selfInterrupt()，所以正常情况下，下面应该返回false
 78 // 这个方法非常重要，应该说真正的线程挂起，然后被唤醒后去获取锁，都在这个方法里了
 79 final boolean acquireQueued(final Node node, int arg) {
 80     boolean failed = true;
 81     try {
 82         boolean interrupted = false;
 83         for (;;) {
 84             //获取node的prev节点(node的上一个节点)
 85             final Node p = node.predecessor();
 86             // p == head 说明当前节点虽然进到了阻塞队列，但是是阻塞队列的第一个，因为它的前驱是head
 87             // 注意，阻塞队列不包含head节点，head一般指的是占有锁的线程，head后面的才称为阻塞队列
 88             // 所以当前节点可以去试抢一下锁
 89             // 这里我们说一下，为什么可以去试试：
 90             // 首先，它是队头，这个是第一个条件，其次，当前的head有可能是刚刚初始化的node，
 91             // enq(node) 方法里面有提到，head是延时初始化的，而且new Node()的时候没有设置任何线程
 92             // 也就是说，当前的head不属于任何一个线程，所以作为队头，可以去试一试，
 93             // tryAcquire已经分析过了, 忘记了请往前看一下，就是简单用CAS试操作一下state
 94             if (p == head && tryAcquire(arg)) {
 95                 //到这里说明刚加入到等待队列里面的node只有一个，并且此时获取锁成功，设置head为node
 96                 setHead(node);
 97                 //将之前的head的next设为null方便jvm垃圾回收
 98                 p.next = null; // help GC
 99                 failed = false;
100                 //此时interrupted = false;
101                 return interrupted;
102             }
103             // 到这里，说明上面的if分支没有成功，要么当前node本来就不是队头，
104             // 要么就是tryAcquire(arg)没有抢赢别人，继续往下看
105             if (shouldParkAfterFailedAcquire(p, node) &&
106                 parkAndCheckInterrupt())
107                 interrupted = true;
108         }
109     } finally {
110         if (failed)
111             cancelAcquire(node);
112     }
113 }
114 
115 /**
116  * Checks and updates status for a node that failed to acquire.
117  * Returns true if thread should block. This is the main signal
118  * control in all acquire loops.  Requires that pred == node.prev
119  *
120  * @param pred node's predecessor holding status
121  * @param node the node
122  * @return {@code true} if thread should block
123  */
124 // 刚刚说过，会到这里就是没有抢到锁呗，这个方法说的是："当前线程没有抢到锁，是否需要挂起当前线程？"
125 // 第一个参数是前驱节点，第二个参数才是代表当前线程的节点
126 private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
127     int ws = pred.waitStatus;
128     // 前驱节点的 waitStatus == -1 ，说明前驱节点状态正常，当前线程需要挂起，直接可以返回true
129     if (ws == Node.SIGNAL)
130         /*
131          * This node has already set status asking a release
132          * to signal it, so it can safely park.
133          */
134         return true;
135 
136     // 前驱节点 waitStatus大于0 ，之前说过，大于0 说明前驱节点取消了排队。这里需要知道这点：
137     // 进入阻塞队列排队的线程会被挂起，而唤醒的操作是由前驱节点完成的。
138     // 所以下面这块代码说的是将当前节点的prev指向waitStatus<=0的节点，
139     // 简单说，如果前驱节点取消了排队，
140     // 找前驱节点的前驱节，往前循环总能找到一个waitStatus<=0的节点
141     if (ws > 0) {
142         /*
143          * Predecessor was cancelled. Skip over predecessors and
144          * indicate retry.
145          */
146         do {
147             node.prev = pred = pred.prev;
148         } while (pred.waitStatus > 0);
149         pred.next = node;
150     } else {
151         /*
152          * waitStatus must be 0 or PROPAGATE.  Indicate that we
153          * need a signal, but don't park yet.  Caller will need to
154          * retry to make sure it cannot acquire before parking.
155          */
156         // 仔细想想，如果进入到这个分支意味着什么
157         // 前驱节点的waitStatus不等于-1和1，那也就是只可能是0，-2，-3
158         // 在我们前面的源码中，都没有看到有设置waitStatus的，所以每个新的node入队时，waitStatu都是0
159         // 用CAS将前驱节点的waitStatus设置为Node.SIGNAL(也就是-1)
160         compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
161     }
162     return false;
163 }
164 
165 // private static boolean shouldParkAfterFailedAcquire(Node pred, Node node)
166 // 这个方法结束根据返回值我们简单分析下：
167 // 如果返回true, 说明前驱节点的waitStatus==-1，是正常情况，那么当前线程需要被挂起，等待以后被唤醒
168 // 我们也说过，以后是被前驱节点唤醒，就等着前驱节点拿到锁，然后释放锁的时候叫你好了
169 // 如果返回false, 说明当前不需要被挂起，为什么呢？往后看
170 
171 // 跳回到前面是这个方法
172 // if (shouldParkAfterFailedAcquire(p, node) &&
173 //                parkAndCheckInterrupt())
174 //                interrupted = true;
175 
176 // 1. 如果shouldParkAfterFailedAcquire(p, node)返回true，
177 // 那么需要执行parkAndCheckInterrupt():
178 
179 // 这个方法很简单，因为前面返回true，所以需要挂起线程，这个方法就是负责挂起线程的
180 // 这里用了LockSupport.park(this)来挂起线程，然后就停在这里了，等待被唤醒=======
181 private final boolean parkAndCheckInterrupt() {
182     LockSupport.park(this);
183     return Thread.interrupted();
184 }
185 
186 // 2. 接下来说说如果shouldParkAfterFailedAcquire(p, node)返回false的情况
187 
188 // 仔细看shouldParkAfterFailedAcquire(p, node)，我们可以发现，其实第一次进来的时候，一般都不会返回true的，原因很简单，前驱节点的waitStatus=-1是依赖于后继节点设置的。
189 //intwaitStatus默认值为0，也就是说，我都还没给前驱设置-1呢，怎么可能是true呢，但是要看到，这个方法是套在循环里的，所以第二次进来的时候状态就是-1了。
190 
191 // 为什么shouldParkAfterFailedAcquire(p, node)返回false的时候不直接挂起线程：
192 // 如果 head后面的节点 if (ws > 0)这里有多个节点的waitStatus都为1，这里多次循环之后，node的prev指向了head，此时还需要挂起吗？当然是不需要了，下一次for循环，就能获取到锁了。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10172910.html#_labelTop)

## 解锁操作

最后，就是还需要介绍下唤醒的动作了。我们知道，正常情况下，如果线程没获取到锁，线程会被 `LockSupport.park(this);` 挂起停止，等待被唤醒。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 唤醒的代码还是比较简单的，你如果上面加锁的都看懂了，下面都不需要看就知道怎么回事了
public void unlock() {
    sync.release(1);
}

public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

// 回到ReentrantLock看tryRelease方法
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    // 是否完全释放锁
    boolean free = false;
    // 其实就是重入的问题，如果c==0，也就是说没有嵌套锁了，可以释放了，否则还不能释放掉
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}

/**
 * Wakes up node's successor, if one exists.
 *
 * @param node the node
 */
// 唤醒后继节点
// 从上面调用处知道，参数node是head头结点
private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    // 如果head节点当前waitStatus<0, 将其修改为0
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    // 下面的代码就是唤醒后继节点，但是有可能后继节点取消了等待（waitStatus==1）
    // 从队尾往前找，找到waitStatus<=0的所有节点中排在最前面的一个
    Node s = node.next;
    //如果头节点后面的第一个节点状态为-1，并没有被取消，是不会进入到下面的方法中
    if (s == null || s.waitStatus > 0) {
        s = null;
        // 从后往前找，仔细看代码，不必担心中间有节点取消(waitStatus==1)的情况
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        // 唤醒线程
        LockSupport.unpark(s.thread);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

但是为什么要从后面开始遍历寻找waitStatus<=0的所有节点中排在最前面的一个，为什么不从前面的节点开始找呢？
这个问题的答案在 addWaiter(Node mode)方法中，看下面的代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 Node pred = tail;
 2     if (pred != null) {
 3         node.prev = pred;
 4         // 1. 先设置的 tail
 5         if (compareAndSetTail(pred, node)) {
 6             // 2. 设置前驱节点的后继节点为当前节点
 7             pred.next = node;
 8             return node;
 9         }
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里存在并发问题：从前往后寻找不一定能找到刚刚加入队列的后继节点。

如果此时正有一个线程加入等待队列的尾部，执行到上面第7行，第7行还未执行，解锁操作如果从前面开始找 头节点后面的第一个节点状态为-1的节点，此时是找不到这个新加入的节点的，因为尾节点的next 还未指向新加入的node，但是从后面开始遍历的话，那就不存在这种情况。

 

唤醒线程以后，被唤醒的线程将从以下代码中继续往前走：我们刚才是找到head后面第一个状态为-1的节点里面的线程进行唤醒

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this); // 刚刚线程被挂起在这里了
    return Thread.interrupted();
}
// 又回到这个方法了：acquireQueued(final Node node, int arg)，这个时候，node的前驱是head了
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

此时第一个等待节点已经被唤醒，则第一个等待节点里面的线程继续执行 acquireQueued ，此时acquireQueued 方法中 85行处p已经是head节点了,94行处就可以继续尝试获取锁了。依次循环，这个节点获取到锁，解锁后，等待队列head节点后第一个节点进行唤醒获取锁。

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10172910.html#_labelTop)

## 总结

在并发环境下，加锁和解锁需要以下三个部件的协调：

1. 锁状态。我们要知道锁是不是被别的线程占有了，这个就是 state 的作用，它为 0 的时候代表没有线程占有锁，可以去争抢这个锁，用 CAS 将 state 设为 1，如果 CAS 成功，说明抢到了锁，这样其他线程就抢不到了，如果锁重入的话，state进行+1 就可以，解锁就是减 1，直到 state 又变为 0，代表释放锁，所以 lock() 和 unlock() 必须要配对啊。然后唤醒等待队列中的第一个线程，让其来占有锁。
2. 线程的阻塞和解除阻塞。AQS 中采用了 LockSupport.park(thread) 来挂起线程，用 unpark 来唤醒线程。
3. 阻塞队列。因为争抢锁的线程可能很多，但是只能有一个线程拿到锁，其他的线程都必须等待，这个时候就需要一个 queue 来管理这些线程，AQS 用的是一个 FIFO 的队列，就是一个链表，每个 node 都持有后继节点的引用。

# [并发编程（六）——AbstractQueuedSynchronizer 之 Condition 源码分析](https://www.cnblogs.com/java-chen-hao/p/10175770.html)



**目录**

- [公平锁和非公平锁](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_label0)
- Condition
  - [1、大体实现流程](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_label1_0)
  - [2.await方法](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_label1_1)
  - [3. 将节点加入到条件队列](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_label1_2)
  - [4. 完全释放独占锁](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_label1_3)
  - [5. 等待进入阻塞队列](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_label1_4)
  - [6. signal 唤醒线程，转移到阻塞队列](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_label1_5)
  - [7. 获取独占锁](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_label1_6)
  - [8. 带超时机制的 await](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_label1_7)
  - [9. 总结](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_label1_8)

 

**正文**

我们接着上一篇文章继续，本文讲讲解ReentrantLock 公平锁和非公平锁的区别，深入分析 AbstractQueuedSynchronizer 中的 ConditionObject

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_labelTop)

## 公平锁和非公平锁

ReentrantLock 默认采用非公平锁，除非你在构造方法中传入参数 true 。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public ReentrantLock() {
    sync = new NonfairSync();
}
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

公平锁的 lock 方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static final class FairSync extends Sync {
    final void lock() {
        acquire(1);
    }
    // AbstractQueuedSynchronizer.acquire(int arg)
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // 1. 和非公平锁相比，这里多了一个判断：是否有线程在等待
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

非公平锁的 lock 方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
static final class NonfairSync extends Sync {
    final void lock() {
        // 2. 和公平锁相比，这里会直接先进行一次CAS，成功就返回了
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }
    // AbstractQueuedSynchronizer.acquire(int arg)
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
/**
 * Performs non-fair tryLock.  tryAcquire is implemented in
 * subclasses, but both need nonfair try for trylock method.
 */
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

总结：公平锁和非公平锁只有两处不同：

1. 非公平锁在调用 lock 后，首先就会调用 CAS 进行一次抢锁，如果这个时候恰巧锁没有被占用，那么直接就获取到锁返回了。
2. 非公平锁在 CAS 失败后，和公平锁一样都会进入到 tryAcquire 方法，在 tryAcquire 方法中，如果发现锁这个时候被释放了（state == 0），非公平锁会直接 CAS 抢锁，但是公平锁会判断等待队列是否有线程处于等待状态，如果有则不去抢锁，乖乖排到后面。

公平锁和非公平锁就这两点区别，如果这两次 CAS 都不成功，那么后面非公平锁和公平锁是一样的，都要进入到阻塞队列等待唤醒。

非公平锁让获取锁的时间变得更加不确定，可能会导致在阻塞队列中的线程长期处于饥饿状态。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10175770.html#_labelTop)

## Condition

 JUC提供了Lock可以方便的进行锁操作，但是有时候我们也需要对线程进行条件性的阻塞和唤醒，这时我们就需要condition条件变量，它就像是在线程上加了多个开关，可以方便的对持有锁的线程进行阻塞和唤醒。

Condition主要是为了在J.U.C框架中提供和Java传统的监视器风格的wait，notify和notifyAll方法类似的功能。

condition 是依赖于 ReentrantLock 的，不管是调用 await 进入等待还是 signal 唤醒，都必须获取到锁才能进行操作。

每个 ReentrantLock 实例可以通过调用多次 newCondition 产生多个 ConditionObject 的实例：

```
final ConditionObject newCondition() {
    return new ConditionObject();
}
```

我们首先来看下我们关注的 Condition 的实现类 `AbstractQueuedSynchronizer` 类中的 `ConditionObject`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class ConditionObject implements Condition, java.io.Serializable {
        private static final long serialVersionUID = 1173984872572414699L;
        // 条件队列的第一个节点
          // 不要管这里的关键字 transient，是不参与序列化的意思
        private transient Node firstWaiter;
        // 条件队列的最后一个节点
        private transient Node lastWaiter;
        ......
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在上一篇介绍 AQS 的时候，我们有一个阻塞队列，用于保存等待获取锁的线程的队列。这里我们引入另一个概念，叫条件队列（condition queue）



### 1、大体实现流程

AQS等待队列与Condition队列是两个相互独立的队列 
await()就是在当前线程持有锁的基础上释放锁资源，并新建Condition节点加入到Condition的队列尾部，阻塞当前线程 
signal()就是将Condition的头节点移动到AQS等待节点尾部，让其等待再次获取锁

以下是AQS队列和Condition队列的出入结点的示意图，可以通过这几张图看出线程结点在两个队列中的出入关系和条件。

**I.初始化状态**：AQS等待队列有3个Node，Condition队列有1个Node(也有可能1个都没有)

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181226151126091-726564397.png)

**II.节点1执行Condition.await()** 
1.将head后移 
2.释放节点1的锁并从AQS等待队列中移除 
3.将节点1加入到Condition的等待队列中 
4.更新lastWaiter为节点1

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181226151231555-1357367994.png)

**III.节点2执行signal()操作** 
5.将firstWaiter后移 
6.将节点4移出Condition队列 
7.将节点4加入到AQS的等待队列中去 
8.更新AQS的等待队列的tail

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181226151308134-1644885975.png)

基本上，把这几张图看懂，你也就知道 condition 的处理流程了。

　　1.我们知道一个 ReentrantLock 实例可以通过多次调用 newCondition() 来产生多个 Condition 实例，这里对应 condition1 和 condition2。注意，ConditionObject 只有两个属性 firstWaiter 和 lastWaiter；

　　2.每个 condition 有一个关联的条件队列，如线程 1 调用 condition1.await() 方法即可将当前线程 1 包装成 Node 后加入到条件队列中，然后阻塞在这里，不继续往下执行，条件队列是一个单向链表；

　　3.调用 condition1.signal() 会将condition1 对应的条件队列的 firstWaiter 移到阻塞队列的队尾，等待获取锁，获取锁后 await 方法返回，继续往下执行。

 

这里，我们简单回顾下 Node 的属性：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
volatile int waitStatus; // 可取值 0、CANCELLED(1)、SIGNAL(-1)、CONDITION(-2)、PROPAGATE(-3)
volatile Node prev;
volatile Node next;
volatile Thread thread;
Node nextWaiter;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

prev 和 next 用于实现阻塞队列的双向链表，nextWaiter 用于实现条件队列的单向链表



### 2.await方法

ReentrantLock是独占锁，一个线程拿到锁后如果不释放，那么另外一个线程肯定是拿不到锁，所以在lock.lock()和lock.unlock()之间可能有一次释放锁的操作（同样也必然还有一次获取锁的操作）。在进入lock.lock()后唯一可能释放锁的操作就是await()了。*也就是说await()操作实际上就是释放锁，然后挂起线程，一旦条件满足就被唤醒，再次获取锁！*

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter(); //构造一个新的等待队列Node加入到队尾
    int savedState = fullyRelease(node); //释放当前线程的独占锁，不管重入几次，都把state释放为0
    int interruptMode = 0;
    //如果当前节点没有在同步队列上，即还没有被signal，则将当前线程阻塞
    while (!isOnSyncQueue(node)) {
        // 线程挂起
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)  //被中断则直接退出自旋
            break;
    }
    //退出了上面自旋说明当前节点已经在同步队列上，但是当前节点不一定在同步队列队首。acquireQueued将阻塞直到当前节点成为队首，即当前线程获得了锁。然后await()方法就可以退出了，让线程继续执行await()后的代码。
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 3. 将节点加入到条件队列

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 将当前线程对应的节点入队，插入队尾
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // 如果条件队列的最后一个节点取消了，将其清除出去
    if (t != null && t.waitStatus != Node.CONDITION) {
        // 这个方法会遍历整个条件队列，然后会将已取消的所有节点清除出队列
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    // 如果队列为空
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在addWaiter 方法中，有一个 unlinkCancelledWaiters() 方法，该方法用于清除队列中已经取消等待的节点。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 等待队列是一个单向链表，遍历链表将已经取消等待的节点清除出去
// 纯属链表操作，很好理解，看不懂多看几遍就可以了
private void unlinkCancelledWaiters() {
    Node t = firstWaiter;
    Node trail = null;
    while (t != null) {
        Node next = t.nextWaiter;
        // 如果节点的状态不是 Node.CONDITION 的话，这个节点就是被取消的
        if (t.waitStatus != Node.CONDITION) {
            t.nextWaiter = null;
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;
            if (next == null)
                lastWaiter = trail;
        }
        else
            trail = t;
        t = next;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 4. 完全释放独占锁

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 首先，我们要先观察到返回值 savedState 代表 release 之前的 state 值
// 对于最简单的操作：先 lock.lock()，然后 condition1.await()。
//         那么 state 经过这个方法由 1 变为 0，锁释放，此方法返回 1
//         相应的，如果 lock 重入了 n 次，savedState == n
// 如果这个方法失败，会将节点设置为"取消"状态，并抛出异常 IllegalMonitorStateException
final int fullyRelease(Node node) {
    boolean failed = true;
    try {
        int savedState = getState();
        // 这里使用了当前的 state 作为 release 的参数，也就是完全释放掉锁，将 state 置为 0
        if (release(savedState)) {
            failed = false;
            return savedState;
        } else {
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们来看看release方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public final boolean release(int arg) {
    //先将state释放为0
    if (tryRelease(arg)) {
        //取到阻塞队列的头节点
        Node h = head;
        if (h != null && h.waitStatus != 0)
            //唤醒头节点，则第一个等待的节点会继续获取锁
            unparkSuccessor(h);
        return true;
    }
    return false;
}

private void unparkSuccessor(Node node) {
    
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    
    //从后面开始往前找，找到第一个状态为-1的节点
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        //唤醒第一个状态为-1的节点,则该节点会继续获取锁
        LockSupport.unpark(s.thread);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### 5. 等待进入阻塞队列

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
int interruptMode = 0;
while (!isOnSyncQueue(node)) {
    // 线程挂起
    LockSupport.park(this);

    if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
        break;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

isOnSyncQueue(Node node) 用于判断节点是否已经转移到阻塞队列了：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
final boolean isOnSyncQueue(Node node) {
    //如果当前节点状态是CONDITION或node.prev是null，则证明当前节点在等待队列上而不是同步队列上。之所以可以用node.prev来判断，是因为一个节点如果要加入同步队列，在加入前就会设置好prev字段。
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;
    //如果node.next不为null，则一定在同步队列上，因为node.next是在节点加入同步队列后设置的
    if (node.next != null) // If has successor, it must be on queue
        return true;
    return findNodeFromTail(node); //前面的两个判断没有返回的话，就从同步队列队尾遍历一个一个看是不是当前节点。
}

// 从同步队列的队尾往前遍历，如果找到，返回 true
private boolean findNodeFromTail(Node node) {
    Node t = tail;
    for (;;) {
        if (t == node)
            return true;
        if (t == null)
            return false;
        t = t.prev;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

回到前面的循环，isOnSyncQueue(node) 返回 false 的话，那么进到 `LockSupport.park(this);` 这里线程挂起。



### 6. signal 唤醒线程，转移到阻塞队列

为了大家理解，这里我们先看唤醒操作，因为刚刚到 LockSupport.park(this); 把线程挂起了，等待唤醒。

唤醒操作通常由另一个线程来操作，就像生产者-消费者模式中，如果线程因为等待消费而挂起，那么当生产者生产了一个东西后，会调用 signal 唤醒正在等待的线程来消费。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 唤醒等待了最久的线程
// 其实就是，将这个线程对应的 node 从条件队列转移到阻塞队列
public final void signal() {
    // 调用 signal 方法的线程必须持有当前的独占锁
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}

// 从条件队列队头往后遍历，找出第一个需要转移的 node
// 因为前面我们说过，有些线程会取消排队，但是还在队列中
private void doSignal(Node first) {
    do {
          // 将 firstWaiter 指向 first 节点后面的第一个
        // 如果将队头移除后，后面没有节点在等待了，那么需要将 lastWaiter 置为 null
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        // 因为 first 马上要被移到阻塞队列了，和条件队列的链接关系在这里断掉
        first.nextWaiter = null;
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
      // 这里 while 循环，如果 first 转移不成功，那么选择 first 后面的第一个节点进行转移，依此类推
}

// 将节点从条件队列转移到阻塞队列
// true 代表成功转移
// false 代表在 signal 之前，节点已经取消了
final boolean transferForSignal(Node node) {

    // CAS 如果失败，说明此 node 的 waitStatus 已不是 Node.CONDITION，说明节点已经取消，
    // 既然已经取消，也就不需要转移了，方法返回，转移后面一个节点
    // 否则，将 waitStatus 置为 0
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    // enq(node): 自旋进入阻塞队列的队尾
    // 注意，这里的返回值 p 是 node 在阻塞队列的前驱节点
    Node p = enq(node);
    int ws = p.waitStatus;
    // ws > 0 说明 node 在阻塞队列中的前驱节点取消了等待锁，直接唤醒 node 对应的线程。唤醒之后会怎么样，后面再解释
    // 如果 ws <= 0, 那么 compareAndSetWaitStatus 将会被调用，上篇介绍的时候说过，节点入队后，需要把前驱节点的状态设为 Node.SIGNAL(-1)
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        // 如果前驱节点取消或者 CAS 失败，会进到这里唤醒线程，之后的操作看下一节
        LockSupport.unpark(node.thread);
    return true;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

正常情况下，`ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL)` 这句中，ws <= 0，而且 compareAndSetWaitStatus(p, ws, Node.SIGNAL) 会返回 true，所以一般也不会进去 if 语句块中唤醒 node 对应的线程。然后这个方法返回 true，也就意味着 signal 方法结束了，节点进入了阻塞队列,此时await()还是挂起状态,并没有被唤醒。 我们可以看到，signal方法只是将Node修改了状态，并没有唤醒线程。要将修改状态后的Node唤醒，唤起线程是在unlock()中。这个方法会对阻塞队列里面的线程从头到尾对状态为-1的节点做唤醒操作，具体可以看我上一篇文章,[并发编程（五）——AbstractQueuedSynchronizer 之 ReentrantLock源码分析](https://www.cnblogs.com/java-chen-hao/p/10172910.html)

unlock（）将此线程唤醒后，await()中可以继续执行，此线程被唤醒的时候它的前驱节点肯定是首节点了，因为unlock（）方法是从头到尾进行唤醒

假设发生了阻塞队列中的前驱节点取消等待，或者 CAS 失败，只要唤醒线程，让其进到下一步即可。



### 7. 获取独占锁

线程被唤醒后，此时节点已经在缓存队列的第一个等待节点了，while (!isOnSyncQueue(node)) 将会退出循环。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
 2     interruptMode = REINTERRUPT;
 3 
 4 final boolean acquireQueued(final Node node, int arg) {
 5     boolean failed = true;
 6     try {
 7         boolean interrupted = false;
 8         for (;;) {
 9             final Node p = node.predecessor();
10             if (p == head && tryAcquire(arg)) {
11                 setHead(node);
12                 p.next = null; // help GC
13                 failed = false;
14                 return interrupted;
15             }
16             if (shouldParkAfterFailedAcquire(p, node) &&
17                 parkAndCheckInterrupt())
18                 interrupted = true;
19         }
20     } finally {
21         if (failed)
22             cancelAcquire(node);
23     }
24 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

重新获取锁就在acquireQueued方法中，上一篇文章中已经详细分析了此方法，上面已经说过，unlock（）解锁时，此线程已经在阻塞队里的第一个节点，所以第10行代码处就能尝试获取锁，并将state设置为之前的状态。此时就可以接着await()后面的业务代码继续执行了。

我们想想，上面 if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL)) 处，ws <= 0，而且 compareAndSetWaitStatus(p, ws, Node.SIGNAL) 会返回 true时，不执行LockSupport.unpark(node.thread); 呢？

其实笔者认为这里不加这个判断条件应该也是可以的。只是对于CAS修改前驱节点状态为SIGNAL成功这种情况来说，如果不加这个判断条件，提前唤醒了线程，等进入acquireQueued方法了节点发现自己的前驱不是首节点，还要再阻塞，等到其前驱节点成为首节点并释放锁时再唤醒一次；而如果加了这个条件，线程被唤醒的时候它的前驱节点肯定是首节点了，线程就有机会直接获取同步状态从而避免二次阻塞，节省了硬件资源。



### 8. 带超时机制的 await

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public final boolean await(long time, TimeUnit unit)
 2         throws InterruptedException {
 3     //将时间转换成纳秒，需要等待的纳秒数
 4     long nanosTimeout = unit.toNanos(time);
 5     if (Thread.interrupted())
 6         throw new InterruptedException();
 7     //构造一个新的等待队列Node加入到队尾
 8     Node node = addConditionWaiter();
 9     //释放当前线程的独占锁，不管重入几次，都把state释放为0
10     int savedState = fullyRelease(node);
11     // 过期时间(纳秒)=当前时间(纳秒) + 等待时长(纳秒)
12     final long deadline = System.nanoTime() + nanosTimeout;
13     // 用于返回 await 是否超时
14     boolean timedout = false;
15     int interruptMode = 0;
16     //判断当前线程是否在阻塞队列(是否从条件等待队列移动到了阻塞队列)
17     while (!isOnSyncQueue(node)) {
18         // 时间到啦，一直自旋，直到nanosTimeout减少到0
19         if (nanosTimeout <= 0L) {
20             // 这里因为要 break 取消等待了。取消等待的话一定要调用 transferAfterCancelledWait(node) 这个方法
21             // 如果这个方法返回 true，在这个方法内，将节点转移到阻塞队列成功
22             // 返回 false 的话，说明 signal 已经发生，signal 方法将节点转移了。也就是说没有超时嘛
23             timedout = transferAfterCancelledWait(node);
24             break;
25         }
26         //static final long spinForTimeoutThreshold = 1000L;
27         // spinForTimeoutThreshold 的值是 1000 纳秒，也就是 1 毫秒
28         // 也就是说，如果不到 1 毫秒了，那就不要选择 parkNanos 了，自旋的性能反而更好
29         if (nanosTimeout >= spinForTimeoutThreshold)
30             //线程将一直阻塞，阻塞nanosTimeout后自动唤醒。
31             LockSupport.parkNanos(this, nanosTimeout);
32         if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
33             break;
34         // 得到剩余时间，这里计算时间是不到1毫秒时用的，因为非常短的超时等待parkNanos无法做到十分精确，所以小于1毫秒就一直自旋，直到nanosTimeout小于或者等于0就结束循环。
35         nanosTimeout = deadline - System.nanoTime();
36     }
37     if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
38         interruptMode = REINTERRUPT;
39     if (node.nextWaiter != null)
40         unlinkCancelledWaiters();
41     if (interruptMode != 0)
42         reportInterruptAfterWait(interruptMode);
43     return !timedout;
44 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

超时的思路还是很简单的，不带超时参数的 await 是 park，然后等待别人唤醒。而现在就是调用 parkNanos 方法来休眠指定的时间，醒来后判断是否 signal 调用了，调用了就是没有超时，否则就是超时了。超时的话，自己来进行转移到阻塞队列，然后抢锁。



### 9. 总结

总的来说，Condition的本质就是等待队列和同步队列的交互：

当一个持有锁的线程调用Condition.await()时，它会执行以下步骤：

1. 构造一个新的等待队列节点加入到等待队列队尾
2. 释放锁，也就是将它的同步队列节点从同步队列队首移除
3. 自旋，直到它在等待队列上的节点移动到了同步队列（通过其他线程调用signal()）或被中断
4. 阻塞当前节点，直到它获取到了锁，也就是它在同步队列上的节点排队排到了队首。

当一个持有锁的线程调用Condition.signal()时，它会执行以下操作：

从等待队列的队首开始，尝试对队首节点执行唤醒操作；如果节点CANCELLED，就尝试唤醒下一个节点；如果再CANCELLED则继续迭代。

对每个节点执行唤醒操作时，首先将节点加入同步队列，此时await()操作的步骤3的解锁条件就已经开启了。然后分两种情况讨论：

1. 如果先驱节点的状态为CANCELLED(>0) 或设置先驱节点的状态为SIGNAL失败，那么就立即唤醒当前节点对应的线程，此时await()方法就会完成步骤3，进入步骤4.
2. 如果成功把先驱节点的状态设置为了SIGNAL，那么就不立即唤醒了。等到先驱节点成为同步队列首节点并释放了同步状态后，会自动唤醒当前节点对应线程的，这时候await()的步骤3才执行完成，而且有很大概率快速完成步骤4.

# [并发编程（七）——AbstractQueuedSynchronizer 之 CountDownLatch、CyclicBarrier、Semaphore 源码分析](https://www.cnblogs.com/java-chen-hao/p/10191106.html)



**目录**

- CountDownLatch
  - [使用例子](https://www.cnblogs.com/java-chen-hao/p/10191106.html#_label0_0)
  - [源码分析](https://www.cnblogs.com/java-chen-hao/p/10191106.html#_label0_1)
  - [总结](https://www.cnblogs.com/java-chen-hao/p/10191106.html#_label0_2)
- [CyclicBarrier](https://www.cnblogs.com/java-chen-hao/p/10191106.html#_label1)
- [Semaphore](https://www.cnblogs.com/java-chen-hao/p/10191106.html#_label2)

 

**正文**

这篇，我们的关注点是 AQS 最后的部分，共享模式的使用。本文先用 CountDownLatch 将共享模式说清楚，然后顺着把其他 AQS 相关的类 CyclicBarrier、Semaphore 的源码一起过一下。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10191106.html#_labelTop)

## CountDownLatch

CountDownLatch 这个类是比较典型的 AQS 的共享模式的使用，这是一个高频使用的类。使用方法在前面一篇文章中有介绍 [并发编程（二）—— CountDownLatch、CyclicBarrier和Semaphore](https://www.cnblogs.com/java-chen-hao/p/9970581.html)



### 使用例子

我们看下 Doug Lea 在 java doc 中给出的例子，这个例子非常实用，我们经常会写这个代码。

假设我们有 N ( N > 0 ) 个任务，那么我们会用 N 来初始化一个 CountDownLatch，然后将这个 latch 的引用传递到各个线程中，在每个线程完成了任务后，调用 latch.countDown() 代表完成了一个任务。

调用 latch.await() 的方法的线程会阻塞，直到所有的任务完成。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class Driver2 { // ...
    void main() throws InterruptedException {
        CountDownLatch doneSignal = new CountDownLatch(N);
        Executor e = Executors.newFixedThreadPool(8);

        // 创建 N 个任务，提交给线程池来执行
        for (int i = 0; i < N; ++i) // create and start threads
            e.execute(new WorkerRunnable(doneSignal, i));

        // 等待所有的任务完成，这个方法才会返回
        doneSignal.await();           // wait for all to finish
    }
}

class WorkerRunnable implements Runnable {
    private final CountDownLatch doneSignal;
    private final int i;

    WorkerRunnable(CountDownLatch doneSignal, int i) {
        this.doneSignal = doneSignal;
        this.i = i;
    }

    public void run() {
        try {
            doWork(i);
            // 这个线程的任务完成了，调用 countDown 方法
            doneSignal.countDown();
        } catch (InterruptedException ex) {
        } // return;
    }

    void doWork() { ...}
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

所以说 CountDownLatch 非常实用，我们常常会将一个比较大的任务进行拆分，然后开启多个线程来执行，等所有线程都执行完了以后，再往下执行其他操作。这里例子中，只有 main 线程调用了 await 方法。

我们再来看另一个例子，这个例子很典型，用了两个 CountDownLatch：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class Driver { // ...
    void main() throws InterruptedException {
        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(N);

        for (int i = 0; i < N; ++i) // create and start threads
            new Thread(new Worker(startSignal, doneSignal)).start();

        // 这边插入一些代码，确保上面的每个线程先启动起来，才执行下面的代码。
        doSomethingElse();            // don't let run yet
        // 因为这里 N == 1，所以，只要调用一次，那么所有的 await 方法都可以通过
        startSignal.countDown();      // let all threads proceed
        doSomethingElse();
        // 等待所有任务结束
        doneSignal.await();           // wait for all to finish
    }
}

class Worker implements Runnable {
    private final CountDownLatch startSignal;
    private final CountDownLatch doneSignal;

    Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
        this.startSignal = startSignal;
        this.doneSignal = doneSignal;
    }

    public void run() {
        try {
            // 为了让所有线程同时开始任务，我们让所有线程先阻塞在这里
            // 等大家都准备好了，再打开这个门栓
            startSignal.await();
            doWork();
            doneSignal.countDown();
        } catch (InterruptedException ex) {
        } // return;
    }

    void doWork() { ...}
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个例子中，doneSignal 同第一个例子的使用，我们说说这里的 startSignal。N 个新开启的线程都调用了startSignal.await() 进行阻塞等待，它们阻塞在栅栏上，只有当条件满足的时候（startSignal.countDown()），它们才能同时通过这个栅栏。

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181228145249869-1687803573.png)

如果始终只有一个线程调用 await 方法等待任务完成，那么 CountDownLatch 就会简单很多，所以之后的源码分析读者一定要在脑海中构建出这么一个场景：有 m 个线程是做任务的，有 n 个线程在某个栅栏上等待这 m 个线程做完任务，直到所有 m 个任务完成后，n 个线程同时通过栅栏。



### 源码分析

构造方法，需要传入一个不小于 0 的整数：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
// 老套路了，内部封装一个 Sync 类继承自 AQS
private static final class Sync extends AbstractQueuedSynchronizer {
    Sync(int count) {
        // 这样就 state == count 了
        setState(count);
    }
    ...
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

先分析套路：AQS 里面的 state 是一个整数值，这边用一个 int count 参数其实初始化就是设置了这个值，所有调用了 await 方法的等待线程会挂起，然后有其他一些线程调用会做 state = state - 1 操作，当 state 减到 0 的同时，那个线程会负责唤醒调用了 await 方法的所有线程。

对于 CountDownLatch，我们仅仅需要关心两个方法，一个是 countDown() 方法，另一个是 await() 方法。countDown() 方法每次调用都会将 state 减 1，直到 state 的值为 0；而 await 是一个阻塞方法，当 state 减为 0 的时候，await 方法才会返回。await 可以被多个线程调用，读者这个时候脑子里要有个图：所有调用了 await 方法的线程阻塞在 AQS 的阻塞队列中，等待条件满足（state == 0），将线程从队列中一个个唤醒过来。

我们用以下程序来分析源码，t1 和 t2 负责调用 countDown() 方法，t3 和 t4 调用 await 方法阻塞：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class CountDownLatchDemo {
 2 
 3     public static void main(String[] args) {
 4 
 5         CountDownLatch latch = new CountDownLatch(2);
 6 
 7         Thread t1 = new Thread(new Runnable() {
 8             @Override
 9             public void run() {
10                 try {
11                     Thread.sleep(5000);
12                 } catch (InterruptedException ignore) {
13                 }
14                 // 休息 5 秒后(模拟线程工作了 5 秒)，调用 countDown()
15                 latch.countDown();
16             }
17         }, "t1");
18 
19         Thread t2 = new Thread(new Runnable() {
20             @Override
21             public void run() {
22                 try {
23                     Thread.sleep(10000);
24                 } catch (InterruptedException ignore) {
25                 }
26                 // 休息 10 秒后(模拟线程工作了 10 秒)，调用 countDown()
27                 latch.countDown();
28             }
29         }, "t2");
30 
31         t1.start();
32         t2.start();
33 
34         Thread t3 = new Thread(new Runnable() {
35             @Override
36             public void run() {
37                 try {
38                     // 阻塞，等待 state 减为 0
39                     latch.await();
40                     System.out.println("线程 t3 从 await 中返回了");
41                 } catch (InterruptedException e) {
42                     System.out.println("线程 t3 await 被中断");
43                     Thread.currentThread().interrupt();
44                 }
45             }
46         }, "t3");
47         Thread t4 = new Thread(new Runnable() {
48             @Override
49             public void run() {
50                 try {
51                     // 阻塞，等待 state 减为 0
52                     latch.await();
53                     System.out.println("线程 t4 从 await 中返回了");
54                 } catch (InterruptedException e) {
55                     System.out.println("线程 t4 await 被中断");
56                     Thread.currentThread().interrupt();
57                 }
58             }
59         }, "t4");
60 
61         t3.start();
62         t4.start();
63     }
64 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 上述程序，大概在过了 10 秒左右的时候，会输出：

```
线程 t3 从 await 中返回了
线程 t4 从 await 中返回了
// 这两条输出，顺序不是绝对的
// 后面的分析，我们假设 t3 先进入阻塞队列
```

 

接下来，我们按照流程一步一步走：先 await 等待，然后被唤醒，await 方法返回。

首先，我们来看 await() 方法，它代表线程阻塞，等待 state 的值减为 0。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    // 这也是老套路了，我在第二篇的中断那一节说过了
    if (Thread.interrupted())
        throw new InterruptedException();
    // t3 和 t4 调用 await 的时候，state 都大于 0。
    // 也就是说，这个 if 返回 true，然后往里看
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
// 只有当 state == 0 的时候，这个方法才会返回 1
protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从方法名我们就可以看出，这个方法是获取共享锁，并且此方法是可中断的（中断的时候抛出 InterruptedException 退出这个方法）。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void doAcquireSharedInterruptibly(int arg)
 2     throws InterruptedException {
 3     // 1. 入队
 4     final Node node = addWaiter(Node.SHARED);
 5     boolean failed = true;
 6     try {
 7         for (;;) {
 8             final Node p = node.predecessor();
 9             if (p == head) {
10                 // 同上，只要 state 不等于 0，那么这个方法返回 -1
11                 int r = tryAcquireShared(arg);
12                 // r=-1时，这里if不会进入
13                 if (r >= 0) {
14                     setHeadAndPropagate(node, r);
15                     p.next = null; // help GC
16                     failed = false;
17                     return;
18                 }
19             }
20             // 2. 这和第一篇AQS里面代码一样，修改前驱节点的waitStatus 为-1,同时挂起当前线程
21             if (shouldParkAfterFailedAcquire(p, node) &&
22                 parkAndCheckInterrupt())
23                 throw new InterruptedException();
24         }
25     } finally {
26         if (failed)
27             cancelAcquire(node);
28     }
29 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们来仔细分析这个方法，线程 t3 经过第 1 步 第4行 addWaiter 入队以后，我们应该可以得到这个：

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181228150947833-2057561998.png)

由于 tryAcquireShared 这个方法会返回 -1，所以 if (r >= 0) 这个分支不会进去。到 shouldParkAfterFailedAcquire 的时候，t3 将 head 的 waitStatus 值设置为 -1，如下：

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181228151017379-500706798.png)

然后进入到 parkAndCheckInterrupt 的时候，t3 挂起。

我们再分析 t4 入队，t4 会将前驱节点 t3 所在节点的 waitStatus 设置为 -1，t4 入队后，应该是这样的：

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181228151058268-2133757989.png)

然后，t4 也挂起。接下来，t3 和 t4 就等待唤醒了。

接下来，我们来看唤醒的流程，我们假设用 10 初始化 CountDownLatch。

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181228151135062-211890929.png)

当然，我们的例子中，其实没有 10 个线程，只有 2 个线程 t1 和 t2，只是为了让图好看些罢了。

我们再一步步看具体的流程。首先，我们看 countDown() 方法:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void countDown() {
 2     sync.releaseShared(1);
 3 }
 4 public final boolean releaseShared(int arg) {
 5     // 只有当 state 减为 0 的时候，tryReleaseShared 才返回 true
 6     // 否则只是简单的 state = state - 1 那么 countDown 方法就结束了
 7     if (tryReleaseShared(arg)) {
 8         // 唤醒 await 的线程
 9         doReleaseShared();
10         return true;
11     }
12     return false;
13 }
14 // 这个方法很简单，用自旋的方法实现 state 减 1
15 protected boolean tryReleaseShared(int releases) {
16     for (;;) {
17         int c = getState();
18         if (c == 0)
19             return false;
20         int nextc = c-1;
21         //通过CAS将state的值减1，失败就不会进入return,继续for循环，直至CAS成功
22         if (compareAndSetState(c, nextc))
23             //state减到0就返回true,否则返回false
24             return nextc == 0;
25     }
26 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

countDown 方法就是每次调用都将 state 值减 1，如果 state 减到 0 了，那么就调用下面的方法进行唤醒阻塞队列中的线程：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 调用这个方法的时候，state == 0
 2 private void doReleaseShared() {
 3     for (;;) {
 4         Node h = head;
 5         if (h != null && h != tail) {
 6             int ws = h.waitStatus;
 7             // t3 入队的时候，已经将头节点的 waitStatus 设置为 Node.SIGNAL（-1） 了
 8             if (ws == Node.SIGNAL) {
 9                 if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
10                     continue;            // loop to recheck cases
11                 // 就是这里，唤醒 head 的后继节点，也就是阻塞队列中的第一个节点
12                 // 在这里，也就是唤醒 t3 , t3的await()方法可以接着运行了
13                 unparkSuccessor(h);
14             }
15             else if (ws == 0 &&
16                      !compareAndSetWaitStatus(h, 0, Node.PROPAGATE)) // todo
17                 continue;                // loop on failed CAS
18         }
19         //此时 h == head 说明被唤醒的 t3线程 还没有执行到await()方法中的setHeadAndPropagate(node, r)这一步,则此时循环结束;
20         //如果执行完setHeadAndPropagate(node, r)，则head就为t3了，这里的h和head就不相等，会继续循环
21         if (h == head)                   // loop if head changed
22             break;
23     }
24 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

一旦 t3 被唤醒后，我们继续回到 await 的这段代码，在第24行代码 parkAndCheckInterrupt 返回继续接着运行，我们先不考虑中断的情况：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void doAcquireSharedInterruptibly(int arg)
 2     throws InterruptedException {
 3     final Node node = addWaiter(Node.SHARED);
 4     boolean failed = true;
 5     try {
 6         for (;;) {
 7             //p表示当前节点的前驱节点
 8             final Node p = node.predecessor();
 9             //此时被唤醒的是之前head的后继节点，所以此线程的前驱节点是head
10             if (p == head) {
11                 //此时state已经为0，r为1
12                 int r = tryAcquireShared(arg);
13                 if (r >= 0) {
14                     // 2. 这里将唤醒t3的后续节点t4,以此类推，t4被唤醒后，会在t4的await中唤醒t4的后续节点
15                     setHeadAndPropagate(node, r); 
16                     // 将已经唤醒的t3节点从队列中去除
17                     p.next = null; // help GC
18                     failed = false;
19                     return;
20                 }
21             }
22             if (shouldParkAfterFailedAcquire(p, node) &&
23                 // 1. 唤醒后这个方法返回
24                 parkAndCheckInterrupt())
25                 throw new InterruptedException();
26         }
27     } finally {
28         if (failed)
29             cancelAcquire(node);
30     }
31 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

接下来，t3 会循环一次进到 setHeadAndPropagate(node, r) 这个方法，先把 head 给占了，然后唤醒队列中其他的线程：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void setHeadAndPropagate(Node node, int propagate) {
 2     Node h = head; // Record old head for check below
 3     setHead(node);
 4 
 5     // 下面说的是，唤醒当前 node 之后的节点，即 t3 已经醒了，马上唤醒 t4
 6     // 类似的，如果 t4 后面还有 t5，那么 t4 醒了以后，马上将 t5 给唤醒了
 7     if (propagate > 0 || h == null || h.waitStatus < 0 ||
 8         (h = head) == null || h.waitStatus < 0) {
 9         Node s = node.next;
10         if (s == null || s.isShared())
11             // 又是这个方法，只是现在的 head 已经不是原来的空节点了，是 t3 的节点了
12             doReleaseShared();
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

又回到这个方法了，那么接下来，我们好好分析 doReleaseShared 这个方法，我们根据流程，头节点 head 此时是 t3 节点了：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 调用这个方法的时候，state == 0
 2 private void doReleaseShared() {
 3     for (;;) {
 4         Node h = head;
 5         if (h != null && h != tail) {
 6             int ws = h.waitStatus;
 7             // t4 将头节点(此时是 t3)的 waitStatus 设置为 Node.SIGNAL（-1） 了
 8             if (ws == Node.SIGNAL) {
 9                 if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
10                     continue;            // loop to recheck cases
11                 // 就是这里，唤醒 head 的后继节点，也就是阻塞队列中的第一个节点
12                 // 在这里，也就是唤醒 t4
13                 unparkSuccessor(h);
14             }
15             else if (ws == 0 &&
16                      // 这个 CAS 失败的场景是：执行到这里的时候，刚好有一个节点入队，入队会将这个 ws 设置为 -1
17                      !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
18                 continue;                // loop on failed CAS
19         }
20         // 如果到这里的时候，前面唤醒的线程已经占领了 head，那么再循环
21         // 否则，就是 head 没变，那么退出循环，
22         // 退出循环是不是意味着阻塞队列中的其他节点就不唤醒了？当然不是，唤醒的线程之后还是会在await()方法中调用此方法接着唤醒后续节点
23         if (h == head)                   // loop if head changed
24             break;
25     }
26 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 



### 总结

总的来说，CountDownLatch 就是线程入队阻塞，依次唤醒的过程

使用过程会执行以下操作：

　　1.当创建一个CountDownLatch 的实例后，AQS中的state会设置一个正整数

　　2.一个线程调用await(),当前线程加入到阻塞队列中，当前线程挂起

　　3.一个线程调用countDown()唤醒方法，state减1,直到state被减为0时，唤醒阻塞队列中第一个等待节点中的线程

　　4.第一个线程被唤醒后，当前线程继续执行await()方法，将当前线程设置为head,并在此方法中唤醒head的下一个节点，依次类推

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10191106.html#_labelTop)

## CyclicBarrier

字面意思是“可重复使用的栅栏”，CyclicBarrier 相比 CountDownLatch 来说，要简单很多，其源码没有什么高深的地方，它是 ReentrantLock 和 Condition 的组合使用。看如下示意图，CyclicBarrier 和 CountDownLatch 是不是很像，只是 CyclicBarrier 可以有不止一个栅栏，因为它的栅栏（Barrier）可以重复使用（Cyclic）。

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181229151538696-527942080.png)

首先，CyclicBarrier 的源码实现和 CountDownLatch 大相径庭，CountDownLatch 基于 AQS 的共享模式的使用，而 CyclicBarrier 基于 Condition 来实现。

因为 CyclicBarrier 的源码相对来说简单许多，读者只要熟悉了前面关于 Condition 的分析，那么这里的源码是毫无压力的，就是几个特殊概念罢了。

废话结束，先上基本属性和构造方法，往下拉一点点，和图一起看：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class CyclicBarrier {
 2     // 我们说了，CyclicBarrier 是可以重复使用的，我们把每次从开始使用到穿过栅栏当做"一代"
 3     private static class Generation {
 4         boolean broken = false;
 5     }
 6 
 7     /** The lock for guarding barrier entry */
 8     private final ReentrantLock lock = new ReentrantLock();
 9     // CyclicBarrier 是基于 Condition 的
10     // Condition 是“条件”的意思，CyclicBarrier 的等待线程通过 barrier 的“条件”是大家都到了栅栏上
11     private final Condition trip = lock.newCondition();
12 
13     // 参与的线程数
14     private final int parties;
15 
16     // 如果设置了这个，代表越过栅栏之前，要执行相应的操作
17     private final Runnable barrierCommand;
18 
19     // 当前所处的“代”
20     private Generation generation = new Generation();
21 
22     // 还没有到栅栏的线程数，这个值初始为 parties，然后递减
23     // 还没有到栅栏的线程数 = parties - 已经到栅栏的数量
24     private int count;
25 
26     public CyclicBarrier(int parties, Runnable barrierAction) {
27         if (parties <= 0) throw new IllegalArgumentException();
28         this.parties = parties;
29         this.count = parties;
30         this.barrierCommand = barrierAction;
31     }
32 
33     public CyclicBarrier(int parties) {
34         this(parties, null);
35     }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我用一图来描绘下 CyclicBarrier 里面的一些概念：

![img](https://img2018.cnblogs.com/blog/1168971/201812/1168971-20181229151614812-180387985.png)

现在开始分析最重要的等待通过栅栏方法 await 方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 不带超时机制
 2 public int await() throws InterruptedException, BrokenBarrierException {
 3     try {
 4         return dowait(false, 0L);
 5     } catch (TimeoutException toe) {
 6         throw new Error(toe); // cannot happen
 7     }
 8 }
 9 // 带超时机制，如果超时抛出 TimeoutException 异常
10 public int await(long timeout, TimeUnit unit)
11     throws InterruptedException,
12            BrokenBarrierException,
13            TimeoutException {
14     return dowait(true, unit.toNanos(timeout));
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

继续往里看：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private int dowait(boolean timed, long nanos)
 2         throws InterruptedException, BrokenBarrierException,
 3                TimeoutException {
 4     final ReentrantLock lock = this.lock;
 5     // 先要获取到锁，然后在 finally 中要记得释放锁
 6     // 如果记得 Condition 部分的话，我们知道 condition 的 await 会释放锁，signal 的时候需要重新获取锁
 7     lock.lock();
 8     try {
 9         final Generation g = generation;
10         // 检查栅栏是否被打破，如果被打破，抛出 BrokenBarrierException 异常
11         if (g.broken)
12             throw new BrokenBarrierException();
13         // 检查中断状态，如果中断了，抛出 InterruptedException 异常
14         if (Thread.interrupted()) {
15             breakBarrier();
16             throw new InterruptedException();
17         }
18         // index 是这个 await 方法的返回值
19         // 注意到这里，这个是从 count 递减后得到的值
20         int index = --count;
21 
22         //最后一个线程到达后, 唤醒所有等待的线程，开启新的一代（设置新的generation）
23         // 如果等于 0，说明所有的线程都到栅栏上了，准备通过
24         if (index == 0) {  // tripped
25             boolean ranAction = false;
26             try {
27                 // 如果在初始化的时候，指定了通过栅栏前需要执行的操作，在这里会得到执行
28                 final Runnable command = barrierCommand;
29                 if (command != null)
30                     command.run();
31                 // 如果 ranAction 为 true，说明执行 command.run() 的时候，没有发生异常退出的情况
32                 ranAction = true;
33                 // 唤醒等待的线程，然后开启新的一代
34                 nextGeneration();
35                 return 0;
36             } finally {
37                 if (!ranAction)
38                     // 进到这里，说明执行指定操作的时候，发生了异常，那么需要打破栅栏
39                     // 之前我们说了，打破栅栏意味着唤醒所有等待的线程，设置 broken 为 true，重置 count 为 parties
40                     breakBarrier();
41             }
42         }
43 
44         // loop until tripped, broken, interrupted, or timed out
45         // 如果是最后一个线程调用 await，那么上面就返回了
46         // 下面的操作是给那些不是最后一个到达栅栏的线程执行的
47         for (;;) {
48             try {
49                 // 如果带有超时机制，调用带超时的 Condition 的 await 方法等待，直到最后一个线程调用 await
50                 if (!timed)
51                     //此线程会添加到Condition条件队列中，并在此阻塞
52                     trip.await();
53                 else if (nanos > 0L)
54                     nanos = trip.awaitNanos(nanos);
55             } catch (InterruptedException ie) {
56                 // 如果到这里，说明等待的线程在 await（是 Condition 的 await）的时候被中断
57                 if (g == generation && ! g.broken) {
58                     // 打破栅栏
59                     breakBarrier();
60                     // 打破栅栏后，重新抛出这个 InterruptedException 异常给外层调用的方法
61                     throw ie;
62                 } else {
63                     Thread.currentThread().interrupt();
64                 }
65             }
66 
67               // 唤醒后，检查栅栏是否是“破的”
68             if (g.broken)
69                 throw new BrokenBarrierException();
70                 
71             // 上面最后一个线程执行nextGeneration()后，generation被重写设置
72             // 我们要清楚，最后一个线程在执行完指定任务(如果有的话)，会调用 nextGeneration 来开启一个新的代
73             // 然后释放掉锁，其他线程从 Condition 的 await 方法中得到锁并返回，然后到这里的时候，其实就会满足 g != generation 的，因为最后一个到达的线程已经重写设置了generation
74             if (g != generation)
75                 return index;
76 
77             // 如果醒来发现超时了，打破栅栏，抛出异常
78             if (timed && nanos <= 0L) {
79                 breakBarrier();
80                 throw new TimeoutException();
81             }
82         }
83     } finally {
84         lock.unlock();
85     }
86 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

我们看看怎么开启新的一代：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 // 开启新的一代，当最后一个线程到达栅栏上的时候，调用这个方法来唤醒其他线程，同时初始化“下一代”
2 private void nextGeneration() {
3     // 首先，需要唤醒所有的在栅栏上等待的线程
4     trip.signalAll();
5     // 更新 count 的值
6     count = parties;
7     // 重新生成“新一代”
8     generation = new Generation();
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

看看怎么打破一个栅栏：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 private void breakBarrier() {
2     // 设置状态 broken 为 true
3     generation.broken = true;
4     // 重置 count 为初始值 parties
5     count = parties;
6     // 唤醒所有已经在等待的线程
7     trip.signalAll();
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

整个过程已经很清楚了。

下面我们来看看怎么得到有多少个线程到了栅栏上，处于等待状态：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public int getNumberWaiting() {
2     final ReentrantLock lock = this.lock;
3     lock.lock();
4     try {
5         return parties - count;
6     } finally {
7         lock.unlock();
8     }
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

判断一个栅栏是否被打破了，这个很简单，直接看 broken 的值即可：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public boolean isBroken() {
2     final ReentrantLock lock = this.lock;
3     lock.lock();
4     try {
5         return generation.broken;
6     } finally {
7         lock.unlock();
8     }
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

最后，我们来看看怎么重置一个栅栏：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void reset() {
 2     final ReentrantLock lock = this.lock;
 3     lock.lock();
 4     try {
 5         breakBarrier();   // break the current generation
 6         nextGeneration(); // start a new generation
 7     } finally {
 8         lock.unlock();
 9     }
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10191106.html#_labelTop)

## Semaphore

有了 CountDownLatch 的基础后，分析 Semaphore 会简单很多。Semaphore 是什么呢？它类似一个资源池（读者可以类比线程池），每个线程需要调用 acquire() 方法获取资源，然后才能执行，执行完后，需要 release 资源，让给其他的线程用。

套路解读：创建 Semaphore 实例的时候，需要一个参数 permits，这个基本上可以确定是设置给 AQS 的 state 的，然后每个线程调用 acquire 的时候，执行 state = state - 1，release 的时候执行 state = state + 1，当然，acquire 的时候，如果 state = 0，说明没有资源了，需要等待其他线程 release。

构造方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 这里和 ReentrantLock 类似，用了公平策略和非公平策略。

看 acquire 方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
public void acquireUninterruptibly() {
    sync.acquireShared(1);
}
public void acquire(int permits) throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireSharedInterruptibly(permits);
}
public void acquireUninterruptibly(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireShared(permits);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

这几个方法也是老套路了，大家基本都懂了吧，这边多了两个可以传参的 acquire 方法，不过大家也都懂的吧，如果我们需要一次获取超过一个的资源，会用得着这个的。

我们接下来看不抛出 InterruptedException 异常的 acquireUninterruptibly() 方法吧：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void acquireUninterruptibly() {
    sync.acquireShared(1);
}
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

前面说了，Semaphore 分公平策略和非公平策略，我们对比一下两个 tryAcquireShared 方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 公平策略：
 2 protected int tryAcquireShared(int acquires) {
 3     for (;;) {
 4         // 区别就在于是不是会先判断是否有线程在排队，然后才进行 CAS 减操作
 5         // 这个就不分析了，第一篇AQS中已经讲过
 6         if (hasQueuedPredecessors())
 7             //进入到这里说明阻塞队列中已经有线程在等着获取资源
 8             return -1;
 9         int available = getState();
10         int remaining = available - acquires;
11         //当remaining最小为0时，会CAS设置state为0,成功返回remaining
12         //当remaining小于0时，这里会直接返回remaining，这里不会执行compareAndSetState
13         if (remaining < 0 ||
14             compareAndSetState(available, remaining))
15             return remaining;
16     }
17 }
18 // 非公平策略：
19 protected int tryAcquireShared(int acquires) {
20     return nonfairTryAcquireShared(acquires);
21 }
22 final int nonfairTryAcquireShared(int acquires) {
23     for (;;) {
24         int available = getState();
25         int remaining = available - acquires;
26         if (remaining < 0 ||
27             compareAndSetState(available, remaining))
28             return remaining;
29     }
30 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们再回到 acquireShared 方法

```
1 public final void acquireShared(int arg) {
2     if (tryAcquireShared(arg) < 0)
3         doAcquireShared(arg);
4 }
```

 

当 tryAcquireShared(arg)大于或者等于0时，获取资源成功，接着执行acquire()后面的业务代码;

当 tryAcquireShared(arg) 返回小于 0 的时候，说明 state 已经小于 0 了（没资源了），此时 acquire 不能立马拿到资源，需要进入到阻塞队列等待,即执行上面第3行代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void doAcquireShared(int arg) {
 2     final Node node = addWaiter(Node.SHARED);
 3     boolean failed = true;
 4     try {
 5         boolean interrupted = false;
 6         for (;;) {
 7             final Node p = node.predecessor();
 8             if (p == head) {
 9                 int r = tryAcquireShared(arg);
10                 if (r >= 0) {
11                     setHeadAndPropagate(node, r);
12                     p.next = null; // help GC
13                     if (interrupted)
14                         selfInterrupt();
15                     failed = false;
16                     return;
17                 }
18             }
19             if (shouldParkAfterFailedAcquire(p, node) &&
20                 parkAndCheckInterrupt())
21                 interrupted = true;
22         }
23     } finally {
24         if (failed)
25             cancelAcquire(node);
26     }
27 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个方法我就不介绍了，前面有很多地方介绍过这个方法，线程挂起后等待有资源被 release 出来。接下来，我们就要看 release 的方法了：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 任务介绍，释放一个资源
 2 public void release() {
 3     sync.releaseShared(1);
 4 }
 5 public final boolean releaseShared(int arg) {
 6     if (tryReleaseShared(arg)) {
 7         doReleaseShared();
 8         return true;
 9     }
10     return false;
11 }
12 
13 protected final boolean tryReleaseShared(int releases) {
14     for (;;) {
15         int current = getState();
16         int next = current + releases;
17         // 溢出，当然，我们一般也不会用这么大的数
18         if (next < current) // overflow
19             throw new Error("Maximum permit count exceeded");
20 　　　　//释放资源后，将state的值又加上释放资源数
21         if (compareAndSetState(current, next))
22             return true;
23     }
24 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

tryReleaseShared 方法总是会返回 true，此时state的资源数已经加上了，然后是 doReleaseShared，这个也是我们熟悉的方法了，我就贴下代码，不分析了，这个方法用于唤醒所有的等待线程中的第一个等待的线程：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void doReleaseShared() {
 2     for (;;) {
 3         Node h = head;
 4         if (h != null && h != tail) {
 5             int ws = h.waitStatus;
 6             if (ws == Node.SIGNAL) {
 7                 if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
 8                     continue;            // loop to recheck cases
 9                 unparkSuccessor(h);
10             }
11             else if (ws == 0 &&
12                      !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
13                 continue;                // loop on failed CAS
14         }
15         if (h == head)                   // loop if head changed
16             break;
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

第一个等待的线程被唤醒后，doReleaseShared终止，接着doAcquireShared()方法被唤醒接着运行，如果资源还够用，则唏嘘唤醒下一个等待节点，可以看到doAcquireShared()方法中第11行处 设置当前节点为head节点，并唤醒下一个等待节点

Semphore 的源码确实很简单，方法都和CountDownLatch 中差不多，基本上都是分析过的老代码的组合使用了。

# [并发编程（八）—— Java 并发队列 BlockingQueue 实现之 ArrayBlockingQueue 源码分析](https://www.cnblogs.com/java-chen-hao/p/10234149.html)



**目录**

- 阻塞队列概要
  - [使用示例](https://www.cnblogs.com/java-chen-hao/p/10234149.html#_label0_0)
- 源码剖析
  - [添加](https://www.cnblogs.com/java-chen-hao/p/10234149.html#_label1_0)
  - [（获取）删除](https://www.cnblogs.com/java-chen-hao/p/10234149.html#_label1_1)

 

**正文**

开篇先介绍下 BlockingQueue 这个接口的规则，后面再看其实现。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10234149.html#_labelTop)

## 阻塞队列概要

阻塞队列与我们平常接触的普通队列(LinkedList或ArrayList等)的最大不同点，在于阻塞队列的阻塞添加和阻塞删除方法。

**阻塞添加**
所谓的阻塞添加是指当阻塞队列元素已满时，队列会阻塞加入元素的线程，直队列元素不满时才重新唤醒线程执行元素加入操作。

**阻塞删除**
阻塞删除是指在队列元素为空时，删除队列元素的线程将被阻塞，直到队列不为空再执行删除操作(一般都会返回被删除的元素)。

 

由于Java中的阻塞队列接口BlockingQueue继承自Queue接口，因此先来看看阻塞队列接口为我们提供的主要方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public interface BlockingQueue<E> extends Queue<E> {
 2 
 3     //将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量）
 4     //在成功时返回 true，如果此队列已满，则抛IllegalStateException。 
 5     boolean add(E e); 
 6 
 7     //将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量） 
 8     // 将指定的元素插入此队列的尾部，如果该队列已满， 
 9     //则在到达指定的等待时间之前等待可用的空间,该方法可中断 
10     boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException; 
11 
12     //将指定的元素插入此队列的尾部，如果该队列已满，则一直等到（阻塞）。 
13     void put(E e) throws InterruptedException; 
14 
15     //获取并移除此队列的头部，如果没有元素则等待（阻塞）， 
16     //直到有元素将唤醒等待线程执行该操作 
17     E take() throws InterruptedException; 
18 
19     //获取并移除此队列的头部，在指定的等待时间前一直等到获取元素， //超过时间方法将结束
20     E poll(long timeout, TimeUnit unit) throws InterruptedException; 
21 
22     //从此队列中移除指定元素的单个实例（如果存在）。 
23     boolean remove(Object o); 
24 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里我们把上述操作进行分类

**插入方法：**

　　add(E e) : 添加成功返回true，失败抛IllegalStateException异常
　　offer(E e) : 成功返回 true，如果此队列已满，则返回 false。
　　put(E e) :将元素插入此队列的尾部，如果该队列已满，则一直阻塞
**删除方法：**

　　remove(Object o) :移除指定元素,成功返回true，失败返回false
　　poll() : 获取并移除此队列的头元素，若队列为空，则返回 null
　　take()：获取并移除此队列头元素，若没有元素则一直阻塞。

阻塞队列的对元素的增删查操作主要就是上述的三类方法，通常情况下我们都是通过这3类方法操作阻塞队列，了解完阻塞队列的基本方法后，下面我们将分析阻塞队列中的两个实现类ArrayBlockingQueue和LinkedBlockingQueue的简单使用和实现原理，其中实现原理是这篇文章重点分析的内容。

# ArrayBlockingQueue

在看源码之前，通过查询API发现对ArrayBlockingQueue特点的简单介绍：

1、一个由数组支持的有界队列，此队列按FIFO（先进先出）原则对元素进行排序。
2、新元素插入到队列的尾部，队列获取操作则是从队列头部开始获得元素
3、这是一个简单的“有界缓存区”，一旦创建，就不能在增加其容量
4、在向已满队列中添加元素会导致操作阻塞，从空队列中提取元素也将导致阻塞
5、此类支持对等待的生产者线程和使用者线程进行排序的可选公平策略。默认情况下，不保证是这种排序的。然而通过将公平性（fairness）设置为true，而构造的队列允许按照FIFO顺序访问线程。公平性通常会降低吞吐量，但也减少了可变性和避免了“不平衡性”。

简单的来说，ArrayBlockingQueue 是一个用数组实现的有界阻塞队列，其内部按先进先出的原则对元素进行排序，其中put方法和take方法为添加和删除的阻塞方法，下面我们通过ArrayBlockingQueue队列实现一个生产者消费者的案例，通过该案例简单了解其使用方式



### 使用示例

Consumer 消费者和 Producer 生产者，通过ArrayBlockingQueue 队列获取和添加元素，其中消费者调用了take()方法获取元素当队列没有元素就阻塞，生产者调用put()方法添加元素，当队列满时就阻塞，通过这种方式便实现生产者消费者模式。比直接使用等待唤醒机制或者Condition条件队列来得更加简单。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 package com.zejian.concurrencys.Queue;
 2 import java.util.concurrent.ArrayBlockingQueue;
 3 import java.util.concurrent.TimeUnit;
 4 
 5 /**
 6  * Created by chenhao on 2018/01/07
 7  */
 8 public class ArrayBlockingQueueDemo {
 9     private final static ArrayBlockingQueue<Apple> queue= new ArrayBlockingQueue<>(1);
10     public static void main(String[] args){
11         new Thread(new Producer(queue)).start();
12         new Thread(new Producer(queue)).start();
13         new Thread(new Consumer(queue)).start();
14         new Thread(new Consumer(queue)).start();
15     }
16 }
17 
18  class Apple {
19     public Apple(){
20     }
21  }
22 
23 /**
24  * 生产者线程
25  */
26 class Producer implements Runnable{
27     private final ArrayBlockingQueue<Apple> mAbq;
28     Producer(ArrayBlockingQueue<Apple> arrayBlockingQueue){
29         this.mAbq = arrayBlockingQueue;
30     }
31 
32     @Override
33     public void run() {
34         while (true) {
35             Produce();
36         }
37     }
38 
39     private void Produce(){
40         try {
41             Apple apple = new Apple();
42             mAbq.put(apple);
43             System.out.println("生产:"+apple);
44         } catch (InterruptedException e) {
45             e.printStackTrace();
46         }
47     }
48 }
49 
50 /**
51  * 消费者线程
52  */
53 class Consumer implements Runnable{
54 
55     private ArrayBlockingQueue<Apple> mAbq;
56     Consumer(ArrayBlockingQueue<Apple> arrayBlockingQueue){
57         this.mAbq = arrayBlockingQueue;
58     }
59 
60     @Override
61     public void run() {
62         while (true){
63             try {
64                 TimeUnit.MILLISECONDS.sleep(1000);
65                 comsume();
66             } catch (InterruptedException e) {
67                 e.printStackTrace();
68             }
69         }
70     }
71 
72     private void comsume() throws InterruptedException {
73         Apple apple = mAbq.take();
74         System.out.println("消费Apple="+apple);
75     }
76 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

输出：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 生产:com.zejian.concurrencys.Queue.Apple@109967f
2 消费Apple=com.zejian.concurrencys.Queue.Apple@109967f
3 生产:com.zejian.concurrencys.Queue.Apple@269a77
4 生产:com.zejian.concurrencys.Queue.Apple@1ce746e
5 消费Apple=com.zejian.concurrencys.Queue.Apple@269a77
6 消费Apple=com.zejian.concurrencys.Queue.Apple@1ce746e
7 ........
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10234149.html#_labelTop)

## 源码剖析

ArrayBlockingQueue内部的阻塞队列是通过重入锁ReenterLock和Condition条件队列实现的，所以ArrayBlockingQueue中的元素存在公平访问与非公平访问的区别，对于公平访问队列，被阻塞的线程可以按照阻塞的先后顺序访问队列，即先阻塞的线程先访问队列。而非公平队列，当队列可用时，阻塞的线程将进入争夺访问资源的竞争中，也就是说谁先抢到谁就执行，没有固定的先后顺序。创建公平与非公平阻塞队列代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //默认非公平阻塞队列
 2 ArrayBlockingQueue queue = new ArrayBlockingQueue(2);
 3 //公平阻塞队列
 4 ArrayBlockingQueue queue1 = new ArrayBlockingQueue(2,true);
 5 
 6 //构造方法源码
 7 public ArrayBlockingQueue(int capacity) {
 8      this(capacity, false);
 9  }
10 
11 public ArrayBlockingQueue(int capacity, boolean fair) {
12      if (capacity <= 0)
13          throw new IllegalArgumentException();
14      this.items = new Object[capacity];
15      lock = new ReentrantLock(fair);
16      notEmpty = lock.newCondition();
17      notFull =  lock.newCondition();
18  }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ArrayBlockingQueue的内部是通过一个可重入锁ReentrantLock和两个Condition条件对象来实现阻塞，这里先看看其内部成员变量

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class ArrayBlockingQueue<E> extends AbstractQueue<E>
 2         implements BlockingQueue<E>, java.io.Serializable {
 3 
 4     /** 存储数据的数组 */
 5     final Object[] items;
 6 
 7     /**获取数据的索引，主要用于take，poll，peek，remove方法 */
 8     int takeIndex;
 9 
10     /**添加数据的索引，主要用于 put, offer, or add 方法*/
11     int putIndex;
12 
13     /** 队列元素的个数 */
14     int count;
15 
16 
17     /** 控制并非访问的锁 */
18     final ReentrantLock lock;
19 
20     /**notEmpty条件对象，用于通知take方法队列已有元素，可执行获取操作 */
21     private final Condition notEmpty;
22 
23     /**notFull条件对象，用于通知put方法队列未满，可执行添加操作 */
24     private final Condition notFull;
25 
26     /**
27        迭代器
28      */
29     transient Itrs itrs = null;
30 
31 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从成员变量可看出，ArrayBlockingQueue内部确实是通过数组对象items来存储所有的数据，值得注意的是ArrayBlockingQueue通过一个ReentrantLock来同时控制添加线程与移除线程的并非访问，这点与LinkedBlockingQueue区别很大(稍后会分析)。而对于notEmpty条件对象则是用于存放等待或唤醒调用take方法的线程，告诉他们队列已有元素，可以执行获取操作。同理notFull条件对象是用于等待或唤醒调用put方法的线程，告诉它们，队列未满，可以执行添加元素的操作。takeIndex代表的是下一个方法(take，poll，peek，remove)被调用时获取数组元素的索引，putIndex则代表下一个方法（put, offer, or add）被调用时元素添加到数组中的索引。图示如下

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190107155116806-1578732421.png)



### 添加

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //add方法实现，间接调用了offer(e)
 2 public boolean add(E e) {
 3         if (offer(e))
 4             return true;
 5         else
 6             throw new IllegalStateException("Queue full");
 7     }
 8 
 9 //offer方法
10 public boolean offer(E e) {
11      checkNotNull(e);//检查元素是否为null
12      final ReentrantLock lock = this.lock;
13      lock.lock();//加锁
14      try {
15          if (count == items.length)//判断队列是否满
16              return false;
17          else {
18              enqueue(e);//添加元素到队列
19              return true;
20          }
21      } finally {
22          lock.unlock();
23      }
24  }
25 
26 //入队操作
27 private void enqueue(E x) {
28     //获取当前数组
29     final Object[] items = this.items;
30     //通过putIndex索引对数组进行赋值
31     items[putIndex] = x;
32     //索引自增，如果已是最后一个位置，重新设置 putIndex = 0;
33     if (++putIndex == items.length)
34         putIndex = 0;
35     count++;//队列中元素数量加1
36     //唤醒调用take()方法的线程，执行元素获取操作。
37     notEmpty.signal();
38 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里的add方法和offer方法实现比较简单，其中需要注意的是enqueue(E x)方法，当putIndex索引大小等于数组长度时，需要将putIndex重新设置为0，因为后面讲到的取值也是从数组中第一个开始依次往后面取，取了之后会将原位置的值设置为null，方便循环put操作，这里要注意并不是每次都是取数组中的第一个值，takeIndex也会增加。因为做了添加操作，数组中肯定不会空，则 notEmpty条件会唤醒take()方法取值。

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190107162545355-767790211.png)

ok~,接着看put方法，它是一个阻塞添加的方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //put方法，阻塞时可中断
 2  public void put(E e) throws InterruptedException {
 3      checkNotNull(e);
 4       final ReentrantLock lock = this.lock;
 5       lock.lockInterruptibly();//该方法可中断
 6       try {
 7           //当队列元素个数与数组长度相等时，无法添加元素
 8           while (count == items.length)
 9               //将当前调用线程挂起，添加到notFull条件队列中等待唤醒
10               notFull.await();
11           enqueue(e);//如果队列没有满直接添加。。
12       } finally {
13           lock.unlock();
14       }
15   }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

put方法是一个阻塞的方法，如果队列元素已满，那么当前线程将会被notFull条件对象挂起加到等待队列中，直到队列有空档才会唤醒执行添加操作。但如果队列没有满，那么就直接调用enqueue(e)方法将元素加入到数组队列中。到此我们对三个添加方法即put，offer，add都分析完毕，其中offer，add在正常情况下都是无阻塞的添加，而put方法是阻塞添加。



### （获取）删除

关于删除先看poll方法，该方法获取并移除此队列的头元素，若队列为空，则返回 null。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public E poll() {
 2   final ReentrantLock lock = this.lock;
 3    lock.lock();
 4    try {
 5        //判断队列是否为null，不为null执行dequeue()方法，否则返回null
 6        return (count == 0) ? null : dequeue();
 7    } finally {
 8        lock.unlock();
 9    }
10 }
11  //删除队列头元素并返回
12  private E dequeue() {
13      //拿到当前数组的数据
14      final Object[] items = this.items;
15       @SuppressWarnings("unchecked")
16       //获取要删除的对象
17       E x = (E) items[takeIndex];
18       将数组中takeIndex索引位置设置为null
19       items[takeIndex] = null;
20       //takeIndex索引加1并判断是否与数组长度相等，
21       //如果相等说明已到尽头，恢复为0
22       if (++takeIndex == items.length)
23           takeIndex = 0;
24       count--;//队列个数减1
25       if (itrs != null)
26           itrs.elementDequeued();//同时更新迭代器中的元素数据
27       //删除了元素说明队列有空位，唤醒notFull条件对象添加线程，执行添加操作
28       notFull.signal();
29       return x;
30  }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

接着看take()方法，是一个阻塞方法，获取队列头元素并删除。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //从队列头部删除，队列没有元素就阻塞，可中断
 2  public E take() throws InterruptedException {
 3     final ReentrantLock lock = this.lock;
 4       lock.lockInterruptibly();//中断
 5       try {
 6           //如果队列没有元素
 7           while (count == 0)
 8               //执行阻塞操作
 9               notEmpty.await();
10           return dequeue();//如果队列有元素执行删除操作
11       } finally {
12           lock.unlock();
13       }
14  }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

take和poll的区别是，队列为空时，poll返回null，take则被挂起阻塞，直到有元素添加进来，take线程被唤醒，然后获取第一个元素并删除。

 

peek方法非常简单，直接返回当前队列的头元素但不删除任何元素。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public E peek() {
 2       final ReentrantLock lock = this.lock;
 3       lock.lock();
 4       try {
 5        //直接返回当前队列的头元素，但不删除
 6           return itemAt(takeIndex); // null when queue is empty
 7       } finally {
 8           lock.unlock();
 9       }
10   }
11 
12 final E itemAt(int i) {
13       return (E) items[i];
14   }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ok~，到此对于ArrayBlockingQueue的主要方法就分析完了。

# [并发编程（九）—— Java 并发队列 BlockingQueue 实现之 LinkedBlockingQueue 源码分析](https://www.cnblogs.com/java-chen-hao/p/10234833.html)



**目录**

- 源码剖析
  - [添加方法的实现原理](https://www.cnblogs.com/java-chen-hao/p/10234833.html#_label0_0)
  - [移除方法的实现原理](https://www.cnblogs.com/java-chen-hao/p/10234833.html#_label0_1)
- [LinkedBlockingQueue和ArrayBlockingQueue迥异](https://www.cnblogs.com/java-chen-hao/p/10234833.html#_label1)

 

**正文**

# LinkedBlockingQueue

在看源码之前，通过查询API发现对LinkedBlockingQueue特点的简单介绍：

1、LinkedBlockingQueue是一个由链表实现的有界队列阻塞队列。
2、新元素插入到队列的尾部，队列获取操作则是从队列头部开始获得元素
3、大小默认值为Integer.MAX_VALUE，所以我们在使用LinkedBlockingQueue时建议手动传值，为其提供我们所需的大小，避免队列过大造成机器负载或者内存爆满等情况。

4、链接队列的吞吐量通常要高于基于数组的对列（ArrayBlockingQueue）,但是在大多数并发应用程序中，其可预知的性能要低

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10234833.html#_labelTop)

## 源码剖析

通常情况下，我们建议创建指定大小的LinkedBlockingQueue阻塞队列。在正常情况下，链接队列的吞吐量要高于基于数组的队列（ArrayBlockingQueue），因为其内部实现添加和删除操作使用的两个ReenterLock来控制并发执行，而ArrayBlockingQueue内部只是使用一个ReenterLock控制并发，因此LinkedBlockingQueue的吞吐量要高于ArrayBlockingQueue。

其构造函数如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //默认大小为Integer.MAX_VALUE
 2 public LinkedBlockingQueue() {
 3        this(Integer.MAX_VALUE);
 4 }
 5 
 6 //创建指定大小为capacity的阻塞队列
 7 public LinkedBlockingQueue(int capacity) {
 8      if (capacity <= 0) throw new IllegalArgumentException();
 9      this.capacity = capacity;
10      last = head = new Node<E>(null);
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们先看看LinkedBlockingQueue的内部成员变量：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class LinkedBlockingQueue<E> extends AbstractQueue<E>
 2         implements BlockingQueue<E>, java.io.Serializable {
 3 
 4     /**
 5      * 节点类，用于存储数据
 6      */
 7     static class Node<E> {
 8         E item;
 9 
10         /**
11          * One of:
12          * - the real successor Node
13          * - this Node, meaning the successor is head.next
14          * - null, meaning there is no successor (this is the last node)
15          */
16         Node<E> next;
17 
18         Node(E x) { item = x; }
19     }
20 
21     /** 阻塞队列的大小，默认为Integer.MAX_VALUE */
22     private final int capacity;
23 
24     /** 当前阻塞队列中的元素个数 */
25     private final AtomicInteger count = new AtomicInteger();
26 
27     /**
28      * 阻塞队列的头结点
29      */
30     transient Node<E> head;
31 
32     /**
33      * 阻塞队列的尾节点
34      */
35     private transient Node<E> last;
36 
37     /** 获取并移除元素时使用的锁，如take, poll, etc */
38     private final ReentrantLock takeLock = new ReentrantLock();
39 
40     /** notEmpty条件对象，当队列没有数据时用于挂起执行删除的线程 */
41     private final Condition notEmpty = takeLock.newCondition();
42 
43     /** 添加元素时使用的锁如 put, offer, etc */
44     private final ReentrantLock putLock = new ReentrantLock();
45 
46     /** notFull条件对象，当队列数据已满时用于挂起执行添加的线程 */
47     private final Condition notFull = putLock.newCondition();
48 
49 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

与ArrayBlockingQueue不同的是，LinkedBlockingQueue内部分别使用了takeLock 和 putLock 对并发进行控制，也就是说，添加和删除操作并不是互斥操作，可以同时进行，这样也就可以大大提高吞吐量。这里再次强调如果没有给LinkedBlockingQueue指定容量大小，其默认值将是Integer.MAX_VALUE，如果存在添加速度大于删除速度时候，有可能会内存溢出，这点在使用前希望慎重考虑。

这里用了两个锁，两个 Condition，简单介绍如下：

**takeLock 和 notEmpty 怎么搭配**：如果要获取（take）一个元素，需要获取 takeLock 锁，但是获取了锁还不够，如果队列此时为空，还需要队列不为空（notEmpty）这个条件（Condition）。

**putLock 需要和 notFull 搭配**：如果要插入（put）一个元素，需要获取 putLock 锁，但是获取了锁还不够，如果队列此时已满，还需要队列不是满的（notFull）这个条件（Condition）。

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190107171109398-746679734.png)

 

下面我们看看其其内部添加过程和删除过程是如何实现的。



### 添加方法的实现原理

对于添加方法，主要指的是add，offer以及put，这里先看看add方法和offer方法的实现

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public boolean add(E e) {
2      if (offer(e))
3          return true;
4      else
5          throw new IllegalStateException("Queue full");
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从源码可以看出，add方法间接调用的是offer方法，如果add方法添加失败将抛出IllegalStateException异常，添加成功则返回true，那么下面我们直接看看offer的相关方法实现

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public boolean offer(E e) {
 2      //添加元素为null直接抛出异常
 3      if (e == null) throw new NullPointerException();
 4       //获取队列的个数
 5       final AtomicInteger count = this.count;
 6       //判断队列是否已满
 7       if (count.get() == capacity)
 8           return false;
 9       int c = -1;
10       //构建节点
11       Node<E> node = new Node<E>(e);
12       final ReentrantLock putLock = this.putLock;
13       //此时可能有多个线程都在执行添加操作，抢占锁
14       //没有抢占到锁的线程会加入到putLock的阻塞队列，并且挂起
15       putLock.lock();
16       try {
17           //再次判断队列是否已满，考虑并发情况
18           if (count.get() < capacity) {
19               enqueue(node);//添加元素
20               c = count.getAndIncrement();//拿到当前未添加新元素时的队列长度
21               //如果容量还没满
22               if (c + 1 < capacity)
23                   //此时会唤醒上面第15行处添加到putLock的阻塞队列的线程，被唤醒的线程满足条件后接着唤醒下一个，直到队列被添加满
24                   notFull.signal();//唤醒下一个添加线程，执行添加操作
25           }
26       } finally {
27           putLock.unlock();
28       }
29       // 由于存在添加锁和消费锁，而消费锁和添加锁都会持续唤醒等到线程，因此count肯定会变化。
30       //这里的if条件表示如果队列中还有1条数据
31       if (c == 0) 
32         signalNotEmpty();//如果还存在数据那么就唤醒消费锁
33     return c >= 0; // 添加成功返回true，否则返回false
34 }
35 
36 //入队操作
37 private void enqueue(Node<E> node) {
38      //队列尾节点指向新的node节点
39      last = last.next = node;
40 }
41 
42 //signalNotEmpty方法
43 private void signalNotEmpty() {
44       final ReentrantLock takeLock = this.takeLock;
45       takeLock.lock();
46           //唤醒获取并删除元素的线程
47           notEmpty.signal();
48       } finally {
49           takeLock.unlock();
50       }
51 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

这里的Offer()方法做了两件事，第一件事是判断队列是否满，满了就直接释放锁，没满就将节点封装成Node入队，然后再次判断队列添加完成后是否已满，不满就继续唤醒等到在条件对象notFull上的添加线程。第二件事是，判断是否需要唤醒等到在notEmpty条件对象上的消费线程。这里我们可能会有点疑惑，为什么添加完成后是继续唤醒在条件对象notFull上的添加线程而不是像ArrayBlockingQueue那样直接唤醒notEmpty条件对象上的消费线程？而又为什么要当if (c == 0)时才去唤醒消费线程呢？

　　唤醒添加线程的原因，在添加新元素完成后，会判断队列是否已满，不满就继续唤醒在条件对象notFull上的添加线程，这点与前面分析的ArrayBlockingQueue很不相同，在ArrayBlockingQueue内部完成添加操作后，会直接唤醒消费线程对元素进行获取，这是因为ArrayBlockingQueue只用了一个ReenterLock同时对添加线程和消费线程进行控制，这样如果在添加完成后再次唤醒添加线程的话，消费线程可能永远无法执行，而对于LinkedBlockingQueue来说就不一样了，其内部对添加线程和消费线程分别使用了各自的ReenterLock锁对并发进行控制，也就是说添加线程和消费线程是不会互斥的，所以添加锁只要管好自己的添加线程即可，添加线程自己直接唤醒自己的其他添加线程，如果没有等待的添加线程，直接结束了。如果有就直到队列元素已满才结束挂起，当然offer方法并不会挂起，而是直接结束，只有put方法才会当队列满时才执行挂起操作。注意消费线程的执行过程也是如此。这也是为什么LinkedBlockingQueue的吞吐量要相对大些的原因。

　　为什么要判断if (c == 0)时才去唤醒消费线程呢，这是因为消费线程一旦被唤醒是一直在消费的（前提是有数据），所以c值是一直在变化的，c值是添加完元素前队列的大小，此时c只可能是0或c>0，如果是c=0，那么说明之前消费线程已停止，条件对象上可能存在等待的消费线程，添加完数据后应该是c+1，那么有数据就直接唤醒等待消费线程，如果没有就结束啦，等待下一次的消费操作。如果c>0那么消费线程就不会被唤醒，只能等待下一个消费操作（poll、take、remove）的调用，那为什么不是条件c>0才去唤醒呢？我们要明白的是消费线程一旦被唤醒会和添加线程一样，一直不断唤醒其他消费线程，如果添加前c>0，那么很可能上一次调用的消费线程后，数据并没有被消费完，条件队列上也就不存在等待的消费线程了，还有可能，消费线程正在消费，消费队列中也有等待线程，但是消费线程消费完会自动唤醒下一个等待的消费线程，所以c>0唤醒消费线程得意义不是很大，当然如果添加线程一直添加元素，那么一直c>0，消费线程执行的换就要等待下一次调用消费操作了（poll、take、remove）。

 

我们来看看带超时的offer()

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public boolean offer(E e, long timeout, TimeUnit unit)
 2     throws InterruptedException {
 3     return offerLast(e, timeout, unit);
 4 }
 5 
 6 public boolean offerLast(E e, long timeout, TimeUnit unit)
 7     throws InterruptedException {
 8     if (e == null) throw new NullPointerException();
 9     Node<E> node = new Node<E>(e);
10     long nanos = unit.toNanos(timeout);
11     final ReentrantLock lock = this.lock;
12     lock.lockInterruptibly();
13     try {
14         //加入队列失败就进入while循环
15         while (!linkLast(node)) {
16             if (nanos <= 0)
17                 //过了nano还没有入队成功，返回false
18                 return false;
19             //如果没有被signal，就一直挂起nanos纳秒后醒来
20             nanos = notFull.awaitNanos(nanos);
21         }
22         //走到这里说明入队成功，返回true
23         return true;
24     } finally {
25         lock.unlock();
26     }
27 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

我们可以知道offer方法进行了阻塞超时处理，在 timeout 时间内没有入队成功，就一直阻塞，入队成功，返回false,超过 timeout 还没入队成功，返回false。

接下来我们看看put方法，它是一个阻塞添加的方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void put(E e) throws InterruptedException {
 2     if (e == null) throw new NullPointerException();
 3     int c = -1;
 4     Node<E> node = new Node(e);
 5     final ReentrantLock putLock = this.putLock;
 6     final AtomicInteger count = this.count;
 7     putLock.lockInterruptibly();
 8     try {
 9         // 如果队列满，等待 notFull 的条件满足。
10         while (count.get() == capacity) {
11             notFull.await();
12         }
13         // 入队
14         enqueue(node);
15         // count 原子加 1，c 还是加 1 前的值
16         c = count.getAndIncrement();
17         if (c + 1 < capacity)
18             notFull.signal();
19     } finally {
20         // 入队后，释放掉 putLock
21         putLock.unlock();
22     }
23     if (c == 0)
24         signalNotEmpty();
25 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

put 和 offer最大的区别是put方法是阻塞的，看offer 方法中的第6行，如果队列满了，则直接返回false；put方法的第10行，如果队列满了，则添加到 notFull 的等待队列中并挂起，其他的基本都一样。

 



### 移除方法的实现原理

我们先来看看poll()方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public E poll() {
 2          //获取当前队列的大小
 3         final AtomicInteger count = this.count;
 4         if (count.get() == 0)//如果没有元素直接返回null
 5             return null;
 6         E x = null;
 7         int c = -1;
 8         final ReentrantLock takeLock = this.takeLock;
 9         takeLock.lock();
10         try {
11             //判断队列是否有数据
12             if (count.get() > 0) {
13                 //如果有，直接删除并获取该元素值
14                 x = dequeue();
15                 //当前队列大小减一
16                 c = count.getAndDecrement();
17                 //如果队列未空，继续唤醒等待在条件对象notEmpty上的消费线程
18                 if (c > 1)
19                     notEmpty.signal();
20             }
21         } finally {
22             takeLock.unlock();
23         }
24         //判断c是否等于capacity，这是因为如果满说明NotFull条件对象上
25         //可能存在等待的添加线程
26         if (c == capacity)
27             signalNotFull();
28         return x;
29 }
30 
31 private E dequeue() {
32         Node<E> h = head;//获取头结点
33         Node<E> first = h.next; 获取头结的下一个节点（要删除的节点）
34         h.next = h; // help GC//自己next指向自己，即被删除
35         head = first;//更新头结点
36         E x = first.item;//获取删除节点的值
37         first.item = null;//清空数据，因为first变成头结点是不能带数据的，这样也就删除队列的带数据的第一个节点
38         return x;
39 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

poll方法也比较简单，如果队列没有数据就返回null，如果队列有数据，那么就取出来，如果队列还有数据那么唤醒等待在条件对象notEmpty上的消费线程。然后判断if (c == capacity)为true就唤醒添加线程，这点与前面分析if(c==0)是一样的道理。因为只有可能队列满了，notFull条件对象上才可能存在等待的添加线程。

我们再看看take()方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public E take() throws InterruptedException {
 2         E x;
 3         int c = -1;
 4         //获取当前队列大小
 5         final AtomicInteger count = this.count;
 6         final ReentrantLock takeLock = this.takeLock;
 7         takeLock.lockInterruptibly();//可中断
 8         try {
 9             //如果队列没有数据，挂机当前线程到条件对象的等待队列中
10             while (count.get() == 0) {
11                 notEmpty.await();
12             }
13             //如果存在数据直接删除并返回该数据
14             x = dequeue();
15             c = count.getAndDecrement();//队列大小减1
16             if (c > 1)
17                 notEmpty.signal();//还有数据就唤醒后续的消费线程
18         } finally {
19             takeLock.unlock();
20         }
21         //满足条件，唤醒条件对象上等待队列中的添加线程
22         if (c == capacity)
23             signalNotFull();
24         return x;
25 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

take方法是一个可阻塞可中断的移除方法，主要做了两件事，一是，如果队列没有数据就挂起当前线程到 notEmpty条件对象的等待队列中一直等待，如果有数据就删除节点并返回数据项，同时唤醒后续消费线程，二是尝试唤醒条件对象notFull上等待队列中的添加线程。 到此关于remove、poll、take的实现也分析完了，其中只有take方法具备阻塞功能。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10234833.html#_labelTop)

## LinkedBlockingQueue和ArrayBlockingQueue迥异

对于LinkedBlockingQueue和ArrayBlockingQueue的基本使用以及内部实现原理我们已较为熟悉了，这里我们就对它们两间的区别来个小结

1.队列大小有所不同，ArrayBlockingQueue是有界的初始化必须指定大小，而LinkedBlockingQueue可以是有界的也可以是无界的(Integer.MAX_VALUE)，对于后者而言，当添加速度大于移除速度时，在无界的情况下，可能会造成内存溢出等问题。

2.数据存储容器不同，ArrayBlockingQueue采用的是数组作为数据存储容器，而LinkedBlockingQueue采用的则是以Node节点作为连接对象的链表。

3.由于ArrayBlockingQueue采用的是数组的存储容器，因此在插入或删除元素时不会产生或销毁任何额外的对象实例，而LinkedBlockingQueue则会生成一个额外的Node对象。这可能在长时间内需要高效并发地处理大批量数据的时，对于GC可能存在较大影响。

4.两者的实现队列添加或移除的锁不一样，ArrayBlockingQueue实现的队列中的锁是没有分离的，即添加操作和移除操作采用的同一个ReenterLock锁，而LinkedBlockingQueue实现的队列中的锁是分离的，其添加采用的是putLock，移除采用的则是takeLock，这样能大大提高队列的吞吐量，也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。

# [并发编程（十）—— Java 并发队列 BlockingQueue 实现之 SynchronousQueue源码分析](https://www.cnblogs.com/java-chen-hao/p/10238762.html)



**目录**

- [BlockingQueue 实现之 SynchronousQueue](https://www.cnblogs.com/java-chen-hao/p/10238762.html#_label0)
- [总结](https://www.cnblogs.com/java-chen-hao/p/10238762.html#_label1)

 

**正文**

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10238762.html#_labelTop)

## BlockingQueue 实现之 SynchronousQueue

[SynchronousQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/SynchronousQueue.html)是一个没有数据缓冲的[BlockingQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/BlockingQueue.html)，生产者线程对其的插入操作put必须等待消费者的移除操作take，反过来也一样。

不像ArrayBlockingQueue或LinkedListBlockingQueue，SynchronousQueue内部并没有数据缓存空间，你不能调用peek()方法来看队列中是否有数据元素，因为数据元素只有当你试着取走的时候才可能存在，不取走而只想偷窥一下是不行的，当然遍历这个队列的操作也是不允许的。队列头元素是第一个排队要插入数据的**线程**，而不是要交换的数据。数据是在配对的生产者和消费者线程之间直接传递的，并不会将数据缓冲数据到队列中。可以这样来理解：生产者和消费者互相等待对方，握手，然后**一起**离开。

SynchronousQueue的一个使用场景是在线程池里。Executors.newCachedThreadPool()就使用了SynchronousQueue，这个线程池根据需要（新任务到来时）创建新的线程，如果有空闲线程则会重复使用，线程空闲了60秒后会被回收。

接下来，我们来看看具体的源码实现吧，它的源码不是很简单的那种，我们需要先搞清楚它的设计思想。

我们先看大框架：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 构造时，我们可以指定公平模式还是非公平模式，本文主要讲解公平模式
 2 public SynchronousQueue(boolean fair) {
 3     transferer = fair ? new TransferQueue() : new TransferStack();
 4 }
 5 abstract static class Transferer {
 6     // 从方法名上大概就知道，这个方法用于转移元素，从生产者手上转到消费者手上
 7     // 也可以被动地，消费者调用这个方法来从生产者手上取元素
 8     // 第一个参数 e 如果不是 null，代表场景为：将元素从生产者转移给消费者
 9     // 如果是 null，代表消费者等待生产者提供元素，然后返回值就是相应的生产者提供的元素
10     // 第二个参数代表是否设置超时，如果设置超时，超时时间是第三个参数的值
11     // 返回值如果是 null，代表超时，或者中断。具体是哪个，可以通过检测中断状态得到。
12     abstract Object transfer(Object e, boolean timed, long nanos);
13 }
14 
15 TransferQueue() {
16     //初始化时，head和tail都是空节点
17     QNode h = new QNode(null, false); // initialize to dummy node.
18     head = h;
19     tail = h;
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Transferer 有两个内部实现类，是因为构造 SynchronousQueue 的时候，我们可以指定公平策略。公平模式意味着，所有的读写线程都遵守先来后到，FIFO 嘛，对应 TransferQueue。而非公平模式则对应 TransferStack。

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190104162101303-1264535523.png)

本文我们主要看公平模式源码，接下来，我们看看 put 方法和 take 方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 写入值
 2 public void put(E o) throws InterruptedException {
 3     if (o == null) throw new NullPointerException();
 4     if (transferer.transfer(o, false, 0) == null) { // 1
 5         Thread.interrupted();
 6         throw new InterruptedException();
 7     }
 8 }
 9 // 读取值并移除
10 public E take() throws InterruptedException {
11     Object e = transferer.transfer(null, false, 0); // 2
12     if (e != null)
13         return (E)e;
14     Thread.interrupted();
15     throw new InterruptedException();
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看到，写操作 put(E o) 和读操作 take() 都是调用 Transferer.transfer(…) 方法，区别在于第一个参数是否为 null 值。

我们来看看 transfer 的设计思路，其基本算法如下：

1. 当调用这个方法时，如果队列是空的，或者队列中的节点和当前的线程操作类型一致（如当前操作是 put 操作，而队列中的元素也都是写线程）。这种情况下，将**当前线程加入到等待队列并阻塞线程**。
2. 如果队列中有等待节点，而且与当前操作可以匹配（1、如队列中都是读操作线程，当前线程是写操作线程；2、如队列中都是写操作线程，当前线程是读操作；）。这种情况下，**匹配等待队列的队头，出队，返回相应数据**。

 其实这里有个隐含的条件被满足了，队列如果不为空，肯定都是同种类型的节点，要么都是读操作，要么都是写操作。这个就要看到底是读线程积压了，还是写线程积压了。

我们可以假设出一个男女配对的场景：

　　1、一个男的过来，如果一个人都没有，那么他需要等待；如果发现有一堆男的在等待，那么他需要排到队列后面；如果发现是一堆女的在排队，那么他直接牵走队头的那个女的；

　　2、相反一个女的过来，如果一个人都没有，那么她需要等待；如果发现有一堆女的在等待，那么她需要排到队列后面；如果发现是一堆男的在排队，那么队头的那个男的直接出队牵走这个女的；

既然这里说到了等待队列，我们先看看其实现，也就是 QNode:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 static final class QNode {
 2     volatile QNode next;          // 可以看出来，等待队列是单向链表
 3     volatile Object item;         // CAS'ed to or from null
 4     volatile Thread waiter;       // 将线程对象保存在这里，用于挂起和唤醒
 5     final boolean isData;         // 用于判断是写线程节点(isData == true)，还是读线程节点
 6 
 7     QNode(Object item, boolean isData) {
 8         this.item = item;
 9         this.isData = isData;
10     }
11   ......
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们再来看 transfer 方法的代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /**
 2  * Puts or takes an item.
 3  */
 4 Object transfer(Object e, boolean timed, long nanos) {
 5 
 6     QNode s = null; // constructed/reused as needed
 7     boolean isData = (e != null);
 8 
 9     for (;;) {
10         QNode t = tail;
11         QNode h = head;
12         if (t == null || h == null)         // saw uninitialized value
13             //说明还没有初始化，则跳出继续循环，直至初始化完成
14             continue;                       // spin
15 
16         // 走到这里，说明已经初始化完成，但是初始化时head = h;tail = h;head和tail都是相同的空节点
17         // 如果h == t为false，则判断t.isData == isData，判断队尾节点和当前节点类型是否一致
18         // 队列空，或队列中节点类型和当前节点一致，
19         // 即我们说的第一种情况，将节点入队即可。读者要想着这块 if 里面方法其实就是入队
20         if (h == t || t.isData == isData) { // empty or same-mode
21             QNode tn = t.next;
22             // t != tail 说明刚刚有节点入队，continue 即可
23             if (t != tail)                  // inconsistent read
24                 continue;
25             // 有其他节点入队，但是 tail 还是指向原来的，此时设置 tail 即可
26             if (tn != null) {               // lagging tail
27                 // 这个方法就是：如果 tail 此时为 t 的话，设置为 tn
28                 advanceTail(t, tn);
29                 continue;
30             }
31             // 
32             if (timed && nanos <= 0)        // can't wait
33                 return null;
34             // s == null，则创建一个新节点
35             if (s == null)
36                 s = new QNode(e, isData);
37             // 将当前节点，插入到 tail 的后面
38             if (!t.casNext(null, s))        // failed to link in
39                 continue;
40 
41             // 将当前节点设置为新的 tail
42             advanceTail(t, s);              // swing tail and wait
43             // 看到这里，请读者先往下滑到这个方法，看完了以后再回来这里，思路也就不会断了
44             Object x = awaitFulfill(s, e, timed, nanos);
45             // 到这里，说明之前入队的线程被唤醒了，准备往下执行
46             // 若返回的x == s表示，当前线程已经超时或者中断，不然的话s == null或者是匹配的节点
47             if (x == s) {                   // wait was cancelled
48                 clean(t, s);
49                 return null;
50             }
51             // 若s节点被设置为取消
52             if (!s.isOffList()) {           // not already unlinked
53                 advanceHead(t, s);          // unlink if head
54                 if (x != null)              // and forget fields
55                     s.item = s;
56                 s.waiter = null;
57             }
58             return (x != null) ? x : e;
59 
60         // 这里的 else 分支就是上面说的第二种情况，有相应的读或写相匹配的情况
61         } else {                            // complementary-mode
62             QNode m = h.next;               // node to fulfill
63             // 不一致读，表明有其他线程修改了队列
64             if (t != tail || m == null || h != head)
65                 continue;                   // inconsistent read
66 
67             Object x = m.item;
68             if (isData == (x != null) ||    // m already fulfilled
69                 x == m ||                   // m cancelled
70                 !m.casItem(x, e)) {         // lost CAS
71                 advanceHead(h, m);          // dequeue and retry
72                 continue;
73             }
74 
75             advanceHead(h, m);              // successfully fulfilled
76             LockSupport.unpark(m.waiter);
77             return (x != null) ? x : e;
78         }
79     }
80 }
81 
82 void advanceTail(QNode t, QNode nt) {
83     if (tail == t)
84         UNSAFE.compareAndSwapObject(this, tailOffset, t, nt);
85 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

注意44行e为即将要挂起线程的node的值，如果是put，则为其传的值，如果是get，则是null。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 自旋或阻塞，直到满足条件，这个方法返回
 2 Object awaitFulfill(QNode s, Object e, boolean timed, long nanos) {
 3 
 4     long lastTime = timed ? System.nanoTime() : 0;
 5     Thread w = Thread.currentThread();
 6     // 判断需要自旋的次数，
 7     int spins = ((head.next == s) ?
 8                  (timed ? maxTimedSpins : maxUntimedSpins) : 0);
 9     for (;;) {
10         // 如果被中断了，那么取消这个节点
11         if (w.isInterrupted())
12             // 就是将当前节点 s 中的 item 属性设置为 this
13             s.tryCancel(e);
14         Object x = s.item;
15         // 这里是这个方法的唯一的出口
16         if (x != e)
17             return x;
18         // 如果需要，检测是否超时
19         if (timed) {
20             long now = System.nanoTime();
21             nanos -= now - lastTime;
22             lastTime = now;
23             if (nanos <= 0) {
24                 s.tryCancel(e);
25                 continue;
26             }
27         }
28         if (spins > 0)
29             --spins;
30         // 如果自旋达到了最大的次数，那么检测
31         else if (s.waiter == null)
32             s.waiter = w;
33         // 如果自旋到了最大的次数，或者没有设置超时，那么线程挂起，等待唤醒
34         else if (!timed)
35             //挂起当前线程
36             LockSupport.park(this);
37         else if (nanos > spinForTimeoutThreshold)
38             LockSupport.parkNanos(this, nanos);
39     }
40 }
41 
42  void tryCancel(Object cmp) {
43      //将节点item设置为自己，代表此节点取消排队
44      UNSAFE.compareAndSwapObject(this, itemOffset, cmp, this);
45  }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看第14行，上面我们已经说过第44行e为即将要挂起线程的node的值，如果是put，则为其传的值，如果是get，则是null。则如果队列里都是相同的操作类型，会直接挂起，再看上上个方法的第二个else分支，也就是不同的操作类型的时候，会直接获取队列的第一个等待节点，并且第70行，通过cas设置其节点的item为本次操作的值，也就是如果本次操作为put，则队列的节点为take类型，也就是队列节点的item值为null，现在通过cas设置其值，然后再将其唤醒，则awaitFulfill唤醒后，第17行处节点的值已经被cas更改了，自然不能和原始的e相等，就会直接返回给take。

现在我们来按照实际情况来走一遍流程：

　　1、线程1初始化 new SynchronousQueue(true) ，调用 **put(E o)写入值**，我们看 **transfer(Object e, boolean timed, long nanos)** 方法第7行，isData 为true,接在到第20行，因为是刚刚初始化，tail和head都为空节点，36行新建一个节点，38行将当前节点，插入到 tail 的后面,42行将当前节点设置为新的 tail，所以队列中有三个节点，两个是空节点，一个是当前节点，我们再看到 **awaitFulfill(QNode s, Object e, boolean timed, long nanos)** 方法第16行，此时x = e，不返回，36行处挂起当前线程。

　　   此时线程2调用 **take()** 取值，我们看 **transfer(Object e, boolean timed, long nanos)** 方法第7行，isData 为false,接在到第20行，tail和head不相同，之前线程1写入时，QNode 的isData 为true,所以 if (h == t || t.isData == isData) 不满足，进入到**transfer**的62行，取到头节点的后面一个节点，很明显，这个节点的item是**null**,68行处x != null为false, isData == (x != null) 为true,则执行71行将头节点后移一位，相当于去除了头结点，跳出循环继续；这一次循环第62行取到的是线程1中添加的节点，x != null为true,isData == (x != null)为false,x == m 为false,执行 m.casItem(x, e) ，此时e为null,将线程1中添加的节点的item的值设置为null,此时成功，不执行72行处，我们可以看到75行处，将头结点后移一位，相当于线程一put进去的值被移除了，76行处唤醒线程1，刚才说过这次循环62行取到的是线程1中添加的节点，68行处x为线程一中添加的元素，77行 return (x != null) ? x : e; x != null 为true，则retrue x，此时take（）拿到了线程1中添加的元素，并唤醒线程1，将线程一添加的节点去除，take()方法结束。我们再来看看线程1被唤醒后，看 **awaitFulfill 方法中第36行，**被唤醒后接着for循环，第14行获取被挂起之前添加的节点中的item,可是上面讲的线程2中m.casItem(x, e) 已经将此节点的e设置为null ,则17行处retrue null，再到**transfer 方法44行处，X为null,**58行处返回e,put方法结束。

 

　　2、线程1初始化 new SynchronousQueue(true) ，调用 **take() 取值**，我们看 **transfer(Object e, boolean timed, long nanos)** 方法第7行，isData 为false,，因为是刚刚初始化，tail和head都为空节点，36行新建一个节点，38行将当前节点，插入到 tail 的后面,42行将当前节点设置为新的 tail，所以队列中有三个节点，两个是空节点，一个是当前节点，我们再看到 **awaitFulfill(QNode s, Object e, boolean timed, long nanos)** 方法第16行，此时x = e =null，不返回，36行处挂起当前线程。

　　   此时线程2调用 **put(E o)写入值**，我们看 **transfer(Object e, boolean timed, long nanos)** 方法第7行，isData 为true,接在到第20行，tail和head不相同，之前线程1取值时，QNode 的isData 为false,所以 if (h == t || t.isData == isData) 不满足，进入到**transfer**的62行，取到头节点的后面一个节点，很明显，这个节点的item是**null**,68行处x != null为false, isData == (x != null) 为true,则执行71行将头节点后移一位，相当于去除了头结点，跳出循环继续；这一次循环第62行取到的是线程1中等待取值的节点，x != null为false,isData == (x != null)为false,x == m 为false,执行 m.casItem(x, e) ，将线程1中take()的节点的item的值设置为e,此时成功，不执行72行处，我们可以看到75行处，将头结点后移一位，相当于线程1取到值后就将线程从等待队列中移除了，76行处唤醒线程1，刚才说过这次循环62行取到的是线程1中等待取值的节点，68行处x为null，77行 return (x != null) ? x : e; x != null 为false，则 retrue e，此时put（）方法给线程1的QNode设置了item为e，并唤醒线程1，将线程一添加的节点去除，take()方法结束。我们再来看看线程1被唤醒后，看 **awaitFulfill 方法中第36行，**被唤醒后接着for循环，第14行获取被挂起之前添加的节点中的item,可是上面讲的线程2 put() 中m.casItem(x, e) 已经将此节点的e设置为 e ,则17行处retrue 的值为put()的 e ，再到**transfer 方法44行处，X为e,**58行处返回X,也就是put()的e ,take方法结束。

 

我们再来看看offer()和poll()

```
1 public boolean offer(E e) {
2     if (e == null) throw new NullPointerException();
3     return transferer.transfer(e, true, 0) != null;
4 }
```

 

 

 我们可以看出，也是调用了 transfer 方法，如果队列为空了，第一次offer添加元素的话，transfer第32行，timed =true,nanos =0,则此时reture null;

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public E poll(long timeout, TimeUnit unit) throws InterruptedException {
2     E e = transferer.transfer(null, true, unit.toNanos(timeout));
3     if (e != null || !Thread.interrupted())
4         return e;
5     throw new InterruptedException();
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

带超时的poll，如果队列为空，则会添加到等待队列，并且阻塞，经过 timeout 还没有拿到元素，则超时返回false,拿到了元素则返回元素。

由此可见，只有线程在等待着取元素， offer(E e) 才有可能成功，如果没有线程等待着取，则一定会返回失败。

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10238762.html#_labelTop)

## 总结

1、线程做相同类型的操作：

　　　　多个线程 **take()** ，则将线程包装成QNode节点，item为null，将节点添加到队列，将线程挂起；

　　　　多个线程 **put****()** ，将线程包装成QNode节点，item为 e，将节点添加到队列，将线程挂起。

2、线程做不同类型的操作：

　　　　有线程先做了**put****()** ，其他线程做**take()** 操作时，take取到队列中的第一个等待节点中的item，take返回item,并将第一个等待节点唤醒，put返回e；

　　　　有线程先做了**take() ,**其他线程**put****()** 操作时，put将元素e赋值给队列中第一个等待节点的item，put返回e,并将第一个等待节点唤醒，take返回e。

# [并发编程（十一）—— Java 线程池 实现原理与源码深度解析（一）](https://www.cnblogs.com/java-chen-hao/p/10254260.html)



**目录**

- [总览](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_label0)
- [使用示例](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_label1)
- [Executor 接口](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_label2)
- [ExecutorService](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_label3)
- [AbstractExecutorService](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_label4)
- [ThreadPoolExecutor](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_label5)
- [总结](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_label6)

 

**正文**

史上最清晰的线程池源码分析

鼎鼎大名的线程池。不需要多说!!!!!

这篇博客深入分析 Java 中线程池的实现。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_labelTop)

## 总览

下图是 java 线程池几个相关类的继承结构：

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190111114116333-1819080613.png) ![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190111114156817-1284283557.png) ![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190111114207491-413118134.png)

先简单说说这个继承结构，Executor 位于最顶层，也是最简单的，就一个 execute(Runnable runnable) 接口方法定义。

ExecutorService 也是接口，在 Executor 接口的基础上添加了很多的接口方法，所以一般来说我们会使用这个接口。

然后再下来一层是 AbstractExecutorService，从名字我们就知道，这是抽象类，这里实现了非常有用的一些方法供子类直接使用，之后我们再细说。

然后才到我们的重点部分 ThreadPoolExecutor 类，这个类提供了关于线程池所需的非常丰富的功能。

线程池中的 BlockingQueue 也是非常重要的概念，如果线程数达到 corePoolSize，我们的每个任务会提交到等待队列中，等待线程池中的线程来取任务并执行。这里的 BlockingQueue 通常我们使用其实现类 LinkedBlockingQueue、ArrayBlockingQueue 和 SynchronousQueue，每个实现类都有不同的特征，使用场景之后会慢慢分析。想要详细了解各个 BlockingQueue 的读者，可以参考我的前面的一篇对 BlockingQueue 的各个实现类进行详细分析的文章。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_labelTop)

## 使用示例

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 package main.java.Juc;
 2 
 3 import java.util.concurrent.ExecutorService;
 4 import java.util.concurrent.Executors;
 5 
 6 class MyRunnable implements Runnable {
 7     @Override
 8     public void run() {
 9         for (int x = 0; x < 100; x++) {
10             System.out.println(Thread.currentThread().getName() + ":" + x);
11         }
12     }
13 }
14 
15 public class TestThreadPool {
16     public static void main(String[] args) {
17         // 创建一个线程池对象，控制要创建几个线程对象。
18         ExecutorService pool = Executors.newFixedThreadPool(2);
19 
20         // 可以执行Runnable对象或者Callable对象代表的线程
21         pool.execute(new MyRunnable());
22         pool.execute(new MyRunnable());
23 
24         //结束线程池
25         pool.shutdown();
26     }
27 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190111142317937-426750245.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_labelTop)

## Executor 接口

```
1 public interface Executor {
2     void execute(Runnable command);
3 }
```

我们可以看到 Executor 接口非常简单，就一个 `void execute(Runnable command)` 方法，代表提交一个任务。

 当然了，Executor 这个接口只有提交任务的功能，太简单了，我们想要更丰富的功能，比如我们想知道执行结果、我们想知道当前线程池有多少个线程活着、已经完成了多少任务等等，这些都是这个接口的不足的地方。接下来我们要介绍的是继承自 `Executor` 接口的 `ExecutorService` 接口，这个接口提供了比较丰富的功能，也是我们最常使用到的接口。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_labelTop)

## ExecutorService

那么我们简单初略地来看一下这个接口中都有哪些方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public interface ExecutorService extends Executor {
 2 
 3     // 关闭线程池，已提交的任务继续执行，不接受继续提交新任务
 4     void shutdown();
 5 
 6     // 关闭线程池，尝试停止正在执行的所有任务，不接受继续提交新任务
 7     // 它和前面的方法相比，加了一个单词“now”，区别在于它会去停止当前正在进行的任务
 8     List<Runnable> shutdownNow();
 9 
10     // 线程池是否已关闭
11     boolean isShutdown();
12 
13     // 如果调用了 shutdown() 或 shutdownNow() 方法后，所有任务结束了，那么返回true
14     // 这个方法必须在调用shutdown或shutdownNow方法之后调用才会返回true
15     boolean isTerminated();
16 
17     // 等待所有任务完成，并设置超时时间
18     // 我们这么理解，实际应用中是，先调用 shutdown 或 shutdownNow，
19     // 然后再调这个方法等待所有的线程真正地完成，返回值意味着有没有超时
20     boolean awaitTermination(long timeout, TimeUnit unit)
21             throws InterruptedException;
22 
23     // 提交一个 Callable 任务
24     <T> Future<T> submit(Callable<T> task);
25 
26     // 提交一个 Runnable 任务，第二个参数将会放到 Future 中，作为返回值，
27     // 因为 Runnable 的 run 方法本身并不返回任何东西
28     <T> Future<T> submit(Runnable task, T result);
29 
30     // 提交一个 Runnable 任务
31     Future<?> submit(Runnable task);
32     
33     ......
34 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这些方法都很好理解，一个简单的线程池主要就是这些功能，能提交任务，能获取结果，能关闭线程池，这也是为什么我们经常用这个接口的原因。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_labelTop)

## AbstractExecutorService

 

AbstractExecutorService 抽象类派生自 ExecutorService 接口，然后在其基础上实现了几个实用的方法，这些方法提供给子类进行调用。

这个抽象类实现了 ExecutorService 中的 submit 方法，newTaskFor 方法用于将任务包装成 FutureTask。定义于最上层接口 Executor中的 `void execute(Runnable command)` 由于不需要获取结果，不会进行 FutureTask 的包装。

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public abstract class AbstractExecutorService implements ExecutorService {
 2 
 3     // RunnableFuture 是用于获取执行结果的，我们常用它的子类 FutureTask
 4     // 下面两个 newTaskFor 方法用于将我们的任务包装成 FutureTask 提交到线程池中执行
 5     protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
 6         return new FutureTask<T>(runnable, value);
 7     }
 8 
 9     protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
10         return new FutureTask<T>(callable);
11     }
12 
13     // 提交任务
14     public Future<?> submit(Runnable task) {
15         if (task == null) throw new NullPointerException();
16         // 1. 将任务包装成 FutureTask
17         RunnableFuture<Void> ftask = newTaskFor(task, null);
18         // 2. 交给执行器执行，execute 方法由具体的子类来实现
19         // 前面也说了，FutureTask 间接实现了Runnable 接口。
20         execute(ftask);
21         return ftask;
22     }
23 
24     public <T> Future<T> submit(Runnable task, T result) {
25         if (task == null) throw new NullPointerException();
26         // 1. 将任务包装成 FutureTask
27         RunnableFuture<T> ftask = newTaskFor(task, result);
28         // 2. 交给执行器执行
29         execute(ftask);
30         return ftask;
31     }
32 
33     public <T> Future<T> submit(Callable<T> task) {
34         if (task == null) throw new NullPointerException();
35         // 1. 将任务包装成 FutureTask
36         RunnableFuture<T> ftask = newTaskFor(task);
37         // 2. 交给执行器执行
38         execute(ftask);
39         return ftask;
40     }
41 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

到这里，我们发现，这个抽象类包装了一些基本的方法，可是 submit等方法，它们都没有真正开启线程来执行任务，它们都只是在方法内部调用了 execute 方法，所以最重要的 execute(Runnable runnable) 方法还没出现，这里我们要说的就是 ThreadPoolExecutor 类了。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_labelTop)

## ThreadPoolExecutor

我们经常会使用 `Executors` 这个工具类来快速构造一个线程池，对于初学者而言，这种工具类是很有用的，开发者不需要关注太多的细节，只要知道自己需要一个线程池，仅仅提供必需的参数就可以了，其他参数都采用作者提供的默认值。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public static ExecutorService newFixedThreadPool(int nThreads) {
 2     return new ThreadPoolExecutor(nThreads, nThreads,
 3                                   0L, TimeUnit.MILLISECONDS,
 4                                   new LinkedBlockingQueue<Runnable>());
 5 }
 6 public static ExecutorService newCachedThreadPool() {
 7     return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
 8                                   60L, TimeUnit.SECONDS,
 9                                   new SynchronousQueue<Runnable>());
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里先不说有什么区别，它们最终都会导向这个构造方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public ThreadPoolExecutor(int corePoolSize,
 2                           int maximumPoolSize,
 3                           long keepAliveTime,
 4                           TimeUnit unit,
 5                           BlockingQueue<Runnable> workQueue,
 6                           ThreadFactory threadFactory,
 7                           RejectedExecutionHandler handler) {
 8     if (corePoolSize < 0 ||
 9         maximumPoolSize <= 0 ||
10         maximumPoolSize < corePoolSize ||
11         keepAliveTime < 0)
12         throw new IllegalArgumentException();
13     // 这几个参数都是必须要有的
14     if (workQueue == null || threadFactory == null || handler == null)
15         throw new NullPointerException();
16 
17     this.corePoolSize = corePoolSize;
18     this.maximumPoolSize = maximumPoolSize;
19     this.workQueue = workQueue;
20     this.keepAliveTime = unit.toNanos(keepAliveTime);
21     this.threadFactory = threadFactory;
22     this.handler = handler;
23 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面的构造方法中列出了我们最需要关心的几个属性了，下面逐个介绍下构造方法中出现的这几个属性：

- **corePoolSize**

　　　　线程池中的核心线程数。

- **maximumPoolSize**

　　　　最大线程数，线程池允许创建的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；当阻塞队列是无界队列, 则maximumPoolSize则不起作用, 因为无法提交至核心线程池的线程会一直持续地放入workQueue

- **workQueue**

　　　　用来保存等待被执行的任务的阻塞队列. 在JDK中提供了如下阻塞队列：

　　　　(1) ArrayBlockingQueue：基于数组结构的有界阻塞队列，按FIFO排序任务；
　　　　(2) LinkedBlockingQuene：基于链表结构的阻塞队列，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQuene；
　　　　(3) SynchronousQuene：一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene；
　　　　(4) priorityBlockingQuene：具有优先级的无界阻塞队列；

　　　　有兴趣的可以看看我前面关于BlockingQuene的文章

- **keepAliveTime**

　　　　空闲线程的保活时间，如果某线程的空闲时间超过这个值都没有任务给它做，那么可以被关闭了。注意这个值并不会对所有线程起作用，如果线程池中的线程数少于等于核心线程数 corePoolSize，那么这些线程不会因为空闲太长时间而被关闭，当然，也可以通过调用 `allowCoreThreadTimeOut(true)`使核心线程数内的线程也可以被回收；默认情况下，该参数只在线程数大于`corePoolSize`时才有用, 超过这个时间的空闲线程将被终止。

- **unit**

　　　　keepAliveTime的单位

- **threadFactory**

　　　　用于生成线程，一般我们可以用默认的就可以了。通常，我们可以通过它将我们的线程的名字设置得比较可读一些，如 Message-Thread-1， Message-Thread-2 类似这样。

- **handler**

　　　　线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了4种策略： 

　　　　　　AbortPolicy：直接抛出异常，默认策略；
　　　　　　CallerRunsPolicy：用调用者所在的线程来执行任务；
　　　　　　DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务；
　　　　　　DiscardPolicy：直接丢弃任务；
　　　　当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务。


除了上面几个属性外，我们再看看其他重要的属性。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
 2 
 3 // 这里 COUNT_BITS 设置为 29(32-3)，意味着前三位用于存放线程状态，后29位用于存放线程数
 4 private static final int COUNT_BITS = Integer.SIZE - 3;
 5 
 6 // 000 11111111111111111111111111111
 7 // 这里得到的是 29 个 1，也就是说线程池的最大线程数是 2^29-1=536870911
 8 // 以我们现在计算机的实际情况，这个数量还是够用的
 9 private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
10 
11 // 我们说了，线程池的状态存放在高 3 位中
12 // 运算结果为 111跟29个0：111 00000000000000000000000000000
13 private static final int RUNNING    = -1 << COUNT_BITS;
14 // 000 00000000000000000000000000000
15 private static final int SHUTDOWN   =  0 << COUNT_BITS;
16 // 001 00000000000000000000000000000
17 private static final int STOP       =  1 << COUNT_BITS;
18 // 010 00000000000000000000000000000
19 private static final int TIDYING    =  2 << COUNT_BITS;
20 // 011 00000000000000000000000000000
21 private static final int TERMINATED =  3 << COUNT_BITS;
22 
23 // 将整数 c 的低 29 位修改为 0，就得到了线程池的状态
24 private static int runStateOf(int c)     { return c & ~CAPACITY; }
25 // 将整数 c 的高 3 为修改为 0，就得到了线程池中的线程数
26 private static int workerCountOf(int c)  { return c & CAPACITY; }
27 
28 private static int ctlOf(int rs, int wc) { return rs | wc; }
29 
30 private static boolean runStateLessThan(int c, int s) {
31     return c < s;
32 }
33 
34 private static boolean runStateAtLeast(int c, int s) {
35     return c >= s;
36 }
37 
38 private static boolean isRunning(int c) {
39     return c < SHUTDOWN;
40 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在这里，介绍下线程池中的各个状态和状态变化的转换过程：

- RUNNING：这个没什么好说的，这是最正常的状态：接受新的任务，处理等待队列中的任务
- SHUTDOWN：不接受新的任务提交，但是会继续处理等待队列中的任务
- STOP：不接受新的任务提交，不再处理等待队列中的任务，中断正在执行任务的线程
- TIDYING：所有的任务都销毁了，workCount 为 0。线程池的状态在转换为 TIDYING 状态时，会执行钩子方法 terminated()
- TERMINATED：terminated() 方法结束后，线程池的状态就会变成这个

看了这几种状态的介绍，读者大体也可以猜到十之八九的状态转换了，各个状态的转换过程有以下几种：

- RUNNING -> SHUTDOWN：当调用了 shutdown() 后，会发生这个状态转换，这也是最重要的
- (RUNNING or SHUTDOWN) -> STOP：当调用 shutdownNow() 后，会发生这个状态转换，这下要清楚 shutDown() 和 shutDownNow() 的区别了
- SHUTDOWN -> TIDYING：当任务队列和线程池都清空后，会由 SHUTDOWN 转换为 TIDYING
- STOP -> TIDYING：当任务队列清空后，发生这个转换
- TIDYING -> TERMINATED：这个前面说了，当 terminated() 方法结束后

另外，我们还要看看一个内部类 Worker，因为 Doug Lea 把线程池中的线程包装成了一个个 Worker，翻译成工人，就是线程池中做任务的线程。所以到这里，我们知道任务是 Runnable（内部叫 task 或 command），线程是 Worker。

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private final class Worker
 2     extends AbstractQueuedSynchronizer
 3     implements Runnable{
 4     private static final long serialVersionUID = 6138294804551838833L;
 5 
 6     // 这个是真正的线程，任务靠你啦
 7     final Thread thread;
 8 
 9     // 前面说了，这里的 Runnable 是任务。为什么叫 firstTask？因为在创建线程的时候，如果同时指定了
10     // 这个线程起来以后需要执行的第一个任务，那么第一个任务就是存放在这里的(线程可不止执行这一个任务)
11     // 当然了，也可以为 null，这样线程起来了，自己到任务队列（BlockingQueue）中取任务（getTask 方法）就行了
12     Runnable firstTask;
13 
14     // 用于存放此线程完全的任务数，注意了，这里用了 volatile，保证可见性
15     volatile long completedTasks;
16 
17     // Worker 只有这一个构造方法，传入 firstTask，也可以传 null
18     Worker(Runnable firstTask) {
19         setState(-1); // inhibit interrupts until runWorker
20         this.firstTask = firstTask;
21         // 调用 ThreadFactory 来创建一个新的线程,这里创建的线程到时候用来执行任务
22         this.thread = getThreadFactory().newThread(this);
23     }
24 
25     // 这里调用了外部类的 runWorker 方法
26     public void run() {
27         runWorker(this);
28     }
29 
30     ...
31 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

有了上面的这些基础后，我们终于可以看看 ThreadPoolExecutor 的 execute 方法了，前面源码分析的时候也说了，各种方法都最终依赖于 execute 方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void execute(Runnable command) {
 2     if (command == null)
 3         throw new NullPointerException();
 4 
 5     // 前面说的那个表示 "线程池状态" 和 "线程数" 的整数
 6     int c = ctl.get();
 7 
 8     // 如果当前线程数少于核心线程数，那么直接添加一个 worker 来执行任务，
 9     // 创建一个新的线程，并把当前任务 command 作为这个线程的第一个任务(firstTask)
10     if (workerCountOf(c) < corePoolSize) {
11         // 添加任务成功，那么就结束了。提交任务嘛，线程池已经接受了这个任务，这个方法也就可以返回了
12         // 至于执行的结果，到时候会包装到 FutureTask 中。
13         // 这里的true代表当前线程数小于corePoolSize，表示以corePoolSize为线程数界限
14         if (addWorker(command, true))
15             return;
16         c = ctl.get();
17     }
18     // 到这里说明，要么当前线程数大于等于核心线程数，要么刚刚 addWorker 失败了
19     // 如果线程池处于 RUNNING 状态，把这个任务添加到任务队列 workQueue 中
20     if (isRunning(c) && workQueue.offer(command)) {
21         int recheck = ctl.get();
22         // 如果线程池已不处于 RUNNING 状态，那么移除已经入队的这个任务，并且执行拒绝策略
23         if (! isRunning(recheck) && remove(command))
24             reject(command);
25         else if (workerCountOf(recheck) == 0)
26             addWorker(null, false);
27     }
28     // 如果 workQueue 队列满了，那么进入到这个分支
29     // 这里的false代表当前线程数大于corePoolSize，表示以 maximumPoolSize 为界创建新的 worker
30     // 如果失败，说明当前线程数已经达到 maximumPoolSize，执行拒绝策略
31     else if (!addWorker(command, false))
32         reject(command);
33 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 我们可以看看大体的执行流程

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190111155038544-483389124.png)

 

这个方法非常重要 addWorker(Runnable firstTask, boolean core) 方法，我们看看它是怎么创建新的线程的：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 第一个参数是准备提交给这个线程执行的任务，之前说了，可以为 null
 2 // 第二个参数为 true 代表使用核心线程数 corePoolSize 作为创建线程的界线，也就说创建这个线程的时候，
 3 //         如果线程池中的线程总数已经达到 corePoolSize，那么返回false
 4 //         如果是 false，代表使用最大线程数 maximumPoolSize 作为界线，线程池中的线程总数已经达到 maximumPoolSize，那么返回false
 5 private boolean addWorker(Runnable firstTask, boolean core) {
 6     retry:
 7     for (;;) {
 8         int c = ctl.get();
 9         int rs = runStateOf(c);
10 
11         // 如果线程池已关闭，并满足以下条件之一，那么不创建新的 worker：
12         // 1. 线程池状态大于 SHUTDOWN，其实也就是 STOP, TIDYING, 或 TERMINATED
13         // 2. firstTask != null
14         // 3. workQueue.isEmpty()
15         if (rs >= SHUTDOWN &&
16             ! (rs == SHUTDOWN &&
17                firstTask == null &&
18                ! workQueue.isEmpty()))
19             return false;
20 
21         for (;;) {
22             int wc = workerCountOf(c);
23             //这里就是通过core参数对当前线程数的判断
24             if (wc >= CAPACITY ||
25                 wc >= (core ? corePoolSize : maximumPoolSize))
26                 return false;
27             if (compareAndIncrementWorkerCount(c))
28                 break retry;
29             c = ctl.get();
30             if (runStateOf(c) != rs)
31                 continue retry;
32             // else CAS failed due to workerCount change; retry inner loop
33         }
34     }
35 
36     /* 
37      * 到这里，我们认为在当前这个时刻，可以开始创建线程来执行任务了，
38      */
39 
40     // worker 是否已经启动
41     boolean workerStarted = false;
42     // 是否已将这个 worker 添加到 workers 这个 HashSet 中
43     boolean workerAdded = false;
44     Worker w = null;
45     try {
46         final ReentrantLock mainLock = this.mainLock;
47         // 把 firstTask 传给 worker 的构造方法
48         w = new Worker(firstTask);
49         // 取 worker 中的线程对象，之前说了，Worker的构造方法会调用 ThreadFactory 来创建一个新的线程
50         final Thread t = w.thread;
51         if (t != null) {
52             // 这个是整个类的全局锁，因为关闭一个线程池需要这个锁，至少我持有锁的期间，线程池不会被关闭
53             mainLock.lock();
54             try {
55 
56                 int c = ctl.get();
57                 int rs = runStateOf(c);
58 
59                 // 小于 SHUTTDOWN 那就是 RUNNING
60                 // 如果等于 SHUTDOWN，前面说了，不接受新的任务，但是会继续执行等待队列中的任务
61                 if (rs < SHUTDOWN ||
62                     (rs == SHUTDOWN && firstTask == null)) {
63                     // worker 里面的 thread 可不能是已经启动的
64                     if (t.isAlive())
65                         throw new IllegalThreadStateException();
66                     // 加到 workers 这个 HashSet 中
67                     workers.add(w);
68                     int s = workers.size();
69                     // largestPoolSize 用于记录 workers 中的个数的最大值
70                     // 因为 workers 是不断增加减少的，通过这个值可以知道线程池的大小曾经达到的最大值
71                     if (s > largestPoolSize)
72                         largestPoolSize = s;
73                     workerAdded = true;
74                 }
75             } finally {
76                 mainLock.unlock();
77             }
78             // 添加成功的话，启动这个线程
79             if (workerAdded) {
80                 // 启动线程，最重要的就是这里，下面我们会讲解如何执行任务
81                 t.start();
82                 workerStarted = true;
83             }
84         }
85     } finally {
86         // 如果线程没有启动，需要做一些清理工作，如前面 workCount 加了 1，将其减掉
87         if (! workerStarted)
88             addWorkerFailed(w);
89     }
90     // 返回线程是否启动成功
91     return workerStarted;
92 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面第81行代码处已经启动了线程，w = new Worker(firstTask); t = w.thread，我们接着看看Worker这个类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private final class Worker
 2     extends AbstractQueuedSynchronizer
 3     implements Runnable{
 4     private static final long serialVersionUID = 6138294804551838833L;
 5     final Thread thread;
 6     Runnable firstTask;
 7     volatile long completedTasks;
 8 
 9     // Worker 只有这一个构造方法，传入 firstTask
10     Worker(Runnable firstTask) {
11         setState(-1); // inhibit interrupts until runWorker
12         this.firstTask = firstTask;
13         // 调用 ThreadFactory 来创建一个新的线程,这里创建的线程到时候用来执行任务
14         // 我们发现创建线程的时候传入的值是this，我们知道创建线程可以通过继承Runnable的方法，
15         // Worker继承了Runnable，并且下面重写了run()方法
16         this.thread = getThreadFactory().newThread(this);
17     }
18 
19     // 由上面创建线程时传入的this,上面的thread启动后，会执行这里的run()方法，并且此时runWorker传入的也是this
20     public void run() {
21         runWorker(this);
22     }
23 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

继续往下看 runWorker 方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 此方法由 worker 线程启动后调用，这里用一个 while 循环来不断地从等待队列中获取任务并执行
 2 // 前面说了，worker 在初始化的时候，可以指定 firstTask，那么第一个任务也就可以不需要从队列中获取
 3 final void runWorker(Worker w) {
 4     Thread wt = Thread.currentThread();
 5     // 该线程的第一个任务(如果有的话)
 6     Runnable task = w.firstTask;
 7     w.firstTask = null;
 8     w.unlock(); // allow interrupts
 9     boolean completedAbruptly = true;
10     try {
11         // 循环调用 getTask 获取任务
12         while (task != null || (task = getTask()) != null) {
13             w.lock();          
14             // 如果线程池状态大于等于 STOP，那么意味着该线程也要中断
15             if ((runStateAtLeast(ctl.get(), STOP) ||
16                  (Thread.interrupted() &&
17                   runStateAtLeast(ctl.get(), STOP))) &&
18                 !wt.isInterrupted())
19                 wt.interrupt();
20             try {
21                 beforeExecute(wt, task);
22                 Throwable thrown = null;
23                 try {
24                     // 到这里终于可以执行任务了，这里是最重要的，task是什么？是Worker 中的firstTask属性
25                     // 也就是上面我们使用示例里面的 new MyRunnable()实例，这里就是真正的执行run方法里面的代码
26                     task.run();
27                 } catch (RuntimeException x) {
28                     thrown = x; throw x;
29                 } catch (Error x) {
30                     thrown = x; throw x;
31                 } catch (Throwable x) {
32                     thrown = x; throw new Error(x);
33                 } finally {
34                     afterExecute(task, thrown);
35                 }
36             } finally {
37                 // 一个任务执行完了，这个线程还可以复用，接着去队列中拉取任务执行
38                 // 置空 task，准备 getTask 获取下一个任务
39                 task = null;
40                 // 累加完成的任务数
41                 w.completedTasks++;
42                 // 释放掉 worker 的独占锁
43                 w.unlock();
44             }
45         }
46         completedAbruptly = false;
47     } finally {
48         // 如果到这里，需要执行线程关闭：
49         // 说明 getTask 返回 null，也就是超过corePoolSize的线程过了超时时间还没有获取到任务，也就是说，这个 worker 的使命结束了，执行关闭
50         processWorkerExit(w, completedAbruptly);
51     }
52 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们看看 getTask() 是怎么获取任务的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 此方法有三种可能：
 2 // 1. 阻塞直到获取到任务返回。我们知道，默认 corePoolSize 之内的线程是不会被回收的，
 3 //      它们会一直等待任务
 4 // 2. 超时退出。keepAliveTime 起作用的时候，也就是如果这么多时间内都没有任务，那么应该执行关闭
 5 // 3. 如果发生了以下条件，此方法必须返回 null:
 6 //    - 池中有大于 maximumPoolSize 个 workers 存在(通过调用 setMaximumPoolSize 进行设置)
 7 //    - 线程池处于 SHUTDOWN，而且 workQueue 是空的，前面说了，这种不再接受新的任务
 8 //    - 线程池处于 STOP，不仅不接受新的线程，连 workQueue 中的线程也不再执行
 9 private Runnable getTask() {
10     boolean timedOut = false; // Did the last poll() time out?
11 
12     retry:
13     for (;;) {
14         int c = ctl.get();
15         int rs = runStateOf(c);
16         // 两种可能
17         // 1. rs == SHUTDOWN && workQueue.isEmpty()
18         // 2. rs >= STOP
19         if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
20             // CAS 操作，减少工作线程数
21             decrementWorkerCount();
22             return null;
23         }
24 
25         boolean timed;      // Are workers subject to culling?
26         for (;;) {
27             int wc = workerCountOf(c);
28             // 允许核心线程数内的线程回收，或当前线程数超过了核心线程数，那么有可能发生超时关闭
29             timed = allowCoreThreadTimeOut || wc > corePoolSize;
30             if (wc <= maximumPoolSize && ! (timedOut && timed))
31                 break;
32             if (compareAndDecrementWorkerCount(c))
33                 return null;
34             c = ctl.get();  // Re-read ctl
35             // compareAndDecrementWorkerCount(c) 失败，线程池中的线程数发生了改变
36             if (runStateOf(c) != rs)
37                 continue retry;
38             // else CAS failed due to workerCount change; retry inner loop
39         }
40         // wc <= maximumPoolSize 同时没有超时
41         try {
42             // 到 workQueue 中获取任务
43             // 如果timed=wc > corePoolSize=false,我们知道核心线程数之内的线程永远不会销毁，则执行workQueue.take();我前面文章中讲过，take()方法是阻塞方法，如果队里中有任务则取到任务，如果没有任务，则一直阻塞在这里知道有任务被唤醒。
44             //如果timed=wc > corePoolSize=true,这里将执行超时策略，poll(keepAliveTime, TimeUnit.NANOSECONDS)会阻塞keepAliveTime这么长时间，没超时就返回任务，超时则返回null.
45             Runnable r = timed ?
46                 workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
47                 workQueue.take();
48             if (r != null)
49                 return r;
50             timedOut = true;
51         } catch (InterruptedException retry) {
52             // 如果此 worker 发生了中断，采取的方案是重试
53             // 解释下为什么会发生中断，这个读者要去看 setMaximumPoolSize 方法，
54             // 如果开发者将 maximumPoolSize 调小了，导致其小于当前的 workers 数量，
55             // 那么意味着超出的部分线程要被关闭。重新进入 for 循环，自然会有部分线程会返回 null
56             timedOut = false;
57         }
58     }
59 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

到这里，基本上也说完了整个流程，读者这个时候应该回到 execute(Runnable command) 方法，有两种情况会调用 reject(command) 来处理任务，因为按照正常的流程，线程池此时不能接受这个任务，所以需要执行我们的拒绝策略。接下来，我们说一说 ThreadPoolExecutor 中的拒绝策略。

```
1 final void reject(Runnable command) {
2     // 执行拒绝策略
3     handler.rejectedExecution(command, this);
4 }
```

此处的 handler 我们需要在构造线程池的时候就传入这个参数，它是 RejectedExecutionHandler 的实例。

RejectedExecutionHandler 在 ThreadPoolExecutor 中有四个已经定义好的实现类可供我们直接使用，当然，我们也可以实现自己的策略，不过一般也没有必要。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 只要线程池没有被关闭，那么由提交任务的线程自己来执行这个任务。
 2 public static class CallerRunsPolicy implements RejectedExecutionHandler {
 3     public CallerRunsPolicy() { }
 4     public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
 5         if (!e.isShutdown()) {
 6             r.run();
 7         }
 8     }
 9 }
10 
11 // 不管怎样，直接抛出 RejectedExecutionException 异常
12 // 这个是默认的策略，如果我们构造线程池的时候不传相应的 handler 的话，那就会指定使用这个
13 public static class AbortPolicy implements RejectedExecutionHandler {
14     public AbortPolicy() { }
15     public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
16         throw new RejectedExecutionException("Task " + r.toString() +
17                                              " rejected from " +
18                                              e.toString());
19     }
20 }
21 
22 // 不做任何处理，直接忽略掉这个任务
23 public static class DiscardPolicy implements RejectedExecutionHandler {
24     public DiscardPolicy() { }
25     public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
26     }
27 }
28 
29 // 这个相对霸道一点，如果线程池没有被关闭的话，
30 // 把队列队头的任务(也就是等待了最长时间的)直接扔掉，然后提交这个任务到等待队列中
31 public static class DiscardOldestPolicy implements RejectedExecutionHandler {
32     public DiscardOldestPolicy() { }
33     public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
34         if (!e.isShutdown()) {
35             e.getQueue().poll();
36             e.execute(r);
37         }
38     }
39 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

到这里，ThreadPoolExecutor 算是分析得差不多了

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10254260.html#_labelTop)

## 总结

我们简单回顾下线程创建的流程

1. 如果当前线程数少于 corePoolSize，那么提交任务的时候创建一个新的线程，并由这个线程执行这个任务；
2. 如果当前线程数已经达到 corePoolSize，那么将提交的任务添加到队列中，等待线程池中的线程去队列中取任务；
3. 如果队列已满，那么创建新的线程来执行任务，需要保证池中的线程数不会超过 maximumPoolSize，如果此时线程数超过了 maximumPoolSize，那么执行拒绝策略。

# [并发编程（十二）—— Java 线程池 实现原理与源码深度解析 之 submit 方法 （二）](https://www.cnblogs.com/java-chen-hao/p/10255981.html)



**目录**

- [AbstractExecutorService](https://www.cnblogs.com/java-chen-hao/p/10255981.html#_label0)
- 使用示例
  - [submit(Callable task)](https://www.cnblogs.com/java-chen-hao/p/10255981.html#_label1_0)
  - [submit(Runnable task, T result)](https://www.cnblogs.com/java-chen-hao/p/10255981.html#_label1_1)
  - [submit(Runnable task)](https://www.cnblogs.com/java-chen-hao/p/10255981.html#_label1_2)
- 源码分析
  - [submit(Callable task)](https://www.cnblogs.com/java-chen-hao/p/10255981.html#_label2_0)
  - [submit(Runnable task, T result)](https://www.cnblogs.com/java-chen-hao/p/10255981.html#_label2_1)

 

**正文**

在上一篇[《并发编程（十一）—— Java 线程池 实现原理与源码深度解析（一）》](https://www.cnblogs.com/java-chen-hao/p/10254260.html)中提到了线程池ThreadPoolExecutor的原理以及它的execute方法。这篇文章是接着上一篇文章写的，如果你没有阅读上一篇文章，建议你去读读。本文解析ThreadPoolExecutor#submit。

　　对于一个任务的执行有时我们不需要它返回结果，但是有我们需要它的返回执行结果。对于线程来讲，如果不需要它返回结果则实现Runnable，而如果需要执行结果的话则可以实现Callable。在线程池同样execute提供一个不需要返回结果的任务执行，而对于需要结果返回的则可调用其submit方法。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10255981.html#_labelTop)

## AbstractExecutorService

我们把上一篇文章的代码贴过来

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public abstract class AbstractExecutorService implements ExecutorService {
 2 
 3     // RunnableFuture 是用于获取执行结果的，我们常用它的子类 FutureTask
 4     // 下面两个 newTaskFor 方法用于将我们的任务包装成 FutureTask 提交到线程池中执行
 5     protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
 6         return new FutureTask<T>(runnable, value);
 7     }
 8 
 9     protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
10         return new FutureTask<T>(callable);
11     }
12 
13     // 提交任务
14     public Future<?> submit(Runnable task) {
15         if (task == null) throw new NullPointerException();
16         // 1. 将任务包装成 FutureTask
17         RunnableFuture<Void> ftask = newTaskFor(task, null);
18         // 2. 交给执行器执行，execute 方法由具体的子类来实现
19         // 前面也说了，FutureTask 间接实现了Runnable 接口。
20         execute(ftask);
21         return ftask;
22     }
23 
24     public <T> Future<T> submit(Runnable task, T result) {
25         if (task == null) throw new NullPointerException();
26         // 1. 将任务包装成 FutureTask
27         RunnableFuture<T> ftask = newTaskFor(task, result);
28         // 2. 交给执行器执行
29         execute(ftask);
30         return ftask;
31     }
32 
33     public <T> Future<T> submit(Callable<T> task) {
34         if (task == null) throw new NullPointerException();
35         // 1. 将任务包装成 FutureTask
36         RunnableFuture<T> ftask = newTaskFor(task);
37         // 2. 交给执行器执行
38         execute(ftask);
39         return ftask;
40     }
41 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

尽管submit方法能提供线程执行的返回值，但只有实现了Callable才会有返回值，而实现Runnable的线程则是没有返回值的，也就是说在上面的3个方法中，submit(Callable<T> task)能获取到它的返回值，submit(Runnable task, T result)能通过传入的载体result间接获得线程的返回值或者准确来说交给线程处理一下，而最后一个方法submit(Runnable task)则是没有返回值的，就算获取它的返回值也是null。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10255981.html#_labelTop)

## 使用示例



### **submit(Callable<T> task)**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /**
 2  * @author: ChenHao
 3  * @Date: Created in 14:54 2019/1/11
 4  */
 5 public class Test {
 6     public static void main(String[] args) throws ExecutionException, InterruptedException {
 7         Callable<String> callable = new Callable<String>() {
 8             public String call() throws Exception {
 9                 System.out.println("This is ThreadPoolExetor#submit(Callable<T> task) method.");
10                 return "result";
11             }
12         };
13 
14         ExecutorService executor = Executors.newSingleThreadExecutor();
15         Future<String> future = executor.submit(callable);
16         executor.shutdown();
17         System.out.println(future.get());
18     }
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190111165152052-1687216141.png)



### **submit(Runnable task, T result)**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /**
 2  * @author: ChenHao
 3  * @Date: Created in 14:54 2019/1/11
 4  */
 5 public class Test {
 6     public static void main(String[] args) throws ExecutionException, InterruptedException {
 7 
 8         ExecutorService executor = Executors.newSingleThreadExecutor();
 9         Data data = new Data();
10         Future<Data> future = executor.submit(new Task(data), data);
11         executor.shutdown();
12         System.out.println(future.get().getName());
13     }
14 }
15 class Data {
16     String name;
17     public String getName() {
18         return name;
19     }
20     public void setName(String name) {
21         this.name = name;
22     }
23 }
24 
25 class Task implements Runnable {
26     Data data;
27     public Task(Data data) {
28         this.data = data;
29     }
30     @Override
31     public void run() {
32         System.out.println("This is ThreadPoolExetor#submit(Runnable task, T result) method.");
33         data.setName("陈浩");
34     }
35 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 运行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190111170053931-493110054.png)



### **submit(Runnable task)**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /**
 2  * @author: ChenHao
 3  * @Date: Created in 14:54 2019/1/11
 4  */
 5 public class Test {
 6     public static void main(String[] args) throws ExecutionException, InterruptedException {
 7         Runnable runnable = new Runnable() {
 8             @Override
 9             public void run() {
10                 System.out.println("This is ThreadPoolExetor#submit(Runnable runnable) method.");
11             }
12         };
13 
14         ExecutorService executor = Executors.newSingleThreadExecutor();
15         Future future = executor.submit(runnable);
16         executor.shutdown();
17         System.out.println(future.get());
18     }
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190111170317661-1861930683.png)

 

从上面的源码可以看到，这三者方法几乎是一样的，关键就在于：

```
1 RunnableFuture<T> ftask = newTaskFor(task);
2 execute(ftask);
```

是如何将一个任务作为参数传递给了newTaskFor，然后调用execute方法，最后进而返回ftask的呢？

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
2     return new FutureTask<T>(runnable, value);
3 }
4 
5 protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
6     return new FutureTask<T>(callable);
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10255981.html#_labelTop)

## 源码分析

这里我建议大家去看看我之前的一篇文章《[Java 多线程（五）—— 线程池基础 之 FutureTask源码解析](https://www.cnblogs.com/java-chen-hao/p/10243509.html)》



### submit(Callable<T> task)

我们看上面的源码中知道实际上是调用了如下代码

```
1 protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
2     return new FutureTask<T>(callable);
3 }
```

 

 我们看看 FutureTask 的结构

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class FutureTask<V> implements RunnableFuture<V> { 
 2     private volatile int state; 
 3     private static final int NEW = 0; //初始状态 
 4     private static final int COMPLETING = 1; //结果计算完成或响应中断到赋值给返回值之间的状态。 
 5     private static final int NORMAL = 2; //任务正常完成，结果被set 
 6     private static final int EXCEPTIONAL = 3; //任务抛出异常 
 7     private static final int CANCELLED = 4; //任务已被取消 
 8     private static final int INTERRUPTING = 5; //线程中断状态被设置ture，但线程未响应中断 
 9     private static final int INTERRUPTED = 6; //线程已被中断 
10 
11     //将要执行的任务 
12     private Callable<V> callable; //用于get()返回的结果，也可能是用于get()方法抛出的异常 
13     private Object outcome; // non-volatile, protected by state reads/writes //执行callable的线程，调用FutureTask.run()方法通过CAS设置 
14     private volatile Thread runner; //栈结构的等待队列，该节点是栈中的最顶层节点。 
15     private volatile WaitNode waiters; 
16 
17     public FutureTask(Callable<V> callable) {
18         if (callable == null)
19             throw new NullPointerException();
20         this.callable = callable;
21         this.state = NEW;       // ensure visibility of callable
22     }
23     ....
24 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public interface RunnableFuture<V> extends Runnable, Future<V> {
2     /**
3      * Sets this Future to the result of its computation
4      * unless it has been cancelled.
5      */
6     void run();
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

 我们知道 FutureTask 继承了 Runnable，这里将 Callable<T> callable 的实例封装成 FutureTask 传给 execute(ftask);我们再来看看上一篇文章中线程运行的代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 此方法由 worker 线程启动后调用，这里用一个 while 循环来不断地从等待队列中获取任务并执行
 2 // 前面说了，worker 在初始化的时候，可以指定 firstTask，那么第一个任务也就可以不需要从队列中获取
 3 final void runWorker(Worker w) {
 4     Thread wt = Thread.currentThread();
 5     // 该线程的第一个任务(如果有的话)
 6     Runnable task = w.firstTask;
 7     w.firstTask = null;
 8     w.unlock(); // allow interrupts
 9     boolean completedAbruptly = true;
10     try {
11         // 循环调用 getTask 获取任务
12         while (task != null || (task = getTask()) != null) {
13             w.lock();          
14             // 如果线程池状态大于等于 STOP，那么意味着该线程也要中断
15             if ((runStateAtLeast(ctl.get(), STOP) ||
16                  (Thread.interrupted() &&
17                   runStateAtLeast(ctl.get(), STOP))) &&
18                 !wt.isInterrupted())
19                 wt.interrupt();
20             try {
21                 beforeExecute(wt, task);
22                 Throwable thrown = null;
23                 try {
24                     // 到这里终于可以执行任务了，这里是最重要的，task是什么？是Worker 中的firstTask属性
25                     
26                     task.run();
27                 } catch (RuntimeException x) {
28                     thrown = x; throw x;
29                 } catch (Error x) {
30                     thrown = x; throw x;
31                 } catch (Throwable x) {
32                     thrown = x; throw new Error(x);
33                 } finally {
34                     afterExecute(task, thrown);
35                 }
36             } finally {
37                 // 一个任务执行完了，这个线程还可以复用，接着去队列中拉取任务执行
38                 // 置空 task，准备 getTask 获取下一个任务
39                 task = null;
40                 // 累加完成的任务数
41                 w.completedTasks++;
42                 // 释放掉 worker 的独占锁
43                 w.unlock();
44             }
45         }
46         completedAbruptly = false;
47     } finally {
48         // 如果到这里，需要执行线程关闭：
49         // 说明 getTask 返回 null，也就是超过corePoolSize的线程过了超时时间还没有获取到任务，也就是说，这个 worker 的使命结束了，执行关闭
50         processWorkerExit(w, completedAbruptly);
51     }
52 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 由上面第6行代码 task 就是execute(ftask)传入的任务，第26行 task.run(); 实际上就是 new FutureTask<T>(callable).run(),我们看看FutureTask中的run()方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void run() {
 2     //保证callable任务只被运行一次
 3     if (state != NEW || !UNSAFE.compareAndSwapObject(this, runnerOffset, null, Thread.currentThread()))
 4         return;
 5     try {
 6         Callable < V > c = callable;
 7         if (c != null && state == NEW) {
 8             V result;
 9             boolean ran;
10             try { 
11                 //执行任务，上面的例子我们可以看出，call()里面可能是一个耗时的操作，不过这里是同步的
12                 result = c.call();
13                 //上面的call()是同步的，只有上面的result有了结果才会继续执行
14                 ran = true;
15             } catch (Throwable ex) {
16                 result = null;
17                 ran = false;
18                 setException(ex);
19             }
20             if (ran)
21                 //执行完了，设置result
22                 set(result);
23         }
24     }
25     finally {
26         runner = null;
27         int s = state;
28         //判断该任务是否正在响应中断，如果中断没有完成，则等待中断操作完成
29         if (s >= INTERRUPTING)
30             handlePossibleCancellationInterrupt(s);
31     }
32 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

在 FutureTask的构造方法中 this.callable = callable; ，因此我们可以知道上面run()方法中第6行 c 就是 代码示例中的 new Callable<String>()，c.call()就是调用 代码示例中new Callable 的call方法，并且这里可以取到返回结果，第22行处设置FutureTask 中 outcome 的值，这样线程就可以取到返回值了。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 protected void set(V v) {
2     if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
3         outcome = v;
4         UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
5         finishCompletion();
6     }
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

取值我就不分析了，我之前的文章里面有详细分析。



### **submit(Runnable task, T result)**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public <T> Future<T> submit(Runnable task, T result) {
 2     if (task == null) throw new NullPointerException();
 3     RunnableFuture<T> ftask = newTaskFor(task, result);
 4     execute(ftask);
 5     return ftask;
 6 }
 7 
 8 protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
 9     return new FutureTask<T>(runnable, value);
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

我们来看看FutureTask的另外一个构造方法

```
1 public FutureTask(Runnable runnable, V result) {
2     this.callable = Executors.callable(runnable, result);
3     this.state = NEW;       // ensure visibility of callable
4 }
```

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public static <T> Callable<T> callable(Runnable task, T result) {
 2     if (task == null)
 3         throw new NullPointerException();
 4     return new RunnableAdapter<T>(task, result);
 5 }
 6 
 7 static final class RunnableAdapter<T> implements Callable<T> {
 8     final Runnable task;
 9     final T result;
10     RunnableAdapter(Runnable task, T result) {
11         this.task = task;
12         this.result = result;
13     }
14     public T call() {
15         task.run();
16         return result;
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

上面将 runnable, result 封装成了 RunnableAdapter 作为FutureTask的callable属性，这和上面的submit(Callable<T> task) 是不同的，submit(Callable<T> task)是直接将 Callable<T> task作为FutureTask的callable属性。我们看看FutureTask中的run()方法中第6行 c 就是FutureTask 构造方法中的new RunnableAdapter<T>(task, result) ，c.call()就是调用 RunnableAdapter<T>(task, result) 的call方法，call()中的task.run()就是上面代码示例中new Task(data) 中的 run()，run()方法中业务大代码改变了data对象的属性，callable(Runnable task, T result)中也是传的相同的对象data, 所以，result = c.call(); 就是把更改后的data返回，并且将data设置为设置FutureTask 中 outcome 的值，后面的逻辑就是一样的了。

这里可以看成将同一个data传入线程进行处理，同时这个data也传入FutureTask中，并且在RunnableAdapter通过属性进行保存data,等线程将data处理完了，由于是同一个对象，RunnableAdapter中的result也就是data指向的是同一个对象，然后把此result返回到FutureTask保存在属性outcome中，就可以通过FutureTask.get()取到运行结果了。

如果new FutureTask<T>(runnable, null),则result = c.call(); 返回的值也是null,最后从线程池中get的值也是null。

# [并发编程（十三）—— Java 线程池 实现原理与源码深度解析 之 Executors（三）](https://www.cnblogs.com/java-chen-hao/p/10265938.html)



**目录**

- [newFixedThreadPool](https://www.cnblogs.com/java-chen-hao/p/10265938.html#_label0)
- [newSingleThreadExecutor](https://www.cnblogs.com/java-chen-hao/p/10265938.html#_label1)
- [newCachedThreadPool](https://www.cnblogs.com/java-chen-hao/p/10265938.html#_label2)

 

**正文**

前两篇文章讲了线程池的源码分析，再来看这篇文章就比较简单了， 本文主要讲解 Executors 这个工具类，看看长江创建线程池的几种方法。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10265938.html#_labelTop)

## newFixedThreadPool

- 生成一个固定大小的线程池：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public static ExecutorService newFixedThreadPool(int nThreads) {
2     return new ThreadPoolExecutor(nThreads, nThreads,
3                                   0L, TimeUnit.MILLISECONDS,
4                                   new LinkedBlockingQueue<Runnable>());
5 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

最大线程数设置为与核心线程数相等，则不会创建临时线程，创建的线程都是核心线程，线程也不会被回收。此时 keepAliveTime 设置为 0（因为这里它是没用的，即使不为 0，线程池默认也不会回收 corePoolSize 内的线程），任务队列采用 LinkedBlockingQueue，无界队列，所以FixedThreadPool永远不会拒绝, 即饱和策略失效。

过程分析：刚开始，每提交一个任务都创建一个 worker，当 worker 的数量达到 nThreads 后，不再创建新的线程，而是把任务提交到 LinkedBlockingQueue 中，而且之后线程数始终为 nThreads。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10265938.html#_labelTop)

## newSingleThreadExecutor

- 生成只有一个线程的固定线程池

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public static ExecutorService newSingleThreadExecutor() {
2     return new FinalizableDelegatedExecutorService
3         (new ThreadPoolExecutor(1, 1,
4                                 0L, TimeUnit.MILLISECONDS,
5                                 new LinkedBlockingQueue<Runnable>()));
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个更简单，和上面的一样，只要设置线程数为 1 就可以了。

初始化的线程池中只有一个线程，如果该线程异常结束，会重新创建一个新的线程继续执行任务，唯一的线程可以保证所提交任务的顺序执行。

由于使用了无界队列, 所以SingleThreadPool永远不会拒绝, 即饱和策略失效。

由于newFixedThreadPool和SingleThreadPool都是使用的LinkedBlockingQueue，并且核心线程固定，如果此时并发有大量任务进行添加，线程处理速度过慢，将会全部添加到LinkedBlockingQueue中，此时会出现内存溢出。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10265938.html#_labelTop)

## newCachedThreadPool

- 生成一个需要的时候就创建新的线程

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

核心线程数为 0，最大线程数为 Integer.MAX_VALUE，keepAliveTime 为 60 秒，任务队列采用 SynchronousQueue，所以创建的线程都是临时线程，都可以被回收。

这种线程池对于任务可以比较快速地完成的情况有比较好的性能。如果线程空闲了 60 秒都没有任务，那么将关闭此线程并从线程池中移除。所以如果线程池空闲了很长时间也不会有问题，因为随着所有的线程都会被关闭，整个线程池不会占用任何的系统资源。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 int c = ctl.get();
 2 // corePoolSize 为 0，所以不会进到这个 if 分支
 3 if (workerCountOf(c) < corePoolSize) {
 4     if (addWorker(command, true))
 5         return;
 6     c = ctl.get();
 7 }
 8 // offer 如果有空闲线程刚好可以接收此任务，那么返回 true，否则返回 false
 9 if (isRunning(c) && workQueue.offer(command)) {
10     int recheck = ctl.get();
11     if (! isRunning(recheck) && remove(command))
12         reject(command);
13     else if (workerCountOf(recheck) == 0)
14         addWorker(null, false);
15 }
16 else if (!addWorker(command, false))
17     reject(command);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

过程分析：我把 execute 方法的主体粘贴过来，让大家看得明白些。鉴于 corePoolSize 是 0，那么提交任务的时候，直接将任务提交到队列中，由于采用了 SynchronousQueue，所以如果是第一个任务提交的时候，offer 方法肯定会返回 false，因为此时没有任何 worker 对这个任务进行接收，那么将进入到最后一个分支来创建第一个 worker，第一个worker执行完后就getTask（）从队列中取任务。之后再提交任务的话，取决于是否有空闲下来的线程对任务进行接收，如果有，会进入到第二个 if 语句块中把当前任务给正在等待的worker，如果没有空闲的线程在等待取任务，就是和第一个任务一样，进到最后的 else if 分支创建worker。

我们来仔细分析下代码，第一次添加任务时，执行到第9行 workQueue.offer(command)，我把以前文章里面的offer()代码贴过来，如果有感兴趣的可以去看看《[并发编程（十）—— Java 并发队列 BlockingQueue 实现之 SynchronousQueue源码分析](https://www.cnblogs.com/java-chen-hao/p/10238762.html)》

```
1 public boolean offer(E e) {
2     if (e == null) throw new NullPointerException();
3     return transferer.transfer(e, true, 0) != null;
4 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /**
 2  * Puts or takes an item.
 3  */
 4 Object transfer(Object e, boolean timed, long nanos) {
 5 
 6     QNode s = null; // constructed/reused as needed
 7     boolean isData = (e != null);
 8 
 9     for (;;) {
10         QNode t = tail;
11         QNode h = head;
12         if (t == null || h == null)         // saw uninitialized value
13             //说明还没有初始化，则跳出继续循环，直至初始化完成
14             continue;                       // spin
15 
16         // 走到这里，说明已经初始化完成，但是初始化时head = h;tail = h;head和tail都是相同的空节点
17         // 如果h == t为false，则判断t.isData == isData，判断队尾节点和当前节点类型是否一致
18         // 队列空，或队列中节点类型和当前节点一致，
19         // 即我们说的第一种情况，将节点入队即可。读者要想着这块 if 里面方法其实就是入队
20         if (h == t || t.isData == isData) { // empty or same-mode
21             QNode tn = t.next;
22             // t != tail 说明刚刚有节点入队，continue 即可
23             if (t != tail)                  // inconsistent read
24                 continue;
25             // 有其他节点入队，但是 tail 还是指向原来的，此时设置 tail 即可
26             if (tn != null) {               // lagging tail
27                 // 这个方法就是：如果 tail 此时为 t 的话，设置为 tn
28                 advanceTail(t, tn);
29                 continue;
30             }
31             // 
32             if (timed && nanos <= 0)        // can't wait
33                 return null;
34             // s == null，则创建一个新节点
35             if (s == null)
36                 s = new QNode(e, isData);
37             // 将当前节点，插入到 tail 的后面
38             if (!t.casNext(null, s))        // failed to link in
39                 continue;
40 
41             // 将当前节点设置为新的 tail
42             advanceTail(t, s);              // swing tail and wait
43             // 看到这里，请读者先往下滑到这个方法，看完了以后再回来这里，思路也就不会断了
44             Object x = awaitFulfill(s, e, timed, nanos);
45             // 到这里，说明之前入队的线程被唤醒了，准备往下执行
46             // 若返回的x == s表示，当前线程已经超时或者中断，不然的话s == null或者是匹配的节点
47             if (x == s) {                   // wait was cancelled
48                 clean(t, s);
49                 return null;
50             }
51             // 若s节点被设置为取消
52             if (!s.isOffList()) {           // not already unlinked
53                 advanceHead(t, s);          // unlink if head
54                 if (x != null)              // and forget fields
55                     s.item = s;
56                 s.waiter = null;
57             }
58             return (x != null) ? x : e;
59 
60         // 这里的 else 分支就是上面说的第二种情况，有相应的读或写相匹配的情况
61         } else {                            // complementary-mode
62             QNode m = h.next;               // node to fulfill
63             // 不一致读，表明有其他线程修改了队列
64             if (t != tail || m == null || h != head)
65                 continue;                   // inconsistent read
66 
67             Object x = m.item;
68             if (isData == (x != null) ||    // m already fulfilled
69                 x == m ||                   // m cancelled
70                 !m.casItem(x, e)) {         // lost CAS
71                 advanceHead(h, m);          // dequeue and retry
72                 continue;
73             }
74 
75             advanceHead(h, m);              // successfully fulfilled
76             LockSupport.unpark(m.waiter);
77             return (x != null) ? x : e;
78         }
79     }
80 }
81 
82 void advanceTail(QNode t, QNode nt) {
83     if (tail == t)
84         UNSAFE.compareAndSwapObject(this, tailOffset, t, nt);
85 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

第一次offer(command)时，我们可以看到 transfer 方法中 第32行处 timed && nanos <= 0 成立，此时return null,则offer返回false,所以第一次添加任务时，就会执行最后的 else if (!addWorker(command, false)) 添加一个worker,如果这个worker执行完任务，在getTask()中从等待队列中取任务，这时如果有线程提交任务，则在 if (isRunning(c) && workQueue.offer(command)) 处给到空闲的线程；如果等待超过60秒，则关闭此线程；如果此时线程还在执行任务，还有线程提交任务，则还会执行到最后的 else if (!addWorker(command, false)) 添加一个worker。

SynchronousQueue 是一个比较特殊的 BlockingQueue，其本身不储存任何元素，它有一个虚拟队列（或虚拟栈），不管读操作还是写操作，如果当前队列中存储的是与当前操作相同模式的线程，那么当前操作也进入队列中等待；如果是相反模式，则配对成功，从当前队列中取队头节点。具体的信息，可以看我的另一篇关于 BlockingQueue 的文章。

第一次offer(command)时，如果此时没有相反操作的在getTask,这时添加队列并不会阻塞，直接返回false，然后创建一个worker,执行当前任务，当前worker在60秒内如果有其他线程offer，则会继续getTask执行任务，如果超时60秒，则会回收当前worker，如果并发很多同时提交任务，并且处理任务过慢，则会同时创建很多线程，因为没有空闲的线程等待getTask。如果在60秒内执行完任务，且又有任务来 ，则入队的线程直接将任务给空闲的线程

# [并发编程（十四）—— ScheduledThreadPoolExecutor 实现原理与源码深度解析 之 DelayedWorkQueue](https://www.cnblogs.com/java-chen-hao/p/10275910.html)



**目录**

- 什么是堆？
  - [满二叉树](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_label0_0)
  - [完全二叉树](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_label0_1)
- 堆的实现
  - [最大堆的插入（ADD）](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_label1_0)
  - [最大堆的删除（DELETE）](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_label1_1)
- DelayedWorkQueue类
  - [属性](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_label2_0)
  - [插入元素方法](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_label2_1)
  - [等待获取队列头元素](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_label2_2)
  - [超时等待获取队列头元素](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_label2_3)
- [总结](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_label3)

 

**正文**

我们知道线程池运行时，会不断从任务队列中获取任务，然后执行任务。如果我们想实现延时或者定时执行任务，重要一点就是任务队列会根据任务延时时间的不同进行排序，延时时间越短地就排在队列的前面，先被获取执行。

队列是先进先出的数据结构，就是先进入队列的数据，先被获取。但是有一种特殊的队列叫做优先级队列，它会对插入的数据进行优先级排序，保证优先级越高的数据首先被获取，与数据的插入顺序无关。

实现优先级队列高效常用的一种方式就是使用堆。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_labelTop)

## 什么是堆？

堆通常是一个可以被看做一棵树的数组对象。

堆(heap)又被为优先队列(priority queue)。尽管名为优先队列，但堆并不是队列。

因为队列中允许的操作是先进先出（FIFO），在队尾插入元素，在队头取出元素。

而堆虽然在堆底插入元素，在堆顶取出元素，但是堆中元素的排列不是按照到来的先后顺序，而是按照一定的优先顺序排列的。

 

这里来说明一下满二叉树的概念与完全二叉树的概念。



### 满二叉树

　　**除了叶子节点，所有的节点的左右孩子都不为空，就是一棵满二叉树，如下图。**

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190116095700140-1985424643.png)

可以看出：满二叉树所有的节点都拥有左孩子，又拥有右孩子。



### 完全二叉树

　　**不一定是一个满二叉树，但它不满的那部分一定在右下侧，如下图**

**![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190116095932695-978681924.jpg)**

 

堆总是满足下列性质：

- 堆中某个节点的值总是不大于或不小于其父节点的值；
- 堆总是一棵完全二叉树。

 

- 最大值时，称为“最大堆”，也称大顶堆；
- 最小值时，称为“最小堆”，也称小顶堆。

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190116100200025-116514243.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_labelTop)

## 堆的实现

堆是一个二叉树，但是它最简单的方式是通过数组去实现二叉树，而且因为堆是一个完全二叉树，就不存在数组空间的浪费。怎么使用数组来存储二叉树呢？

就是用数组的下标来模拟二叉树的各个节点,比如说根节点就是0，第一层的左节点是1，右节点是2。由此我们可以得出下列公式：

```
1 // 对于n位置的节点来说：
2 int left = 2 * n + 1; // 左子节点
3 int right = 2 * n + 2; // 右子节点
4 int parent = (n - 1) / 2; // 父节点，当然n要大于0，根节点是没有父节点的
```

对于堆来说，只有两个操作，插入insert和删除remove，不管插入还是删除保证堆的成立条件，1.是完全二叉树，2.父节点的值不能小于子节点的值。



### 最大堆的插入（ADD）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void insert(int value) {
 2      // 第一步将插入的值，直接放在最后一个位置。并将长度加一
 3      store[size++] = value;
 4      // 得到新插入值所在位置。
 5      int index = size - 1;
 6      while(index > 0) {
 7          // 它的父节点位置坐标
 8          int parentIndex = (index - 1) / 2;
 9          // 如果父节点的值小于子节点的值，你不满足堆的条件，那么就交换值
10          if (store[index] > store[parentIndex]) {
11              swap(store, index, parentIndex);
12              index = parentIndex;
13          } else {
14              // 否则表示这条路径上的值已经满足降序，跳出循环
15              break;
16          }
17      }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

主要步骤:

- 直接将value插入到size位置，并将size自增，这样store数组中插入一个值了。

- 要保证从这个叶节点到根节点这条路径上的节点，满足父节点的值不能小于子节点。

- 通过int parentIndex = (index - 1) / 2得到父节点，如果比父节点值大，那么两者位置的值交换，然后再拿这个父节点和它的父父节点比较。

  直到这个节点值比父节点值小，或者这个节点已经是根节点就退出循环。

因为每次循环index都是除以2这种倍数递减的方式，所以它最多循环次数是(log N)次。



### 最大堆的删除（DELETE）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public int remove() {
 2       // 将根的值记录，最后返回
 3       int result = store[0];
 4       // 将最后位置的值放到根节点位置
 5       store[0] = store[--size];
 6       int index = 0;
 7       // 通过循环，保证父节点的值不能小于子节点。
 8       while(true) {
 9           int leftIndex = 2 * index + 1; // 左子节点
10           int rightIndex = 2 * index + 2; // 右子节点
11           // leftIndex >= size 表示这个子节点还没有值。
12           if (leftIndex >= size) break;
13           int maxIndex = leftIndex;
14           //找到左右节点中较大的一个节点
15           if (store[leftIndex] < store[rightIndex]) maxIndex = rightIndex;
16           //与子节点中较大的子节点比较，如果子节点更大，则交换位置
17           //为什么要与较大的子节点比较呢？如果和较小的节点比较，没有交换位置，但有可能比较大的节点小
18           if (store[index] < store[maxIndex]) {
19               swap(store, index, maxIndex);
20               index = maxIndex;
21           } else {
22               //满足子节点比当前节点小，退出循环
23               break;
24           }
25       }
26       //返回最开始的第一个值
27       return result;
28 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在堆中最大值就在根节点，所以操作步骤：

1. 将根节点的值保存到result中。
2. 将最后节点的值移动到根节点，再将长度减一，这样满足堆成立第一个条件，堆是一个完全二叉树。
3. 使用循环，来满足堆成立的第二个条件，父节点的值不能小于子节点的值。
4. 最后返回result。

每次循环我们都是以2的倍数递增，所以它也是最多循环次数是(log N)次。

所以通过堆这种方式可以快速实现优先级队列，它的插入和删除操作的效率都是O(log N)。


那么怎么实现堆排序？这个很简单，利用优先队列的特性：

1. 先遍历数组。将数组中的值依次插入到堆中。
2. 然后再用一个循环将值从堆中取出来。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private static void headSort(int[] arr) {
 2       int size = arr.length;
 3       Head head = new Head(size);
 4       for (int i = 0; i < size; i++) {
 5           head.insert(arr[i]);
 6       }
 7       for (int i = 0; i < size; i++) {
 8           //  实现从大到小的排序
 9           arr[size - 1 - i] = head.remove();
10       }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://user-gold-cdn.xitu.io/2018/12/20/167c913a8d07b809?w=950&h=534&f=gif&s=486254)

堆排序的效率：因为每次插入数据效率是O(log N)，而我们需要进行n次循环，将数组中每个值插入到堆中，所以它的执行时间是O(N * log N)级。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_labelTop)

## DelayedWorkQueue类

```
1 static class DelayedWorkQueue extends AbstractQueue<Runnable>
2         implements BlockingQueue<Runnable> {
```

从定义中看出DelayedWorkQueue是一个阻塞队列。并且DelayedWorkQueue是一个最小堆，最顶点的值最小，即堆中某个节点的值总是不小于其父节点的值。



### 属性

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 初始时，数组长度大小。
 2 private static final int INITIAL_CAPACITY = 16;
 3 // 使用数组来储存队列中的元素。
 4 private RunnableScheduledFuture<?>[] queue =
 5     new RunnableScheduledFuture<?>[INITIAL_CAPACITY];
 6 // 使用lock来保证多线程并发安全问题。
 7 private final ReentrantLock lock = new ReentrantLock();
 8 // 队列中储存元素的大小
 9 private int size = 0;
10 
11 //特指队列头任务所在线程
12 private Thread leader = null;
13 
14 // 当队列头的任务延时时间到了，或者有新的任务变成队列头时，用来唤醒等待线程
15 private final Condition available = lock.newCondition();
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

DelayedWorkQueue是用数组来储存队列中的元素，那么我们看看它是怎么实现优先级队列的。



### 插入元素方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void put(Runnable e) {
 2     offer(e);
 3 }
 4 
 5 public boolean add(Runnable e) {
 6     return offer(e);
 7 }
 8 
 9 public boolean offer(Runnable e, long timeout, TimeUnit unit) {
10     return offer(e);
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们发现与普通阻塞队列相比，这三个添加方法都是调用offer方法。那是因为它没有队列已满的条件，也就是说可以不断地向DelayedWorkQueue添加元素,当元素个数超过数组长度时，会进行数组扩容。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public boolean offer(Runnable x) {
 2     if (x == null)
 3         throw new NullPointerException();
 4     RunnableScheduledFuture<?> e = (RunnableScheduledFuture<?>)x;
 5     // 使用lock保证并发操作安全
 6     final ReentrantLock lock = this.lock;
 7     lock.lock();
 8     try {
 9         int i = size;
10         // 如果要超过数组长度，就要进行数组扩容
11         if (i >= queue.length)
12             // 数组扩容
13             grow();
14         // 将队列中元素个数加一
15         size = i + 1;
16         // 如果是第一个元素，那么就不需要排序，直接赋值就行了
17         if (i == 0) {
18             queue[0] = e;
19             setIndex(e, 0);
20         } else {
21             // 调用siftUp方法，使插入的元素变得有序。
22             siftUp(i, e);
23         }
24         // 表示新插入的元素是队列头，更换了队列头，
25         // 那么就要唤醒正在等待获取任务的线程。
26         if (queue[0] == e) {
27             leader = null;
28             // 唤醒正在等待等待获取任务的线程
29             available.signal();
30         }
31     } finally {
32         lock.unlock();
33     }
34     return true;
35 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

数组扩容方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 private void grow() {
2     int oldCapacity = queue.length;
3     // 每次扩容增加原来数组的一半数量。
4     int newCapacity = oldCapacity + (oldCapacity >> 1); // grow 50%
5     if (newCapacity < 0) // overflow
6         newCapacity = Integer.MAX_VALUE;
7     // 使用Arrays.copyOf来复制一个新数组
8     queue = Arrays.copyOf(queue, newCapacity);
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

插入元素排序siftUp方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void siftUp(int k, RunnableScheduledFuture<?> key) {
 2     // 当k==0时，就到了堆二叉树的根节点了，跳出循环
 3     while (k > 0) {
 4         // 父节点位置坐标, 相当于(k - 1) / 2
 5         int parent = (k - 1) >>> 1;
 6         // 获取父节点位置元素
 7         RunnableScheduledFuture<?> e = queue[parent];
 8         // 如果key元素大于父节点位置元素，满足条件，那么跳出循环
 9         // 因为是从小到大排序的。
10         if (key.compareTo(e) >= 0)
11             break;
12         // 否则就将父节点元素存放到k位置
13         queue[k] = e;
14         // 这个只有当元素是ScheduledFutureTask对象实例才有用，用来快速取消任务。
15         setIndex(e, k);
16         // 重新赋值k，寻找元素key应该插入到堆二叉树的那个节点
17         k = parent;
18     }
19     // 循环结束，k就是元素key应该插入的节点位置
20     queue[k] = key;
21     setIndex(key, k);
22 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

主要是三步：

- 元素个数超过数组长度，就会调用grow()方法，进行数组扩容。
- 将新元素e添加到优先级队列中对应的位置，通过siftUp方法，保证按照元素的优先级排序。
- 如果新插入的元素是队列头，即更换了队列头，那么就要唤醒正在等待获取任务的线程。这些线程可能是因为原队列头元素的延时时间没到，而等待的。

我们来看看动画

![img](https://user-gold-cdn.xitu.io/2018/12/20/167c913a8cefdd7e?w=953&h=538&f=gif&s=142887)

假设现有元素 5 需要插入，为了维持**完全二叉树**的特性，新插入的元素一定是放在结点 6 的右子树；同时为了**满足任一结点的值要小于左右子树的值**这一特性，新插入的元素要和其父结点作比较，如果比父结点小，就要把父结点拉下来顶替当前结点的位置，自己则依次不断向上寻找，找到比自己大的父结点就拉下来，直到没有符合条件的值为止。

**动画讲解：**

> 1. 在这里先将元素 5 插入到末尾，即放在结点 6 的右子树。
> 2. 然后与父类比较， 6 > 5 ，父类数字大于子类数字，子类与父类交换。
> 3. 重复此操作，直到不发生替换。

### 立即获取队列头元素

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public RunnableScheduledFuture<?> poll() {
 2     final ReentrantLock lock = this.lock;
 3     lock.lock();
 4     try {
 5         RunnableScheduledFuture<?> first = queue[0];
 6         // 队列头任务是null，或者任务延时时间没有到，都返回null
 7         if (first == null || first.getDelay(NANOSECONDS) > 0)
 8             return null;
 9         else
10             // 移除队列头元素
11             return finishPoll(first);
12     } finally {
13         lock.unlock();
14     }
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public long getDelay(TimeUnit unit) {
2     return unit.convert(time - now(), NANOSECONDS);
3 }
```

当队列头任务是null，或者任务延时时间没有到，表示这个任务还不能返回，因此直接返回null。否则调用finishPoll方法，移除队列头元素并返回。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 移除队列头元素
 2 private RunnableScheduledFuture<?> finishPoll(RunnableScheduledFuture<?> f) {
 3     // 将队列中元素个数减一
 4     int s = --size;
 5     // 获取队列末尾元素x
 6     RunnableScheduledFuture<?> x = queue[s];
 7     // 原队列末尾元素设置为null
 8     queue[s] = null;
 9     if (s != 0)
10         // 将队列最后一个元素移动到对列头元素位置，然后向下排序
11         // 因为移除了队列头元素，所以进行重新排序。
12         siftDown(0, x);
13     setIndex(f, -1);
14     return f;
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个方法与我们在第一节中，介绍堆的删除方法一样。

> 1. 先将队列中元素个数减一。
> 2. 将原队列末尾元素设置成队列头元素，再将队列末尾元素设置为null。
> 3. 调用siftDown(0, x)方法，保证按照元素的优先级排序。

移除元素排序siftDown方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void siftDown(int k, RunnableScheduledFuture<?> key) {
 2     int half = size >>> 1;
 3     // 通过循环，保证父节点的值不能大于子节点。
 4     while (k < half) {
 5         // 左子节点, 相当于 (k * 2) + 1
 6         int child = (k << 1) + 1;
 7         // 左子节点位置元素
 8         RunnableScheduledFuture<?> c = queue[child];
 9         // 右子节点, 相当于 (k * 2) + 2
10         int right = child + 1;
11         // 如果左子节点元素值大于右子节点元素值，那么右子节点才是较小值的子节点。
12         // 就要将c与child值重新赋值
13         if (right < size && c.compareTo(queue[right]) > 0)
14             c = queue[child = right];
15         // 如果父节点元素值小于较小的子节点元素值，那么就跳出循环
16         if (key.compareTo(c) <= 0)
17             break;
18         // 否则，父节点元素就要和子节点进行交换
19         queue[k] = c;
20         setIndex(c, k);
21         k = child;
22     }
23     // 循环结束，k就是元素key应该插入的节点位置
24     queue[k] = key;
25     setIndex(key, k);
26 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

我们来看看动画

![img](https://user-gold-cdn.xitu.io/2018/12/20/167c913a8d1790ad?w=952&h=533&f=gif&s=160974)

 

**核心点：将最后一个元素填充到堆顶，然后不断的下沉这个元素。**

假设要从节点 1 ，也可以称为取出节点 1 ，为了维持完全二叉树的特性 ，我们将最后一个元素 6 去替代这个 1 ；然后比较 1 和其子树的大小关系，如果比左右子树大（如果存在的话），就要从左右子树中找一个较小的值替换它，而它能自己就要跑到对应子树的位置，再次循环这种操作，直到没有子树比它小。

通过这样的操作，堆依然是堆，总结一下：

- 找到要删除的节点（取出的节点）在数组中的位置
- 用数组中最后一个元素替代这个位置的元素
- 当前位置和其左右子树比较，保证符合最小堆的节点间规则
- 删除最后一个元素



### 等待获取队列头元素

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public RunnableScheduledFuture<?> take() throws InterruptedException {
 2     final ReentrantLock lock = this.lock;
 3     lock.lockInterruptibly();
 4     try {
 5         for (;;) {
 6             RunnableScheduledFuture<?> first = queue[0];
 7             // 如果没有任务，就让线程在available条件下等待。
 8             if (first == null)
 9                 available.await();
10             else {
11                 // 获取任务的剩余延时时间
12                 long delay = first.getDelay(NANOSECONDS);
13                 // 如果延时时间到了，就返回这个任务，用来执行。
14                 if (delay <= 0)
15                     return finishPoll(first);
16                 // 将first设置为null，当线程等待时，不持有first的引用
17                 first = null; // don't retain ref while waiting
18 
19                 // 如果还是原来那个等待队列头任务的线程，
20                 // 说明队列头任务的延时时间还没有到，继续等待。
21                 if (leader != null)
22                     available.await();
23                 else {
24                     // 记录一下当前等待队列头任务的线程
25                     Thread thisThread = Thread.currentThread();
26                     leader = thisThread;
27                     try {
28                         // 当任务的延时时间到了时，能够自动超时唤醒。
29                         available.awaitNanos(delay);
30                     } finally {
31                         if (leader == thisThread)
32                             leader = null;
33                     }
34                 }
35             }
36         }
37     } finally {
38         if (leader == null && queue[0] != null)
39             // 唤醒等待任务的线程
40             available.signal();
41         lock.unlock();
42     }
43 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果队列中没有任务，那么就让当前线程在available条件下等待。如果队列头任务的剩余延时时间delay大于0，那么就让当前线程在available条件下等待delay时间。



### 超时等待获取队列头元素

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public RunnableScheduledFuture<?> poll(long timeout, TimeUnit unit)
 2     throws InterruptedException {
 3     long nanos = unit.toNanos(timeout);
 4     final ReentrantLock lock = this.lock;
 5     lock.lockInterruptibly();
 6     try {
 7         for (;;) {
 8             RunnableScheduledFuture<?> first = queue[0];
 9             // 如果没有任务。
10             if (first == null) {
11                 // 超时时间已到，那么就直接返回null
12                 if (nanos <= 0)
13                     return null;
14                 else
15                     // 否则就让线程在available条件下等待nanos时间
16                     nanos = available.awaitNanos(nanos);
17             } else {
18                 // 获取任务的剩余延时时间
19                 long delay = first.getDelay(NANOSECONDS);
20                 // 如果延时时间到了，就返回这个任务，用来执行。
21                 if (delay <= 0)
22                     return finishPoll(first);
23                 // 如果超时时间已到，那么就直接返回null
24                 if (nanos <= 0)
25                     return null;
26                 // 将first设置为null，当线程等待时，不持有first的引用
27                 first = null; // don't retain ref while waiting
28                 // 如果超时时间小于任务的剩余延时时间，那么就有可能获取不到任务。
29                 // 在这里让线程等待超时时间nanos
30                 if (nanos < delay || leader != null)
31                     nanos = available.awaitNanos(nanos);
32                 else {
33                     Thread thisThread = Thread.currentThread();
34                     leader = thisThread;
35                     try {
36                         // 当任务的延时时间到了时，能够自动超时唤醒。
37                         long timeLeft = available.awaitNanos(delay);
38                         // 计算剩余的超时时间
39                         nanos -= delay - timeLeft;
40                     } finally {
41                         if (leader == thisThread)
42                             leader = null;
43                     }
44                 }
45             }
46         }
47     } finally {
48         if (leader == null && queue[0] != null)
49             // 唤醒等待任务的线程
50             available.signal();
51         lock.unlock();
52     }
53 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

与take方法相比较，就要考虑设置的超时时间，如果超时时间到了，还没有获取到有用任务，那么就返回null。其他的与take方法中逻辑一样。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10275910.html#_labelTop)

## 总结

使用优先级队列DelayedWorkQueue，保证添加到队列中的任务，会按照任务的延时时间进行排序，延时时间少的任务首先被获取。

# [并发编程（十五）——定时器 ScheduledThreadPoolExecutor 实现原理与源码深度解析](https://www.cnblogs.com/java-chen-hao/p/10283413.html)



**目录**

- [类声明](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_label0)
- [ScheduledExecutorService](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_label1)
- [使用例子](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_label2)
- 源码分析
  - [构造器](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_label3_0)
  - [ScheduledFutureTask](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_label3_1)
  - [schedule(Runnable command, long delay,TimeUnit unit)](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_label3_2)
  - [scheduleAtFixedRate(Runnable command,long initialDelay,long period,TimeUnit unit)](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_label3_3)
- [ScheduledThreadPoolExecutor总结](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_label4)

 

**正文**

在上一篇线程池的文章[《并发编程（十一）—— Java 线程池 实现原理与源码深度解析（一）》](https://www.cnblogs.com/java-chen-hao/p/10254260.html)中从ThreadPoolExecutor源码分析了其运行机制。限于篇幅，留下了ScheduledThreadPoolExecutor未做分析，因此本文继续从源代码出发分析ScheduledThreadPoolExecutor的内部原理。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_labelTop)

## 类声明

```
1 public class ScheduledThreadPoolExecutor
2         extends ThreadPoolExecutor
3         implements ScheduledExecutorService {
```

ScheduledThreadPoolExecutor继承了ThreadPoolExecutor，实现了ScheduledExecutorService。因此它具有ThreadPoolExecutor的所有能力。所不同的是它具有定时执行，以周期或间隔循环执行任务等功能。

这里我们先看下ScheduledExecutorService的源码：

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_labelTop)

## ScheduledExecutorService

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //可调度的执行者服务接口
 2 public interface ScheduledExecutorService extends ExecutorService {
 3 
 4     //指定时延后调度执行任务，只执行一次，没有返回值
 5     public ScheduledFuture<?> schedule(Runnable command,
 6                                        long delay, TimeUnit unit);
 7 
 8     //指定时延后调度执行任务，只执行一次，有返回值
 9     public <V> ScheduledFuture<V> schedule(Callable<V> callable,
10                                            long delay, TimeUnit unit);
11 
12     //指定时延后开始执行任务，以后每隔period的时长再次执行该任务
13     public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
14                                                   long initialDelay,
15                                                   long period,
16                                                   TimeUnit unit);
17 
18     //指定时延后开始执行任务，以后任务执行完成后等待delay时长，再次执行任务
19     public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
20                                                      long initialDelay,
21                                                      long delay,
22                                                      TimeUnit unit);
23 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

其中schedule方法用于单次调度执行任务。这里主要理解下后面两个方法。

- scheduleAtFixedRate：该方法在initialDelay时长后第一次执行任务，以后每隔period时长，再次执行任务。注意，period是从任务开始执行算起的。开始执行任务后，定时器每隔period时长检查该任务是否完成，如果完成则再次启动任务，否则等该任务结束后才再次启动任务，看下图示例

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190117153740947-1165273185.png)

- scheduleWithFixDelay：该方法在initialDelay时长后第一次执行任务，以后每当任务执行完成后，等待delay时长，再次执行任务，看下图示例。

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190117153811793-464883421.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_labelTop)

## 使用例子

**1、schedule(Runnable command,long delay, TimeUnit unit)**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /**
 2  * @author: ChenHao
 3  * @Date: Created in 14:54 2019/1/11
 4  */
 5 public class Test1 {
 6     public static void main(String[] args) throws ExecutionException, InterruptedException {
 7         // 延迟1s后开始执行，只执行一次，没有返回值
 8         ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(10);
 9         ScheduledFuture<?> result = executorService.schedule(new Runnable() {
10             @Override
11             public void run() {
12                 System.out.println("gh");
13                 try {
14                     Thread.sleep(3000);
15                 } catch (InterruptedException e) {
16                     // TODO Auto-generated catch block
17                     e.printStackTrace();
18                 }
19             }
20         }, 1000, TimeUnit.MILLISECONDS);
21         System.out.println(result.get());
22     }
23 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190117154908693-978334132.png)

 

**2、schedule(Callable<V> callable, long delay, TimeUnit unit);**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Test2 {
 2     public static void main(String[] args) throws ExecutionException, InterruptedException {
 3         // 延迟1s后开始执行，只执行一次，有返回值
 4         ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(10);
 5         ScheduledFuture<String> result = executorService.schedule(new Callable<String>() {
 6             @Override
 7             public String call() throws Exception {
 8                 try {
 9                     Thread.sleep(3000);
10                 } catch (InterruptedException e) {
11                     // TODO Auto-generated catch block
12                     e.printStackTrace();
13                 }
14                 return "ghq";
15             }
16         }, 1000, TimeUnit.MILLISECONDS);
17         // 阻塞，直到任务执行完成
18         System.out.print(result.get());
19     }
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190117155224469-1956893840.png)

 

**3、scheduleAtFixedRate**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * @author: ChenHao
 * @Date: Created in 14:54 2019/1/11
 */
public class Test3 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(10);
        // 从加入任务开始算1s后开始执行任务，1+2s开始执行，1+2*2s执行，1+n*2s开始执行；
        // 但是如果执行任务时间大于2s则不会并发执行后续任务,当前执行完后,接着执行下次任务。
        ScheduledFuture<?> result = executorService.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                System.out.println(System.currentTimeMillis());
            }
        }, 1000, 2000, TimeUnit.MILLISECONDS);
        
        //一个ScheduledExecutorService里可以同时添加多个定时任务，这样就是形成堆
        ScheduledFuture<?> result2 = executorService.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                System.out.println(System.currentTimeMillis());
            }
        }, 1000, 2000, TimeUnit.MILLISECONDS);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里可以看到一个ScheduledExecutorService 中可以添加多个定时任务，这是就会形成堆

运行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190117155331074-278240945.png)

 

**4、scheduleWithFixedDelay**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /**
 2  * @author: ChenHao
 3  * @Date: Created in 14:54 2019/1/11
 4  */
 5 public class Test4 {
 6     public static void main(String[] args) throws ExecutionException, InterruptedException {
 7         //任务间以固定时间间隔执行，延迟1s后开始执行任务，任务执行完毕后间隔2s再次执行，任务执行完毕后间隔2s再次执行，依次往复
 8         ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(10);
 9         ScheduledFuture<?> result = executorService.scheduleWithFixedDelay(new Runnable() {
10             @Override
11             public void run() {
12                 System.out.println(System.currentTimeMillis());
13             }
14         }, 1000, 2000, TimeUnit.MILLISECONDS);
15 
16         // 由于是定时任务，一直不会返回
17         result.get();
18         System.out.println("over");
19     }
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190117155421114-220830757.png)

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_labelTop)

## 源码分析



### 构造器

```
1 public ScheduledThreadPoolExecutor(int corePoolSize) {
2        super(corePoolSize, Integer.MAX_VALUE, 0, TimeUnit.NANOSECONDS,
3              new DelayedWorkQueue());
4 }
```

 

 内部其实都是调用了父类**ThreadPoolExecutor**的构造器，因此它具有ThreadPoolExecutor的所有能力。

 通过super方法的参数可知，核心线程的数量即传入的参数，而线程池的线程数为Integer.MAX_VALUE，几乎为无上限。
 这里采用了**DelayedWorkQueue**任务队列，也是定时任务的核心，是一种优先队列，时间小的排在前面，所以获取任务的时候就能先获取到时间最小的执行，可以看我上篇文章《[并发编程（十四）—— ScheduledThreadPoolExecutor 实现原理与源码深度解析 之 DelayedWorkQueue](https://www.cnblogs.com/java-chen-hao/p/10275910.html)》。

 由于这里队列没有定义大小，所以队列不会添加满，因此最大的线程数就是核心线程数，超过核心线程数的任务就放在队列里，并不重新开启临时线程。

我们先来看看几个入口方法的实现：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public ScheduledFuture<?> schedule(Runnable command,
 2                                    long delay,
 3                                    TimeUnit unit) {
 4     if (command == null || unit == null)
 5         throw new NullPointerException();
 6     RunnableScheduledFuture<?> t = decorateTask(command,
 7         new ScheduledFutureTask<Void>(command, null,
 8                                       triggerTime(delay, unit)));
 9     delayedExecute(t);
10     return t;
11 }
12 
13 public <V> ScheduledFuture<V> schedule(Callable<V> callable,
14                                        long delay,
15                                        TimeUnit unit) {
16     if (callable == null || unit == null)
17         throw new NullPointerException();
18     RunnableScheduledFuture<V> t = decorateTask(callable,
19         new ScheduledFutureTask<V>(callable,
20                                    triggerTime(delay, unit)));
21     delayedExecute(t);
22     return t;
23 }
24 
25 public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
26                                               long initialDelay,
27                                               long period,
28                                               TimeUnit unit) {
29     if (command == null || unit == null)
30         throw new NullPointerException();
31     if (period <= 0)
32         throw new IllegalArgumentException();
33     ScheduledFutureTask<Void> sft =
34         new ScheduledFutureTask<Void>(command,
35                                       null,
36                                       triggerTime(initialDelay, unit),
37                                       unit.toNanos(period));
38     RunnableScheduledFuture<Void> t = decorateTask(command, sft);
39     sft.outerTask = t;
40     delayedExecute(t);
41     return t;
42 }
43 
44 public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
45                                                  long initialDelay,
46                                                  long delay,
47                                                  TimeUnit unit) {
48     if (command == null || unit == null)
49         throw new NullPointerException();
50     if (delay <= 0)
51         throw new IllegalArgumentException();
52     ScheduledFutureTask<Void> sft =
53         new ScheduledFutureTask<Void>(command,
54                                       null,
55                                       triggerTime(initialDelay, unit),
56                                       unit.toNanos(-delay));
57     RunnableScheduledFuture<Void> t = decorateTask(command, sft);
58     sft.outerTask = t;
59     delayedExecute(t);
60     return t;
61 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 这几个方法都是将任务封装成了ScheduledFutureTask，上面做的首先把runnable装饰为delay队列所需要的格式的元素，然后把元素加入到阻塞队列，然后线程池线程会从阻塞队列获取超时的元素任务进行处理，下面看下队列元素如何实现的。



### ScheduledFutureTask

ScheduledFutureTask是一个延时定时任务，它可以返回任务剩余延时时间，可以被周期性地执行。

**属性**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private class ScheduledFutureTask<V>
 2         extends FutureTask<V> implements RunnableScheduledFuture<V> {
 3         /** 是一个序列，每次创建任务的时候，都会自增。 */
 4         private final long sequenceNumber;
 5 
 6         /** 任务能够开始执行的时间 */
 7         private long time;
 8 
 9         /**
10          * 任务周期执行的时间
11          * 0表示不是一个周期定时任务
12          * 正数表示固定周期时间去执行任务
13          * 负数表示任务完成之后，延时period时间再去执行任务
14          */
15         private final long period;
16 
17         /** 表示再次执行的任务，在reExecutePeriodic中调用 */
18         RunnableScheduledFuture<V> outerTask = this;
19 
20         /**
21          * 表示在任务队列中的索引位置，用来支持快速从队列中删除任务。
22          */
23         int heapIndex;
24 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ScheduledFutureTask继承了 FutureTask 和 RunnableScheduledFuture

属性说明：

> 1. sequenceNumber: 是一个序列，每次创建任务的时候，都会自增。
> 2. time: 任务能够开始执行的时间。
> 3. period: 任务周期执行的时间。0表示不是一个周期定时任务。
> 4. outerTask: 表示再次执行的任务，在reExecutePeriodic中调用
> 5. heapIndex: 表示在任务队列中的索引位置，用来支持快速从队列中删除任务。

**构造器**

- **创建延时任务**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /**
 2  * 创建延时任务
 3  */
 4 ScheduledFutureTask(Runnable r, V result, long ns) {
 5     // 调用父类的方法
 6     super(r, result);
 7     // 任务开始的时间
 8     this.time = ns;
 9     // period是0，不是一个周期定时任务
10     this.period = 0;
11     // 每次创建任务的时候，sequenceNumber都会自增
12     this.sequenceNumber = sequencer.getAndIncrement();
13 }
14 
15  /**
16  * 创建延时任务
17  */
18 ScheduledFutureTask(Callable<V> callable, long ns) {
19     // 调用父类的方法
20     super(callable);
21     // 任务开始的时间
22     this.time = ns;
23     // period是0，不是一个周期定时任务
24     this.period = 0;
25     // 每次创建任务的时候，sequenceNumber都会自增
26     this.sequenceNumber = sequencer.getAndIncrement();
27 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 我们看看super()，其实就是FutureTask 里面的构造方法，关于FutureTask 可以看看我之前的文章《[Java 多线程（五）—— 线程池基础 之 FutureTask源码解析](https://www.cnblogs.com/java-chen-hao/p/10243509.html)》

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public FutureTask(Runnable runnable, V result) {
 2     this.callable = Executors.callable(runnable, result);
 3     this.state = NEW;       // ensure visibility of callable
 4 }
 5 public FutureTask(Callable<V> callable) {
 6     if (callable == null)
 7         throw new NullPointerException();
 8     this.callable = callable;
 9     this.state = NEW;       // ensure visibility of callable
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- **创建延时定时任务**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /**
 2  * 创建延时定时任务
 3  */
 4 ScheduledFutureTask(Runnable r, V result, long ns, long period) {
 5     // 调用父类的方法
 6     super(r, result);
 7     // 任务开始的时间
 8     this.time = ns;
 9     // 周期定时时间
10     this.period = period;
11     // 每次创建任务的时候，sequenceNumber都会自增
12     this.sequenceNumber = sequencer.getAndIncrement();
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

延时定时任务不同的是设置了period，后面通过判断period是否为0来确定是否是定时任务。

**run()**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void run() {
 2     // 是否是周期任务
 3     boolean periodic = isPeriodic();
 4     // 如果不能在当前状态下运行，那么就要取消任务
 5     if (!canRunInCurrentRunState(periodic))
 6         cancel(false);
 7     // 如果只是延时任务，那么就调用run方法，运行任务。
 8     else if (!periodic)
 9         ScheduledFutureTask.super.run();
10     // 如果是周期定时任务，调用runAndReset方法，运行任务。
11     // 这个方法不会改变任务的状态，所以可以反复执行。
12     else if (ScheduledFutureTask.super.runAndReset()) {
13         // 设置周期任务下一次执行的开始时间time
14         setNextRunTime();
15         // 重新执行任务outerTask
16         reExecutePeriodic(outerTask);
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个方法会在ThreadPoolExecutor的runWorker方法中调用，而且这个方法调用，说明肯定已经到了任务的开始时间time了。这个方法我们待会会再继续来回看一下

> 1. 先判断当前线程状态能不能运行任务，如果不能，就调用cancel()方法取消本任务。
> 2. 如果任务只是一个延时任务，那么调用父类的run()运行任务，改变任务的状态，表示任务已经运行完成了。
> 3. 如果任务只是一个周期定时任务，那么就任务必须能够反复执行，那么就不能调用run()方法，它会改变任务的状态。而是调用runAndReset()方法，只是简单地运行任务，而不会改变任务状态。
> 4. 设置周期任务下一次执行的开始时间time，并重新执行任务。



### schedule(Runnable command, long delay,TimeUnit unit)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public ScheduledFuture<?> schedule(Runnable command,
 2                                   long delay,
 3                                   TimeUnit unit) {
 4    if (command == null || unit == null)
 5        throw new NullPointerException();
 6 
 7    //装饰任务，主要实现public long getDelay(TimeUnit unit)和int compareTo(Delayed other)方法
 8    RunnableScheduledFuture<?> t = decorateTask(command,
 9        new ScheduledFutureTask<Void>(command, null,
10                                      triggerTime(delay, unit)));
11    //添加任务到延迟队列
12    delayedExecute(t);
13    return t;
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 获取延时执行时间

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private long triggerTime(long delay, TimeUnit unit) {
 2     return triggerTime(unit.toNanos((delay < 0) ? 0 : delay));
 3 }
 4 
 5 /**
 6  * Returns the trigger time of a delayed action.
 7  */
 8 long triggerTime(long delay) {
 9     //当前时间加上延时时间
10     return now() +
11         ((delay < (Long.MAX_VALUE >> 1)) ? delay : overflowFree(delay));
12 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上述的**decorateTask**方法把Runnable任务包装成ScheduledFutureTask，用户可以根据自己的需要覆写该方法：

```
1 protected <V> RunnableScheduledFuture<V> decorateTask(Runnable runnable, RunnableScheduledFuture<V> task) {
2     return task;
3 }
```

schedule的核心是其中的**delayedExecute**方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void delayedExecute(RunnableScheduledFuture<?> task) {
 2     if (isShutdown())   // 线程池已关闭
 3         reject(task);   // 任务拒绝策略
 4     else {
 5         //将任务添加到任务队列，会根据任务延时时间进行排序
 6         super.getQueue().add(task);
 7         // 如果线程池状态改变了，当前状态不能运行任务，那么就尝试移除任务，
 8         // 移除成功，就取消任务。
 9         if (isShutdown() && !canRunInCurrentRunState(task.isPeriodic()) && remove(task))
10             task.cancel(false);  // 取消任务
11         else
12             // 预先启动工作线程，确保线程池中有工作线程。
13             ensurePrestart();
14     }
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个方法的主要作用就是将任务添加到任务队列中，因为这里任务队列是优先级队列DelayedWorkQueue，它会根据任务的延时时间进行排序。

- 如果线程池不是RUNNING状态，不能执行延时任务task，那么调用reject(task)方法，拒绝执行任务task。
- 将任务添加到任务队列中，会根据任务的延时时间进行排序。
- 因为是多线程并发环境，就必须判断在添加任务的过程中，线程池状态是否被别的线程更改了，那么就可能要取消任务了。
- 将任务添加到任务队列后，还要确保线程池中有工作线程，不然任务也不为执行。所以ensurePrestart()方法预先启动工作线程，确保线程池中有工作线程。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 void ensurePrestart() {
 2     // 线程池中的线程数量
 3     int wc = workerCountOf(ctl.get());
 4     // 如果小于核心池数量，就创建新的工作线程
 5     if (wc < corePoolSize)
 6         addWorker(null, true);
 7     // 说明corePoolSize数量是0，必须创建一个工作线程来执行任务
 8     else if (wc == 0)
 9         addWorker(null, false);
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过ensurePrestart可以看到，如果核心线程池未满，则新建的工作线程会被放到核心线程池中。如果核心线程池已经满了，ScheduledThreadPoolExecutor不会像ThreadPoolExecutor那样再去创建归属于非核心线程池的工作线程，加入到队列就完了，等待核心线程执行完任务再拉取队列里的任务。也就是说，在ScheduledThreadPoolExecutor中，一旦核心线程池满了，就不会再去创建工作线程。

***这里思考一点，什么时候会执行else if (wc == 0)创建一个归属于非核心线程池的工作线程？\***
答案是，当通过setCorePoolSize方法设置核心线程池大小为0时，这里必须要保证任务能够被执行，所以会创建一个工作线程，放到非核心线程池中。

看到 addWorker(null, true); 并没有将任务设置进入，而是设置的null, 则说明线程池里线程第一次启动时， runWorker中取到的 firstTask为null,需要通过 getTask() 从队列中取任务，这里可以看看我之前写的关于线程池的文章《[并发编程（十一）—— Java 线程池 实现原理与源码深度解析（一）](https://www.cnblogs.com/java-chen-hao/p/10254260.html)》。

getTask()中  Runnable r = timed ? workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :workQueue.take();如果是存在核心线程则调用take()，如果传入的核心线程为0，则存在一个临时线程，调用poll()，这两个方法都会先获取时间，看看有没有达到执行时间，没有达到执行时间则阻塞，可以看看我上一篇文章，达到执行时间，则取到任务，就会执行下面的run方法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public void run() {
 2     // 是否是周期任务
 3     boolean periodic = isPeriodic();
 4     // 如果不能在当前状态下运行，那么就要取消任务
 5     if (!canRunInCurrentRunState(periodic))
 6         cancel(false);
 7     // 如果只是延时任务，那么就调用run方法，运行任务。
 8     else if (!periodic)
 9         ScheduledFutureTask.super.run();
10     // 如果是周期定时任务，调用runAndReset方法，运行任务。
11     // 这个方法不会改变任务的状态，所以可以反复执行。
12     else if (ScheduledFutureTask.super.runAndReset()) {
13         // 设置周期任务下一次执行的开始时间time
14         setNextRunTime();
15         // 重新执行任务outerTask
16         reExecutePeriodic(outerTask);
17     }
18 }
19 
20 public boolean isPeriodic() {
21     return period != 0;
22 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 schedule不是周期任务，那么调用父类的run()运行任务，改变任务的状态，表示任务已经运行完成了。



### scheduleAtFixedRate(Runnable command,long initialDelay,long period,TimeUnit unit)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
 2                                              long initialDelay,
 3                                              long period,
 4                                              TimeUnit unit) {
 5    if (command == null || unit == null)
 6        throw new NullPointerException();
 7    if (period <= 0)
 8        throw new IllegalArgumentException();
 9    //装饰任务类，注意period=period>0，不是负的
10    ScheduledFutureTask<Void> sft =
11        new ScheduledFutureTask<Void>(command,
12                                      null,
13                                      triggerTime(initialDelay, unit),
14                                      unit.toNanos(period));
15    RunnableScheduledFuture<Void> t = decorateTask(command, sft);
16    sft.outerTask = t;
17    //添加任务到队列
18    delayedExecute(t);
19    return t;
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果是周期任务则执行上面run()方法中的第12行，调用父类中的runAndReset（），这个方法同run方法比较的区别是call方法执行后不设置结果，因为周期型任务会多次执行，所以为了让FutureTask支持这个特性除了发生异常不设置结果。

执行完任务后通过setNextRunTime方法计算下一次启动时间:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private void setNextRunTime() {
 2    long p = period;
 3   //period=delay;
 4    if (p > 0)
 5        time += p;//由于period>0所以执行这里，设置time=time+delay
 6    else
 7        time = triggerTime(-p);
 8 }
 9 
10 long triggerTime(long delay) {
11     return now() +
12         ((delay < (Long.MAX_VALUE >> 1)) ? delay : overflowFree(delay));
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

scheduleAtFixedRate会执行到情况一，下一次任务的启动时间最早为上一次任务的启动时间加period。
scheduleWithFixedDelay会执行到情况二，这里很巧妙的将period参数设置为负数到达这段代码块，在此又将负的period转为正数。情况二将下一次任务的启动时间设置为当前时间加period。

然后将任务再次添加到任务队列:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /**
 2  * 重新执行任务task
 3  */
 4 void reExecutePeriodic(RunnableScheduledFuture<?> task) {
 5     // 判断当前线程池状态能不能运行任务
 6     if (canRunInCurrentRunState(true)) {
 7         // 将任务添加到任务队列，会根据任务延时时间进行排序
 8         super.getQueue().add(task);
 9         // 如果线程池状态改变了，当前状态不能运行任务，那么就尝试移除任务，
10         // 移除成功，就取消任务。
11         if (!canRunInCurrentRunState(true) && remove(task))
12             task.cancel(false);
13         else
14             // 预先启动工作线程，确保线程池中有工作线程。
15             ensurePrestart();
16     }
17 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个方法与delayedExecute方法很像，都是将任务添加到任务队列中。

> 1. 如果当前线程池状态能够运行任务，那么任务添加到任务队列。
> 2. 如果在在添加任务的过程中，线程池状态是否被别的线程更改了，那么就要进行判断，是否需要取消任务。
> 3. 调用ensurePrestart()方法，预先启动工作线程，确保线程池中有工作线程。

### ScheduledFuture的get方法

既然ScheduledFuture的实现是ScheduledFutureTask，而ScheduledFutureTask继承自FutureTask，所以ScheduledFuture的get方法的实现就是FutureTask的get方法的实现，FutureTask的get方法的实现分析在ThreadPoolExecutor篇已经写过，这里不再叙述。要注意的是ScheduledFuture的get方法对于非周期任务才是有效的。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10283413.html#_labelTop)

## ScheduledThreadPoolExecutor总结

- ScheduledThreadPoolExecutor和ThreadPoolExecutor的区别：

　　　　ThreadPoolExecutor每次addwoker就会将自己的Task传进新创建的woker中的线程执行，因此woker会第一时间执行当前Task，只有线程数超过了核心线程才会将任务放进队列里

　　　　ScheduledThreadPoolExecutor是直接入队列，并且创建woker时传到woker的是null，说明woker中的线程刚启动时并没有任务执行，只能通过getTask去队列里取任务，取任务时会判断是否到了执行时间，因此具有了延时执行的特性，并且task执行完了，会将当前任务重新放进堆里，并设置下次执行的时间。

- ScheduledThreadPoolExecutor是实现自ThreadPoolExecutor的线程池，构造方法中传入参数n，则最多会有n个核心线程工作，空闲的核心线程不会被自动终止,而是一直阻塞在DelayedWorkQueue的take方法尝试获取任务。构造方法传入的参数为0，ScheduledThreadPoolExecutor将以非核心线程工作，并且最多只会创建一个非核心线程，参考上文中**ensurePrestart方法**的执行过程。而这个非核心线程以poll方法获取定时任务之所以不会因为超时就被回收，是因为任务队列并不为空，只有在任务队列为空时才会将空闲线程回收，详见ThreadPoolExecutor篇的**runWorker方法**,之前我以为空闲的非核心线程超时就会被回收是不正确的,还要具备任务队列为空这个条件。
- ScheduledThreadPoolExecutor的定时执行任务依赖于DelayedWorkQueue，其内部用可扩容的数组实现以启动时间升序的二叉树。
- 工作线程尝试获取DelayedWorkQueue的任务只有在任务到达指定时间才会成功，否则非核心线程会超时返回null，核心线程一直阻塞。
- 对于非周期型任务只会执行一次并且可以通过ScheduledFuture的get方法阻塞得到结果，其内部实现依赖于FutureTask的get方法。
- 周期型任务通过get方法无法获取有效结果，因为FutureTask对于周期型任务执行的是runAndReset方法，并不会设置结果。周期型任务执行完毕后会重新计算下一次启动时间并且再次添加到DelayedWorkQueue中，所有的Task会公用一个队列，如果一个定时器里添加多个任务，此时就会形成堆，如果只是一个定时任务，则每次只有堆顶一个数据，并且也只需要一个核心线程就够用了，因为只有当前任务执行完才会再将该任务添加到堆里。

# [并发编程（十六）——java7 深入并发包 ConcurrentHashMap 源码解析](https://www.cnblogs.com/java-chen-hao/p/10327280.html)



**目录**

- JDK1.7的实现
  - [初始化](https://www.cnblogs.com/java-chen-hao/p/10327280.html#_label0_0)
  - [put 过程分析](https://www.cnblogs.com/java-chen-hao/p/10327280.html#_label0_1)
  - [get 过程分析](https://www.cnblogs.com/java-chen-hao/p/10327280.html#_label0_2)
  - [size操作](https://www.cnblogs.com/java-chen-hao/p/10327280.html#_label0_3)
  - [并发问题分析](https://www.cnblogs.com/java-chen-hao/p/10327280.html#_label0_4)

 

**正文**

以前写过介绍HashMap的文章，文中提到过HashMap在put的时候，插入的元素超过了容量（由负载因子决定）的范围就会触发扩容操作，就是rehash，这个会重新将原数组的内容重新hash到新的扩容数组中，在多线程的环境下，存在同时其他的元素也在进行put操作，如果hash值相同，可能出现同时在同一数组下用链表表示，造成闭环，导致在get时会出现死循环，所以HashMap是线程不安全的。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10327280.html#_labelTop)

## JDK1.7的实现

整个 ConcurrentHashMap 由一个个 Segment 组成，Segment 代表”部分“或”一段“的意思，所以很多地方都会将其描述为分段锁。注意，行文中，我很多地方用了“槽”来代表一个 segment。

简单理解就是，ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190125170257376-1816689764.png)

 

concurrencyLevel：并行级别、并发数、Segment 数。默认是 16，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。

再具体到每个 Segment 内部，其实每个 Segment 很像之前介绍的 HashMap，不过它要保证线程安全，所以处理起来要麻烦些。



### 初始化

initialCapacity：初始容量，这个值指的是整个 ConcurrentHashMap 的初始容量，实际操作的时候需要平均分给每个 Segment。

loadFactor：负载因子，之前我们说了，Segment 数组不可以扩容，所以这个负载因子是给每个 Segment 内部使用的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public ConcurrentHashMap(int initialCapacity,
 2                          float loadFactor, int concurrencyLevel) {
 3     if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
 4         throw new IllegalArgumentException();
 5     if (concurrencyLevel > MAX_SEGMENTS)
 6         concurrencyLevel = MAX_SEGMENTS;
 7     // Find power-of-two sizes best matching arguments
 8     int sshift = 0;
 9     int ssize = 1;
10     // 计算并行级别 ssize，因为要保持并行级别是 2 的 n 次方
11     while (ssize < concurrencyLevel) {
12         ++sshift;
13         ssize <<= 1;
14     }
15     // 我们这里先不要那么烧脑，用默认值，concurrencyLevel 为 16，sshift 为 4
16     // 那么计算出 segmentShift 为 28，segmentMask 为 15，后面会用到这两个值
17     this.segmentShift = 32 - sshift;
18     this.segmentMask = ssize - 1;
19 
20     if (initialCapacity > MAXIMUM_CAPACITY)
21         initialCapacity = MAXIMUM_CAPACITY;
22 
23     // initialCapacity 是设置整个 map 初始的大小，
24     // 这里根据 initialCapacity 计算 Segment 数组中每个位置可以分到的大小
25     // 如 initialCapacity 为 64，那么每个 Segment 或称之为"槽"可以分到 4 个
26     int c = initialCapacity / ssize;
27     if (c * ssize < initialCapacity)
28         ++c;
29     // 默认 MIN_SEGMENT_TABLE_CAPACITY 是 2，这个值也是有讲究的，因为这样的话，对于具体的槽上，
30     // 插入一个元素不至于扩容，插入第二个的时候才会扩容
31     int cap = MIN_SEGMENT_TABLE_CAPACITY; 
32     while (cap < c)
33         cap <<= 1;
34 
35     // 创建 Segment 数组，
36     // 并创建数组的第一个元素 segment[0]
37     Segment<K,V> s0 =
38         new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
39                          (HashEntry<K,V>[])new HashEntry[cap]);
40     Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
41     // 往数组写入 segment[0]
42     UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
43     this.segments = ss;
44 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

初始化完成，我们得到了一个 Segment 数组。

我们就当是用 new ConcurrentHashMap() 无参构造函数进行初始化的，那么初始化完成后：

- Segment 数组长度为 16，不可以扩容
- Segment[i] 的默认大小为 2，负载因子是 0.75，得出初始阈值为 1.5，也就是以后插入第一个元素不会触发扩容，插入第二个会进行第一次扩容
- 这里初始化了 segment[0]，其他位置还是 null，至于为什么要初始化 segment[0]，后面的代码会介绍
- 当前 segmentShift 的值为 32 - 4 = 28，segmentMask 为 16 - 1 = 15，姑且把它们简单翻译为移位数和掩码，这两个值马上就会用到

#### Segment 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 static class Segment<K,V> extends ReentrantLock implements Serializable {
2 
3     transient volatile HashEntry<K,V>[] table;
4     
5     transient int count;
6     
7     transient int modCount;
8     
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

从上Segment的继承体系可以看出，Segment实现了ReentrantLock,也就带有锁的功能,table使用volatile修饰，保证了内存可见性。



### put 过程分析

我们先看 put 的主流程，对于其中的一些关键细节操作，后面会进行详细介绍。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public V put(K key, V value) {
 2     Segment<K,V> s;
 3     if (value == null)
 4         throw new NullPointerException();
 5     // 1. 计算 key 的 hash 值
 6     int hash = hash(key);
 7     // 2. 根据 hash 值找到 Segment 数组中的位置 j
 8     //    hash 是 32 位，无符号右移 segmentShift(28) 位，剩下高 4 位，
 9     //    然后和 segmentMask(15) 做一次与操作，也就是说 j 是 hash 值的高 4 位，也就是槽的数组下标
10     int j = (hash >>> segmentShift) & segmentMask;
11     // 刚刚说了，初始化的时候初始化了 segment[0]，但是其他位置还是 null，
12     // ensureSegment(j) 对 segment[j] 进行初始化
13     if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
14          (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
15         s = ensureSegment(j);
16     // 3. 插入新值到 槽 s 中
17     return s.put(key, hash, value, false);
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

#### 初始化槽: ensureSegment

ConcurrentHashMap 初始化的时候会初始化第一个槽 segment[0]，对于其他槽来说，在插入第一个值的时候进行初始化。

这里需要考虑并发，因为很可能会有多个线程同时进来初始化同一个槽 segment[k]，不过只要有一个成功了就可以。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private Segment<K,V> ensureSegment(int k) {
 2     final Segment<K,V>[] ss = this.segments;
 3     long u = (k << SSHIFT) + SBASE; // raw offset
 4     Segment<K,V> seg;
 5     if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
 6         // 这里看到为什么之前要初始化 segment[0] 了，
 7         // 使用当前 segment[0] 处的数组长度和负载因子来初始化 segment[k]
 8         // 为什么要用“当前”，因为 segment[0] 可能早就扩容过了
 9         Segment<K,V> proto = ss[0];
10         int cap = proto.table.length;
11         float lf = proto.loadFactor;
12         int threshold = (int)(cap * lf);
13 
14         // 初始化 segment[k] 内部的数组
15         HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
16         if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
17             == null) { // 再次检查一遍该槽是否被其他线程初始化了。
18 
19             Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
20             // 使用 while 循环，内部用 CAS，当前线程成功设值或其他线程成功设值后，退出,如果其他线程成功设置后，这里获取到直接返回
21             while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
22                    == null) {
23                 if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
24                     break;
25             }
26         }
27     }
28     return seg;
29 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

总的来说，ensureSegment(int k) 比较简单，对于并发操作使用 CAS 进行控制。

第一层很简单，根据 hash 值很快就能找到相应的 Segment，之后就是 Segment 内部的 put 操作了。

#### Segment 内部是由 **数组**+**链表** 组成的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 final V put(K key, int hash, V value, boolean onlyIfAbsent) {
 2     // 在往该 segment 写入前，需要先获取该 segment 的独占锁
 3     //    先看主流程，后面还会具体介绍这部分内容
 4     HashEntry<K,V> node = tryLock() ? null :
 5         scanAndLockForPut(key, hash, value);
 6     V oldValue;
 7     try {
 8         // 这个是 segment 内部的数组
 9         HashEntry<K,V>[] tab = table;
10         // 再利用 hash 值，求应该放置的数组下标
11         int index = (tab.length - 1) & hash;
12         // first 是数组该位置处的链表的表头
13         HashEntry<K,V> first = entryAt(tab, index);
14 
15         // 下面这串 for 循环虽然很长，不过也很好理解，想想该位置没有任何元素和已经存在一个链表这两种情况
16         for (HashEntry<K,V> e = first;;) {
17             if (e != null) {
18                 K k;
19                 if ((k = e.key) == key ||
20                     (e.hash == hash && key.equals(k))) {
21                     oldValue = e.value;
22                     if (!onlyIfAbsent) {
23                         // 覆盖旧值
24                         e.value = value;
25                         ++modCount;
26                     }
27                     break;
28                 }
29                 // 继续顺着链表走
30                 e = e.next;
31             }
32             else {
33                 // node 到底是不是 null，这个要看获取锁的过程，不过和这里都没有关系。
34                 // 如果不为 null，那就直接将它设置为链表表头；如果是null，初始化并设置为链表表头。
35                 if (node != null)
36                     node.setNext(first);
37                 else
38                     node = new HashEntry<K,V>(hash, key, value, first);
39 
40                 int c = count + 1;
41                 // 如果超过了该 segment 的阈值，这个 segment 需要扩容
42                 if (c > threshold && tab.length < MAXIMUM_CAPACITY)
43                     rehash(node); // 扩容后面也会具体分析
44                 else
45                     // 没有达到阈值，将 node 放到数组 tab 的 index 位置，
46                     // 其实就是将新的节点设置成原链表的表头
47                     setEntryAt(tab, index, node);
48                 ++modCount;
49                 count = c;
50                 oldValue = null;
51                 break;
52             }
53         }
54     } finally {
55         // 解锁
56         unlock();
57     }
58     return oldValue;
59 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

整体流程还是比较简单的，由于有独占锁的保护，所以 segment 内部的操作并不复杂。至于这里面的并发问题，我们稍后再进行介绍。

到这里 put 操作就结束了，接下来，我们说一说其中几步关键的操作。

#### 获取写入锁: scanAndLockForPut

前面我们看到，在往某个 segment 中 put 的时候，首先会调用 node = tryLock() ? null : scanAndLockForPut(key, hash, value)，也就是说先进行一次 tryLock() 快速获取该 segment 的独占锁，如果失败，那么进入到 scanAndLockForPut 这个方法来获取锁。

下面我们来具体分析这个方法中是怎么控制加锁的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
 2     HashEntry<K,V> first = entryForHash(this, hash);
 3     HashEntry<K,V> e = first;
 4     HashEntry<K,V> node = null;
 5     int retries = -1; // negative while locating node
 6 
 7     // 循环获取锁
 8     while (!tryLock()) {
 9         HashEntry<K,V> f; // to recheck first below
10         if (retries < 0) {
11             if (e == null) {
12                 if (node == null) // speculatively create node
13                     // 进到这里说明数组该位置的链表是空的，没有任何元素
14                     // 当然，进到这里的另一个原因是 tryLock() 失败，所以该槽存在并发，不一定是该位置
15                     node = new HashEntry<K,V>(hash, key, value, null);
16                 retries = 0;
17             }
18             else if (key.equals(e.key))
19                 retries = 0;
20             else
21                 // 顺着链表往下走
22                 e = e.next;
23         }
24         // 重试次数如果超过 MAX_SCAN_RETRIES（单核1多核64），那么不抢了，进入到阻塞队列等待锁
25         //    lock() 是阻塞方法，直到获取锁后返回
26         else if (++retries > MAX_SCAN_RETRIES) {
27             lock();
28             break;
29         }
30         else if ((retries & 1) == 0 &&
31                  // 这个时候是有大问题了，那就是有新的元素进到了链表，成为了新的表头
32                  //     所以这边的策略是，相当于重新走一遍这个 scanAndLockForPut 方法
33                  (f = entryForHash(this, hash)) != first) {
34             e = first = f; // re-traverse if entry changed
35             retries = -1;
36         }
37     }
38     return node;
39 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

这个方法有两个出口，一个是 tryLock() 成功了，循环终止，另一个就是重试次数超过了 MAX_SCAN_RETRIES，进到 lock() 方法，此方法会阻塞等待，直到成功拿到独占锁。

这个方法就是看似复杂，但是其实就是做了一件事，那就是获取该 segment 的独占锁，如果需要的话顺便实例化了一下 node。

获取锁时，并不直接使用lock来获取，因为该方法获取锁失败时会挂起。事实上，它使用了自旋锁，如果tryLock获取锁失败，说明锁被其它线程占用，此时通过循环再次以tryLock的方式申请锁。如果在循环过程中该Key所对应的链表头被修改，则重置retry次数。如果retry次数超过一定值，则使用lock方法申请锁。

这里使用自旋锁是因为自旋锁的效率比较高，但是它消耗CPU资源比较多，因此在自旋次数超过阈值时切换为互斥锁。

#### 扩容: rehash

重复一下，segment 数组不能扩容，扩容是 segment 数组某个位置内部的数组 HashEntry\<k,v>[] 进行扩容，扩容后，容量为原来的 2 倍。

首先，我们要回顾一下触发扩容的地方，put 的时候，如果判断该值的插入会导致该 segment 的元素个数超过阈值，那么先进行扩容，再插值，读者这个时候可以回去 put 方法看一眼。

该方法不需要考虑并发，因为到这里的时候，是持有该 segment 的独占锁的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 方法参数上的 node 是这次扩容后，需要添加到新的数组中的数据。
 2 private void rehash(HashEntry<K,V> node) {
 3     HashEntry<K,V>[] oldTable = table;
 4     int oldCapacity = oldTable.length;
 5     // 2 倍
 6     int newCapacity = oldCapacity << 1;
 7     threshold = (int)(newCapacity * loadFactor);
 8     // 创建新数组
 9     HashEntry<K,V>[] newTable =
10         (HashEntry<K,V>[]) new HashEntry[newCapacity];
11     // 新的掩码，如从 16 扩容到 32，那么 sizeMask 为 31，对应二进制 ‘000...00011111’
12     int sizeMask = newCapacity - 1;
13 
14     // 遍历原数组，老套路，将原数组位置 i 处的链表拆分到 新数组位置 i 和 i+oldCap 两个位置
15     for (int i = 0; i < oldCapacity ; i++) {
16         // e 是链表的第一个元素
17         HashEntry<K,V> e = oldTable[i];
18         if (e != null) {
19             HashEntry<K,V> next = e.next;
20             // 计算应该放置在新数组中的位置，
21             // 假设原数组长度为 16，e 在 oldTable[3] 处，那么 idx 只可能是 3 或者是 3 + 16 = 19
22             int idx = e.hash & sizeMask;
23             if (next == null)   // 该位置处只有一个元素，那比较好办
24                 newTable[idx] = e;
25             else { // Reuse consecutive sequence at same slot
26                 // e 是链表表头
27                 HashEntry<K,V> lastRun = e;
28                 // idx 是当前链表的头结点 e 的新位置
29                 int lastIdx = idx;
30 
31                 // 下面这个 for 循环会找到一个 lastRun 节点，这个节点之后的所有元素是将要放到一起的
32                 for (HashEntry<K,V> last = next;
33                      last != null;
34                      last = last.next) {
35                     int k = last.hash & sizeMask;
36                     if (k != lastIdx) {
37                         lastIdx = k;
38                         lastRun = last;
39                     }
40                 }
41                 // 将 lastRun 及其之后的所有节点组成的这个链表放到 lastIdx 这个位置
42                 newTable[lastIdx] = lastRun;
43                 // 下面的操作是处理 lastRun 之前的节点，
44                 //    这些节点可能分配在另一个链表中，也可能分配到上面的那个链表中
45                 for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
46                     V v = p.value;
47                     int h = p.hash;
48                     int k = h & sizeMask;
49                     HashEntry<K,V> n = newTable[k];
50                     newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
51                 }
52             }
53         }
54     }
55     // 将新来的 node 放到新数组中刚刚的 两个链表之一 的 头部
56     int nodeIndex = node.hash & sizeMask; // add the new node
57     node.setNext(newTable[nodeIndex]);
58     newTable[nodeIndex] = node;
59     table = newTable;
60 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

总结一下put的流程：

当执行put操作时，会进行第一次key的hash来定位Segment的位置，如果该Segment还没有初始化，即通过CAS操作进行赋值，然后进行第二次hash操作，找到相应的HashEntry的位置，这里会利用继承过来的锁的特性，在将数据插入指定的HashEntry位置时（链表的尾端），会通过继承ReentrantLock的tryLock（）方法尝试去获取锁，如果获取成功就直接插入相应的位置，如果已经有线程获取该Segment的锁，那当前线程会以自旋的方式去继续的调用tryLock（）方法去获取锁，超过指定次数就挂起，等待唤醒。



### get 过程分析

相对于 put 来说，get 真的不要太简单。

1. 计算 hash 值，找到 segment 数组中的具体位置，或我们前面用的“槽”
2. 槽中也是一个数组，根据 hash 找到数组中具体的位置
3. 到这里是链表了，顺着链表进行查找即可

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public V get(Object key) {
 2     Segment<K,V> s; // manually integrate access methods to reduce overhead
 3     HashEntry<K,V>[] tab;
 4     // 1. hash 值
 5     int h = hash(key);
 6     long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
 7     // 2. 根据 hash 找到对应的 segment
 8     if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
 9         (tab = s.table) != null) {
10         // 3. 找到segment 内部数组相应位置的链表，遍历
11         for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
12                  (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
13              e != null; e = e.next) {
14             K k;
15             if ((k = e.key) == key || (e.hash == h && key.equals(k)))
16                 return e.value;
17         }
18     }
19     return null;
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 



### size操作

put、remove和get操作只需要关心一个Segment，而size操作需要遍历所有的Segment才能算出整个Map的大小。一个简单的方案是，先锁住所有Sgment，计算完后再解锁。但这样做，在做size操作时，不仅无法对Map进行写操作，同时也无法进行读操作，不利于对Map的并行操作。

为更好支持并发操作，ConcurrentHashMap会在不上锁的前提逐个Segment计算3次size，如果某相邻两次计算获取的所有Segment的更新次数（每个Segment都与HashMap一样通过modCount跟踪自己的修改次数，Segment每修改一次其modCount加一）相等，说明这两次计算过程中无更新操作，则这两次计算出的总size相等，可直接作为最终结果返回。如果这三次计算过程中Map有更新，则对所有Segment加锁重新计算Size。该计算方法代码如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public int size() {
 2   final Segment<K,V>[] segments = this.segments;
 3   int size;
 4   boolean overflow; // true if size overflows 32 bits
 5   long sum;         // sum of modCounts
 6   long last = 0L;   // previous sum
 7   int retries = -1; // first iteration isn't retry
 8   try {
 9     for (;;) {
10       if (retries++ == RETRIES_BEFORE_LOCK) {
11         for (int j = 0; j < segments.length; ++j)
12           ensureSegment(j).lock(); // force creation
13       }
14       sum = 0L;
15       size = 0;
16       overflow = false;
17       for (int j = 0; j < segments.length; ++j) {
18         Segment<K,V> seg = segmentAt(segments, j);
19         if (seg != null) {
20           sum += seg.modCount;
21           int c = seg.count;
22           if (c < 0 || (size += c) < 0)
23             overflow = true;
24         }
25       }
26       if (sum == last)
27         break;
28       last = sum;
29     }
30   } finally {
31     if (retries > RETRIES_BEFORE_LOCK) {
32       for (int j = 0; j < segments.length; ++j)
33         segmentAt(segments, j).unlock();
34     }
35   }
36   return overflow ? Integer.MAX_VALUE : size;
37 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

ConcurrentHashMap的Size方法是一个嵌套循环，大体逻辑如下：

1.遍历所有的Segment。

2.把Segment的元素数量累加起来。

3.把Segment的修改次数累加起来。

4.判断所有Segment的总修改次数是否大于上一次的总修改次数。如果大于，说明统计过程中有修改，重新统计，尝试次数+1；如果不是。说明没有修改，统计结束。

5.如果尝试次数超过阈值，则对每一个Segment加锁，再重新统计。

6.再次判断所有Segment的总修改次数是否大于上一次的总修改次数。由于已经加锁，次数一定和上次相等。

7.释放锁，统计结束。



### 并发问题分析

现在我们已经说完了 put 过程和 get 过程，我们可以看到 get 过程中是没有加锁的，那自然我们就需要去考虑并发问题。

添加节点的操作 put 和删除节点的操作 remove 都是要加 segment 上的独占锁的，所以它们之间自然不会有问题，我们需要考虑的问题就是 get 的时候在同一个 segment 中发生了 put 或 remove 操作。

1. put 操作的线程安全性。

   1. 初始化槽，这个我们之前就说过了，使用了 CAS 来初始化 Segment 中的数组。
   2. 添加节点到链表的操作是插入到表头的，所以，如果这个时候 get 操作在链表遍历的过程已经到了中间，是不会影响的。当然，另一个并发问题就是 get 操作在 put 之后，需要保证刚刚插入表头的节点被读取，这个依赖于 setEntryAt 方法中使用的 UNSAFE.putOrderedObject。
   3. 扩容。扩容是新创建了数组，然后进行迁移数据，最后面将 newTable 设置给属性 table。所以，如果 get 操作此时也在进行，那么也没关系，如果 get 先行，那么就是在旧的 table 上做查询操作；而 put 先行，那么 put 操作的可见性保证就是 table 使用了 volatile 关键字。

2. remove 操作的线程安全性。

   remove 操作我们没有分析源码，所以这里说的读者感兴趣的话还是需要到源码中去求实一下的。

   get 操作需要遍历链表，但是 remove 操作会"破坏"链表。

   如果 remove 破坏的节点 get 操作已经过去了，那么这里不存在任何问题。

   如果 remove 先破坏了一个节点，分两种情况考虑。 1、如果此节点是头结点，那么需要将头结点的 next 设置为数组该位置的元素，table 虽然使用了 volatile 修饰，但是 volatile 并不能提供数组内部操作的可见性保证，所以源码中使用了 UNSAFE 来操作数组，请看方法 setEntryAt。2、如果要删除的节点不是头结点，它会将要删除节点的后继节点接到前驱节点中，这里的并发保证就是 next 属性是 volatile 的。

最后我们来看看并发操作示意图

**Case1：不同Segment的并发写入**

![img](http://5b0988e595225.cdn.sohucs.com/images/20171120/5c19c5021da0475a8b5f69449f2549f0.png)

不同Segment的写入是可以并发执行的。

**Case2：同一Segment的一写一读**

![img](http://5b0988e595225.cdn.sohucs.com/images/20171120/40851a75d322440184ace228869edaa2.png)

同一Segment的写和读是可以并发执行的。

**Case3：同一Segment的并发写入**

![img](http://5b0988e595225.cdn.sohucs.com/images/20171120/dacaa8c462b74751be50a758c65baf0f.png)

Segment的写入是需要上锁的，因此对同一Segment的并发写入会被阻塞。

由此可见，ConcurrentHashMap当中每个Segment各自持有一把锁。在保证线程安全的同时降低了锁的粒度，让并发操作效率更高。

 

# [深入并发包 ConcurrentHashMap 源码解析](https://www.cnblogs.com/java-chen-hao/p/10320783.html)



**目录**

- JDK1.7的实现
  - [初始化](https://www.cnblogs.com/java-chen-hao/p/10320783.html#_label0_0)
  - [put 过程分析](https://www.cnblogs.com/java-chen-hao/p/10320783.html#_label0_1)
  - [get 过程分析](https://www.cnblogs.com/java-chen-hao/p/10320783.html#_label0_2)
  - [size操作](https://www.cnblogs.com/java-chen-hao/p/10320783.html#_label0_3)
  - [并发问题分析](https://www.cnblogs.com/java-chen-hao/p/10320783.html#_label0_4)

 

**正文**

以前写过介绍HashMap的文章，文中提到过HashMap在put的时候，插入的元素超过了容量（由负载因子决定）的范围就会触发扩容操作，就是rehash，这个会重新将原数组的内容重新hash到新的扩容数组中，在多线程的环境下，存在同时其他的元素也在进行put操作，如果hash值相同，可能出现同时在同一数组下用链表表示，造成闭环，导致在get时会出现死循环，所以HashMap是线程不安全的。

[回到顶部](https://www.cnblogs.com/java-chen-hao/p/10320783.html#_labelTop)

## JDK1.7的实现

整个 ConcurrentHashMap 由一个个 Segment 组成，Segment 代表”部分“或”一段“的意思，所以很多地方都会将其描述为分段锁。注意，行文中，我很多地方用了“槽”来代表一个 segment。

简单理解就是，ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

![img](https://img2018.cnblogs.com/blog/1168971/201901/1168971-20190125170257376-1816689764.png)

 

concurrencyLevel：并行级别、并发数、Segment 数。默认是 16，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。

再具体到每个 Segment 内部，其实每个 Segment 很像之前介绍的 HashMap，不过它要保证线程安全，所以处理起来要麻烦些。



### 初始化

initialCapacity：初始容量，这个值指的是整个 ConcurrentHashMap 的初始容量，实际操作的时候需要平均分给每个 Segment。

loadFactor：负载因子，之前我们说了，Segment 数组不可以扩容，所以这个负载因子是给每个 Segment 内部使用的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public ConcurrentHashMap(int initialCapacity,
 2                          float loadFactor, int concurrencyLevel) {
 3     if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
 4         throw new IllegalArgumentException();
 5     if (concurrencyLevel > MAX_SEGMENTS)
 6         concurrencyLevel = MAX_SEGMENTS;
 7     // Find power-of-two sizes best matching arguments
 8     int sshift = 0;
 9     int ssize = 1;
10     // 计算并行级别 ssize，因为要保持并行级别是 2 的 n 次方
11     while (ssize < concurrencyLevel) {
12         ++sshift;
13         ssize <<= 1;
14     }
15     // 我们这里先不要那么烧脑，用默认值，concurrencyLevel 为 16，sshift 为 4
16     // 那么计算出 segmentShift 为 28，segmentMask 为 15，后面会用到这两个值
17     this.segmentShift = 32 - sshift;
18     this.segmentMask = ssize - 1;
19 
20     if (initialCapacity > MAXIMUM_CAPACITY)
21         initialCapacity = MAXIMUM_CAPACITY;
22 
23     // initialCapacity 是设置整个 map 初始的大小，
24     // 这里根据 initialCapacity 计算 Segment 数组中每个位置可以分到的大小
25     // 如 initialCapacity 为 64，那么每个 Segment 或称之为"槽"可以分到 4 个
26     int c = initialCapacity / ssize;
27     if (c * ssize < initialCapacity)
28         ++c;
29     // 默认 MIN_SEGMENT_TABLE_CAPACITY 是 2，这个值也是有讲究的，因为这样的话，对于具体的槽上，
30     // 插入一个元素不至于扩容，插入第二个的时候才会扩容
31     int cap = MIN_SEGMENT_TABLE_CAPACITY; 
32     while (cap < c)
33         cap <<= 1;
34 
35     // 创建 Segment 数组，
36     // 并创建数组的第一个元素 segment[0]
37     Segment<K,V> s0 =
38         new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
39                          (HashEntry<K,V>[])new HashEntry[cap]);
40     Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
41     // 往数组写入 segment[0]
42     UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
43     this.segments = ss;
44 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

初始化完成，我们得到了一个 Segment 数组。

我们就当是用 new ConcurrentHashMap() 无参构造函数进行初始化的，那么初始化完成后：

- Segment 数组长度为 16，不可以扩容
- Segment[i] 的默认大小为 2，负载因子是 0.75，得出初始阈值为 1.5，也就是以后插入第一个元素不会触发扩容，插入第二个会进行第一次扩容
- 这里初始化了 segment[0]，其他位置还是 null，至于为什么要初始化 segment[0]，后面的代码会介绍
- 当前 segmentShift 的值为 32 - 4 = 28，segmentMask 为 16 - 1 = 15，姑且把它们简单翻译为移位数和掩码，这两个值马上就会用到

#### Segment 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 static class Segment<K,V> extends ReentrantLock implements Serializable {
2 
3     transient volatile HashEntry<K,V>[] table;
4     
5     transient int count;
6     
7     transient int modCount;
8     
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从上Segment的继承体系可以看出，Segment实现了ReentrantLock,也就带有锁的功能,table使用volatile修饰，保证了内存可见性。



### put 过程分析

我们先看 put 的主流程，对于其中的一些关键细节操作，后面会进行详细介绍。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public V put(K key, V value) {
 2     Segment<K,V> s;
 3     if (value == null)
 4         throw new NullPointerException();
 5     // 1. 计算 key 的 hash 值
 6     int hash = hash(key);
 7     // 2. 根据 hash 值找到 Segment 数组中的位置 j
 8     //    hash 是 32 位，无符号右移 segmentShift(28) 位，剩下高 4 位，
 9     //    然后和 segmentMask(15) 做一次与操作，也就是说 j 是 hash 值的高 4 位，也就是槽的数组下标
10     int j = (hash >>> segmentShift) & segmentMask;
11     // 刚刚说了，初始化的时候初始化了 segment[0]，但是其他位置还是 null，
12     // ensureSegment(j) 对 segment[j] 进行初始化
13     if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
14          (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
15         s = ensureSegment(j);
16     // 3. 插入新值到 槽 s 中
17     return s.put(key, hash, value, false);
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 初始化槽: ensureSegment

ConcurrentHashMap 初始化的时候会初始化第一个槽 segment[0]，对于其他槽来说，在插入第一个值的时候进行初始化。

这里需要考虑并发，因为很可能会有多个线程同时进来初始化同一个槽 segment[k]，不过只要有一个成功了就可以。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private Segment<K,V> ensureSegment(int k) {
 2     final Segment<K,V>[] ss = this.segments;
 3     long u = (k << SSHIFT) + SBASE; // raw offset
 4     Segment<K,V> seg;
 5     if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
 6         // 这里看到为什么之前要初始化 segment[0] 了，
 7         // 使用当前 segment[0] 处的数组长度和负载因子来初始化 segment[k]
 8         // 为什么要用“当前”，因为 segment[0] 可能早就扩容过了
 9         Segment<K,V> proto = ss[0];
10         int cap = proto.table.length;
11         float lf = proto.loadFactor;
12         int threshold = (int)(cap * lf);
13 
14         // 初始化 segment[k] 内部的数组
15         HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
16         if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
17             == null) { // 再次检查一遍该槽是否被其他线程初始化了。
18 
19             Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
20             // 使用 while 循环，内部用 CAS，当前线程成功设值或其他线程成功设值后，退出,如果其他线程成功设置后，这里获取到直接返回
21             while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
22                    == null) {
23                 if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
24                     break;
25             }
26         }
27     }
28     return seg;
29 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

总的来说，ensureSegment(int k) 比较简单，对于并发操作使用 CAS 进行控制。

第一层很简单，根据 hash 值很快就能找到相应的 Segment，之后就是 Segment 内部的 put 操作了。

#### Segment 内部是由 **数组**+**链表** 组成的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 final V put(K key, int hash, V value, boolean onlyIfAbsent) {
 2     // 在往该 segment 写入前，需要先获取该 segment 的独占锁
 3     //    先看主流程，后面还会具体介绍这部分内容
 4     HashEntry<K,V> node = tryLock() ? null :
 5         scanAndLockForPut(key, hash, value);
 6     V oldValue;
 7     try {
 8         // 这个是 segment 内部的数组
 9         HashEntry<K,V>[] tab = table;
10         // 再利用 hash 值，求应该放置的数组下标
11         int index = (tab.length - 1) & hash;
12         // first 是数组该位置处的链表的表头
13         HashEntry<K,V> first = entryAt(tab, index);
14 
15         // 下面这串 for 循环虽然很长，不过也很好理解，想想该位置没有任何元素和已经存在一个链表这两种情况
16         for (HashEntry<K,V> e = first;;) {
17             if (e != null) {
18                 K k;
19                 if ((k = e.key) == key ||
20                     (e.hash == hash && key.equals(k))) {
21                     oldValue = e.value;
22                     if (!onlyIfAbsent) {
23                         // 覆盖旧值
24                         e.value = value;
25                         ++modCount;
26                     }
27                     break;
28                 }
29                 // 继续顺着链表走
30                 e = e.next;
31             }
32             else {
33                 // node 到底是不是 null，这个要看获取锁的过程，不过和这里都没有关系。
34                 // 如果不为 null，那就直接将它设置为链表表头；如果是null，初始化并设置为链表表头。
35                 if (node != null)
36                     node.setNext(first);
37                 else
38                     node = new HashEntry<K,V>(hash, key, value, first);
39 
40                 int c = count + 1;
41                 // 如果超过了该 segment 的阈值，这个 segment 需要扩容
42                 if (c > threshold && tab.length < MAXIMUM_CAPACITY)
43                     rehash(node); // 扩容后面也会具体分析
44                 else
45                     // 没有达到阈值，将 node 放到数组 tab 的 index 位置，
46                     // 其实就是将新的节点设置成原链表的表头
47                     setEntryAt(tab, index, node);
48                 ++modCount;
49                 count = c;
50                 oldValue = null;
51                 break;
52             }
53         }
54     } finally {
55         // 解锁
56         unlock();
57     }
58     return oldValue;
59 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

整体流程还是比较简单的，由于有独占锁的保护，所以 segment 内部的操作并不复杂。至于这里面的并发问题，我们稍后再进行介绍。

到这里 put 操作就结束了，接下来，我们说一说其中几步关键的操作。

#### 获取写入锁: scanAndLockForPut

前面我们看到，在往某个 segment 中 put 的时候，首先会调用 node = tryLock() ? null : scanAndLockForPut(key, hash, value)，也就是说先进行一次 tryLock() 快速获取该 segment 的独占锁，如果失败，那么进入到 scanAndLockForPut 这个方法来获取锁。

下面我们来具体分析这个方法中是怎么控制加锁的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
 2     HashEntry<K,V> first = entryForHash(this, hash);
 3     HashEntry<K,V> e = first;
 4     HashEntry<K,V> node = null;
 5     int retries = -1; // negative while locating node
 6 
 7     // 循环获取锁
 8     while (!tryLock()) {
 9         HashEntry<K,V> f; // to recheck first below
10         if (retries < 0) {
11             if (e == null) {
12                 if (node == null) // speculatively create node
13                     // 进到这里说明数组该位置的链表是空的，没有任何元素
14                     // 当然，进到这里的另一个原因是 tryLock() 失败，所以该槽存在并发，不一定是该位置
15                     node = new HashEntry<K,V>(hash, key, value, null);
16                 retries = 0;
17             }
18             else if (key.equals(e.key))
19                 retries = 0;
20             else
21                 // 顺着链表往下走
22                 e = e.next;
23         }
24         // 重试次数如果超过 MAX_SCAN_RETRIES（单核1多核64），那么不抢了，进入到阻塞队列等待锁
25         //    lock() 是阻塞方法，直到获取锁后返回
26         else if (++retries > MAX_SCAN_RETRIES) {
27             lock();
28             break;
29         }
30         else if ((retries & 1) == 0 &&
31                  // 这个时候是有大问题了，那就是有新的元素进到了链表，成为了新的表头
32                  //     所以这边的策略是，相当于重新走一遍这个 scanAndLockForPut 方法
33                  (f = entryForHash(this, hash)) != first) {
34             e = first = f; // re-traverse if entry changed
35             retries = -1;
36         }
37     }
38     return node;
39 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这个方法有两个出口，一个是 tryLock() 成功了，循环终止，另一个就是重试次数超过了 MAX_SCAN_RETRIES，进到 lock() 方法，此方法会阻塞等待，直到成功拿到独占锁。

这个方法就是看似复杂，但是其实就是做了一件事，那就是获取该 segment 的独占锁，如果需要的话顺便实例化了一下 node。

获取锁时，并不直接使用lock来获取，因为该方法获取锁失败时会挂起。事实上，它使用了自旋锁，如果tryLock获取锁失败，说明锁被其它线程占用，此时通过循环再次以tryLock的方式申请锁。如果在循环过程中该Key所对应的链表头被修改，则重置retry次数。如果retry次数超过一定值，则使用lock方法申请锁。

这里使用自旋锁是因为自旋锁的效率比较高，但是它消耗CPU资源比较多，因此在自旋次数超过阈值时切换为互斥锁。

#### 扩容: rehash

重复一下，segment 数组不能扩容，扩容是 segment 数组某个位置内部的数组 HashEntry\<k,v>[] 进行扩容，扩容后，容量为原来的 2 倍。

首先，我们要回顾一下触发扩容的地方，put 的时候，如果判断该值的插入会导致该 segment 的元素个数超过阈值，那么先进行扩容，再插值，读者这个时候可以回去 put 方法看一眼。

该方法不需要考虑并发，因为到这里的时候，是持有该 segment 的独占锁的。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 // 方法参数上的 node 是这次扩容后，需要添加到新的数组中的数据。
 2 private void rehash(HashEntry<K,V> node) {
 3     HashEntry<K,V>[] oldTable = table;
 4     int oldCapacity = oldTable.length;
 5     // 2 倍
 6     int newCapacity = oldCapacity << 1;
 7     threshold = (int)(newCapacity * loadFactor);
 8     // 创建新数组
 9     HashEntry<K,V>[] newTable =
10         (HashEntry<K,V>[]) new HashEntry[newCapacity];
11     // 新的掩码，如从 16 扩容到 32，那么 sizeMask 为 31，对应二进制 ‘000...00011111’
12     int sizeMask = newCapacity - 1;
13 
14     // 遍历原数组，老套路，将原数组位置 i 处的链表拆分到 新数组位置 i 和 i+oldCap 两个位置
15     for (int i = 0; i < oldCapacity ; i++) {
16         // e 是链表的第一个元素
17         HashEntry<K,V> e = oldTable[i];
18         if (e != null) {
19             HashEntry<K,V> next = e.next;
20             // 计算应该放置在新数组中的位置，
21             // 假设原数组长度为 16，e 在 oldTable[3] 处，那么 idx 只可能是 3 或者是 3 + 16 = 19
22             int idx = e.hash & sizeMask;
23             if (next == null)   // 该位置处只有一个元素，那比较好办
24                 newTable[idx] = e;
25             else { // Reuse consecutive sequence at same slot
26                 // e 是链表表头
27                 HashEntry<K,V> lastRun = e;
28                 // idx 是当前链表的头结点 e 的新位置
29                 int lastIdx = idx;
30 
31                 // 下面这个 for 循环会找到一个 lastRun 节点，这个节点之后的所有元素是将要放到一起的
32                 for (HashEntry<K,V> last = next;
33                      last != null;
34                      last = last.next) {
35                     int k = last.hash & sizeMask;
36                     if (k != lastIdx) {
37                         lastIdx = k;
38                         lastRun = last;
39                     }
40                 }
41                 // 将 lastRun 及其之后的所有节点组成的这个链表放到 lastIdx 这个位置
42                 newTable[lastIdx] = lastRun;
43                 // 下面的操作是处理 lastRun 之前的节点，
44                 //    这些节点可能分配在另一个链表中，也可能分配到上面的那个链表中
45                 for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
46                     V v = p.value;
47                     int h = p.hash;
48                     int k = h & sizeMask;
49                     HashEntry<K,V> n = newTable[k];
50                     newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
51                 }
52             }
53         }
54     }
55     // 将新来的 node 放到新数组中刚刚的 两个链表之一 的 头部
56     int nodeIndex = node.hash & sizeMask; // add the new node
57     node.setNext(newTable[nodeIndex]);
58     newTable[nodeIndex] = node;
59     table = newTable;
60 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

总结一下put的流程：

当执行put操作时，会进行第一次key的hash来定位Segment的位置，如果该Segment还没有初始化，即通过CAS操作进行赋值，然后进行第二次hash操作，找到相应的HashEntry的位置，这里会利用继承过来的锁的特性，在将数据插入指定的HashEntry位置时（链表的尾端），会通过继承ReentrantLock的tryLock（）方法尝试去获取锁，如果获取成功就直接插入相应的位置，如果已经有线程获取该Segment的锁，那当前线程会以自旋的方式去继续的调用tryLock（）方法去获取锁，超过指定次数就挂起，等待唤醒。



### get 过程分析

相对于 put 来说，get 真的不要太简单。

1. 计算 hash 值，找到 segment 数组中的具体位置，或我们前面用的“槽”
2. 槽中也是一个数组，根据 hash 找到数组中具体的位置
3. 到这里是链表了，顺着链表进行查找即可

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public V get(Object key) {
 2     Segment<K,V> s; // manually integrate access methods to reduce overhead
 3     HashEntry<K,V>[] tab;
 4     // 1. hash 值
 5     int h = hash(key);
 6     long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
 7     // 2. 根据 hash 找到对应的 segment
 8     if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
 9         (tab = s.table) != null) {
10         // 3. 找到segment 内部数组相应位置的链表，遍历
11         for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
12                  (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
13              e != null; e = e.next) {
14             K k;
15             if ((k = e.key) == key || (e.hash == h && key.equals(k)))
16                 return e.value;
17         }
18     }
19     return null;
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)



### size操作

put、remove和get操作只需要关心一个Segment，而size操作需要遍历所有的Segment才能算出整个Map的大小。一个简单的方案是，先锁住所有Sgment，计算完后再解锁。但这样做，在做size操作时，不仅无法对Map进行写操作，同时也无法进行读操作，不利于对Map的并行操作。

为更好支持并发操作，ConcurrentHashMap会在不上锁的前提逐个Segment计算3次size，如果某相邻两次计算获取的所有Segment的更新次数（每个Segment都与HashMap一样通过modCount跟踪自己的修改次数，Segment每修改一次其modCount加一）相等，说明这两次计算过程中无更新操作，则这两次计算出的总size相等，可直接作为最终结果返回。如果这三次计算过程中Map有更新，则对所有Segment加锁重新计算Size。该计算方法代码如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public int size() {
 2   final Segment<K,V>[] segments = this.segments;
 3   int size;
 4   boolean overflow; // true if size overflows 32 bits
 5   long sum;         // sum of modCounts
 6   long last = 0L;   // previous sum
 7   int retries = -1; // first iteration isn't retry
 8   try {
 9     for (;;) {
10       if (retries++ == RETRIES_BEFORE_LOCK) {
11         for (int j = 0; j < segments.length; ++j)
12           ensureSegment(j).lock(); // force creation
13       }
14       sum = 0L;
15       size = 0;
16       overflow = false;
17       for (int j = 0; j < segments.length; ++j) {
18         Segment<K,V> seg = segmentAt(segments, j);
19         if (seg != null) {
20           sum += seg.modCount;
21           int c = seg.count;
22           if (c < 0 || (size += c) < 0)
23             overflow = true;
24         }
25       }
26       if (sum == last)
27         break;
28       last = sum;
29     }
30   } finally {
31     if (retries > RETRIES_BEFORE_LOCK) {
32       for (int j = 0; j < segments.length; ++j)
33         segmentAt(segments, j).unlock();
34     }
35   }
36   return overflow ? Integer.MAX_VALUE : size;
37 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

ConcurrentHashMap的Size方法是一个嵌套循环，大体逻辑如下：

1.遍历所有的Segment。

2.把Segment的元素数量累加起来。

3.把Segment的修改次数累加起来。

4.判断所有Segment的总修改次数是否大于上一次的总修改次数。如果大于，说明统计过程中有修改，重新统计，尝试次数+1；如果不是。说明没有修改，统计结束。

5.如果尝试次数超过阈值，则对每一个Segment加锁，再重新统计。

6.再次判断所有Segment的总修改次数是否大于上一次的总修改次数。由于已经加锁，次数一定和上次相等。

7.释放锁，统计结束。



### 并发问题分析

现在我们已经说完了 put 过程和 get 过程，我们可以看到 get 过程中是没有加锁的，那自然我们就需要去考虑并发问题。

添加节点的操作 put 和删除节点的操作 remove 都是要加 segment 上的独占锁的，所以它们之间自然不会有问题，我们需要考虑的问题就是 get 的时候在同一个 segment 中发生了 put 或 remove 操作。

1. put 操作的线程安全性。

   1. 初始化槽，这个我们之前就说过了，使用了 CAS 来初始化 Segment 中的数组。
   2. 添加节点到链表的操作是插入到表头的，所以，如果这个时候 get 操作在链表遍历的过程已经到了中间，是不会影响的。当然，另一个并发问题就是 get 操作在 put 之后，需要保证刚刚插入表头的节点被读取，这个依赖于 setEntryAt 方法中使用的 UNSAFE.putOrderedObject。
   3. 扩容。扩容是新创建了数组，然后进行迁移数据，最后面将 newTable 设置给属性 table。所以，如果 get 操作此时也在进行，那么也没关系，如果 get 先行，那么就是在旧的 table 上做查询操作；而 put 先行，那么 put 操作的可见性保证就是 table 使用了 volatile 关键字。

2. remove 操作的线程安全性。

   remove 操作我们没有分析源码，所以这里说的读者感兴趣的话还是需要到源码中去求实一下的。

   get 操作需要遍历链表，但是 remove 操作会"破坏"链表。

   如果 remove 破坏的节点 get 操作已经过去了，那么这里不存在任何问题。

   如果 remove 先破坏了一个节点，分两种情况考虑。 1、如果此节点是头结点，那么需要将头结点的 next 设置为数组该位置的元素，table 虽然使用了 volatile 修饰，但是 volatile 并不能提供数组内部操作的可见性保证，所以源码中使用了 UNSAFE 来操作数组，请看方法 setEntryAt。2、如果要删除的节点不是头结点，它会将要删除节点的后继节点接到前驱节点中，这里的并发保证就是 next 属性是 volatile 的。

最后我们来看看并发操作示意图

**Case1：不同Segment的并发写入**

![img](http://5b0988e595225.cdn.sohucs.com/images/20171120/5c19c5021da0475a8b5f69449f2549f0.png)

不同Segment的写入是可以并发执行的。

**Case2：同一Segment的一写一读**

![img](http://5b0988e595225.cdn.sohucs.com/images/20171120/40851a75d322440184ace228869edaa2.png)

同一Segment的写和读是可以并发执行的。

**Case3：同一Segment的并发写入**

![img](http://5b0988e595225.cdn.sohucs.com/images/20171120/dacaa8c462b74751be50a758c65baf0f.png)

Segment的写入是需要上锁的，因此对同一Segment的并发写入会被阻塞。

由此可见，ConcurrentHashMap当中每个Segment各自持有一把锁。在保证线程安全的同时降低了锁的粒度，让并发操作效率更高。

# Java 并发

<!-- GFM-TOC -->

* [Java 并发](#java-并发)
  * [一、使用线程](#一使用线程)
    * [实现 Runnable 接口](#实现-runnable-接口)
    * [实现 Callable 接口](#实现-callable-接口)
    * [继承 Thread 类](#继承-thread-类)
    * [实现接口 VS 继承 Thread](#实现接口-vs-继承-thread)
  * [二、基础线程机制](#二基础线程机制)
    * [Executor](#executor)
    * [Daemon](#daemon)
    * [sleep()](#sleep)
    * [yield()](#yield)
  * [三、中断](#三中断)
    * [InterruptedException](#interruptedexception)
    * [interrupted()](#interrupted)
    * [Executor 的中断操作](#executor-的中断操作)
  * [四、互斥同步](#四互斥同步)
    * [synchronized](#synchronized)
    * [ReentrantLock](#reentrantlock)
    * [比较](#比较)
    * [使用选择](#使用选择)
  * [五、线程之间的协作](#五线程之间的协作)
    * [join()](#join)
    * [wait() notify() notifyAll()](#wait-notify-notifyall)
    * [await() signal() signalAll()](#await-signal-signalall)
  * [六、线程状态](#六线程状态)
    * [新建（NEW）](#新建new)
    * [可运行（RUNABLE）](#可运行runable)
    * [阻塞（BLOCKED）](#阻塞blocked)
    * [无限期等待（WAITING）](#无限期等待waiting)
    * [限期等待（TIMED_WAITING）](#限期等待timed_waiting)
    * [死亡（TERMINATED）](#死亡terminated)
  * [七、J.U.C - AQS](#七juc---aqs)
    * [CountDownLatch](#countdownlatch)
    * [CyclicBarrier](#cyclicbarrier)
    * [Semaphore](#semaphore)
  * [八、J.U.C - 其它组件](#八juc---其它组件)
    * [FutureTask](#futuretask)
    * [BlockingQueue](#blockingqueue)
    * [ForkJoin](#forkjoin)
  * [九、线程不安全示例](#九线程不安全示例)
  * [十、Java 内存模型](#十java-内存模型)
    * [主内存与工作内存](#主内存与工作内存)
    * [内存间交互操作](#内存间交互操作)
    * [内存模型三大特性](#内存模型三大特性)
    * [先行发生原则](#先行发生原则)
  * [十一、线程安全](#十一线程安全)
    * [不可变](#不可变)
    * [互斥同步](#互斥同步)
    * [非阻塞同步](#非阻塞同步)
    * [无同步方案](#无同步方案)
  * [十二、锁优化](#十二锁优化)
    * [自旋锁](#自旋锁)
    * [锁消除](#锁消除)
    * [锁粗化](#锁粗化)
    * [轻量级锁](#轻量级锁)
    * [偏向锁](#偏向锁)
  * [十三、多线程开发良好的实践](#十三多线程开发良好的实践)
  * [参考资料](#参考资料)
    <!-- GFM-TOC -->



## 一、使用线程

有三种使用线程的方法：

- 实现 Runnable 接口；
- 实现 Callable 接口；
- 继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以理解为任务是通过线程驱动从而执行的。

### 实现 Runnable 接口

需要实现接口中的 run() 方法。

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        // ...
    }
}
```

使用 Runnable 实例再创建一个 Thread 实例，然后调用 Thread 实例的 start() 方法来启动线程。

```java
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```

### 实现 Callable 接口

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

```java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
```

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```

### 继承 Thread 类

同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口。

当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

```java
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}
```

```java
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```

### 实现接口 VS 继承 Thread

实现接口会更好一些，因为：

- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

## 二、基础线程机制

### Executor

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor：

- CachedThreadPool：一个任务创建一个线程；
- FixedThreadPool：所有任务只能使用固定大小的线程；
- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 5; i++) {
        executorService.execute(new MyRunnable());
    }
    executorService.shutdown();
}
```

### Daemon

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

main() 属于非守护线程。

在线程启动之前使用 setDaemon() 方法可以将一个线程设置为守护线程。

```java
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```

### sleep()

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

```java
public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

### yield()

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

```java
public void run() {
    Thread.yield();
}
```

## 三、中断

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

### InterruptedException

通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

```java
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new MyThread1();
    thread1.start();
    thread1.interrupt();
    System.out.println("Main run");
}
```

```html
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

### interrupted()

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

```java
public class InterruptExample {

    private static class MyThread2 extends Thread {
        @Override
        public void run() {
            while (!interrupted()) {
                // ..
            }
            System.out.println("Thread end");
        }
    }
}
```

```java
public static void main(String[] args) throws InterruptedException {
    Thread thread2 = new MyThread2();
    thread2.start();
    thread2.interrupt();
}
```

```html
Thread end
```

### Executor 的中断操作

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

以下使用 Lambda 创建线程，相当于创建了一个匿名内部线程。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> {
        try {
            Thread.sleep(2000);
            System.out.println("Thread run");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    executorService.shutdownNow();
    System.out.println("Main run");
}
```

```html
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at ExecutorInterruptExample.lambda$main$0(ExecutorInterruptExample.java:9)
    at ExecutorInterruptExample$$Lambda$1/1160460865.run(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)
```

如果只想中断 Executor 中的一个线程，可以通过使用 submit() 方法来提交一个线程，它会返回一个 Future\<?\> 对象，通过调用该对象的 cancel(true) 方法就可以中断线程。

```java
Future<?> future = executorService.submit(() -> {
    // ..
});
future.cancel(true);
```

## 四、互斥同步

Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

### synchronized

**1. 同步一个代码块**  

```java
public void func() {
    synchronized (this) {
        // ...
    }
}
```

它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

```java
public class SynchronizedExample {

    public void func1() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

```java
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e1.func1());
}
```

```html
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

对于以下代码，两个线程调用了不同对象的同步代码块，因此这两个线程就不需要同步。从输出结果可以看出，两个线程交叉执行。

```java
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e2.func1());
}
```

```html
0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
```


**2. 同步一个方法**  

```java
public synchronized void func () {
    // ...
}
```

它和同步代码块一样，作用于同一个对象。

**3. 同步一个类**  

```java
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

```java
public class SynchronizedExample {

    public void func2() {
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

```java
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func2());
    executorService.execute(() -> e2.func2());
}
```

```html
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

**4. 同步一个静态方法**  

```java
public synchronized static void fun() {
    // ...
}
```

作用于整个类。

### ReentrantLock

ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

```java
public class LockExample {

    private Lock lock = new ReentrantLock();

    public void func() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        } finally {
            lock.unlock(); // 确保释放锁，从而避免发生死锁。
        }
    }
}
```

```java
public static void main(String[] args) {
    LockExample lockExample = new LockExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> lockExample.func());
    executorService.execute(() -> lockExample.func());
}
```

```html
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```


### 比较

**1. 锁的实现**  

synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。

**2. 性能**  

新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

**3. 等待可中断**  

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

**4. 公平锁**  

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

**5. 锁绑定多个条件**  

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

### 使用选择

除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

## 五、线程之间的协作

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

### join()

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。

```java
public class JoinExample {

    private class A extends Thread {
        @Override
        public void run() {
            System.out.println("A");
        }
    }

    private class B extends Thread {

        private A a;

        B(A a) {
            this.a = a;
        }

        @Override
        public void run() {
            try {
                a.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("B");
        }
    }

    public void test() {
        A a = new A();
        B b = new B(a);
        b.start();
        a.start();
    }
}
```

```java
public static void main(String[] args) {
    JoinExample example = new JoinExample();
    example.test();
}
```

```
A
B
```

### wait() notify() notifyAll()

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

```java
public class WaitNotifyExample {

    public synchronized void before() {
        System.out.println("before");
        notifyAll();
    }

    public synchronized void after() {
        try {
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("after");
    }
}
```

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    WaitNotifyExample example = new WaitNotifyExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
```

```html
before
after
```

**wait() 和 sleep() 的区别**  

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。

### await() signal() signalAll()

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

使用 Lock 来获取一个 Condition 对象。

```java
public class AwaitSignalExample {

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void before() {
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    AwaitSignalExample example = new AwaitSignalExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
```

```html
before
after
```

## 六、线程状态

一个线程只能处于一种状态，并且这里的线程状态特指 Java 虚拟机的线程状态，不能反映线程在特定操作系统下的状态。

### 新建（NEW）

创建后尚未启动。

### 可运行（RUNABLE）

正在 Java 虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就进入运行状态。所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。

### 阻塞（BLOCKED）

请求获取 monitor lock 从而进入 synchronized 函数或者代码块，但是其它线程已经占用了该 monitor lock，所以出于阻塞状态。要结束该状态进入从而 RUNABLE 需要其他线程释放 monitor lock。

### 无限期等待（WAITING）

等待其它线程显式地唤醒。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取 monitor lock。而等待是主动的，通过调用  Object.wait() 等方法进入。

| 进入方法                                   | 退出方法                             |
| ------------------------------------------ | ------------------------------------ |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法                    | LockSupport.unpark(Thread)           |

### 限期等待（TIMED_WAITING）

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

| 进入方法                                 | 退出方法                                        |
| ---------------------------------------- | ----------------------------------------------- |
| Thread.sleep() 方法                      | 时间结束                                        |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
| LockSupport.parkNanos() 方法             | LockSupport.unpark(Thread)                      |
| LockSupport.parkUntil() 方法             | LockSupport.unpark(Thread)                      |

调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

### 死亡（TERMINATED）

可以是线程结束任务之后自己结束，或者产生了异常而结束。

[Java SE 9 Enum Thread.State](https://docs.oracle.com/javase/9/docs/api/java/lang/Thread.State.html)

## 七、J.U.C - AQS

java.util.concurrent（J.U.C）大大提高了并发性能，AQS 被认为是 J.U.C 的核心。

### CountDownLatch

用来控制一个或者多个线程等待多个线程。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/ba078291-791e-4378-b6d1-ece76c2f0b14.png" width="300px"> </div><br>

```java
public class CountdownLatchExample {

    public static void main(String[] args) throws InterruptedException {
        final int totalThread = 10;
        CountDownLatch countDownLatch = new CountDownLatch(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("run..");
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        System.out.println("end");
        executorService.shutdown();
    }
}
```

```html
run..run..run..run..run..run..run..run..run..run..end
```

### CyclicBarrier

用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

CyclicBarrier 有两个构造函数，其中 parties 指示计数器的初始值，barrierAction 在所有线程都到达屏障的时候会执行一次。

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/f71af66b-0d54-4399-a44b-f47b58321984.png" width="300px"> </div><br>

```java
public class CyclicBarrierExample {

    public static void main(String[] args) {
        final int totalThread = 10;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("before..");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.print("after..");
            });
        }
        executorService.shutdown();
    }
}
```

```html
before..before..before..before..before..before..before..before..before..before..after..after..after..after..after..after..after..after..after..after..
```

### Semaphore

Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

以下代码模拟了对某个服务的并发请求，每次只能有 3 个客户端同时访问，请求总数为 10。

```java
public class SemaphoreExample {

    public static void main(String[] args) {
        final int clientCount = 3;
        final int totalRequestCount = 10;
        Semaphore semaphore = new Semaphore(clientCount);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalRequestCount; i++) {
            executorService.execute(()->{
                try {
                    semaphore.acquire();
                    System.out.print(semaphore.availablePermits() + " ");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            });
        }
        executorService.shutdown();
    }
}
```

```html
2 1 2 2 2 2 2 1 2 2
```

## 八、J.U.C - 其它组件

### FutureTask

在介绍 Callable 时我们知道它可以有返回值，返回值通过 Future\<V\> 进行封装。FutureTask 实现了 RunnableFuture 接口，该接口继承自 Runnable 和 Future\<V\> 接口，这使得 FutureTask 既可以当做一个任务执行，也可以有返回值。

```java
public class FutureTask<V> implements RunnableFuture<V>
```

```java
public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask 可用于异步获取执行结果或取消执行任务的场景。当一个计算任务需要执行很长时间，那么就可以用 FutureTask 来封装这个任务，主线程在完成自己的任务之后再去获取结果。

```java
public class FutureTaskExample {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int result = 0;
                for (int i = 0; i < 100; i++) {
                    Thread.sleep(10);
                    result += i;
                }
                return result;
            }
        });

        Thread computeThread = new Thread(futureTask);
        computeThread.start();

        Thread otherThread = new Thread(() -> {
            System.out.println("other task is running...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        otherThread.start();
        System.out.println(futureTask.get());
    }
}
```

```html
other task is running...
4950
```

### BlockingQueue

java.util.concurrent.BlockingQueue 接口有以下阻塞队列的实现：

-   **FIFO 队列**  ：LinkedBlockingQueue、ArrayBlockingQueue（固定长度）
-   **优先级队列**  ：PriorityBlockingQueue

提供了阻塞的 take() 和 put() 方法：如果队列为空 take() 将阻塞，直到队列中有内容；如果队列为满 put() 将阻塞，直到队列有空闲位置。

**使用 BlockingQueue 实现生产者消费者问题**  

```java
public class ProducerConsumer {

    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce..");
        }
    }

    private static class Consumer extends Thread {

        @Override
        public void run() {
            try {
                String product = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consume..");
        }
    }
}
```

```java
public static void main(String[] args) {
    for (int i = 0; i < 2; i++) {
        Producer producer = new Producer();
        producer.start();
    }
    for (int i = 0; i < 5; i++) {
        Consumer consumer = new Consumer();
        consumer.start();
    }
    for (int i = 0; i < 3; i++) {
        Producer producer = new Producer();
        producer.start();
    }
}
```

```html
produce..produce..consume..consume..produce..consume..produce..consume..produce..consume..
```

### ForkJoin

主要用于并行计算中，和 MapReduce 原理类似，都是把大的计算任务拆分成多个小任务并行计算。

```java
public class ForkJoinExample extends RecursiveTask<Integer> {

    private final int threshold = 5;
    private int first;
    private int last;

    public ForkJoinExample(int first, int last) {
        this.first = first;
        this.last = last;
    }

    @Override
    protected Integer compute() {
        int result = 0;
        if (last - first <= threshold) {
            // 任务足够小则直接计算
            for (int i = first; i <= last; i++) {
                result += i;
            }
        } else {
            // 拆分成小任务
            int middle = first + (last - first) / 2;
            ForkJoinExample leftTask = new ForkJoinExample(first, middle);
            ForkJoinExample rightTask = new ForkJoinExample(middle + 1, last);
            leftTask.fork();
            rightTask.fork();
            result = leftTask.join() + rightTask.join();
        }
        return result;
    }
}
```

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ForkJoinExample example = new ForkJoinExample(1, 10000);
    ForkJoinPool forkJoinPool = new ForkJoinPool();
    Future result = forkJoinPool.submit(example);
    System.out.println(result.get());
}
```

ForkJoin 使用 ForkJoinPool 来启动，它是一个特殊的线程池，线程数量取决于 CPU 核数。

```java
public class ForkJoinPool extends AbstractExecutorService
```

ForkJoinPool 实现了工作窃取算法来提高 CPU 的利用率。每个线程都维护了一个双端队列，用来存储需要执行的任务。工作窃取算法允许空闲的线程从其它线程的双端队列中窃取一个任务来执行。窃取的任务必须是最晚的任务，避免和队列所属线程发生竞争。例如下图中，Thread2 从 Thread1 的队列中拿出最晚的 Task1 任务，Thread1 会拿出 Task2 来执行，这样就避免发生竞争。但是如果队列中只有一个任务时还是会发生竞争。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/e42f188f-f4a9-4e6f-88fc-45f4682072fb.png" width="300px"> </div><br>

## 九、线程不安全示例

如果多个线程对同一个共享数据进行访问而不采取同步操作的话，那么操作的结果是不一致的。

以下代码演示了 1000 个线程同时对 cnt 执行自增操作，操作结束之后它的值有可能小于 1000。

```java
public class ThreadUnsafeExample {

    private int cnt = 0;

    public void add() {
        cnt++;
    }

    public int get() {
        return cnt;
    }
}
```

```java
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    ThreadUnsafeExample example = new ThreadUnsafeExample();
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```

```html
997
```

## 十、Java 内存模型

Java 内存模型试图屏蔽各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。

### 主内存与工作内存

处理器上的寄存器的读写的速度比内存快几个数量级，为了解决这种速度矛盾，在它们之间加入了高速缓存。

加入高速缓存带来了一个新的问题：缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些协议来解决这个问题。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/942ca0d2-9d5c-45a4-89cb-5fd89b61913f.png" width="600px"> </div><br>

所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/15851555-5abc-497d-ad34-efed10f43a6b.png" width="600px"> </div><br>

### 内存间交互操作

Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/8b7ebbad-9604-4375-84e3-f412099d170c.png" width="450px"> </div><br>

- read：把一个变量的值从主内存传输到工作内存中
- load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
- use：把工作内存中一个变量的值传递给执行引擎
- assign：把一个从执行引擎接收到的值赋给工作内存的变量
- store：把工作内存的一个变量的值传送到主内存中
- write：在 store 之后执行，把 store 得到的值放入主内存的变量中
- lock：作用于主内存的变量
- unlock

### 内存模型三大特性

#### 1. 原子性

Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的。但是 Java 内存模型允许虚拟机将没有被 volatile 修饰的 64 位数据（long，double）的读写操作划分为两次 32 位的操作来进行，即 load、store、read 和 write 操作可以不具备原子性。

有一个错误认识就是，int 等原子性的类型在多线程环境中不会出现线程安全问题。前面的线程不安全示例代码中，cnt 属于 int 类型变量，1000 个线程对它进行自增操作之后，得到的值为 997 而不是 1000。

为了方便讨论，将内存间的交互操作简化为 3 个：load、assign、store。

下图演示了两个线程同时对 cnt 进行操作，load、assign、store 这一系列操作整体上看不具备原子性，那么在 T1 修改 cnt 并且还没有将修改后的值写入主内存，T2 依然可以读入旧值。可以看出，这两个线程虽然执行了两次自增运算，但是主内存中 cnt 的值最后为 1 而不是 2。因此对 int 类型读写操作满足原子性只是说明 load、assign、store 这些单个操作具备原子性。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/2797a609-68db-4d7b-8701-41ac9a34b14f.jpg" width="300px"> </div><br>

AtomicInteger 能保证多个线程修改的原子性。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/dd563037-fcaa-4bd8-83b6-b39d93a12c77.jpg" width="300px"> </div><br>

使用 AtomicInteger 重写之前线程不安全的代码之后得到以下线程安全实现：

```java
public class AtomicExample {
    private AtomicInteger cnt = new AtomicInteger();

    public void add() {
        cnt.incrementAndGet();
    }

    public int get() {
        return cnt.get();
    }
}
```

```java
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    AtomicExample example = new AtomicExample(); // 只修改这条语句
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```

```html
1000
```

除了使用原子类之外，也可以使用 synchronized 互斥锁来保证操作的原子性。它对应的内存间交互操作为：lock 和 unlock，在虚拟机实现上对应的字节码指令为 monitorenter 和 monitorexit。

```java
public class AtomicSynchronizedExample {
    private int cnt = 0;

    public synchronized void add() {
        cnt++;
    }

    public synchronized int get() {
        return cnt;
    }
}
```

```java
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    AtomicSynchronizedExample example = new AtomicSynchronizedExample();
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```

```html
1000
```

#### 2. 可见性

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

主要有三种实现可见性的方式：

- volatile
- synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
- final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。

对前面的线程不安全示例中的 cnt 变量使用 volatile 修饰，不能解决线程不安全问题，因为 volatile 并不能保证操作的原子性。

#### 3. 有序性

有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

### 先行发生原则

上面提到了可以用 volatile 和 synchronized 来保证有序性。除此之外，JVM 还规定了先行发生原则，让一个操作无需控制就能先于另一个操作完成。

#### 1. 单一线程原则

> Single Thread rule

在一个线程内，在程序前面的操作先行发生于后面的操作。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/874b3ff7-7c5c-4e7a-b8ab-a82a3e038d20.png" width="180px"> </div><br>

#### 2. 管程锁定规则

> Monitor Lock Rule

一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/8996a537-7c4a-4ec8-a3b7-7ef1798eae26.png" width="350px"> </div><br>

#### 3. volatile 变量规则

> Volatile Variable Rule

对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/942f33c9-8ad9-4987-836f-007de4c21de0.png" width="400px"> </div><br>

#### 4. 线程启动规则

> Thread Start Rule

Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/6270c216-7ec0-4db7-94de-0003bce37cd2.png" width="380px"> </div><br>

#### 5. 线程加入规则

> Thread Join Rule

Thread 对象的结束先行发生于 join() 方法返回。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/233f8d89-31d7-413f-9c02-042f19c46ba1.png" width="400px"> </div><br>

#### 6. 线程中断规则

> Thread Interruption Rule

对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生。

#### 7. 对象终结规则

> Finalizer Rule

一个对象的初始化完成（构造函数执行结束）先行发生于它的 finalize() 方法的开始。

#### 8. 传递性

> Transitivity

如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。

## 十一、线程安全

多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

线程安全有以下几种实现方式：

### 不可变

不可变（Immutable）的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

不可变的类型：

- final 关键字修饰的基本数据类型
- String
- 枚举类型
- Number 部分子类，如 Long 和 Double 等数值包装类型，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。

对于集合类型，可以使用 Collections.unmodifiableXXX() 方法来获取一个不可变的集合。

```java
public class ImmutableExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(map);
        unmodifiableMap.put("a", 1);
    }
}
```

```html
Exception in thread "main" java.lang.UnsupportedOperationException
    at java.util.Collections$UnmodifiableMap.put(Collections.java:1457)
    at ImmutableExample.main(ImmutableExample.java:9)
```

Collections.unmodifiableXXX() 先对原始的集合进行拷贝，需要对集合进行修改的方法都直接抛出异常。

```java
public V put(K key, V value) {
    throw new UnsupportedOperationException();
}
```

### 互斥同步

synchronized 和 ReentrantLock。

### 非阻塞同步

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

#### 1. CAS

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

#### 2. AtomicInteger

J.U.C 包里面的整数原子类 AtomicInteger 的方法调用了 Unsafe 类的 CAS 操作。

以下代码使用了 AtomicInteger 执行了自增的操作。

```java
private AtomicInteger cnt = new AtomicInteger();

public void add() {
    cnt.incrementAndGet();
}
```

以下代码是 incrementAndGet() 的源码，它调用了 Unsafe 的 getAndAddInt() 。

```java
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```

以下代码是 getAndAddInt() 源码，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。通过 getIntVolatile(var1, var2) 得到旧的预期值，通过调用 compareAndSwapInt() 来进行 CAS 比较，如果该字段内存地址中的值等于 var5，那么就更新内存地址为 var1+var2 的变量为 var5+var4。

可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

#### 3. ABA

如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。

J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步可能会比原子类更高效。

### 无同步方案

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

#### 1. 栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

```java
public class StackClosedExample {
    public void add100() {
        int cnt = 0;
        for (int i = 0; i < 100; i++) {
            cnt++;
        }
        System.out.println(cnt);
    }
}
```

```java
public static void main(String[] args) {
    StackClosedExample example = new StackClosedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> example.add100());
    executorService.execute(() -> example.add100());
    executorService.shutdown();
}
```

```html
100
100
```

#### 2. 线程本地存储（Thread Local Storage）

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消费队列的架构模式（如“生产者-消费者”模式）都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典 Web 交互模型中的“一个请求对应一个服务器线程”（Thread-per-Request）的处理方式，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用 java.lang.ThreadLocal 类来实现线程本地存储功能。

对于以下代码，thread1 中设置 threadLocal 为 1，而 thread2 设置 threadLocal 为 2。过了一段时间之后，thread1 读取 threadLocal 依然是 1，不受 thread2 的影响。

```java
public class ThreadLocalExample {
    public static void main(String[] args) {
        ThreadLocal threadLocal = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal.set(1);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(threadLocal.get());
            threadLocal.remove();
        });
        Thread thread2 = new Thread(() -> {
            threadLocal.set(2);
            threadLocal.remove();
        });
        thread1.start();
        thread2.start();
    }
}
```

```html
1
```

为了理解 ThreadLocal，先看以下代码：

```java
public class ThreadLocalExample1 {
    public static void main(String[] args) {
        ThreadLocal threadLocal1 = new ThreadLocal();
        ThreadLocal threadLocal2 = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal1.set(1);
            threadLocal2.set(1);
        });
        Thread thread2 = new Thread(() -> {
            threadLocal1.set(2);
            threadLocal2.set(2);
        });
        thread1.start();
        thread2.start();
    }
}
```

它所对应的底层结构图为：

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/6782674c-1bfe-4879-af39-e9d722a95d39.png" width="500px"> </div><br>

每个 Thread 都有一个 ThreadLocal.ThreadLocalMap 对象。

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

当调用一个 ThreadLocal 的 set(T value) 方法时，先得到当前线程的 ThreadLocalMap 对象，然后将 ThreadLocal-\>value 键值对插入到该 Map 中。

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

get() 方法类似。

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

ThreadLocal 从理论上讲并不是用来解决多线程并发问题的，因为根本不存在多线程竞争。

在一些场景 (尤其是使用线程池) 下，由于 ThreadLocal.ThreadLocalMap 的底层数据结构导致 ThreadLocal 有内存泄漏的情况，应该尽可能在每次使用 ThreadLocal 后手动调用 remove()，以避免出现 ThreadLocal 经典的内存泄漏甚至是造成自身业务混乱的风险。

#### 3. 可重入代码（Reentrant Code）

这种代码也叫做纯代码（Pure Code），可以在代码执行的任何时刻中断它，转而去执行另外一段代码（包括递归调用它本身），而在控制权返回后，原来的程序不会出现任何错误。

可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。

## 十二、锁优化

这里的锁优化主要是指 JVM 对 synchronized 的优化。

### 自旋锁

互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。

自旋锁虽然能避免进入阻塞状态从而减少开销，但是它需要进行忙循环操作占用 CPU 时间，它只适用于共享数据的锁定状态很短的场景。

在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。

### 锁消除

锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。

锁消除主要是通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。

对于一些看起来没有加锁的代码，其实隐式的加了很多锁。例如下面的字符串拼接代码就隐式加了锁：

```java
public static String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}
```

String 是一个不可变的类，编译器会对 String 的拼接自动优化。在 JDK 1.5 之前，会转化为 StringBuffer 对象的连续 append() 操作：

```java
public static String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```

每个 append() 方法中都有一个同步块。虚拟机观察变量 sb，很快就会发现它的动态作用域被限制在 concatString() 方法内部。也就是说，sb 的所有引用永远不会逃逸到 concatString() 方法之外，其他线程无法访问到它，因此可以进行消除。

### 锁粗化

如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

上一节的示例代码中连续的 append() 方法就属于这类情况。如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。对于上一节的示例代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。

### 轻量级锁

JDK 1.6 引入了偏向锁和轻量级锁，从而让锁拥有了四个状态：无锁状态（unlocked）、偏向锁状态（biasble）、轻量级锁状态（lightweight locked）和重量级锁状态（inflated）。

以下是 HotSpot 虚拟机对象头的内存布局，这些数据被称为 Mark Word。其中 tag bits 对应了五个状态，这些状态在右侧的 state 表格中给出。除了 marked for gc 状态，其它四个状态已经在前面介绍过了。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/bb6a49be-00f2-4f27-a0ce-4ed764bc605c.png" width="500"/> </div><br>

下图左侧是一个线程的虚拟机栈，其中有一部分称为 Lock Record 的区域，这是在轻量级锁运行过程创建的，用于存放锁对象的 Mark Word。而右侧就是一个锁对象，包含了 Mark Word 和其它信息。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/051e436c-0e46-4c59-8f67-52d89d656182.png" width="500"/> </div><br>

轻量级锁是相对于传统的重量级锁而言，它使用 CAS 操作来避免重量级锁使用互斥量的开销。对于绝大部分的锁，在整个同步周期内都是不存在竞争的，因此也就不需要都使用互斥量进行同步，可以先采用 CAS 操作进行同步，如果 CAS 失败了再改用互斥量进行同步。

当尝试获取一个锁对象时，如果锁对象标记为 0 01，说明锁对象的锁未锁定（unlocked）状态。此时虚拟机在当前线程的虚拟机栈中创建 Lock Record，然后使用 CAS 操作将对象的 Mark Word 更新为 Lock Record 指针。如果 CAS 操作成功了，那么线程就获取了该对象上的锁，并且对象的 Mark Word 的锁标记变为 00，表示该对象处于轻量级锁状态。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/baaa681f-7c52-4198-a5ae-303b9386cf47.png" width="400"/> </div><br>

如果 CAS 操作失败了，虚拟机首先会检查对象的 Mark Word 是否指向当前线程的虚拟机栈，如果是的话说明当前线程已经拥有了这个锁对象，那就可以直接进入同步块继续执行，否则说明这个锁对象已经被其他线程线程抢占了。如果有两条以上的线程争用同一个锁，那轻量级锁就不再有效，要膨胀为重量级锁。

### 偏向锁

偏向锁的思想是偏向于让第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作，甚至连 CAS 操作也不再需要。

当锁对象第一次被线程获得的时候，进入偏向状态，标记为 1 01。同时使用 CAS 操作将线程 ID 记录到 Mark Word 中，如果 CAS 操作成功，这个线程以后每次进入这个锁相关的同步块就不需要再进行任何同步操作。

当有另外一个线程去尝试获取这个锁对象时，偏向状态就宣告结束，此时撤销偏向（Revoke Bias）后恢复到未锁定状态或者轻量级锁状态。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/390c913b-5f31-444f-bbdb-2b88b688e7ce.jpg" width="600"/> </div><br>

## 十三、多线程开发良好的实践

- 给线程起个有意义的名字，这样可以方便找 Bug。

- 缩小同步范围，从而减少锁争用。例如对于 synchronized，应该尽量使用同步块而不是同步方法。

- 多用同步工具少用 wait() 和 notify()。首先，CountDownLatch, CyclicBarrier, Semaphore 和 Exchanger 这些同步类简化了编码操作，而用 wait() 和 notify() 很难实现复杂控制流；其次，这些同步类是由最好的企业编写和维护，在后续的 JDK 中还会不断优化和完善。

- 使用 BlockingQueue 实现生产者消费者问题。

- 多用并发集合少用同步集合，例如应该使用 ConcurrentHashMap 而不是 Hashtable。

- 使用本地变量和不可变类来保证线程安全。

- 使用线程池而不是直接创建线程，这是因为创建线程代价很高，线程池可以有效地利用有限的线程来启动任务。

## 参考资料

- 官方文档 https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html
- 并发官方教程 https://docs.oracle.com/javase/tutorial/essential/concurrency/atomic.html
- Doug Lea并发编程文章全部译文 http://ifeve.com/doug-lea/
- Java并发知识点总结 https://github.com/CL0610/Java-concurrency
- 线程与多线程必知必会(基础篇) https://zhuanlan.zhihu.com/p/33616143

- BruceEckel. Java 编程思想: 第 4 版 [M]. 机械工业出版社, 2007.
- 周志明. 深入理解 Java 虚拟机 [M]. 机械工业出版社, 2011.
- [Threads and Locks](https://docs.oracle.com/javase/specs/jvms/se6/html/Threads.doc.html)
- [线程通信](http://ifeve.com/thread-signaling/#missed_signal)
- [Java 线程面试题 Top 50](http://www.importnew.com/12773.html)
- [BlockingQueue](http://tutorials.jenkov.com/java-util-concurrent/blockingqueue.html)
- [thread state java](https://stackoverflow.com/questions/11265289/thread-state-java)
- [CSC 456 Spring 2012/ch7 MN](http://wiki.expertiza.ncsu.edu/index.php/CSC_456_Spring_2012/ch7_MN)
- [Java - Understanding Happens-before relationship](https://www.logicbig.com/tutorials/core-java-tutorial/java-multi-threading/happens-before.html)
- [6장 Thread Synchronization](https://www.slideshare.net/novathinker/6-thread-synchronization)
- [How is Java's ThreadLocal implemented under the hood?](https://stackoverflow.com/questions/1202444/how-is-javas-threadlocal-implemented-under-the-hood/15653015)
- [Concurrent](https://sites.google.com/site/webdevelopart/21-compile/06-java/javase/concurrent?tmpl=%2Fsystem%2Fapp%2Ftemplates%2Fprint%2F&showPrintDialog=1)
- [JAVA FORK JOIN EXAMPLE](http://www.javacreed.com/java-fork-join-example/ "Java Fork Join Example")
- [聊聊并发（八）——Fork/Join 框架介绍](http://ifeve.com/talk-concurrency-forkjoin/)
- [Eliminating SynchronizationRelated Atomic Operations with Biased Locking and Bulk Rebiasing](http://www.oracle.com/technetwork/java/javase/tech/biasedlocking-oopsla2006-preso-150106.pdf)

# 面试题

## 相关文章

- Java 并发 - 知识体系
- Java 并发 - 理论基础
  - 多线程的出现是要解决什么问题的?
  - 线程不安全是指什么? 举例说明
  - 并发出现线程不安全的本质什么? 可见性，原子性和有序性。
  - Java是怎么解决并发问题的? 3个关键字，JMM和8个Happens-Before
  - 线程安全是不是非真即假? 不是
  - 线程安全有哪些实现思路?
  - 如何理解并发和并行的区别?
- Java 并发 - 线程基础
  - 线程有哪几种状态? 分别说明从一种状态到另一种状态转变有哪些方式?
  - 通常线程有哪几种使用方式?
  - 基础线程机制有哪些?
  - 线程的中断方式有哪些?
  - 线程的互斥同步方式有哪些? 如何比较和选择?
  - 线程之间有哪些协作方式?
- 关键字: synchronized详解
  - Synchronized可以作用在哪里? 分别通过对象锁和类锁进行举例。
  - Synchronized本质上是通过什么保证线程安全的? 分三个方面回答：加锁和释放锁的原理，可重入原理，保证可见性原理。
  - Synchronized由什么样的缺陷?  Java Lock是怎么弥补这些缺陷的。
  - Synchronized和Lock的对比，和选择?
  - Synchronized在使用时有何注意事项?
  - Synchronized修饰的方法在抛出异常时,会释放锁吗?
  - 多个线程等待同一个snchronized锁的时候，JVM如何选择下一个获取锁的线程?
  - Synchronized使得同时只有一个线程可以执行，性能比较差，有什么提升的方法?
  - 我想更加灵活地控制锁的释放和获取(现在释放锁和获取锁的时机都被规定死了)，怎么办?
  - 什么是锁的升级和降级? 什么是JVM里的偏斜锁、轻量级锁、重量级锁?
  - 不同的JDK中对Synchronized有何优化?
- 关键字: volatile详解
  - volatile关键字的作用是什么?
  - volatile能保证原子性吗?
  - 之前32位机器上共享的long和double变量的为什么要用volatile? 现在64位机器上是否也要设置呢?
  - i++为什么不能保证原子性?
  - volatile是如何实现可见性的?  内存屏障。
  - volatile是如何实现有序性的?  happens-before等
  - 说下volatile的应用场景?
- 关键字: final详解
  - 所有的final修饰的字段都是编译期常量吗?
  - 如何理解private所修饰的方法是隐式的final?
  - 说说final类型的类如何拓展? 比如String是final类型，我们想写个MyString复用所有String中方法，同时增加一个新的toMyString()的方法，应该如何做?
  - final方法可以被重载吗? 可以
  - 父类的final方法能不能够被子类重写? 不可以
  - 说说final域重排序规则?
  - 说说final的原理?
  - 使用 final 的限制条件和局限性?
  - 看本文最后的一个思考题

> B. Java进阶 - Java 并发之J.U.C框架：然后需要对J.U.C框架五大类详细解读，包括：Lock框架，并发集合, 原子类, 线程池和工具类。@pdai

- JUC - 类汇总和学习指南
  - JUC框架包含几个部分?
  - 每个部分有哪些核心的类?
  - 最最核心的类有哪些?

> B.1 Java进阶 - Java 并发之J.U.C框架【1/5】：CAS及原子类：从最核心的CAS, Unsafe和原子类开始分析。

- JUC原子类: CAS, Unsafe和原子类详解
  - 线程安全的实现方法有哪些?
  - 什么是CAS?
  - CAS使用示例，结合AtomicInteger给出示例?
  - CAS会有哪些问题?
  - 针对这这些问题，Java提供了哪几个解决的?
  - AtomicInteger底层实现? CAS+volatile
  - 请阐述你对Unsafe类的理解?
  - 说说你对Java原子类的理解? 包含13个，4组分类，说说作用和使用场景。
  - AtomicStampedReference是什么?
  - AtomicStampedReference是怎么解决ABA的? 内部使用Pair来存储元素值及其版本号
  - java中还有哪些类可以解决ABA的问题? AtomicMarkableReference

> B.2 Java进阶 - Java 并发之J.U.C框架【2/5】：锁：然后分析JUC中锁。

- JUC锁: LockSupport详解
  - 为什么LockSupport也是核心基础类? AQS框架借助于两个类：Unsafe(提供CAS操作)和LockSupport(提供park/unpark操作)
  - 写出分别通过wait/notify和LockSupport的park/unpark实现同步?
  - LockSupport.park()会释放锁资源吗? 那么Condition.await()呢?
  - Thread.sleep()、Object.wait()、Condition.await()、LockSupport.park()的区别? 重点
  - 如果在wait()之前执行了notify()会怎样?
  - 如果在park()之前执行了unpark()会怎样?
- JUC锁: 锁核心类AQS详解
  - 什么是AQS? 为什么它是核心?
  - AQS的核心思想是什么? 它是怎么实现的? 底层数据结构等
  - AQS有哪些核心的方法?
  - AQS定义什么样的资源获取方式? AQS定义了两种资源获取方式：独占(只有一个线程能访问执行，又根据是否按队列的顺序分为公平锁和非公平锁，如ReentrantLock) 和共享(多个线程可同时访问执行，如Semaphore、CountDownLatch、 CyclicBarrier )。ReentrantReadWriteLock可以看成是组合式，允许多个线程同时对某一资源进行读。
  - AQS底层使用了什么样的设计模式? 模板
  - AQS的应用示例?
- JUC锁: ReentrantLock详解
  - 什么是可重入，什么是可重入锁? 它用来解决什么问题?
  - ReentrantLock的核心是AQS，那么它怎么来实现的，继承吗? 说说其类内部结构关系。
  - ReentrantLock是如何实现公平锁的?
  - ReentrantLock是如何实现非公平锁的?
  - ReentrantLock默认实现的是公平还是非公平锁?
  - 使用ReentrantLock实现公平和非公平锁的示例?
  - ReentrantLock和Synchronized的对比?
- JUC锁: ReentrantReadWriteLock详解
  - 为了有了ReentrantLock还需要ReentrantReadWriteLock?
  - ReentrantReadWriteLock底层实现原理?
  - ReentrantReadWriteLock底层读写状态如何设计的? 高16位为读锁，低16位为写锁
  - 读锁和写锁的最大数量是多少?
  - 本地线程计数器ThreadLocalHoldCounter是用来做什么的?
  - 缓存计数器HoldCounter是用来做什么的?
  - 写锁的获取与释放是怎么实现的?
  - 读锁的获取与释放是怎么实现的?
  - RentrantReadWriteLock为什么不支持锁升级?
  - 什么是锁的升降级? RentrantReadWriteLock为什么不支持锁升级?

> B.3 Java进阶 - Java 并发之J.U.C框架【3/5】：集合：再理解JUC中重要的支持并发的集合。

- JUC集合: ConcurrentHashMap详解
  - 为什么HashTable慢? 它的并发度是什么? 那么ConcurrentHashMap并发度是什么?
  - ConcurrentHashMap在JDK1.7和JDK1.8中实现有什么差别? JDK1.8解決了JDK1.7中什么问题
  - ConcurrentHashMap JDK1.7实现的原理是什么? 分段锁机制
  - ConcurrentHashMap JDK1.8实现的原理是什么? 数组+链表+红黑树，CAS
  - ConcurrentHashMap JDK1.7中Segment数(concurrencyLevel)默认值是多少? 为何一旦初始化就不可再扩容?
  - ConcurrentHashMap JDK1.7说说其put的机制?
  - ConcurrentHashMap JDK1.7是如何扩容的? rehash(注：segment 数组不能扩容，扩容是 segment 数组某个位置内部的数组 HashEntry<K,V>[] 进行扩容)
  - ConcurrentHashMap JDK1.8是如何扩容的? tryPresize
  - ConcurrentHashMap JDK1.8链表转红黑树的时机是什么? 临界值为什么是8?
  - ConcurrentHashMap JDK1.8是如何进行数据迁移的? transfer
- JUC集合: CopyOnWriteArrayList详解
  - 请先说说非并发集合中Fail-fast机制?
  - 再为什么说ArrayList查询快而增删慢?
  - 对比ArrayList说说CopyOnWriteArrayList的增删改查实现原理? COW基于拷贝
  - 再说下弱一致性的迭代器原理是怎么样的? COWIterator<E>
  - CopyOnWriteArrayList为什么并发安全且性能比Vector好?
  - CopyOnWriteArrayList有何缺陷，说说其应用场景?
- JUC集合: ConcurrentLinkedQueue详解
  - 要想用线程安全的队列有哪些选择? Vector，Collections.synchronizedList( List<T> list), ConcurrentLinkedQueue等
  - ConcurrentLinkedQueue实现的数据结构?
  - ConcurrentLinkedQueue底层原理?  全程无锁(CAS)
  - ConcurrentLinkedQueue的核心方法有哪些? offer()，poll()，peek()，isEmpty()等队列常用方法
  - 说说ConcurrentLinkedQueue的HOPS(延迟更新的策略)的设计?
  - ConcurrentLinkedQueue适合什么样的使用场景?
- JUC集合: BlockingQueue详解
  - 什么是BlockingDeque?
  - BlockingQueue大家族有哪些? ArrayBlockingQueue, DelayQueue, LinkedBlockingQueue, SynchronousQueue...
  - BlockingQueue适合用在什么样的场景?
  - BlockingQueue常用的方法?
  - BlockingQueue插入方法有哪些? 这些方法(add(o),offer(o),put(o),offer(o, timeout, timeunit))的区别是什么?
  - BlockingDeque 与BlockingQueue有何关系，请对比下它们的方法?
  - BlockingDeque适合用在什么样的场景?
  - BlockingDeque大家族有哪些?
  - BlockingDeque 与BlockingQueue实现例子?

> B.4 Java进阶 - Java 并发之J.U.C框架【4/5】：线程池：再者分析JUC中非常常用的线程池等。

- JUC线程池: FutureTask详解
  - FutureTask用来解决什么问题的? 为什么会出现?
  - FutureTask类结构关系怎么样的?
  - FutureTask的线程安全是由什么保证的?
  - FutureTask结果返回机制?
  - FutureTask内部运行状态的转变?
  - FutureTask通常会怎么用? 举例说明。
- JUC线程池: ThreadPoolExecutor详解
  - 为什么要有线程池?
  - Java是实现和管理线程池有哪些方式?  请简单举例如何使用。
  - 为什么很多公司不允许使用Executors去创建线程池? 那么推荐怎么使用呢?
  - ThreadPoolExecutor有哪些核心的配置参数? 请简要说明
  - ThreadPoolExecutor可以创建哪是哪三种线程池呢?
  - 当队列满了并且worker的数量达到maxSize的时候，会怎么样?
  - 说说ThreadPoolExecutor有哪些RejectedExecutionHandler策略? 默认是什么策略?
  - 简要说下线程池的任务执行机制? execute –> addWorker –>runworker (getTask)
  - 线程池中任务是如何提交的?
  - 线程池中任务是如何关闭的?
  - 在配置线程池的时候需要考虑哪些配置因素?
  - 如何监控线程池的状态?
- JUC线程池: ScheduledThreadPool详解
  - ScheduledThreadPoolExecutor要解决什么样的问题?
  - ScheduledThreadPoolExecutor相比ThreadPoolExecutor有哪些特性?
  - ScheduledThreadPoolExecutor有什么样的数据结构，核心内部类和抽象类?
  - ScheduledThreadPoolExecutor有哪两个关闭策略? 区别是什么?
  - ScheduledThreadPoolExecutor中scheduleAtFixedRate 和 scheduleWithFixedDelay区别是什么?
  - 为什么ThreadPoolExecutor 的调整策略却不适用于 ScheduledThreadPoolExecutor?
  - Executors 提供了几种方法来构造 ScheduledThreadPoolExecutor?
- JUC线程池: Fork/Join框架详解
  - Fork/Join主要用来解决什么样的问题?
  - Fork/Join框架是在哪个JDK版本中引入的?
  - Fork/Join框架主要包含哪三个模块? 模块之间的关系是怎么样的?
  - ForkJoinPool类继承关系?
  - ForkJoinTask抽象类继承关系? 在实际运用中，我们一般都会继承 RecursiveTask 、RecursiveAction 或 CountedCompleter 来实现我们的业务需求，而不会直接继承 ForkJoinTask 类。
  - 整个Fork/Join 框架的执行流程/运行机制是怎么样的?
  - 具体阐述Fork/Join的分治思想和work-stealing 实现方式?
  - 有哪些JDK源码中使用了Fork/Join思想?
  - 如何使用Executors工具类创建ForkJoinPool?
  - 写一个例子: 用ForkJoin方式实现1+2+3+...+100000?
  - Fork/Join在使用时有哪些注意事项? 结合JDK中的斐波那契数列实例具体说明。

> B.5 Java进阶 - Java 并发之J.U.C框架【5/5】：工具类：最后来看下JUC中有哪些工具类，以及线程隔离术ThreadLocal。

- JUC工具类: CountDownLatch详解
  - 什么是CountDownLatch?
  - CountDownLatch底层实现原理?
  - CountDownLatch一次可以唤醒几个任务? 多个
  - CountDownLatch有哪些主要方法? await(),countDown()
  - CountDownLatch适用于什么场景?
  - 写道题：实现一个容器，提供两个方法，add，size 写两个线程，线程1添加10个元素到容器中，线程2实现监控元素的个数，当个数到5个时，线程2给出提示并结束? 使用CountDownLatch 代替wait notify 好处。
- JUC工具类: CyclicBarrier详解
  - 什么是CyclicBarrier?
  - CyclicBarrier底层实现原理?
  - CountDownLatch和CyclicBarrier对比?
  - CyclicBarrier的核心函数有哪些?
  - CyclicBarrier适用于什么场景?
- JUC工具类: Semaphore详解
  - 什么是Semaphore?
  - Semaphore内部原理?
  - Semaphore常用方法有哪些? 如何实现线程同步和互斥的?
  - Semaphore适合用在什么场景?
  - 单独使用Semaphore是不会使用到AQS的条件队列?
  - Semaphore中申请令牌(acquire)、释放令牌(release)的实现?
  - Semaphore初始化有10个令牌，11个线程同时各调用1次acquire方法，会发生什么?
  - Semaphore初始化有10个令牌，一个线程重复调用11次acquire方法，会发生什么?
  - Semaphore初始化有1个令牌，1个线程调用一次acquire方法，然后调用两次release方法，之后另外一个线程调用acquire(2)方法，此线程能够获取到足够的令牌并继续运行吗?
  - Semaphore初始化有2个令牌，一个线程调用1次release方法，然后一次性获取3个令牌，会获取到吗?
- JUC工具类: Phaser详解
  - Phaser主要用来解决什么问题?
  - Phaser与CyclicBarrier和CountDownLatch的区别是什么?
  - 如果用CountDownLatch来实现Phaser的功能应该怎么实现?
  - Phaser运行机制是什么样的?
  - 给一个Phaser使用的示例?
- JUC工具类: Exchanger详解
  - Exchanger主要解决什么问题?
  - 对比SynchronousQueue，为什么说Exchanger可被视为 SynchronousQueue 的双向形式?
  - Exchanger在不同的JDK版本中实现有什么差别?
  - Exchanger实现机制?
  - Exchanger已经有了slot单节点，为什么会加入arena node数组? 什么时候会用到数组?
  - arena可以确保不同的slot在arena中是不会相冲突的，那么是怎么保证的呢?
  - 什么是伪共享，Exchanger中如何体现的?
  - Exchanger实现举例
- Java 并发 - ThreadLocal详解
  - 什么是ThreadLocal? 用来解决什么问题的?
  - 说说你对ThreadLocal的理解
  - ThreadLocal是如何实现线程隔离的?
  - 为什么ThreadLocal会造成内存泄露? 如何解决
  - 还有哪些使用ThreadLocal的应用场景?

> C. Java进阶 - Java 并发之 本质与模式：最后站在更高的角度看其本质(协作，分工和互斥)，同时总结上述知识点所使用的模式。@pdai

- Java 并发 - 并发的本质：协作,分工和互斥
- Java 并发 - 并发的模式梳理



