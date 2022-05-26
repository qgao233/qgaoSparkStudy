# RDD 数据集

## 1 maven 配置

```
<!-- 新增 -->
<dependency> 
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.12</artifactId>
    <version>2.4.2</version>
    <scope>provided</scope>
</dependency>
<dependency> 
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version><your-hdfs-version></version>
</dependency>
```

## 2 初始化spark

Spark程序必须做的第一件事是创建一个JavaSparkContext对象，它告诉Spark如何访问集群。

要创建SparkContext，首先需要构建包含应用程序信息的SparkConf对象。

```
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.SparkConf;

SparkConf conf = new SparkConf().setAppName(appName).setMaster(master);
JavaSparkContext sc = new JavaSparkContext(conf);
```

### 2.1 RDD数据集

Spark围绕弹性分布式数据集(RDD)的概念展开，RDD是一个可以并行操作的容错元素集合。

有2种创建rdd的方法:

* 在驱动程序中并行化一个现有的集合，
* 或者在外部存储系统中引用一个数据集，例如共享文件系统、HDFS、HBase或任何提供Hadoop InputFormat的数据源。

#### 2.1.1 并行集合

通过在驱动程序中调用JavaSparkContext的`parallelize`方法可以创建并行集合。集合的元素被复制，形成一个可以并行操作的分布式数据集。

例如，下面是如何创建一个包含数字1到5的并行集合:

```
List<Integer> data = Arrays.asList(1, 2, 3, 4, 5);
JavaRDD<Integer> distData = sc.parallelize(data);
```

一旦创建了分布式数据集`distData`，就可以并行操作它了。例如，我们可以调用`distData.reduce ((a, b) -> a + b)`将列表中的元素相加。

>稍后将描述对分布式数据集的操作。

并行集合的一个重要参数是将数据集分割成多个**分区**。Spark将为集群的每个分区运行一个任务。

>通常，集群中的每个CPU需要2-4个分区。

通常，Spark会根据集群自动设置分区数量。但是，你也可以通过将它作为第二个参数传入parallelize来手动设置它(例如`sc.parallelize(data, 10))`。注意:代码中的某些地方使用**术语片**(分区的同义词)来保持向后兼容性。

#### 2.1.2 外部数据集

Spark可以从Hadoop支持的任何存储源创建分布式数据集，包括：
* 本地文件系统、
* HDFS、
* Cassandra、
* HBase、
* Amazon S3等。

Spark支持文本文件、SequenceFiles和任何其他Hadoop InputFormat。

文本文件rdd可以使用SparkContext的`textFile`方法创建。该方法接受文件的URI(机器上的本地路径，或者hdfs://， s3a://等URI)，并将其作为**行集合**读取。下面是一个示例调用:

```
JavaRDD<String> distFile = sc.textFile("data.txt");
```

我们可以使用`map`和`reduce`操作将所有行的大小相加，如下所示:`map(s - > s.length()).reduce((a, b) -> a + b)`

#### 2.1.3 外部数据集API

Spark读取文件的一些注意事项: 如果使用本地文件系统上的路径，则必须在工作节点上的相同路径上访问该文件。

* 要么将文件复制到所有工作人员，
* 要么使用网络挂载的共享文件系统。

所有Spark基于文件的输入方法，包括`textFile`，都支持在目录、压缩文件和通配符上运行。

例如，可以使用

* textFile("/my/directory")、
* textFile("/my/directory/.txt")
* textFile("/my/directory/.gz")。

textFile方法还接受第二个可选参数，用于控制文件的分区数。

默认情况下，Spark为文件的每个块创建一个分区(HDFS默认为128MB)，但你也可以通过传递一个更大的值来请求更多的分区。注意，分区不能少于块。

---

除了文本文件，Spark的Java API还支持其他几种数据格式: 

1. **wholeTextFiles**：`JavaSparkContext.wholeTextFiles`允许您读取包含多个小文本文件的目录，并以(文件名，内容)对的形式返回每个小文本文件。这与textFile相反，textFile在每个文件中每行返回一条记录。
2. **SequenceFiles**：使用`SparkContext.sequenceFile[K, V]`方法，其中K和V是文件中的键和值类型。这些应该是Hadoop的Writable接口的子类，比如IntWritable和Text。
3. **其他Hadoop InputFormats**：使用`JavaSparkContext.hadoopRDD`，它接受一个任意的JobConf和输入格式类、键类和值类。设置这些参数的方法与使用输入源设置Hadoop作业的方法相同。你也可以使用JavaSparkContext.newAPIHadoopRDD用于InputFormats，基于“新的”MapReduce API (org.apache.hadoop.mapreduce)。
4. **JavaSparkContext.objectFile**：`JavaRDD.saveAsObjectFile`，objectFile支持以由序列化的Java对象组成的简单格式保存RDD。虽然这种格式不如Avro这样的专用格式有效，但它提供了一种保存任何RDD的简单方法。