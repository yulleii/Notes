- JVM运行时数据区域 vs Java内存模型 （**这2不一样！！！**）
- Java内存模型与happen-before原则
- 内存泄露、内存溢出以及栈溢出
- JVM类加载机制及其作用与对象的初始化
- JVM垃圾回收
  - 如何判断对象已经死亡？引用计数法 vs 可达性分析
  - 如何回收对象？垃圾收集算法
  - Minor GC vs Full GC
  - 常用的垃圾收集器及其特点
  - 内存分配与回收策略
- GC调优
  - GC调优的思路
  - JVM常用参数
  - 基于JDK命令行工具监控Java进程， 如 jps,jinfo,jstat,jmap,jstack
  - 基于图形化工具监控Java进程，如MAT(Memory Analyzer),VisualVM,Btrace

# Java运行时数据区域

## 程序计数器

程序执行的字节码的行号指示器

特点：线程私有，此内存区域是Java虚拟机规范唯一一个没有规定任何OutOfMememory情况的区域

## 虚拟机栈

每个方法在执行的同时都会创建一个栈帧用于存储局部变量表、操作数栈、动态链接、方法出口等信息。

其中局部变量表包括：各种基本数据类型、**对象引用类型**

特点：线程私有，生命周期与线程相同

异常：StackOverflowError和OOM

## 本地方法栈

虚拟机使用的Native方法服务

异常：StackOverflowError和OOM

## Java堆

存放对象实例

特点：Java虚拟机所管理的内存中最大的一块；被所有线程共享的一块内存区域；垃圾收集管理的主要区域

Java堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的。

异常：OOM

## 方法区

存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据

特点：各个线程共享的区域；不要连续的内存空间；可以选择不进行垃圾收集

异常：OOM

### 运行时常量池

常量池，用于存放编译期生成的各种字面量和符号引用

异常：OOM

## 直接内存

并不是虚拟机运行时数据的一部分，也不是Java虚拟机规范中定义的内存区域。

NIO类，引用一种基于通道和缓冲区的IO方式。使用Native函数库**直接分配**堆外内存，通过存储在Java堆中的**DirectByteBuffer对象**作为这块内存的引用进行操作。避免了堆和Native堆来回复制，提升了性能。

# 垃圾收集器与内存分配策略

程序计数器、虚拟机栈、本机方法栈随线程而生，随线程而灭；这几个区域的内存分配和回收都具有确定性，不需要过多考虑回收的问题。而**Java堆和方法区** ，一个接口中的多个实现类需要的内存可能不一样，一个方法的中的多个分支需要的分支可能不一样，只有在程序处于运行期间时才能知道会创建哪些对象。

## 判断对象是否已经死亡

### 引用计数算法

引用时加一，引用失效时减一

缺点：很难解决对象之间相互循环引用的问题

### 可达性分析算法

通过一系列的称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连时，则证明该对象时不可用的。

> 可以作为GC Roots的对象
>
> - 虚拟机栈中引用的对象
> - 方法区中静态属性引用的对象
> - 方法区中常量引用的对象
> - 本地方法中JNI引用的对象

### 引用的分类

**强引用（Strong Reference）**

垃圾收集器永远不会回收掉被引用的对象

**软引用（Soft Reference）**

系统发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存，才会抛出内存溢出异常。

**弱引用（Weak Reference）**

只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。

**虚引用**

不会对其生存时间构成影响，无法通过一个虚引用来取得一个对象的实例。唯一目的就是能在这个对象被收集器回收时收到一个系统通知。

### 回收Java堆——两次标记来决定一个对象死亡

第一个阶段：经过可达性分析以后发现没有与GC Roots相连接的对象，被第一次标记并进行一次筛选。筛选的条件时是否有必要执行**finalize()**方法。当对象没有覆盖finalize（）方法时或者finalize（）方法已经被虚拟机调用过，视为“没有必要执行”。当这个对象有必要执行finalize（）方法，那么这个对象会放置在F-Queue之中，稍后由虚拟机自动建立的、低优先级的Finalizer线程去执行它。

> 这里的执行是指虚拟机会触发这个方法，但并不承诺等待它运行结束。原因是如果对象在finalize（）方法中执行缓慢，或者发生了死循环，将可能会导致F-Queue队列中的其他对象永久处于等待，甚至导致整个内存回收系统崩溃。

第二个阶段：GC将对F-Queue中的对象进行第二次小规模的标记，如果对象要在finalize（）中成功拯救自己，只要重新与引用链上的任何一个对象建立关联即可，那么它将在第二次标记时被移出“即将回收”的集合；如果对象这时候还没有逃脱，那就被真正的回收了。

> 任何一个对象的finalize（）方法只会被系统自动调用一次

### 回收方法区

主要回收两部分内容：废弃的常量和无用的类

**废弃的常量**：和回收对象相似，如果没有任何String对象引用常量池中的“abc”常量，也没有其他地方引用了这个字面量，如果这时候发生内存回收，而且有必要的话，那么这个“abc”常量会被系统清理出常量池。

**无用的类**：判定依据有三点

- 该类所有的实例已经被回收
- 加载该类的ClassLoader已经被回收
- 该类对应的java.lang.class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

## 垃圾收集算法

### 垃圾-清除算法（Mark-Sweep）

效率问题，标记和清除两个过程的效率都不高

空间问题：标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程序分配大对象时，无法找到足够的连续内存而不得不提起触发另一次GC。

### 复制算法（Copying）

将可用内存按容量分为大小相等的两块，每次只使用其中的一块。当一块的内存使用完了，就将还存活着的对象**复制**到另一块上面，然后再把已使用过的内存空间一次清理掉。

优点：不需要考虑内存碎片问题，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。

缺点：算法的代价为将内存缩小了原来的一半。即使有8:1的分配，**对象存活率较高时**就要进行很多的复制操作，效率会变低。除此之外还需要额外的空间作为分配担保，不适合所有对象100%存活的极端情况，**不适合老年代使用**。

HotSpot复制算法策略：将内存分为一块较大的Eden和两块较小的Survivor，每次使用Eden和其中一块Survivor。回收时，将Eden和Survivor中还存活的对象一次性复制到另一块Survivor中，最后清理掉Eden和Survivor空间。Eden：Survivor=8:1。

> 如果另一块Survivor空间没有足够空间存放上一次新生代收集下来的存活对象，这些对象将直接通过**分配担保**机制进入老年代。

### 标记-整理算法（Mark-Compact）

根据老年代的特点，提出该算法。先进行标记，然后让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

### HotSpot收集算法实现

#### GCRoots枚举

可达性分析工作必须在一个能确保一致性的快照中进行，GC进行时必须停顿所有Java执行线程（**Stop The World**）

不需要一个不漏地检查完所有执行上下文和全局的引用位置，虚拟机使用一组称为**OopMap**的数据结构直接得知哪些地方存放着对象引用。

#### 安全点

只有在特定位置上来记录信息，只有在到达安全点时才能暂停。

这些位置的选定标准：是否具有让程序长时间执行的特征。指令序列复用，例如方法调用、循环跳转、异常跳转，具有这些功能的指令才会产生SafePoint。

问题：如何GC发生时让所有线程都跑到最近的安全点上再停顿下来？

抢先式中断：所有线程中断，如果发现线程中断的地方不在安全点上，就恢复线程，让它跑到安全点上。

主动式中断：GC需要中断时，不直接堆线程操作，仅仅简单地设置一个轮询标志，各个线程执行时主动去轮询这个标志，发现中断标志为真时就自己中断挂起。

#### 安全区域

如果线程处于Blocked或者Sleep状态时，无法响应JVM的中断请求。

安全区域是指一端代码片段之中，引用关系不会发生变化。在这个区域中的任意地方开始GC都是安全的。

当线程执行到Safe Region中的代码时，首先标识自己已经进入了Safe Region，当JVM发起GC时，就不用管自己为Safe Region状态的线程了。在线程离开安全区时，它要检查系统是否已经完成了根节点枚举，如果完成了，那线程就继续执行否则它就等待直到收到可以安全离开Safe Region的信号为止。

## 垃圾收集器

HotSpot垃圾收集器的图片

### Serial收集器

**单线程**收集器，进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束。新生代采用复制算法，老年代采用标记-整理算法

优点：简单高效；在用户的桌面应用场景中，分配给虚拟机管理的内存不会很大（几十兆或者几百兆）停顿时间完全可以控制在几十毫秒最多一百毫秒以内。使用于运行在**Client模式**下的虚拟机

### ParNew收集器

Serial收集器的多线程版本，使用于工作在**Server模式**的虚拟机，**目前只有它能与CMS收集器配合工作**

在单CPU的环境中绝对不会有比Serial收集更好的效果

### Parallel Scavenge收集器

吞吐量优先收集器：**关注点不同**，其他收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间，而Parallel Scavenge收集器的目的是达到一个可控制的吞吐量。吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值。

设置参数：最大垃圾收集停顿时间 -XX:MaxGCPauseMillis 直接设置吞吐量大小 -XX:GCTimeRatio，开关参数：-XX:+UseAdaptiveSizePolicy，不需要设置手工指定新生代大小、比例、晋升老年代对象大小等细节参数，**GC自适应的调节策略**。这点是与Parnew收集器重要区别。

### Serial Old收集器

单线程，标记整理。主要意义是在于Client模式的虚拟机使用。

如果在Server模式有两大用途：在JDK1.5版本以及之前的版本中，与Parallel Scavenge收集器搭配。作为CMS收集器的后备预案，Concurrent Mode Failure时使用。

### Parallel Old收集器

Parallel Scavenge的老年代版本，使用多线程和“标记-整理”算法。

适用于注重吞吐量以及CPU资源敏感的场合，可以优先考虑Parallel Scavenge 加 Parallel Old。

### CMS收集器

一种获取最短回收停顿时间为目标的收集器，重视服务器的响应速度，希望系统停顿时间最短，给用户带来较好的体验。

标记-清除的运作过程：

**初始标记**只是标记一下GC Roots能直接关联到的对象，速度很快。**并发标记**阶段进行GC Roots Tracing的过程，而**重新标记**阶段则是为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象，这个阶段的时间比初始时间稍长，但远比并发标记时间短。最后**并发清除**标记的对象。

缺点

- CMS收集器对CPU资源非常敏感。虚拟机提供了一种“增量式并发收集器”，尽量减少GC线程的独占资源的时间。效果一般，已经被弃用。
- CMS收集器无法处理浮动垃圾，可能出现Concurrent Mode Failure失败而导致另一次Full GC的产生。由于CMS并发清理阶段用户线程还在运行，伴随程序运行自然还会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS无法在当次收集中处理掉它们，这一部分垃圾称为浮动垃圾。CMS收集器需要预留一部分空间提供并发收集时的程序运作使用。JDK1.6CMS的启动阈值为92%，要是CMS运行期间预留的内存无法满足程序需要就会出现一次“Concurrent Mode Failure”。
- 收集结束会有大量的空间碎片产生。给分配大对象带来很大麻烦，如果没有找到足够的空间来分配当前对象就会触发一次Full GC。使用-XX：+UseCMSCompactAtFullCollection开关参数开启内存碎片的合并整理过程，内存过程无法并发，空间碎片没有了，但停顿时间不得不变长了。

### G1收集器

面向服务端应用的垃圾收集器

G1收集器运作：初始标记、并发标记、最终标记、筛选回收

特点

- 并行与并发：充分利用多CPU、多核环境的优势
- 分代收集
- 空间整合：整体使用标记-整理，局部之间使用复制。将Java堆分为多个大小相等的独立区域。
- 可预测的停顿：相对于CMS的另一大优势。建立**可预测的停顿时间模型**，有计划的避免进行全区域的垃圾收集。G1跟踪各个Region里面的垃圾堆积的价值大小（回收获得的空间大小以及回收所需时间的经验值） 在后台维护一个优先列表，每次根据允许的收集时间，**优先**回收价值最大的Region。从而来保证在有限的时间内可以获取尽可能高的收集效率。

每个Region都有一个与对应的Remembered Set来避免全堆扫描。

## 内存分配与回收策略

- 大多数对象优先在Eden中分配，当Eden区没有足够空间进行分配时，虚拟机发起一次Minor GC
- 大对象直接进入老年代，使用-XX：PretenureSizeThreshold参数令大于设置值对象直接在老年代中分配。该参数只能在Serial和ParNew中使用
- 长期存活的对象直接进入老年代。每熬过一次Minor GC年龄就增加一岁，当超过默认15岁就会晋升到老年代。通过设置-XX:MaxTenuringThreshold
- 动态对象年龄判定：如果在Survivor空间中相同年龄所有对象大小的总和大于或等于Survivor空间的一半，年龄大于或等于该年龄的对象直接进入老年代
- 空间分配担保：在发生Minor GC前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间或者历次晋升的平均大小，如果条件成立，则可以确保Minor GC是安全的。否则就会进行一次Full GC。

## Minor GC vs Full GC

Full gc就是回收整个堆，包括新生代、老年代。

- 当要触发一次Minor GC时，如果发现young GC的平均晋升大小比目前的老年代的剩余空间大，则不会触发Minor GC而是触发一个FullGC。除了CMS的ConCurrent Collection，其他能收集old gen的GC都会同时收集整个GC
- System.gc（）默认触发Full GC
- 老年代空间不足

minor gc回收的对象只有新生代。当新生代的Eden区满时就会触发MinorGC

# 类文件结构

将编写的程序编译成二进制本地机器码（Native Code）已经不是唯一选择，越来越多的程序语言选择**与操作系统和机器指令集无关的、平台中立的格式作为程序编译后的存储格式**。

## 字节码

各种不同平台的虚拟机与所有平台统一使用的程序存储格式——字节码（ByteCode）

Java虚拟机不和包括Java在内的任何语言绑定，它只与**“Class文件“**这种特定的二进制文件格式所关联，Class文件包含了Java虚拟机指令集和符号表以及若干其他辅助信息。

## Class类文件的结构

> 任何一个Class文件对应着唯一一个类或者接口的定义信息，但类或接口不一定都得定义在文件里。（譬如类或接口可以通过类加载器直接生成）

Class文件是一组以8位字节为基础单位的二进制流，严格按照顺序，中间没有任何分隔符。

主要有两种数据类型

- 无符号数：用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值，u1、u2、u4、u8
- 表：多个无符号数或者其他表组成，用于描述有层次关系的复合结构的数据

使用**容量计数器**加若干个连续的数据项的形式来描述同一类型但数量不定的多个数据

#### 魔数+Class文件版本

头4个字节称为Magic Number，用来确定为一个被虚拟机接受的Class文件。值为0xCAFEBABE

紧接着的4个字节存储Class文件版本号，5、6（次）+7、8（主）

#### 常量池

接着主次版本号之后是常量池入口，Class文件之中的资源仓库。

特点：与其他项目关联最多的数据类型；占用Class文件空间最大；第一个出现表类型的数据项目

常量池中主要存储两大类常量

- 字面量（Literal）：文本字符串、声明为final的常量值
- 符号引用（Symbolic References）：类和接口的全限定名、字段的名称和描述符、方法的名称和描述符

例如存储一个Class_info的数据：

1. u2类型的数据，代表容量池计数值。从1开始，0在特定情况下表达“不引用任何一个常量池项目“
2. 表开始的第一位是一个u1类型的标志位（Tag）代表这个常量属于哪种常量类型（14+3种）。不同的表具有不同的数据结构，以下为Class_info的数据结构
3. u2类的name_index，指向常量池的偏移，假设从池中找到第二项常量的标志为0x01，表示为Utf8的常量。
4. u2类的length，最大长度为65535。
5. u1类型的字符串的内容，长度为length。使用Utf8缩写编码：1-127 一个字节表示，后续部分两个，最后一部分三个。
6. 后续常量和上述描述相同

> 注意：常量池中有一些在代码中没有出现过的常量，是为字段表、方法表、属性表引用所需


#### 访问标志
常量池结束以后，紧接着两个字节代表访问标志，包括：Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是类的话是否被定义为final等

#### 类索引、父类索引与接口索引集合
类索引和父类索引为u2类型（单继承），接口索引为一组u2类型的集合（多实现）

#### 字段表集合

用于描述接口或者类中声明的变量。包括类级变量以及实例级变量，但不包括在方法内内部声明的局部变量

全限定名：类的全限定名，org/fenixsoft/clazz/TestClass;

 方法和字段的描述符：用来描述字段的数据类型、方法的参数列表和返回值

- 基本数据类型（byte、char、double、float、int、long、short、boolean）使用一个大写字符来表示
- 数组类型：每一维度将使用一个前置的“[”来描述
- 描述方法：按照参数列表，后返回值的顺序描述

字符段集合：容量计数器（u2）+access_flags(u2)+descriptor_index(u2)+（属性集合）

#### 方法表集合

方法表集合结构

计数器（u2）+access_flags（u2）+name_index(u2)+attributes_count+attribute_name_index

对于方法的重载：特征签名就是一个方法中各个参数在常量池中的字段符号引用的集合

#### 属性表集合

不再要求各个属性具有严格顺序，只要不与已有属性名重复。

**Code属性**

用于存储方法里的Java代码，但接口或者抽象类的中的方法不在Code属性中

属性值的长度等于整个属性表的长度减去6个字节（attribute_name_index(u2)+attribute_length(u4)）

**Slot**是虚拟机为局部变量分配内存所使用的最小单位，局部变量中的Slot可以重用

attribute_length虽然为为u4，最大为2的32次方减一，但是虚拟机规范明确规定了一个方法中

**后续还有很多属性介绍，详细介绍参照<<深入理解JVM>>**

## 字节码指令介绍

一个字节

优点：放弃操作数长度对齐，可以省略很多填充和间隔符号。短小精悍的编译代码，小数据量、高传输效率。

缺点：虚拟机处理那些超过一个字节数据的时候，不得不在运行时从字节中重建出具体数据的结构，执行时会损失一些性能。

## 虚拟机的公有设计和私有设计

一个优秀的虚拟机实现，在满足虚拟机规范的约束下对具体实现做出修改和优化也是完全可行的。

虚拟机实现者可以使用这种伸缩性来让Java虚拟机获得更高的性能、更低的内存消耗或者更好的可移植性。

# 虚拟机类加载机制

代码编译的过程就是将本地机器码转换字节码的过程。

类的加载机制：把描述类的数据从Class文件中加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java模型。

类是在程序运行期间第一次使用时动态加载的，而不是一次加载所有类。

![](image/classloader.png)

一个类的生命周期：加载、验证、准备、解析、初始化、使用以及卸载。其中解析阶段可以在初始化以后再开始

## new一个对象的过程

- 在方法区查找这个类是否已经被加载，如果没有加载这个类
- 为这个类分配内存：1.指针碰撞法 2.查表法
- 对所有实例变量赋默认值
- 对象头设置（哈希码、GC分代年龄等）
- 执行初始化代码


## 类加载的过程

包含加载、验证、准备、解析、初始化

### 1.加载

主要完成以下3件事情：

- 通过一个类的全限定名来获取定义此类的二进制字节流
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
- 在内存中生成一个代表这个类的Class对象，作为方法区这个类的各种数据的访问入口

其中二进制字节流可以从以下方式获取：

- 从ZIP包中读取，成为JAR、EAR、WAR格式的基础
- 从网络中获取，应用场景Applet
- 运行时计算生成，例如动态代理技术，在 java.lang.reflect.Proxy 使用 ProxyGenerator.generateProxyClass 的代理类的二进制字节流。
- 由其他文件生成，应用场景JSP应用，由JSP文件生成对应的Class类

### 2.验证

为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全

### 3.准备

为类变量分配内存并设置类变量初始值的阶段。其中**类变量仅包括被static修饰的静态变量**，使用的是方法区的内存

实例变量不会在这阶段分配内存，它会在对象实例化时随着对象一起被分配在堆中。应该注意到，实例化不是类加载的一个过程，类加载发生在所有实例化操作之前，并且类加载只进行一次，实例化可以进行多次。

初始值一般为 0 值，例如下面的类变量 value 被初始化为 0 而不是 123。

```java
public static int value = 123;
```

如果类变量是常量，那么它将初始化为表达式所定义的值而不是 0。例如下面的常量 value 被初始化为 123 而不是 0。

```java
public static final int value = 123;
```

### 4.解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程

其中解析过程在某些情况下可以在初始化阶段之后再开始，这是为了支持 Java 的动态绑定。

### 5.初始化

根据程序员通过程序制定的主观计划去初始化类变量和其他资源。也就是执行类构造器`<clinit>()`方法的过程

`<clinit>()`方法是由编译器自动收集类中的所有类变量的赋值操作和静态语句块中的语句合并产生的。收集的顺序由语句在源文件中出现的顺序决定。**特别注意的是，静态语句块只能访问到定义在静态语句块之前的变量，定义在它之后的变量只能赋值，不能访问。**

~~~java
public class Test{
    static{
        i=0;//正常编译通过
        System.out.println(i);//编译器提示“非法向前引用”
    }
    static int i=1;
}
~~~

虚拟机会保证子类`<clinit>()`方法执行之前，父类的方法已经执行完毕。这也就意味着父类定义的静态语句优先于子类的变量赋值操作。如下：

```java
static class Parent {
    public static int A = 1;
    static {
        A = 2;
    }
}

static class Sub extends Parent {
    public static int B = A;
}

public static void main(String[] args) {
     System.out.println(Sub.B);  // 2
}
```

如果没有静态语句块或者变量的赋值操作，那么编译器可以不为这个类生成`<clinit>()`方法

执行接口的`clinit<>()`方法不需要先执行父接口的`<clinit>()`。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的 `<clinit>() `方法。

虚拟机会保证一个类的` <clinit>() `方法在多线程环境下被正确的加锁和同步，如果多个线程同时初始化一个类，只会有一个线程执行这个类的` <clinit>() `方法，其它线程都会阻塞等待，直到活动线程执行 `<clinit>()` 方法完毕。如果在一个类的 `<clinit>()` 方法中有耗时的操作，就可能造成多个线程阻塞，在实际过程中此种阻塞很隐蔽。

## 初始化时机

### 1.主动引用

- 遇到new、getstatic、putstatic或者invokestatic4条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。最常见的Java代码场景是：使用new、读取或者设置一个类的静态字段（被final修饰、已在编译器期把结果放入常量池的静态字段除外）、调用一个类的静态方法时
- 使用java.lang.reflect包方法对类进行反射调用的时候，如果类没有初始化，则先需要触发初始化
- 初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先初始化其父类
- 当虚拟机启动时，用户需要指定一个要执行的主类（包含 main() 方法的那个类），虚拟机会先初始化这个主类
- 当使用 JDK 1.7 的动态语言支持时，如果一个 java.lang.invoke.MethodHandle 实例最后的解析结果为 REF_getStatic, REF_putStatic, REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化；

### 2.被动引用

- 通过子类来引用父类中定义的静态字段时，只会触发父类的初始化而不会触发子类的初始化

```java
System.out.println(SubClass.value);  // value 字段在 SuperClass 中定义
```

- 通过数组引用类，不会触发类的初始化。该过程会对数组类进行初始化，数组类是一个由虚拟机自动生成的、直接继承自 Object 的子类，其中包含了数组的属性和方法

```java
SuperClass[] sca = new SuperClass[10];
```

- 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。对常量的引用会被转化为类对自身常量池的引用

```java
System.out.println(ConstClass.HELLOWORLD);
```

## 类加载器

### 类与类加载器

比较两个类是否相等，需要类本身相等，且使用同一个类加载器进行加载。因为每一个类加载器都有一个独立的类名称空间。

这里的相等包括代表类的equals方法、isAssignalbeFrom（）方法、isInstance（）方法的返回结构，也包括使用instanceof关键字做对象所属关系判定等情况。

### 类加载器的分类

从 Java 虚拟机的角度来讲，只存在以下两种不同的类加载器：

- 启动类加载器，使用C++实现，是虚拟机自身的一部分。
- 所有其它类的加载器，使用 Java 实现，独立于虚拟机，继承自抽象类 java.lang.ClassLoader。

从Java开发人员的角度，可以分的更为详细一点

- 启动类加载器（Bootstrap ClassLoader）此类加载器负责将存放在 <JRE_HOME>\lib 目录中的，或者被 -Xbootclasspath 参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如 rt.jar，名字不符合的类库即使放在 lib 目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被 Java 程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给启动类加载器，直接使用 null 代替即可。
- 扩展类加载器（Extension ClassLoader）这个类加载器是由 ExtClassLoader（sun.misc.Launcher$ExtClassLoader）实现的。它负责将 <JAVA_HOME>/lib/ext 或者被 java.ext.dir 系统变量所指定路径中的所有类库加载到内存中，开发者可以直接使用扩展类加载器。
- 应用程序类加载器（Application ClassLoader）这个类加载器是由 AppClassLoader（sun.misc.Launcher$AppClassLoader）实现的。由于这个类加载器是 ClassLoader 中的 getSystemClassLoader() 方法的返回值，因此一般称为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。  

### 双亲委派模型

应用程序是由三种类加载器互相配合从而实现类加载，除此之外还可以加入自己定义的类加载器。

这里类加载器之间的父子关系一般不会以继承的关系实现，而是都使用组合关系来复用父加载器的代码

![](image/parentDelegationModel.png)

**工作过程：**如果一个类加载器收到一个类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成。只有当父类加载器反馈自己没有办法完成这个加载请求时，子加载器才会尝试自己去加载。

**好处**：Java类随着它的类加载器一起具备了一种带有**优先级的层次关系**。例如 java.lang.Object 存放在 rt.jar 中，如果编写另外一个 java.lang.Object 并放到 ClassPath 中，程序可以编译通过。由于双亲委派模型的存在，所以在 rt.jar 中的 Object 比在 ClassPath 中的 Object 优先级更高，这是因为 rt.jar 中的 Object 使用的是启动类加载器，而 ClassPath 中的 Object 使用的是应用程序类加载器。rt.jar 中的 Object 优先级更高，那么程序中所有的 Object 都是这个 Object。

**实现**

以下是抽象类 java.lang.ClassLoader 的代码片段，其中的 loadClass() 方法运行过程如下：先检查类是否已经加载过，如果没有则让父类加载器去加载。当父类加载器加载失败时抛出 ClassNotFoundException，此时尝试自己去加载。

```java
public abstract class ClassLoader {
    // The parent class loader for delegation
    private final ClassLoader parent;

    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }

    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
}
```

## 自定义类加载器实现

以下代码中的 FileSystemClassLoader 是自定义类加载器，继承自 java.lang.ClassLoader，用于加载文件系统上的类。它首先根据类的全名在文件系统上查找类的字节代码文件（.class 文件），然后读取该文件内容，最后通过 defineClass() 方法来把这些字节代码转换成 java.lang.Class 类的实例。

java.lang.ClassLoader 的 loadClass() 实现了双亲委派模型的逻辑，自定义类加载器一般不去重写它，但是需要重写 findClass() 方法。

```java
public class FileSystemClassLoader extends ClassLoader {

    private String rootDir;

    public FileSystemClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = getClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] getClassData(String className) {
        String path = classNameToPath(className);
        try {
            InputStream ins = new FileInputStream(path);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 4096;
            byte[] buffer = new byte[bufferSize];
            int bytesNumRead;
            while ((bytesNumRead = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    private String classNameToPath(String className) {
        return rootDir + File.separatorChar
                + className.replace('.', File.separatorChar) + ".class";
    }
}
```