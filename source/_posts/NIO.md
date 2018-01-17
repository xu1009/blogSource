---
title: NIO
date: 2017-09-07 20:55:49
tags:　java nio
---

* channels
* buffers
* selectors
> channel 相当于数据来源，buffer相当于缓冲区。selector相当于一个监听器，NIO的主要特点就是实现非阻塞的IO，以单线程的方式实现多路通道的监听。

![](http://ifeve.com/wp-content/uploads/2013/06/overview-channels-buffers1.png)

java NIO中的channel主要包括以下几种类型：

* filechannel
* datagramchannel
* socketchannel
* serversocketchannel

buffer也有好几种类型：
* bytebuffer
* charbuffer
* doublebuffer
* floatbuffer
* intbuffer
* longbuffer
* shortbuffer

selector允许线程处理多个channel,channel向selector注册，然后调用它的select()方法，这个方法会一直阻塞到某个注册的通道有事件就绪。

![](http://ifeve.com/wp-content/uploads/2013/06/overview-selectors.png)


NIO和流有一些不同：
* NIO既可以从通道中读取数据，又可以写数据到通道，但是流的读写通常是单向的
* 通道可以实现异步读写
* 通道中的数据总要先经过buffer

```java
   public static void main(String[] args){
        try {
            RandomAccessFile file = new RandomAccessFile("tst.dat", "rw");
            FileChannel channel = file.getChannel();
            ByteBuffer byteBuffer = ByteBuffer.allocate(48);  // 48 byte
            int byteRead = channel.read(byteBuffer);   // 
            while (byteRead != -1){
                System.out.println(byteRead);
                byteBuffer.flip();  // transfer wrote mode to read mode
                while (byteBuffer.hasRemaining()){
                    System.out.println(byteBuffer.get());
                }
                byteBuffer.clear();
                byteRead = channel.read(byteBuffer);
            }
            file.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
关于buffer

buffer有三个属性，capacity、position、limit。分别是什么意思呢，下图就很清晰

![](http://ifeve.com/wp-content/uploads/2013/06/buffers-modes.png)
在读模式和写模式中，这几个意思不太一样。

* capacity 就是buffer固定的大小，就是  ByteBuffer byteBuffer = ByteBuffer.allocate(48);这个，就是你初始化的buffer的大小
* position 在写模式中，就表示当前位置，position最大值很明显就是容量  capacity - 1，在读模式中，从写模式切到读模式时，需要使用 byteBuffer.flip();这时position会被重置为0，从头开始读，最大到limit。
* limit，在写模式中，limit就是capacity，在读模式中，limit就是上次写模式的position。

clear 和compact

都是清空为了下次接收，clear是清空全部，compact则清空buffer中已读的数据。


分散和聚集

分散（scatter）从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。

聚集（gather）写入Channel是指在写操作时将多个buffer的数据写入同一个Channel，因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。

![](http://ifeve.com/wp-content/uploads/2013/06/scatter.png)
![](http://ifeve.com/wp-content/uploads/2013/06/gather.png)


通道之间的数据传输


其实这个也挺方便的，之间在通道之间就将数据传输了，就相当于copy了一份

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

toChannel.transferFrom(position, count, fromChannel);



RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```


Selector

Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。


多线程导致线程之间切换上下文的开销很大，而且每个线程都要占用系统的一些资源，使用的线程越少越好。

使用selector必须使channel工作在非阻塞模式下，但是filechannel不能工作在非阻塞模式，所以不能使用selector。


selector可以监听四种事件，connect、accept、read、write。

通道在selector注册之后会返回一个selectionKey对象，这个对象包含了你感兴趣的集合

* interest集合
* ready集合
* channel
* selector

非阻塞式服务器

一个非阻塞式IO管道是由各个处理非阻塞式IO组件组成的链。其中包括读/写IO。下图就是一个简单的非阻塞式IO管道组成：

![](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-1.png)





