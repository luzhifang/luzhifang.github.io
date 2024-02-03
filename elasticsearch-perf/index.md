# Elasticsearch 性能优化


## 硬件配置优化

### CPU 配置

一般说来，CPU 繁忙的原因有以下几个：

- 线程中有无限空循环、无阻塞、正则匹配或者单纯的计算；
- 发生了频繁的 GC；
- 多线程的上下文切换；

大多数 Elasticsearch 部署往往对 CPU 要求不高（写入的时候比较耗 CPU）。因此，相对其它资源，具体配置多少个（CPU）不是那么关键。你应该选择具有多个内核的现代处理器，常见的集群使用 2 到 8 个核的机器。

### 内存配置

如果有一种资源是最先被耗尽的，它可能是内存。排序和聚合都很耗内存，所以有足够的堆空间来应付它们是很重要的。即使堆空间是比较小的时候，也能为操作系统文件缓存提供额外的内存。因为 Lucene 使用的许多数据结构是基于磁盘的格式，Elasticsearch 利用操作系统缓存能产生很大效果。

64 GB 内存的机器是非常理想的，但是 32 GB 和 16 GB 机器也是很常见的。少于 8 GB 会适得其反（你最终需要很多很多的小机器），大于 64 GB 的机器也会有问题。

由于 ES 构建基于 lucene，而 lucene 设计强大之处在于 lucene 能够很好的利用操作系统内存来缓存索引数据，以提供快速的查询性能。lucene 的索引文件 segements 是存储在单文件中的，并且不可变，对于 OS 来说，能够很友好地将索引文件保持在 cache 中，以便快速访问；因此，我们很有必要将一半的物理内存留给 lucene；另一半的物理内存留给 ES（JVM heap）。

#### 内存分配

当机器内存小于 64G 时，遵循通用的原则，50% 给 ES，50% 留给 lucene。

当机器内存大于 64G 时，遵循以下原则：Jvm heap 大小不要超过物理内存的 50%，最大也不要超过 32GB（compressed oop），它可用于其内部缓存的内存就越多，但可供操作系统用于文件系统缓存的内存就越少，heap 过大会导致 GC 时间过长。

#### 禁止 swap

禁止 swap，一旦允许内存与磁盘的交换，会引起致命的性能问题。可以通过在 elasticsearch.yml 中 bootstrap.memory_lock: true，以保持 JVM 锁定内存，保证 ES 的性能。

#### GC 设置

修改 jvm.options 文件

```
-XX:+UseG1GC
-XX:MaxGCPauseMillis=50
```

其中 -XX:MaxGCPauseMillis 是控制预期的最高 GC 时长，默认值为 200ms，如果线上业务特性对于 GC 停顿非常敏感，可以适当设置低一些。但是 这个值如果设置过小，可能会带来比较高的 cpu 消耗。

G1 对于集群正常运作的情况下减轻 G1 停顿对服务时延的影响还是很有效的，但是如果是你描述的 GC 导致集群卡死，那么很有可能换 G1 也无法根本上解决问题。 通常都是集群的数据模型或者 Query 需要优化。

### 磁盘

在经济压力能承受的范围下，尽量使用固态硬盘（SSD）。

使用 RAID0 是提高硬盘速度的有效途径，对机械硬盘和 SSD 来说都是如此。

## 操作系统

```
#设置虚拟内存大小，这个是MMPFILE存储内存需要;禁用内存交换
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
echo "vm.swappiness=1" >> /etc/sysctl.conf sysctl -p
#设置文件句柄数与线程限制，因为ES的索引会产生和调用大量文件数
echo "* soft nproc 131072" >> /etc/security/limits.conf
echo "* hard nproc 131072" >> /etc/security/limits.conf
echo "* soft nofile 131072" >> /etc/security/limits.conf
echo "* hard nofile 131072" >> /etc/security/limits.conf
#内存锁定交换
echo "* hard memlock unlimited" >> /etc/security/limits.conf
echo "* soft memlock unlimited" >> /etc/security/limits.conf
```

## 写入速度优化

[配置参考](https://weread.qq.com/web/reader/f9c32dc07184876ef9cdeb6k1af32e502831afa34a7fe44)

### 调整配置项 translog 的持久化策略 `index.translog.durability`

1. index.translog.durability: `request`，这是影响 ES 写入速度的最大因素。但是只有这样，写操作才有可能是可靠的。
2. index.translog.durability: `async`，设置为 async 表示 translog 的刷盘策略按 sync_interval 配置指定的时间周期进行。
3. index.translog.sync_interval: 120s，加大 translog 刷盘间隔时间。默认为 5s，不可低于 100ms。
4. index.translog.flush_threshold_size: 1024mb，超过这个大小会导致 refresh 操作，产生新的 Lucene 分段。默认值为 512MB。

### 调整配置项索引刷新间隔 `index.refresh_interval`

默认情况下索引的 refresh_interval 为 1 秒，这意味着数据写 1 秒后就可以被搜索到，每次索引的 refresh 会产生一个新的 Lucene 段，这会导致频繁的 segment merge 行为，如果不需要这么高的搜索实时性，应该降低索引 refresh 周期，例如：index.refresh_interval: 120s

### 段合并优化

segment merge 操作对系统 I/O 和内存占用都比较高，从 ES 2.0 开始，merge 行为不再由 ES 控制，而是由 Lucene 控制。

1. `index.merge.scheduler.max_thread_count`

   最大线程数 max_thread_count 的默认值如下：Math.max（1, Math.min（4, Runtime.getRuntime（）.availableProcessors（） / 2））以上是一个比较理想的值，如果只有一块硬盘并且非 SSD，则应该把它设置为 1，因为在旋转存储介质上并发写，由于寻址的原因，只会降低写入速度。

2. `index.merge.policy`

   merge 策略 index.merge.policy 有三种：

   - tiered（默认策略，推荐）；
   - log_byte_size；
   - log_doc。

索引创建时合并策略就已确定，不能更改，但是可以动态更新策略参数，可以不做此项调整。

如果堆栈经常有很多 merge，则可以尝试调整以下策略配置：index.merge.policy.segments_per_tier 该属性指定了每层分段的数量，取值越小则最终 segment 越少，因此需要 merge 的操作更多，可以考虑适当增加此值。默认为 10，其应该大于等于 index.merge.policy.max_merge_at_once。

### 减少副本数量 Elasticsearch

默认副本数量为 3 个，虽然这样会提高集群的可用性，增加搜索的并发数，但是同时也会影响写入索引的效率。

在索引过程中，需要把更新的文档发到副本节点上，等副本节点生效后在进行返回结束。使用 Elasticsearch 做业务搜索的时候，建议副本数目还是设置为 3 个，但是像内部 ELK 日志系统、分布式跟踪系统中，完全可以将副本数目设置为 1 个。

`index.merge.policy.max_merged_segment` 指定了单个 segment 的最大容量，默认为 5GB，可以考虑适当降低此值。

### indexing buffer

indexing buffer 在为 doc 建立索引时使用，当缓冲满时会刷入磁盘，生成一个新的 segment，这是除 refresh_interval 刷新索引外，另一个生成新 segment 的机会。每个 shard 有自己的 indexing buffer。

在执行大量的索引操作时，`indices.memory.index_buffer_size`的默认设置可能不够，这和可用堆内存、单节点上的 shard 数量相关，可以考虑适当**增大该值**。

### 使用 bulk 请求

建立索引的过程属于计算密集型任务，应该使用**固定大小的线程池配置**，来不及处理的任务放入队列。线程池最大线程数量应配置为 CPU 核心数+1，这也是 bulk 线程池的默认设置，可以避免过多的上下文切换。**队列大小可以适当增加，但一定要严格控制大小，过大的队列导致较高的 GC 压力，并可能导致 FGC 频繁发生**。

### 索引过程调整和优化

- **自动生成 doc ID**，通过 ES 写入流程可以看出，写入 doc 时如果外部指定了 id，则 ES 会先尝试读取原来 doc 的版本号，以判断是否需要更新。这会涉及一次读取磁盘的操作，通过自动生成 doc ID 可以避免这个环节。

- **调整字段 Mappings**

  1. 减少字段数量，对于不需要建立索引的字段，不写入 ES。
  2. 将不需要建立索引的字段 index 属性设置为 not_analyzed 或 no。对字段不分词，或者不索引，可以减少很多运算操作，降低 CPU 占用。尤其是 binary 类型，默认情况下占用 CPU 非常高，而这种类型进行分词通常没有什么意义。
  3. 减少字段内容长度，如果原始数据的大段内容无须全部建立索引，则可以尽量减少不必要的内容。
  4. 使用不同的分析器（analyzer），不同的分析器在索引过程中运算复杂度也有较大的差异。

- **调整\_source 字段**，\_source 字段用于存储 doc 原始数据，对于部分不需要存储的字段，可以通过 includes excludes 过滤，或者将\_source 禁用，一般用于索引和数据分离。
- **对 Analyzed 的字段禁用 Norms**，Norms 用于在搜索时计算 doc 的评分，如果不需要评分，则可以将其禁用：＂title＂: {＂type＂: ＂string＂,＂norms＂: {＂enabled＂: false}}
- **index_options 设置**，index_options 用于控制在建立倒排索引过程中，哪些内容会被添加到倒排索引，例如，doc 数量、词频、positions、offsets 等信息，优化这些设置可以一定程度降低索引过程中的运算任务，节省 CPU 占用率。不过在实际场景中，通常很难确定业务将来会不会用到这些信息，除非一开始方案就明确是这样设计的。

## 搜索速度优化

### 为文件系统 cache 预留足够的内存

cache 保存在系统物理内存中（线上应该禁用 swap），命中 cache 可以降低对磁盘的直接访问频率。搜索很依赖对系统 cache 的命中，如果某个请求需要从磁盘读取数据，则一定会产生相对较高的延迟。应该至少为系统 cache 预留一半的可用物理内存，更大的内存有更高的 cache 命中率。

### 使用更快的硬件

写入性能对 CPU 的性能更敏感，而搜索性能在一般情况下更多的是在于 I/O 能力，使用 SSD 会比旋转类存储介质好得多。尽量避免使用 NFS 等远程文件系统，如果 NFS 比本地存储慢 3 倍，则在搜索场景下响应速度可能会慢 10 倍左右。这可能是因为搜索请求有更多的随机访问。

如果搜索类型属于计算比较多，则可以考虑使用更快的 CPU。

### 文档模型

为了让搜索时的成本更低，文档应该合理建模。特别是应该避免 join 操作，嵌套（nested）会使查询慢几倍，父子（parent-child）关系可能使查询慢数百倍。

### 字段映射 mappings

有些字段的内容是数值，但并不意味着其总是应该被映射为数值类型，例如，一些标识符，将它们映射为 keyword 可能会比 integer 或 long 更好。

### 为只读索引执行 force-merge

为不再更新的只读索引执行 force merge，将 Lucene 索引合并为单个分段，可以提升查询速度。当一个 Lucene 索引存在多个分段时，每个分段会单独执行搜索再将结果合并，将只读索引强制合并为一个 Lucene 分段不仅可以优化搜索过程，对索引恢复速度也有好处。

基于日期进行轮询的索引的旧数据一般都不会再更新。此前的章节中说过，应该避免持续地写一个固定的索引，直到它巨大无比，而应该按一定的策略，例如，每天生成一个新的索引，然后用别名关联，或者使用索引通配符。这样，可以每天选一个时间点对昨天的索引执行 force-merge、Shrink 等操作。

### 预索引

利用查询中的模式来优化数据的索引方式。例如，如果所有文档都有一个 price 字段，并且大多数查询 range 在固定的范围列表上运行聚合，可以通过将范围预先索引到索引中并使用聚合来加快聚合速度。

### 使用 filter 代替 query

query 和 filter 的主要区别在： filter 是结果导向的而 query 是过程导向。query 倾向于“当前文档和查询的语句的相关度”而 filter 倾向于“当前文档和查询的条件是不是相符”。即在查询过程中，query 是要对查询的每个结果计算相关性得分的，而 filter 不会。另外 filter 有相应的缓存机制，可以提高查询效率。

### 避免深度分页

避免单页数据过大，可以参考百度或者淘宝的做法。es 提供两种解决方案 scroll search 和 search after。关于深度分页的详细原理，推荐阅读：详解 Elasticsearch 深度分页问题

### 使用 Keyword 类型

并非所有数值数据都应映射为数值字段数据类型。Elasticsearch 为 查询优化数字字段，例如 integeror long。如果不需要范围查找，对于 term 查询而言，keyword 比 integer 性能更好。

### 避免使用脚本

Scripting 是 Elasticsearch 支持的一种专门用于复杂场景下支持自定义编程的强大的脚本功能。相对于 DSL 而言，脚本的性能更差，DSL 能解决 80% 以上的查询需求，如非必须，尽量避免使用 Script

### 预热文件系统 cache

如果 ES 主机重启，则文件系统缓存将为空，此时搜索会比较慢。可以使用 index.store.preload 设置，通过指定文件扩展名，显式地告诉操作系统应该将哪些文件加载到内存中，

### 调节搜索请求中的 `batched_reduce_size`

该字段是搜索请求中的一个参数。默认情况下，聚合操作在协调节点需要等所有的分片都取回结果后才执行，使用 batched_reduce_size 参数可以不等待全部分片返回结果，而是在指定数量的分片返回结果之后就可以先处理一部分（reduce）。这样可以避免协调节点在等待全部结果的过程中占用大量内存，避免极端情况下可能导致的 OOM。该字段的默认值为 512，从 ES 5.4 开始支持。

### 限制搜索请求的分片数

一个搜索请求涉及的分片数量越多，协调节点的 CPU 和内存压力就越大。默认情况下，ES 会拒绝超过 1000 个分片的搜索请求。我们应该更好地组织数据，让搜索请求的分片数更少。如果想调节这个值，则可以通过 action.search.shard_count 配置项进行修改。

虽然限制搜索的分片数并不能直接提升单个搜索请求的速度，但协调节点的压力会间接影响搜索速度，例如，占用更多内存会产生更多的 GC 压力，可能导致更多的 stop-the-world 时间等，因此间接影响了协调节点的性能。

## 参考

https://weread.qq.com/web/reader/f9c32dc07184876ef9cdeb6k2023270027b202cb962a56f

https://blog.csdn.net/wlei0618/article/details/124104738

https://pdai.tech/md/db/nosql-es/elasticsearch-y-peformance.html

