---
title: 总结一哈java中一些问题
date: 2018-01-17 22:48:49
tags: java NIO 反射
---


### Java NIO Overview

1. channel 
2. buffer
3. selector
> channel与selector搭配使用，必须将selector配置成非阻塞模式，但是filechannel只能配置成阻塞模式，所以不能和selector搭配使用。

#### 非阻塞式服务器

非阻塞是IO管道（pipeline）
![](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-1.png)

> 非阻塞式管道和阻塞式管道最大的区别是他们如何从地城channel（socket或者file）读取数据

IO管道从流中读取数据，并将这些数据拆分为一系列连贯的消息，这个功能模块成为消息读取器，messageReader。
![](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-2.png)
阻塞式的消息读取器就很简单，它从流中每次一个字节的读取，而且读和写都是阻塞的，只有当前的读或者写完成才会返回，对于部分数据读取写入不用考虑,但是阻塞式有一个缺点就是每个线程只能处理一个IO,只有等这个io完成之后才能被释放，这样就会造成为每个操作维护一个线程，每个线程都会占用栈32bit-64bit的内存。所以一百万个线程占用的内存将会达到1TB！虽然可以使用线程池，但是最后还是会有很多问题，而且线程之间上下文的切换的开销也很大。
#### 基础非阻塞是IO管道设计
一个非阻塞式IO管道可以使用一个单独的线程向多个流读取数据。这需要流可以被切换到非阻塞模式。在非阻塞模式下，当你读取流信息时可能会返回0个字节或更多字节的信息。如果流中没有数据可读就返回0字节，如果流中有数据可读就返回1+字节。

这个检查的过程NIO 的selector帮我们完成了，这就节省了很大的工作量，channel都向selector注册，当你调用Selector的select()或者 selectNow() 方法它只会返回有数据读取的SelectableChannel的实例. 
![](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-4.png)

> 非阻塞，有个问题就是数据完整性，messageReader需要判断当前消息是否是完整的，这样部分消息就存储到buffer中了，这就是缓冲区。

关于非阻塞写数据

引入messageWrite，每次写入的字节数不确定，非阻塞是直接返回的，这就是挑战：跟踪部分写入的消息，以便最终可以发送一条消息的所有字节。

当一个消息被写入Message Writer，Message Writer向Selector注册其相关Channel（如果尚未注册）。

当你的服务器有时间时，它检查Selector以查看哪些注册的Channel实例已准备好进行写入。 对于每个写就绪Channel，请求其关联的Message Writer将数据写入Channel。 如果Message Writer将其所有消息写入其Channel，则Channel将再次从Selector注册。

### NIO 和 IO的区别
IO                NIO
面向流            面向缓冲
阻塞IO            非阻塞IO
无                选择器


面向流与面向缓冲

NIO面向缓冲的，读取的数据首先放到缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。

IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。


阻塞与非阻塞

IO中当读取或者写入数据时，必须等待当前写入或者读取完成，不然当前线程会一直被阻塞，不能处理其它事情，而NIO则不是这样，写和读都是立即返回的。

selector

Java NIO的选择器允许一个单独的线程来监视多个输入通道，你可以注册多个通道使用一个选择器，然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得一个单独的线程很容易来管理多个通道。

### 应用环境

NIO的好处就是非阻塞可以实现单线程管理多通道，但是数据的解析就非常麻烦，而IO就是读取全部的数据返回给你，完全没有这个问题。因此应用场合要选好。

一般 NIO适用于数据量少，但是连接量大的，如果需要管理同时打开的成千上万个连接，这些连接每次只是发送少量的数据，例如聊天服务器，实现NIO的服务器可能是一个优势。同样，如果你需要维持许多打开的连接到其他计算机上，如P2P网络中，使用一个单独的线程来管理你所有出站连接，可能是一个优势。

IO的话正好相反，少量连接和大量数据的情况下。

### 反射

> 反射可以让我们在编译期之外的运行期检查类，接口，变量以及方法的信息，在运行期实例化对象，调用方法，通过调用get/set方法获取变量的值。

```java
class Reflection{
    public static void main(String[] args){
        Method[] methods = Reflection.class.getMethods();
        for (Method method : methods){
            System.out.println(method.getName());
        }
    }
}
输出结果：
main
wait
wait
wait
equals
toString
hashCode
getClass
notify
notifyAll
除了main，其它都是从object类继承来的
```
反射获取的方法和域都是public的。

反射获取public的方法，然后可以传参数执行。

* 反射访问私有变量和私有方法









