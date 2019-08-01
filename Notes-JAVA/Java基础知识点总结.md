# String vs StringBuffer vs StringBuilder

## String

作为一个对象，且String是一个**final类**，不能被继承也不可变。一旦创建就不能修改它的值

那么，平时我们使用“+”来拼接字符串是什么实现的？

在每次对 String 类型进行改变的时候其实都等同于生成了一个新的 String 对象，然后将指针指向新的 String 对象。

## StringBuffer

- 一个类似于 String 的字符串缓冲区，对它的修改的不会像String那样重创建对象。 
- 使用append()方法修改Stringbuffer的值，使用toString()方法转换为字符串。
- 线程安全的，建议多线程使用

## StringBuilder

- StringBuild是jdk1.5后用来替换stringBuffer的一个类，大多数时候可以替换StringBuffer。和StringBuffer

- 区别在于Stringbuild是一个单线程使用的类，不值执行线程同步所以比StringBuffer的速度快，效率高。

- 线程非安全的，建议单线程使用

> 注意:不能通过赋值符号对他进行赋值.

## VS

### String vs StringBuffer

主要区别在于String是一个final类，每次对String的更改都是重新生成一个新的String对象，旧的对象被回收。当经常使用连接操作的字符串不要使用String，每次更改都会产生垃圾。当内存中无引用对象多了以后，JVM的GC就会开始工作。

### StringBuffer vs StringBuilder

StringBuffer线程安全，而StringBuilder线程非安全。多线程场景使用StringBuffer来支持并发操作，而StringBuilder其在单线程的性能要比StringBuffer高。

### String vs StringBuffer vs StringBuilder

三者在字符串连接操作的执行速度方面比较：StringBuilder>StringBuffer>String