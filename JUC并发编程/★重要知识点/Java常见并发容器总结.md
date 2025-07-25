# Java常见并发容器总结

# Java 常见并发容器总结
JDK 提供的这些容器大部分在java.util.concurrent包中。

+ **ConcurrentHashMap**: 线程安全的HashMap
+ **CopyOnWriteArrayList**: 线程安全的List，在读多写少的场合性能非常好，远远好于Vector。
+ **ConcurrentLinkedQueue**: 高效的并发队列，使用链表实现。可以看做一个线程安全的LinkedList，这是一个非阻塞队列。
+ **BlockingQueue**: 这是一个接口，JDK 内部通过链表、数组等方式实现了这个接口。表示阻塞队列，非常适合用于作为数据共享的通道。
+ **ConcurrentSkipListMap**: 跳表的实现。这是一个 Map，使用跳表的数据结构进行快速查找。

## ConcurrentHashMap
我们知道HashMap不是线程安全的，在并发场景下如果要保证一种可行的方式是使用Collections.synchronizedMap()方法来包装我们的HashMap。但这是通过使用一个全局的锁来同步不同线程间的并发访问，因此会带来不可忽视的性能问题。

所以就有了HashMap的线程安全版本——ConcurrentHashMap的诞生。

在 JDK1.7 的时候，ConcurrentHashMap对整个桶数组进行了分割分段(Segment，分段锁)，每一把锁只锁容器其中一部分数据（下面有示意图），多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。

到了 JDK1.8 的时候，ConcurrentHashMap已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用synchronized和 CAS 来操作。（JDK1.6 以后synchronized锁做了很多优化） 整个看起来就像是优化过且线程安全的HashMap，虽然在 JDK1.8 中还能看到Segment的数据结构，但是已经简化了属性，只是为了兼容旧版本。

关于ConcurrentHashMap的详细介绍，请看我写的这篇文章：[ConcurrentHashMap](https://javaguide.cn/java/collection/concurrent-hash-map-source-code.html)[源码分析](https://javaguide.cn/java/collection/concurrent-hash-map-source-code.html)。

## CopyOnWriteArrayList
在 JDK1.5 之前，如果想要使用并发安全的List只能选择Vector。而Vector是一种老旧的集合，已经被淘汰。Vector对于增删改查等方法基本都加了synchronized，这种方式虽然能够保证同步，但这相当于对整个Vector加上了一把大锁，使得每个方法执行的时候都要去获得锁，导致性能非常低下。

JDK1.5 引入了Java.util.concurrent（JUC）包，其中提供了很多线程安全且并发性能良好的容器，其中唯一的线程安全List实现就是CopyOnWriteArrayList。

对于大部分业务场景来说，读取操作往往是远大于写入操作的。由于读取操作不会对原有数据进行修改，因此，对于每次读取都进行加锁其实是一种资源浪费。相比之下，我们应该允许多个线程同时访问List的内部数据，毕竟对于读取操作来说是安全的。

这种思路与ReentrantReadWriteLock读写锁的设计思想非常类似，即读读不互斥、读写互斥、写写互斥（只有读读不互斥）。CopyOnWriteArrayList更进一步地实现了这一思想。为了将读操作性能发挥到极致，CopyOnWriteArrayList中的读取操作是完全无需加锁的。更加厉害的是，写入操作也不会阻塞读取操作，只有写写才会互斥。这样一来，读操作的性能就可以大幅度提升。

CopyOnWriteArrayList线程安全的核心在于其采用了**写时复制（Copy-On-Write）**的策略，从CopyOnWriteArrayList的名字就能看出了。

当需要修改（add，set、remove等操作）CopyOnWriteArrayList的内容时，不会直接修改原数组，而是会先创建底层数组的副本，对副本数组进行修改，修改完之后再将修改后的数组赋值回去，这样就可以保证写操作不会影响读操作了。

关于CopyOnWriteArrayList的详细介绍，请看我写的这篇文章：[CopyOnWriteArrayList](https://javaguide.cn/java/collection/copyonwritearraylist-source-code.html)[源码分析](https://javaguide.cn/java/collection/copyonwritearraylist-source-code.html)。

## ConcurrentLinkedQueue
Java 提供的线程安全的Queue可以分为**阻塞队列**和**非阻塞队列**，其中阻塞队列的典型例子是BlockingQueue，非阻塞队列的典型例子是ConcurrentLinkedQueue，在实际应用中要根据实际需要选用阻塞队列或者非阻塞队列。**阻塞队列可以通过加锁来实现，非阻塞队列可以通过 CAS 操作实现。**

从名字可以看出，ConcurrentLinkedQueue这个队列使用链表作为其数据结构．ConcurrentLinkedQueue应该算是在高并发环境中性能最好的队列了。它之所有能有很好的性能，是因为其内部复杂的实现。

ConcurrentLinkedQueue内部代码我们就不分析了，大家知道ConcurrentLinkedQueue主要使用 CAS 非阻塞算法来实现线程安全就好了。

ConcurrentLinkedQueue适合在对性能要求相对较高，同时对队列的读写存在多个线程同时进行的场景，即如果对队列加锁的成本较高则适合使用无锁的ConcurrentLinkedQueue来替代。

## BlockingQueue
### BlockingQueue 简介
上面我们己经提到了ConcurrentLinkedQueue作为高性能的非阻塞队列。下面我们要讲到的是阻塞队列——BlockingQueue。阻塞队列（BlockingQueue）被广泛使用在“生产者-消费者”问题中，其原因是BlockingQueue提供了可阻塞的插入和移除的方法。当队列容器已满，生产者线程会被阻塞，直到队列未满；当队列容器为空时，消费者线程会被阻塞，直至队列非空时为止。

BlockingQueue是一个接口，继承自Queue，所以其实现类也可以作为Queue的实现来使用，而Queue又继承自Collection接口。下面是BlockingQueue的相关实现类：

![1732497489886-8c4ebd1c-4fc2-4a91-89c7-dd10adaa4bad.png](./img/OEVjXpRm2h_vzPro/1732497489886-8c4ebd1c-4fc2-4a91-89c7-dd10adaa4bad-717378.png)

BlockingQueue 的实现类

下面主要介绍一下 3 个常见的BlockingQueue的实现类：ArrayBlockingQueue、LinkedBlockingQueue、PriorityBlockingQueue。

### ArrayBlockingQueue
ArrayBlockingQueue是BlockingQueue接口的有界队列实现类，底层采用数组来实现。

```java
public class ArrayBlockingQueue<E>
extends AbstractQueue<E>
implements BlockingQueue<E>, Serializable{}
```

ArrayBlockingQueue一旦创建，容量不能改变。其并发控制采用可重入锁ReentrantLock，不管是插入操作还是读取操作，都需要获取到锁才能进行操作。当队列容量满时，尝试将元素放入队列将导致操作阻塞;尝试从一个空队列中取一个元素也会同样阻塞。

ArrayBlockingQueue默认情况下不能保证线程访问队列的公平性，所谓公平性是指严格按照线程等待的绝对时间顺序，即最先等待的线程能够最先访问到ArrayBlockingQueue。而非公平性则是指访问ArrayBlockingQueue的顺序不是遵守严格的时间顺序，有可能存在，当ArrayBlockingQueue可以被访问时，长时间阻塞的线程依然无法访问到ArrayBlockingQueue。如果保证公平性，通常会降低吞吐量。如果需要获得公平性的ArrayBlockingQueue，可采用如下代码：

```java
private static ArrayBlockingQueue<Integer> blockingQueue = new ArrayBlockingQueue<Integer>(10,true);
```

### LinkedBlockingQueue
LinkedBlockingQueue底层基于**单向链表**实现的阻塞队列，可以当做无界队列也可以当做有界队列来使用，同样满足 FIFO 的特性，与ArrayBlockingQueue相比起来具有更高的吞吐量，为了防止LinkedBlockingQueue容量迅速增，损耗大量内存。通常在创建LinkedBlockingQueue对象时，会指定其大小，如果未指定，容量等于Integer.MAX_VALUE。

**相关构造方法:**

```java
/**
     *某种意义上的无界队列
     * Creates a {@code LinkedBlockingQueue} with a capacity of
     * {@link Integer#MAX_VALUE}.
     */
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}

/**
     *有界队列
     * Creates a {@code LinkedBlockingQueue} with the given (fixed) capacity.
     *
     * @param capacity the capacity of this queue
     * @throws IllegalArgumentException if {@code capacity} is not greater
     *         than zero
     */
public LinkedBlockingQueue(int capacity) {
if (capacity <= 0) throw new IllegalArgumentException();
this.capacity = capacity;
last = head = new Node<E>(null);
}
```

### PriorityBlockingQueue
PriorityBlockingQueue是一个支持优先级的无界阻塞队列。默认情况下元素采用自然顺序进行排序，也可以通过自定义类实现compareTo()方法来指定元素排序规则，或者初始化时通过构造器参数Comparator来指定排序规则。

PriorityBlockingQueue并发控制采用的是可重入锁ReentrantLock，队列为无界队列（ArrayBlockingQueue是有界队列，LinkedBlockingQueue也可以通过在构造函数中传入capacity指定队列最大的容量，但是PriorityBlockingQueue只能指定初始的队列大小，后面插入元素的时候，**如果空间不够的话会自动扩容**）。

简单地说，它就是PriorityQueue的线程安全版本。不可以插入 null 值，同时，插入队列的对象必须是可比较大小的（comparable），否则报ClassCastException异常。它的插入操作 put 方法不会 block，因为它是无界队列（take 方法在队列为空的时候会阻塞）。

**推荐文章：**[《解读 Java 并发队列 BlockingQueue》](https://javadoop.com/post/java-concurrent-queue)[](https://javadoop.com/post/java-concurrent-queue)

## ConcurrentSkipListMap
下面这部分内容参考了极客时间专栏[《数据结构与算法之美》](https://time.geekbang.org/column/intro/126?code=zl3GYeAsRI4rEJIBNu5B/km7LSZsPDlGWQEpAYw5Vu0=&utm_term=SPoster)[](https://time.geekbang.org/column/intro/126?code=zl3GYeAsRI4rEJIBNu5B/km7LSZsPDlGWQEpAYw5Vu0=&utm_term=SPoster)以及《实战 Java 高并发程序设计》。

为了引出ConcurrentSkipListMap，先带着大家简单理解一下跳表。

对于一个单链表，即使链表是有序的，如果我们想要在其中查找某个数据，也只能从头到尾遍历链表，这样效率自然就会很低，跳表就不一样了。跳表是一种可以用来快速查找的数据结构，有点类似于平衡树。它们都可以对元素进行快速的查找。但一个重要的区别是：对平衡树的插入和删除往往很可能导致平衡树进行一次全局的调整。而对跳表的插入和删除只需要对整个数据结构的局部进行操作即可。这样带来的好处是：在高并发的情况下，你会需要一个全局锁来保证整个平衡树的线程安全。而对于跳表，你只需要部分锁即可。这样，在高并发环境下，你就可以拥有更好的性能。而就查询的性能而言，跳表的时间复杂度也是**O(logn)**所以在并发数据结构中，JDK 使用跳表来实现一个 Map。

跳表的本质是同时维护了多个链表，并且链表是分层的，

![1732497489954-ca17a8c3-4796-4013-bc6a-7648e9755025.jpeg](./img/OEVjXpRm2h_vzPro/1732497489954-ca17a8c3-4796-4013-bc6a-7648e9755025-677625.jpeg)

2级索引跳表

最低层的链表维护了跳表内所有的元素，每上面一层链表都是下面一层的子集。

跳表内的所有链表的元素都是排序的。查找时，可以从顶级链表开始找。一旦发现被查找的元素大于当前链表中的取值，就会转入下一层链表继续找。这也就是说在查找过程中，搜索是跳跃式的。如上图所示，在跳表中查找元素 18。

![1732497490079-e4647d45-dce6-4470-81e1-baae6d9cce95.jpeg](./img/OEVjXpRm2h_vzPro/1732497490079-e4647d45-dce6-4470-81e1-baae6d9cce95-894310.jpeg)

在跳表中查找元素18

查找 18 的时候原来需要遍历 18 次，现在只需要 7 次即可。针对链表长度比较大的时候，构建索引查找效率的提升就会非常明显。

从上面很容易看出，**跳表是一种利用空间换时间的算法。**

使用跳表实现Map和使用哈希算法实现Map的另外一个不同之处是：哈希并不会保存元素的顺序，而跳表内所有的元素都是排序的。因此在对跳表进行遍历时，你会得到一个有序的结果。所以，如果你的应用需要有序性，那么跳表就是你不二的选择。JDK 中实现这一数据结构的类是ConcurrentSkipListMap。

## 参考
+ 《实战 Java 高并发程序设计》
+ [https://javadoop.com/post/java-concurrent-queue](https://javadoop.com/post/java-concurrent-queue)[](https://javadoop.com/post/java-concurrent-queue)
+ [https://juejin.im/post/5aeebd02518825672f19c546](https://juejin.im/post/5aeebd02518825672f19c546)[](https://juejin.im/post/5aeebd02518825672f19c546)



> 更新: 2024-01-26 20:50:27  
原文: [https://www.yuque.com/vip6688/neho4x/nvhbbm40r2nxlsop](https://www.yuque.com/vip6688/neho4x/nvhbbm40r2nxlsop)
>



> 更新: 2024-11-25 09:18:10  
> 原文: <https://www.yuque.com/neumx/laxg2e/c00b0d94a612266f0ec44e87e4e6a848>