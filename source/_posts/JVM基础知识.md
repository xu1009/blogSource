---
title: JVM基础知识
date: 2017-07-15 07:46:05
tags: JVM
---

## JVM运行时数据区域
* **程序计数器**   `线程独享`   程序计数器是一块内存较小的存储空间，可以看做当前线程所执行的字节码的行号指示器，字节码解释器工作时就是通过改变这个计数器值来选取下一条需要执行的字节码指令，线程执行的是Java方法的话，这个计数器记录的就是正在执行的字节码指令地址，如果时native方法，那么记录的就是空，该区域没有规定内存错误。
* **Java虚拟机栈** `线程独享`   生命周期和线程相同，每个方法创建时都会创建一个栈帧，用于存储局部变量表（存储了基本数据类型和对象引用类型）、操作数栈、动态链接、方法出口信息。栈帧在栈中的入栈出栈就是对应方法调用到执行的过程。
* **本地方法栈**   `线程独享`  这个和虚拟机栈一样功能，只不过虚拟机栈是为Java方法服务，这个是为本地方法服务的，即为native方法服务。
* **Java堆**   `线程共享`   这个方法是JVM管理的主要区域，也称为GC堆，主要存储的时对象实例和数组。
* **方法区域**   `线程共享`   主要存储的是类信息、常量、静态变量，垃圾回收主要对这个区域中的运行时常量池的回收和对类型的卸载。
## 对象内存布局和定位
* 对象在内存中的存储布局分为3块区域：对象头、实例数据、对齐和填充
* 对象定位有两种，一种时句柄，一种是直接指针，句柄的话就是堆中有一个句柄池存储各个对象实例的地址，栈中的reference可以通过句柄访问，这种访问慢，但是对象地址变化的话，只需要改变句柄池不需要操作栈中的内容，直接指针的话就是访问的比较快，直接操作的对象
## JVM垃圾回收机制
##### 判断什么是垃圾
* **引用计数算法**   引用计数算法是jdk1.2之前的算法，算法的主要过程是，当一个对象被引用时计数器就会加1，引用失效时就减1，最后当这个计数器为0时就会被确认成垃圾，这个有个问题就是无法回收互相引用的对象。
* **可达性分析算法** 可达性分析算法也是目前用来判断对象是否为垃圾的一个算法，它使用图论的知识，他会从一系列GC ROOTS起点向下搜索，当一个节点到GC ROOTS没用任何引用链相连，那么那就会认为他是可回收的。可作为GC ROOTS的对象有java虚拟机栈中的引用的对象，方法区域中静态和常量引用的对象，本地方法栈中的JNI引用的对象，这个算法分析时必须停止其它所有线程。
##### 引用分类
* **强引用**   也是平时最常用的引用，这类引用，只要还存在，垃圾收集器就永远不会回收它，及时出现内存溢出。
* **软引用**    当内存不够时才会回收它，比如用做缓存，描述还有用但是非必要对象。
* **弱引用**   比软引用还要弱的引用，不管内存是否充足都会被回收掉，   如果这个对象是偶尔的使用，并且希望在使用时随时就能获取到，但又不想影响此对象的垃圾收集，那么你应该用 Weak Reference 来记住此对象。
* **虚引用**   也叫幽灵引用，最弱的一种引用关系，无法通过虚引用获取一个对象实例，为一个对象设置虚引用关联的唯一目的就是，当该对象被回收时收到一个系统通知。
##### 垃圾收集算法
* **标记清除算法**    标记清除算法两个过程，一个是标记，一个是清除，这个算法是最基础的垃圾收集算法，主要有两个缺点，一个是标记和清除的效率都很低，另一个是标记清除导致内存碎片，当存储大对象时，由于找不到连续的存储空间，会触发另一次收集。
* **复制算法**    为了解决效率的问题，这个算法主要是将内存分区，一个是用来使用，另一个用来存储，现在商业虚拟机都是分为Eden、from survivor、to survivor，比例时8:1:1，Eden和from survivor用来使用，将存活的对象复制到to survivor，然后回收那些垃圾，当存活的对象大于to   survivor的空间时，那些对象会进入老年代。
* **标记整理算法**   存活的对象向一边移动，这样就解决了内存碎片的问题。
* **分代收集算法**    现在使用大部分都是这个算法，根据新生代和老年代选择合适的算法收集，比如G1收集器使用的就是这个算法。
##### 垃圾收集器
* **serial收集器**   单线程收集器，收集器采用复制算法，所以回收的对象时新生代的对象，收集的时候必须停止其它所有线程，也就是stop the  world直到它收集结束，这个时间也就是停顿时间，`简单高效，多用于client模式下的新生代收集器，由于是一个线程，就避免了切换线程所带来的开销，所以在client模式下是一个很好的选择`
* **parnew收集器**     `serial收集器的多线程版本，其它没啥区别，只是在使用复制算法的时候使用了多线程，在单核cpu的情况下，由于切换线程带来的开销，收集的效率还不如serial，收集器收集期间还是需要停止其它所有线程`
* **parallel scavenge收集器**  `并行的多线程收集器`关注的不是停顿时间，关注的时吞吐量，这种适合那些在后台运算不要交互的任务，还有就是他能自适应调整，这也是和parnew的区别。
* **serialold收集器**  `serial收集器的老年代版本`
* **paraell old收集器**  `parnew收集器的老年代版本`
* **CMS收集器**   `使用标记清除算法，实现了并发收集，就是在收集期间其它用户线程不会停止，GC线程和用户线程交替运行，`缺点是并发收集对cpu资源比较敏感，cpu还要在运行用户线程的同时还要分出一部分去收集，当cpu负荷比较大时cms算法会对程序有很大影响，还有就是无法收集浮动垃圾，然后就是采用标记清除算法的缺点，该收集主要包括四个过程：初始标记、并发标记、重新标记、并发清除。
* **G1收集器**    当前最新的垃圾收集器，主要过程是初始标记、并发标记、最终标记、筛选回收。分代收集、可预测的停顿、还有就是G1之前的收集算法进行收集的范围都是整个老年代或者新生代，G1则是把内存分为若干个大小相等的region，G1跟踪每个region里面垃圾堆积的价值大小，在后台维护一个优先级列表，每次都会回收价值最大的那个region。
##### 内存分配和回收策略
* 大多数情况下，对象在新生代Eden分配，当Eden没有足够的空间进行分配时，就会触发一次minor GC。
###### 对象进入老年代的情况
* 大对象直接进入老年代，JVM提供一个专门设置这个阈值的参数，超过这个阈值的参数直接进入老年代
* 长期存活的对象进入老年代，对象年龄是在survivor没熬过一次minor GC就加1，具体超过多少，也有一个JVM阈值可以设置。
* 相同年龄的对象总和大于survivor空间的一半，那大于等于这个年龄的对象进入老年代。
* 一次minor GC之后，存活的对象进入survivor，最后无法容纳的进入老年代。
##### 一些JVM指令和可视化工具
* 这些都是为了深入观察jvm，比如堆栈，新生代，老年代，线程等，自带的命令行工具，比如jps显示JVM中的进程，jmap生成heap转储快照，jstack生成线程的快照。
* 可视化工具jconsole和visualVM，
##### 有关JVM参数设置问题
* Xmx最大堆内存,通常和Xms搭配使用,配置堆内存的,Xmn配置新生代的内存.
##### 类文件结构
* .class文件,也就是字节码文件,是JVM想要实现语言无关,已经是平台无关了,JVM不和语言绑定,他只和字节码文件相关联,任何其它语言都可以使用JVM作为语言的产品交付媒介,只要这些语言的编译器能够生成符合JVM要求的字节码文件规范.
* class采用一种伪结构来存储数据,这种结构中只有无符号数和表两种类型数据.
* class文件开头是4个字节的魔数,该数标志该文件是否能被JVM接受,很多文件存储标准中都有魔数来进行身份识别.Java中class文件的魔数是咖啡宝贝,紧接着4个字节是class版本号,也就是jdk版本号,低版本jdk不能执行高版本的字节码文件.
* 常量池包含了字面量和符号引用.
* 再接着是访问标志,用于识别这是类还是接口,是否定义为public,是否定义为abstract,是否被final修饰等.
* 类索引,父类索引,接口索引集合,这三项数据来确定这个类的继承关系.这三项都是用于缺点权限定名.
* 字段表集合,用于描述接口或者类中声明的变量.
* 方法表集合,用于描述接口或者类中声明的方法
##### 虚拟机类加载机制
* Java类的加载,链接和初始化都是在程序运行期间完成的,虽然会增加开销,但是提高了灵活性.
* 类整个生命周期包括:加载 --- 验证--- 准备---解析--- 初始化---使用---卸载
* **加载**   通过类权限定名获取该类的二进制字节流,将这个字节流所代表的静态存储结构转化为方法区域的运行时数据结构,在内存中生成一个代表该类的对象.
* **验证**   确保class文件字节流中包含的信息符合当前虚拟机的要求,包括,文件格式验证,元数据验证,字节码验证,符号引用验证.
* **准备**	正式为类变量分配内存并设置类变量初始值.
* **解析**	解析是将符号引用替换为直接引用.
* **初始化**   类加载的最后一步,这里才真正执行类中的Java代码,之前分配的变量都是初始值,现在才是赋值真正的.
###### 加载过程详述
* 这个加载就是前面所说的步骤中的第一步,通过类全限定名获取类的二进制流,这个在Java中是放到JVM之外了,而这个代码模块被称为`类加载器`
* 判断两个类是否相等,必须在同一个类加载器加载的前提下判断才有意义.
* **双亲委派模型** 	Java中分为两种类加载器,一种是启动类加载器还有就是其它,启动类加载器是在JVM内部,其它独立于jvm.
* `启动类加载器`   bootstrap classloader,这个会将Javahome/lib目录下的被jvm识别的类库加载到虚拟机
* `拓展类加载器`  extension classloader,负责加载Javahome/lib/ext中的类库,
* `应用程序类加载器`  application classloader,负责加载用户路径上的类库.
* `双亲委派模型`  表示类加载器之间的层次关系,要求除了启动类加载器之外,其它所有类加载器都要有自己的父类加载器,他们之间不是以继承的关系而是组合的关系.
* `双亲委派模型工作过程`   当一个类加载器收到类加载请求时,他不会自己加载,而是将这个请求传给自己的父类,这样依次向上,最后到了启动类加载器,如果启动类加载器无法加载,再向下传递.这个模型的好处就是类随着他的类加载器一起具备了一种带有优先级的层次关系.
##### JVM执行引擎
* Java虚拟机栈存储的是栈帧,栈帧存储了局部变量表,操作数栈,动态链接和方法返回地址等信息.
* `局部变量表`    这个存放了基本类型和对象引用,局部变量表以变量槽为最小单位.
* `操作数栈`    这个是后入先出的栈
###### Tomcat的类加载器
* 有很多类加载器,一个是分离一个是共享,多级类加载器.
##### Java优化

##### Java并发相关