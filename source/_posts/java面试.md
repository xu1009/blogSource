---
title: '''java面试'''
date: 2018-08-27 20:17:49
tags: java 基础 面试
---

> JRE 是java运行环境，包括JVM和java类库，以及一些模块等，JDK是JRE的一个超集，提供了更多工具，比如编译器、各种诊断工具等。


#### java是解释执行？
> 明显不是，问题比较简单，回答不能笼统，首先是经过javac把java文件编译成字节码，也就是class文件，然后通过jvm内嵌的解释器转换成机器码，通常jvm是oracle jdk的hotspot，内嵌解释器是JIT，会将一部分热点代码编译成机器码，属于编译执行。





java分为编译期和运行时， 编译指的是javac把源码编译成字节码文件,运行时JVM会通过类加载器加载字节码，转成机器码执行。

是这样的， javac把源码编译成字节码文件，然后JVM加载这个字节码文件，使用内部解释器进行解释执行，同时也引入了JIT技术，也就是对于某些需要反复执行的代码（热点代码）转成机器码，这样下次就不用解释执行了。然后AOT是啥呢，是通过工具直接把字节码全转换成机器码，这样就不用解释执行了，提高效率。

java通过jvm这种跨平台的抽象，屏蔽了操作系统和硬件的细节，实现一次编译，到处执行的基础，class-loader加载字节码，解释或者编译执行。

JDK8使用的是解释和编译混合的一种模式。
java启动参数 -Xint就是高数jvm只进行解释执行，不对代码进行编译。
-Xcomp 关闭解释运行，最大化优化级别。

AOT 直接将字节码文件编译成机器码，避免了类加载过程。

java蓝图：

![](https://static001.geekbang.org/resource/image/20/32/20bc6a900fc0b829c2f0e723df050732.png)

#### Exception和Error区别？
> 都继承了Throwable类，Exception 是程序正常运行中，可以预料的意外情况，可以预料的意外情况，可能并且应该被捕获，进行相应处理。Error是指在正常情况下不大可能出现的情况，比如常见的内存泄露，不能捕获。



说一下Exception

分为可检查异常和不检查异常，可检查异常是指在源码里面显示地进行捕获处理，编译期就会进行检查，不检查异常又称为运行时异常比如NPC等，通常这个可以编码避免错误，具体根据需要判断是否需要捕获。

看一下下面这张图：

![](https://static001.geekbang.org/resource/image/ac/00/accba531a365e6ae39614ebfa3273900.png)


ClassNotFoundException和NoClassDefFoundError的区别?

#### final、finally、finalize区别

下面这段代码里面的finally是不会执行的，这是个特例
```java
try{
    System.exit(1);
}
finally{
    System.out.println("asd");
}
```
finalize阻止gc，不推荐使用，回收之前干些事情，会降低回收效能。

#### 强引用、软引用、弱引用、幻想引用

> 不同的引用类型，主要体现的是对象的可达性状态和对垃圾收集的影响。

看一下这个图,状态切换，软引用和弱引用多用于缓存，虚引用用于监控？
![](https://static001.geekbang.org/resource/image/36/b0/36d3c7b158eda9421ef32463cb4d4fb0.png)

虚引用必须和引用队列搭配使用，主要是用于跟踪对象被垃圾回收的活动，因为虚引用所引用的对象在被垃圾回收之前,先把引用放入队列，这样可以通过判断队列是否有内容来判断对象状态，在垃圾回收之前发出通知。


#### String、StringBuffer StringBuilder区别

String 还是有很多东西的，比如给你几个用例判断创建了几个对象，还有intern()方法，以及在几个jdk版本中的区别


主要在1.7之后将对于字符串分为堆内存和常量池。


* String s1 = "first";
* String s2 = "se" + "cond";
* String s12 = "first" + s2;
* String s3 = new String("three");
* String s4 = new String("fo") + "ur";
* String s5 = new String("fo") + new String("ur");

上述在内存中的存储如下：
![](https://user-gold-cdn.xitu.io/2018/1/5/160c3dd4be37720c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

对于 "" 这种类型是在编译期间就能确定的， 属于字符串常量，会直接在常量池中创建，对于 new String()创建的字符串引用就不会。美滋滋。

还有一个intern()方法，这个方法的作用是先看一下常量池中是否有该字符串，如果有的话就返回常量池中的，如果没有的话，就把堆中引用地址加到常量池返回的是同一个对象，或者说两个引用同一个对象。



#### 动态代理机制

> java是一种强类型静态语言，静态是指在编译时检查，动态是指在运行时检查。强类型和弱类型就不用解释了。

反射给java提供了部分动态语言的能力。

说一下动态代理，jdk本身有的利用的是反射技术，还有cglib、asm等基于字节码操作机制。

jdk动态代理确定是必须实现某个接口，以及强制写invocationHandler有点侵入式。
cglib基于字节码基于子类？后面再看。

#### 包装类和普通数据类型（Integer， int）

> 装箱和拆箱，较小值缓存，比如int类型-128到127这个可以配置最大值范围。

对象由三部分组成： 对象头、对象实例、对齐填充。

对象头第一部分存储对象自身运行时数据，比如hashcode、gc分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，这部分称为Mark word。

第二部分是类型指针，对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

实例数据就是包含真正存储的信息。

对齐填充是相当于占位符。

#### Vector、ArrayList、LinkedList

> 都支持按位置进行定位，添加删除等操作，集合框架中的List，都提供迭代器遍历。

Vector：线程安全的动态数组，线程安全是通过synchronize关键字实现的，性能比较差，自动扩容，当数组满时，会创建新的数组，拷贝原有数组数据。
ArrayList： 非线程安全的动态数组，扩容逻辑和Vector不一样。
LinkedList：非线程安全的双向链表，对于删除和插入操作来说，相对比较好。
看一下狭义的集合框架：
![](https://static001.geekbang.org/resource/image/67/c7/675536edf1563b11ab7ead0def1215c7.png)

Collections.synchronizedList(new ArrayList<>()) 可以将那些非线程安全的集合包装成线程安全的，怎么实现的呢，就是很简单粗暴，将那些get、set、add之类的方法全部加上synchronize关键字。

jdk内部排序，比如Arrays.sort()和Collections.sort()使用的是什么排序算法，这要根据数据类型来分。

原始数据类型使用的是快速排序的变种，对象数据类型使用的是二分和归并的变种。


#### HashTable、hashmap、treemap

看一下map的结构，比如下图：
![](https://static001.geekbang.org/resource/image/26/7c/266cfaab2573c9777b1157816784727c.png)

散列是啥，散列是根据key得到内存中的地址，key到内存地址过程叫散列，映射函数叫散列函数，映射表叫散列集。


面试必问的hashmap源码：
* 基本实现
* 容量和负载因子
* 树化
看一下map的结构，数组加链表不用说了
![](https://static001.geekbang.org/resource/image/1f/56/1f72306a9d8719c66790b56ef7977c56.png)

具体hashmap可以看一下这个[博客](http://welkinbai.coding.me/2017/08/12/jdk-collection-3/#HashMap%EF%BC%88%E5%AE%9E%E7%8E%B0%E5%B1%82%EF%BC%89)


#### jdk并发包concurrenthashmap
> 分段式加锁，线程安全，1.7和1.8区别。

![](https://static001.geekbang.org/resource/image/d4/d9/d45bcf9a34da2ef1ef335532b0198bd9.png)

线程安全的list还有个copyonwriteArrayList
还有就是可以通过Collections提供的同步包装器，只是简单的把所有操作加上synchronize关键字

#### java io nio nio2(aio)
> NIO面试的时候问的比较多一些，然后NIO(同步非阻塞)的话是1.4新加的，1.7又做了很大改变，可以称为NIO2或者AIO,异步非阻塞IO


说一下同步和异步与阻塞和非阻塞的区别和联系

* 同步和异步，同步是一种可靠额有序运行机制，当我们进行同步操作时，后续任务是等待当前调用返回，才会进行下一步，异步则不需要等待，通常是依赖回调机制来实现任务间的次序关系，事件驱动（感觉比阻塞非阻塞更上一层）
* 阻塞非阻塞，是线程级别的，和同步和非同步相比要更底层一些，上面是任务级别的。

看一下java io的uml图：
![](https://static001.geekbang.org/resource/image/43/8b/4338e26731db0df390896ab305506d8b.png)


NIO 使用单线程管理channel，减少了线程切换开销
看一下NIO的框图：

![](https://static001.geekbang.org/resource/image/ad/a2/ad3b4a49f4c1bff67124563abc50a0a2.png)

再看一下NIO2
```java
  AsynchronousServerSocketChannel serverSocketChannel = AsynchronousServerSocketChannel.open().bind(new InetSocketAddress(InetAddress.getLocalHost(), 8888));
        serverSocketChannel.accept(serverSocketChannel, new CompletionHandler<AsynchronousSocketChannel, AsynchronousServerSocketChannel>() {
            @Override
            public void completed(AsynchronousSocketChannel asynchronousSocketChannel, AsynchronousServerSocketChannel asynchronousServerSocketChannel) {
                System.out.println("12321");
            }

            @Override
            public void failed(Throwable throwable, AsynchronousServerSocketChannel asynchronousServerSocketChannel) {

            }
        });
```
和ajax很像啊，事件驱动，异步回调。



详细说一下同步异步，阻塞和非阻塞问题

* BIO传统的IO模型
* 同步非阻塞IO,默认创建的socket都是阻塞的，非阻塞IO要求socket被设置为NONBLOCK,并非是java NIO
* IO多路复用，即经典的Reactor设计模式，有时也称为异步阻塞IO,java中的selector和Linux中的epoll都是这种模型
* 异步IO，即经典的Proactor设计模式，异步非阻塞IO

同步和异步描述的是用户线程与内核的**交互方式**：同步是指用户线程发起IO请求后需要等待或者轮询内核IO操作完成后才能继续执行，而异步是指用户线程发起IO请求后仍继续执行，当内核IO操作完成之后通知用户，或者调用用户线程注册的回调函数。

阻塞和非阻塞的概念描述的是用户线程调用内核IO的**操作方式**：阻塞是指IO操作需要彻底完成后才返回到用户空间，而非阻塞是指IO操作被调用后立即返回给用户一个状态值，无需等到IO操作彻底完成。

感觉一样，看一下图：

同步阻塞IO
这个图还是比较形象的，被动等待，需要等待数据准备好后才能读取。
![](http://images.cnitblog.com/blog/405877/201411/142330286789443.png)

同步非阻塞IO
同步非阻塞IO是在同步阻塞IO基础上，将socket设置为非阻塞，这样用户线程可以在发起IO请求后立即返回。这个是用户线程主动去查，不在内核阻塞，内核会立即返回，然后用户线程主动的去轮询状态。这样会导致产生大量重复请求，耗费大量CPU资源。
![](http://images.cnitblog.com/blog/405877/201411/142332004602984.png)


IO多路复用
IO多路复用模型是建立在内核提供的多路分离函数select基础之上的，使用select函数可以避免非阻塞IO模型中轮询等待的问题。
![](http://images.cnitblog.com/blog/405877/201411/142332187256396.png)

主要是把socket添加到select中，然后阻塞等待select系统调用返回，当数据到达时，socket被激活，select函数返回，用户线程正式发起read请求，读取数据并执行。这样其实也是阻塞的，阻塞在select那，只不过实现了单线程监听多个通道，提高了cpu利用率，用户可以注册自己感兴趣的socket或者IO请求，然后去做自己的事情。

其实利用的是reactor设计模式，见下图：
![](http://images.cnitblog.com/blog/405877/201411/142332350853195.png)

异步阻塞，阻塞主要是调用内核那个select函数时阻塞。
![](http://images.cnitblog.com/blog/405877/201411/142333254136604.png)


异步IO使用了Proactor设计模式：

![](http://images.cnitblog.com/blog/405877/201411/151608309061672.jpg)

Proactor模式和Reactor模式在结构上比较相似，不过在用户（Client）使用方式上差别较大。Reactor模式中，用户线程通过向Reactor对象注册感兴趣的事件监听，然后事件触发时调用事件处理函数。而Proactor模式中，用户线程将AsynchronousOperation（读/写等）、Proactor以及操作完成时的CompletionHandler注册到AsynchronousOperationProcessor。AsynchronousOperationProcessor使用Facade模式提供了一组异步操作API（读/写等）供用户使用，当用户线程调用异步API后，便继续执行自己的任务。AsynchronousOperationProcessor 会开启独立的内核线程执行异步操作，实现真正的异步。当异步IO操作完成时，AsynchronousOperationProcessor将用户线程与AsynchronousOperation一起注册的Proactor和CompletionHandler取出，然后将CompletionHandler与IO操作的结果数据一起转发给Proactor，Proactor负责回调每一个异步操作的事件完成处理函数handle_event。虽然Proactor模式中每个异步操作都可以绑定一个Proactor对象，但是一般在操作系统中，Proactor被实现为Singleton模式，以便于集中化分发操作完成事件。

![](http://images.cnitblog.com/blog/405877/201411/142333511475767.png)

异步IO模型中，用户线程直接使用内核提供的异步IO API发起read请求，且发起后立即返回，继续执行用户线程代码。不过此时用户线程已经将调用的AsynchronousOperation和CompletionHandler注册到内核，然后操作系统开启独立的内核线程去处理IO操作。当read请求的数据到达时，由内核负责读取socket中的数据，并写入用户指定的缓冲区中。最后内核将read的数据和用户线程注册的CompletionHandler分发给内部Proactor，Proactor将IO完成的信息通知给用户线程（一般通过调用用户线程注册的完成事件处理函数），完成异步IO。


有个理解比较形象，就是区分同步异步以及阻塞非阻塞。

阻塞非阻塞是对于判断数据是否就绪是由用户线程还是内核线程完成，异步还是同步由数据拷贝是由用户线程还是内核线程完成。

#### 说一下文件拷贝？
说一下用户态空间和内核态空间：

系统调用时操作系统的最小功能单位。

用户态空间是指应用，内核态空间是指底层硬件资源比如cpu，内存等，为上层提供访问接口也就是“系统调用”

看一下IO操作写入和读取操作：
进行了多次上下文切换，先从磁盘读取到内核缓存，用户再到内核缓存去拿数据，反向同理。
![](https://static001.geekbang.org/resource/image/6d/85/6d2368424431f1b0d2b935386324b585.png)


但是NIO拷贝文件实现了0拷贝，是啥呢，就是直接在内核层面操作，不涉及用户态空间。看一下下面这个图：

![](https://static001.geekbang.org/resource/image/b0/ea/b0c8226992bb97adda5ad84fe25372ea.png)


#### 接口和抽象类


接口中只能包含public方法或者public static final 常量，可以实现多个接口，不能实例化，1.8有自己的实现default方法。

抽象类，不能实例化，不能多继承，可以有抽象方法，也可以没有，子类必须实现抽象方法。


#### synchronize关键字和reentrantLock

再入锁，表示一个线程试图获取一个他已经获取的锁时，这个获取动作自动完成，锁的持有是以线程为单位而不是基于调用次数。

再入锁要比synchronized关键字能实现更加细腻的操作，比如公平性，实现带有超时的锁，条件变量。通过CAS操作，将线程对象放到一个双向链表中，然后每次取出双向链表中的头结点和当前线程相比。

synchronized 是基于monitor实现的，monitorenter和monitorexit，jdk 1.6之后对synchronized进行了优化，实现了三种不同的锁： 偏斜锁，轻量级锁，重量级锁。


并发包将锁分为读写锁，因为读之间是不需要同步的。


#### 线程

> 线程是系统调度的最小单元，一个进程可以包含多个线程，作为任务真正的执行者，有自己的栈，寄存器和本地存储，但是会和进程内其他线程共享文件描述符、虚拟地址空间等。

线程状态： 新创建，可运行，阻塞，等待，计时等待，终止。

状态转移看下图：
![](https://static001.geekbang.org/resource/image/31/dc/3169b7ca899afeb0359f132fb77c29dc.png)


死锁发生条件就是循环依赖,详情见下面代码，下面还有检测死锁的代码

```java
package com.company.interview;

import java.lang.management.ManagementFactory;
import java.lang.management.ThreadInfo;
import java.lang.management.ThreadMXBean;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class LockSample extends Thread{
    private String lockA;
    private String lockB;
    LockSample(String name, String lockA, String lockB){
        super(name);
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @Override
    public void run() {
        synchronized (lockA){
            System.out.println(this.getName() + "   " + lockA);
            try{
                System.out.println("before " + this.getName());
//                Thread.sleep(1000L);
                System.out.println("after " + this.getName());
                synchronized (lockB){
                    System.out.println(this.getName() + "   " + lockB);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {

        ThreadMXBean mxBean = ManagementFactory.getThreadMXBean();
        Runnable dlCheck = () -> {
          long[] threadIds = mxBean.findDeadlockedThreads();
          if (threadIds != null){
              ThreadInfo[] threadInfo = mxBean.getThreadInfo(threadIds);
              for (ThreadInfo threadInfo1 : threadInfo){
                  System.out.println(threadInfo1.getThreadName());
              }
          }
        };
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);
        scheduledExecutorService.scheduleAtFixedRate(dlCheck, 5, 10, TimeUnit.SECONDS);
        String lockA = "1";
        String lockB = "2";
        LockSample lockSample1 = new LockSample("THREAD1", lockA, lockB);
        LockSample lockSample2 = new LockSample("THREAD2", lockB, lockA);
        lockSample1.start();
        lockSample2.start();
        lockSample1.join();
        lockSample2.join();
    }
}

```


copyonwrite 的操作是是不修改原来的数组，直接copy之后修改然后替换。

信号量是啥就是允许同一时间多少线程可以运行，类似于线程池？

countdownLatch有点像定时器中断？


#### 函数式编程
命令式编程和函数式编程，命令式编程就是那种在代码上很明显一步步实现功能，函数式是更抽象的描述，高阶函数，让函数像数据一样进行传递，而且函数内部不依赖于外部变量，利于单测，感觉就像高阶函数，传递的是函数，函数又像java匿名内部类那种，代码简洁。


命令式： 完全自己一步步实现

声明式： 底层一部分实现依赖于库

函数式： 基于声明式，加入高阶函数


函数式编程的特点：
1. 函数可以作为参数传递给另一个函数（高阶函数）
2. 只用表达式不用语句，这里和js很像，expression和statement的区别，表达式有返回值，语句没有，函数式编程要求只用表达式
3. 没有副作用，就是不修改外部变量，是个独立模块，每次都是返回新值

函数式编程意义：
1. 代码简洁
2. 接近自然语言
3. 方便做单元测试，因为是独立模块，不依赖于外部。
4. 易于并发编程，因为是独立模块，不修改外部变量，都是自己局部变量。
