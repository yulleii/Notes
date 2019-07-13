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