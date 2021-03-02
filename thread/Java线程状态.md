# Java线程状态

![](..\images\image-20210302132918451.png)

常见的线程状态有六种：

new一个线程时，还没有调用start()该线程处于**新建状态**

线程对象调用start()方法时候，他会被线程调度器来执行，也就是交给操作系统来执行了，那么操作系统来运行的时候，这整个状态叫Runnable，Runnable内部有两个状态**(1)Ready就绪状态/(2)Running运行状态**。就绪状态是说扔到CPU的等待队列里面去排队等待CPU运行，等真正扔到CPU上去运行的时候才叫Runnable运行状态。(调用yiled时候会从Running状态跑到Ready状态去，线程配调度器选中执行的时候又从Ready状态跑到Running状态去)

如果线程顺利执行完了就会进**(3)Teminated结束状态**。(需要注意Teminated完了之后还可不可以回到new状态再调用start？这是不行的，这就是结束了)

在Runnable这个状态里头还有其他一些状态的变迁**(4)TimedWaiting等待**、**(5)Waiting等待**、**(6)Blocked阻塞**，在同步代码块的情况下就没得到锁就会**阻塞状态**，获得锁的时候是就绪状态运行。在运行的时候如果调用了o.wait()、t.join()、LockSupport.park()进入**Waiting状态**，调用o.notify()、o.notifyAll()、LockSupport.unpark()就又回到Running状态。**TimedWaiting**按照时间等待，等待时间结束自己就回去了，Thread.sleep(time)、o.wait(time)、t.join(time)、LockSupport.parkNanos()、LockSupport.parkUntill()这些都是关于时间等待的方法。