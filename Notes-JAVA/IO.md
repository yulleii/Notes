# IO基础

## 流的分类

- 数据单位：字节流、字符流
- 流向：输入输出
- 角色：节点流、处理流、缓冲流、转换流

|   抽象基类   |      节点流      |        缓冲流        |       转换流       |
| :----------: | :--------------: | :------------------: | :----------------: |
| InputStream  | FileInputStream  | BufferedInputStream  |                    |
| OutputStream | FileOutputStream | BufferedOutputStream |                    |
|    Reader    |    FileReader    |    BufferedReader    | InputStreamReader  |
|    Writer    |    FileWriter    |    BufferedWriter    | OutputStreamWriter |

非文本文件只能使用字节流，例如视频、图片、音频等

缓冲流是用来提升文件的效率，在最后写完后需要加上flush来填满buffer

编码：字符->字节 OutputStreamWriter

解码：逆向，InputStreamReader

## 对象流

**1.对象的序列化过程**

将内存中的对象通过ObjectOutputStream转换为二进制流，存储在硬盘文件中

要求

- 此类必须是可序列化的，即必须实现Serializable或者Externlizable接口
- 类的属性也要是可序列化的，实现Serializable接口
- 加入序列版本号：private static final long

注意：不能序列化static和transient修饰的成员变量

**2.对象的反序列化过程**

将硬盘中的文件通过ObjectInputStream转换为相应的对象

# 一些概念

## 同步与异步

- 同步：发起一个调用以后，**被调用者未处理完请求之前**，调用不返回；

- 异步：发起一个调用以后，**立刻得到被调用者的回应**表示已经接受到请求，但是被调用者并没有返回结果。此时我们可以处理其他请求，被调用者通常依靠事件和回调等机制来通知调用者其返回结果。

## 阻塞和非阻塞

- 阻塞：发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，只有当条件就绪才能继续。

- 非阻塞：非阻塞就是发起一个请求，调用者不用等待结果返回，可以先去做其他事情。

# BIO（Blocking I/O）

## 传统I/O

同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。

采用BIO通信模型的服务端，由一个独立的Acceptor线程负责监听客户端的连接。我们一般通过在`while(true)`循环中服务端会循环调用`accecpt()`方法等待接收客户端的连接的方式监听请求，请求一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能接收其他客户的请求，只能等待同当前连接的客户端的操作执行完成后。

可以通过**多线程来支持多个客户端的连接**。为每一个客户端创建一个新的线程进行链路处理，处理完成后，通过输出流返回给客户端，线程销毁。典型的**一请求一应达通信模型**。如果这个线程不做任何事情，就会造成不必要的开销。可以通过**线程池机制**来改善，线程池可以让线程的创建和回收成本相对较低，使用`FixedThreadPool`可以有效地控制线程的最大数量，保证了系统有限的资源控制。

当客户端并发访问量增加以后这种模型会出现什么问题？

**线程的创建、销毁成本很高同时线程的切换成本也是很高的**。尤其在Linux系统中，线程本质就是一个进程，创建和销毁都是重量级的系统函数。如果并发访问量增加会导致线程数急剧膨胀**可能导致堆栈溢出、创建新线程失败**等问题，最终导致进程宕机或者僵死，不能对外提供服务。

## 伪异步IO

对传统IO进行优化——后端通过一个**线程池来处理多个客户端的请求接入**，形成客户端个数M:线程池最大线程数N的比例关系。M可以远大于N，通过线程池来灵活地调配线程资源，设置线程的最大值，防止海量并发接入导致线程耗尽。

新客户接入时，将客户端的Socket封装成一个Task（该任务实现Runnable接口）投递到后端的线程池中进行处理，JDK的**线程池维护一个消息队列和N个活跃线程**，对消息队列中的任务进行处理。这样一来，它的**资源占用是可控的**，无论多少个客户端并发访问，都不会导致资源的耗尽和宕机。

底层仍然是同步阻塞的BIO模型，无法从根本上解决问题。

```
在活动连接数不是特别高的情况下，这种模型是比较不错的。但是当面对十万甚至百万级别的连接时，传统的BIO模型也是无能为力的。
```

# NIO（New IO）

同步非阻塞IO模型，对应java.nio，提供了Channel、Selector、Buffer等抽象

支持面向缓存的，基于通道的IO操作方法。NIO提供了与传统模型中断Socket和ServerSocket相对应的*SocketChannel*和*ServerSocketChannel*两种不同的套接字通道实现，两种通道都支持阻塞和非阻塞两种模式。**阻塞模式**使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；**非阻塞模式**正好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发。

## NIO的特性或者NIO与IO的区别

首先肯定要从 NIO 流是非阻塞 IO 而 IO 流是阻塞 IO 说起。然后，可以从 NIO 的3个核心组件/特性为 NIO 带来的一些改进来分析。

- **IO流是阻塞的，NIO流是非阻塞的。**NIO可以进行非阻塞IO操作，例如单线程中从通道读取数据到buffer，同时可以继续做其他事情，当数据读取到buffer中后，线程再继续处理数据。而IO的各种流都是阻塞的，当一个线程调用`read`或者`write`时，线程被阻塞，直到数据被完全读写完成。

- **IO面向流，NIO面向缓冲区。**面向流的IO将数据直接写入或将数据直接读到Stream对象中。虽然Stream中也有Buffer开头的扩展类，但是只是流的包装类，还是从流读到缓冲区，而NIO却是直接读到Buffer中进行操作。

  在NIO库中，所有数据都用缓冲区处理。读取数据时，它是直接读到缓冲区中；写入数据，也是写入到缓冲区中。

  每一种Java基本类型都对应有一种缓冲区。

- **Channel（通道）**：和传统IO中的Stream是一个差不多的等级。NIO通过Channel进行读写。通道是双向的，可读也可写。而流的读写是单向的。无论读写，通道只能和Buffer交互。因为Buffer，通道可以异步读写。NIO中的Channel的主要实现有：FileChannel、DatagramChannel、SocketChannel、ServerSocketChannel

- **Selectors（选择器）：**选择器用于使用单个线程处理多个通道。因此，它需要较小的线程来处理这些通道。线程之间的切换对于操作系统来说是昂贵的。为了提高效率，选择器是必要的。

## Buffer

NIO中的关键Buffer实现有：ByteBuffer、CharBuffer、DoubleBuffer、 FloatBuffer、IntBuffer、 LongBuffer,、ShortBuffer，分别对应基本数据类型: byte、char、double、 float、int、 long、 short。当然NIO中还有MappedByteBuffer, HeapByteBuffer, DirectByteBuffer等这里先不具体陈述其用法细节。

**DirectByteBuffer 与 HeapByteBuffer 的区别？**

它们 ByteBuffer 分配内存的两种方式。HeapByteBuffer 顾名思义其内存空间在 JVM 的 heap（堆）上分配，可以看做是 jdk 对于 byte[] 数组的封装；而 DirectByteBuffer 则直接利用了系统接口进行内存申请，其内存分配在直接内存中，这样就减少了内存之间的拷贝操作。如此一来，在使用 DirectByteBuffer 时，系统就可以直接从内存将数据写入到 Channel 中，而无需进行 Java 堆的内存申请、复制等操作，提高了性能。

既然如此，为什么不直接使用 DirectByteBuffer，还要来个 HeapByteBuffer？原因在于， DirectByteBuffer 是通过full gc来回收内存的，DirectByteBuffer会自己检测情况而调用 system.gc()，但是如果参数中使用了 DisableExplicitGC 那么就无法回收该快内存了。-XX:+DisableExplicitGC标志自动将 System.gc() 调用转换成一个空操作，就是应用中调用 System.gc() 会变成一个空操作，那么如果设置了就需要我们手动来回收内存了，所以DirectByteBuffer使用起来相对于完全托管于 java 内存管理的Heap ByteBuffer 来说更复杂一些，如果用不好可能会引起OOM。Direct ByteBuffer 的内存大小受 -XX:MaxDirectMemorySize JVM 参数控制（默认大小64M），在 DirectByteBuffer 申请内存空间达到该设置大小后，会触发 Full GC。

## Selector

NIO实现多路复用的基础，运行单线程来处理多个Channel。要使用Selector需要先向Selector注册Channel，然后调用select（）方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦方法返回，线程就可以处理这些事件。

## NIO读写数据的方式

NIO所有的IO都是从Channel开始的。

- 从通道进行数据读取：创建一个缓冲区，然后请求通道读取数据
- 从通道进行数据写入：创建一个缓冲区，填充数据，并要求通道写入数据

## NIO的局限

- 编程复杂、编程模型难
- NIO的底层由epoll实现，空轮询bug会导致CPU飙升100%
- 项目庞大以后，自行实现的NIO很容易出现各类bug，维护成本较高

# AIO（Asynchronous IO）异步非阻塞IO

Java对异步IO提供支持的NIO.2，在JDK 7中实现。异步IO是基于事件和回调机制实现

NIO需要使用者线程不停的轮询IO对象，来确定是否有数据准备好可以读了，而AIO则是在数据准备好之后，才会通知数据使用者，这样使用者就不需要不停地轮询了。当然AIO的异步特性并不是Java实现的伪异步，而是使用了系统底层API的支持，在Unix系统下，采用了epoll IO模型，而windows便是使用了IOCP模型。