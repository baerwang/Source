# Java 阻塞队列

## 什么是阻塞队列

​	阻塞队列(BlockingQueue)是一个支持两个附加操作的队列。

1. 支持阻塞的插入方法：当队列满的时候，队列会阻塞插入元素的线程，知道对了不满。
2. 支持阻塞的移除方法：当队列为空时候，获取元素的线程会等待队列变为非空。

​	阻塞队列常用生产者和消费者的场景，生产者是向队列里添加元素的线程，消费者从队列里获取元素。阻塞队列就是生产者用来存放元素、消费者用来获取元素的容器。

​	在阻塞队列不可用时，附加操作提供了四种处理方式

| 方法/处理方式 | 抛出异常  | 返回特殊值 | 一直阻塞 |      超时退出      |
| :-----------: | :-------: | :--------: | :------: | :----------------: |
|   插入方法    |  add(e)   |  offer(e)  |  put(e)  | offer(e,time.unit) |
|   移除方法    | remove(0) |   poll()   |  take()  |  poll(time,unit)   |
|   检查方法    | element() |   peek()   |  不可用  |       不可用       |

- 抛出异常

  当队列满时，如果在往队列里插入元素，会抛出IllegalStateException("Queue full")异常。当队列空时，从队列里获取元素会抛出NoSuchElementException异常。

- 返回特殊值

  当往队列插入元素时，会返回元素是否插入成功，成功返回true。如果是移除方法，则是从队列里取出一个元素，如果没有则发挥null。

- 一直阻塞

  当阻塞队列满时，如果生产者线程往队列里put元素，队列会一直阻塞生产者线程，直到队列可用或者响应中断退出。当队列空时，如果消费者线程从队列里take元素，队列会阻塞住消费者线程，知道队列不为空。

- 超时退出

  当阻塞队列满时，如果生产者线程往队列插入元素，队列会阻塞生产者线程一段时间，如果超过了指定时间，生产者线程就会退出。

​	附加操作处理方式不方便记忆，put和take分别尾首含有字母t，offer和poll都含有字母o。

​	**如果是无界阻塞队列，队列不可能会出现满的情况，所以使用put或offer方法永远不会被阻塞，而且使用offer方法时，该方法永远返回true。**

## Java阻塞队列

1. [ArrayBlockingQueue](#ArrayBlockingQueue)：数组结构组成的有界阻塞队列。
2. [LinkedBlockingQueue](#LinkedBlockingQueue)：链表结构组成的有界阻塞队列。
3. [PriorityBlockingQueue](#PriorityBlockingQueue)：支持优先级排序的无界阻塞队列。
4. [DelayQueue](#DelayQueue)：使用优先级队列实现的无界阻塞队列。
5. [SynchronousQueue](#SynchronousQueue)：不存储元素的阻塞队列。
6. [LinkedTransferQueue](#LinkedTransferQueue)：链表结构组成的无界队列。
7. [LinkedBlockingDeque](#LinkedBlockingDeque)：链表结构组成的双向阻塞队列。

### <span id="ArrayBlockingQueue">ArrayBlockingQueue</span>

​	ArrayBlockingQueue是一个用数组实现的有界阻塞队列。此队列按照先进先出(FIFO)的原则对元素进行排序。

​	默认情况下不保证线程公平的访问队列，所谓公平访问队列是指阻塞的队列，可以按照阻塞的先后顺序访问队列，即先阻塞先访问队列。非公平性是对先等待的线程时非公平的，当队列可用时，阻塞的线程都会争夺访问队列的资格，有可能先阻塞的线程最后才访问队列。为了保证公平性，通常会降低吞吐量。

访问者的公平性是使用可重入锁实现的，默认是非公平锁。

```java
    public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }
```

### <span id="LinkedBlockingQueue">LinkedBlockingQueue</span>

​	LinkedBlockingQueue是一个链表实现的有界阻塞队列。此队列的默认和最大长度为Integer.MAX_VALUE。此队列按照先进先出的原则对元素进行排序。

### <span id="PriorityBlockingQueue">PriorityBlockingQueue</span>

​	PriorityBlockingQueue是一个支持优先级的无界阻塞队列。默认情况下元素采取自然顺序升序排列。也可以自定眼泪实现compareTo()方法来指定元素排序规则，或者初始化PriorityBlockingQueue时，指定构造参数Comparator来对元素进行排序。不能保证同优先级元素的顺序。

### <span id="DelayQueue">DelayQueue</span>

​	DelayQueue是一个延时获取元素的无界阻塞队列。队列使用PriorityQueue来实现。队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素，只有在延迟期满时才能从队列中提取元素。

​	**应用场景**：

- 缓存系统的设计

  可以用DelayQueue保存元素的有效期，使用一个线程循环查询DelayQueue，一旦能从DelayQueue中获取元素，表示缓存有效期到了。

- 定时任务调度

  使用DelayQueue保存当天将会执行的任务和执行时间，一旦从DelayQueue中获取到任务就开始执行，比如TimerQueue就是使用DelayQueue实现的。

### <span id="SynchronousQueue">SynchronousQueue</span>

​	SynchronousQueue是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。

​	它支持公平访问队列。默认情况下线程采用非公平性策略访问队列。使用以下构造方法可以创建公平性访问的SynchronousQueue，如果设置为true，则等待的线程会采用先进先出的顺序访问队列。

```java
    public SynchronousQueue(boolean fair) {
        transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();
    }
```



​	SynchronousQueue可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常合适传递性场景。SynchronousQueue的吞吐量高于LinkedBlockingQueue和ArrayBlockingQueue。

### <span id="LinkedTransferQueue">LinkedTransferQueue</span>

​	LinkedTransferQueue是一个由链表结构组成的无界阻塞TransferQueue队列。相对于其他的阻塞队列，LinkedTransferQueue多了tryTransfer和transfer方法。

1. transfer方法

   如果当前由消费者正在等待接收元素(消费者使用take()方法或带时间限制的poll()方法时)，transfer方法可用把生产者传入的元素立刻transfer(传输)给消费者。如果没有消费者在等待接收元素，transfer方法会将元素存放在队列的tail节点，并等到该元素被消费者消费了才返回。

   ```java
   Node pred = tryAppend(s, haveData);
   return awaitMatch(s, pred, e, (how == TIMED), nanos);
   ```

   第一行代码是试图把存放当前元素的s节点作为tail节点。第二行代码是让CPU自旋等待消费者消费元素。因为自旋会消耗CPU，所以自旋一定的次数后使用Thread.yield()方法来暂停当前正在执行的线程，并执行其他线程。

2. tryTransfer方法

   tryTransfer方法是用来试探生产者传入的元素是否能直接传给消费者。如果没有消费者等待接收元素，则返回false。和transfer方法的区别是tryTransfer无论消费者是否接受，方法立即返回，而transfer方法是必须等到消费者消费了才返回。

   带有时间限制的tryTransfer(E e,long timeout,TimeUnit unit)方法，试图把生产者传入的元素直接传给消费者，但是如果没有消费者消费该元素则等待指定的时间在返回，如果超时还没消费元素，则返回false，如果在超时时间内消费了元素，则返回true。

### <span id="LinkedBlockingDeque">LinkedBlockingDeque</span>

​	LinkedBlockingDeque是一个由链表结构组成的双向阻塞队列。双向队列指的是可用从队列的两端插入和移出元素。双向队列因为多了一个操作队列的入口，在多线程同时入队时，也就减少了一半的竞争。相比其他的阻塞队列，LinkedBlockingDeque多了addFirst、addLast、offerFirst、offerLast、peekFirst和peekLast等方法，以First单词结尾的方法。表示插入，获取(peek)或移除双端队列的第一个元素。以Last单词结尾的方法，表示插入、获取或移除双端队列的最后一个元素。另外，插入方法add等同于addLast，移除方法remove等效于removeFirst。

​	在初始化LinkedBlockingDeque时可以设置容量防止过度膨胀。双向阻塞队列可以运行在**工作窃取**模式中。

#### 什么是工作窃取模式

​	工作窃取(work-stealing)模式是指某个线程从其他队列里窃取任务来执行。假如我们需要做一个比较大的任务，可以把这个任务分割为若干互不依赖的子任务，为了减少线程间的竞争，把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来执行队列里的任务，线程和队列--对应。比如A线程负责处理A队列里的任务。但是有的线程会先把自己队列的任务干完，而其他线程对应的队列里还有任务等待处理。干完活的线程等着，不如帮其他线程干活，于是它就去其他线程的队列里窃取一个任务来执行。而这时它们会访问同一个队列，所以为了减少窃取任务线程和被窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。

**优点**：充分利用线程进行并行计算，减少了线程间的竞争。

**缺点**：在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且该算法会小号了更多的系统资源，比如创建多个线程和多个双端队列。