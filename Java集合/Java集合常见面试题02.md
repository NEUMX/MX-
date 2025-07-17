# Java集合常见面试题02

# Java集合常见面试题02
## Map（重要）
### HashMap 和 Hashtable 的区别
+ **线程是否安全：**HashMap是非线程安全的，Hashtable是线程安全的,因为Hashtable内部的方法基本都经过synchronized修饰。（如果你要保证线程安全的话就使用ConcurrentHashMap吧！）；
+ **效率：**因为线程安全的问题，HashMap要比Hashtable效率高一点。另外，Hashtable基本被淘汰，不要在代码中使用它；
+ **对 Null key 和 Null value 的支持：**HashMap可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；Hashtable 不允许有 null 键和 null 值，否则会抛出NullPointerException。
+ **初始容量大小和每次扩充容量大小的不同：**
    - ① 创建时如果不指定容量初始值，Hashtable默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。HashMap默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。
    - ② 创建时如果给定了容量初始值，那么Hashtable会直接使用你给定的大小，而HashMap会将其扩充为 2 的幂次方大小（HashMap中的tableSizeFor()方法保证，下面给出了源代码）。也就是说HashMap总是使用 2 的幂作为哈希表的大小,后面会介绍到为什么是 2 的幂次方。
+ **底层数据结构：**JDK1.8 以后的HashMap在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）时，将链表转化为红黑树（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树），以减少搜索时间（后文中我会结合源码对这一过程进行分析）。Hashtable没有这样的机制。

**HashMap****中带有初始容量的构造函数：**

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
public HashMap(int initialCapacity) {
this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

下面这个方法保证了HashMap总是使用 2 的幂作为哈希表的大小。

```java
/**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

### HashMap 和 HashSet 区别
如果你看过HashSet源码的话就应该知道：HashSet底层就是基于HashMap实现的。（HashSet的源码非常非常少，因为除了clone()、writeObject()、readObject()是HashSet自己不得不实现之外，其他方法都是直接调用HashMap中的方法。

| HashMap | HashSet |
| :---: | :---: |
| 实现了Map接口 | 实现Set接口 |
| 存储键值对 | 仅存储对象 |
| 调用put()向 map 中添加元素 | 调用add()方法向Set中添加元素 |
| HashMap使用键（Key）计算hashcode | HashSet使用成员对象来计算hashcode值，对于两个对象来说hashcode可能相同，所以equals()方法用来判断对象的相等性 |


### HashMap 和 TreeMap 区别
TreeMap和HashMap都继承自AbstractMap，但是需要注意的是TreeMap它还实现了NavigableMap接口和SortedMap接口。

![1732497587080-03a607af-8c14-4324-93e8-6fef18f33002.png](./img/LsaypSuGBTIuN4gW/1732497587080-03a607af-8c14-4324-93e8-6fef18f33002-379394.png)

TreeMap 继承关系图

实现NavigableMap接口让TreeMap有了对集合内元素的搜索的能力。

实现SortedMap接口让TreeMap有了对集合中的元素根据键排序的能力。默认是按 key 的升序排序，不过我们也可以指定排序的比较器。示例代码如下：

```java
/**
 * @author shuang.kou
 * @createTime 2020年06月15日 17:02:00
 */
public class Person {
    private Integer age;

    public Person(Integer age) {
        this.age = age;
    }

    public Integer getAge() {
        return age;
    }


    public static void main(String[] args) {
        TreeMap<Person, String> treeMap = new TreeMap<>(new Comparator<Person>() {
            @Override
            public int compare(Person person1, Person person2) {
                int num = person1.getAge() - person2.getAge();
                return Integer.compare(num, 0);
            }
        });
        treeMap.put(new Person(3), "person1");
        treeMap.put(new Person(18), "person2");
        treeMap.put(new Person(35), "person3");
        treeMap.put(new Person(16), "person4");
        treeMap.entrySet().stream().forEach(personStringEntry -> {
            System.out.println(personStringEntry.getValue());
        });
    }
}
```

输出:

```java
person1
person4
person2
person3
```

可以看出，TreeMap中的元素已经是按照Person的 age 字段的升序来排列了。

上面，我们是通过传入匿名内部类的方式实现的，你可以将代码替换成 Lambda 表达式实现的方式：

```java
TreeMap<Person, String> treeMap = new TreeMap<>((person1, person2) -> {
    int num = person1.getAge() - person2.getAge();
    return Integer.compare(num, 0);
});
```

**综上，相比于****HashMap****来说****TreeMap****主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力。**

### HashSet 如何检查重复?
以下内容摘自我的 Java 启蒙书《Head first java》第二版：

当你把对象加入HashSet时，HashSet会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals()方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。

在 JDK1.8 中，HashSet的add()方法只是简单的调用了HashMap的put()方法，并且判断了一下返回值以确保是否有重复元素。直接看一下HashSet中的源码：

```java
// Returns: true if this set did not already contain the specified element
// 返回值：当 set 中没有包含 add 的元素时返回真
public boolean add(E e) {
return map.put(e, PRESENT)==null;
}
```

而在HashMap的putVal()方法中也能看到如下说明：

```java
// Returns : previous value, or null if none
// 返回值：如果插入位置没有元素返回null，否则返回上一个元素
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    ...
}
```

也就是说，在 JDK1.8 中，实际上无论HashSet中是否已经存在了某元素，HashSet都会直接插入，只是会在add()方法的返回值处告诉我们插入前是否存在相同元素。

### HashMap 的底层实现
#### JDK1.8 之前
JDK1.8 之前HashMap底层是**数组和链表**结合在一起使用也就是**链表散列**。HashMap 通过 key 的hashcode经过扰动函数处理过后得到 hash 值，然后通过(n - 1) & hash判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

所谓扰动函数指的就是 HashMap 的hash方法。使用hash方法也就是扰动函数是为了防止一些实现比较差的hashCode()方法 换句话说使用扰动函数之后可以减少碰撞。

**JDK 1.8 HashMap 的 hash 方法源码:**

JDK 1.8 的 hash 方法 相比于 JDK 1.7 hash 方法更加简化，但是原理不变。

```java
static final int hash(Object key) {
int h;
// key.hashCode()：返回散列值也就是hashcode
// ^：按位异或
// >>>:无符号右移，忽略符号位，空位都以0补齐
return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

对比一下 JDK1.7 的 HashMap 的 hash 方法源码.

```java
static int hash(int h) {
// This function ensures that hashCodes that differ only by
// constant multiples at each bit position have a bounded
// number of collisions (approximately 8 at default load factor).

h ^= (h >>> 20) ^ (h >>> 12);
return h ^ (h >>> 7) ^ (h >>> 4);
}
```

相比于 JDK1.8 的 hash 方法 ，JDK 1.7 的 hash 方法的性能会稍差一点点，因为毕竟扰动了 4 次。

所谓**“拉链法”**就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。

![1732497587206-1a2f1f3c-e190-4f07-b1ba-193ef7554a31.png](./img/LsaypSuGBTIuN4gW/1732497587206-1a2f1f3c-e190-4f07-b1ba-193ef7554a31-611715.png)

jdk1.8 之前的内部结构-HashMap

#### JDK1.8 之后
相比于之前的版本， JDK1.8 之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

![1732497587274-6092666c-0904-4d7e-b7c2-258dcc459831.png](./img/LsaypSuGBTIuN4gW/1732497587274-6092666c-0904-4d7e-b7c2-258dcc459831-124948.png)

jdk1.8之后的内部结构-HashMap

TreeMap、TreeSet 以及 JDK1.8 之后的 HashMap 底层都用到了红黑树。红黑树就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构。

我们来结合源码分析一下HashMap链表到红黑树的转换。

*_1、__**putVal**__方法中执行链表转红黑树的判断逻辑。_*

链表的长度大于 8 的时候，就执行treeifyBin（转换红黑树）的逻辑。

```java
// 遍历链表
for (int binCount = 0; ; ++binCount) {
    // 遍历到链表最后一个节点
    if ((e = p.next) == null) {
        p.next = newNode(hash, key, value, null);
        // 如果链表元素个数大于等于TREEIFY_THRESHOLD（8）
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            // 红黑树转换（并不会直接转换成红黑树）
            treeifyBin(tab, hash);
        break;
    }
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        break;
    p = e;
}
```

*_2、__**treeifyBin**__方法中判断是否真的转换为红黑树。_*

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 判断当前数组的长度是否小于 64
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        // 如果当前数组的长度小于 64，那么会选择先进行数组扩容
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        // 否则才将列表转换为红黑树

        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树。

### HashMap 的长度为什么是 2 的幂次方
为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。我们上面也讲到了过了，Hash 值的范围值-2147483648 到 2147483647，前后加起来大概 40 亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个 40 亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“(n - 1) & hash”。（n 代表数组长度）。这也就解释了 HashMap 的长度为什么是 2 的幂次方。

**这个算法应该如何设计呢？**

我们首先可能会想到采用%取余的操作来实现。但是，重点来了：**“取余(%)操作中如果除数是 2 的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是 2 的 n 次方；）。”****并且****采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是 2 的幂次方。**

### HashMap 多线程操作导致死循环问题
JDK1.7 及之前版本的HashMap在多线程环境下扩容操作可能存在死循环问题，这是由于当一个桶位中有多个元素需要进行扩容时，多个线程同时对链表进行操作，头插法可能会导致链表中的节点指向错误的位置，从而形成一个环形链表，进而使得查询元素的操作陷入死循环无法结束。

为了解决这个问题，JDK1.8 版本的 HashMap 采用了尾插法而不是头插法来避免链表倒置，使得插入的节点永远都是放在链表的末尾，避免了链表中的环形结构。但是还是不建议在多线程下使用HashMap，因为多线程下使用HashMap还是会存在数据覆盖的问题。并发环境下，推荐使用ConcurrentHashMap。

一般面试中这样介绍就差不多，不需要记各种细节，个人觉得也没必要记。如果想要详细了解HashMap扩容导致死循环问题，可以看看耗子叔的这篇文章：[Java HashMap 的死循环](https://coolshell.cn/articles/9606.html)。

### HashMap 为什么线程不安全？
JDK1.7 及之前版本，在多线程环境下，HashMap扩容时会造成死循环和数据丢失的问题。

数据丢失这个在 JDK1.7 和 JDK 1.8 中都存在，这里以 JDK 1.8 为例进行介绍。

JDK 1.8 后，在HashMap中，多个键值对可能会被分配到同一个桶（bucket），并以链表或红黑树的形式存储。多个线程对HashMap的put操作会导致线程不安全，具体来说会有数据覆盖的风险。

举个例子：

+ 两个线程 1,2 同时进行 put 操作，并且发生了哈希冲突（hash 函数计算出的插入下标是相同的）。
+ 不同的线程可能在不同的时间片获得 CPU 执行的机会，当前线程 1 执行完哈希冲突判断后，由于时间片耗尽挂起。线程 2 先完成了插入操作。
+ 随后，线程 1 获得时间片，由于之前已经进行过 hash 碰撞的判断，所有此时会直接进行插入，这就导致线程 2 插入的数据被线程 1 覆盖了。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // ...
    // 判断是否出现 hash 碰撞
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
        // 桶中已经存在元素（处理hash冲突）
    else {
        // ...
    }
```

还有一种情况是这两个线程同时put操作导致size的值不正确，进而导致数据覆盖的问题：

1. 线程 1 执行if(++size > threshold)判断时，假设获得size的值为 10，由于时间片耗尽挂起。
2. 线程 2 也执行if(++size > threshold)判断，获得size的值也为 10，并将元素插入到该桶位中，并将size的值更新为 11。
3. 随后，线程 1 获得时间片，它也将元素放入桶位中，并将 size 的值更新为 11。
4. 线程 1、2 都执行了一次put操作，但是size的值只增加了 1，也就导致实际上只有一个元素被添加到了HashMap中。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // ...
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

### HashMap 常见的遍历方式?
[HashMap 的 7 种遍历方式与性能分析](https://mp.weixin.qq.com/s/zQBN3UvJDhRTKP6SzcZFKw)

_**�**_* 修正（参见：[issue#1411](https://github.com/Snailclimb/JavaGuide/issues/1411)）**：

这篇文章对于 parallelStream 遍历方式的性能分析有误，先说结论：**存在阻塞时 parallelStream 性能最高, 非阻塞时 parallelStream 性能最低**。

当遍历不存在阻塞时, parallelStream 的性能是最低的：

```java
Benchmark               Mode  Cnt     Score      Error  Units
Test.entrySet           avgt    5   288.651 ±   10.536  ns/op
Test.keySet             avgt    5   584.594 ±   21.431  ns/op
Test.lambda             avgt    5   221.791 ±   10.198  ns/op
Test.parallelStream     avgt    5  6919.163 ± 1116.139  ns/op
```

加入阻塞代码Thread.sleep(10)后, parallelStream 的性能才是最高的:

```java
Benchmark               Mode  Cnt           Score          Error  Units
Test.entrySet           avgt    5  1554828440.000 ± 23657748.653  ns/op
Test.keySet             avgt    5  1550612500.000 ±  6474562.858  ns/op
Test.lambda             avgt    5  1551065180.000 ± 19164407.426  ns/op
Test.parallelStream     avgt    5   186345456.667 ±  3210435.590  ns/op
```

### ConcurrentHashMap 和 Hashtable 的区别
ConcurrentHashMap和Hashtable的区别主要体现在实现线程安全的方式上不同。

+ **底层数据结构：****JDK1.7 的ConcurrentHashMap底层采用****分段的数组+链表**实现，JDK1.8 采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑二叉树。Hashtable和 JDK1.8 之前的HashMap的底层数据结构类似都是采用**数组+链表**的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；
+ **实现线程安全的方式（重要）：**
    - 在 JDK1.7 的时候，ConcurrentHashMap对整个桶数组进行了分割分段(Segment，分段锁)，每一把锁只锁容器其中一部分数据（下面有示意图），多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。
    - 到了 JDK1.8 的时候，ConcurrentHashMap已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用synchronized和 CAS 来操作。（JDK1.6 以后synchronized锁做了很多优化） 整个看起来就像是优化过且线程安全的HashMap，虽然在 JDK1.8 中还能看到Segment的数据结构，但是已经简化了属性，只是为了兼容旧版本；
    - **Hashtable****(同一把锁)**:使用synchronized来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

下面，我们再来看看两者底层数据结构的对比图。

**Hashtable**:

![1732497587361-d6d35006-7c7b-4607-a937-edd76f878847.png](./img/LsaypSuGBTIuN4gW/1732497587361-d6d35006-7c7b-4607-a937-edd76f878847-230469.png)

Hashtable 的内部结构

[https://www.cnblogs.com/chengxiao/p/6842045.html>](https://www.cnblogs.com/chengxiao/p/6842045.html>)

**JDK1.7 的 ConcurrentHashMap**：

![1732497587495-467ee855-b6a4-4b77-b517-6aa1eed55065.png](./img/LsaypSuGBTIuN4gW/1732497587495-467ee855-b6a4-4b77-b517-6aa1eed55065-013786.png)

Java7 ConcurrentHashMap 存储结构

ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。

Segment数组中的每个元素包含一个HashEntry数组，每个HashEntry数组属于链表结构。

**JDK1.8 的 ConcurrentHashMap**：

![1732497587571-8bb5dfac-eaf9-4369-a236-91a17f7bdfca.png](./img/LsaypSuGBTIuN4gW/1732497587571-8bb5dfac-eaf9-4369-a236-91a17f7bdfca-434523.png)

Java8 ConcurrentHashMap 存储结构

JDK1.8 的ConcurrentHashMap不再是**Segment 数组 + HashEntry 数组 + 链表**，而是**Node 数组 + 链表 / 红黑树**。不过，Node 只能用于链表的情况，红黑树的情况需要使用**TreeNode**。当冲突链表达到一定长度时，链表会转换成红黑树。

TreeNode是存储红黑树节点，被TreeBin包装。TreeBin通过root属性维护红黑树的根结点，因为红黑树在旋转的时候，根结点可能会被它原来的子节点替换掉，在这个时间点，如果有其他线程要写这棵红黑树就会发生线程不安全问题，所以在ConcurrentHashMap中TreeBin通过waiter属性维护当前使用这棵红黑树的线程，来防止其他线程的进入。

```java
static final class TreeBin<K,V> extends Node<K,V> {
    TreeNode<K,V> root;
    volatile TreeNode<K,V> first;
    volatile Thread waiter;
    volatile int lockState;
    // values for lockState
    static final int WRITER = 1; // set while holding write lock
    static final int WAITER = 2; // set when waiting for write lock
    static final int READER = 4; // increment value for setting read lock
    ...
}
```

### ConcurrentHashMap 线程安全的具体实现方式/底层具体实现
#### JDK1.8 之前
![1732497587639-adacb8f6-9932-47b4-987e-46626648fe7a.png](./img/LsaypSuGBTIuN4gW/1732497587639-adacb8f6-9932-47b4-987e-46626648fe7a-462665.png)

Java7 ConcurrentHashMap 存储结构

首先将数据分为一段一段（这个“段”就是Segment）的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

**ConcurrentHashMap****是由****Segment****数组结构和****HashEntry****数组结构组成**。

Segment继承了ReentrantLock,所以Segment是一种可重入锁，扮演锁的角色。HashEntry用于存储键值对数据。

```java
static class Segment<K,V> extends ReentrantLock implements Serializable {
}
```

一个ConcurrentHashMap里包含一个Segment数组，Segment的个数一旦**初始化就不能改变**。Segment数组的大小默认是 16，也就是说默认可以同时支持 16 个线程并发写。

Segment的结构和HashMap类似，是一种数组和链表结构，一个Segment包含一个HashEntry数组，每个HashEntry是一个链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改时，必须首先获得对应的Segment的锁。也就是说，对同一Segment的并发写入会被阻塞，不同Segment的写入是可以并发执行的。

#### JDK1.8 之后
![1732497587776-17a22f66-5c50-4ba5-a653-4ed9efe701d7.png](./img/LsaypSuGBTIuN4gW/1732497587776-17a22f66-5c50-4ba5-a653-4ed9efe701d7-395776.png)

Java8 ConcurrentHashMap 存储结构

Java 8 几乎完全重写了ConcurrentHashMap，代码量从原来 Java 7 中的 1000 多行，变成了现在的 6000 多行。

ConcurrentHashMap取消了Segment分段锁，采用Node + CAS + synchronized来保证并发安全。数据结构跟HashMap1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))）。

Java 8 中，锁粒度更细，synchronized只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，就不会影响其他 Node 的读写，效率大幅提升。

### JDK 1.7 和 JDK 1.8 的 ConcurrentHashMap 实现有什么不同？
+ **线程安全实现方式**：JDK 1.7 采用Segment分段锁来保证安全，Segment是继承自ReentrantLock。JDK1.8 放弃了Segment分段锁的设计，采用Node + CAS + synchronized保证线程安全，锁粒度更细，synchronized只锁定当前链表或红黑二叉树的首节点。
+ **Hash 碰撞解决方法**: JDK 1.7 采用拉链法，JDK1.8 采用拉链法结合红黑树（链表长度超过一定阈值时，将链表转换为红黑树）。
+ **并发度**：JDK 1.7 最大并发度是 Segment 的个数，默认是 16。JDK 1.8 最大并发度是 Node 数组的大小，并发度更大。

### ConcurrentHashMap 为什么 key 和 value 不能为 null？
ConcurrentHashMap的 key 和 value 不能为 null 主要是为了避免二义性。null 是一个特殊的值，表示没有对象或没有引用。如果你用 null 作为键，那么你就无法区分这个键是否存在于ConcurrentHashMap中，还是根本没有这个键。同样，如果你用 null 作为值，那么你就无法区分这个值是否是真正存储在ConcurrentHashMap中的，还是因为找不到对应的键而返回的。

拿 get 方法取值来说，返回的结果为 null 存在两种情况：

+ 值没有在集合中 ；
+ 值本身就是 null。

这也就是二义性的由来。

具体可以参考[ConcurrentHashMap 源码分析](https://www.yuque.com/vip6688/neho4x/fcgn9g4lv136780q)

多线程环境下，存在一个线程操作该ConcurrentHashMap时，其他的线程将该ConcurrentHashMap修改的情况，所以无法通过containsKey(key)来判断否存在这个键值对，也就没办法解决二义性问题了。

与此形成对比的是，HashMap可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个。如果传入 null 作为参数，就会返回 hash 值为 0 的位置的值。单线程环境下，不存在一个线程操作该 HashMap 时，其他的线程将该HashMap修改的情况，所以可以通过contains(key)来做判断是否存在这个键值对，从而做相应的处理，也就不存在二义性问题。

也就是说，**<font style="background-color:#d9eafc;">多线程下无法正确判定键值对是否存在（存在其他线程修改的情况），单线程是可以的（不存在其他线程修改的情况）。</font>**

如果你确实需要在 ConcurrentHashMap 中使用 null 的话，可以使用一个特殊的静态空对象来代替 null。

```java
public static final Object NULL = new Object();
```

最后，再分享一下ConcurrentHashMap作者本人 (Doug Lea)对于这个问题的回答：

The main reason that nulls aren't allowed in ConcurrentMaps (ConcurrentHashMaps, ConcurrentSkipListMaps) is that ambiguities that may be just barely tolerable in non-concurrent maps can't be accommodated. The main one is that ifmap.get(key)returnsnull, you can't detect whether the key explicitly maps tonullvs the key isn't mapped. In a non-concurrent map, you can check this viamap.contains(key), but in a concurrent one, the map might have changed between calls.

翻译过来之后的，大致意思还是单线程下可以容忍歧义，而多线程下无法容忍。

### ConcurrentHashMap 能保证复合操作的原子性吗？
ConcurrentHashMap是线程安全的，意味着它可以保证多个线程同时对它进行读写操作时，不会出现数据不一致的情况，也不会导致 JDK1.7 及之前版本的HashMap多线程操作导致死循环问题。但是，这并不意味着它可以保证所有的复合操作都是原子性的，一定不要搞混了！

复合操作是指由多个基本操作(如put、get、remove、containsKey等)组成的操作，例如先判断某个键是否存在containsKey(key)，然后根据结果进行插入或更新put(key, value)。这种操作在执行过程中可能会被其他线程打断，导致结果不符合预期。

例如，有两个线程 A 和 B 同时对ConcurrentHashMap进行复合操作，如下：

```java
// 线程 A
if (!map.containsKey(key)) {
    map.put(key, value);
}
// 线程 B
if (!map.containsKey(key)) {
    map.put(key, anotherValue);
}
```

如果线程 A 和 B 的执行顺序是这样：

1. 线程 A 判断 map 中不存在 key
2. 线程 B 判断 map 中不存在 key
3. 线程 B 将 (key, anotherValue) 插入 map
4. 线程 A 将 (key, value) 插入 map

那么最终的结果是 (key, value)，而不是预期的 (key, anotherValue)。这就是复合操作的非原子性导致的问题。

**那如何保证****ConcurrentHashMap****复合操作的原子性呢？**

ConcurrentHashMap提供了一些原子性的复合操作，如putIfAbsent、compute、computeIfAbsent、computeIfPresent、merge等。这些方法都可以接受一个函数作为参数，根据给定的 key 和 value 来计算一个新的 value，并且将其更新到 map 中。

上面的代码可以改写为：

```java
// 线程 A
map.putIfAbsent(key, value);
// 线程 B
map.putIfAbsent(key, anotherValue);
```

或者：

```java
// 线程 A
map.computeIfAbsent(key, k -> value);
// 线程 B
map.computeIfAbsent(key, k -> anotherValue);
```

很多同学可能会说了，这种情况也能加锁同步呀！确实可以，但不建议使用加锁的同步机制，违背了使用ConcurrentHashMap的初衷。在使用ConcurrentHashMap的时候，尽量使用这些原子性的复合操作方法来保证原子性。

## Collections 工具类（不重要）
**Collections****工具类常用方法**:

+ 排序
+ 查找,替换操作
+ 同步控制(不推荐，需要线程安全的集合类型时请考虑使用 JUC 包下的并发集合)

### 排序操作
```java
void reverse(List list)//反转
void shuffle(List list)//随机排序
void sort(List list)//按自然排序的升序排序
void sort(List list, Comparator c)//定制排序，由Comparator控制排序逻辑
void swap(List list, int i , int j)//交换两个索引位置的元素
void rotate(List list, int distance)//旋转。当distance为正数时，将list后distance个元素整体移到前面。当distance为负数时，将 list的前distance个元素整体移到后面
```

### 查找,替换操作
```java
int binarySearch(List list, Object key)//对List进行二分查找，返回索引，注意List必须是有序的
int max(Collection coll)//根据元素的自然顺序，返回最大的元素。 类比int min(Collection coll)
int max(Collection coll, Comparator c)//根据定制排序，返回最大元素，排序规则由Comparatator类控制。类比int min(Collection coll, Comparator c)
void fill(List list, Object obj)//用指定的元素代替指定list中的所有元素
int frequency(Collection c, Object o)//统计元素出现次数
int indexOfSubList(List list, List target)//统计target在list中第一次出现的索引，找不到则返回-1，类比int lastIndexOfSubList(List source, list target)
boolean replaceAll(List list, Object oldVal, Object newVal)//用新元素替换旧元素
```

### 同步控制
Collections提供了多个synchronizedXxx()方法·，该方法可以将指定集合包装成线程同步的集合，从而解决多线程并发访问集合时的线程安全问题。

我们知道HashSet，TreeSet，ArrayList,LinkedList,HashMap,TreeMap都是线程不安全的。Collections提供了多个静态方法可以把他们包装成线程同步的集合。

**最好不要用下面这些方法，效率非常低，需要线程安全的集合类型时请考虑使用 JUC 包下的并发集合。**

方法如下：

```java
synchronizedCollection(Collection<T>  c) //返回指定 collection 支持的同步（线程安全的）collection。
synchronizedList(List<T> list)//返回指定列表支持的同步（线程安全的）List。
synchronizedMap(Map<K,V> m) //返回由指定映射支持的同步（线程安全的）Map。
synchronizedSet(Set<T> s) //返回指定 set 支持的同步（线程安全的）set。
```





> 更新: 2024-01-15 23:15:07  
原文: [https://www.yuque.com/vip6688/neho4x/zesg3fb5drc6zkmu](https://www.yuque.com/vip6688/neho4x/zesg3fb5drc6zkmu)
>



> 更新: 2024-11-25 09:19:48  
> 原文: <https://www.yuque.com/neumx/laxg2e/f5418b06079b017eba50a6ac8f743cf2>