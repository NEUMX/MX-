# JVM线上问题排查和性能调优案例

# JVM线上问题排查和性能调优案例
JVM 线上问题排查和性能调优也是面试常问的一个问题，尤其是社招中大厂的面试。

这篇文章，我会分享一些我看到的相关的案例。

下面是正文。

[一次线上 OOM 问题分析 - 艾小仙 - 2023](https://juejin.cn/post/7205141492264976445)[open in new window](https://juejin.cn/post/7205141492264976445)

+ **现象**：线上某个服务有接口非常慢，通过监控链路查看发现，中间的 GAP 时间非常大，实际接口并没有消耗很多时间，并且在那段时间里有很多这样的请求。
+ **分析**：使用 JDK 自带的jvisualvm分析 dump 文件(MAT 也能分析)。
+ **建议**：对于 SQL 语句，如果监测到没有where条件的全表查询应该默认增加一个合适的limit作为限制，防止这种问题拖垮整个系统
+ **资料**：[实战案例：记一次 dump 文件分析历程转载 - HeapDump - 2022](https://heapdump.cn/article/3489050)[open in new window](https://heapdump.cn/article/3489050)。

[生产事故-记一次特殊的 OOM 排查 - 程语有云 - 2023](https://www.cnblogs.com/mylibs/p/production-accident-0002.html)[open in new window](https://www.cnblogs.com/mylibs/p/production-accident-0002.html)

+ **现象**：网络没有问题的情况下，系统某开放接口从 2023 年 3 月 10 日 14 时许开始无法访问和使用。
+ **临时解决办法**：紧急回滚至上一稳定版本。
+ **分析**：使用 MAT (Memory Analyzer Tool)工具分析 dump 文件。
+ **建议**：正常情况下，-Xmn参数（控制 Young 区的大小）总是应当小于-Xmx参数（控制堆内存的最大大小），否则就会触发 OOM 错误。
+ **资料**：[最重要的 JVM 参数总结 - JavaGuide - 2023](https://javaguide.cn/java/jvm/jvm-parameters-intro.html)[open in new window](https://javaguide.cn/java/jvm/jvm-parameters-intro.html)

[一次大量 JVM Native 内存泄露的排查分析（64M 问题） - 掘金 - 2022](https://juejin.cn/post/7078624931826794503)[open in new window](https://juejin.cn/post/7078624931826794503)

+ **现象**：线上项目刚启动完使用 top 命令查看 RES 占用了超过 1.5G。
+ **分析**：整个分析流程用到了较多工作，可以跟着作者思路一步一步来，值得学习借鉴。
+ **建议**：远离 Hibernate。
+ **资料**：[Linux top 命令里的内存相关字段（VIRT, RES, SHR, CODE, DATA）](https://liam.page/2020/07/17/memory-stat-in-TOP/)[open in new window](https://liam.page/2020/07/17/memory-stat-in-TOP/)

[YGC 问题排查，又让我涨姿势了！ - IT 人的职场进阶 - 2021](https://www.heapdump.cn/article/1661497)[open in new window](https://www.heapdump.cn/article/1661497)

+ **现象**：广告服务在新版本上线后，收到了大量的服务超时告警。
+ **分析**：使用 MAT (Memory Analyzer Tool) 工具分析 dump 文件。
+ **建议**：学会 YGC（Young GC） 问题的排查思路，掌握 YGC 的相关知识点。

[听说 JVM 性能优化很难？今天我小试了一把！ - 陈树义 - 2021](https://shuyi.tech/archives/have-a-try-in-jvm-combat)[open in new window](https://shuyi.tech/archives/have-a-try-in-jvm-combat)

通过观察 GC 频率和停顿时间，来进行 JVM 内存空间调整，使其达到最合理的状态。调整过程记得小步快跑，避免内存剧烈波动影响线上服务。 这其实是最为简单的一种 JVM 性能调优方式了，可以算是粗调吧。

[你们要的线上 GC 问题案例来啦 - 编了个程 - 2021](https://mp.weixin.qq.com/s/df1uxHWUXzhErxW1sZ6OvQ)[open in new window](https://mp.weixin.qq.com/s/df1uxHWUXzhErxW1sZ6OvQ)

+ **案例 1**：使用 guava cache 的时候，没有设置最大缓存数量和弱引用，导致频繁触发 Young GC
+ **案例 2**： 对于一个查询和排序分页的 SQL，同时这个 SQL 需要 join 多张表，在分库分表下，直接调用 SQL 性能很差。于是，查单表，再在内存排序分页，用了一个 List 来保存数据，而有些数据量大，造成了这个现象。

[Java 中 9 种常见的 CMS GC 问题分析与解决 - 美团技术团 - 2020](https://tech.meituan.com/2020/11/12/java-9-cms-gc.html)[open in new window](https://tech.meituan.com/2020/11/12/java-9-cms-gc.html)

这篇文章共 2w+ 字，详细介绍了 GC 基础，总结了 CMS GC 的一些常见问题分析与解决办法。



> 更新: 2024-01-03 01:11:11  
原文: [https://www.yuque.com/vip6688/neho4x/ltxh30u3ckdl2g3i](https://www.yuque.com/vip6688/neho4x/ltxh30u3ckdl2g3i)
>



> 更新: 2024-11-25 09:18:56  
> 原文: <https://www.yuque.com/neumx/laxg2e/02911f4b7c077bc794db43fb246ee47c>