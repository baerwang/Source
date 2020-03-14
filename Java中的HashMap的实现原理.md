# Java中的HashMap的实现原理

> hashCode是什么

1. 哈希表
   - hash表也称散列表(Hash table)，是根据关键码值(Key value)而注解进行访问的数据结构。也就是说，它通过把关键码值映射到表中的一个位置来访问记录，以加快查找的速度。这个映射函数也叫做散列函数，存放记录的数组叫做散列表。
   - 给定表M，存在函数f(key)，对任意给定的关键字值key，代入函数后若能得到包含该关键字的记录在表中的地址，则称表M为哈希(Hash)表，函数f(key)为哈希(Hash)函数。
   - 简单的理解就是：在记录的存储位置和它的关键字之间建立一个确定的对应关系f，使每个关键字和结构中一个唯一的存储位置相对应。
   - 具有快速查找和插入操作的优点。
2. hashCode

	- hashCode通过hash函数计算得到，hashCode就是在hash表中有对应的位置
	- 每个对象都有hashCode，通过对象的物理地址转换为一个整数，将整数通过hash计算就可以得到hashCode

> hachCode 有什么作用

- hashCode的存在主要是用于查找快捷性，如HashTable，HashMap等。hashCode是用来散列存储结构中确定对象的存储地址。
- 如果两个对象相同，就适用于equals(java.lang.Object)方法，那么这个两个对象的hashCode一定要相同。
- 如果对象的equals方法被重写，那么对象的hashCode也要尽量重写，并且产生hashCode使用的对象，一定要和equals方法中使用的一致，否则就会违法上面提到的第2点。
- 两个对象的hashCode相同，并不一定表示两个对象就相同，也就是不一定适用于equals(java.lang.Object)方法，只能够说明这两个对象在散列存储结果中。

**如何判断集合中是否已经存在该对象了**

​	首先想到的方法就是调用equals()方法。但是如果集合中已经存在大量的数据或者更多的数据，采用equals方法去逐一比较，效率必然是一个问题。此时hashCode方法的作用就体现出来了，当集合要添加新得对象时，先调用这个对象得hashCode方法，得到对应得hashCode值，实际上在HashMap得具体实现中会一个表保存已经存进去得对象得hashCode值，如果table中没有该hashCode的值，它就可以存进去，不用再进行任何比较了。如果存在该hashCode值，就调用它的equals方法与新元素进行比较，相同的话就不存了，不相同就散列其他的地址，所以这里存在一个冲突的解决问题，这样依赖实际调用equals方法的次数就大大降低了。

​	这也就解释了为什么equals()相等，则hashCode()必须相等，如果两个对象equals()相等，则它们在哈希表(如HashSet，HashMap等)中应该出现一次。如果hashCode()不相等，那么他们会被散列到哈希表的不同位置，哈希表中出现了不止一次。

​	**hashCode方法的存在是为了减少equals方法的调用次数，从而提高程序效率。**

> hashCode() 和 equals()

​	**Java的基类Object中的equals()方法用于判断两个对象是否相等，hashCode()方法用于计算对象的哈希码。equals()和hashCode都不是final方法，都可以被重写。**

> hashCode()方法

 - Object类中的hashCode()方法的声明如下：

   ```java
   public native int hashCode();
   ```

-	可以看出，hashCode()是一个native方法，而且返回值类型是整形。实际上，该native方法将对象在内存中地址最为哈希码返回，可以保证不同对象的返回值不同。

-	与equals()方法类型，hashCode()方法可以被重写。JDK中对hashCode()方法的作用，以及实现时的注意的说明

  - [ ] hashCode()在哈希表中起作用，如java.util.HashMap。
  - [ ] 如果对象在equals()中使用的信息都没有改变，那么hashCode()值始终不变。
  - [ ] 如果两个对象使用equals()方法判断为相等，则hashCode()方法也应该相等。
  - [ ] 如果两个对象使用equals()方法判断为不相等，则不要求hashCode()也必须不相等。但是开发人员应该认识到，不相等的对象产生不相同的hashCode可以提高哈希表的性能。

-	重写hashCode()的原则

  - [ ] 如果重写了equals()方法，检查条件"**两个对象使用equals()方法判断为相等，则hashCode()方法也应该相等**"是否相等，如果不成立，则重写hashCode()方法。
  - [ ] hashCode()方法不能太过于简单，否则哈希冲突过多。
  - [ ] hashCode()方法不能太过复杂，否则计算复杂度过高，影响性能。

> equals方法

1. equals()和==

   ==用于比较引用和比较基本数据类型时具有不同的功能：

   ​	比较基本数据类型，如果两个值相同，结果为true。

   ​	在比较引用时，如果引用指向内存中的同一对象，结果为true。

   equals()作为方法，实现对象的比较。由于运算符不允许我们进行覆盖，也就是说它限制了我们的表达。因此我们重写equals()方法，达到比较对象的内容是否相同的目的。而通过==运算符是做不到的。

2. object类的equals()方法的比较规则为：如果两个对象的类型一致，并且内容一致，则返回true。

   ```java
   String str1 = new String("abc");
   String str2 = new String("abc");
   System.out.print(str == str2);		// false
   System.out.print(str.equals(str2)); // true
   ```


> HashMap中的hash()函数

- HashMap中没有使用KV中原有的hash值。在HashMap的put，get操作时也未使用K中原有的hash值，而使用了hash()方法。

  ```java
  static final int hash(Object key){
      int h;
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
  ```

- 这段代码类似作用是为了增加hashCode的随机性。

- key.hashCode()的作用返回键值key所属性类型自带的hashCode，返回的类型是int，如果直接拿散列值作为下表访问HashMap的主数组的话，考虑到int类型值得方位[ -2^31，2^31-1 ]，虽然只要hash表映射比较松散得话，碰撞几率很小，但是映射空间太大，内存放不下，所以先做对数组得长度取模运算，得到得余数才能用来访问数组下标。

> HashMap的resize()

​	当hashmap中的元素越来越多的时候，碰撞的几率也就越来越高(因为数组的长度是固定的)，所以为了提高查询的效率，就要对hashmap的数组进行扩容，数组扩容这个操作也会出现在ArrayList中，所以这是一个通用的操作，很多人对它的性能表示过怀疑，想想我们的"均摊"原理，而在hashmap数组扩容之后，最消耗性能的点就出现了：原数组中的数组必须重新计算其在新数组中的位置，并放进去，这就是resize。

数组初始化以及数组元素个数大于阈值时进行扩容操作，一部分索引会增加原长度大小的长度(用到了高位1)，一部分仍保证原索引(高位为0)。

```java
final Node<K,V>[] resize() {
    	// 将旧数组进行保存
        Node<K,V>[] oldTab = table;
    	// 保存旧数组的长度
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
    	// 保存旧数组的阈值
        int oldThr = threshold;
    	// 定义新的长度和阈值
        int newCap, newThr = 0;
        if (oldCap > 0) {
            // 数组以及达到最大容量，直接返回
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 新数组长度为旧数组长度*2
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                // 阈值同样*2
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults，默认的初始化操作
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
    	// 将新的阈值付给成员变量
        threshold = newThr;
    	// 创建一个新的数组，大小为newCap
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
    	// 将旧数组元素放入到新数组中
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 如果当前索引只有一个节点
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
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

> HashMap的put()

- 先判断数组是否初始化，如果没有初始化，则进一次初始化操作(扩容)，同时将数组大小赋给n

- 找到具体的桶，并判断此位置是否有元素，如果没有元素，则创建一个Node直接插入

- 如果出现冲突

  1. 如果为红黑树节点，调用红黑树方法插入数据
  2. 如果普通节点，插入链表未尾，并且长度达到临界值时，将链表转为红黑树

- 如果桶中存在重复的键，将该键替换新值value

- size大于阈值threshold，进行扩容

  ```java
  public V put(K key,V value){
      return putVal(hash(key), key, value, false, true);
  }
  final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                     boolean evict) {
  		// 初始化一个tab以及node
          Node<K,V>[] tab; Node<K,V> p; int n, i;
      	// 此处菜进行tab的初始化。tab为空或者数组大小为0，对数组进行初始化操作，并将数组大小赋值给n
          if ((tab = table) == null || (n = tab.length) == 0)
              n = (tab = resize()).length;
      	// 通过hash与数组大小-1的与运算计算出所在桶位置的元素p,如果node为null，创建一个新节点直接插入，如果出现冲突，进入分支判断
          if ((p = tab[i = (n - 1) & hash]) == null)
              tab[i] = newNode(hash, key, value, null);
          else {
              Node<K,V> e; K k;
              // 如果插入的元素的hash值与p相等以及p的key与要插入的key相同，将p(原位置节点)赋给e
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  e = p;
              // p为红黑树节点，则调用putTreeVal插入数据，如果为覆盖，则e为旧节点
              else if (p instanceof TreeNode)
                  e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
              else {
                  // 链表节点
                  for (int binCount = 0; ; ++binCount) {
                      // 找到链表的尾节点，此时e==null，p为链表的最后一个节点
                      if ((e = p.next) == null) {
                          // 在未尾创建一个节点赋给p.next，此时e仍为null
                          p.next = newNode(hash, key, value, null);
                         // 如果找到当前节点已经循环了7此，即该链表在插入元素大小为8，将链表转为红黑树   
                          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                              // 链表转红黑树，传入tab数组以及该键的hash值(可计算出数组的具体索引)
                              treeifyBin(tab, hash);
                          break;
                      }
                      // 如果找到了具有相同的key的元素，也停止录找
                      if (e.hash == hash &&
                          ((k = e.key) == key || (key != null && key.equals(k))))
                          break;
                      p = e;
                  }
              }
  		   // 若此时e不为null，说明找到了一个具有相同的key的值
              if (e != null) { // existing mapping for key
                  // 保存一下旧节点的value值
                  V oldValue = e.value;
                  // 是否要改变之前存在的值(默认为false)或者之前存在的值为null，将value进行一个覆盖
                  if (!onlyIfAbsent || oldValue == null)
                      e.value = value;
                  // 回调相关方法，HashMap该方法默认实现为空，LinkedHashMap在此会进行一些处理
                  afterNodeAccess(e);
                  return oldValue;
              }
          }
  	    // 记录下map的修改次数
          ++modCount;
          // 如果元素个数大于阈值，进行扩容操作
          if (++size > threshold)
              resize();
          afterNodeInsertion(evict);
          return null;
      }
  ```

> 链表转红黑树(treeifyBin)

  ```java
  final void treeifyBin(Node<K,V>[] tab, int hash) {
          int n, index; Node<K,V> e;
      	// 如果tab数组为空或者tab数组大小小于链表转红黑树的最小要求值，则进行扩容操作
          if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
              resize();
      	// 拿到当前要转换的桶的起始节点
          else if ((e = tab[index = (n - 1) & hash]) != null) {
              // 初始化头结点和尾节点
              TreeNode<K,V> hd = null, tl = null;
              // 循环将链表结点转化为红黑树节点
              do {
                  // 利用链表节点来创建一个树的节点
                  TreeNode<K,V> p = replacementTreeNode(e, null);
                  // 如果t1为null，表示红黑树还没有节点，将p赋给头结点
                  if (tl == null)
                      hd = p;
                  // 将p节点与尾节点相连
                  else {
                      p.prev = tl;
                      tl.next = p;
                  }
                  // 更新尾节点
                  tl = p;
              } while ((e = e.next) != null);
              if ((tab[index] = hd) != null)
                  // 将各个树节结点转化为红黑树
                  hd.treeify(tab);
          }
      }
  ```

> 删除方法(remove)

  ```java
  public V remove(Object key){
      Node<K,V> e;
      return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
  }
  final Node<K,V> removeNode(int hash, Object key, Object value,
                                 boolean matchValue, boolean movable) {
          Node<K,V>[] tab; Node<K,V> p; int n, index;
          if ((tab = table) != null && (n = tab.length) > 0 &&
              (p = tab[index = (n - 1) & hash]) != null) {
              Node<K,V> node = null, e; K k; V v;
              // 初始节点为要找的节点，赋值给node
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  node = p;
              // 向下找节点
              else if ((e = p.next) != null) {
                  if (p instanceof TreeNode)
                      node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                  else {
                      do {
                          if (e.hash == hash &&
                              ((k = e.key) == key ||
                               (key != null && key.equals(k)))) {
                              node = e;
                              break;
                          }
                          p = e;
                      } while ((e = e.next) != null);
                  }
              }
              // 删除操作
              if (node != null && (!matchValue || (v = node.value) == value ||
                                   (value != null && value.equals(v)))) {
                  if (node instanceof TreeNode)
                      ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                  else if (node == p)
                      tab[index] = node.next;
                  else
                      p.next = node.next;
                  ++modCount;
                  --size;
                  afterNodeRemoval(node);
                  return node;
              }
          }
          return null;
      }
  ```

> 查找方法(get)

  ```java
  public V get(Object key) {
      Node<K,V> e;
      return (e = getNode(hash(key), key)) == null ? null : e.value;
  }
  final Node<K,V> getNode(int hash, Object key) {
          Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
          if ((tab = table) != null && (n = tab.length) > 0 &&
              (first = tab[(n - 1) & hash]) != null) {
              if (first.hash == hash && // always check first node
                  ((k = first.key) == key || (key != null && key.equals(k))))
                  return first;
              if ((e = first.next) != null) {
                  if (first instanceof TreeNode)
                      return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                  do {
                      if (e.hash == hash &&
                          ((k = e.key) == key || (key != null && key.equals(k))))
                          return e;
                  } while ((e = e.next) != null);
              }
          }
          return null;
      }
  ```

> HashMap的初始化设计

为了尽可能的避免hashmap的扩容操作，提高性能，如果明确知道存储的数据量大小i时，初始化值如下

```java
Map<String,String> map = new HashMap<>(initialCapacity);

initialCapactiry = (需要存储的元素的个数 / 负载因子) + 1; 
```



**总结：HashMap的实现原理**

1. **利用key的hashCode重新计算hash计算出当前对象的元素在数组中的下标。**
2. **存储时，如果出现hash值相同的key，此时有两种情况，如果key相同，则覆盖原始值；如果key不同(出现冲突)，则将当前的K-V放入链表中。**
3. **获取时，直接找到hash值对应的下表=标，在进一步判断key是否相同，从而找到对应的值。**
4. **理解了以上过程就不难明白HashMap是如何解决hash冲突的问题，核心就是使用了数组的存储方式，然后将冲突的key的对象放入链表重，一旦发现冲突就在链表中做进一步的对比。**
5. **JDK1.7中采用头插入法，在扩容时会改变链表中的元素原本的顺序，以至于在并发场景下导致链表称环的问题。在JDK1.8中查用尾插入法，在扩容时会保证链表元素原本的顺序，就不会出现链表称环的问题了。**
6. **1.7采用数组+单链表，1.8在单链表超过一定长度后改成红黑树的存储。**
7. **1.7扩容时需要重新计算哈希值和索引位置，1.8并不重新计算哈希值，巧妙地采用和扩容后容量进行&操作来计算新的索引位置。**
8. **1.7插入元素到单链表重采用头插入法，1.8采用的是尾插入法。**



解读摘自其他博客：

https://www.cnblogs.com/yuanblog/p/4441017.html

https://github.com/g908682550/Java-StudyNotes/blob/master/src/HashMap.md

