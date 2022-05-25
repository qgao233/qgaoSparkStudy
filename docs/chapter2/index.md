# spark-java 运行流程

## 1 maven配置

找java的spark。

```
<!-- Spark dependency -->
<dependency> 
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.12</artifactId>
    <version>2.4.2</version>
    <scope>provided</scope>
</dependency>
```

## 2 java程序

计算在`idcard.txt`中包含' 1 '和包含' 2 '的行数。

```
package org.example;

import org.apache.spark.api.java.function.FilterFunction;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.Dataset;

public class SparkTest1 {
    public static void main(String[] args){
        String logFile = "file:///home/pyspark/idcard.txt"; // Should be some file on your system
        SparkSession spark = SparkSession.builder().appName("Simple Application").getOrCreate();
        Dataset<String> logData = spark.read().textFile(logFile).cache();

        long numAs = logData.filter((FilterFunction<String>) s -> s.contains("1")).count();
        long numBs = logData.filter((FilterFunction<String>) s -> s.contains("2")).count();

        System.out.println("Lines with 1: " + numAs + ", lines with 2: " + numBs);

        spark.stop();
    }
}
```

## 3 打包到spark环境

`mvn clean package`到`target`目录下，将`SparkStudy-1.0-SNAPSHOT.jar`包复制到`spark`环境中，然后执行命令：

```
spark-submit \
  --class org.example.SparkTest1 \
  --master local[2] \
  /home/javaspark/SparkStudy-1.0-SNAPSHOT.jar
```

