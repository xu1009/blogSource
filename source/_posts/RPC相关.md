---
title: RPC相关
date: 2018-01-17 22:42:15
tags: RPC 序列化
---




> 序列化就是将对象转化为二进制流，不同序列化框架会将对象转成不同二进制流。在RPC调用中，不可避免的要接触到数据的序列化和解序列化。Java对象在一个网络节点中序列化好，放到网络中传输，到另一个网络节点时必须能够反序列化，对象得到正确的解析，这是RPC里的最基本问题。

### Java序列化机制
* Java序列化的对象需要实现serialabel接口，没有特殊指定的话，就是用默认的序列化方式，该序列化方式会将对象本身，及对象的引用全部序列化，开销比较大，（深拷贝的应用）依赖于jvm，可以实现不同平台之间的序列化和反序列化。
* Java序列化就是将对象转换成一堆byte，就是二进制的序列化。
Java序列化后包含的信息是：
1. 所有非静态且未声明未transient的成员；
2. 类的固有信息，如字段名，方法名，版本等；
3. 父类的序列化信息；
4. 对象成员变量的序列化信息。


### hessian序列化
* hessian主要用于java序列化。它的实现机制是着重于数据，附带简单的类型信息的方法
* 对于简单的数据类型。就像Integer a = 1，hessian会序列化成I 1这样的流，I表示int or Integer，1就是数据内容
* 对于复杂对象，通过Java的反射机制，hessian把对象所有的属性当成一个Map来序列化，产生类似M className propertyName1 I 1 propertyName S stringValue
* 对于引用对象，在序列化过程中，如果一个对象之前出现过，hessian会直接插入一个R index这样的块来表示一个引用位置，从而省去再次序列化和反序列化的时间
### thrift序列化
Thrift是Facebook开源提供的一个高性能，轻量级RPC服务框架，其产生正是为了满足当前大数据量、分布式、跨语言、跨平台数据通讯的需求。 但是，Thrift并不仅仅是序列化协议，而是一个RPC框架。相对于JSON和XML而言，Thrift在空间开销和解析性能上有了比较大的提升，对于对性能要求比较高的分布式系统，它是一个优秀的RPC解决方案；但是由于Thrift的序列化被嵌入到Thrift框架里面，Thrift框架本身并没有透出序列化和反序列化接口，这导致其很难和其他传输层协议共同使用（例如HTTP）。
### protobuf序列化
序列化数据非常简洁，紧凑，析速度非常快，提供了非常友好的动态库。使用简介，反序列化只需要一行代码。但是在JavaBean和proto之间的转换较麻烦。
### avro序列化
* Avro的产生解决了JSON的冗长和没有IDL的问题。 Avro提供两种序列化格式：JSON格式或者Binary格式。Binary格式在空间开销和解析性能方面可以和Protobuf媲美，JSON格式方便测试阶段的调试。
* 动态类型：Avro并不需要生成代码，模式和数据存放在一起，而模式使得整个数据的处理过程并不生成代码、静态数据类型等等。这方便了数据处理系统和语言的构造。
* 未标记的数据：由于读取数据的时候模式是已知的，那么需要和数据一起编码的类型信息就很少了，这样序列化的规模也就小了。
* 不需要用户指定字段号：即使模式改变，处理数据时新旧模式都是已知的，所以通过使用字段名称可以解决差异问题。


### thrift

thrift是目前使用比较多的rpc框架，使用IDL中间语言编写相关，然后再生成相应语言，因此可以实现跨语言的框架。

* 基于二进制的高性能编解码
* 基于NIO的底层通信
* 相对简单的服务调用模型
* 使用IDL支持跨平台
thrif核心组件有：
* TProtocol 协议和编解码组件
* TTransport 传输组件
* TProcessor 服务调用组件
* TServer，Client 服务器和客户端组件

RPC中客户端和服务器药传输的数据有：
客户端
* 方法名称，包括类名称和方法名称
* 方法参数，包括参数类型和参数值
* 附加数据，包括附件，超时信息，自定义的控制信息。
服务器
* 调用的返回码
* 返回值
* 异常信息
发送消息的组合方式在IDL中都定义好了，服务器和客户端是一一对应的，所以不会存在解析问题。

1. writeMessgeBegin方法写了消息头，包括4字节的版本号和类型信息，字符串类型的方法名，４字节的序列号seqId

2. writeFieldBegin，写了１个字节的字段数据类型，和2个字节字段的顺序号

序列化消息头加消息体的格式，先写消息头再写消息体，消息头包括4字节的版本号和调用方法的类型，字符串类型的方法名，4字节的序列号，消息体就是写方法的参数，接下来就是写字段，传入的具体参数类型及数值。方法参数的顺序服务器和客户端保持一致，客户端写的顺序和服务器端读的顺序一致，也就是按照方法入口参数的顺序来写的。

thrift中IDL支持的数据类型
bool 布尔型
byte ８位整数
i16  16位整数
i32  32位整数
i64  64位整数
double 双精度浮点数
string 字符串
binary 字节数组
list<i16> List集合，必须指明泛型
map<string, string> Map类型，必须指明泛型
set<i32> Set集合，必须指明泛型 

IDL生成的类主要有5个部分
* 接口类型，服务端使用它作为顶层接口，编写接口实现，客户端用它作为生成代理的服务接口。
* 客户端类型，一个同步客户端，一个异步客户端。
* processor，用来支持方法调用，服务的实现类都要使用processor注册。
* 方法参数封装类。
* 方法返回值封装类。

1. TProcessor就定义了一个顶层的调用方法process，参数是输入流和输出流

2. 抽象类TBaseProcessor提供了TProcessor的process的默认实现，先读消息头，拿到要调用的方法名，然后从维护的一个Map中取ProcessFunction对象。ProcessFunction对象是实际方法的抽象，调用它的process方法，实际是调用了实际的方法。

3. Processor类是自动生成了，它依赖Iface接口，负责把实际的方法实现和方法的key关联起来，放到Map中维护
核心类：
1. 自动生成的Iface接口，是远程方法的顶层接口
2. 自动生成的Processor类及相关父类，包括TProcessor接口，TBaseProcess抽象类

3. ProcessFunction抽象类，抽象了一个具体的方法调用，包含了方法名信息，调用方法的抽象过程等

4. TNonblcokingServer，是NIO服务器的默认实现，通过Args参数来配置Processor等信息
5. FrameBuffer类，服务器NIO的缓冲区对象，这个对象在服务器端收到全包并解码后，会调用Processor去完成实际的方法调用
6. 服务器端的方法的具体实现类，实现Iface接口

TProcessor 是一个接口，只定义了一个方法，process，TBaseProcessor实现了该接口，首先读取消息头，拿到要调用的方法名，然后根据方法名从Map中拿出一个processFunction对象，该对象是具体方法的抽象，调用它的process方法就是调用了该方法，framebuffer用于NIO服务器保存中间态，
![](http://img.blog.csdn.net/20140930104345373?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSVRlcl9aQw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
### frameBuffer
这个是NIO的服务端的核心组件，一方面是缓冲区，另一方面承担了RPC方法调用。framebuffer提供了invoke方法，当读满包时，就会通过他管理的processor来完成实际方法的调用。
![](http://img.blog.csdn.net/20140930114738752?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSVRlcl9aQw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
framebuffer 写消息体responseReady()
1. 创建ByteBuffer
2. 修改状态到AWAITING_REGISTER_WRITE
3. 调用requestSelecinterestChange()注册change的写事件。
4. Selector会根据状态调用FrameBuffer的write方法


### Transport传输层分析
TIOStreamTransport和TSocket是阻塞同步IO,TSocket封装了Socket接口。

TNonblockingTransport，TNonblockingSocket对应非阻塞IO。

thrift协议和具体的传输对象绑定的。使用具体的transport来读写数据。thrift使用NIO服务器读取数据，每次先读取消息头，消息头是封装了消息的长度，然后再读取消息体，所以在客户端发送消息时，消息的封装也使用了相同的方式，这里都是使用TframedTransport包装流来传输数据
### TServer服务器分析
Thrift使用Tserver作为服务器的抽象，使用TServerTransport作为服务器的acceptor抽象，监听端口，创建客户端socket连接，TServerTransport分为两种，阻塞和非阻塞。
1. TNonblockingServerTransport和TNonblockingServerSocket作为非阻塞IO的Acceptor,封装了ServerSocketChannel
2. TServerSocket作为阻塞同步IO的Acceptor，封装了ServerSocket

TProtocol序列化
1. write/read Message，读写消息头，消息头包含了方法名，序列号等信息

2. write/read Struct，将RPC方法的参数/返回值封装成结构体，读写结构体即表示要读写RPC方法参数了

3. write/read Field，每一个参数都被抽象成Field，Field主要包含了字段的索引信息，类型信息等

4. write/read Type，即读写各种具体的数据
来一段服务器和客户端的代码感受一下，下面是TNonBlockingServer


```java
package com.yangyang.thrift.tnonblockingserver;

import com.yangyang.thrift.api.HelloService;
import com.yangyang.thrift.service.HelloServiceImpl;
import org.apache.thrift.TException;
import org.apache.thrift.TProcessor;
import org.apache.thrift.protocol.TCompactProtocol;
import org.apache.thrift.server.TNonblockingServer;
import org.apache.thrift.transport.TFramedTransport;
import org.apache.thrift.transport.TNonblockingServerSocket;

/**
 * **
 * 注册服务端
 *  使用非阻塞式IO，服务端和客户端需要指定 TFramedTransport 数据传输的方式。 TNonblockingServer
 * Created by chenshunyang on 2016/10/31.
 */
public class HelloTNonblockingServer {
    // 注册端口
    public static final int SERVER_PORT = 8080;

    public static void main(String[] args) throws TException{
        //处理器
        TProcessor processor = new HelloService.Processor<HelloService.Iface>(new HelloServiceImpl());
        // 传输通道 - 非阻塞方式
        TNonblockingServerSocket serverTransport = new TNonblockingServerSocket(SERVER_PORT);
        //异步IO，需要使用TFramedTransport，它将分块缓存读取。
        TNonblockingServer.Args tArgs = new TNonblockingServer.Args(serverTransport);
        tArgs.processor(processor);
        // 使用非阻塞式IO，服务端和客户端需要指定TFramedTransport数据传输的方式
        tArgs.transportFactory(new TFramedTransport.Factory());
        //使用高密度二进制协议
        tArgs.protocolFactory(new TCompactProtocol.Factory());

        TNonblockingServer server = new TNonblockingServer(tArgs);
        System.out.println("HelloTNonblockingServer start....");
        server.serve();


    }
}

```

```java
package com.yangyang.thrift.tnonblockingserver;

import com.yangyang.thrift.api.HelloService;
import org.apache.thrift.TException;
import org.apache.thrift.protocol.TCompactProtocol;
import org.apache.thrift.protocol.TProtocol;
import org.apache.thrift.transport.TFramedTransport;
import org.apache.thrift.transport.TSocket;
import org.apache.thrift.transport.TTransport;

/**
 *
 * 客户端调用HelloTNonblockingServer,HelloTHsHaServer
 * 非阻塞
 * Created by chenshunyang on 2016/10/31.
 */
public class HelloTNonblockingClient {
    public static final String SERVER_IP = "127.0.0.1";
    public static final int SERVER_PORT = 8080;
    public static final int TIMEOUT = 30000;

    public static void main(String[] args) throws TException{
        //设置传输通道，对于非阻塞服务，需要使用TFramedTransport，它将数据分块发送
        TTransport transport = new TFramedTransport(new TSocket(SERVER_IP,SERVER_PORT,TIMEOUT));

        // 协议要和服务端HelloTNonblockingServer一致,使用高密度二进制协议
        TProtocol protocol = new TCompactProtocol(transport);

        HelloService.Client client = new HelloService.Client(protocol);
        transport.open();
        String result = client.hello("jack");
        System.out.println("result : " + result);
        //关闭资源
        transport.close();

    }
}

```











