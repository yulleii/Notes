并发与多线程（应用及底层实现）

- 线程的创建方式
- 多线程应用场景
- 线程状态与转换
- 线程安全与同步机制：volatile vs synchronized vs Lock(ReentrantLock)
- volatile底层原理
- synchronized底层原理及其锁的升级与降级
- Lock(ReentrantLock)底层原理
- ThreadLocal
- 线程通信
- 线程池（底层实现）
- 死锁的出现场景、定位以及修复
- CAS 与 Atomic*类型实现原理
- AQS：并发包基础技术
- Java并发包（java.util.concurrent及其子包）提供的并发工具类
  - 比synchronized更加高级的各种同步结构，如：Semaphore，CyclicBarrier， CountDownLatch
  - 各种线程安全的容器（主要有四类：Queue,List,Set，Map），如：ConcurrentHashMap,ConcurrentSkipListMap,CopyOnWriteArrayList
  - 各种并发队列的实现，如各种BlockingQueue的实现（ArrayBlockingQueue, LinkedBlockingQueue, SynchorousQueue, PriorityBlockingQueue,DelayQueue,LinkedTranferQueue）等。
  - Executor框架与线程池



# 线程的创建方式

三种使用线程的方式：

- 使用Runnable接口
- 实现Callable接口
- 继承Thread类

实现Runnable和Callable接口的类只能当做一个可以在线程中完成的任务，不能真正意义的线程，因此最好还是要通过Thread调用。可以说任务是通过线程驱动从而执行的。

## 实现Runnable

需要实现run（）方法

通过使用Thread调用start()方法来启动线程

```java
public class MyRunnable implements Runnable{
	public void run(){
	
	}
}
```

```java
public static void main(String []args){
    MyRunable instance=new MyRunnable();
    Thread thread=new Thread(instance);
    thread.start();
}
```

## 实现Callable接口

Callable可以有返回值，返回值通过FutureTask进行封装

```java
public class MyCallable implements Callable<Integer>{
    public Integer call(){
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

## 继承Thread类

Thread类也实现了Runable接口，所以也要重写run（）方法

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

## 继承Thread vs 实现接口

实现接口会好一点：

- java不支持多重继承，因此继承了Thread类就无法继承其他类，但是可以实现多接口
- 类可能只要求可执行，继承整个Thread开销太大

# Java内存模型

对特定的内存或高速缓存进行读写访问的过程抽象

## 主内存和工作内存

由于计算机的存储设备与处理器的运算速度有几个数量级的差距，为了解决这种速度矛盾，在他们之间加入高速缓存。

加入高速缓存也带来了一个新问题：**缓存一致性**。如果多个缓存共享一块主内存区域，那么多个缓存数据可能会不一致，需要遵循一些协议。

所有的变量都存储在主内存中，每个线程有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了被该线程使用到的变量的主内存副本拷贝。

> 注意：这里的变量表示是实例字段、静态字段和构成数组对象的元素，但不包括局部变量与方法参数

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

## 内存之间交互操作

![](image/Snipaste_2019-07-17_10-40-34.png)

- read：把一个变量的值从主内存传输到工作内存中
- load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
- use：把工作内存中一个变量的值传递给执行引擎
- assign：把一个从执行引擎接收到的值赋给工作内存的变量
- store：把工作内存的一个变量的值传送到主内存中
- write：在 store 之后执行，把 store 得到的值放入主内存的变量中
- lock：作用于主内存的变量，把一个变量标识为一条线程独占的状态
- unlock：作用于主内存的变量，把一个处于锁定状态的变量释放出来，释放后变量才可以被其他线程锁定

## 内存模型三大特性

### 1. 原子性

Java内存模型保证了内存八大操作的原子性变量操作。但是内存模型允许将没有被volatile修饰的64位数据（long、double）的读写操作划分为两次32位操作来进行，即load、store、read、write操作可以不具备原子性。虽然允许这样操作，但目前各种平台下的商用虚拟机几乎都选择把64位数据的读写操作作为原子操作来对待。

有一个错误认识就是，int 等原子性的类型在多线程环境中不会出现线程安全问题。前面的线程不安全示例代码中，cnt 属于 int 类型变量，1000 个线程对它进行自增操作之后，得到的值为 997 而不是 1000。

![](image/Snipaste_2019-07-17_10-55-50.png)

如上图所示，在T1修改cnt并且还没有将修改后的值写入主内存时，T2依然写入旧值。两个都执行了一次自增运算，但最后值却是1而不是2。因此**对变量读写操作满足原子性只是说明这些load、assign、store等单个操作具有原子性。**AtomicInteger能保证多个线程修改的原子性。如下代码：

~~~java
public void AtomicExample{
    private AtomicInteger cnt=new AtomicInteger();
    public void add(){
        cnt.incrementAndGet();
    }
    public int get(){
        return cnt.get();
    }
}

public static void main(String[]args)throws InterruptedException{
    final int threadSize=1000;
    
}
~~~

除了原子类以外，也可以使用synchronized互斥锁来保证操作的原子性。它对应的内存间交互操作为：lock和unlock，在虚拟机实现上对应的字节码指令为monitorenter和monitorexit

### 2.可见性

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

主要有三种实现方式：

- volatile
- synchronized，对一个变量执行unlock操作之前，必须把变量同步回主内存
- final，被final关键字修饰的字段在构造器中一旦初始化完成，并且没有发生this逃逸（其他线程通过this引用访问到初始化一半的对象），那么其他线程就能看见final字段的值。

### 3.有序性

有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

## 先行发生happens-before原则

如果Java内存模型的所有有序性都仅仅靠volatile和synchronized来完成，那么有一些操作将变得非常繁琐。JVM规定了先行发生原则，让一个操作无需控制就能先于另一个操作完成。

### 1.程序次序规则（Program Order Rule）

在一个线程内，按照程序代码顺序执行，程序前面的操作先行发生于后面的操作

### 2.管程锁定规则（Monitor Lock Rule）

一个unlock操作先行发生于后面对同一个锁的lock操作

### 3.volatile变量规则（Volatile Variable Rule）

对一个volatile变量的写操作先行发生于后面对这个变量的读操作

### 4. 线程启动规则（Thread Start Rule）

Thread对象的start（）方法先行发生于线程的每一个操作

### 5. 线程加入规则（Thread Join Rule）

Thread 对象的结束先行发生于对此线程的终止检测。Thread.join()、Thread.isAlive()

### 6. 线程中断规则（Thread Interruption Rule）

对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生。

### 7. 对象终结规则(Finalizer Rule)

一个对象的初始化完成（构造函数执行结束）先行发生于它的 finalize() 方法的开始。

### 8. 传递性(Transitivity)

如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。