# 1 es相关问题

## 1.1 搜索引擎提供的能力

为产品其他应用提供全文检索,定制排序规则,向量检索,智能推荐等功能

## 1.2 es提供的能力

### 1.2.1 主要特点

1. 分布式与可扩展性
2. 实时性能,存储文档时，会在1 秒内近乎实时地为其建立索引并完全可搜索
3. 强大的查询和聚合能力,是用户能够执行复杂的查询和数据分析
4. 文档导向,以文档为单位进行数据存储和检索

# 2 es的分布式与扩展性原理

Elasticsearch 索引实际上是一个或多个物理分片的逻辑分组，其中每个分片实际上是一个独立的索引。通过将索引中的文档分布在多个分片上，并将这些分片分布在多个节点上，Elasticsearch 可以确保备份，这既可以防止硬件故障，又可以在将节点添加到集群时提高查询容量。随着集群的增长（或缩小），Elasticsearch 会自动迁移分片以重新平衡集群。

## 2.1 分片

es有两种类型的分片：主分片和副本分片。索引中的每个文档都属于一个主分片。副本分片是主分片的副本。副本提供数据的冗余副本，以防止硬件故障并提高服务读取请求（例如搜索或检索文档）的能力。

索引中主分片的数量在创建索引时是固定的，但副本分片的数量可以随时更改，而无需中断索引或查询操作

作为惯例,es的每个分片的数据量大小应在几G到十几G之间;另一个参考标准是:避免无数碎片问题。节点可以的分片数量与可用堆空间成正比。作为一般规则，每 G 堆空间的分片数量应小于 20

### 2.1.1 分片自平衡

分片子平衡触发

- 集群节点当机
- 增加副本
- 索引的动态均衡,包括集群内部的借点数量调整 删除索引副本,删除索引等

自平衡策略由以下配置项触发

- 分片因子
- 索引因子
- 阈值

## 2.2 ELK及衍生方案

elk是一个日志集中式日志收集处理解决方案；能够完成日志的收集，数据清洗，存储与索引，分析等；发展至今已有不少优化替换方案

- 收集：Logstash，FileBeats，Elastic Agent

- 数据清洗：Logstash（filter），FileBeats（Beats processors），Ingest pipelines

- 存储与索引：Elastic Search

- 分析：Kibana

### 2.2.1 各组件简单介绍

#### 2.2.1.1 Logstash

Logstash 是一款基于插件的数据收集和处理引擎。Logstash 配有大量的插件，以便人们能够轻松进行配置以在多种不同的架构中收集、处理并转发数据。

处理过程可分为一个或多个管道。在每个管道中，会有一个或多个[输入插件](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)接收或收集数据，然后这些数据会加入内部队列。默认情况下，这些数据很少并且会存储于内存中，但是为了提高可靠性和弹性，也可进行配置以[扩大规模并长期存储在磁盘上](https://www.elastic.co/guide/en/logstash/6.2/persistent-queues.html)。

![image-1-logstash-processing-pipeline.png](https://images.contentstack.io/v3/assets/bltefdd0b53724fa2ce/bltb24f0ba0dd34b37a/5c583f05516e21cf0b2a1218/image-1-logstash-processing-pipeline.png)

处理线程会以小批量的形式从队列中读取数据，并通过任何配置的[过滤插件](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html)按顺序进行处理。Logstash 自带大量的插件，能够满足特定类型的操作需要，也就是解析、处理并丰富数据的过程。

处理完数据之后，处理线程会将数据发送到对应的输出插件，这些输出插件负责对数据进行格式化并进一步发送数据（例如发送到 Elasticsearch）。

输入和输出插件也可以配置 [codec 插件](https://www.elastic.co/guide/en/logstash/6.2/pipeline.html#_codecs)。这样便可以在将数据添加到内部队列或发送到输出插件之前对数据进行解析和/或格式化。

**输入**

常用插件有file，beats，redis，kafka，Logstash等

[Input plugins | Logstash Reference [8.12] | Elastic](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)

**过滤器**

常用插件有grok、date、mutate、mutiline等

[Filter plugins | Logstash Reference [8.12] | Elastic](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html)

##### 2.2.1.1.1 输出

常用插件为elasticsearch，redis，kafka，Logstash等

[Output plugins | Logstash Reference [8.12] | Elastic](https://www.elastic.co/guide/en/logstash/current/output-plugins.html)

##### 2.2.1.1.2 Persistent Queue

###### 2.2.1.1.2.1 PQ介绍

PQ，也即永久队列。默认情况下，Logstash 在input和 filter + output 管道阶段之间使用内存中排队。 这使它可以吸收负载中的小峰值，而无需保持将事件推入 Logstash 的连接，但是缓冲区仅限于内存容量。 如果 Logstash 发生故障，则正在处理的运行中和内存中数据将丢失。

为了提供不限于内存容量的缓冲区，并防止异常终止期间的数据丢失，Logstash 引入了持久队列功能。 它将输入数据作为自适应缓冲区存储在磁盘上，并在处理完成后将流水线确认送回到持久队列。 重新启动 Logstash 时，将重播任何未确认的数据（未完全处理和输出）。

总而言之，启用持久队列的好处如下：

- 无需大量的外部缓冲机制（例如Redis或Apache Kafka）即可吸收突发事件。

- 提供一次至少一次传递保证，以防止在正常关闭以及Logstash异常终止时丢失消息。 如果在发生事件时重新启动Logstash，则Logstash将尝试传递存储在持久队列中的消息，直到传递至少成功一次。

![](https://img-blog.csdnimg.cn/20200730183745748.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1VidW50dVRvdWNo,size_16,color_FFFFFF,t_70)

###### 2.2.1.1.2.2 初步设计目标

**至少一次交付**：PQ设计的主要目标是在出现异常故障时避免数据丢失，并旨在提供至少一次交付的保证。至少一次交付意味着发生故障时，重新启动 Logstash 时将重播所有未确认的数据。根据故障情况，“至少一次” 还意味着可以重播已处理但未确认的数据，并导致重复处理。这是至少一次的权衡；以无故障的情况下避免数据丢失为代价，可能会有重复数据的代价；重复数据可以文件的fingerprint与其他数据信息生成es的doc _id来处理

**Spooling**：PQ 还可在磁盘上增大大小，并在不对输入施加反压的情况下帮助处理数据摄取峰值或输出减慢。在这两种情况下，输入或摄取端的处理速度都比 Logstash 的过滤和输出端更快。使用 PQ，传入的数据将在磁盘上本地缓冲，直到过滤和输出阶段能够赶上。对于使用外部排队层处理吞吐量峰值和减轻背压的现有用户，PQ 现在能够自然满足这些要求。因此，用户可以选择删除此外部队列，以简化总体摄取架构。

###### 2.2.1.1.2.3 弹性和耐久性

有3种类型的故障会影响Logstash：

1. 应用程序/进程级别崩溃/中断的关机。这是 PQ 帮助解决潜在数据丢失的最常见原因。当 Logstash 异常崩溃或在进程级别被杀死或通常以防止安全关闭的方式被中断时，会发生这种情况。使用内存队列时，这通常会导致数据丢失。使用 PQ，无论持久性设置如何（请参阅 queue.checkpoint.writes 设置），都不会丢失任何数据，并且在重新启动 Logstash 时将重播任何未确认的数据。

2. 操作系统级崩溃：内核崩溃或计算机重新启动。使用 PQ 默认设置，可能会丢失在上一次 fsync 和 OS 级崩溃之间没有安全地写入磁盘（fsync）的数据。可以更改此行为并将 PQ 配置为具有最大持久性，其中每个传入事件都将被强制写入磁盘（fsync）。请参阅 “Durability” 部分和queue.checkpoint.writes设置。

3. 存储硬件故障。 PQ 不提供复制来帮助解决存储硬件故障。防止磁盘故障时数据丢失的唯一方法是在云环境中使用 RAID 等外部复制技术或等效的冗余磁盘选件。

###### 2.2.1.1.2.4 耐用性

 PQ 使用大小为 queue.page_capacity（默认为250mb）的页面文件来写入输入数据。 最新页面称为首页，是仅以追加方式写入数据的页面。 当首页到达 queue.page_capacity 时，它变成不可变的尾页，并创建一个新的首页以写入新数据。首页的持久性特征是使用 queue.checkpoint.writes 设置控制的。 默认情况下，PQ 将在每 1024 个收到的输入事件时强制将数据写入磁盘，以减少 OS 级崩溃时潜在的数据丢失。 请注意，无论 queue.checkpoint.writes 设置如何，应用程序和进程级别的崩溃或中断的关闭都绝不会导致数据丢失。 如果要防止操作系统级别崩溃的潜在数据丢失，可以使用 queue.checkpoint.writes:1 设置将 PQ 设置为最大持久性，但是请注意，这将导致 IO 性能显着降低。

## 2.3 生命周期管理与数据分层

<br/>

https://www.elastic.co/cn/blog/implementing-hot-warm-cold-in-elasticsearch-with-index-lifecycle-management

<br/>

可搜索快照

<br/>

bkd tree

es的底层结构

<br/>

es提供的数据类型与底层实现

knn的原理

向量的计算

nested对性能的影响

join类型的优缺点,执行检索的原理是什么

rescore对性能的影响

为什么选用hanlp分词
