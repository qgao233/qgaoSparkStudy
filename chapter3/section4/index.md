# RDD 操作实例

## 1 初始化RDD

### 1.1 通过集合创建RDD

**java代码**

```
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.SparkConf;
import java.util.Arrays;
import java.util.List;

public class RDDTest1 {
    public static void main(String[] args){
        SparkConf conf = new SparkConf().setAppName("RDDTest1").setMaster("local[*]");
        JavaSparkContext sc = new JavaSparkContext(conf);

        //通过集合创建rdd
        List<Integer> data = Arrays.asList(1, 2, 3, 4, 5);
        JavaRDD<Integer> rdd = sc.parallelize(data);

        //查看list被分成了几部分
        System.out.println(rdd.getNumPartitions() + "\n");
        //查看分区状况
        System.out.println(rdd.glom().collect() + "\n");
        System.out.println(rdd.collect() + "\n");

        sc.stop();
    }
}
```

**spark环境运行**

```
spark-submit \
   --class org.example.RDDTest1 \
   --master local[2] \
   /home/javaspark/SparkStudy-1.0-SNAPSHOT.jar
```

**运行结果**

```
4
[[1], [2], [3], [4, 5]]
[1, 2, 3, 4, 5]
```

### 1.2 通过文件创建rdd

读取一个`idcard.txt`，获取年龄和性别。

**java代码**

```
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.function.VoidFunction;

import java.util.Calendar;

public class RDDTest2 {
    public static void main(String[] args){
        SparkConf conf = new SparkConf().setAppName("RDDTest2").setMaster("local[2]");
        JavaSparkContext sc = new JavaSparkContext(conf);

        JavaRDD<String> rdd1 = sc.textFile("file:///home/pyspark/idcard.txt");

        rdd1.foreach(new VoidFunction<String>() {
            @Override
            public void call(String s) throws Exception {
                //System.out.println(s + "\n");

                Calendar cal = Calendar.getInstance();
                int yearNow = cal.get(Calendar.YEAR);
                int monthNow = cal.get(Calendar.MONTH)+1;
                int dayNow = cal.get(Calendar.DATE);

                int year = Integer.valueOf(s.substring(6, 10));
                int month = Integer.valueOf(s.substring(10,12));
                int day = Integer.valueOf(s.substring(12,14));

                String age;
                if ((month < monthNow) || (month == monthNow && day<= dayNow) ){
                    age = String.valueOf(yearNow - year);
                }else {
                    age = String.valueOf(yearNow - year-1);
                }

                int sexType = Integer.valueOf(s.substring(16).substring(0, 1));
                String sex = "";
                if (sexType%2 == 0) {
                    sex = "女";
                } else {
                    sex = "男";
                }

                System.out.println("年龄: " + age + "," + "性别:" + sex + ";");

            }
        });

        sc.stop();
    }
}
```

**运行结果**

```
年龄: 65,性别:男;
年龄: 51,性别:男;
年龄: 52,性别:男;
年龄: 40,性别:男;
年龄: 42,性别:男;
年龄: 52,性别:男;
年龄: 52,性别:女;
年龄: 30,性别:男;
年龄: 45,性别:男;
年龄: 57,性别:女;
年龄: 41,性别:男;
```

## 2 RDD的map操作

求txt文档文本总长度

**java代码**

```
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.VoidFunction;

public class RDDTest3 {
    public static <integer> void main(String[] args){
        SparkConf conf = new SparkConf().setAppName("RDDTest3").setMaster("local[2]");
        JavaSparkContext sc = new JavaSparkContext(conf);

        JavaRDD<String> rdd1 = sc.textFile("file:///home/pyspark/idcard.txt");
        JavaRDD<Integer> rdd2 = rdd1.map(s -> s.length());  //RDD使用函数：s.length()=>len(s)
        Integer total = rdd2.reduce((a, b) -> a + b);
        System.out.println("\n" + "\n" + "\n");
        System.out.println(total);
    }

    public static int len(String s){
        int str_length;
        str_length = s.length();
        return str_length;
    }
}
```

**运行结果**

```
[18, 18, 18, 18, 18, 18, 18, 18, 18, 18, 18]
198
```