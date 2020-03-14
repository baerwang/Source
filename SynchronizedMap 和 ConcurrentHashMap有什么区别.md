# SynchronizedMap 和 ConcurrentHashMap有什么区别

​	**SynchronizedMap **一次锁住整张表来保证线程安全，所以每次只能又一个线程来访问map。

​	**ConcurrentHashMap** 使用分段锁来保证多线程下的性能。

​	**ConcurrentHashMap** 中