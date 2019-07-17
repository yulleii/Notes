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
- CAS 与 Atomic类型实现原理
- AQS：并发包基础技术
- Java并发包（java.util.concurrent及其子包）提供的并发工具类
  - 比synchronized更加高级的各种同步结构，如：Semaphore，CyclicBarrier， CountDownLatch
  - 各种线程安全的容器（主要有四类：Queue,List,Set，Map），如：ConcurrentHashMap,ConcurrentSkipListMap,CopyOnWriteArrayList
  - 各种并发队列的实现，如各种BlockingQueue的实现（ArrayBlockingQueue, LinkedBlockingQueue, SynchorousQueue, PriorityBlockingQueue,DelayQueue,LinkedTranferQueue）等。
  - Executor框架与线程池



# 并发编程

## 上下切换

CPU通过时间片分配算法来循环执行任务，当前任务执行一个时间片后会切换到下一个任务。但是，在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这个任务的状态。**任务从保存到再加载的过程就是一次上下文切换。**

线程有创建和上下文切换的开销。

减少上下文切换的开销的方法有：

- 无锁并发编程，例如将数据的ID按照Hash算法取模分段，不同的线程处理不同段的数据
- CAS算法
- 使用最少线程和使用协程，在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换

## 锁的状态

为了减少获得锁带来的性能消耗，引入了“偏向锁”和“轻量级锁”。在Java1.6中锁一共有4种状态，级别从低到高分别是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态。

锁可以升级但不能降级，目的是为了提高获得锁和释放锁的效率

### 偏向锁

为了让线程获得锁的代价更低而引入了偏向锁。默认启用，但是是在应用程序启动几秒钟以后才激活。使用`-XX:BiasedLockingStartupDelay=0`来调整激活时间，使用`XX:UseBiasedLocking=false`来关闭偏向锁。

当一个线程访问同步块并获取锁是，会在**对象头和栈帧中的锁记录里存储锁偏向的进程ID**，以后该进程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需要简单地测试一下**对象的头的Mark Word**是否存储着指向当前线程的偏向锁。如果测试成功，表示线程已经获得了锁，否则需要再测试一下Mark Word偏向锁的标识是否设置成1：如果没有设置，则使用CAS竞争锁；否则尝试使用CAS将对象头的偏向锁指向当前线程。

偏向锁是一种**等到竞争出现才释放锁的机制**。如果CAS获取偏向锁失败，则表示有竞争。当到达全局安全点（safepoint）时获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。（撤销偏向锁的时候会导致stop the word）

### 轻量级锁

线程在执行同步块之前，JVM会先在当前线程的栈帧中创建用于**存储锁记录的空间**，并将对象头中的Mark Word复制到锁记录中（Displaced Mark Word）。然后线程尝试使用**CAS将对象头的Mark Word替换**为指向锁记录的指针。如果替换成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用**自旋来获取锁**。

### 重量级锁

轻量级解锁时，会使用原子的CAS操作将Displaced Mark Word替换回到对象头上，如果成功，则表示没有竞争发生。如果失败，表示当前锁存在竞争，锁就会膨胀成重量级锁。

当锁处于这个状态，其他线程试图获取锁时，都会被阻塞住，当持有锁的进程放锁之后唤醒这些线程，被唤醒的线程进行新一轮的夺锁。

| 锁       |                             优点                             |                       缺点                       |               适用场景               |
| -------- | :----------------------------------------------------------: | :----------------------------------------------: | :----------------------------------: |
| 偏向锁   | 加锁和解锁不需要额外的消耗，和执行非同步方法比仅存在纳秒级的差距。 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗。 |  适用于只有一个线程访问同步块场景。  |
| 轻量级锁 |          竞争的线程不会阻塞，提高了程序的响应速度。          |  如果始终得不到锁竞争的线程使用自旋会消耗CPU。   | 追求响应时间。同步块执行速度非常快。 |
| 重量级锁 |              线程竞争不使用自旋，不会消耗CPU。               |             线程阻塞，响应时间缓慢。             |   追求吞吐量。同步块执行速度较长。   |

## CAS

Compare and Swap：CAS操作需要输入两个数值，一个旧值和一个新值，在操作期间先比较旧值有没有发生变化，如果没有发生变化，才交换成新值，发生变化则不交换。

JVM中的CAS操作利用了处理器提供的CMPXCHG指令实现的。

## 原子操作

**使用循环CAS实现原子操作的三大问题**

- ABA问题。如果一个原来是A，变成了B，又变成了C，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上发生变化了。解决思路：**使用版本号，在变量前面追加上版本号，**每次更新变量的时候把版本号加1，那么A-B-A就会变成1A-2B-3A。JDK的Atomic包提供了一个类AtomicStampedReference来解决ABA问题。

  ![](image/Snipaste_2019-07-17_21-43-52.png)

- 循环时间长开销大。自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。pause指令

- 只能保证一个共享变量的原子操作。可以使用锁，除此之外还可以把多个共享变量合成一个共享变量来操作。JDK提供AtomicReference类来保证对象引用之间的原子性。

**使用锁机制实现原子操作**

锁机制保证了只有获得锁的线程才能够操作锁定的内存区域。除了偏向锁，JVM实现锁的方式都用了循环CAS，即当一个线程想进入同步块的时候使用循环CAS的方式来获取锁，当它退出同步块的时候使用循环CAS释放锁。

# 线程的状态与转换

![](image/Snipaste_2019-07-17_20-02-59.png)

## *NEW*

初始状态，线程被构建，但是还没有调用start（）方法

## *Runnable*

可能在运行，也可能在等待CPU时间片

包含操作系统的Running和Ready状态

## *Blocked*

获取一个排他锁，如果其线程释放了锁就会结束此状态。线程阻塞在synchronized关键字修饰的方法或代码块时的状态。

> 但是java.current包中Lock接口的线程状态却是等待状态，因为java.current包中的Lock接口对于阻塞的实现均使用了LockSupport类中的方法。

## *WAITING*

等待状态，表示当前线程需要等待其他线程做一些特定的动作

| 进入方法                                   | 退出方法                             |
| :----------------------------------------- | ------------------------------------ |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法                    | LockSupport.unpark(Thread)           |

## *TIME_WAITING*

超时等待状态，可以在指定的时间自行返回。无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。

调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。

睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。

| 进入方法                                 | 退出方法                                        |
| ---------------------------------------- | ----------------------------------------------- |
| Thread.sleep() 方法                      | 时间结束                                        |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
| LockSupport.parkNanos() 方法             | LockSupport.unpark(Thread)                      |
| LockSupport.parkUntil() 方法             | LockSupport.unpark(Thread)                      |

## *TERMINATED*

终止状态，表示当前线程已经执行完毕

# 线程通信

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

## *join()*

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

具体请看JavaPractice中TestJoin

## *wait() notify() notifyAll()*

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。

使用 wait() 挂起期间，线程会释放锁。如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

**wait() 和 sleep() 的区别**

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。

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
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    WaitNotifyExample example = new WaitNotifyExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
```

## *await() signal() signalAll()*

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
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    AwaitSignalExample example = new AwaitSignalExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
```

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

### 1.原子性

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

**1.程序次序规则**

Program Order Rule，在一个线程内，按照程序代码顺序执行，程序前面的操作先行发生于后面的操作

**2.管程锁定规则**

Monitor Lock Rule，一个unlock操作先行发生于后面对同一个锁的lock操作

**3.volatile变量规则**

Volatile Variable Rule，对一个volatile变量的写操作先行发生于后面对这个变量的读操作

**4. 线程启动规则**

Thread Start Rule，Thread对象的start（）方法先行发生于线程的每一个操作

**5. 线程加入规则**

Thread Join Rule，Thread 对象的结束先行发生于对此线程的终止检测。Thread.join()、Thread.isAlive()

**6. 线程中断规则**

Thread Interruption Rule，对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生。

**7. 对象终结规则(Finalizer Rule)**

一个对象的初始化完成（构造函数执行结束）先行发生于它的 finalize() 方法的开始。

**8. 传递性(Transitivity)**

如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。

# 线程安全与同步机制

## volatile

如果一个变量被volatile修饰，那么Java线程内存模型确保所有线程看到这个变量的值是一致的。

### 底层原理

有volatile变量修饰的共享变量进行写操作的时候，汇编代码中会出现**Lock前缀的指令**。该指令在多核处理器会引发两件事情：

- 将当前处理器缓存行的数据写回到系统内存。Lock信号一般不锁总线，而是锁缓存，锁总线开销的比较大。
- 这个写回内存的操作会使在其他CPU缓存了**该内存地址的数据无效**。处理器使用嗅探技术保证它的内部缓存、系统内存和其他处理器的缓存数据在总线上保持一致。如果通过嗅探一个处理器来检测其他处理器打算写内存地址，而这个地址当前处于共享状态，那么正在嗅探的处理将使它的缓存行无效，在下次访问相同内存地址时，强制缓存行填充。

## synchronized

实现原理：**monitorenter**指令是在编译后插入到同步代码块的开始位置，而**monitorexit**是插入到方法结束处和异常处，从而保证每个monitorenter必须有对应的monitorexit与之匹配。

## Lock（ReentrantLock）

