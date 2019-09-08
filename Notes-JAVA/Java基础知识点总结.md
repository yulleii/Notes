# Java基础

- 新版本特性
- 跨平台特性
- Java四个基本特性
- Object类及其方法实现（尤其是equals() and hashCode()）
- 基本数据类型与引用数据类型
- Exception vs Error
- finally vs final vs finalize
- final vs static
- 四种引用：强引用、软引用、弱引用、幻象引用的比较
- 重载 vs 重写
- 接口 vs 抽象类
- 深克隆 vs 浅克隆
- String vs StringBuffer vs StringBuilder （底层实现）
- IO vs NIO vs AIO
- 反射与动态代理
- 序列化与反序列化（底层实现）

# Java8新特性

引入lambda表达式和函数式引用来简洁代码

将Java内存中的永久代变为了元数据区

接口有了默认方法和静态方法

注解可以使用重复注解，也可以注解在任何地方

# Object类及其方法

getclass（）用于获取对象的运行时类，在反射时候用的比较多

~~~java
public final native Class<?> getClass();
~~~

不能重写，实现在C++层

还有hashCode（）计算对象的hash值。

equals方法，判断两个对象是否相等的方法。true表示两个对象相等，false不等

clone方法：是protected修饰的，如果子类不显示的重写clone，那么就无法使用clone。并且需要实现cloneable接口。clone方法可以实现浅拷贝和深拷贝，引用对象的不同。浅拷贝和原来的对象共用一个对象，而深拷贝新创建一个对象，二者引用的是不同对象。**使用clone拷贝一个对象既复杂又有风险，它也会抛出异常，并且还需要类型转换。可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象**

# 接口和抽象类

接口：属性默认是public static final的，方法默认都是public abstract

抽象类：本质还是一个类，所有可以有属性、方法、构造器。抽象类中可以有抽象方法，也可以没有。有抽象方法的类一定是抽象类。

# 基本数据类型与引用数据类型

## 包装类Wrapper

### 基本数据类型、包装类与String类之间的转换

```java
//基本数据类型、包装类 ---> String类:调用String类的重载的valueOf(XXX x)方法
int i1=10;
String str1=i1+"";//"10"

String str2=String.valueOf(i1);
String str3=String.valueof(true);//"true"
//String 类 ---> 基本数据类型、包装类：调用包装类的parseXxx(String str)方法
int i3=Integer.parseInt(str2);
```

### Integer用==比较127相等而128不相等的原因

自动装箱时，并不一定new出来一个新的对象。**Integer.class在装载（Java虚拟机启动）时，其内部类型IntegerCache的static块即开始执行，实例化并暂存数值在-128到127之间的Integer类型对象。当自动装箱int型值在-128到127之间时，即直接返回IntegerCache中暂存的Integer类型对象。**

为什么Java这么设计？出于效率考虑，因为自动装箱经常遇到，尤其是小数值的自动装箱；而如果每次自动装箱都触发new，在堆中分配内存，就显得太慢了；所以不如预先将那些常用的值提前生成好，自动装箱时直接拿出来返回。哪些值是常用的？就是-128到127了

不仅int，Java中的另外7种基本类型也都可以自动装箱和拆箱，其中也有用到缓存：

| 基本类型 | 装箱类型  | 取值范围           | 是否缓存 | 缓存范围        |
| :------- | :-------- | :----------------- | :------- | :-------------- |
| byte     | Byte      | -128 ~ 127         | 是       | -128 ~ 127      |
| short    | Short     | -2^15 ~ (2^15 - 1) | 是       | -128 ~ 127      |
| int      | Integer   | -2^31 ~ (2^31 - 1) | 是       | -128 ~ 127      |
| long     | Long      | -2^63 ~ (2^63 - 1) | 是       | -128～127       |
| float    | Float     | --                 | 否       | --              |
| double   | Double    | --                 | 否       | --              |
| boolean  | Boolean   | true, false        | 是       | true, false     |
| char     | Character | \u0000 ~ \uffff    | 是       | \u0000 ~ \u007f |

> 注意
>
> ~~~java
> Integer a=new Integer(1);
> Integer b=new Integer(1);
> //a!=b 原因在于new强制产生两个不同的对象，而没有使用缓存IntegerCache的对象
> ~~~

### String的相等问题的讨论

`String s1="s"`首先判断常量池中是否有“s”，如果有直接返回。如果没有，则在heap中创建新的String对象，将其引用返回，同时将该引用添加至常量池中。

~~~java
String s1="s";
String s2="s";//s1==s2,
~~~

`String s3=new String ("i")`立即在heap中创建一个String对象，然后将该对象的引用返回给用户。并不会主动将该引用放入到常量池中

~~~java
String s3=new String ("i");
String s4=new String("i");//s3!=s4

String s5=new String ("a");
String s6="a";//s5!=s6
~~~

`intern()`作用：首先查看常量池有没有对象，如果没有，则在堆中新建一个对象，然后将新对象引用放入到常量池中;如果有，直接返回它的引用。

~~~java
String s7="b";
s7=s7.intern();//intern
String s8="b";//s7==s8
~~~

关于字符串的衔接操作时的小问题：

~~~java
String str1 = "hello quanjizhu";
String str2 ="hello" +"quanjizhu";
 String str3 ="hello "+"quanjizhu";//在编译的时候会优化成String str3 = "hello quanjizhu";所有str1和str2指向的是同一内存地址。

String var = “quanjizhu“;
String str4 = “hello “+var;//输出结果是false,证明了String str4 = “hello “+var;在内存堆中会重新分配空间，而不是让str4指向var的地址。
str4 = (“hello“+var4).intern();
//intern()方法告诉编译器将此结果放到String pool里，此,System.out.println(str1= =str4）输出结构将是true;
~~~

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

- 区别在于StringBuilder是一个单线程使用的类，不值执行线程同步所以比StringBuffer的速度快，效率高。

- 线程非安全的，建议单线程使用

> 注意:不能通过赋值符号对他进行赋值.

## VS

### String vs StringBuffer

主要区别在于String是一个final类，每次对String的更改都是重新生成一个新的String对象，旧的对象被回收。当经常使用连接操作的字符串不要使用String，每次更改都会产生垃圾。当内存中无引用对象多了以后，JVM的GC就会开始工作。

### StringBuffer vs StringBuilder

StringBuffer线程安全，而StringBuilder线程非安全。多线程场景使用StringBuffer来支持并发操作，而StringBuilder其在单线程的性能要比StringBuffer高。

### String vs StringBuffer vs StringBuilder

三者在字符串连接操作的执行速度方面比较：StringBuilder>StringBuffer>String

# 基本数据类型

## 默认值

| **数据类型**           | **默认值**  |
| :--------------------- | :---------: |
| byte                   |      0      |
| short                  |      0      |
| int                    |      0      |
| long                   |     0L      |
| float                  |    0.0f     |
| double                 |    0.0d     |
| char                   | **'u0000'** |
| String (or any object) |    null     |
| boolean                |    false    |

\u0000表示Unicode编码的空字符

## 长度

| 数据类型 | 长度  |
| -------- | :---: |
| int      | 4字节 |
| short    | 2字节 |
| long     | 8字节 |
| byte     | 1字节 |
| float    | 4字节 |
| double   | 8字节 |
| char     | 2字节 |
| boolean  | 1字节 |

## 字符的ASCII码

常用的char字符有控制字符对应的ASCII为0~31，32是空格。

33~126为可显示的字符。其中字符0~9对应48~57，字符A~Z对应65~90，小写字母a~z对应97~122。

（“789”）

# 集合框架

## Vector vs ArrayList vs LinkedList

### Vector

底层基于数组实现，默认大小为10，Vector对数组的操作都是通过加锁的方式保证数据写入的一致性。

Vector具备数组索引快，增删慢的特性，因为增删都需要移动数组元素位置；而数组支持通过下标可以快速索引元素，时间复杂度O（1），但是增删一个元素时间复杂度为O（n-1）n。

### ArrayList

ArrayList底层也是基于数组实现的，默认大小10。但是ArrayList是线程不安全的，单线程下ArrayList的性能要比Vector更出色。扩容：Vecotr默认扩展容量是原来的一倍，而ArrayList是原来的1/2

> 值得注意的是：每次进行增删操作时，除了扩容或者减少容量这样的常规操作外，还有一个变量在默默增加，它就是modCount。它是用来记录ArrayList结构变化的次数，起作用的地方是在使用iterator时，判断modCount是否等于ExpectedModCount，如果不等于将会抛出ConcurrentModificationException异常。也就是说设计者用modCount来规避多线程中的并发问题，由此也可以看出ArrayList是非线程安全的类

### LinkedList

LinedList底层是通过双链表实现数据存储相关操作。LinkedList很适合数据的动态插入和删除，随机访问和遍历速度则比较慢。

## HashTable vs HashMap vs TreeMap

### HashMap

HashMap是一个用于存储K-V键值对的集合，每一个键值对也叫作Entry。这些键值对分散存储在一个数组当中，这个数组就是HashMap的主干。

#### 常用的两个方法：PUT和GET

PUT操作：例如插入一个Key为“apple”的元素，首先使用哈希函数来确定Entry的插入位置index，但是由于HashMap长度是有限的，当插入的Entry越来越多时，就会产生index冲突。可以利用链表来解决此问题，数组中的每个元素不止是一个Entry对象，也是一个链表的节点。每一个Entry对象通过Next指针指向它的下一个Entry节点。当新来的Entry对象映射到冲突的数组位置时，使用**头插法**插入链表。**Java8开始使用尾插法，防止出现死循环问题。**

> 为什么使用头插法？HashMap的发明者认为，后插入的Entry被查找的可能性更大

GET：使用GET根据Key来查找Value时，首先把Key做一次Hash映射得到对应的index。由于可能会产生hash冲突，同一个位置可能匹配多个Entry，这时候需要顺着对应链表的头节点，一个一个向下找。

HashMap的初始长度为16，每次手动扩展或是手动初始化时，必须是2的幂次方。

为了实现高效的Hash算法，HashMap的发明者采用了**位运算**的方式。

> 位运算(&)效率要比代替取模运算(%)高很多，主要原因是位运算直接对内存数据进行操作，不需要转成十进制，因此处理速度非常快。HashMap为了提高效率使用位运算代替哈希，这又引入了哈希分布不均匀的问题，所以HashMap为解决这问题，又对hash算法做了一些改进，进行了扰动计算。
>
> ```java
> static final int spread(int h) {    return (h ^ (h >>> 16)) & HASH_BITS;}
> //HASH_BITS=0x7fffffff
> ```
>
> HashTable计算hash仍旧采用取模运算，`int index = (hash & 0x7FFFFFFF) % tab.length;`。也就是说，HashMap和HashTable对于计算数组下标这件事，采用了两种方法。HashMap采用的是位运算，而HashTable采用的是直接取模。

**index = HashCode（Key） & （Length - 1）**，可以说Hash算法最终得到的结果，完全取决于Key的HashCode的最后几位。为什么使用16或者2的幂，只要HashCode本身分布均匀，那么后几位每一位也是均匀的。 

#### ReHash过程

HashMap的容量有限，经过多次元素的插入，使得HashMap达到一定的饱和度，Key映射位置发生冲突的几率会逐渐提高。需要扩展它的长度。

**影响发生Resize的因素**：

- Capacity：当前长度，默认为16.
- LoadFactor：负载因子。默认值是0.75

条件：HashMap.Size>=Capacity*LoadFactor

**开始Resize**：

1. 扩容：创建一个新的Entry空数组，长度为原数组的2倍
2. Rehash：遍历原Entry数组，把所有的Entry重写Hash到新数组上。

> 在单线程下执行没有问题，但是hashMap并非线程安全。多线程情况下的扩容，可能会产生环形链表。

#### 1.8开始的HashMap结构

当Hash冲突严重时，在桶上形成的链表会变的越来越长，这样查询的效率会越来越低。1.8开始，通过一个判断TREEIFY_THRESHOLD来判断是否需要将链表转换为红黑树的阈值。HashEntry修改为Node。

**PUT**：

- 判断当前桶是否为空，空就进行初始化操作。resize中会判断是否进行初始化
- 根据当前的key的hashcode定位到具体的桶中并判断是否为空，为空表明没有hash冲突就直接在当前位置创建一个新Node节点即可。
- 如果当前桶有值，那么就要比较当前节点位置中的key、key的hashCode与写入的key是否相等，相等就覆盖当前节点的v值。通过onlyIfAbsent来决定是否需要覆盖
- 如果当前节点是红黑树，就按照红黑树的方式写入数据
- 如果是个链表，就需要当当前的key、value封装成一个新节点写入到当前桶的后面（形成链表）
- 判断当前链表的大小是否大于预设的阈值，大于时就要转换为红黑树
- 如果e!=null说明存在相同的key，那就需要覆盖，最后判断是否进行扩容操作。

~~~java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
~~~

**get方法**：

- 首先将keyhash之后取得所定位的桶，如果桶为空，则返回一个null
- 否则判断桶的第一个位置的key是否为查询的key，如果是直接返回value
- 如果第一个不匹配，则判断它的下一个是红黑树还是链表
- 红黑树就按照树的查找方式返回值
- 不然按照链表的方式遍历匹配返回值

~~~java
//强烈建议使用第一种 EntrySet 进行遍历。
//第一种可以把 key value 同时取出，第二种还得需要通过 key 取一次 value，效率较低。
Iterator<Map.Entry<String, Integer>> entryIterator = map.entrySet().iterator();
        while (entryIterator.hasNext()) {
            Map.Entry<String, Integer> next = entryIterator.next();
            System.out.println("key=" + next.getKey() + " value=" + next.getValue());
        }
        
Iterator<String> iterator = map.keySet().iterator();
        while (iterator.hasNext()){
            String key = iterator.next();
            System.out.println("key=" + key + " value=" + map.get(key));

        }

~~~

### ConcurrentHashMap

和hashMap非常相似，唯一的区别就是其中的核心数据如value，以及链表都是volatile修饰的，保证了获取时的可见性。

> 1.7：ConcurrentHashMap 采用了**分段锁技术**，其中 Segment 继承于 ReentrantLock。不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。
>
> 1.8：抛弃了Segment分段锁，通过CAS+Synchronized来保证并发安全性。将1.7中的HashEntry改为Node。其中val和next都用了volatile修饰，保证了可见性

**put**：

- 根据key计算出HashCode，判断是否需要初始化
- `f` 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
- 如果当前位置的 `hashcode == MOVED == -1`,则需要进行扩容。
- 如果都不满足，则利用 synchronized 锁写入数据。
- 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。

**get**：

- 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
- 如果是红黑树那就按照树的方式获取值。
- 就不满足那就按照链表的方式遍历获取值。

### 参考

[全网把Map中的hash()分析的最透彻的文章，别无二家](https://juejin.im/post/5ab99afff265da23a2291dee)

## HashTable vs HashMap vs ConcurrentHashMap

### 关于null

HashTable中，key和value不允许出现null值。如果进行put（null，null），编译可以通过，但是运行时会抛出空指针异常

HashMap中，null可以作为键值，这样的键只有一个；可以有一个或者多个键所对应的值为null。当get方法返回为null值时，可能是map中没有该键，也可能是该键对应的值为null。HashMap中如果key为空，会先进行让hash值为空。

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
ConcurrentMap也不能将null作为键值

### HashTable

初始容量为11，负载因子0.75。HashTable的所有操作都使用Synchronized锁住的，而Concurrent只是锁住了put中添加操作的代码。

HashTable默认的初始大小为11，之后每次扩充为原来的2n+1。

也就是说，HashTable的链表数组的默认大小是一个素数、奇数。之后的每次扩充结果也都是奇数。

由于HashTable会尽量使用素数、奇数作为容量的大小。当哈希表的大小为素数时，简单的取模哈希的结果会更加均匀。

## HashSet vs TreeSet

HashSet底层是HashMap，键为HashSet的添加元素，值为常量类`private static final Object PRESENT = new Object();`。添加操作add(Object e)就是对hashMap.put(e,PRESENT)的封装操作。remove操作同理。

LinkedHashSet继承HashSet，底层使用LinkedHashMap。使用链表来保证元素的插入顺序。

TreeSet底层数据结构为红黑树，TreeSet不仅保证元素的唯一性，也保证了元素的顺序。可以使用自然排序或者比较器排序。TreeSet不允许添加空值

可以使用Collections.synchronizedSet()保证线程安全。