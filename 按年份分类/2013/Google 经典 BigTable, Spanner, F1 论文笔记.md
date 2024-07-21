# Google 经典 BigTable/Spanner/F1 论文笔记.md

# BigTable

Bigtable 是一个分布式的结构化数据存储系统，它被设计用来处理海量数据：通常是分布在数千台普通服务器上的 PB 级的数据。Google 的很多项目使用 Bigtable 存储数据，包括 Web 索引、GoogleEarth、Google Finance。

Bigtable 是建立在其它的几个 Google 基础构件上的。BigTable 使用 Google 的分布式文件系统(GFS)存储日志文件和数据文件。BigTable 集群通常运行在一个共享的机器池中，池中的机器还会运行其它的各种各样的分布式应用程序，BigTable 的进程经常要和其它应用的进程共享机器。BigTable 依赖集群管理系统来调度任务、管理共享的机器上的资源、处理机器的故障、以及监视机器的状态。

## 系统架构

Bigtable 包括了三个主要的组件：链接到客户程序中的库、一个 Master 主服务器和多个 Tablet 片 服务器。
Bigtable 会将表（table）进行分片，片（tablet）的大小维持在 100-200MB 范围，一旦超出范围就将分裂成更小的片，或者合并成更大的片。每个片服务器负责一定量的片，处理对其片的读写请求，以及片的分裂或合并。片服务器可以根据负载随时添加和删除。这里片服务器并不真实存储数据，而相当于一个连接 Bigtable 和 GFS 的代理，客户端的一些数据操作都通过片服务器代理间接访问 GFS。主服务器负责将片分配给片服务器，监控片服务器的添加和删除，平衡片服务器的负载，处理表和列族的创建等。注意，主服务器不存储任何片，不提供任何数据服务，也不提供片的定位信息。
客户端需要读写数据时，直接与片服务器联系。因为客户端并不需要从主服务器获取片的位置信息，所以大多数客户端从来不需要访问主服务器，主服务器的负载一般很轻。
Master 服务器主要负责以下工作：为 Tablet 服务器分配 Tablets、检测新加入的或者过期失效的 Table 服务器、对 Tablet 服务器进行负载均衡、以及对保存在 GFS 上的文件进行垃圾收集。除此之外，它还处理对模式的相关修改操作，例如建立表和列族。

BigTable 还依赖一个高可用的、序列化的分布式锁服务组件，叫做 Chubby。Chubby 有五个活跃副本，同时只有一个主副本提供服务，副本之间用 Paxos 算法维持一致性，Chubby 提供了一个命名空间（包括一些目录和文件），每个目录和文件就是一个锁，Chubby 的客户端必须和 Chubby 保持会话，客户端的会话若过期则会丢失所有的锁。

## 存储结构

以一个存储 Web 网页的例子的表的片段为例：

![](https://ww1.sinaimg.cn/large/007rAy9hly1fzz6uqvl1kj30fd03sab1.jpg)

Bigtable 通过行关键字的字典顺序来组织数据。表中的每个行都可以动态分区。每个分区叫做一个”Tablet”，Tablet 是数据分布和负载均衡调整的最小单位。访问控制、磁盘和内存的使用统计都是在列族层面进行的。

行名是一个反向 URL。contents 列族存放的是网页的内容，anchor 列族存放引用该网页的锚链接文本（alex 注：如果不知道 HTML 的 Anchor，请 Google 一把）。CNN 的主页被 Sports Illustrater 和 MY-look 的主页引用，因此该行包含了名为 “anchor:cnnsi.com” 和“anchhor:my.look.ca”的列。每个锚链接只有一个版本（alex 注：注意时间戳标识了列的版本，t9 和 t8 分别标识了两个锚链接的版本）；而 contents 列则有三个版本，分别由时间戳 t3，t5，和 t6 标识。

不同版本的数据通过时间戳来索引。Bigtable 时间戳的类型是 64 位整型。
Bigtable 可以给时间戳赋值，用来表示精确到毫秒的“实时”时间；用户程序也可以给时间戳赋值。如果应用程序需要避免数据版本冲突，那么它必须自己生成具有唯一性的时间戳。数据项中，不同版本的数据按照时间戳倒序排序，即最新的数据排在最前面。为了减轻多个版本数据的管理负担，我们对每一个列族配有两个设置参数，Bigtable 通过这两个参数可以对废弃版本的数据自动进行垃圾收集。用户可以指定只保存最后 n 个版本的数据，或者只保存“足够新”的版本的数据。

## Tablet

使用一个三层的、类似Ｂ+树的结构存储 Tablet 的位置信息。

![](https://ww1.sinaimg.cn/large/007rAy9hly1fzz6uqvl1kj30fd03sab1.jpg)

第一层是一个存储在 Chubby 中的文件，它包含了 Root Tablet 的位置信息。这个 Chubby 文件属于 Chubby 服务的一部分，一旦 Chubby 不可用，就意味着丢失了 root tablet 的位置，整个 Bigtable 也就不可用了。
第二层是 root tablet。root tablet 其实是元数据表（METADATA table）的第一个分片，它保存着元数据表其它片的位置。root tablet 很特别，为了保证树的深度不变，root tablet 从不分裂。
第三层是其它的元数据片，它们和 root tablet 一起组成完整的元数据表。每个元数据片都包含了许多用户片的位置信息。

## GFS 与 SSTable

片的数据最终还是写到 GFS 里的，片在 GFS 里的物理形态就是若干个 SSTable 文件。下图展示了读写操作基本情况。

![](https://ww1.sinaimg.cn/large/007rAy9hly1fzz6uqvl1kj30fd03sab1.jpg)

SSTable 是一个持久化的、排序的、不可更改的 Map 结构，而 Map 是一个 key-value 映射的数据结构，key 和 value 的值都是任意的 Byte 串，从内部看，SSTable 是一系列的数据块（通常每个块的大小是 64KB，这个大小是可以配置的）。。SSTable 使用块索引（通常存储在 SSTable 的最后）来定位数据块；在打开 SSTable 的时候，索引被加载到内存。每次查找都可以通过一次磁盘搜索完成：首先使用二分查找法在内存中的索引里找到数据块的位置，然后再从硬盘读取相应的数据块。也可以选择把整个 SSTable 都放在内存中，这样就不必访问硬盘了。

集群包括主服务器和片服务器，主服务器负责将片分配给片服务器，而具体的数据服务则全权由片服务器负责。但是不要误以为片服务器真的存储了数据（除了内存中 memtable 的数据），数据的真实位置只有 GFS 才知道，主服务器将片分配给片服务器的意思应该是，片服务器获取了片的所有 SSTable 文件名，片服务器通过一些索引机制可以知道所需要的数据在哪个 SSTable 文件，然后从 GFS 中读取 SSTable 文件的数据，这个 SSTable 文件可能分布在好几台 chunkserver 上。

# Links

- https://www.jianshu.com/p/cbdf895f9019
- https://www.jianshu.com/p/a42dbbdf9706?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation
