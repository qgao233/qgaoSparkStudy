# spark介绍

## 1 hadoop MapReduce框架局限性

1. 处理效率低效 
   * 1.1 Map 结果写磁盘，Reduce 写HDFS ，多个MR 之间通过HDFS 交换数据 ； 
   * 1.2 任务调度和启动开销大 ； 
   * 1.3 无法充分利用内存
2. 不适合迭代计算（如机器学习 、 图计算等 ）， 交互式处理（数据挖掘 ）
3. 不适合流式处理（点击日志分析 ）
4. MapReduce编程不够灵活，仅支持Map 和Reduce 两种操作

因此需要一种灵活的框架可同时进行**批处理**、**流式计算**、**交互式计算**。

于是引入了`Spark`计算引擎。

## 2 spark 核心模块

Apache Spark是一个用于大规模数据处理的统一分析引擎。

它提供了`Java`、`Scala`、`Python`和`R`的高级api，以及支持通用执行图的优化引擎。

1. spark core：核心提供基础
2. spark sql：处理结构化数据操作
3. spark stream：处理增量计算和流式数据操作
4. spark mLib：机器学习相关
5. spark graphX：图形挖掘计算

## 3 Spark的好处

### 3.1 优势

* 内存计算引擎 ， 提供Cache 机制来支持需要反复迭代计算或者多次数据共享 ， 减少数据读取的IO 开销
* DAG 引擎 ， 减少多次计算之间中间结果写到HDFS 的开销
* 使用多线程池模型来减少task 启动开稍 ，shuffle 过程中避免不必要的sort 操作以及减少磁盘IO

### 3.2 特点

* 易用：提供了丰富的API ， 支持Java ，Scala ，Python 和R 四种语言 R语言很少被用到，基本都是使用Java、Scala、Python来操作Spark
* 代码量比MapReduce 少2~5 倍
* 与Hadoop 集成 读写HDFS/Hbase 与YARN 集成

### 3.3 新特性

* SparkSession：新的上下文入口，统一SQLContext和HiveContext
* dataframe与dataset统一，dataframe只是dataset[Row]的类型别名。由于Python是弱类型语言，只能使用DataFrame
* Spark SQL 支持sql 2003标准
* 支持ansi-sql
* 支持ddl命令
* 支持子查询：in/not in、exists/not exists
* 提升catalyst查询优化器的性能