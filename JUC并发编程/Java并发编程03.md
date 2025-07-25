# Java并发编程03

# Java并发编程03
## <font style="color:#000000;">ThreadLocal</font>
### <font style="color:#000000;">1.1 ThreadLocal 有什么用？</font>
<font style="color:#000000;">通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。</font>**<font style="color:#000000;">如果想实现每一个线程都有自己的专属本地变量该如何解决呢？</font>**

<font style="color:#000000;">JDK 中自带的ThreadLocal类正是为了解决这样的问题。</font>**<font style="color:#000000;background-color:#d9dffc;">ThreadLocal类主要解决的就是让每个线程绑定自己的值，可以将ThreadLocal类形象的比喻成存放数据的盒子，盒子中可以存储每个线程的私有数据。</font>**

<font style="color:#000000;">如果你创建了一个</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">变量，那么访问这个变量的每个线程都会有这个变量的本地副本，这也是</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">变量名的由来。他们可以使用</font><font style="color:#000000;">get()</font><font style="color:#000000;">和</font><font style="color:#000000;">set()</font><font style="color:#000000;">方法来获取默认值或将其值更改为当前线程所存的副本的值，从而避免了线程安全问题。</font>

<font style="color:#000000;">再举个简单的例子：两个人去宝屋收集宝物，这两个共用一个袋子的话肯定会产生争执，但是给他们两个人每个人分配一个袋子的话就不会出现这样的问题。如果把这两个人比作线程的话，那么 ThreadLocal 就是用来避免这两个线程竞争的。</font>

### <font style="color:#000000;">1.2 如何使用 ThreadLocal？</font>
<font style="color:#000000;">相信看了上面的解释，大家已经搞懂</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">类是个什么东西了。下面简单演示一下如何在项目中实际使用</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">。</font>

```java
import java.text.SimpleDateFormat;
import java.util.Random;

public class ThreadLocalExample implements Runnable{

    // SimpleDateFormat 不是线程安全的，所以每个线程都要有自己独立的副本
    private static final ThreadLocal<SimpleDateFormat> formatter = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyyMMdd HHmm"));

    public static void main(String[] args) throws InterruptedException {
        ThreadLocalExample obj = new ThreadLocalExample();
        for(int i=0 ; i<10; i++){
            Thread t = new Thread(obj, ""+i);
            Thread.sleep(new Random().nextInt(1000));
            t.start();
        }
    }

    @Override
    public void run() {
        System.out.println("Thread Name= "+Thread.currentThread().getName()+" default Formatter = "+formatter.get().toPattern());
        try {
            Thread.sleep(new Random().nextInt(1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //formatter pattern is changed here by thread, but it won't reflect to other threads
        formatter.set(new SimpleDateFormat());

        System.out.println("Thread Name= "+Thread.currentThread().getName()+" formatter = "+formatter.get().toPattern());
    }

}
```

<font style="color:#000000;">输出结果 :</font>

```java
Thread Name= 0 default Formatter = yyyyMMdd HHmm
Thread Name= 0 formatter = yy-M-d ah:mm
Thread Name= 1 default Formatter = yyyyMMdd HHmm
Thread Name= 2 default Formatter = yyyyMMdd HHmm
Thread Name= 1 formatter = yy-M-d ah:mm
Thread Name= 3 default Formatter = yyyyMMdd HHmm
Thread Name= 2 formatter = yy-M-d ah:mm
Thread Name= 4 default Formatter = yyyyMMdd HHmm
Thread Name= 3 formatter = yy-M-d ah:mm
Thread Name= 4 formatter = yy-M-d ah:mm
Thread Name= 5 default Formatter = yyyyMMdd HHmm
Thread Name= 5 formatter = yy-M-d ah:mm
Thread Name= 6 default Formatter = yyyyMMdd HHmm
Thread Name= 6 formatter = yy-M-d ah:mm
Thread Name= 7 default Formatter = yyyyMMdd HHmm
Thread Name= 7 formatter = yy-M-d ah:mm
Thread Name= 8 default Formatter = yyyyMMdd HHmm
Thread Name= 9 default Formatter = yyyyMMdd HHmm
Thread Name= 8 formatter = yy-M-d ah:mm
Thread Name= 9 formatter = yy-M-d ah:mm
```

<font style="color:#000000;">从输出中可以看出，虽然</font><font style="color:#000000;">Thread-0</font><font style="color:#000000;">已经改变了</font><font style="color:#000000;">formatter</font><font style="color:#000000;">的值，但</font><font style="color:#000000;">Thread-1</font><font style="color:#000000;">默认格式化值与初始化值相同，其他线程也一样。</font>

<font style="color:#000000;">上面有一段代码用到了创建</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">变量的那段代码用到了 Java8 的知识，它等于下面这段代码，如果你写了下面这段代码的话，IDEA 会提示你转换为 Java8 的格式(IDEA 真的不错！)。因为 ThreadLocal 类在 Java 8 中扩展，使用一个新的方法</font><font style="color:#000000;">withInitial()</font><font style="color:#000000;">，将 Supplier 功能接口作为参数。</font>

```java
private static final ThreadLocal<SimpleDateFormat> formatter = new ThreadLocal<SimpleDateFormat>(){
    @Override
    protected SimpleDateFormat initialValue(){
        return new SimpleDateFormat("yyyyMMdd HHmm");
    }
};
```

### <font style="color:#000000;">1.3 ThreadLocal 原理了解吗？</font>
<font style="color:#000000;">从</font><font style="color:#000000;">Thread</font><font style="color:#000000;">类源代码入手。</font>

```java
public class Thread implements Runnable {
    //......
    //与此线程有关的ThreadLocal值。由ThreadLocal类维护
    ThreadLocal.ThreadLocalMap threadLocals = null;

    //与此线程有关的InheritableThreadLocal值。由InheritableThreadLocal类维护
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    //......
}
```

<font style="color:#000000;">从上面</font><font style="color:#000000;">Thread</font><font style="color:#000000;">类 源代码可以看出</font><font style="color:#000000;">Thread</font><font style="color:#000000;">类中有一个</font><font style="color:#000000;">threadLocals</font><font style="color:#000000;">和 一个</font><font style="color:#000000;">inheritableThreadLocals</font><font style="color:#000000;">变量，它们都是</font><font style="color:#000000;">ThreadLocalMap</font><font style="color:#000000;">类型的变量,我们可以把</font><font style="color:#000000;">ThreadLocalMap</font><font style="color:#000000;">理解为</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">类实现的定制化的</font><font style="color:#000000;">HashMap</font><font style="color:#000000;">。默认情况下这两个变量都是 null，只有当前线程调用</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">类的</font><font style="color:#000000;">set</font><font style="color:#000000;">或</font><font style="color:#000000;">get</font><font style="color:#000000;">方法时才创建它们，实际上调用这两个方法的时候，我们调用的是</font><font style="color:#000000;">ThreadLocalMap</font><font style="color:#000000;">类对应的</font><font style="color:#000000;">get()</font><font style="color:#000000;">、</font><font style="color:#000000;">set()</font><font style="color:#000000;">方法。</font>

<font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">类的</font><font style="color:#000000;">set()</font><font style="color:#000000;">方法</font>

```java
public void set(T value) {
//获取当前请求的线程
Thread t = Thread.currentThread();
//取出 Thread 类内部的 threadLocals 变量(哈希表结构)
ThreadLocalMap map = getMap(t);
if (map != null)
    // 将需要存储的值放入到这个哈希表中
    map.set(this, value);
else
    createMap(t, value);
}
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

<font style="color:#000000;">通过上面这些内容，我们足以通过猜测得出结论：</font>**<font style="color:#000000;">最终的变量是放在了当前线程的</font>****<font style="color:#000000;">ThreadLocalMap</font>****<font style="color:#000000;">中，并不是存在</font>****<font style="color:#000000;">ThreadLocal</font>****<font style="color:#000000;">上，</font>****<font style="color:#000000;">ThreadLocal</font>****<font style="color:#000000;">可以理解为只是</font>****<font style="color:#000000;">ThreadLocalMap</font>****<font style="color:#000000;">的封装，传递了变量值。</font>**<font style="color:#000000;">ThrealLocal</font><font style="color:#000000;">类中可以通过</font><font style="color:#000000;">Thread.currentThread()</font><font style="color:#000000;">获取到当前线程对象后，直接通过</font><font style="color:#000000;">getMap(Thread t)</font><font style="color:#000000;">可以访问到该线程的</font><font style="color:#000000;">ThreadLocalMap</font><font style="color:#000000;">对象。</font>

**<font style="color:#000000;">每个</font>****<font style="color:#000000;">Thread</font>****<font style="color:#000000;">中都具备一个</font>****<font style="color:#000000;">ThreadLocalMap</font>****<font style="color:#000000;">，而</font>****<font style="color:#000000;">ThreadLocalMap</font>****<font style="color:#000000;">可以存储以</font>****<font style="color:#000000;">ThreadLocal</font>****<font style="color:#000000;">为 key ，Object 对象为 value 的键值对。</font>**

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    //......
}
```

<font style="color:#000000;">比如我们在同一个线程中声明了两个</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">对象的话，</font><font style="color:#000000;">Thread</font><font style="color:#000000;">内部都是使用仅有的那个</font><font style="color:#000000;">ThreadLocalMap</font><font style="color:#000000;">存放数据的，</font><font style="color:#000000;">ThreadLocalMap</font><font style="color:#000000;">的 key 就是</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">对象，value 就是</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">对象调用</font><font style="color:#000000;">set</font><font style="color:#000000;">方法设置的值。</font>

<font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">数据结构如下图所示：</font>

![1732497478404-211e4eef-b8bc-440c-b772-1c0a3d75657a.png](./img/8BrS57PdqzNqWLK7/1732497478404-211e4eef-b8bc-440c-b772-1c0a3d75657a-718568.png)

<font style="color:#000000;">ThreadLocal 数据结构</font>

<font style="color:#000000;">ThreadLocalMap</font><font style="color:#000000;">是</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">的静态内部类。</font>

![1732497478539-9eb3b0ff-6637-4128-bfeb-134c8bc85ad1.png](./img/8BrS57PdqzNqWLK7/1732497478539-9eb3b0ff-6637-4128-bfeb-134c8bc85ad1-901632.png)

<font style="color:#000000;">ThreadLocal内部类</font>

### <font style="color:#000000;">1.4 ThreadLocal 内存泄露问题是怎么导致的？</font>
<font style="color:#000000;">ThreadLocalMap</font><font style="color:#000000;">中使用的 key 为</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">的弱引用，而 value 是强引用。所以，如果</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。</font>

<font style="color:#000000;">这样一来，</font><font style="color:#000000;">ThreadLocalMap</font><font style="color:#000000;">中就会出现 key 为 null 的 Entry。假如我们不做任何措施的话，value 永远无法被 GC 回收，这个时候就可能会产生内存泄露。</font><font style="color:#000000;">ThreadLocalMap</font><font style="color:#000000;">实现中已经考虑了这种情况，在调用</font><font style="color:#000000;">set()</font><font style="color:#000000;">、</font><font style="color:#000000;">get()</font><font style="color:#000000;">、</font><font style="color:#000000;">remove()</font><font style="color:#000000;">方法的时候，会清理掉 key 为 null 的记录。使用完</font><font style="color:#000000;">ThreadLocal</font><font style="color:#000000;">方法后最好手动调用</font><font style="color:#000000;">remove()</font><font style="color:#000000;">方法</font>

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

**<font style="color:#000000;">弱引用介绍：</font>**

<font style="color:#000000;">如果一个对象只具有弱引用，那就类似于</font>**<font style="color:#000000;">可有可无的生活用品</font>**<font style="color:#000000;">。弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它 所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。</font><font style="color:#000000;">弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java 虚拟机就会把这个弱引用加入到与之关联的引用队列中。</font>

## <font style="color:#000000;">线程池</font>
### <font style="color:#000000;">2.1 什么是线程池?</font>
<font style="color:#000000;">顾名思义，线程池就是管理一系列线程的资源池。当有任务要处理时，直接从线程池中获取线程来处理，处理完之后线程并不会立即被销毁，而是等待下一个任务。</font>

### <font style="color:#000000;">2.2 为什么要用线程池？</font>
<font style="color:#000000;">池化技术想必大家已经屡见不鲜了，线程池、数据库连接池、HTTP 连接池等等都是对这个思想的应用。池化技术的思想主要是为了减少每次获取资源的消耗，提高对资源的利用率。</font>

**<font style="color:#000000;">线程池</font>**<font style="color:#000000;">提供了一种限制和管理资源（包括执行一个任务）的方式。 每个</font>**<font style="color:#000000;">线程池</font>**<font style="color:#000000;">还维护一些基本统计信息，例如已完成任务的数量。</font>

<font style="color:#000000;">这里借用《Java 并发编程的艺术》提到的来说一下</font>**<font style="color:#000000;">使用线程池的好处</font>**<font style="color:#000000;">：</font>

+ **<font style="color:#000000;">降低资源消耗</font>**<font style="color:#000000;">。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。</font>
+ **<font style="color:#000000;">提高响应速度</font>**<font style="color:#000000;">。当任务到达时，任务可以不需要等到线程创建就能立即执行。</font>
+ **<font style="color:#000000;">提高线程的可管理性</font>**<font style="color:#000000;">。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。</font>

### <font style="color:#000000;">2.3 如何创建线程池？</font>
**<font style="color:#000000;">方式一：通过</font>****<font style="color:#000000;">ThreadPoolExecutor</font>****<font style="color:#000000;">构造函数来创建（推荐）。</font>**

![1732497478610-cfcd9513-5deb-4ddc-b140-719633cdbdea.jpeg](./img/8BrS57PdqzNqWLK7/1732497478610-cfcd9513-5deb-4ddc-b140-719633cdbdea-481881.jpeg)

<font style="color:#000000;">通过构造方法实现</font>

**<font style="color:#000000;">方式二：通过</font>****<font style="color:#000000;">Executor</font>****<font style="color:#000000;">框架的工具类</font>****<font style="color:#000000;">Executors</font>****<font style="color:#000000;">来创建。</font>**

<font style="color:#000000;">我们可以创建多种类型的</font><font style="color:#000000;">ThreadPoolExecutor</font><font style="color:#000000;">：</font>

+ **<font style="color:#000000;">FixedThreadPool</font>**<font style="color:#000000;">：</font>**<font style="color:#000000;background-color:#d9dffc;">该方法返回一个固定线程数量的线程池</font>**<font style="color:#000000;">。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。</font>
+ **<font style="color:#000000;">SingleThreadExecutor：</font>************<font style="color:#000000;background-color:#d9dffc;">该方法返回一个只有一个线程的线程池</font>**<font style="color:#000000;">。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。</font>
+ **<font style="color:#000000;">CachedThreadPool：</font>************<font style="color:#000000;background-color:#d9dffc;">该方法返回一个可根据实际情况调整线程数量的线程池</font>**<font style="color:#000000;">。初始大小为 0。当有新任务提交时，如果当前线程池中没有线程可用，它会创建一个新的线程来处理该任务。如果在一段时间内（默认为 60 秒）没有新任务提交，核心线程会超时并被销毁，从而缩小线程池的大小。</font>
+ **<font style="color:#000000;">ScheduledThreadPool</font>**<font style="color:#000000;">：</font>**<font style="color:#000000;background-color:#d9dffc;">该方法返回一个用来在给定的延迟后运行任务或者定期执行任务的线程池</font>**<font style="color:#000000;">。</font>

<font style="color:#000000;">对应</font><font style="color:#000000;">Executors</font><font style="color:#000000;">工具类中的方法如图所示：</font>

![1732497478692-5dd47cdb-fbfd-42b3-bcf8-16fb1d67aeed.png](./img/8BrS57PdqzNqWLK7/1732497478692-5dd47cdb-fbfd-42b3-bcf8-16fb1d67aeed-658182.png)

### <font style="color:#000000;">2.4 为什么不推荐使用内置线程池？</font>
<font style="color:#000000;">在《阿里巴巴 Java 开发手册》“并发处理”这一章节，明确指出</font>**<font style="color:#000000;background-color:#d9dffc;">线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。</font>**

**<font style="color:#000000;">为什么呢？</font>**

<font style="color:#000000;">使用线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源开销，解决资源不足的问题。如果不使用线程池，有可能会造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。</font>

<font style="color:#000000;">另外，《阿里巴巴 Java 开发手册》中</font>**<font style="color:#000000;background-color:#d9dffc;">强制线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor构造函数的方式</font>**<font style="color:#000000;">，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险</font>

<font style="color:#000000;">Executors</font><font style="color:#000000;">返回线程池对象的弊端如下(后文会详细介绍到)：</font>

+ **<font style="color:#000000;">FixedThreadPool</font>****<font style="color:#000000;">和</font>****<font style="color:#000000;">SingleThreadExecutor</font>**<font style="color:#000000;">：使用的是无界的</font><font style="color:#000000;">LinkedBlockingQueue</font><font style="color:#000000;">，任务队列最大长度为</font><font style="color:#000000;">Integer.MAX_VALUE</font><font style="color:#000000;">,可能堆积大量的请求，从而导致 OOM。</font>
+ **<font style="color:#000000;">CachedThreadPool</font>**<font style="color:#000000;">：使用的是同步队列</font><font style="color:#000000;">SynchronousQueue</font><font style="color:#000000;">, 允许创建的线程数量为</font><font style="color:#000000;">Integer.MAX_VALUE</font><font style="color:#000000;">，如果任务数量过多且执行速度较慢，可能会创建大量的线程，从而导致 OOM。</font>
+ **<font style="color:#000000;">ScheduledThreadPool</font>****<font style="color:#000000;">和</font>****<font style="color:#000000;">SingleThreadScheduledExecutor</font>**<font style="color:#000000;">: 使用的无界的延迟阻塞队列</font><font style="color:#000000;">DelayedWorkQueue</font><font style="color:#000000;">，任务队列最大长度为</font><font style="color:#000000;">Integer.MAX_VALUE</font><font style="color:#000000;">,可能堆积大量的请求，从而导致 OOM。</font>

```java
// 无界队列 LinkedBlockingQueue
public static ExecutorService newFixedThreadPool(int nThreads) {

    return new ThreadPoolExecutor(nThreads, nThreads,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());

}

// 无界队列 LinkedBlockingQueue
public static ExecutorService newSingleThreadExecutor() {

    return new FinalizableDelegatedExecutorService (new ThreadPoolExecutor(1, 1,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>()));

}

// 同步队列 SynchronousQueue，没有容量，最大线程数是 Integer.MAX_VALUE`
public static ExecutorService newCachedThreadPool() {

    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,60L, TimeUnit.SECONDS,new SynchronousQueue<Runnable>());

}

// DelayedWorkQueue（延迟阻塞队列）
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```

### <font style="color:#000000;">2.5 线程池常见参数有哪些？如何解释？</font>
```java
/**
     * 用给定的初始参数创建一个新的ThreadPoolExecutor。
     */
public ThreadPoolExecutor(int corePoolSize,//线程池的核心线程数量
                          int maximumPoolSize,//线程池的最大线程数
                          long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
                          TimeUnit unit,//时间单位
                          BlockingQueue<Runnable> workQueue,//任务队列，用来储存等待执行任务的队列
                          ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
                          RejectedExecutionHandler handler//拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
                         ) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

**<font style="color:#000000;">ThreadPoolExecutor</font>************<font style="color:#000000;">3 个最重要的参数：</font>**

+ **<font style="color:#000000;">corePoolSize</font>************<font style="color:#000000;">:</font>**<font style="color:#000000;">任务队列未达到队列容量时，最大可以同时运行的线程数量。</font>
+ **<font style="color:#000000;">maximumPoolSize</font>************<font style="color:#000000;">:</font>**<font style="color:#000000;">任务队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。</font>
+ **<font style="color:#000000;">workQueue</font>************<font style="color:#000000;">:</font>**<font style="color:#000000;">新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。</font>

<font style="color:#000000;">ThreadPoolExecutor</font><font style="color:#000000;">其他常见参数 :</font>

+ **<font style="color:#000000;">keepAliveTime</font>**<font style="color:#000000;">:线程池中的线程数量大于</font><font style="color:#000000;">corePoolSize</font><font style="color:#000000;">的时候，如果这时没有新的任务提交，多余的空闲线程不会立即销毁，而是会等待，直到等待的时间超过了</font><font style="color:#000000;">keepAliveTime</font><font style="color:#000000;">才会被回收销毁，线程池回收线程时，会对核心线程和非核心线程一视同仁，直到线程池中线程的数量等于</font><font style="color:#000000;">corePoolSize</font><font style="color:#000000;">，回收过程才会停止。</font>
+ **<font style="color:#000000;">unit</font>**<font style="color:#000000;">:</font><font style="color:#000000;">keepAliveTime</font><font style="color:#000000;">参数的时间单位。</font>
+ **<font style="color:#000000;">threadFactory</font>**<font style="color:#000000;">:executor 创建新线程的时候会用到。</font>
+ **<font style="color:#000000;">handler</font>**<font style="color:#000000;">:饱和策略。关于饱和策略下面单独介绍一下。</font>

<font style="color:#000000;">下面这张图可以加深你对线程池中各个参数的相互关系的理解（图片来源：《Java 性能调优实战》）：</font>

![1732497478809-ffdb0829-4b26-4cad-aa54-270661a277c9.jpeg](./img/8BrS57PdqzNqWLK7/1732497478809-ffdb0829-4b26-4cad-aa54-270661a277c9-557286.jpeg)

<font style="color:#000000;">线程池各个参数的关系</font>

### <font style="color:#000000;">2.6 线程池的饱和策略有哪些？</font>
<font style="color:#000000;">如果当前同时运行的线程数量达到最大线程数量并且队列也已经被放满了任务时，</font><font style="color:#000000;">ThreadPoolTaskExecutor</font><font style="color:#000000;">定义一些策略:</font>

+ **<font style="color:#000000;">ThreadPoolExecutor.AbortPolicy</font>************<font style="color:#000000;">：</font>**<font style="color:#000000;">抛出</font><font style="color:#000000;">RejectedExecutionException</font><font style="color:#000000;">来拒绝新任务的处理。</font>
+ **<font style="color:#000000;">ThreadPoolExecutor.CallerRunsPolicy</font>************<font style="color:#000000;">：</font>**<font style="color:#000000;">调用执行自己的线程运行任务，也就是直接在调用</font><font style="color:#000000;">execute</font><font style="color:#000000;">方法的线程中运行(</font><font style="color:#000000;">run</font><font style="color:#000000;">)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。</font>
+ **<font style="color:#000000;">ThreadPoolExecutor.DiscardPolicy</font>************<font style="color:#000000;">：</font>**<font style="color:#000000;">不处理新任务，直接丢弃掉。</font>
+ **<font style="color:#000000;">ThreadPoolExecutor.DiscardOldestPolicy</font>************<font style="color:#000000;">：</font>**<font style="color:#000000;">此策略将丢弃最早的未处理的任务请求。</font>

<font style="color:#000000;">举个例子：Spring 通过</font><font style="color:#000000;">ThreadPoolTaskExecutor</font><font style="color:#000000;">或者我们直接通过</font><font style="color:#000000;">ThreadPoolExecutor</font><font style="color:#000000;">的构造函数创建线程池的时候，当我们不指定</font><font style="color:#000000;">RejectedExecutionHandler</font><font style="color:#000000;">饱和策略来配置线程池的时候，默认使用的是</font><font style="color:#000000;">AbortPolicy</font><font style="color:#000000;">。在这种饱和策略下，如果队列满了，</font><font style="color:#000000;">ThreadPoolExecutor</font><font style="color:#000000;">将抛出</font><font style="color:#000000;">RejectedExecutionException</font><font style="color:#000000;">异常来拒绝新来的任务 ，这代表你将丢失对这个任务的处理。如果不想丢弃任务的话，可以使用</font><font style="color:#000000;">CallerRunsPolicy</font><font style="color:#000000;">。</font><font style="color:#000000;">CallerRunsPolicy</font><font style="color:#000000;">和其他的几个策略不同，它既不会抛弃任务，也不会抛出异常，而是将任务回退给调用者，使用调用者的线程来执行任务</font>

```java
public static class CallerRunsPolicy implements RejectedExecutionHandler {

        public CallerRunsPolicy() { }

        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                // 直接主线程执行，而不是线程池中的线程执行
                r.run();
            }
        }
    }
```

### <font style="color:#000000;">2.7 线程池常用的阻塞队列有哪些？</font>
<font style="color:#000000;">新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。</font>

<font style="color:#000000;">不同的线程池会选用不同的阻塞队列，我们可以结合内置线程池来分析。</font>

+ <font style="color:#000000;">容量为</font><font style="color:#000000;">Integer.MAX_VALUE</font><font style="color:#000000;">的</font><font style="color:#000000;">LinkedBlockingQueue</font><font style="color:#000000;">（无界队列）：</font><font style="color:#000000;">FixedThreadPool</font><font style="color:#000000;">和</font><font style="color:#000000;">SingleThreadExector</font><font style="color:#000000;">。</font><font style="color:#000000;">FixedThreadPool</font><font style="color:#000000;">最多只能创建核心线程数的线程（核心线程数和最大线程数相等），</font><font style="color:#000000;">SingleThreadExector</font><font style="color:#000000;">只能创建一个线程（核心线程数和最大线程数都是 1），二者的任务队列永远不会被放满。</font>
+ <font style="color:#000000;">SynchronousQueue</font><font style="color:#000000;">（同步队列）：</font><font style="color:#000000;">CachedThreadPool</font><font style="color:#000000;">。</font><font style="color:#000000;">SynchronousQueue</font><font style="color:#000000;">没有容量，不存储元素，目的是保证对于提交的任务，如果有空闲线程，则使用空闲线程来处理；否则新建一个线程来处理任务。也就是说，</font><font style="color:#000000;">CachedThreadPool</font><font style="color:#000000;">的最大线程数是</font><font style="color:#000000;">Integer.MAX_VALUE</font><font style="color:#000000;">，可以理解为线程数是可以无限扩展的，可能会创建大量线程，从而导致 OOM。</font>
+ <font style="color:#000000;">DelayedWorkQueue</font><font style="color:#000000;">（延迟阻塞队列）：</font><font style="color:#000000;">ScheduledThreadPool</font><font style="color:#000000;">和</font><font style="color:#000000;">SingleThreadScheduledExecutor</font><font style="color:#000000;">。</font><font style="color:#000000;">DelayedWorkQueue</font><font style="color:#000000;">的内部元素并不是按照放入的时间排序，而是会按照延迟的时间长短对任务进行排序，内部采用的是“堆”的数据结构，可以保证每次出队的任务都是当前队列中执行时间最靠前的。</font><font style="color:#000000;">DelayedWorkQueue</font><font style="color:#000000;">添加元素满了之后会自动扩容原来容量的 1/2，即永远不会阻塞，最大扩容可达</font><font style="color:#000000;">Integer.MAX_VALUE</font><font style="color:#000000;">，所以最多只能创建核心线程数的线程。</font>

### <font style="color:#000000;">2.8 线程池处理任务的流程了解吗？</font>
![1732497478917-c6269f21-49e9-4b60-8e46-e278d17738fa.png](./img/8BrS57PdqzNqWLK7/1732497478917-c6269f21-49e9-4b60-8e46-e278d17738fa-062483.png)

<font style="color:#000000;">图解线程池实现原理</font>

1. <font style="color:#000000;">如果当前运行的线程数小于核心线程数，那么就会新建一个线程来执行任务。</font>
2. <font style="color:#000000;">如果当前运行的线程数等于或大于核心线程数，但是小于最大线程数，那么就把该任务放入到任务队列里等待执行。</font>
3. <font style="color:#000000;">如果向任务队列投放任务失败（任务队列已经满了），但是当前运行的线程数是小于最大线程数的，就新建一个线程来执行任务。</font>
4. <font style="color:#000000;">如果当前运行的线程数已经等同于最大线程数了，新建线程将会使当前运行的线程超出最大线程数，那么当前任务会被拒绝，饱和策略会调用</font><font style="color:#000000;">RejectedExecutionHandler.rejectedExecution()</font><font style="color:#000000;">方法。</font>

### <font style="color:#000000;">2.9 如何给线程池命名？</font>
<font style="color:#000000;">初始化线程池的时候需要显示命名（设置线程池名称前缀），有利于定位问题。</font>

<font style="color:#000000;">默认情况下创建的线程名字类似</font><font style="color:#000000;">pool-1-thread-n</font><font style="color:#000000;">这样的，没有业务含义，不利于我们定位问题。</font>

<font style="color:#000000;">给线程池里的线程命名通常有下面两种方式：</font>

**<font style="color:#000000;">1、利用 guava 的</font>************<font style="color:#000000;">ThreadFactoryBuilder</font>**

```java
ThreadFactory threadFactory = new ThreadFactoryBuilder()
                        .setNameFormat(threadNamePrefix + "-%d")
                        .setDaemon(true).build();
ExecutorService threadPool = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, TimeUnit.MINUTES, workQueue, threadFactory);
```

**<font style="color:#000000;">2、自己实现</font>****<font style="color:#000000;">ThreadFactory</font>****<font style="color:#000000;">。</font>**

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicInteger;
/**
 * 线程工厂，它设置线程名称，有利于我们定位问题。
 */
public final class NamingThreadFactory implements ThreadFactory {

    private final AtomicInteger threadNum = new AtomicInteger();
    private final ThreadFactory delegate;
    private final String name;

    /**
     * 创建一个带名字的线程池生产工厂
     */
    public NamingThreadFactory(ThreadFactory delegate, String name) {
        this.delegate = delegate;
        this.name = name; // TODO consider uniquifying this
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread t = delegate.newThread(r);
        t.setName(name + " [#" + threadNum.incrementAndGet() + "]");
        return t;
    }

}
```

### <font style="color:#000000;">2.10 如何设定线程池的大小？</font>
<font style="color:#000000;">很多人甚至可能都会觉得把线程池配置过大一点比较好！我觉得这明显是有问题的。就拿我们生活中非常常见的一例子来说：</font>**<font style="color:#000000;">并不是人多就能把事情做好，增加了沟通交流成本。你本来一件事情只需要 3 个人做，你硬是拉来了 6 个人，会提升做事效率嘛？我想并不会。</font>**<font style="color:#000000;">线程数量过多的影响也是和我们分配多少人做事情一样，对于多线程这个场景来说主要是增加了</font>**<font style="color:#000000;">上下文切换</font>**<font style="color:#000000;">成本。不清楚什么是上下文切换的话，可以看我下面的介绍。</font>

<font style="color:#000000;">上下文切换：</font><font style="color:#000000;">多线程编程中一般线程的个数都大于 CPU 核心的个数，而一个 CPU 核心在任意时刻只能被一个线程使用，为了让这些线程都能得到有效执行，CPU 采取的策略是为每个线程分配时间片并轮转的形式。当一个线程的时间片用完的时候就会重新处于就绪状态让给其他线程使用，这个过程就属于一次上下文切换。概括来说就是：当前任务在执行完 CPU 时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换回这个任务时，可以再加载这个任务的状态。</font>**<font style="color:#000000;">任务从保存到再加载的过程就是一次上下文切换</font>**<font style="color:#000000;">。</font><font style="color:#000000;">上下文切换通常是计算密集型的。也就是说，它需要相当可观的处理器时间，在每秒几十上百次的切换中，每次切换都需要纳秒量级的时间。所以，上下文切换对系统来说意味着消耗大量的 CPU 时间，事实上，可能是操作系统中时间消耗最大的操作。</font><font style="color:#000000;">Linux 相比与其他操作系统（包括其他类 Unix 系统）有很多的优点，其中有一项就是，其上下文切换和模式切换的时间消耗非常少。</font>

<font style="color:#000000;">类比于实现世界中的人类通过合作做某件事情，我们可以肯定的一点是线程池大小设置过大或者过小都会有问题，合适的才是最好。</font>

+ <font style="color:#000000;">如果我们设置的线程池数量太小的话，如果同一时间有大量任务/请求需要处理，可能会导致大量的请求/任务在任务队列中排队等待执行，甚至会出现任务队列满了之后任务/请求无法处理的情况，或者大量任务堆积在任务队列导致 OOM。这样很明显是有问题的，CPU 根本没有得到充分利用。</font>
+ <font style="color:#000000;">如果我们设置线程数量太大，大量线程可能会同时在争取 CPU 资源，这样会导致大量的上下文切换，从而增加线程的执行时间，影响了整体执行效率。</font>

<font style="color:#000000;">有一个简单并且适用面比较广的公式：</font>

+ **<font style="color:#000000;">CPU 密集型任务(N+1)：</font>**<font style="color:#000000;">这种任务消耗的主要是 CPU 资源，可以将线程数设置为 N（CPU 核心数）+1。比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。</font>
+ **<font style="color:#000000;">I/O 密集型任务(2N)：</font>**<font style="color:#000000;">这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 2N。</font>

**<font style="color:#000000;">如何判断是 CPU 密集任务还是 IO 密集任务？</font>**

<font style="color:#000000;">CPU 密集型简单理解就是利用 CPU 计算能力的任务比如你在内存中对大量数据进行排序。但凡涉及到网络读取，文件读取这类都是 IO 密集型，这类任务的特点是 CPU 计算耗费时间相比于等待 IO 操作完成的时间来说很少，大部分时间都花在了等待 IO 操作完成上。</font>

<font style="color:#000000;">🌈</font><font style="color:#000000;"> 拓展一下（参见：</font>[<font style="color:#000000;">issue#1737</font>](https://github.com/Snailclimb/JavaGuide/issues/1737)[](https://github.com/Snailclimb/JavaGuide/issues/1737)<font style="color:#000000;">）：</font><font style="color:#000000;">线程数更严谨的计算的方法应该是：</font><font style="color:#000000;">最佳线程数 = N（CPU 核心数）∗（1+WT（线程等待时间）/ST（线程计算时间））</font><font style="color:#000000;">，其中</font><font style="color:#000000;">WT（线程等待时间）=线程运行总时间 - ST（线程计算时间）</font><font style="color:#000000;">。</font><font style="color:#000000;">线程等待时间所占比例越高，需要越多线程。线程计算时间所占比例越高，需要越少线程。</font><font style="color:#000000;">我们可以通过 JDK 自带的工具 VisualVM 来查看</font><font style="color:#000000;">WT/ST</font><font style="color:#000000;">比例。</font><font style="color:#000000;">CPU 密集型任务的</font><font style="color:#000000;">WT/ST</font><font style="color:#000000;">接近或者等于 0，因此， 线程数可以设置为 N（CPU 核心数）∗（1+0）= N，和我们上面说的 N（CPU 核心数）+1 差不多。</font><font style="color:#000000;">IO 密集型任务下，几乎全是线程等待时间，从理论上来说，你就可以将线程数设置为 2N（按道理来说，WT/ST 的结果应该比较大，这里选择 2N 的原因应该是为了避免创建过多线程吧）。</font>

<font style="color:#000000;">公示也只是参考，具体还是要根据项目实际线上运行情况来动态调整。我在后面介绍的美团的线程池参数动态配置这种方案就非常不错，很实用！</font>

### <font style="color:#000000;">2.11 如何动态修改线程池的参数？</font>
<font style="color:#000000;">美团技术团队在</font>[<font style="color:#000000;">《Java 线程池实现原理及其在美团业务中的实践》</font>](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)[](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)<font style="color:#000000;">这篇文章中介绍到对线程池参数实现可自定义配置的思路和方法。</font>

<font style="color:#000000;">美团技术团队的思路是主要对线程池的核心参数实现自定义可配置。这三个核心参数是：</font>

+ **<font style="color:#000000;">corePoolSize</font>************<font style="color:#000000;">:</font>**<font style="color:#000000;">核心线程数线程数定义了最小可以同时运行的线程数量。</font>
+ **<font style="color:#000000;">maximumPoolSize</font>************<font style="color:#000000;">:</font>**<font style="color:#000000;">当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。</font>
+ **<font style="color:#000000;">workQueue</font>************<font style="color:#000000;">:</font>**<font style="color:#000000;">当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。</font>

**<font style="color:#000000;">为什么是这三个参数？</font>**

[Java 线程池详解](https://www.yuque.com/vip6688/neho4x/sw3wc1so1855wcmi)<font style="color:#000000;">这篇文章中就说过这三个参数是ThreadPoolExecutor最重要的参数，它们基本决定了线程池对于任务的处理策略。</font>

**<font style="color:#000000;">如何支持参数动态配置？</font>**<font style="color:#000000;">且看</font><font style="color:#000000;">ThreadPoolExecutor</font><font style="color:#000000;">提供的下面这些方法。</font>

![1732497478986-c55f9e6f-884b-4845-8ec2-5d645023d303.png](./img/8BrS57PdqzNqWLK7/1732497478986-c55f9e6f-884b-4845-8ec2-5d645023d303-646051.png)

<font style="color:#000000;">格外需要注意的是</font><font style="color:#000000;">corePoolSize</font><font style="color:#000000;">， 程序运行期间的时候，我们调用</font><font style="color:#000000;">setCorePoolSize（）</font><font style="color:#000000;">这个方法的话，线程池会首先判断当前工作线程数是否大于</font><font style="color:#000000;">corePoolSize</font><font style="color:#000000;">，如果大于的话就会回收工作线程。</font>

<font style="color:#000000;">另外，你也看到了上面并没有动态指定队列长度的方法，美团的方式是自定义了一个叫做</font><font style="color:#000000;">ResizableCapacityLinkedBlockIngQueue</font><font style="color:#000000;">的队列（主要就是把</font><font style="color:#000000;">LinkedBlockingQueue</font><font style="color:#000000;">的 capacity 字段的 final 关键字修饰给去掉了，让它变为可变的）。</font>

<font style="color:#000000;">最终实现的可动态修改线程池参数效果如下。</font><font style="color:#000000;">👏👏👏</font>

![1732497479080-7d230ee0-3714-44c5-8a56-b53bba846170.png](./img/8BrS57PdqzNqWLK7/1732497479080-7d230ee0-3714-44c5-8a56-b53bba846170-302443.png)

<font style="color:#000000;">动态配置线程池参数最终效果</font>

<font style="color:#000000;">还没看够？推荐 why 神的</font>[<font style="color:#000000;">如何设置线程池参数？美团给出了一个让面试官虎躯一震的回答。</font>](https://mp.weixin.qq.com/s/9HLuPcoWmTqAeFKa1kj-_A)[](https://mp.weixin.qq.com/s/9HLuPcoWmTqAeFKa1kj-_A)<font style="color:#000000;">这篇文章，深度剖析，很不错哦！</font>

<font style="color:#000000;">如果我们的项目也想要实现这种效果的话，可以借助现成的开源项目：</font>

+ [<font style="color:#000000;">Hippo4j</font>](https://github.com/opengoofy/hippo4j)[](https://github.com/opengoofy/hippo4j)<font style="color:#000000;">：异步线程池框架，支持线程池动态变更&监控&报警，无需修改代码轻松引入。支持多种使用模式，轻松引入，致力于提高系统运行保障能力。</font>
+ [<font style="color:#000000;">Dynamic TP</font>](https://github.com/dromara/dynamic-tp)[](https://github.com/dromara/dynamic-tp)<font style="color:#000000;">：轻量级动态线程池，内置监控告警功能，集成三方中间件线程池管理，基于主流配置中心（已支持 Nacos、Apollo，Zookeeper、Consul、Etcd，可通过 SPI 自定义实现）。</font>

### <font style="color:#000000;">2.12 如何设计一个能够根据任务的优先级来执行的线程池？</font>
<font style="color:#000000;">这是一个常见的面试问题，本质其实还是在考察求职者对于线程池以及阻塞队列的掌握。</font>

<font style="color:#000000;">我们上面也提到了，不同的线程池会选用不同的阻塞队列作为任务队列，比如</font><font style="color:#000000;">FixedThreadPool</font><font style="color:#000000;">使用的是</font><font style="color:#000000;">LinkedBlockingQueue</font><font style="color:#000000;">（无界队列），由于队列永远不会被放满，因此</font><font style="color:#000000;">FixedThreadPool</font><font style="color:#000000;">最多只能创建核心线程数的线程。</font>

<font style="color:#000000;">假如我们需要实现一个优先级任务线程池的话，那可以考虑使用</font><font style="color:#000000;">PriorityBlockingQueue</font><font style="color:#000000;">（优先级阻塞队列）作为任务队列（</font><font style="color:#000000;">ThreadPoolExecutor</font><font style="color:#000000;">的构造函数有一个</font><font style="color:#000000;">workQueue</font><font style="color:#000000;">参数可以传入任务队列）。</font>

![1732497479167-90d5417c-c53c-4b99-a3a2-aadf5af4b6f2.jpeg](./img/8BrS57PdqzNqWLK7/1732497479167-90d5417c-c53c-4b99-a3a2-aadf5af4b6f2-759957.jpeg)

<font style="color:#000000;">ThreadPoolExecutor构造函数</font>

<font style="color:#000000;">PriorityBlockingQueue</font><font style="color:#000000;">是一个支持优先级的无界阻塞队列，可以看作是线程安全的</font><font style="color:#000000;">PriorityQueue</font><font style="color:#000000;">，两者底层都是使用小顶堆形式的二叉堆，即值最小的元素优先出队。不过，</font><font style="color:#000000;">PriorityQueue</font><font style="color:#000000;">不支持阻塞操作。</font>

<font style="color:#000000;">要想让</font><font style="color:#000000;">PriorityBlockingQueue</font><font style="color:#000000;">实现对任务的排序，传入其中的任务必须是具备排序能力的，方式有两种：</font>

1. <font style="color:#000000;">提交到线程池的任务实现</font><font style="color:#000000;">Comparable</font><font style="color:#000000;">接口，并重写</font><font style="color:#000000;">compareTo</font><font style="color:#000000;">方法来指定任务之间的优先级比较规则。</font>
2. <font style="color:#000000;">创建</font><font style="color:#000000;">PriorityBlockingQueue</font><font style="color:#000000;">时传入一个</font><font style="color:#000000;">Comparator</font><font style="color:#000000;">对象来指定任务之间的排序规则(推荐)。</font>

<font style="color:#000000;">不过，这存在一些风险和问题，比如：</font>

+ <font style="color:#000000;">PriorityBlockingQueue</font><font style="color:#000000;">是无界的，可能堆积大量的请求，从而导致 OOM。</font>
+ <font style="color:#000000;">可能会导致饥饿问题，即低优先级的任务长时间得不到执行。</font>
+ <font style="color:#000000;">由于需要对队列中的元素进行排序操作以及保证线程安全（并发控制采用的是可重入锁</font><font style="color:#000000;">ReentrantLock</font><font style="color:#000000;">），因此会降低性能。</font>

<font style="color:#000000;">对于 OOM 这个问题的解决比较简单粗暴，就是继承</font><font style="color:#000000;">PriorityBlockingQueue</font><font style="color:#000000;">并重写一下</font><font style="color:#000000;">offer</font><font style="color:#000000;">方法(入队)的逻辑，当插入的元素数量超过指定值就返回 false 。</font>

<font style="color:#000000;">饥饿问题这个可以通过优化设计来解决（比较麻烦），比如等待时间过长的任务会被移除并重新添加到队列中，但是优先级会被提升。</font>

<font style="color:#000000;">对于性能方面的影响，是没办法避免的，毕竟需要对任务进行排序操作。并且，对于大部分业务场景来说，这点性能影响是可以接受的。</font>

## <font style="color:#000000;">Future</font>
### <font style="color:#000000;">3.1 Future 类有什么用？</font>
<font style="color:#000000;">Future</font><font style="color:#000000;">类是异步思想的典型运用，主要用在一些需要执行耗时任务的场景，避免程序一直原地等待耗时任务执行完成，执行效率太低。具体来说是这样的：当我们执行某一耗时的任务时，可以将这个耗时任务交给一个子线程去异步执行，同时我们可以干点其他事情，不用傻傻等待耗时任务执行完成。等我们的事情干完后，我们再通过</font><font style="color:#000000;">Future</font><font style="color:#000000;">类获取到耗时任务的执行结果。这样一来，程序的执行效率就明显提高了。</font>

<font style="color:#000000;">这其实就是多线程中经典的</font>**<font style="color:#000000;">Future 模式</font>**<font style="color:#000000;">，你可以将其看作是一种设计模式，核心思想是异步调用，主要用在多线程领域，并非 Java 语言独有。</font>

<font style="color:#000000;">在 Java 中，</font><font style="color:#000000;">Future</font><font style="color:#000000;">类只是一个泛型接口，位于</font><font style="color:#000000;">java.util.concurrent</font><font style="color:#000000;">包下，其中定义了 5 个方法，主要包括下面这 4 个功能：</font>

+ <font style="color:#000000;">取消任务；</font>
+ <font style="color:#000000;">判断任务是否被取消;</font>
+ <font style="color:#000000;">判断任务是否已经执行完成;</font>
+ <font style="color:#000000;">获取任务执行结果。</font>

```java
// V 代表了Future执行的任务返回值的类型
public interface Future<V> {
    // 取消任务执行
    // 成功取消返回 true，否则返回 false
    boolean cancel(boolean mayInterruptIfRunning);
    // 判断任务是否被取消
    boolean isCancelled();
    // 判断任务是否已经执行完成
    boolean isDone();
    // 获取任务执行结果
    V get() throws InterruptedException, ExecutionException;
    // 指定时间内没有返回计算结果就抛出 TimeOutException 异常
    V get(long timeout, TimeUnit unit)

        throws InterruptedException, ExecutionException, TimeoutExceptio

}
```

<font style="color:#000000;">简单理解就是：我有一个任务，提交给了</font><font style="color:#000000;">Future</font><font style="color:#000000;">来处理。任务执行期间我自己可以去做任何想做的事情。并且，在这期间我还可以取消任务以及获取任务的执行状态。一段时间之后，我就可以</font><font style="color:#000000;">Future</font><font style="color:#000000;">那里直接取出任务执行结果。</font>

### <font style="color:#000000;">3.2 Callable 和 Future 有什么关系？</font>
<font style="color:#000000;">我们可以通过</font><font style="color:#000000;">FutureTask</font><font style="color:#000000;">来理解</font><font style="color:#000000;">Callable</font><font style="color:#000000;">和</font><font style="color:#000000;">Future</font><font style="color:#000000;">之间的关系。</font>

<font style="color:#000000;">FutureTask</font><font style="color:#000000;">提供了</font><font style="color:#000000;">Future</font><font style="color:#000000;">接口的基本实现，常用来封装</font><font style="color:#000000;">Callable</font><font style="color:#000000;">和</font><font style="color:#000000;">Runnable</font><font style="color:#000000;">，具有取消任务、查看任务是否执行完成以及获取任务执行结果的方法。</font><font style="color:#000000;">ExecutorService.submit()</font><font style="color:#000000;">方法返回的其实就是</font><font style="color:#000000;">Future</font><font style="color:#000000;">的实现类</font><font style="color:#000000;">FutureTask</font><font style="color:#000000;">。</font>

```java
<T> Future<T> submit(Callable<T> task);
Future<?> submit(Runnable task);
```

<font style="color:#000000;">FutureTask</font><font style="color:#000000;">不光实现了</font><font style="color:#000000;">Future</font><font style="color:#000000;">接口，还实现了</font><font style="color:#000000;">Runnable</font><font style="color:#000000;">接口，因此可以作为任务直接被线程执行。</font>

![1732497479259-ba49f6ff-0ca0-466d-8818-124df735603a.jpeg](./img/8BrS57PdqzNqWLK7/1732497479259-ba49f6ff-0ca0-466d-8818-124df735603a-437225.jpeg)

<font style="color:#000000;">FutureTask</font><font style="color:#000000;">有两个构造函数，可传入</font><font style="color:#000000;">Callable</font><font style="color:#000000;">或者</font><font style="color:#000000;">Runnable</font><font style="color:#000000;">对象。实际上，传入</font><font style="color:#000000;">Runnable</font><font style="color:#000000;">对象也会在方法内部转换为</font><font style="color:#000000;">Callable</font><font style="color:#000000;">对象。</font>

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;
}
public FutureTask(Runnable runnable, V result) {
    // 通过适配器RunnableAdapter来将Runnable对象runnable转换成Callable对象
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;
}
```

<font style="color:#000000;">FutureTask</font><font style="color:#000000;">相当于对</font><font style="color:#000000;">Callable</font><font style="color:#000000;">进行了封装，管理着任务执行的情况，存储了</font><font style="color:#000000;">Callable</font><font style="color:#000000;">的</font><font style="color:#000000;">call</font><font style="color:#000000;">方法的任务执行结果。</font>

### <font style="color:#000000;">3.3 CompletableFuture 类有什么用？</font>
<font style="color:#000000;">Future</font><font style="color:#000000;">在实际使用过程中存在一些局限性比如不支持异步任务的编排组合、获取计算结果的</font><font style="color:#000000;">get()</font><font style="color:#000000;">方法为阻塞调用。</font>

<font style="color:#000000;">Java 8 才被引入</font><font style="color:#000000;">CompletableFuture</font><font style="color:#000000;">类可以解决</font><font style="color:#000000;">Future</font><font style="color:#000000;">的这些缺陷。</font><font style="color:#000000;">CompletableFuture</font><font style="color:#000000;">除了提供了更为好用和强大的</font><font style="color:#000000;">Future</font><font style="color:#000000;">特性之外，还提供了函数式编程、异步任务编排组合（可以将多个异步任务串联起来，组成一个完整的链式调用）等能力。</font>

<font style="color:#000000;">下面我们来简单看看</font><font style="color:#000000;">CompletableFuture</font><font style="color:#000000;">类的定义。</font>

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
}
```

<font style="color:#000000;">可以看到，</font><font style="color:#000000;">CompletableFuture</font><font style="color:#000000;">同时实现了</font><font style="color:#000000;">Future</font><font style="color:#000000;">和</font><font style="color:#000000;">CompletionStage</font><font style="color:#000000;">接口。</font>

![1732497479329-8cd9baf4-ecd2-40d1-adc1-7be63a482016.jpeg](./img/8BrS57PdqzNqWLK7/1732497479329-8cd9baf4-ecd2-40d1-adc1-7be63a482016-094731.jpeg)

<font style="color:#000000;">CompletionStage</font><font style="color:#000000;">接口描述了一个异步计算的阶段。很多计算可以分成多个阶段或步骤，此时可以通过它将所有步骤组合起来，形成异步计算的流水线。</font>

<font style="color:#000000;">CompletionStage</font><font style="color:#000000;">接口中的方法比较多，</font><font style="color:#000000;">CompletableFuture</font><font style="color:#000000;">的函数式能力就是这个接口赋予的。从这个接口的方法参数你就可以发现其大量使用了 Java8 引入的函数式编程。</font>

![1732497479498-9c84f030-ddc0-4e5f-bd53-f92a980b4ca8.png](./img/8BrS57PdqzNqWLK7/1732497479498-9c84f030-ddc0-4e5f-bd53-f92a980b4ca8-816972.png)

## <font style="color:#000000;">AQS</font>
### <font style="color:#000000;">4.1 AQS 是什么？</font>
<font style="color:#000000;">AQS 的全称为</font><font style="color:#000000;">AbstractQueuedSynchronizer</font><font style="color:#000000;">，翻译过来的意思就是抽象队列同步器。这个类在</font><font style="color:#000000;">java.util.concurrent.locks</font><font style="color:#000000;">包下面。</font>

![1732497479566-d0f32e63-1963-431b-8468-8879d1ef4456.png](./img/8BrS57PdqzNqWLK7/1732497479566-d0f32e63-1963-431b-8468-8879d1ef4456-381959.png)

<font style="color:#000000;">AQS 就是一个抽象类，主要用来构建锁和同步器。</font>

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {
}
```

<font style="color:#000000;">AQS 为构建锁和同步器提供了一些通用功能的实现，因此，使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的</font><font style="color:#000000;">ReentrantLock</font><font style="color:#000000;">，</font><font style="color:#000000;">Semaphore</font><font style="color:#000000;">，其他的诸如</font><font style="color:#000000;">ReentrantReadWriteLock</font><font style="color:#000000;">，</font><font style="color:#000000;">SynchronousQueue</font><font style="color:#000000;">等等皆是基于 AQS 的。</font>

### <font style="color:#000000;">4.2 AQS 的原理是什么？</font>
<font style="color:#000000;">AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用</font>**<font style="color:#000000;">CLH 队列锁</font>**<font style="color:#000000;">实现的，即将暂时获取不到锁的线程加入到队列中。</font>

<font style="color:#000000;">CLH(Craig,Landin,and Hagersten) 队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配。在 CLH 同步队列中，一个节点表示一个线程，它保存着线程的引用（thread）、 当前节点在队列中的状态（waitStatus）、前驱节点（prev）、后继节点（next）。</font>

<font style="color:#000000;">CLH 队列结构如下图所示：</font>

![1732497479631-c6262fd7-67be-45b2-9d61-586c6c0523c0.png](./img/8BrS57PdqzNqWLK7/1732497479631-c6262fd7-67be-45b2-9d61-586c6c0523c0-528631.png)

<font style="color:#000000;">AQS(</font><font style="color:#000000;">AbstractQueuedSynchronizer</font><font style="color:#000000;">)的核心原理图（图源</font>[<font style="color:#000000;">Java 并发之 AQS 详解</font>](https://www.cnblogs.com/waterystone/p/4920797.html)[](https://www.cnblogs.com/waterystone/p/4920797.html)<font style="color:#000000;">）如下：</font>

![1732497479689-83925531-7268-4ec7-8676-dc262a0d47ac.png](./img/8BrS57PdqzNqWLK7/1732497479689-83925531-7268-4ec7-8676-dc262a0d47ac-081018.png)

<font style="color:#000000;">AQS 使用</font>**<font style="color:#000000;">int 成员变量</font>****<font style="color:#000000;">state</font>****<font style="color:#000000;">表示同步状态</font>**<font style="color:#000000;">，通过内置的</font>**<font style="color:#000000;">线程等待队列</font>**<font style="color:#000000;">来完成获取资源线程的排队工作。</font>

<font style="color:#000000;">state</font><font style="color:#000000;">变量由</font><font style="color:#000000;">volatile</font><font style="color:#000000;">修饰，用于展示当前临界资源的获锁情况。</font>

```java
// 共享变量，使用volatile修饰保证线程可见性
private volatile int state;
```

<font style="color:#000000;">另外，状态信息</font><font style="color:#000000;">state</font><font style="color:#000000;">可以通过</font><font style="color:#000000;">protected</font><font style="color:#000000;">类型的</font><font style="color:#000000;">getState()</font><font style="color:#000000;">、</font><font style="color:#000000;">setState()</font><font style="color:#000000;">和</font><font style="color:#000000;">compareAndSetState()</font><font style="color:#000000;">进行操作。并且，这几个方法都是</font><font style="color:#000000;">final</font><font style="color:#000000;">修饰的，在子类中无法被重写。</font>

```java
//返回同步状态的当前值
protected final int getState() {
     return state;
}
 // 设置同步状态的值
protected final void setState(int newState) {
     state = newState;
}
//原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
protected final boolean compareAndSetState(int expect, int update) {
      return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

<font style="color:#000000;">以</font><font style="color:#000000;">ReentrantLock</font><font style="color:#000000;">为例，</font><font style="color:#000000;">state</font><font style="color:#000000;">初始值为 0，表示未锁定状态。A 线程</font><font style="color:#000000;">lock()</font><font style="color:#000000;">时，会调用</font><font style="color:#000000;">tryAcquire()</font><font style="color:#000000;">独占该锁并将</font><font style="color:#000000;">state+1</font><font style="color:#000000;">。此后，其他线程再</font><font style="color:#000000;">tryAcquire()</font><font style="color:#000000;">时就会失败，直到 A 线程</font><font style="color:#000000;">unlock()</font><font style="color:#000000;">到</font><font style="color:#000000;">state=</font><font style="color:#000000;">0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（</font><font style="color:#000000;">state</font><font style="color:#000000;">会累加），这就是可重入的概念。但要注意，获取多少次就要释放多少次，这样才能保证 state 是能回到零态的。</font>

<font style="color:#000000;">再以</font><font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">以例，任务分为 N 个子线程去执行，</font><font style="color:#000000;">state</font><font style="color:#000000;">也初始化为 N（注意 N 要与线程个数一致）。这 N 个子线程是并行执行的，每个子线程执行完后</font><font style="color:#000000;">countDown()</font><font style="color:#000000;">一次，state 会 CAS(Compare and Swap) 减 1。等到所有子线程都执行完后(即</font><font style="color:#000000;">state=0</font><font style="color:#000000;">)，会</font><font style="color:#000000;">unpark()</font><font style="color:#000000;">主调用线程，然后主调用线程就会从</font><font style="color:#000000;">await()</font><font style="color:#000000;">函数返回，继续后余动作。</font>

### <font style="color:#000000;">4.3 Semaphore 有什么用？</font>
<font style="color:#000000;">synchronized</font><font style="color:#000000;">和</font><font style="color:#000000;">ReentrantLock</font><font style="color:#000000;">都是一次只允许一个线程访问某个资源，而</font><font style="color:#000000;">Semaphore</font><font style="color:#000000;">(信号量)可以用来控制同时访问特定资源的线程数量。</font>

<font style="color:#000000;">Semaphore 的使用简单，我们这里假设有 N(N>5) 个线程来获取</font><font style="color:#000000;">Semaphore</font><font style="color:#000000;">中的共享资源，下面的代码表示同一时刻 N 个线程中只有 5 个线程能获取到共享资源，其他线程都会阻塞，只有获取到共享资源的线程才能执行。等到有线程释放了共享资源，其他阻塞的线程才能获取到。</font>

```java
// 初始共享资源数量
final Semaphore semaphore = new Semaphore(5);
// 获取1个许可
semaphore.acquire();
// 释放1个许可
semaphore.release();
```

<font style="color:#000000;">当初始的资源个数为 1 的时候，</font><font style="color:#000000;">Semaphore</font><font style="color:#000000;">退化为排他锁。</font>

<font style="color:#000000;">Semaphore</font><font style="color:#000000;">有两种模式：。</font>

+ **<font style="color:#000000;">公平模式：</font>**<font style="color:#000000;">调用</font><font style="color:#000000;">acquire()</font><font style="color:#000000;">方法的顺序就是获取许可证的顺序，遵循 FIFO；</font>
+ **<font style="color:#000000;">非公平模式：</font>**<font style="color:#000000;">抢占式的。</font>

<font style="color:#000000;">Semaphore</font><font style="color:#000000;">对应的两个构造方法如下：</font>

```java
public Semaphore(int permits) {
      sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
      sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

**<font style="color:#000000;">这两个构造方法，都必须提供许可的数量，第二个构造方法可以指定是公平模式还是非公平模式，默认非公平模式。</font>**

<font style="color:#000000;">Semaphore</font><font style="color:#000000;">通常用于那些资源有明确访问数量限制的场景比如限流（仅限于单机模式，实际项目中推荐使用 Redis +Lua 来做限流）。</font>

### <font style="color:#000000;">4.4 Semaphore 的原理是什么？</font>
<font style="color:#000000;">Semaphore</font><font style="color:#000000;">是共享锁的一种实现，它默认构造 AQS 的</font><font style="color:#000000;">state</font><font style="color:#000000;">值为</font><font style="color:#000000;">permits</font><font style="color:#000000;">，你可以将</font><font style="color:#000000;">permits</font><font style="color:#000000;">的值理解为许可证的数量，只有拿到许可证的线程才能执行。</font>

<font style="color:#000000;">调用</font><font style="color:#000000;">semaphore.acquire()</font><font style="color:#000000;">，线程尝试获取许可证，如果</font><font style="color:#000000;">state >= 0</font><font style="color:#000000;">的话，则表示可以获取成功。如果获取成功的话，使用 CAS 操作去修改</font><font style="color:#000000;">state</font><font style="color:#000000;">的值</font><font style="color:#000000;">state=state-1</font><font style="color:#000000;">。如果</font><font style="color:#000000;">state<0</font><font style="color:#000000;">的话，则表示许可证数量不足。此时会创建一个 Node 节点加入阻塞队列，挂起当前线程。</font>

```java
/**
 *  获取1个许可证
 */
public void acquire() throws InterruptedException {
      sync.acquireSharedInterruptibly(1);
}
/**
 * 共享模式下获取许可证，获取成功则返回，失败则加入阻塞队列，挂起线程
 */
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
      throw new InterruptedException();
        // 尝试获取许可证，arg为获取许可证个数，当可用许可证数减当前获取的许可证数结果小于0,则创建一个节点加入阻塞队列，挂起当前线程。
    if (tryAcquireShared(arg) < 0)
      doAcquireSharedInterruptibly(arg);
}
```

<font style="color:#000000;">调用</font><font style="color:#000000;">semaphore.release();</font><font style="color:#000000;">，线程尝试释放许可证，并使用 CAS 操作去修改</font><font style="color:#000000;">state</font><font style="color:#000000;">的值</font><font style="color:#000000;">state=state+1</font><font style="color:#000000;">。释放许可证成功之后，同时会唤醒同步队列中的一个线程。被唤醒的线程会重新尝试去修改</font><font style="color:#000000;">state</font><font style="color:#000000;">的值</font><font style="color:#000000;">state=state-1</font><font style="color:#000000;">，如果</font><font style="color:#000000;">state>=0</font><font style="color:#000000;">则获取令牌成功，否则重新进入阻塞队列，挂起线程。</font>

```java
// 释放一个许可证
public void release() {
      sync.releaseShared(1);
}

// 释放共享锁，同时会唤醒同步队列中的一个线程。
public final boolean releaseShared(int arg) {
    //释放共享锁
    if (tryReleaseShared(arg)) {
      //唤醒同步队列中的一个线程
      doReleaseShared();
      return true;
    }
    return false;
}
```

### <font style="color:#000000;">4.5 CountDownLatch 有什么用？</font>
<font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">允许</font><font style="color:#000000;">count</font><font style="color:#000000;">个线程阻塞在一个地方，直至所有线程的任务都执行完毕。</font>

<font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当</font><font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">使用完毕后，它不能再次被使用。</font>

### <font style="color:#000000;">4.6 CountDownLatch 的原理是什么？</font>
<font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">是共享锁的一种实现,它默认构造 AQS 的</font><font style="color:#000000;">state</font><font style="color:#000000;">值为</font><font style="color:#000000;">count</font><font style="color:#000000;">。当线程使用</font><font style="color:#000000;">countDown()</font><font style="color:#000000;">方法时,其实使用了</font><font style="color:#000000;">tryReleaseShared</font><font style="color:#000000;">方法以 CAS 的操作来减少</font><font style="color:#000000;">state</font><font style="color:#000000;">,直至</font><font style="color:#000000;">state</font><font style="color:#000000;">为 0 。当调用</font><font style="color:#000000;">await()</font><font style="color:#000000;">方法的时候，如果</font><font style="color:#000000;">state</font><font style="color:#000000;">不为 0，那就证明任务还没有执行完毕，</font><font style="color:#000000;">await()</font><font style="color:#000000;">方法就会一直阻塞，也就是说</font><font style="color:#000000;">await()</font><font style="color:#000000;">方法之后的语句不会被执行。直到</font><font style="color:#000000;">count</font><font style="color:#000000;">个线程调用了</font><font style="color:#000000;">countDown()</font><font style="color:#000000;">使 state 值被减为 0，或者调用</font><font style="color:#000000;">await()</font><font style="color:#000000;">的线程被中断，该线程才会从阻塞中被唤醒，</font><font style="color:#000000;">await()</font><font style="color:#000000;">方法之后的语句得到执行。</font>

### <font style="color:#000000;">4.7 用过 CountDownLatch 么？什么场景下用的？</font>
<font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">的作用就是 允许 count 个线程阻塞在一个地方，直至所有线程的任务都执行完毕。之前在项目中，有一个使用多线程读取多个文件处理的场景，我用到了</font><font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">。具体场景是下面这样的：</font>

<font style="color:#000000;">我们要读取处理 6 个文件，这 6 个任务都是没有执行顺序依赖的任务，但是我们需要返回给用户的时候将这几个文件的处理的结果进行统计整理。</font>

<font style="color:#000000;">为此我们定义了一个线程池和 count 为 6 的</font><font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">对象 。使用线程池处理读取任务，每一个线程处理完之后就将 count-1，调用</font><font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">对象的</font><font style="color:#000000;">await()</font><font style="color:#000000;">方法，直到所有文件读取完之后，才会接着执行后面的逻辑。</font>

<font style="color:#000000;">伪代码是下面这样的：</font>

```java
public class CountDownLatchExample1 {
    // 处理文件的数量
    private static final int threadCount = 6;

    public static void main(String[] args) throws InterruptedException {
        // 创建一个具有固定线程数量的线程池对象（推荐使用构造方法创建）
        ExecutorService threadPool = Executors.newFixedThreadPool(10);
        final CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            final int threadnum = i;
            threadPool.execute(() -> {
                try {
                    //处理文件的业务操作
                    //......
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    //表示一个文件已经被完成
                    countDownLatch.countDown();
                }

            });
        }
        countDownLatch.await();
        threadPool.shutdown();
        System.out.println("finish");
    }
}
```

**<font style="color:#000000;">有没有可以改进的地方呢？</font>**

<font style="color:#000000;">可以使用</font><font style="color:#000000;">CompletableFuture</font><font style="color:#000000;">类来改进！Java8 的</font><font style="color:#000000;">CompletableFuture</font><font style="color:#000000;">提供了很多对多线程友好的方法，使用它可以很方便地为我们编写多线程程序，什么异步、串行、并行或者等待所有线程执行完任务什么的都非常方便。</font>

```java
CompletableFuture<Void> task1 =
    CompletableFuture.supplyAsync(()->{
        //自定义业务操作
    });
......
CompletableFuture<Void> task6 =
    CompletableFuture.supplyAsync(()->{
    //自定义业务操作
    });
......
CompletableFuture<Void> headerFuture=CompletableFuture.allOf(task1,.....,task6);

try {
    headerFuture.join();
} catch (Exception ex) {
    //......
}
System.out.println("all done. ");
```

<font style="color:#000000;">上面的代码还可以继续优化，当任务过多的时候，把每一个 task 都列出来不太现实，可以考虑通过循环来添加任务。</font>

```java
//文件夹位置
List<String> filePaths = Arrays.asList(...)
// 异步处理所有文件
List<CompletableFuture<String>> fileFutures = filePaths.stream()
    .map(filePath -> doSomeThing(filePath))
    .collect(Collectors.toList());
// 将他们合并起来
CompletableFuture<Void> allFutures = CompletableFuture.allOf(
    fileFutures.toArray(new CompletableFuture[fileFutures.size()])
);
```

### <font style="color:#000000;">4.8 CyclicBarrier 有什么用？</font>
<font style="color:#000000;">CyclicBarrier</font><font style="color:#000000;">和</font><font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">非常类似，它也可以实现线程间的技术等待，但是它的功能比</font><font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">更加复杂和强大。主要应用场景和</font><font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">类似。</font>

<font style="color:#000000;">CountDownLatch</font><font style="color:#000000;">的实现是基于 AQS 的，而</font><font style="color:#000000;">CycliBarrier</font><font style="color:#000000;">是基于</font><font style="color:#000000;">ReentrantLock</font><font style="color:#000000;">(</font><font style="color:#000000;">ReentrantLock</font><font style="color:#000000;">也属于 AQS 同步器)和</font><font style="color:#000000;">Condition</font><font style="color:#000000;">的。</font>

<font style="color:#000000;">CyclicBarrier</font><font style="color:#000000;">的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是：让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。</font>

### <font style="color:#000000;">4.9 CyclicBarrier 的原理是什么？</font>
<font style="color:#000000;">CyclicBarrier</font><font style="color:#000000;">内部通过一个</font><font style="color:#000000;">count</font><font style="color:#000000;">变量作为计数器，</font><font style="color:#000000;">count</font><font style="color:#000000;">的初始值为</font><font style="color:#000000;">parties</font><font style="color:#000000;">属性的初始化值，每当一个线程到了栅栏这里了，那么就将计数器减 1。如果 count 值为 0 了，表示这是这一代最后一个线程到达栅栏，就尝试执行我们构造方法中输入的任务。</font>

```java
//每次拦截的线程数
private final int parties;
//计数器
private int count;
```

<font style="color:#000000;">下面我们结合源码来简单看看。</font>

<font style="color:#000000;">1、</font><font style="color:#000000;">CyclicBarrier</font><font style="color:#000000;">默认的构造方法是</font><font style="color:#000000;">CyclicBarrier(int parties)</font><font style="color:#000000;">，其参数表示屏障拦截的线程数量，每个线程调用</font><font style="color:#000000;">await()</font><font style="color:#000000;">方法告诉</font><font style="color:#000000;">CyclicBarrier</font><font style="color:#000000;">我已经到达了屏障，然后当前线程被阻塞。</font>

```java
public CyclicBarrier(int parties) {
    this(parties, null);
}

public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}
```

<font style="color:#000000;">其中，</font><font style="color:#000000;">parties</font><font style="color:#000000;">就代表了有拦截的线程的数量，当拦截的线程数量达到这个值的时候就打开栅栏，让所有线程通过。</font>

<font style="color:#000000;">2、当调用</font><font style="color:#000000;">CyclicBarrier</font><font style="color:#000000;">对象调用</font><font style="color:#000000;">await()</font><font style="color:#000000;">方法时，实际上调用的是</font><font style="color:#000000;">dowait(false, 0L)</font><font style="color:#000000;">方法。</font><font style="color:#000000;">await()</font><font style="color:#000000;">方法就像树立起一个栅栏的行为一样，将线程挡住了，当拦住的线程数量达到</font><font style="color:#000000;">parties</font><font style="color:#000000;">的值时，栅栏才会打开，线程才得以通过执行。</font>

```java
public int await() throws InterruptedException, BrokenBarrierException {
  try {
        return dowait(false, 0L);
  } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
  }
}
```

<font style="color:#000000;">dowait(false, 0L)</font><font style="color:#000000;">方法源码分析如下：</font>

```java
// 当线程数量或者请求数量达到 count 时 await 之后的方法才会被执行。上面的示例中 count 的值就为 5。
    private int count;
    /**
     * Main barrier code, covering the various policies.
     */
    private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        final ReentrantLock lock = this.lock;
        // 锁住
        lock.lock();
        try {
            final Generation g = generation;

            if (g.broken)
                throw new BrokenBarrierException();

            // 如果线程中断了，抛出异常
            if (Thread.interrupted()) {
                breakBarrier();
                throw new InterruptedException();
            }
            // cout减1
            int index = --count;
            // 当 count 数量减为 0 之后说明最后一个线程已经到达栅栏了，也就是达到了可以执行await 方法之后的条件
            if (index == 0) {  // tripped
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    if (command != null)
                        command.run();
                    ranAction = true;
                    // 将 count 重置为 parties 属性的初始化值
                    // 唤醒之前等待的线程
                    // 下一波执行开始
                    nextGeneration();
                    return 0;
                } finally {
                    if (!ranAction)
                        breakBarrier();
                }
            }

            // loop until tripped, broken, interrupted, or timed out
            for (;;) {
                try {
                    if (!timed)
                        trip.await();
                    else if (nanos > 0L)
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // We're about to finish waiting even if we had not
                        // been interrupted, so this interrupt is deemed to
                        // "belong" to subsequent execution.
                        Thread.currentThread().interrupt();
                    }
                }

                if (g.broken)
                    throw new BrokenBarrierException();

                if (g != generation)
                    return index;

                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            lock.unlock();
        }
    }
```

## <font style="color:#000000;">虚拟线程</font>
<font style="color:#000000;">虚拟线程在 Java 21 正式发布，这是一项重量级的更新。</font>

<font style="color:#000000;">虽然目前面试中问的不多，但还是建议大家去简单了解一下，具体可以阅读这篇文章：</font>[<font style="color:#000000;">虚拟线程极简入门</font>](https://javaguide.cn/java/concurrent/virtual-thread.html)<font style="color:#000000;">。重点搞清楚虚拟线程和平台线程的关系以及虚拟线程的优势即可。</font>

## <font style="color:#000000;">参考</font>
+ <font style="color:#000000;">《深入理解 Java 虚拟机》</font>
+ <font style="color:#000000;">《实战 Java 高并发程序设计》</font>
+ <font style="color:#000000;">带你了解下 SynchronousQueue（并发队列专题）：</font>[<font style="color:#000000;">https://juejin.cn/post/7031196740128768037</font>](https://juejin.cn/post/7031196740128768037)[](https://juejin.cn/post/7031196740128768037)
+ <font style="color:#000000;">阻塞队列 — DelayedWorkQueue 源码分析：</font>[<font style="color:#000000;">https://zhuanlan.zhihu.com/p/310621485</font>](https://zhuanlan.zhihu.com/p/310621485)
+ <font style="color:#000000;">Java 多线程（三）——FutureTask/CompletableFuture：</font>[<font style="color:#000000;">https://www.cnblogs.com/iwehdio/p/14285282.html</font>](https://www.cnblogs.com/iwehdio/p/14285282.html)
+ <font style="color:#000000;">Java 并发之 AQS 详解：</font>[<font style="color:#000000;">https://www.cnblogs.com/waterystone/p/4920797.html</font>](https://www.cnblogs.com/waterystone/p/4920797.html)
+ <font style="color:#000000;">Java 并发包基石-AQS 详解：</font>[<font style="color:#000000;">https://www.cnblogs.com/chengxiao/archive/2017/07/24/7141160.html</font>](https://www.cnblogs.com/chengxiao/archive/2017/07/24/7141160.html)



> 更新: 2024-01-20 01:10:29  
原文: [https://www.yuque.com/vip6688/neho4x/ui1qar66f1o880m6](https://www.yuque.com/vip6688/neho4x/ui1qar66f1o880m6)
>



> 更新: 2024-11-25 09:18:00  
> 原文: <https://www.yuque.com/neumx/laxg2e/0f1a8d100a5404bc9c3287911cf3188a>