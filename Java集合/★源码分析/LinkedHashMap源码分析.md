# LinkedHashMap源码分析

# LinkedHashMap 源码分析
## LinkedHashMap 简介
LinkedHashMap是 Java 提供的一个集合类，它继承自HashMap，并在HashMap基础上维护一条双向链表，使得具备如下特性:

1. 支持遍历时会按照插入顺序有序进行迭代。
2. 支持按照元素访问顺序排序,适用于封装 LRU 缓存工具。
3. 因为内部使用双向链表维护各个节点，所以遍历时的效率和元素个数成正比，相较于和容量成正比的 HashMap 来说，迭代效率会高很多。

LinkedHashMap逻辑结构如下图所示，它是在HashMap基础上在各个节点之间维护一条双向链表，使得原本散列在不同 bucket 上的节点、链表、红黑树有序关联起来。

![1732497601036-9b7c2644-b24f-4fce-a679-53dcbfa6f470.png](./img/JsmHnEaRo3VEdR_h/1732497601036-9b7c2644-b24f-4fce-a679-53dcbfa6f470-536733.png)

LinkedHashMap 逻辑结构

## LinkedHashMap 使用示例
### 插入顺序遍历
如下所示，我们按照顺序往LinkedHashMap添加元素然后进行遍历。

```plain
HashMap < String, String > map = new LinkedHashMap < > ();
map.put("a", "2");
map.put("g", "3");
map.put("r", "1");
map.put("e", "23");

for (Map.Entry < String, String > entry: map.entrySet()) {
    System.out.println(entry.getKey() + ":" + entry.getValue());
}
```

输出：

```plain
a:2
g:3
r:1
e:23
```

可以看出，LinkedHashMap的迭代顺序是和插入顺序一致的,这一点是HashMap所不具备的。

### 访问顺序遍历
LinkedHashMap定义了排序模式accessOrder(boolean 类型，默认为 false)，访问顺序则为 true，插入顺序则为 false。

为了实现访问顺序遍历，我们可以使用传入accessOrder属性的LinkedHashMap构造方法，并将accessOrder设置为 true，表示其具备访问有序性。

```plain
LinkedHashMap<Integer, String> map = new LinkedHashMap<>(16, 0.75f, true);
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.put(4, "four");
map.put(5, "five");
//访问元素2,该元素会被移动至链表末端
map.get(2);
//访问元素3,该元素会被移动至链表末端
map.get(3);
for (Map.Entry<Integer, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " : " + entry.getValue());
}
```

输出：

```plain
1 : one
4 : four
5 : five
2 : two
3 : three
```

可以看出，LinkedHashMap的迭代顺序是和访问顺序一致的。

### LRU 缓存
从上一个我们可以了解到通过LinkedHashMap我们可以封装一个简易版的 LRU（**L**east**R**ecently**U**sed，最近最少使用） 缓存，确保当存放的元素超过容器容量时，将最近最少访问的元素移除。

![1732497601114-a5048d50-782b-4496-a137-a356c34868b7.png](./img/JsmHnEaRo3VEdR_h/1732497601114-a5048d50-782b-4496-a137-a356c34868b7-980170.png)

具体实现思路如下：

+ 继承LinkedHashMap;
+ 构造方法中指定accessOrder为 true ，这样在访问元素时就会把该元素移动到链表尾部，链表首元素就是最近最少被访问的元素；
+ 重写removeEldestEntry方法，该方法会返回一个 boolean 值，告知LinkedHashMap是否需要移除链表首元素（缓存容量有限）。

```plain
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    /**
     * 判断size超过容量时返回true，告知LinkedHashMap移除最老的缓存项(即链表的第一个元素)
     */
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

测试代码如下，笔者初始化缓存容量为 2，然后按照次序先后添加 4 个元素。

```plain
LRUCache < Integer, String > cache = new LRUCache < > (2);
cache.put(1, "one");
cache.put(2, "two");
cache.put(3, "three");
cache.put(4, "four");
for (int i = 0; i < 4; i++) {
    System.out.println(cache.get(i));
}
```

输出：

```plain
null
null
three
four
```

从输出结果来看，由于缓存容量为 2 ，因此，添加第 3 个元素时，第 1 个元素会被删除。添加第 4 个元素时，第 2 个元素会被删除。

## LinkedHashMap 源码解析
### Node 的设计
在正式讨论LinkedHashMap前，我们先来聊聊LinkedHashMap节点Entry的设计,我们都知道HashMap的 bucket 上的因为冲突转为链表的节点会在符合以下两个条件时会将链表转为红黑树:

1. ~~链表上的节点个数达到树化的阈值 7，即~~~~TREEIFY_THRESHOLD - 1~~~~。~~
2. bucket 的容量达到最小的树化容量即MIN_TREEIFY_CAPACITY。

_**�**_* 修正（参见：[issue#2147](https://github.com/Snailclimb/JavaGuide/issues/2147)[open in new window](https://github.com/Snailclimb/JavaGuide/issues/2147)）**：链表上的节点个数达到树化的阈值是 8 而非 7。因为源码的判断是从链表初始元素开始遍历，下标是从 0 开始的，所以判断条件设置为 8-1=7，其实是迭代到尾部元素时再判断整个链表长度大于等于 8 才进行树化操作。

![1732497601228-4f3ced87-9b29-47dc-b5ef-3097e9bfcc64.png](./img/JsmHnEaRo3VEdR_h/1732497601228-4f3ced87-9b29-47dc-b5ef-3097e9bfcc64-966111.png)

而LinkedHashMap是在HashMap的基础上为 bucket 上的每一个节点建立一条双向链表，这就使得转为红黑树的树节点也需要具备双向链表节点的特性，即每一个树节点都需要拥有两个引用存储前驱节点和后继节点的地址,所以对于树节点类TreeNode的设计就是一个比较棘手的问题。

对此我们不妨来看看两者之间节点类的类图，可以看到:

1. LinkedHashMap的节点内部类Entry基于HashMap的基础上，增加before和after指针使节点具备双向链表的特性。
2. HashMap的树节点TreeNode继承了具备双向链表特性的LinkedHashMap的Entry。

![1732497601353-a81a61e6-8fba-44e1-8e09-0d1e426c60dd.png](./img/JsmHnEaRo3VEdR_h/1732497601353-a81a61e6-8fba-44e1-8e09-0d1e426c60dd-094635.png)

LinkedHashMap 和 HashMap 之间的关系

很多读者此时就会有这样一个疑问，为什么HashMap的树节点TreeNode要通过LinkedHashMap获取双向链表的特性呢?为什么不直接在Node上实现前驱和后继指针呢?

先来回答第一个问题，我们都知道LinkedHashMap是在HashMap基础上对节点增加双向指针实现双向链表的特性,所以LinkedHashMap内部链表转红黑树时，对应的节点会转为树节点TreeNode,为了保证使用LinkedHashMap时树节点具备双向链表的特性，所以树节点TreeNode需要继承LinkedHashMap的Entry。

再来说说第二个问题，我们直接在HashMap的节点Node上直接实现前驱和后继指针,然后TreeNode直接继承Node获取双向链表的特性为什么不行呢？其实这样做也是可以的。只不过这种做法会使得使用HashMap时存储键值对的节点类Node多了两个没有必要的引用，占用没必要的内存空间。

所以，为了保证HashMap底层的节点类Node没有多余的引用，又要保证LinkedHashMap的节点类Entry拥有存储链表的引用，设计者就让LinkedHashMap的节点Entry去继承 Node 并增加存储前驱后继节点的引用before、after，让需要用到链表特性的节点去实现需要的逻辑。然后树节点TreeNode再通过继承Entry获取before、after两个指针。

```plain
static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

但是这样做，不也使得使用HashMap时的TreeNode多了两个没有必要的引用吗?这不也是一种空间的浪费吗？

```plain
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    //略

}
```

对于这个问题,引用作者的一段注释，作者们认为在良好的hashCode算法时，HashMap转红黑树的概率不大。就算转为红黑树变为树节点，也可能会因为移除或者扩容将TreeNode变为Node，所以TreeNode的使用概率不算很大，对于这一点资源空间的浪费是可以接受的。

```plain
Because TreeNodes are about twice the size of regular nodes, we
use them only when bins contain enough nodes to warrant use
(see TREEIFY_THRESHOLD). And when they become too small (due to
removal or resizing) they are converted back to plain bins.  In
usages with well-distributed user hashCodes, tree bins are
rarely used.  Ideally, under random hashCodes, the frequency of
nodes in bins follows a Poisson distribution
```

### 构造方法
LinkedHashMap构造方法有 4 个实现也比较简单，直接调用父类即HashMap的构造方法完成初始化。

```plain
public LinkedHashMap() {
    super();
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity) {
    super(initialCapacity);
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity,
    float loadFactor,
    boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

我们上面也提到了，默认情况下accessOrder为 false，如果我们要让LinkedHashMap实现键值对按照访问顺序排序(即将最近未访问的元素排在链表首部、最近访问的元素移动到链表尾部)，需要调用第 4 个构造方法将accessOrder设置为 true。

### get 方法
get方法是LinkedHashMap增删改查操作中唯一一个重写的方法，accessOrder为 true 的情况下， 它会在元素查询完成之后，将当前访问的元素移到链表的末尾。

```plain
public V get(Object key) {
     Node < K, V > e;
     //获取key的键值对,若为空直接返回
     if ((e = getNode(hash(key), key)) == null)
         return null;
     //若accessOrder为true，则调用afterNodeAccess将当前元素移到链表末尾
     if (accessOrder)
         afterNodeAccess(e);
     //返回键值对的值
     return e.value;
 }
```

从源码可以看出，get的执行步骤非常简单:

1. 调用父类即HashMap的getNode获取键值对，若为空则直接返回。
2. 判断accessOrder是否为 true，若为 true 则说明需要保证LinkedHashMap的链表访问有序性，执行步骤 3。
3. 调用LinkedHashMap重写的afterNodeAccess将当前元素添加到链表末尾。

关键点在于afterNodeAccess方法的实现，这个方法负责将元素移动到链表末尾。

```plain
void afterNodeAccess(Node < K, V > e) { // move node to last
    LinkedHashMap.Entry < K, V > last;
    //如果accessOrder 且当前节点不未链表尾节点
    if (accessOrder && (last = tail) != e) {

        //获取当前节点、以及前驱节点和后继节点
        LinkedHashMap.Entry < K, V > p =
            (LinkedHashMap.Entry < K, V > ) e, b = p.before, a = p.after;

        //将当前节点的后继节点指针指向空，使其和后继节点断开联系
        p.after = null;

        //如果前驱节点为空，则说明当前节点是链表的首节点，故将后继节点设置为首节点
        if (b == null)
            head = a;
        else
            //如果后继节点不为空，则让前驱节点指向后继节点
            b.after = a;

        //如果后继节点不为空，则让后继节点指向前驱节点
        if (a != null)
            a.before = b;
        else
            //如果后继节点为空，则说明当前节点在链表最末尾，直接让last 指向前驱节点,这个 else其实 没有意义，因为最开头if已经确保了p不是尾结点了，自然after不会是null
            last = b;

        //如果last为空，则说明当前链表只有一个节点p，则将head指向p
        if (last == null)
            head = p;
        else {
            //反之让p的前驱指针指向尾节点，再让尾节点的前驱指针指向p
            p.before = last;
            last.after = p;
        }
        //tail指向p，自此将节点p移动到链表末尾
        tail = p;

        ++modCount;
    }
}
```

从源码可以看出，afterNodeAccess方法完成了下面这些操作:

1. 如果accessOrder为 true 且链表尾部不为当前节点 p，我们则需要将当前节点移到链表尾部。
2. 获取当前节点 p、以及它的前驱节点 b 和后继节点 a。
3. 将当前节点 p 的后继指针设置为 null，使其和后继节点 p 断开联系。
4. 尝试将前驱节点指向后继节点，若前驱节点为空，则说明当前节点 p 就是链表首节点，故直接将后继节点 a 设置为首节点，随后我们再将 p 追加到 a 的末尾。
5. 再尝试让后继节点 a 指向前驱节点 b。
6. 上述操作让前驱节点和后继节点完成关联，并将当前节点 p 独立出来，这一步则是将当前节点 p 追加到链表末端，如果链表末端为空，则说明当前链表只有一个节点 p，所以直接让 head 指向 p 即可。
7. 上述操作已经将 p 成功到达链表末端，最后我们将 tail 指针即指向链表末端的指针指向 p 即可。

可以结合这张图理解，展示了 key 为 13 的元素被移动到了链表尾部。

![1732497601481-d7d6a553-6f01-47ac-8f0d-1361b48bc0fa.png](./img/JsmHnEaRo3VEdR_h/1732497601481-d7d6a553-6f01-47ac-8f0d-1361b48bc0fa-185804.png)

LinkedHashMap 移动元素 13 到链表尾部

看不太懂也没关系，知道这个方法的作用就够了，后续有时间再慢慢消化。

### remove 方法后置操作——afterNodeRemoval
LinkedHashMap并没有对remove方法进行重写，而是直接继承HashMap的remove方法，为了保证键值对移除后双向链表中的节点也会同步被移除，LinkedHashMap重写了HashMap的空实现方法afterNodeRemoval。

```plain
final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        //略
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
                //HashMap的removeNode完成元素移除后会调用afterNodeRemoval进行移除后置操作
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
//空实现
void afterNodeRemoval(Node<K,V> p) { }
```

我们可以看到从HashMap继承来的remove方法内部调用的removeNode方法将节点从 bucket 删除后，调用了afterNodeRemoval。

```plain
void afterNodeRemoval(Node<K,V> e) { // unlink

        //获取当前节点p、以及e的前驱节点b和后继节点a
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        //将p的前驱和后继指针都设置为null，使其和前驱、后继节点断开联系
        p.before = p.after = null;

        //如果前驱节点为空，则说明当前节点p是链表首节点，让head指针指向后继节点a即可
        if (b == null)
            head = a;
        else
        //如果前驱节点b不为空，则让b直接指向后继节点a
            b.after = a;

        //如果后继节点为空，则说明当前节点p在链表末端，所以直接让tail指针指向前驱节点a即可
        if (a == null)
            tail = b;
        else
        //反之后继节点的前驱指针直接指向前驱节点
            a.before = b;
    }
```

从源码可以看出，afterNodeRemoval方法的整体操作就是让当前节点 p 和前驱节点、后继节点断开联系，等待 gc 回收，整体步骤为:

1. 获取当前节点 p、以及 e 的前驱节点 b 和后继节点 a。
2. 让当前节点 p 和其前驱、后继节点断开联系。
3. 尝试让前驱节点 b 指向后继节点 a，若 b 为空则说明当前节点 p 在链表首部，我们直接将 head 指向后继节点 a 即可。
4. 尝试让后继节点 a 指向前驱节点 b，若 a 为空则说明当前节点 p 在链表末端，所以直接让 tail 指针指向前驱节点 a 即可。

可以结合这张图理解，展示了 key 为 13 的元素被删除，也就是从链表中移除了这个元素。

![1732497601617-5a4c133b-bd0e-496c-8821-600f6f61f526.png](./img/JsmHnEaRo3VEdR_h/1732497601617-5a4c133b-bd0e-496c-8821-600f6f61f526-362159.png)

LinkedHashMap 删除元素 13

看不太懂也没关系，知道这个方法的作用就够了，后续有时间再慢慢消化。

### put 方法后置操作——afterNodeInsertion
同样的LinkedHashMap并没有实现插入方法，而是直接继承HashMap的所有插入方法交由用户使用，但为了维护双向链表访问的有序性，它做了这样两件事:

1. 重写afterNodeAccess(上文提到过),如果当前被插入的 key 已存在与map中，因为LinkedHashMap的插入操作会将新节点追加至链表末尾，所以对于存在的 key 则调用afterNodeAccess将其放到链表末端。
2. 重写了HashMap的afterNodeInsertion方法，当removeEldestEntry返回 true 时，会将链表首节点移除。

这一点我们可以在HashMap的插入操作核心方法putVal中看到。

```plain
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
            //略
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                 //如果当前的key在map中存在，则调用afterNodeAccess
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
         //调用插入后置方法，该方法被LinkedHashMap重写
        afterNodeInsertion(evict);
        return null;
    }
```

上述步骤的源码上文已经解释过了，所以这里我们着重了解一下afterNodeInsertion的工作流程，假设我们的重写了removeEldestEntry，当链表size超过capacity时，就返回 true。

```plain
/**
 * 判断size超过容量时返回true，告知LinkedHashMap移除最老的缓存项(即链表的第一个元素)
 */
protected boolean removeEldestEntry(Map.Entry < K, V > eldest) {
    return size() > capacity;
}
```

以下图为例，假设笔者最后新插入了一个不存在的节点 19,假设capacity为 4，所以removeEldestEntry返回 true，我们要将链表首节点移除。

![1732497601865-f40ff0bf-35f0-4a3f-99ee-8602c5da7c96.png](./img/JsmHnEaRo3VEdR_h/1732497601865-f40ff0bf-35f0-4a3f-99ee-8602c5da7c96-358449.png)

LinkedHashMap 中插入新元素 19

移除的步骤很简单，查看链表首节点是否存在，若存在则断开首节点和后继节点的关系，并让首节点指针指向下一节点，所以 head 指针指向了 12，节点 10 成为没有任何引用指向的空对象，等待 GC。

![1732497601956-bb0d75de-adba-4343-b6ff-12de95c66878.png](./img/JsmHnEaRo3VEdR_h/1732497601956-bb0d75de-adba-4343-b6ff-12de95c66878-400706.png)

LinkedHashMap 中插入新元素 19

```plain
void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        //如果evict为true且队首元素不为空以及removeEldestEntry返回true，则说明我们需要最老的元素(即在链表首部的元素)移除。
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            //获取链表首部的键值对的key
            K key = first.key;
            //调用removeNode将元素从HashMap的bucket中移除，并和LinkedHashMap的双向链表断开，等待gc回收
            removeNode(hash(key), key, null, false, true);
        }
    }
```

从源码可以看出，afterNodeInsertion方法完成了下面这些操作:

1. 判断eldest是否为 true，只有为 true 才能说明可能需要将最年长的键值对(即链表首部的元素)进行移除，具体是否具体要进行移除，还得确定链表是否为空((first = head) != null)，以及removeEldestEntry方法是否返回 true，只有这两个方法返回 true 才能确定当前链表不为空，且链表需要进行移除操作了。
2. 获取链表第一个元素的 key。
3. 调用HashMap的removeNode方法，该方法我们上文提到过，它会将节点从HashMap的 bucket 中移除，并且LinkedHashMap还重写了removeNode中的afterNodeRemoval方法，所以这一步将通过调用removeNode将元素从HashMap的 bucket 中移除，并和LinkedHashMap的双向链表断开，等待 gc 回收。

## LinkedHashMap 和 HashMap 遍历性能比较
LinkedHashMap维护了一个双向链表来记录数据插入的顺序，因此在迭代遍历生成的迭代器的时候，是按照双向链表的路径进行遍历的。这一点相比于HashMap那种遍历整个 bucket 的方式来说，高效需多。

这一点我们可以从两者的迭代器中得以印证，先来看看HashMap的迭代器，可以看到HashMap迭代键值对时会用到一个nextNode方法，该方法会返回 next 指向的下一个元素，并会从 next 开始遍历 bucket 找到下一个 bucket 中不为空的元素 Node。

```plain
final class EntryIterator extends HashIterator
 implements Iterator < Map.Entry < K, V >> {
     public final Map.Entry < K,
     V > next() {
         return nextNode();
     }
 }

 //获取下一个Node
 final Node < K, V > nextNode() {
     Node < K, V > [] t;
     //获取下一个元素next
     Node < K, V > e = next;
     if (modCount != expectedModCount)
         throw new ConcurrentModificationException();
     if (e == null)
         throw new NoSuchElementException();
     //将next指向bucket中下一个不为空的Node
     if ((next = (current = e).next) == null && (t = table) != null) {
         do {} while (index < t.length && (next = t[index++]) == null);
     }
     return e;
 }
```

相比之下LinkedHashMap的迭代器则是直接使用通过after指针快速定位到当前节点的后继节点，简洁高效需多。

```plain
final class LinkedEntryIterator extends LinkedHashIterator
 implements Iterator < Map.Entry < K, V >> {
     public final Map.Entry < K,
     V > next() {
         return nextNode();
     }
 }
 //获取下一个Node
 final LinkedHashMap.Entry < K, V > nextNode() {
     //获取下一个节点next
     LinkedHashMap.Entry < K, V > e = next;
     if (modCount != expectedModCount)
         throw new ConcurrentModificationException();
     if (e == null)
         throw new NoSuchElementException();
     //current 指针指向当前节点
     current = e;
     //next直接当前节点的after指针快速定位到下一个节点
     next = e.after;
     return e;
 }
```

为了验证笔者所说的观点，笔者对这两个容器进行了压测，测试插入 1000w 和迭代 1000w 条数据的耗时，代码如下:

```plain
int count = 1000_0000;
Map<Integer, Integer> hashMap = new HashMap<>();
Map<Integer, Integer> linkedHashMap = new LinkedHashMap<>();

long start, end;

start = System.currentTimeMillis();
for (int i = 0; i < count; i++) {
    hashMap.put(ThreadLocalRandom.current().nextInt(1, count), ThreadLocalRandom.current().nextInt(0, count));
}
end = System.currentTimeMillis();
System.out.println("map time putVal: " + (end - start));

start = System.currentTimeMillis();
for (int i = 0; i < count; i++) {
    linkedHashMap.put(ThreadLocalRandom.current().nextInt(1, count), ThreadLocalRandom.current().nextInt(0, count));
}
end = System.currentTimeMillis();
System.out.println("linkedHashMap putVal time: " + (end - start));

start = System.currentTimeMillis();
long num = 0;
for (Integer v : hashMap.values()) {
    num = num + v;
}
end = System.currentTimeMillis();
System.out.println("map get time: " + (end - start));

start = System.currentTimeMillis();
for (Integer v : linkedHashMap.values()) {
    num = num + v;
}
end = System.currentTimeMillis();
System.out.println("linkedHashMap get time: " + (end - start));
System.out.println(num);
```

从输出结果来看，因为LinkedHashMap需要维护双向链表的缘故，插入元素相较于HashMap会更耗时，但是有了双向链表明确的前后节点关系，迭代效率相对于前者高效了需多。不过，总体来说却别不大，毕竟数据量这么庞大。

```plain
map time putVal: 5880
linkedHashMap putVal time: 7567
map get time: 143
linkedHashMap get time: 67
63208969074998
```

## LinkedHashMap 常见面试题
### 什么是 LinkedHashMap？
LinkedHashMap是 Java 集合框架中HashMap的一个子类，它继承了HashMap的所有属性和方法，并且在HashMap的基础重写了afterNodeRemoval、afterNodeInsertion、afterNodeAccess方法。使之拥有顺序插入和访问有序的特性。

### LinkedHashMap 如何按照插入顺序迭代元素？
LinkedHashMap按照插入顺序迭代元素是它的默认行为。LinkedHashMap内部维护了一个双向链表，用于记录元素的插入顺序。因此，当使用迭代器迭代元素时，元素的顺序与它们最初插入的顺序相同。

### LinkedHashMap 如何按照访问顺序迭代元素？
LinkedHashMap可以通过构造函数中的accessOrder参数指定按照访问顺序迭代元素。当accessOrder为 true 时，每次访问一个元素时，该元素会被移动到链表的末尾，因此下次访问该元素时，它就会成为链表中的最后一个元素，从而实现按照访问顺序迭代元素。

### LinkedHashMap 如何实现 LRU 缓存？
将accessOrder设置为 true 并重写removeEldestEntry方法当链表大小超过容量时返回 true，使得每次访问一个元素时，该元素会被移动到链表的末尾。一旦插入操作让removeEldestEntry返回 true 时，视为缓存已满，LinkedHashMap就会将链表首元素移除，由此我们就能实现一个 LRU 缓存。

### LinkedHashMap 和 HashMap 有什么区别？
LinkedHashMap和HashMap都是 Java 集合框架中的 Map 接口的实现类。它们的最大区别在于迭代元素的顺序。HashMap迭代元素的顺序是不确定的，而LinkedHashMap提供了按照插入顺序或访问顺序迭代元素的功能。此外，LinkedHashMap内部维护了一个双向链表，用于记录元素的插入顺序或访问顺序，而HashMap则没有这个链表。因此，LinkedHashMap的插入性能可能会比HashMap略低，但它提供了更多的功能并且迭代效率相较于HashMap更加高效。

## 参考文献
+ LinkedHashMap 源码详细分析（JDK1.8）:[https://www.imooc.com/article/22931](https://www.imooc.com/article/22931)[open in new window](https://www.imooc.com/article/22931)
+ HashMap 与 LinkedHashMap:[https://www.cnblogs.com/Spground/p/8536148.html](https://www.cnblogs.com/Spground/p/8536148.html)[open in new window](https://www.cnblogs.com/Spground/p/8536148.html)
+ 源于 LinkedHashMap 源码:[https://leetcode.cn/problems/lru-cache/solution/yuan-yu-linkedhashmapyuan-ma-by-jeromememory/](https://leetcode.cn/problems/lru-cache/solution/yuan-yu-linkedhashmapyuan-ma-by-jeromememory/)[open in new window](https://leetcode.cn/problems/lru-cache/solution/yuan-yu-linkedhashmapyuan-ma-by-jeromememory/)



> 更新: 2024-01-02 20:13:37  
原文: [https://www.yuque.com/vip6688/neho4x/vocz15dpy58maua9](https://www.yuque.com/vip6688/neho4x/vocz15dpy58maua9)
>



> 更新: 2024-11-25 09:20:02  
> 原文: <https://www.yuque.com/neumx/laxg2e/6703944b58e72ed1c6969597b5a51c62>