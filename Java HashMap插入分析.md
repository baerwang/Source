# Java HashMap插入分析

> 插入逻辑分析

首先定义要插入的键值对属于那个桶，定位到桶后，在判断是否为空。如果为空，将键值对存入。如果不为空，将键值对接在链表最后一个位置或者更新键值对。

首先HashMap是变长的集合，所以需要考虑扩容的问题。在JDK8中，HashMap引入了红黑树优化过长链表，还需要考虑多长的链表需要进行优化。

- 插入操作源码：

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 初始化桶数组 table,table被延迟到插入新数据时再进行初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 如果桶中不包含键值对节点引用,则将键值对节点的引用存入桶中即可
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            // 如果键的值以及节点 hash 等于链表中的第一个键值对节点时,将 e 指向该键值对
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 如果桶中的引用类型为 TreeNode,调用红黑树的插入方法
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 对链表进行遍历,并统计链表长度
                for (int binCount = 0; ; ++binCount) {
                    // 如果链表长度大于或等于树化阈值,进行树化操作
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 条件为true,表示当前链表包含要插入的键值对,终止遍历
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // 判断要插入的键值对是否存在HashMap中
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                // onlyIfAbsent 表示是否仅在 oldValue 为 null 的情况下更新键值对的值
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 键值对数量超过阈值,则进行扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

插入操作的入口方法是 **put(K,V)**，但核心逻辑在 **final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict)** 方法中。**putVal** 方法主要做了几件事：

1. 当桶数组 table 为空，通过扩容的方式初始化table
2. 查找要插入的键值对是否存在，存在的话根据条件判断是否用新值替换旧值
3. 如果不存在，则将键值对链入链表中，并根据链表长度决定是否将链表转为红黑树
4. 判断键值对数量是否大于阈值，大于的话就进行扩容操作

> 扩容机制

HashMap 中，桶数组长度均是2的幂，阈值大小为桶数组长度于负载因子的乘积。当 HashMap 中的键值对数量超过阈值时，进行扩容。

HashMap 按当前桶数组长度的2倍进行扩容，阈值也变为原来的2倍。扩容之后要重新计算键值对的位置，并把它们移动到合适的位置上去。

- 扩容过程源码：

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
    	// 如果table为空,表明已经初始化过了
        if (oldCap > 0) {
            // 当table容量超过容量最大值,则不再扩容
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 按旧容量和阈值的2倍计算新容量和阈值的大小
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            // 初始化时,将threshold的值赋给newCap
            // HashMap使用threshold变量暂时保存initialCapacity参数的值
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            // 调用无参构造方法时,桶数组组容量为默认容量
            // 阈值为默认容量与默认负载因子乘积
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
    	// newThr为0,按阈值计算公式进行计算
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
    		// 创建新的桶数组,桶数组的初始化也是在这里完成的
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            // 如果旧的桶数组不为空,则变量桶数组,并将键值对映射到新的桶数组中
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    // 判断是否为红黑树
                    else if (e instanceof TreeNode)
                        // 重新映射时,需要对红黑树进行拆分
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        // 遍历链表,并将链表节点按原顺序进行分组
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 将分组后的链表映射到新桶中
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

resize()方法做了三件事情：

1. 计算新桶数组的容量 **newCap** 和新阈值 **newThr**
2. 根据计算出的 **newCap** 创建新的桶数组，桶数组table也是在这里进行初始化的
3. 将键值对节点重新映射到新桶数组里，如果节点时TreeNode类型，则需要拆分红黑树。如果是普通节点，则节点按顺序进行分组。

