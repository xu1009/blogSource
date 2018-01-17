---
title: java并发基础
date: 2017-09-18 14:07:52
tags: 并发 Java
---
### 线程控制逃逸规则

如果一个资源的创建，使用，销毁都在同一个线程内完成，且永远不会脱离该线程的控制，则该资源的使用就是线程安全的。

竞态条件：当多个线程同时访问同一个资源，并且其中改的一个或者多个线程对这个资源进行了写操作，才会产生静态条件，多个线程同时读取一个资源不会产生竞态条件。

### Java内存模型

线程独享栈，线程共享堆
![](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-1.png)

下面是栈和堆的存储类型

![](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-2.png)

Java内存模型和硬件内存架构之间的关系

通常情况下，当一个CPU需要读取主存时，它会将主存的部分读到CPU缓存中。它甚至可能将缓存中的部分内容读到它的内部寄存器中，然后在寄存器中执行操作。当CPU需要将结果写回到主存中去时，它会将内部寄存器的值刷新到缓存中，然后在某个时间点将值刷新回主存。

Java内存模型与硬件内存架构之间存在差异。硬件内存架构没有区分线程栈和堆。对于硬件，所有的线程栈和堆都分布在主内中。部分线程栈和堆可能有时候会出现在CPU缓存中和CPU内部的寄存器中。如下图所示：
![](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-5.png)

这样的内存模型会带来一些问题

* 线程对共享变量修改的可见性（volatile）
* 读写和检查共享变量时出现竞争条件（同步）
### 同步关键字（synchronize）

同步代码块保证一次只有一个线程可以进入，同时保证变量的修改会被立即刷新到主存，读取也是从主存中先刷新到缓存，就是保证了可见性。

其实就是对于一些非原子操作保证其线程安全

同步块包括四种：
1. 实例方法
2. 静态方法
3. 实例方法中的同步块
4. 静态方法中的同步块


实例方法同步
```java
public synchronized void add(int value){
this.count += value;
}
```
Java实例方法同步是同步在拥有该方法的对象上。这样，每个实例其方法同步都同步在不同的对象上，即该方法所属的实例。只有一个线程能够在实例方法同步块中运行。如果有多个实例存在，那么一个线程一次可以在一个实例同步块中执行操作。一个实例一个线程。

静态方法同步

```java
public static synchronized void add(int value){
 count += value;
 }
```
静态方法的同步是指同步在该方法所在的类对象上。因为在Java虚拟机中一个类只能对应一个类对象，所以同时只允许一个线程执行同一个类中的静态同步方法。

实例方法中的同步块

```java
public void add(int value){

    synchronized(this){
       this.count += value;
    }
  }
```
注意Java同步块构造器用括号将对象括起来。在上例中，使用了“this”，即为调用add方法的实例本身。在同步构造器中用括号括起来的对象叫做监视器对象。上述代码使用监视器对象同步，同步实例方法使用调用方法本身的实例作为监视器对象。


静态方法中的同步块

```java
public class MyClass {
    public static synchronized void log1(String msg1, String msg2){
       log.writeln(msg1);
       log.writeln(msg2);
    }

    public static void log2(String msg1, String msg2){
       synchronized(MyClass.class){
          log.writeln(msg1);
          log.writeln(msg2);
       }
    }
  }
```
如果第二个同步块不是同步在MyClass.class这个对象上。那么这两个方法可以同时被线程访问。

同步的对象不一样，一个同步的是类，一个同步的是类对象的实例，这个很关键。

#### 线程间的通信

* 通过共享对象通信
最简单的方式，线程共享一个对象，一个线程修改对象状态，其它线程读取对象状态，
* 忙等待
准备处理数据的线程B正在等待数据变为可用。换句话说，它在等待线程A的一个信号，这个信号使hasDataToProcess()返回true。线程B运行在一个循环里，以等待这个信号：

```java
protected MySignal sharedSignal = ...

...

while(!sharedSignal.hasDataToProcess()){
  //do nothing... busy waiting
}
```
* wait()，notify()，notifyAll()
忙等待没有对运行等待线程的CPU进行有效的利用，除非平均等待时间非常短。否则，让等待线程进入睡眠或者非运行状态更为明智，直到它接收到它等待的信号。
Java有一个内建的等待机制来允许线程在等待信号的时候变为非运行状态。java.lang.Object 类定义了三个方法，wait()、notify()和notifyAll()来实现这个等待机制。

这个就是线程调用wait方法时进入等待状态，并释放了监视器锁，让其它线程去竞争锁，如果有线程调用了notify方法，则随机一个线程会被唤醒，当然唤醒之后还要去竞争锁，只有竞争到了锁才能推出wait方法。notifyAll是唤醒所有当前对象的等待状态的线程

* 丢失的信号
notify()和notifyAll()方法不会保存调用它们的方法，因为当这两个方法被调用时，有可能没有线程处于等待状态。通知信号过后便丢弃了。因此，如果一个线程先于被通知线程调用wait()前调用了notify()，等待的线程将错过这个信号。
```java
public class MyWaitNotify2{

  MonitorObject myMonitorObject = new MonitorObject();
  boolean wasSignalled = false;

  public void doWait(){
    synchronized(myMonitorObject){
      if(!wasSignalled){
        try{
          myMonitorObject.wait();
         } catch(InterruptedException e){...}
      }
      //clear signal and continue running.
      wasSignalled = false;
    }
  }

  public void doNotify(){
    synchronized(myMonitorObject){
      wasSignalled = true;
      myMonitorObject.notify();
    }
  }
}
```
留意doNotify()方法在调用notify()前把wasSignalled变量设为true。同时，留意doWait()方法在调用wait()前会检查wasSignalled变量。事实上，如果没有信号在前一次doWait()调用和这次doWait()调用之间的时间段里被接收到，它将只调用wait()。
* 假唤醒
由于莫名其妙的原因，线程有可能在没有调用过notify()和notifyAll()的情况下醒来。这就是所谓的假唤醒（spurious wakeups）。无端端地醒过来了。

如果在MyWaitNotify2的doWait()方法里发生了假唤醒，等待线程即使没有收到正确的信号，也能够执行后续的操作。这可能导致你的应用程序出现严重问题。

为了防止假唤醒，保存信号的成员变量将在一个while循环里接受检查，而不是在if表达式里。这样的一个while循环叫做自旋锁（校注：这种做法要慎重，目前的JVM实现自旋会消耗CPU，如果长时间不调用doNotify方法，doWait方法会一直自旋，CPU会消耗太大）。被唤醒的线程会自旋直到自旋锁(while循环)里的条件变为false。以下MyWaitNotify2的修改版本展示了这点：

```java
public class MyWaitNotify3{

  MonitorObject myMonitorObject = new MonitorObject();
  boolean wasSignalled = false;

  public void doWait(){
    synchronized(myMonitorObject){
      while(!wasSignalled){
        try{
          myMonitorObject.wait();
         } catch(InterruptedException e){...}
      }
      //clear signal and continue running.
      wasSignalled = false;
    }
  }

  public void doNotify(){
    synchronized(myMonitorObject){
      wasSignalled = true;
      myMonitorObject.notify();
    }
  }
}
```
* 多个线程等待相同信号
如果你有多个线程在等待，被notifyAll()唤醒，但只有一个被允许继续执行，使用while循环也是个好方法。每次只有一个线程可以获得监视器对象锁，意味着只有一个线程可以退出wait()调用并清除wasSignalled标志（设为false）。一旦这个线程退出doWait()的同步块，其他线程退出wait()调用，并在while循环里检查wasSignalled变量值。但是，这个标志已经被第一个唤醒的线程清除了，所以其余醒来的线程将回到等待状态，直到下次信号到来。

#### ThreadLocal
和线程绑定，每个线程都有自己的私有ThreadLocal
```java
class ThreadTst{
    public static class MyThread implements Runnable{
       private ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>();
        @Override
        public void run() {
            threadLocal.set((int) (Math.random() * 100D));
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + threadLocal.get());
        }
    }
    public static void main(String[] args) throws InterruptedException {
        MyThread tst = new MyThread();
        Thread thread1 = new Thread(tst,"tt");
        thread1.start();
        Thread thread2 = new Thread(tst,"rr");
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("hello");
    }
}
```
#### 死锁及避免死锁

    循环等待条件会导致死锁，就是A,B两个方法，线程1获得A的锁，然后尝试获得B的锁，线程2获得B的锁然后尝试获得A的锁，这就会形成循环等待，最后形成死锁。

避免死锁的方法

* 加锁顺序
按照顺序加锁，按照顺序加锁是一种有效的死锁预防机制。但是，这种方式需要你事先知道所有可能会用到的锁(译者注：并对这些锁做适当的排序)，但总有些时候是无法预知的。每次线程都同时竞争同一个锁。
* 加锁时限
另外一个可以避免死锁的方法是在尝试获取锁的时候加一个超时时间，这也就意味着在尝试获取锁的过程中若超过了这个时限该线程则放弃对该锁请求。若一个线程没有在给定的时限内成功获得所有需要的锁，则会进行回退并释放所有已经获得的锁，然后等待一段随机的时间再重试。这段随机的等待时间让其它线程有机会尝试获取相同的这些锁，并且让该应用在没有获得锁的时候可以继续运行。线程比较多的情况下可能等待时间是一样的。

* 死锁检测
    每当一个线程获得了锁，会在线程和锁相关的数据结构中（map、graph等等）将其记下。除此之外，每当有线程请求锁，也需要记录在这个数据结构中。

    当一个线程请求锁失败时，这个线程可以遍历锁的关系图看看是否有死锁发生。例如，线程A请求锁7，但是锁7这个时候被线程B持有，这时线程A就可以检查一下线程B是否已经请求了线程A当前所持有的锁。如果线程B确实有这样的请求，那么就是发生了死锁（线程A拥有锁1，请求锁7；线程B拥有锁7，请求锁1）。
    检测出死锁就进行回退，释放所有锁。
    
#### 饥饿与公平
饥饿原因
* 高优先级吞噬所有低优先级线程的CPU时间
* 线程被永久堵塞在一个等待进入同步块的状态。
* 线程在等待一个本身也处于永久等待完成的对象(比如调用这个对象的wait方法)。
#### 阻塞队列
阻塞队列与普通队列的区别在于，当队列是空的时，从队列中获取元素的操作将会被阻塞，或者当队列是满时，往队列里添加元素的操作会被阻塞。

#### 线程池
保持一定线程数量，线程池内线程一直在运行，避免线程频繁建立和销毁，内部实现是阻塞队列，空闲线程从队列中取出任务执行，

#### CAS 乐观锁 非阻塞
CAS（Compare and swap）比较和替换是设计并发算法时用到的一种技术。简单来说，比较和替换是使用一个期望值和一个变量的当前值进行比较，如果当前变量的值与我们期望的值相等，就使用一个新值替换当前变量的值。

volatile多用于单个线程写，多线程读的情况。


    










