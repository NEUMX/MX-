# Redis大Key对持久化有什么影响？

# Redis 大 Key 对持久化有什么影响？
<font style="color:rgb(44, 62, 80);">Redis 的持久化方式有两种：AOF 日志和 RDB 快照。</font>

<font style="color:rgb(44, 62, 80);">所以接下来，针对这两种持久化方式具体分析分析。</font>

## [](https://xiaolincoding.com/redis/storage/bigkey_aof_rdb.html#%E5%A4%A7-key-%E5%AF%B9-aof-%E6%97%A5%E5%BF%97%E7%9A%84%E5%BD%B1%E5%93%8D)<font style="color:rgb(44, 62, 80);">大 Key 对 AOF 日志的影响</font>
<font style="color:rgb(44, 62, 80);background-color:rgb(227, 242, 253);">先说说 AOF 日志三种写回磁盘的策略</font>

<font style="color:rgb(44, 62, 80);">Redis 提供了 3 种 AOF 日志写回硬盘的策略，分别是：</font>

+ <font style="color:rgb(44, 62, 80);">Always，这个单词的意思是「总是」，所以它的意思是每次写操作命令执行完后，同步将 AOF 日志数据写回硬盘；</font>
+ <font style="color:rgb(44, 62, 80);">Everysec，这个单词的意思是「每秒」，所以它的意思是每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区，然后每隔一秒将缓冲区里的内容写回到硬盘；</font>
+ <font style="color:rgb(44, 62, 80);">No，意味着不由 Redis 控制写回硬盘的时机，转交给操作系统控制写回的时机，也就是每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区，再由操作系统决定何时将缓冲区内容写回硬盘。</font>

<font style="color:rgb(44, 62, 80);">这三种策略只是在控制 fsync() 函数的调用时机。</font>

<font style="color:rgb(44, 62, 80);">当应用程序向文件写入数据时，内核通常先将数据复制到内核缓冲区中，然后排入队列，然后由内核决定何时写入硬盘。</font>

![1732497711957-369ce742-2e39-4716-95b9-c2d448f4b351.png](./img/rsnKdFVCxIywcAFN/1732497711957-369ce742-2e39-4716-95b9-c2d448f4b351-409086.png)

<font style="color:rgb(44, 62, 80);">如果想要应用程序向文件写入数据后，能立马将数据同步到硬盘，就可以调用 fsync() 函数，这样内核就会将内核缓冲区的数据直接写入到硬盘，等到硬盘写操作完成后，该函数才会返回。</font>

+ <font style="color:rgb(44, 62, 80);">Always 策略就是每次写入 AOF 文件数据后，就执行 fsync() 函数；</font>
+ <font style="color:rgb(44, 62, 80);">Everysec 策略就会创建一个异步任务来执行 fsync() 函数；</font>
+ <font style="color:rgb(44, 62, 80);">No 策略就是永不执行 fsync() 函数;</font>

<font style="color:rgb(44, 62, 80);background-color:rgb(227, 242, 253);">分别说说这三种策略，在持久化大 Key 的时候，会影响什么？</font>

<font style="color:rgb(44, 62, 80);">在使用 Always 策略的时候，主线程在执行完命令后，会把数据写入到 AOF 日志文件，然后会调用 fsync() 函数，将内核缓冲区的数据直接写入到硬盘，等到硬盘写操作完成后，该函数才会返回。</font>

**<font style="color:rgb(48, 79, 254);">当使用 Always 策略的时候，如果写入是一个大 Key，主线程在执行 fsync() 函数的时候，阻塞的时间会比较久，因为当写入的数据量很大的时候，数据同步到硬盘这个过程是很耗时的</font>**<font style="color:rgb(44, 62, 80);">。</font>

<font style="color:rgb(44, 62, 80);">当使用 Everysec 策略的时候，由于是异步执行 fsync() 函数，所以大 Key 持久化的过程（数据同步磁盘）不会影响主线程。</font>

<font style="color:rgb(44, 62, 80);">当使用 No 策略的时候，由于永不执行 fsync() 函数，所以大 Key 持久化的过程不会影响主线程。</font>

## [](https://xiaolincoding.com/redis/storage/bigkey_aof_rdb.html#%E5%A4%A7-key-%E5%AF%B9-aof-%E9%87%8D%E5%86%99%E5%92%8C-rdb-%E7%9A%84%E5%BD%B1%E5%93%8D)<font style="color:rgb(44, 62, 80);">大 Key 对 AOF 重写和 RDB 的影响</font>
<font style="color:rgb(44, 62, 80);">当 AOF 日志写入了很多的大 Key，AOF 日志文件的大小会很大，那么很快就会触发</font><font style="color:rgb(44, 62, 80);"> </font>**<font style="color:rgb(48, 79, 254);">AOF 重写机制</font>**<font style="color:rgb(44, 62, 80);">。</font>

<font style="color:rgb(44, 62, 80);">AOF 重写机制和 RDB 快照（bgsave 命令）的过程，都会分别通过</font><font style="color:rgb(44, 62, 80);"> </font><font style="color:rgb(71, 101, 130);">fork()</font><font style="color:rgb(44, 62, 80);"> </font><font style="color:rgb(44, 62, 80);">函数创建一个子进程来处理任务。</font>

<font style="color:rgb(44, 62, 80);">在创建子进程的过程中，操作系统会把父进程的「页表」复制一份给子进程，这个页表记录着虚拟地址和物理地址映射关系，而不会复制物理内存，也就是说，两者的虚拟空间不同，但其对应的物理空间是同一个。</font>

![1732497712153-c86d4d77-2872-48e2-a3f8-a985ba9406a5.png](./img/rsnKdFVCxIywcAFN/1732497712153-c86d4d77-2872-48e2-a3f8-a985ba9406a5-191429.png)<font style="color:rgb(44, 62, 80);"> </font><font style="color:rgb(44, 62, 80);">这样一来，子进程就共享了父进程的物理内存数据了，这样能够节约物理内存资源，页表对应的页表项的属性会标记该物理内存的权限为</font>**<font style="color:rgb(48, 79, 254);">只读</font>**<font style="color:rgb(44, 62, 80);">。</font>

<font style="color:rgb(44, 62, 80);">随着 Redis 存在越来越多的大 Key，那么 Redis 就会占用很多内存，对应的页表就会越大。</font>

<font style="color:rgb(44, 62, 80);">在通过</font><font style="color:rgb(44, 62, 80);"> </font><font style="color:rgb(71, 101, 130);">fork()</font><font style="color:rgb(44, 62, 80);"> </font><font style="color:rgb(44, 62, 80);">函数创建子进程的时候，虽然不会复制父进程的物理内存，但是</font>**<font style="color:rgb(48, 79, 254);">内核会把父进程的页表复制一份给子进程，如果页表很大，那么这个复制过程是会很耗时的，那么在执行 fork 函数的时候就会发生阻塞现象</font>**<font style="color:rgb(44, 62, 80);">。</font>

<font style="color:rgb(44, 62, 80);">而且，fork 函数是由 Redis 主线程调用的，如果 fork 函数发生阻塞，那么意味着就会阻塞 Redis 主线程。由于 Redis 执行命令是在主线程处理的，所以当 Redis 主线程发生阻塞，就无法处理后续客户端发来的命令。</font>

<font style="color:rgb(44, 62, 80);">我们可以执行</font><font style="color:rgb(44, 62, 80);"> </font><font style="color:rgb(71, 101, 130);">info</font><font style="color:rgb(44, 62, 80);"> </font><font style="color:rgb(44, 62, 80);">命令获取到 latest_fork_usec 指标，表示 Redis 最近一次 fork 操作耗时。</font>



```sql
 最近一次 fork 操作耗时
latest_fork_usec:315
```

<font style="color:rgb(44, 62, 80);">如果 fork 耗时很大，比如超过1秒，则需要做出优化调整：</font>

+ <font style="color:rgb(44, 62, 80);">单个实例的内存占用控制在 10 GB 以下，这样 fork 函数就能很快返回。</font>
+ <font style="color:rgb(44, 62, 80);">如果 Redis 只是当作纯缓存使用，不关心 Redis 数据安全性问题，可以考虑关闭 AOF 和 AOF 重写，这样就不会调用 fork 函数了。</font>
+ <font style="color:rgb(44, 62, 80);">在主从架构中，要适当调大 repl-backlog-size，避免因为 repl_backlog_buffer 不够大，导致主节点频繁地使用全量同步的方式，全量同步的时候，是会创建 RDB 文件的，也就是会调用 fork 函数。</font>

<font style="color:rgb(44, 62, 80);background-color:rgb(227, 242, 253);">那什么时候会发生物理内存的复制呢？</font>

<font style="color:rgb(44, 62, 80);">当父进程或者子进程在向共享内存发起写操作时，CPU 就会触发</font>**<font style="color:rgb(48, 79, 254);">写保护中断</font>**<font style="color:rgb(44, 62, 80);">，这个「写保护中断」是由于违反权限导致的，然后操作系统会在「写保护中断处理函数」里进行物理内存的复制，并重新设置其内存映射关系，将父子进程的内存读写权限设置为可读写，最后才会对内存进行写操作，这个过程被称为「</font>**<font style="color:rgb(48, 79, 254);">写时复制(Copy On Write)</font>**<font style="color:rgb(44, 62, 80);">」。</font>

![1732497712263-9d97583d-7e4f-4e6d-8189-7aa427dea3f2.png](./img/rsnKdFVCxIywcAFN/1732497712263-9d97583d-7e4f-4e6d-8189-7aa427dea3f2-761146.png)

<font style="color:rgb(44, 62, 80);">写时复制顾名思义，在发生写操作的时候，操作系统才会去复制物理内存，这样是为了防止 fork 创建子进程时，由于物理内存数据的复制时间过长而导致父进程长时间阻塞的问题。</font>

<font style="color:rgb(44, 62, 80);">如果创建完子进程后，</font>**<font style="color:rgb(48, 79, 254);">父进程对共享内存中的大 Key 进行了修改，那么内核就会发生写时复制，会把物理内存复制一份，由于大 Key 占用的物理内存是比较大的，那么在复制物理内存这一过程中，也是比较耗时的，于是父进程（主线程）就会发生阻塞</font>**<font style="color:rgb(44, 62, 80);">。</font>

<font style="color:rgb(44, 62, 80);">所以，有两个阶段会导致阻塞父进程：</font>

+ <font style="color:rgb(44, 62, 80);">创建子进程的途中，由于要复制父进程的页表等数据结构，阻塞的时间跟页表的大小有关，页表越大，阻塞的时间也越长；</font>
+ <font style="color:rgb(44, 62, 80);">创建完子进程后，如果子进程或者父进程修改了共享数据，就会发生写时复制，这期间会拷贝物理内存，如果内存越大，自然阻塞的时间也越长；</font>

<font style="color:rgb(44, 62, 80);">这里额外提一下， 如果</font><font style="color:rgb(44, 62, 80);"> </font>**<font style="color:rgb(48, 79, 254);">Linux 开启了内存大页，会影响 Redis 的性能的</font>**<font style="color:rgb(44, 62, 80);">。</font>

<font style="color:rgb(44, 62, 80);">Linux 内核从 2.6.38 开始支持内存大页机制，该机制支持 2MB 大小的内存页分配，而常规的内存页分配是按 4KB 的粒度来执行的。</font>

<font style="color:rgb(44, 62, 80);">如果采用了内存大页，那么即使客户端请求只修改 100B 的数据，在发生写时复制后，Redis 也需要拷贝 2MB 的大页。相反，如果是常规内存页机制，只用拷贝 4KB。</font>

<font style="color:rgb(44, 62, 80);">两者相比，你可以看到，每次写命令引起的</font>**<font style="color:rgb(48, 79, 254);">复制内存页单位放大了 512 倍，会拖慢写操作的执行时间，最终导致 Redis 性能变慢</font>**<font style="color:rgb(44, 62, 80);">。</font>

<font style="color:rgb(44, 62, 80);">那该怎么办呢？很简单，关闭内存大页（默认是关闭的）。</font>

<font style="color:rgb(44, 62, 80);">禁用方法如下：</font>



```shell
echo never >  /sys/kernel/mm/transparent_hugepage/enabled
```

## [](https://xiaolincoding.com/redis/storage/bigkey_aof_rdb.html#%E6%80%BB%E7%BB%93)<font style="color:rgb(44, 62, 80);">总结</font>
<font style="color:rgb(44, 62, 80);">当 AOF 写回策略配置了 Always 策略，如果写入是一个大 Key，主线程在执行 fsync() 函数的时候，阻塞的时间会比较久，因为当写入的数据量很大的时候，数据同步到硬盘这个过程是很耗时的。</font>

<font style="color:rgb(44, 62, 80);">AOF 重写机制和 RDB 快照（bgsave 命令）的过程，都会分别通过</font><font style="color:rgb(44, 62, 80);"> </font><font style="color:rgb(71, 101, 130);">fork()</font><font style="color:rgb(44, 62, 80);"> </font><font style="color:rgb(44, 62, 80);">函数创建一个子进程来处理任务。会有两个阶段会导致阻塞父进程（主线程）：</font>

+ <font style="color:rgb(44, 62, 80);">创建子进程的途中，由于要复制父进程的页表等数据结构，阻塞的时间跟页表的大小有关，页表越大，阻塞的时间也越长；</font>
+ <font style="color:rgb(44, 62, 80);">创建完子进程后，如果父进程修改了共享数据中的大 Key，就会发生写时复制，这期间会拷贝物理内存，由于大 Key 占用的物理内存会很大，那么在复制物理内存这一过程，就会比较耗时，所以有可能会阻塞父进程。</font>

<font style="color:rgb(44, 62, 80);">大 key 除了会影响持久化之外，还会有以下的影响。</font>

+ <font style="color:rgb(44, 62, 80);">客户端超时阻塞。由于 Redis 执行命令是单线程处理，然后在操作大 key 时会比较耗时，那么就会阻塞 Redis，从客户端这一视角看，就是很久很久都没有响应。</font>
+ <font style="color:rgb(44, 62, 80);">引发网络阻塞。每次获取大 key 产生的网络流量较大，如果一个 key 的大小是 1 MB，每秒访问量为 1000，那么每秒会产生 1000MB 的流量，这对于普通千兆网卡的服务器来说是灾难性的。</font>
+ <font style="color:rgb(44, 62, 80);">阻塞工作线程。如果使用 del 删除大 key 时，会阻塞工作线程，这样就没办法处理后续的命令。</font>
+ <font style="color:rgb(44, 62, 80);">内存分布不均。集群模型在 slot 分片均匀情况下，会出现数据和查询倾斜情况，部分有大 key 的 Redis 节点占用内存多，QPS 也会比较大。</font>

<font style="color:rgb(44, 62, 80);">如何避免大 Key 呢？</font>

<font style="color:rgb(44, 62, 80);">最好在设计阶段，就把大 key 拆分成一个一个小 key。或者，定时检查 Redis 是否存在大 key ，如果该大 key 是可以删除的，不要使用 DEL 命令删除，因为该命令删除过程会阻塞主线程，而是用 unlink 命令（Redis 4.0+）删除大 key，因为该命令的删除过程是异步的，不会阻塞主线程。</font>

<font style="color:rgb(44, 62, 80);">完！</font>



> 更新: 2024-01-02 19:32:07  
原文: [https://www.yuque.com/vip6688/neho4x/ppc4a6lyf1wqhb8k](https://www.yuque.com/vip6688/neho4x/ppc4a6lyf1wqhb8k)
>



> 更新: 2024-11-25 16:07:18  
> 原文: <https://www.yuque.com/neumx/laxg2e/78c3c24067881b183ab63410f7722337>