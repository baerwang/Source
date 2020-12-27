# Java并发编程的艺术 - ConcurrentHashMap(JDK1.7版本)

## 为什么要使用ConcurrentHashMap

在并发编程中使用HashMap会导致程序死循环。而使用线程安全的HashTable效率又非常低下。

### 线程不安全的HashMap

​	在多线程环境下，使用HashMap进行put操作会引起死循环，导致CPU利用率接近100%，所以在并发的情况下不能使用HashMap。执行以下代码会造成死循环。

```java
public class ConcurrentHashMapTest {

    final static int CONCURRENT_LENGTH = 10000;

    final static HashMap<String, String> HASH_MAP = new HashMap<>();

    final static CountDownLatch COUNT_DOWN_LATCH = new CountDownLatch(5);

    public static void main(String[] args) throws Exception {
        for (int i = 0; i < COUNT_DOWN_LATCH.getCount(); i++) {
            MyThread myThread = new MyThread();
            new Thread(myThread, "" + i).start();
        }
        COUNT_DOWN_LATCH.await();
    }

    static class MyThread implements Runnable {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "开始执行");
            for (int i = 0; i < CONCURRENT_LENGTH; i++) {
                String uuid = UUID.randomUUID().toString();
                HASH_MAP.put(uuid, uuid);
            }
            COUNT_DOWN_LATCH.countDown();
            System.out.println(Thread.currentThread().getName() + "执行完成");
        }
    }
}
```

HashMap在并发执行put操作时会引起死循环，是因为多线程会导致HashMap的Entry链表形成环形数据结构，一旦形成环形数据结构，Entry的next节点永远不为空，就会产生死循环获取Entry。

### HashTable

​	**HashTable**容器使用**Synchronized**来保证线程安全，但在线程竞争激烈的情况下**HashTable**效率非常低。因为**HashTable**所有的方法都是**同步**的，都带着**Synchronized同步器**，当一个线程访问**HashTable**的同步方法，其他线程也访问**HashTable**的同步方法时，会进入阻塞或沦陷状态。如线程A使用put进行添加，线程B不能使用get和put操作方式，所以竞争越激烈效率越低。

### ConcurrentHashMap

​	**ConcurrentHashMap**的锁分段技术可有效提升并发访问率
​	**HashTable**容器在竞争激烈的并发环境下表现出效率低下的原因是所有访问**HashTable**的线程都必须**竞争同一把锁**，假如容器里又多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效提高并发访问效率，这就是**ConcurrentHashMap**所使用的锁分段技术。首先将数据分成一段一段地存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个数据的时候，其他段的数据也能被其他线程访问。

## ConcurrentHashMap的数据结构

ConcurrentHashMap是由Segment数据结构和HashEntry数组结构组成的。Segment是一种可重入锁(RenntrantLock)，在ConcurrentHashMap里扮演锁的角色；HashEntry则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组。Segment的结构和HashMap类似，是一种数组和链表的结构。一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。

## ConcurrentHashMap的初始化

ConcurrentHashMap初始化方法是通过initialCapacity(初始容量)、loadFactor(负载因子)、concurrencyLevel(并发级别)等几个参数来初始化Segment数组、段偏移量SegementShift、段掩码SegementMask和每个Segment里的HashEntry数组来实现的。

### 初始化Sgements数组

初始化Sgements数组源码

```java
if (concurrencyLevel > MAX_SEGMENTS) {
	concurrencyLevel = MAX_SEGMENTS;
     int sshift = 0;
     int ssize = 1;
     while (ssize < concurrencyLevel) {
     	++sshift;
        ssize <<= 1;
     }
     segmentShift = 32 - sshift;
     segmentMask = ssize - 1;
     this.segments = Segment.newArray(ssize);
}
```

Sgements数组的长度ssize是通过concurrencyLevel计算得出的。为了能通过按位与的散列算法来定位Segments数组的索引，必须保证Segments数组的长度是2的N次方(power-of-two size)，所以必须计算出一个大于或等于concurrencyLevel的最小的2的N次方值来作为Segments数组的长度。假如concurrencyLevel等于14、15或16ssize都会等于16，即容器里的锁的个数也是16。

concurrencyLevel的最大值是65535，这意味着Segments数组的长度最大为65536，对应的二进制是16位。

### 初始化SegementsShift和SegmentMask

两个全局变量需要在定位segment时的散列算法里使用，sshift等于ssize从1向左移位的次数，在默认情况下，concurrencyLevel等于16，1需要向左移位运动4次，所以sshift等于4。segmentShift用于定位参与散列算法的位数，segmentShift等于32减sshift，所以等于28，这里之所以用32是因为ConcurrentHashMap里的hash()方法输出的最大数是32位的，后面的测试中我肯可以看到这点。segmentMask是散列运算的掩码，等于ssize减1，即15，掩码的二进制各个位的值都是1。因为ssize的最大长度是65536，所以segmentShift最大值是16，segmentShift最大值是16，segmentMask最大值是65545，对应的二进制是16位，每个位都是1。

### 初始化每个segment

输入参数initialCapacity是ConcurrentHashMap的初始化容量，loadFactor是每个Segment的负载因子，在构造方法里需要通过这两个参数来初始化数组中的每个Segment。

```java
if (initialCapacity > MAXIMUM_CAPACITY) 
    initialCapacity = MAXIMUM_CAPACITY; 
	int c = initialCapacity / ssize; 
	if (c * ssize < initialCapacity) 
        ++c; 
	int cap = 1; 
	while (cap < c) 
        cap <<= 1; 
	for (int i = 0; i < this.segments.length; ++i) 
        this.segments[i] = new Segment<K,V>(cap, loadFactor);
```

上面代码中的变量cap就是segment里HashEntry数组的长度，它等于initialCapacity除以ssize的倍数c，如果c大于1，就会取大于等于c的2的N次方值，所以cap不是1，就是2的N次方。
Segment的容量threshold=(int)cao*loadFacotr，默认情况下initialCapacity等于16，loadFactor等于0.75，通过运算cap等于1，threshold等于0

### 定位Segment

既然ConcurrentHashMap使用分段锁Segment来保护不同段的数据，那么在插入和获取元素的时候，必须先通过散列算法定位呆segment。可以看到ConcurrentHashMap会首先使用Wang/Jekins hash的变种算法对元素的hashcode进行一次再散列。

```java
private static int hash(int h) {
    h += (h << 15) ^ 0xffffcd7d;
    h ^= (h >>> 10);
    h += (h << 3);
    h ^= (h >>> 6);
    h += (h << 2) + (h << 14);
    return h ^ (h >>> 16);
}
```

进行再散列，目的是减少散列冲突，使元素能够均匀地分布再不同的Segment上，从而提高容器的存取效率，假如散列的质量差到极点，那么所有的元素都在一个segment中，不仅存取元素缓慢，分段锁也会失去意义。
同过这种散列能让数字的每一位都参加到散列运算当中，从而减少散列的冲突。

```java
final Segment<K,V> segmentFor(int hash) {
    return segments[(hash >>> segmentShift) & segmentMask];
}
```

默认情况下segmentShift位28，segmentMask为15，再散列后的数最大是32位二进制数据，向右无符号移动28位，意思是让高4位参与到散列运算中，（hash >>> segmentShift) & segmentMask的运算结果分别是4、15、7和8，可以看到散列值没有发生冲突。

## happen-before 是什么？(HB)

happen-before 就是保证多线程环境中，上一个操作对下一个操作的有序性和操作结果的可见性。

happen-before 规则：
	程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
	监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
	volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
	传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。

两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前！happens-before仅仅要求前一个操作(执行后的结果)对后一个操作科见，且前一个操作按顺序排在第二个操作之前。

volatile 只能保证多线程操作的情况下有序性和可见性，防止了编译器中排序对程序结果的影响，但不保证原子性。

当一个变量被多个线程读取并且至少被一个线程写入时，如果读操作和写操作没有HB关系，则会产生数据竞争问题。要想保证操作B的线程看作操作A的结果(无论A和B是否在一个线程)，那么在A和B之前必须满足HB原则，如果没有，将有可能导致重排序。

## ConcurrentHashMap的操作

### get

Segment的get操作实现非常简单和高效。先经过一次再去散列，然后使用这个散列值通过散列运算定位到segment，再通过散列算法定位到元素。

```java
public v get (Object key) {
    int hash = hash(key.hashCode());
    return segmentFor(hash).get(key,hash);
}
```

get操作的高效之处在于整个get过程不需要加锁，除非读到的值是空才会加锁重读。我们知道HashTable容器的get方法是需要加速的，那么ConcurrentHashMap的get操作是如何做到不加锁呢？

原因是它的get里面将要使用的共享变量都定义为**volatile**类型，如用于统计当前Segment大小的count字段和用于存储值得HashEntry的value。定义成**volatile**的变量，能够在线程之间保证可见性，能够被多线程同时读，并且保证不会读到过期的值，但是只能被单线程写(有一种情况可以被多线程写，就是写入的值不依赖于原值)，在get操作里只需要读不需要写共享变量count和value，所以可以不用加锁。所以不会读到过期的值，是因为根据Java内存模型的**happen before**原则，对**volatile**字段的写入操作先于读操作，即使两个线程同时修改和获取**volatile**变量，get操作也能拿到最新的值，这是用**volatile**替换所的经典应用场景。

```java
transient volatile int count;
volatile V value;
```

在定位元素的代码，定位HashEntry和定位Segment的散列算法是一样，都与数组的长度减去1再相与，但是相与的值不一样，定位Segment使用的元素的hashcode通过再散列后得到的值的高位，而定位HashEntry直接使用的是再散列后的值。目的是避免两次散列后的值一样，虽然元素再Segment里散列开了，但是却没有再HashEntry里散列开。

```java
(hash >>> segmentShift) & segmentMask // 定位Segment所使用的hash算法
int index = hash & (tab.length - 1)   // 定位HashEntry锁使用的hash算法
```

### put

put方法需要对共享变量进行写入操作，所以为了线程安全，在操作共享变量时必须加锁。put方法首先定位到Segment，然后再Segment里进行插入操作。插入操作需要经历两个步骤，第一步判断是否需要对Segment里的HashEntry数组进行扩容，第二步定位添加元素的位置，然后放在HashEntry数组里。

1. 是否需要扩容

   再插入元素前会先判断Segment里的HashEntry数组是否超过容量(threshold)，如果超过阈值，则对数组进行扩容。Segment的扩容判断比HashMap更恰当，因为HashMap是在插入元素后判断元素是否到达容量，如果到达了就进行扩容，但是很有可能扩容之后没有新元素插入，这时HashMap就进行了一次无效的扩容。

2. 如何扩容

   在扩容的时候，首先创建一个容量是原来容量的两倍的数组，然后将原数组里的元素进行再散列后插入到新的数组里。为了高效，ConcurrentHashMap不会对整个容器进行扩容，而直对某个Segment进行扩容。

### size

如果要统计整个ConcurrentHashMap里元素的大小，就必须统计所有Segment里元素的大小后求和。Segment里的全局变量count是一个volatile变量，在多线程场景下，直接把所有Segment的count相加就可以得到整个ConcurrentHashMap的大小呢，并不是，虽然相加时可以获取每个Segment的count的最新值，但是可能累加前受用的count发生了变化，那么统计结果就不准了。所有最安全的做法是再统计size的时候把所有的Segment的put、remove和clean方法全部锁住，但是这种做法非常低效。

在累加count操作过程中，之前累加过的count发生变化的几率非常小，所以ConcurrentHashMap的做法是先尝试2次通过不锁住Segment的方式来统计各个Segment大小，如果统计的过程中，容器的count发生了变化，则再采用加锁的方式来统计所有Segment的大小。

#### 如何再ConcurrentHashMap判断再统计的时候容器发生了变化？

使用modCount变量，再put、remove和clean方法里操作元素前都会将变量modCount进行加1，那么再统计size前后比较modCount是否发生变化，从而得知容器的大小是否发生变化。