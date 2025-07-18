# Redis内存碎片详解

# Redis内存碎片详解
## 什么是内存碎片?
你可以将内存碎片简单地理解为那些不可用的空闲内存。

举个例子：操作系统为你分配了 32 字节的连续内存空间，而你存储数据实际只需要使用 24 字节内存空间，那这多余出来的 8 字节内存空间如果后续没办法再被分配存储其他数据的话，就可以被称为内存碎片。

![1732497697162-f1f7b976-d8fe-46e2-afd0-e3269be16187.png](./img/6XeZ-IXUu3bcVPPu/1732497697162-f1f7b976-d8fe-46e2-afd0-e3269be16187-984544.png)

内存碎片

Redis 内存碎片虽然不会影响 Redis 性能，但是会增加内存消耗。

## 为什么会有 Redis 内存碎片?
Redis 内存碎片产生比较常见的 2 个原因：

**1、Redis 存储存储数据的时候向操作系统申请的内存空间可能会大于数据实际需要的存储空间。**

以下是这段 Redis 官方的原话：

To store user keys, Redis allocates at most as much memory as themaxmemorysetting enables (however there are small extra allocations possible).

Redis 使用zmalloc方法(Redis 自己实现的内存分配方法)进行内存分配的时候，除了要分配size大小的内存之外，还会多分配PREFIX_SIZE大小的内存。

zmalloc方法源码如下（源码地址：[https://github.com/antirez/redis-tools/blob/master/zmalloc.c）：](https://github.com/antirez/redis-tools/blob/master/zmalloc.c%EF%BC%89%EF%BC%9A)[open in new window](https://github.com/antirez/redis-tools/blob/master/zmalloc.c%EF%BC%89%EF%BC%9A)

```plain
void *zmalloc(size_t size) {
   // 分配指定大小的内存
   void *ptr = malloc(size+PREFIX_SIZE);
   if (!ptr) zmalloc_oom_handler(size);
#ifdef HAVE_MALLOC_SIZE
   update_zmalloc_stat_alloc(zmalloc_size(ptr));
   return ptr;
#else
   *((size_t*)ptr) = size;
   update_zmalloc_stat_alloc(size+PREFIX_SIZE);
   return (char*)ptr+PREFIX_SIZE;
#endif
}
```

另外，Redis 可以使用多种内存分配器来分配内存（ libc、jemalloc、tcmalloc），默认使用[jemalloc](https://github.com/jemalloc/jemalloc)[open in new window](https://github.com/jemalloc/jemalloc)，而 jemalloc 按照一系列固定的大小（8 字节、16 字节、32 字节……）来分配内存的。jemalloc 划分的内存单元如下图所示：

![1732497697260-ae63acbc-9112-4d15-a390-8e5c393950f6.png](./img/6XeZ-IXUu3bcVPPu/1732497697260-ae63acbc-9112-4d15-a390-8e5c393950f6-731356.png)

jemalloc 内存单元示意图

当程序申请的内存最接近某个固定值时，jemalloc 会给它分配相应大小的空间，就比如说程序需要申请 17 字节的内存，jemalloc 会直接给它分配 32 字节的内存，这样会导致有 15 字节内存的浪费。不过，jemalloc 专门针对内存碎片问题做了优化，一般不会存在过度碎片化的问题。

**2、频繁修改 Redis 中的数据也会产生内存碎片。**

当 Redis 中的某个数据删除时，Redis 通常不会轻易释放内存给操作系统。

这个在 Redis 官方文档中也有对应的原话:

![1732497697328-03081ca3-84d2-46d0-a87b-66e0de086de2.png](./img/6XeZ-IXUu3bcVPPu/1732497697328-03081ca3-84d2-46d0-a87b-66e0de086de2-083775.png)

文档地址：[https://redis.io/topics/memory-optimization](https://redis.io/topics/memory-optimization)[open in new window](https://redis.io/topics/memory-optimization)。

## 如何查看 Redis 内存碎片的信息？
使用info memory命令即可查看 Redis 内存相关的信息。下图中每个参数具体的含义，Redis 官方文档有详细的介绍：[https://redis.io/commands/INFO](https://redis.io/commands/INFO)[open in new window](https://redis.io/commands/INFO)。

![1732497697398-8d88c029-450a-420b-8d30-d793d9d2f20a.png](./img/6XeZ-IXUu3bcVPPu/1732497697398-8d88c029-450a-420b-8d30-d793d9d2f20a-809952.png)

Redis 内存碎片率的计算公式：mem_fragmentation_ratio（内存碎片率）=used_memory_rss(操作系统实际分配给 Redis 的物理内存空间大小)/used_memory(Redis 内存分配器为了存储数据实际申请使用的内存空间大小)

也就是说，mem_fragmentation_ratio（内存碎片率）的值越大代表内存碎片率越严重。

一定不要误认为used_memory_rss减去used_memory值就是内存碎片的大小！！！这不仅包括内存碎片，还包括其他进程开销，以及共享库、堆栈等的开销。

很多小伙伴可能要问了：“多大的内存碎片率才是需要清理呢？”。

通常情况下，我们认为mem_fragmentation_ratio > 1.5的话才需要清理内存碎片。mem_fragmentation_ratio > 1.5意味着你使用 Redis 存储实际大小 2G 的数据需要使用大于 3G 的内存。

如果想要快速查看内存碎片率的话，你还可以通过下面这个命令：

```plain
> redis-cli -p 6379 info | grep mem_fragmentation_ratio
```

另外，内存碎片率可能存在小于 1 的情况。这种情况我在日常使用中还没有遇到过，感兴趣的小伙伴可以看看这篇文章[故障分析 | Redis 内存碎片率太低该怎么办？- 爱可生开源社区](https://mp.weixin.qq.com/s/drlDvp7bfq5jt2M5pTqJCw)[open in new window](https://mp.weixin.qq.com/s/drlDvp7bfq5jt2M5pTqJCw)。

## 如何清理 Redis 内存碎片？
Redis4.0-RC3 版本以后自带了内存整理，可以避免内存碎片率过大的问题。

直接通过config set命令将activedefrag配置项设置为yes即可。

```plain
config set activedefrag yes
```

具体什么时候清理需要通过下面两个参数控制：

```plain
# 内存碎片占用空间达到 500mb 的时候开始清理
config set active-defrag-ignore-bytes 500mb
# 内存碎片率大于 1.5 的时候开始清理
config set active-defrag-threshold-lower 50
```

通过 Redis 自动内存碎片清理机制可能会对 Redis 的性能产生影响，我们可以通过下面两个参数来减少对 Redis 性能的影响：

```plain
# 内存碎片清理所占用 CPU 时间的比例不低于 20%
config set active-defrag-cycle-min 20
# 内存碎片清理所占用 CPU 时间的比例不高于 50%
config set active-defrag-cycle-max 50
```

另外，重启节点可以做到内存碎片重新整理。如果你采用的是高可用架构的 Redis 集群的话，你可以将碎片率过高的主节点转换为从节点，以便进行安全重启。

## 参考
+ Redis 官方文档：[https://redis.io/topics/memory-optimization](https://redis.io/topics/memory-optimization)[open in new window](https://redis.io/topics/memory-optimization)
+ Redis 核心技术与实战 - 极客时间 - 删除数据后，为什么内存占用率还是很高？：[https://time.geekbang.org/column/article/289140](https://time.geekbang.org/column/article/289140)[open in new window](https://time.geekbang.org/column/article/289140)
+ Redis 源码解析——内存分配：<[https://shinerio.cc/2020/05/17/redis/Redis](https://shinerio.cc/2020/05/17/redis/Redis)[open in new window](https://shinerio.cc/2020/05/17/redis/Redis)源码解析——内存管理>



> 更新: 2024-01-03 17:23:19  
原文: [https://www.yuque.com/vip6688/neho4x/ag9tm0dxl1igdxec](https://www.yuque.com/vip6688/neho4x/ag9tm0dxl1igdxec)
>



> 更新: 2024-11-25 09:21:38  
> 原文: <https://www.yuque.com/neumx/laxg2e/306e9b24c44fc6d2262d6e7ffb6a4e80>