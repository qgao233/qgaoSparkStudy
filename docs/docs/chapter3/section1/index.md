# RDD 概念

RDD（resilient distributed dataset ，弹性分布式数据集），是 Spark 中最基础的抽象。

它表示了一个元素集合，其性质有：

* **并行操作的**、
* **不可变的**、
* **被分区了的**。

用户不需要关心底层复杂的抽象处理，直接使用方便的**算子**处理和计算就可以了。

## 1 特点

1. **分布式**：RDD是一个抽象的概念，RDD在`spark driver`中，通过RDD来引用数据，数据真正存储在节点机的`partition`上。
2. **只读**：在Spark中RDD一旦生成了，就不能修改。 
   >那么为什么要设置为只读，设置为只读的话，因为不存在修改，并发的吞吐量就上来了。
3. **血缘关系**：我们需要对RDD进行一系列的操作，因为RDD是只读的，我们只能不断的生产新的RDD，这样，新的RDD与原来的RDD就会存在一些血缘关系。
   >Spark会记录这些血缘关系，在后期的容错上会有很大的益处。
4. **缓存** 当一个 RDD 需要被重复使用时，或者当任务失败重新计算的时候，这时如果将 RDD 缓存起来，就可以避免重新计算，保证程序运行的性能。

RDD 的缓存有3种方式：`cache`、`persist`、`checkPoint`。 
1. `cache` cache 方法不是在被调用的时候立即进行缓存，而是当**触发了 `action` 类型的算子之后，才会进行缓存**。
   1. cache 和 persist 的区别：其实 cache 底层实际调用的就是 persist 方法，只是缓存的级别默认是 MEMORY_ONLY，而 **`persist` 方法可以指定其他的缓存级别**。
   2. cache 和 checkPoint 的区别：**checkPoint 是将数据缓存到本地或者 HDFS 文件存储系统中**，当某个节点的 executor 宕机了之后，缓存的数据不会丢失，而通过 cache 缓存的数据就会丢掉。
2. `checkPoint` 的时候会把 job 从开始重新再计算一遍，因此在 checkPoint 之前最好先进行一步 cache 操作，cache 不需要重新计算，这样可以节省计算的时间。
   1. persist 和 checkPoint 的区别：persist 也可以选择将数据缓存到磁盘当中，但是它交给 blockManager 管理的，一旦程序运行结束，blockManager 也会被停止，这时候缓存的数据就会被释放掉。而 checkPoint 持久化的数据并不会被释放，是一直存在的，可以被其它的程序所使用。

## 2 核心属性

RDD 调度和计算都依赖于5个属性。

### 2.1 分区列表 

Spark RDD 是被分区的，每一个分区都会被一个计算任务 (Task) 处理，分区数决定了并行计算的数量，RDD 的并行度默认从父 RDD 传给子 RDD。

默认情况下，一个 HDFS 上的数据分片就是一个 partiton，RDD 分片数决定了并行计算的力度，可以在创建 RDD 时指定 RDD 分片个数，

如果不指定分区数量，当 RDD 从：

* 集合创建时，则默认分区数量为该程序所分配到的资源的 CPU 核数 (每个 Core 可以承载 2~4 个 partition)，
* HDFS 文件创建时，默认为文件的 Block 数。

### 2.2 依赖列表 

RDD 是 Spark 的核心数据结构，由于 RDD 每次转换都会生成新的 RDD，所以 RDD 会形成类似流水线一样的前后依赖关系，通过对 RDD 的操作形成整个 Spark 程序。

RDD 之间的依赖有两种：

* 窄依赖 ( Narrow Dependency) 
* 宽依赖 ( Wide Dependency)

当然宽依赖就不类似于流水线了，宽依赖后面的 RDD 具体的数据分片会依赖前面所有的 RDD 的所有数据分片，这个时候数据分片就不进行内存中的 Pipeline，一般都是跨机器的，因为有前后的依赖关系，所以当有分区的数据丢失时， Spark 会通过依赖关系进行重新计算，从而计算出丢失的数据，而不是对 RDD 所有的分区进行重新计算。

### 2.3 Compute函数

用于计算RDD各分区的值。

每个分区都会有计算函数， Spark 的 RDD 的计算函数是以分片为基本单位的，每个 RDD 都会实现 compute 函数，对具体的分片进行计算，RDD 中的分片是并行的，所以是**分布式并行计算**，

有一点非常重要，就是由于 RDD 有前后依赖关系，遇到宽依赖关系，如 reduce By Key 等这些操作时划分成 Stage， Stage 内部的操作都是通过 Pipeline 进行的，在具体处理数据时它会通过 Blockmanager 来获取相关的数据，

因为具体的 split 要从外界读数据，也要把具体的计算结果写入外界，所以用了一个管理器，具体的 split 都会映射成 BlockManager 的 Block，而体的 splt 会被函数处理，函数处理的具体形式是以**任务**的形式进行的。

### 2.4 分区策略

（可选） 

每个 `key-value` 形式的 RDD 都有 Partitioner 属性，它决定了 RDD 如何分区。

当然，Partiton 的个数还决定了每个 Stage 的 Task 个数。

RDD 的分片函数可以分区 ( Partitioner)，可传入相关的参数，如 Hash Partitioner 和 Range Partitioner，它本身针对 `key-value` 的形式，如果不是 `key-value` 的形式它就不会有具体的 Partitioner， 

Partitioner 本身决定了下一步会产生多少并行的分片，同时它本身也决定了当前并行 ( Parallelize) Shuffle 输出的并行数据，从而使 Spark 具有能够控制数据在不同结点上分区的特性，用户可以自定义分区策略，如 Hash 分区等。 

spark 提供了 partition By 运算符，能通过集群对 RDD 进行数据再分配来创建一个新的 RDD。

### 2.5 优先位置列表

（可选，HDFS实现数据本地化，避免数据移动）

优先位置列表会存储每个 Partition 的优先位置，对于一个 HDFS 文件来说，就是每个 Partition 块的位置。

观察运行 Spark 集群的控制台就会发现， Spark 在具体计算、具体分片以前，它已经清楚地知道任务发生在哪个结点上，也就是说任务本身是计算层面的、代码层面的，代码发生运算之前它就已经知道它要运算的数据在什么地方，有具体结点的信息。这就符合大数据中**数据不动代码动**的原则。

数据不动代码动的最高境界是数据就在当前结点的内存中。这时候有可能是 Memory 级别或 Tachyon 级别的， Spark 本身在进行任务调度时会尽可能地将任务分配到处理数据的数据块所在的具体位置。
>据 Spark 的 RDD。 Scala 源代码函数 getParferredlocations 可知，每次计算都符合完美的数据本地性。