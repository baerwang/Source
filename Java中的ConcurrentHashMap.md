# Java中的ConcurrentHashMap

> ConcurrentHashMap的并发度是什么

​		ConcurrentHashMap把实际map划分称若干部分来实现它的可扩展性和线程安全。这种划分是使用并发获得的，它是ConcurrentHashMap类构造函数的一个可选参数，默认值为16，这样在多线程的情况下就能避免争用。

​		在JDK8后，它抛弃了Segment(锁段)的概念，而是启用了一种全新的方法实现，利用CAS算法。同时加入了更多的辅助变量来提高并发度。

concurrenthashmap就是默认把整个hash桶分成16个段，然后对每个段分别上线程锁。

