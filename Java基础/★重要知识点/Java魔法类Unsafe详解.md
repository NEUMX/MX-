# Java魔法类Unsafe详解

# Java 魔法类 Unsafe 详解
阅读过 JUC 源码的同学，一定会发现很多并发工具类都调用了一个叫做Unsafe的类。

那这个类主要是用来干什么的呢？有什么使用场景呢？这篇文章就带你搞清楚！

## Unsafe 介绍
Unsafe是位于sun.misc包下的一个类，主要提供一些用于执行低级别、不安全操作的方法，如直接访问系统内存资源、自主管理内存资源等，这些方法在提升 Java 运行效率、增强 Java 语言底层资源操作能力方面起到了很大的作用。但由于Unsafe类使 Java 语言拥有了类似 C 语言指针一样操作内存空间的能力，这无疑也增加了程序发生相关指针问题的风险。在程序中过度、不正确使用Unsafe类会使得程序出错的概率变大，使得 Java 这种安全的语言变得不再“安全”，因此对Unsafe的使用一定要慎重。

另外，Unsafe提供的这些功能的实现需要依赖本地方法（Native Method）。你可以将本地方法看作是 Java 中使用其他编程语言编写的方法。本地方法使用**native**关键字修饰，Java 代码中只是声明方法头，具体的实现则交给**本地代码**。

![1732497576363-109ffada-28fc-4391-8b89-7d855f7d6b83.png](./img/IHESLtCnx8oKe58G/1732497576363-109ffada-28fc-4391-8b89-7d855f7d6b83-545443.png)

**为什么要使用本地方法呢？**

1. 需要用到 Java 中不具备的依赖于操作系统的特性，Java 在实现跨平台的同时要实现对底层的控制，需要借助其他语言发挥作用。
2. 对于其他语言已经完成的一些现成功能，可以使用 Java 直接调用。
3. 程序对时间敏感或对性能要求非常高时，有必要使用更加底层的语言，例如 C/C++甚至是汇编。

在 JUC 包的很多并发工具类在实现并发机制时，都调用了本地方法，通过它们打破了 Java 运行时的界限，能够接触到操作系统底层的某些功能。对于同一本地方法，不同的操作系统可能会通过不同的方式来实现，但是对于使用者来说是透明的，最终都会得到相同的结果。

## Unsafe 创建
sun.misc.Unsafe部分源码如下：

```plain
public final class Unsafe {
  // 单例对象
  private static final Unsafe theUnsafe;
  ......
  private Unsafe() {
  }
  @CallerSensitive
  public static Unsafe getUnsafe() {
    Class var0 = Reflection.getCallerClass();
    // 仅在引导类加载器`BootstrapClassLoader`加载时才合法
    if(!VM.isSystemDomainLoader(var0.getClassLoader())) {
      throw new SecurityException("Unsafe");
    } else {
      return theUnsafe;
    }
  }
}
```

Unsafe类为一单例实现，提供静态方法getUnsafe获取Unsafe实例。这个看上去貌似可以用来获取Unsafe实例。但是，当我们直接调用这个静态方法的时候，会抛出SecurityException异常：

```plain
Exception in thread "main" java.lang.SecurityException: Unsafe
 at sun.misc.Unsafe.getUnsafe(Unsafe.java:90)
 at com.cn.test.GetUnsafeTest.main(GetUnsafeTest.java:12)
```

**为什么****public static****方法无法被直接调用呢？**

这是因为在getUnsafe方法中，会对调用者的classLoader进行检查，判断当前类是否由Bootstrap classLoader加载，如果不是的话那么就会抛出一个SecurityException异常。也就是说，只有启动类加载器加载的类才能够调用 Unsafe 类中的方法，来防止这些方法在不可信的代码中被调用。

**为什么要对 Unsafe 类进行这么谨慎的使用限制呢?**

Unsafe提供的功能过于底层（如直接访问系统内存资源、自主管理内存资源等），安全隐患也比较大，使用不当的话，很容易出现很严重的问题。

**如若想使用****Unsafe****这个类的话，应该如何获取其实例呢？**

这里介绍两个可行的方案。

1、利用反射获得 Unsafe 类中已经实例化完成的单例对象theUnsafe。

```plain
private static Unsafe reflectGetUnsafe() {
    try {
      Field field = Unsafe.class.getDeclaredField("theUnsafe");
      field.setAccessible(true);
      return (Unsafe) field.get(null);
    } catch (Exception e) {
      log.error(e.getMessage(), e);
      return null;
    }
}
```

2、从getUnsafe方法的使用限制条件出发，通过 Java 命令行命令-Xbootclasspath/a把调用 Unsafe 相关方法的类 A 所在 jar 包路径追加到默认的 bootstrap 路径中，使得 A 被引导类加载器加载，从而通过Unsafe.getUnsafe方法安全的获取 Unsafe 实例。

```plain
java -Xbootclasspath/a: ${path}   // 其中path为调用Unsafe相关方法的类所在jar包路径
```

## Unsafe 功能
概括的来说，Unsafe类实现功能可以被分为下面 8 类：

1. 内存操作
2. 内存屏障
3. 对象操作
4. 数据操作
5. CAS 操作
6. 线程调度
7. Class 操作
8. 系统信息

### 内存操作
#### 介绍
如果你是一个写过 C 或者 C++ 的程序员，一定对内存操作不会陌生，而在 Java 中是不允许直接对内存进行操作的，对象内存的分配和回收都是由 JVM 自己实现的。但是在Unsafe中，提供的下列接口可以直接进行内存操作：

```plain
//分配新的本地空间
public native long allocateMemory(long bytes);
//重新调整内存空间的大小
public native long reallocateMemory(long address, long bytes);
//将内存设置为指定值
public native void setMemory(Object o, long offset, long bytes, byte value);
//内存拷贝
public native void copyMemory(Object srcBase, long srcOffset,Object destBase, long destOffset,long bytes);
//清除内存
public native void freeMemory(long address);
```

使用下面的代码进行测试：

```plain
private void memoryTest() {
    int size = 4;
    long addr = unsafe.allocateMemory(size);
    long addr3 = unsafe.reallocateMemory(addr, size * 2);
    System.out.println("addr: "+addr);
    System.out.println("addr3: "+addr3);
    try {
        unsafe.setMemory(null,addr ,size,(byte)1);
        for (int i = 0; i < 2; i++) {
            unsafe.copyMemory(null,addr,null,addr3+size*i,4);
        }
        System.out.println(unsafe.getInt(addr));
        System.out.println(unsafe.getLong(addr3));
    }finally {
        unsafe.freeMemory(addr);
        unsafe.freeMemory(addr3);
    }
}
```

先看结果输出：

```plain
addr: 2433733895744
addr3: 2433733894944
16843009
72340172838076673
```

分析一下运行结果，首先使用allocateMemory方法申请 4 字节长度的内存空间，调用setMemory方法向每个字节写入内容为byte类型的 1，当使用 Unsafe 调用getInt方法时，因为一个int型变量占 4 个字节，会一次性读取 4 个字节，组成一个int的值，对应的十进制结果为 16843009。

你可以通过下图理解这个过程：

![1732497576496-174844a8-51b9-4dae-823d-f564f21b3d3c.png](./img/IHESLtCnx8oKe58G/1732497576496-174844a8-51b9-4dae-823d-f564f21b3d3c-465873.png)

在代码中调用reallocateMemory方法重新分配了一块 8 字节长度的内存空间，通过比较addr和addr3可以看到和之前申请的内存地址是不同的。在代码中的第二个 for 循环里，调用copyMemory方法进行了两次内存的拷贝，每次拷贝内存地址addr开始的 4 个字节，分别拷贝到以addr3和addr3+4开始的内存空间上：

![1732497576574-fde9788a-0582-4517-8055-ae32d5719e4c.png](./img/IHESLtCnx8oKe58G/1732497576574-fde9788a-0582-4517-8055-ae32d5719e4c-456625.png)

拷贝完成后，使用getLong方法一次性读取 8 个字节，得到long类型的值为 72340172838076673。

需要注意，通过这种方式分配的内存属于 堆外内存 ，是无法进行垃圾回收的，需要我们把这些内存当做一种资源去手动调用freeMemory方法进行释放，否则会产生内存泄漏。通用的操作内存方式是在try中执行对内存的操作，最终在finally块中进行内存的释放。

**为什么要使用堆外内存？**

+ 对垃圾回收停顿的改善。由于堆外内存是直接受操作系统管理而不是 JVM，所以当我们使用堆外内存时，即可保持较小的堆内内存规模。从而在 GC 时减少回收停顿对于应用的影响。
+ 提升程序 I/O 操作的性能。通常在 I/O 通信过程中，会存在堆内内存到堆外内存的数据拷贝操作，对于需要频繁进行内存间数据拷贝且生命周期较短的暂存数据，都建议存储到堆外内存。

#### 典型应用
DirectByteBuffer是 Java 用于实现堆外内存的一个重要类，通常用在通信过程中做缓冲池，如在 Netty、MINA 等 NIO 框架中应用广泛。DirectByteBuffer对于堆外内存的创建、使用、销毁等逻辑均由 Unsafe 提供的堆外内存 API 来实现。

下图为DirectByteBuffer构造函数，创建DirectByteBuffer的时候，通过Unsafe.allocateMemory分配内存、Unsafe.setMemory进行内存初始化，而后构建Cleaner对象用于跟踪DirectByteBuffer对象的垃圾回收，以实现当DirectByteBuffer被垃圾回收时，分配的堆外内存一起被释放。

```plain
DirectByteBuffer(int cap) {                   // package-private

    super(-1, 0, cap, cap);
    boolean pa = VM.isDirectMemoryPageAligned();
    int ps = Bits.pageSize();
    long size = Math.max(1L, (long)cap + (pa ? ps : 0));
    Bits.reserveMemory(size, cap);

    long base = 0;
    try {
        // 分配内存并返回基地址
        base = unsafe.allocateMemory(size);
    } catch (OutOfMemoryError x) {
        Bits.unreserveMemory(size, cap);
        throw x;
    }
    // 内存初始化
    unsafe.setMemory(base, size, (byte) 0);
    if (pa && (base % ps != 0)) {
        // Round up to page boundary
        address = base + ps - (base & (ps - 1));
    } else {
        address = base;
    }
    // 跟踪 DirectByteBuffer 对象的垃圾回收，以实现堆外内存释放
    cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    att = null;
}
```

### 内存屏障
#### 介绍
在介绍内存屏障前，需要知道编译器和 CPU 会在保证程序输出结果一致的情况下，会对代码进行重排序，从指令优化角度提升性能。而指令重排序可能会带来一个不好的结果，导致 CPU 的高速缓存和内存中数据的不一致，而内存屏障（Memory Barrier）就是通过阻止屏障两边的指令重排序从而避免编译器和硬件的不正确优化情况。

在硬件层面上，内存屏障是 CPU 为了防止代码进行重排序而提供的指令，不同的硬件平台上实现内存屏障的方法可能并不相同。在 Java8 中，引入了 3 个内存屏障的函数，它屏蔽了操作系统底层的差异，允许在代码中定义、并统一由 JVM 来生成内存屏障指令，来实现内存屏障的功能。

Unsafe中提供了下面三个内存屏障相关方法：

```plain
//内存屏障，禁止load操作重排序。屏障前的load操作不能被重排序到屏障后，屏障后的load操作不能被重排序到屏障前
public native void loadFence();
//内存屏障，禁止store操作重排序。屏障前的store操作不能被重排序到屏障后，屏障后的store操作不能被重排序到屏障前
public native void storeFence();
//内存屏障，禁止load、store操作重排序
public native void fullFence();
```

内存屏障可以看做对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作。以loadFence方法为例，它会禁止读操作重排序，保证在这个屏障之前的所有读操作都已经完成，并且将缓存数据设为无效，重新从主存中进行加载。

看到这估计很多小伙伴们会想到volatile关键字了，如果在字段上添加了volatile关键字，就能够实现字段在多线程下的可见性。基于读内存屏障，我们也能实现相同的功能。下面定义一个线程方法，在线程中去修改flag标志位，注意这里的flag是没有被volatile修饰的：

```plain
@Getter
class ChangeThread implements Runnable{
    /**volatile**/ boolean flag=false;
    @Override
    public void run() {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("subThread change flag to:" + flag);
        flag = true;
    }
}
```

在主线程的while循环中，加入内存屏障，测试是否能够感知到flag的修改变化：

```plain
public static void main(String[] args){
    ChangeThread changeThread = new ChangeThread();
    new Thread(changeThread).start();
    while (true) {
        boolean flag = changeThread.isFlag();
        unsafe.loadFence(); //加入读内存屏障
        if (flag){
            System.out.println("detected flag changed");
            break;
        }
    }
    System.out.println("main thread end");
}
```

运行结果：

```plain
subThread change flag to:false
detected flag changed
main thread end
```

而如果删掉上面代码中的loadFence方法，那么主线程将无法感知到flag发生的变化，会一直在while中循环。可以用图来表示上面的过程：

![1732497576654-91db44a0-1771-4cd1-b159-10a473845584.png](./img/IHESLtCnx8oKe58G/1732497576654-91db44a0-1771-4cd1-b159-10a473845584-621318.png)

了解 Java 内存模型（JMM）的小伙伴们应该清楚，运行中的线程不是直接读取主内存中的变量的，只能操作自己工作内存中的变量，然后同步到主内存中，并且线程的工作内存是不能共享的。上面的图中的流程就是子线程借助于主内存，将修改后的结果同步给了主线程，进而修改主线程中的工作空间，跳出循环。

#### 典型应用
在 Java 8 中引入了一种锁的新机制——StampedLock，它可以看成是读写锁的一个改进版本。StampedLock提供了一种乐观读锁的实现，这种乐观读锁类似于无锁的操作，完全不会阻塞写线程获取写锁，从而缓解读多写少时写线程“饥饿”现象。由于StampedLock提供的乐观读锁不阻塞写线程获取读锁，当线程共享变量从主内存 load 到线程工作内存时，会存在数据不一致问题。

为了解决这个问题，StampedLock的validate方法会通过Unsafe的loadFence方法加入一个load内存屏障。

```plain
public boolean validate(long stamp) {
   U.loadFence();
   return (stamp & SBITS) == (state & SBITS);
}
```

### 对象操作
#### 介绍
**例子**

```plain
import sun.misc.Unsafe;
import java.lang.reflect.Field;

public class Main {

    private int value;

    public static void main(String[] args) throws Exception{
        Unsafe unsafe = reflectGetUnsafe();
        assert unsafe != null;
        long offset = unsafe.objectFieldOffset(Main.class.getDeclaredField("value"));
        Main main = new Main();
        System.out.println("value before putInt: " + main.value);
        unsafe.putInt(main, offset, 42);
        System.out.println("value after putInt: " + main.value);
    System.out.println("value after putInt: " + unsafe.getInt(main, offset));
    }

    private static Unsafe reflectGetUnsafe() {
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            return (Unsafe) field.get(null);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

}
```

输出结果：

```plain
value before putInt: 0
value after putInt: 42
value after putInt: 42
```

**对象属性**

对象成员属性的内存偏移量获取，以及字段属性值的修改，在上面的例子中我们已经测试过了。除了前面的putInt、getInt方法外，Unsafe 提供了全部 8 种基础数据类型以及Object的put和get方法，并且所有的put方法都可以越过访问权限，直接修改内存中的数据。阅读 openJDK 源码中的注释发现，基础数据类型和Object的读写稍有不同，基础数据类型是直接操作的属性值（value），而Object的操作则是基于引用值（reference value）。下面是Object的读写方法：

```plain
//在对象的指定偏移地址获取一个对象引用
public native Object getObject(Object o, long offset);
//在对象指定偏移地址写入一个对象引用
public native void putObject(Object o, long offset, Object x);
```

除了对象属性的普通读写外，Unsafe还提供了**volatile 读写**和**有序写入**方法。volatile读写方法的覆盖范围与普通读写相同，包含了全部基础数据类型和Object类型，以int类型为例：

```plain
//在对象的指定偏移地址处读取一个int值，支持volatile load语义
public native int getIntVolatile(Object o, long offset);
//在对象指定偏移地址处写入一个int，支持volatile store语义
public native void putIntVolatile(Object o, long offset, int x);
```

相对于普通读写来说，volatile读写具有更高的成本，因为它需要保证可见性和有序性。在执行get操作时，会强制从主存中获取属性值，在使用put方法设置属性值时，会强制将值更新到主存中，从而保证这些变更对其他线程是可见的。

有序写入的方法有以下三个：

```plain
public native void putOrderedObject(Object o, long offset, Object x);
public native void putOrderedInt(Object o, long offset, int x);
public native void putOrderedLong(Object o, long offset, long x);
```

有序写入的成本相对volatile较低，因为它只保证写入时的有序性，而不保证可见性，也就是一个线程写入的值不能保证其他线程立即可见。为了解决这里的差异性，需要对内存屏障的知识点再进一步进行补充，首先需要了解两个指令的概念：

+ Load：将主内存中的数据拷贝到处理器的缓存中
+ Store：将处理器缓存的数据刷新到主内存中

顺序写入与volatile写入的差别在于，在顺序写时加入的内存屏障类型为StoreStore类型，而在volatile写入时加入的内存屏障是StoreLoad类型，如下图所示：

![1732497576738-5d852bf8-3738-4e7e-b9f7-868a3f13fe59.png](./img/IHESLtCnx8oKe58G/1732497576738-5d852bf8-3738-4e7e-b9f7-868a3f13fe59-144827.png)

在有序写入方法中，使用的是StoreStore屏障，该屏障确保Store1立刻刷新数据到内存，这一操作先于Store2以及后续的存储指令操作。而在volatile写入中，使用的是StoreLoad屏障，该屏障确保Store1立刻刷新数据到内存，这一操作先于Load2及后续的装载指令，并且，StoreLoad屏障会使该屏障之前的所有内存访问指令，包括存储指令和访问指令全部完成之后，才执行该屏障之后的内存访问指令。

综上所述，在上面的三类写入方法中，在写入效率方面，按照put、putOrder、putVolatile的顺序效率逐渐降低。

**对象实例化**

使用Unsafe的allocateInstance方法，允许我们使用非常规的方式进行对象的实例化，首先定义一个实体类，并且在构造函数中对其成员变量进行赋值操作：

```plain
@Data
public class A {
    private int b;
    public A(){
        this.b =1;
    }
}
```

分别基于构造函数、反射以及Unsafe方法的不同方式创建对象进行比较：

```plain
public void objTest() throws Exception{
    A a1=new A();
    System.out.println(a1.getB());
    A a2 = A.class.newInstance();
    System.out.println(a2.getB());
    A a3= (A) unsafe.allocateInstance(A.class);
    System.out.println(a3.getB());
}
```

打印结果分别为 1、1、0，说明通过allocateInstance方法创建对象过程中，不会调用类的构造方法。使用这种方式创建对象时，只用到了Class对象，所以说如果想要跳过对象的初始化阶段或者跳过构造器的安全检查，就可以使用这种方法。在上面的例子中，如果将 A 类的构造函数改为private类型，将无法通过构造函数和反射创建对象（可以通过构造函数对象 setAccessible 后创建对象），但allocateInstance方法仍然有效。

#### 典型应用
+ **常规对象实例化方式**：我们通常所用到的创建对象的方式，从本质上来讲，都是通过 new 机制来实现对象的创建。但是，new 机制有个特点就是当类只提供有参的构造函数且无显示声明无参构造函数时，则必须使用有参构造函数进行对象构造，而使用有参构造函数时，必须传递相应个数的参数才能完成对象实例化。
+ **非常规的实例化方式**：而 Unsafe 中提供 allocateInstance 方法，仅通过 Class 对象就可以创建此类的实例对象，而且不需要调用其构造函数、初始化代码、JVM 安全检查等。它抑制修饰符检测，也就是即使构造器是 private 修饰的也能通过此方法实例化，只需提类对象即可创建相应的对象。由于这种特性，allocateInstance 在 java.lang.invoke、Objenesis（提供绕过类构造器的对象生成方式）、Gson（反序列化时用到）中都有相应的应用。

### 数组操作
#### 介绍
arrayBaseOffset与arrayIndexScale这两个方法配合起来使用，即可定位数组中每个元素在内存中的位置。

```plain
//返回数组中第一个元素的偏移地址
public native int arrayBaseOffset(Class<?> arrayClass);
//返回数组中一个元素占用的大小
public native int arrayIndexScale(Class<?> arrayClass);
```

#### 典型应用
这两个与数据操作相关的方法，在java.util.concurrent.atomic包下的AtomicIntegerArray（可以实现对Integer数组中每个元素的原子性操作）中有典型的应用，如下图AtomicIntegerArray源码所示，通过Unsafe的arrayBaseOffset、arrayIndexScale分别获取数组首元素的偏移地址base及单个元素大小因子scale。后续相关原子性操作，均依赖于这两个值进行数组中元素的定位，如下图二所示的getAndAdd方法即通过checkedByteOffset方法获取某数组元素的偏移地址，而后通过 CAS 实现原子性操作。

![1732497576811-147fb259-7167-4747-aaa2-80a79ec1dfeb.png](./img/IHESLtCnx8oKe58G/1732497576811-147fb259-7167-4747-aaa2-80a79ec1dfeb-058727.png)

### CAS 操作
#### 介绍
这部分主要为 CAS 相关操作的方法。

```plain
/**
    *  CAS
  * @param o         包含要修改field的对象
  * @param offset    对象中某field的偏移量
  * @param expected  期望值
  * @param update    更新值
  * @return          true | false
  */
public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object update);

public final native boolean compareAndSwapInt(Object o, long offset, int expected,int update);

public final native boolean compareAndSwapLong(Object o, long offset, long expected, long update);
```

**什么是 CAS?**CAS 即比较并替换（Compare And Swap)，是实现并发算法时常用到的一种技术。CAS 操作包含三个操作数——内存位置、预期原值及新值。执行 CAS 操作的时候，将内存位置的值与预期原值比较，如果相匹配，那么处理器会自动将该位置值更新为新值，否则，处理器不做任何操作。我们都知道，CAS 是一条 CPU 的原子指令（cmpxchg 指令），不会造成所谓的数据不一致问题，Unsafe提供的 CAS 方法（如compareAndSwapXXX）底层实现即为 CPU 指令cmpxchg。

#### 典型应用
在 JUC 包的并发工具类中大量地使用了 CAS 操作，像在前面介绍synchronized和AQS的文章中也多次提到了 CAS，其作为乐观锁在并发工具类中广泛发挥了作用。在Unsafe类中，提供了compareAndSwapObject、compareAndSwapInt、compareAndSwapLong方法来实现的对Object、int、long类型的 CAS 操作。以compareAndSwapInt方法为例：

```plain
public final native boolean compareAndSwapInt(Object o, long offset,int expected,int x);
```

参数中o为需要更新的对象，offset是对象o中整形字段的偏移量，如果这个字段的值与expected相同，则将字段的值设为x这个新值，并且此更新是不可被中断的，也就是一个原子操作。下面是一个使用compareAndSwapInt的例子：

```plain
private volatile int a;
public static void main(String[] args){
    CasTest casTest=new CasTest();
    new Thread(()->{
        for (int i = 1; i < 5; i++) {
            casTest.increment(i);
            System.out.print(casTest.a+" ");
        }
    }).start();
    new Thread(()->{
        for (int i = 5 ; i <10 ; i++) {
            casTest.increment(i);
            System.out.print(casTest.a+" ");
        }
    }).start();
}

private void increment(int x){
    while (true){
        try {
            long fieldOffset = unsafe.objectFieldOffset(CasTest.class.getDeclaredField("a"));
            if (unsafe.compareAndSwapInt(this,fieldOffset,x-1,x))
                break;
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
    }
}
```

运行代码会依次输出：

```plain
1 2 3 4 5 6 7 8 9
```

在上面的例子中，使用两个线程去修改int型属性a的值，并且只有在a的值等于传入的参数x减一时，才会将a的值变为x，也就是实现对a的加一的操作。流程如下所示：

![1732497576918-aa1c2161-c5e1-4c29-b2f2-10d81287ac23.png](./img/IHESLtCnx8oKe58G/1732497576918-aa1c2161-c5e1-4c29-b2f2-10d81287ac23-877469.png)

需要注意的是，在调用compareAndSwapInt方法后，会直接返回true或false的修改结果，因此需要我们在代码中手动添加自旋的逻辑。在AtomicInteger类的设计中，也是采用了将compareAndSwapInt的结果作为循环条件，直至修改成功才退出死循环的方式来实现的原子性的自增操作。

### 线程调度
#### 介绍
Unsafe类中提供了park、unpark、monitorEnter、monitorExit、tryMonitorEnter方法进行线程调度。

```plain
//取消阻塞线程
public native void unpark(Object thread);
//阻塞线程
public native void park(boolean isAbsolute, long time);
//获得对象锁（可重入锁）
@Deprecated
public native void monitorEnter(Object o);
//释放对象锁
@Deprecated
public native void monitorExit(Object o);
//尝试获取对象锁
@Deprecated
public native boolean tryMonitorEnter(Object o);
```

方法park、unpark即可实现线程的挂起与恢复，将一个线程进行挂起是通过park方法实现的，调用park方法后，线程将一直阻塞直到超时或者中断等条件出现；unpark可以终止一个挂起的线程，使其恢复正常。

此外，Unsafe源码中monitor相关的三个方法已经被标记为deprecated，不建议被使用：

```plain
//获得对象锁
@Deprecated
public native void monitorEnter(Object var1);
//释放对象锁
@Deprecated
public native void monitorExit(Object var1);
//尝试获得对象锁
@Deprecated
public native boolean tryMonitorEnter(Object var1);
```

monitorEnter方法用于获得对象锁，monitorExit用于释放对象锁，如果对一个没有被monitorEnter加锁的对象执行此方法，会抛出IllegalMonitorStateException异常。tryMonitorEnter方法尝试获取对象锁，如果成功则返回true，反之返回false。

#### 典型应用
Java 锁和同步器框架的核心类AbstractQueuedSynchronizer(AQS)，就是通过调用LockSupport.park()和LockSupport.unpark()实现线程的阻塞和唤醒的，而LockSupport的park、unpark方法实际是调用Unsafe的park、unpark方式实现的。

```plain
public static void park(Object blocker) {
    Thread t = Thread.currentThread();
    setBlocker(t, blocker);
    UNSAFE.park(false, 0L);
    setBlocker(t, null);
}
public static void unpark(Thread thread) {
    if (thread != null)
        UNSAFE.unpark(thread);
}
```

LockSupport的park方法调用了Unsafe的park方法来阻塞当前线程，此方法将线程阻塞后就不会继续往后执行，直到有其他线程调用unpark方法唤醒当前线程。下面的例子对Unsafe的这两个方法进行测试：

```plain
public static void main(String[] args) {
    Thread mainThread = Thread.currentThread();
    new Thread(()->{
        try {
            TimeUnit.SECONDS.sleep(5);
            System.out.println("subThread try to unpark mainThread");
            unsafe.unpark(mainThread);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();

    System.out.println("park main mainThread");
    unsafe.park(false,0L);
    System.out.println("unpark mainThread success");
}
```

程序输出为：

```plain
park main mainThread
subThread try to unpark mainThread
unpark mainThread success
```

程序运行的流程也比较容易看懂，子线程开始运行后先进行睡眠，确保主线程能够调用park方法阻塞自己，子线程在睡眠 5 秒后，调用unpark方法唤醒主线程，使主线程能继续向下执行。整个流程如下图所示：

![1732497576989-3797a53c-026e-46f0-ad6d-b9784ffd06f2.png](./img/IHESLtCnx8oKe58G/1732497576989-3797a53c-026e-46f0-ad6d-b9784ffd06f2-487162.png)

### Class 操作
#### 介绍
Unsafe对Class的相关操作主要包括类加载和静态变量的操作方法。

**静态属性读取相关的方法**

```plain
//获取静态属性的偏移量
public native long staticFieldOffset(Field f);
//获取静态属性的对象指针
public native Object staticFieldBase(Field f);
//判断类是否需要初始化（用于获取类的静态属性前进行检测）
public native boolean shouldBeInitialized(Class<?> c);
```

创建一个包含静态属性的类，进行测试：

```plain
@Data
public class User {
    public static String name="Hydra";
    int age;
}
private void staticTest() throws Exception {
    User user=new User();
    // 也可以用下面的语句触发类初始化
    // 1.
    // unsafe.ensureClassInitialized(User.class);
    // 2.
    // System.out.println(User.name);
    System.out.println(unsafe.shouldBeInitialized(User.class));
    Field sexField = User.class.getDeclaredField("name");
    long fieldOffset = unsafe.staticFieldOffset(sexField);
    Object fieldBase = unsafe.staticFieldBase(sexField);
    Object object = unsafe.getObject(fieldBase, fieldOffset);
    System.out.println(object);
}
```

运行结果：

```plain
false
Hydra
```

在Unsafe的对象操作中，我们学习了通过objectFieldOffset方法获取对象属性偏移量并基于它对变量的值进行存取，但是它不适用于类中的静态属性，这时候就需要使用staticFieldOffset方法。在上面的代码中，只有在获取Field对象的过程中依赖到了Class，而获取静态变量的属性时不再依赖于Class。

在上面的代码中首先创建一个User对象，这是因为如果一个类没有被初始化，那么它的静态属性也不会被初始化，最后获取的字段属性将是null。所以在获取静态属性前，需要调用shouldBeInitialized方法，判断在获取前是否需要初始化这个类。如果删除创建 User 对象的语句，运行结果会变为：

```plain
true
null
```

**使用****defineClass****方法允许程序在运行时动态地创建一个类**

```plain
public native Class<?> defineClass(String name, byte[] b, int off, int len, ClassLoader loader,ProtectionDomain protectionDomain);
```

在实际使用过程中，可以只传入字节数组、起始字节的下标以及读取的字节长度，默认情况下，类加载器（ClassLoader）和保护域（ProtectionDomain）来源于调用此方法的实例。下面的例子中实现了反编译生成后的 class 文件的功能：

```plain
private static void defineTest() {
    String fileName="F:\\workspace\\unsafe-test\\target\\classes\\com\\cn\\model\\User.class";
    File file = new File(fileName);
    try(FileInputStream fis = new FileInputStream(file)) {
        byte[] content=new byte[(int)file.length()];
        fis.read(content);
        Class clazz = unsafe.defineClass(null, content, 0, content.length, null, null);
        Object o = clazz.newInstance();
        Object age = clazz.getMethod("getAge").invoke(o, null);
        System.out.println(age);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

在上面的代码中，首先读取了一个class文件并通过文件流将它转化为字节数组，之后使用defineClass方法动态的创建了一个类，并在后续完成了它的实例化工作，流程如下图所示，并且通过这种方式创建的类，会跳过 JVM 的所有安全检查。

![1732497577112-8aa77727-6f23-492d-a3c8-dafc0099b71c.png](./img/IHESLtCnx8oKe58G/1732497577112-8aa77727-6f23-492d-a3c8-dafc0099b71c-821553.png)

除了defineClass方法外，Unsafe 还提供了一个defineAnonymousClass方法：

```plain
public native Class<?> defineAnonymousClass(Class<?> hostClass, byte[] data, Object[] cpPatches);
```

使用该方法可以用来动态的创建一个匿名类，在Lambda表达式中就是使用 ASM 动态生成字节码，然后利用该方法定义实现相应的函数式接口的匿名类。在 JDK 15 发布的新特性中，在隐藏类（Hidden classes）一条中，指出将在未来的版本中弃用Unsafe的defineAnonymousClass方法。

#### 典型应用
Lambda 表达式实现需要依赖Unsafe的defineAnonymousClass方法定义实现相应的函数式接口的匿名类。

### 系统信息
#### 介绍
这部分包含两个获取系统相关信息的方法。

```plain
//返回系统指针的大小。返回值为4（32位系统）或 8（64位系统）。
public native int addressSize();
//内存页的大小，此值为2的幂次方。
public native int pageSize();
```

#### 典型应用
这两个方法的应用场景比较少，在java.nio.Bits类中，在使用pageCount计算所需的内存页的数量时，调用了pageSize方法获取内存页的大小。另外，在使用copySwapMemory方法拷贝内存时，调用了addressSize方法，检测 32 位系统的情况。

## 总结
在本文中，我们首先介绍了Unsafe的基本概念、工作原理，并在此基础上，对它的 API 进行了说明与实践。相信大家通过这一过程，能够发现Unsafe在某些场景下，确实能够为我们提供编程中的便利。但是回到开头的话题，在使用这些便利时，确实存在着一些安全上的隐患，在我看来，一项技术具有不安全因素并不可怕，可怕的是它在使用过程中被滥用。尽管之前有传言说会在 Java9 中移除Unsafe类，不过它还是照样已经存活到了 Java16。按照存在即合理的逻辑，只要使用得当，它还是能给我们带来不少的帮助，因此最后还是建议大家，在使用Unsafe的过程中一定要做到使用谨慎使用、避免滥用。





> 更新: 2024-01-02 20:01:47  
原文: [https://www.yuque.com/vip6688/neho4x/ezql2ci6pp8v68p5](https://www.yuque.com/vip6688/neho4x/ezql2ci6pp8v68p5)
>



> 更新: 2024-11-25 09:19:37  
> 原文: <https://www.yuque.com/neumx/laxg2e/4d16ed73da4eecf3d729b6c85684b743>