# HBase 与 MapReduce 整合

> phoenix.apache.org

## Mapreduce运行3种方式

本地方式运行：

### pc环境

1.1、将 Hadoop安装本地解压
1.2、配置 Hadoop的环境变量
添加%HADOOP_HOME%
修改%PATH%添加%HADOOP_HOME%/bin;%HADOOP_HOME%/sbin
3、在解压的 Hadoop的bin目录下添加 winutils.exe工具
Java工程
2.1、jdk一定要使用自己的jdk、不要使用 eclipse自带
2.2、根目录(src目录下)，不要添加任何 Mapreduce的配置文件hdfs-site.xml yarn-site.xml core-site.xml mapred-site.xml
2.3、在代码当中，通过conf.set方式来进行指定。conf set("fs.defaults","hdfs://nodel:8020");
2.4、修改 Hadoop源码
3、右键run执行

### 集群运行两种方式

#### Java工程

1、根目录(Src目录下)，添加 Hadoop的配置文件hdfs-site.xm1 yarn-Site.xm1 core-site.xm1 mapped-site.xm1
2、在代码当中，指定jar包的位置， config.set(" mapped.jar"，"D:\\WR\wc.jar")；
3、修改 Hadoop源码
4、将工程打jar包
5、右键run执行

#### Java工程

根目录(src目录下)，添加 Hadoop的配置文件hdfs-site.xm1 yarn-Site.xm1 core-site.xm1 mapped-site.xm1
2、将工程打jar包
手动将jar包上传到集群当中
4、通过 hadoop命令来运行。 hadoop jar jar位置mr代码入口(例如： hadoop jar/usr/wc.Jar com.sxt.mr.Wcjob)
在代码当中指定 Hbase所使用的 Zookeeper集群

(注意：如果 hbase搭建的是仍分布式，那么对应的 Zookeeper就是那台伪分布式的服务器
conf.set("hbase.zookeeper.quorum","nodel, node 2, node3")
System.setproperty("HADOOP_USER_NAME,"root");



## HBase与MR整合文档

[官方文档](https://hbase.apache.org/book.html#mapreduce)

### 49. MapReduce扫描缓存

TableMapReduceUtil现在恢复在传入的Scan对象上设置扫描程序缓存（将结果返回给客户端之前缓存的行数）的选项。由于HBase 0.95（[HBASE-11558](https://issues.apache.org/jira/browse/HBASE-11558)）中的错误，此功能丢失了，对于HBase 0.98.5和0.96.3是固定的。选择扫描仪缓存的优先顺序如下：

1. 在扫描对象上设置的缓存设置。
2. 通过配置选项指定的缓存设置`hbase.client.scanner.caching`，可以在*hbase-site.xml中*手动设置，也可以通过helper方法设置`TableMapReduceUtil.setScannerCaching()`。
3. 默认值`HConstants.DEFAULT_HBASE_CLIENT_SCANNER_CACHING`，设置为`100`。

优化缓存设置是客户端等待结果的时间与客户端需要接收的结果集数量之间的平衡。如果缓存设置太大，则客户端可能会等待很长时间，甚至请求可能会超时。如果设置太小，则扫描需要分多次返回结果。如果您将扫描视为铲子，则较大的缓存设置类似于较大的铲子，较小的缓存设置等效于进行更多铲斗以填充铲斗。

上面提到的优先级列表允许您设置一个合理的默认值，并为特定操作覆盖它。

有关更多详细信息，请参见[Scan](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/client/Scan.html)的API文档。

### 50.捆绑的HBase MapReduce作业

HBase JAR还可以用作某些捆绑的MapReduce作业的驱动程序。要了解捆绑的MapReduce作业，请运行以下命令。

```
$ ${HADOOP_HOME}/bin/hadoop jar ${HBASE_HOME}/hbase-mapreduce-VERSION.jar
An example program must be given as the first argument.
Valid program names are:
  copytable: Export a table from local cluster to peer cluster
  completebulkload: Complete a bulk data load.
  export: Write table data to HDFS.
  import: Import data written by Export.
  importtsv: Import data in TSV format.
  rowcounter: Count rows in HBase table
```

每个有效的程序名称都是捆绑的MapReduce作业。要运行作业之一，请在以下示例之后对命令建模。

```
$ ${HADOOP_HOME}/bin/hadoop jar ${HBASE_HOME}/hbase-mapreduce-VERSION.jar rowcounter myTable
```

### 51. HBase作为MapReduce作业数据源和数据接收器

HBase可用作MapReduce作业的数据源[TableInputFormat](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/TableInputFormat.html)和数据接收器[TableOutputFormat](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/TableOutputFormat.html)或[MultiTableOutputFormat](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/MultiTableOutputFormat.html)。编写MapReduce作业以读取或写入HBase时，建议将[TableMapper](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/TableMapper.html) 和/或[TableReducer](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/TableReducer.html)子类[化](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/TableReducer.html)。有关基本用法，请参见无作用传递类[IdentityTableMapper](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/IdentityTableMapper.html)和[IdentityTableReducer](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/IdentityTableReducer.html)。有关更多示例，请参阅[RowCounter](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/RowCounter.html)或查看`org.apache.hadoop.hbase.mapreduce.TestTableMapReduce`单元测试。

如果运行使用HBase作为源或接收器的MapReduce作业，则需要在配置中指定源和接收器表及列的名称。

当您从HBase读取数据时，会从HBase `TableInputFormat`请求区域列表，并创建一个地图，该地图可以是`map-per-region`或`mapreduce.job.maps`，以较小者为准。如果您的工作只有两张地图，请加薪`mapreduce.job.maps`数量大于区域数量。如果您在每个节点上运行TaskTracer / NodeManager和RegionServer，则地图将在相邻的TaskTracker / NodeManager上运行。写入HBase时，应避免执行Reduce步骤，然后从地图中写回HBase。当您的工作不需要MapReduce对地图发出的数据执行的排序和排序规则时，此方法有效。插入时，HBase会进行“排序”，因此除非有必要，否则就不会进行点双重排序（并在MapReduce集群周围进行数据改组）。如果不需要精简，则地图可能会在作业结束时发出为报告而处理的记录计数，或者将精简数量设置为零并使用TableOutputFormat。如果根据您的情况运行“减少”步骤，

一个新的HBase分区程序[HRegionPartitioner](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/HRegionPartitioner.html)可以运行与现有区域数一样多的reducer。HRegionPartitioner适用于表较大且上传完成后不会大大改变现有区域数的情况。否则，请使用默认分区程序。

### 52.在批量导入期间直接写入HFile

如果要导入到新表中，则可以绕过HBase API并将内容直接写入文件系统，并格式化为HBase数据文件（HFiles）。您的导入将运行得更快，也许要快一个数量级。有关此机制如何工作的更多信息，请参见[批量加载](https://hbase.apache.org/book.html#arch.bulk.load)。

### 53. RowCounter示例

包含的[RowCounter](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/RowCounter.html) MapReduce作业使用`TableInputFormat`并计算指定表中的所有行。要运行它，请使用以下命令：

```
$ ./bin/hadoop jar hbase-X.X.X.jar
```

这将调用HBase MapReduce驱动程序类。选择`rowcounter`从提供的职位选择。这会将行计数器使用建议打印到标准输出。指定表名，要计数的列和输出目录。如果您遇到类路径错误，请参见[HBase，MapReduce和CLASSPATH](https://hbase.apache.org/book.html#hbase.mapreduce.classpath)。

### 54.Map-Task Splitting

#### 54.1。默认的HBase MapReduce拆分器

当使用[TableInputFormat](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/TableInputFormat.html)来在MapReduce作业中获取HBase表时，其拆分器将为该表的每个区域创建一个映射任务。因此，如果表中有100个区域，则该作业将有100个映射任务-无论在“扫描”中选择了多少列族。

#### 54.2。定制分离器

对于那些有兴趣在实现自定义的分离器，看到法`getSplits`中[TableInputFormatBase](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/TableInputFormatBase.html)。这就是映射任务分配的逻辑所在。

## HBase MapReduce Examples

### HBase MapReduce阅读示例

以下是以只读方式将HBase用作MapReduce源的示例。具体来说，有一个Mapper实例，但没有Reducer，并且没有从Mapper发出任何东西。该工作的定义如下...

```java
Configuration config = HBaseConfiguration.create();
Job job = new Job(config, "ExampleRead");
job.setJarByClass(MyReadJob.class);     // class that contains mapper

Scan scan = new Scan();
scan.setCaching(500);        // 扫描中的默认值为1，这对MapReduce作业不利
scan.setCacheBlocks(false);  // MR工作不要设置为true
// 设置其他scan属性
...

TableMapReduceUtil.initTableMapperJob(
  tableName,        // 输入HBase表名称
  scan,             // scan 实例以控制CF和属性
  MyMapper.class,   // mapper
  null,             // mapper output key
  null,             // mapper output value
  job);
job.setOutputFormatClass(NullOutputFormat.class);   //因为我们没有从mapper发出任何东西

boolean b = job.waitForCompletion(true);
if (!b) {
  throw new IOException("error with job!");
}
```

...而mapper实例将扩展[TableMapper](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/TableMapper.html) ...

```java
public static class MyMapper extends TableMapper<Text, Text> {

  public void map(ImmutableBytesWritable row, Result value, Context context) throws InterruptedException, IOException {
    //处理来自Result实例的行的数据。
   }
}
```

### HBase MapReduce读/写示例

以下是通过MapReduce将HBase用作源和接收器的示例。此示例将简单地将数据从一个表复制到另一个表。

```java
Configuration config = HBaseConfiguration.create();
Job job = new Job(config,"ExampleReadWrite");
job.setJarByClass(MyReadWriteJob.class);    // class that contains mapper

Scan scan = new Scan();
scan.setCaching(500);        // 1 is the default in Scan, which will be bad for MapReduce jobs
scan.setCacheBlocks(false);  // don't set to true for MR jobs
// set other scan attrs

TableMapReduceUtil.initTableMapperJob(
  sourceTable,      // input table
  scan,             // Scan instance to control CF and attribute selection
  MyMapper.class,   // mapper class
  null,             // mapper output key
  null,             // mapper output value
  job);
TableMapReduceUtil.initTableReducerJob(
  targetTable,      // output table
  null,             // reducer class
  job);
job.setNumReduceTasks(0);

boolean b = job.waitForCompletion(true);
if (!b) {
    throw 
```

需要说明`TableMapReduceUtil`正在做什么，尤其是对于减速器。[TableOutputFormat](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/TableOutputFormat.html)被用作outputFormat类，并且在配置（例如`TableOutputFormat.OUTPUT_TABLE`）上设置了几个参数，并将reducer输出键设置为`ImmutableBytesWritable`，reducer值设置为`Writable`。这些可以由程序员在工作和配置上设置，但是`TableMapReduceUtil`试图使事情变得更容易。

以下是示例映射器，它将创建一个`Put`与输入匹配的输出`Result`。注意：这就是CopyTable实用程序的作用。

```java
public static class MyMapper extends TableMapper<ImmutableBytesWritable, Put>  {

  public void map(ImmutableBytesWritable row, Result value, Context context) throws IOException, InterruptedException {
    // 这个例子只是从源表中复制数据..
      context.write(row, resultToPut(row,value));
    }

    private static Put resultToPut(ImmutableBytesWritable key, Result result) throws IOException {
      Put put = new Put(key.get());
      for (Cell cell : result.listCells()) {
        put.add(cell);
      }
      return put;
    }
}
```

实际上没有reducer步骤，因此请`TableOutputFormat`注意将其发送`Put`到目标表。

这只是一个示例，开发人员可以选择不使用`TableOutputFormat`并自己连接到目标表。

### 具有多表输出的HBase MapReduce读/写示例

TODO：的示例`MultiTableOutputFormat`。

### HBase MapReduce汇总到HBase示例

以下示例将HBase用作MapReduce源和接收器，并进行汇总。本示例将对一个表中某个值的不同实例的数量进行计数，并将这些汇总计数写入另一个表中。

```java
Configuration config = HBaseConfiguration.create();
Job job = new Job(config,"ExampleSummary");
job.setJarByClass(MySummaryJob.class);     // class that contains mapper and reducer

Scan scan = new Scan();
scan.setCaching(500);        // 1 is the default in Scan, which will be bad for MapReduce jobs
scan.setCacheBlocks(false);  // don't set to true for MR jobs
// set other scan attrs

TableMapReduceUtil.initTableMapperJob(
  sourceTable,        // input table
  scan,               // Scan instance to control CF and attribute selection
  MyMapper.class,     // mapper class
  Text.class,         // mapper output key
  IntWritable.class,  // mapper output value
  job);
TableMapReduceUtil.initTableReducerJob(
  targetTable,        // output table
  MyTableReducer.class,    // reducer class
  job);
job.setNumReduceTasks(1);   // at least one, adjust as required

boolean b = job.waitForCompletion(true);
if (!b) {
  throw new IOException("error with job!");
}
```

在此示例映射器中，选择具有字符串值的列作为要汇总的值。该值用作从映射器发出的键，并且一个`IntWritable`代表实例计数器。

```java
public static class MyMapper extends TableMapper<Text, IntWritable>  {
  public static final byte[] CF = "cf".getBytes();
  public static final byte[] ATTR1 = "attr1".getBytes();

  private final IntWritable ONE = new IntWritable(1);
  private Text text = new Text();

  public void map(ImmutableBytesWritable row, Result value, Context context) throws IOException, InterruptedException {
    String val = new String(value.getValue(CF, ATTR1));
    text.set(val);     // we can only emit Writables...
    context.write(text, ONE);
  }
}
```

In the reducer, the "ones" are counted (just like any other MR example that does this), and then emits a .

```java
public static class MyTableReducer extends TableReducer<Text, IntWritable, ImmutableBytesWritable>  {
  public static final byte[] CF = "cf".getBytes();
  public static final byte[] COUNT = "count".getBytes();

  public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
    int i = 0;
    for (IntWritable val : values) {
      i += val.get();
    }
    Put put = new Put(Bytes.toBytes(key.toString()));
    put.add(CF, COUNT, Bytes.toBytes(i));

    context.write(null, put);
  }
}
```

### HBase MapReduce摘要文件示例

这与上面的摘要示例非常相似，不同的是，它使用HBase作为MapReduce源，但使用HDFS作为接收器。区别在于作业设置和减速机。映射器保持不变。

```java
onfiguration config = HBaseConfiguration.create();
Job job = new Job(config,"ExampleSummaryToFile");
job.setJarByClass(MySummaryFileJob.class);     // class that contains mapper and reducer

Scan scan = new Scan();
scan.setCaching(500);        // 1 is the default in Scan, which will be bad for MapReduce jobs
scan.setCacheBlocks(false);  // don't set to true for MR jobs
// set other scan attrs

TableMapReduceUtil.initTableMapperJob(
  sourceTable,        // input table
  scan,               // Scan instance to control CF and attribute selection
  MyMapper.class,     // mapper class
  Text.class,         // mapper output key
  IntWritable.class,  // mapper output value
  job);
job.setReducerClass(MyReducer.class);    // reducer class
job.setNumReduceTasks(1);    // at least one, adjust as required
FileOutputFormat.setOutputPath(job, new Path("/tmp/mr/mySummaryFile"));  // adjust directories as required

boolean b = job.waitForCompletion(true);
if (!b) {
  throw new IOException("error with job!");
}
```

如上所述，在此示例中，先前的Mapper可以保持不变。至于Reducer，它是一个“通用” Reducer，而不是扩展TableMapper和发出Puts。

```java
public static class MyReducer extends Reducer<Text, IntWritable, Text, IntWritable>  {

  public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
    int i = 0;
    for (IntWritable val : values) {
      i += val.get();
    }
    context.write(key, new IntWritable(i));
  }
}
```





## 单词统计案例（Maven）

### pom文件

```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <!--定义hadoop版本-->
    <hadoop.version>2.7.5</hadoop.version>
  </properties>

  <dependencies>

    <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-client/0.98.23-hadoop2 -->
    <dependency>
      <groupId>org.apache.hbase</groupId>
      <artifactId>hbase-client</artifactId>
      <version>0.98.23-hadoop2</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-mapreduce -->
    <dependency>
      <groupId>org.apache.hbase</groupId>
      <artifactId>hbase-mapreduce</artifactId>
      <version>2.1.0</version>
    </dependency>
    <!--hadoop客服端依赖-->
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-common</artifactId>
      <version>${hadoop.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-client</artifactId>
      <version>${hadoop.version}</version>
    </dependency>
    <!--hdfs文件系统依赖-->
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-hdfs</artifactId>
      <version>${hadoop.version}</version>
    </dependency>
    <!--MapReduce相关的依赖-->
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-mapreduce-client-core</artifactId>
      <version>${hadoop.version}</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
```

### WCRunner

```java
package icu.shaoyayu.hadoop.hbase.mr;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hbase.CompoundConfiguration;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

import java.io.IOException;

/**
 * @author shaoyayu
 * @date 2020/7/15 11:33
 * @E_Mail
 * @Version 1.0.0
 * @readme ：
 * Hbase与MapReduce结合使用
 */
public class WCRunner {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        //配置环境
        Configuration conf = new CompoundConfiguration();
        conf.set("","hadoopNode02,hadoopNode03,hadoopNode04");
        //需要设置存储的NameNode节点
        conf.set("fs.defaultFS", "hdfs://hadoopNode01:8020");

        Job job = Job.getInstance(conf);
        job.setJarByClass(WCRunner.class);

        job.setMapperClass(WCMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        //最后参数一定写false
        TableMapReduceUtil.initTableReducerJob("wc", WCReducer.class, job, null, null, null, null, false);
        FileInputFormat.addInputPath(job, new Path("/usr/wc"));
        // reduce端输出的key和value的类型
        job.setOutputKeyClass(NullWritable.class);
        job.setOutputValueClass(Put.class);

        // job.setOutputFormatClass(cls);
        // job.setInputFormatClass(cls);

        job.waitForCompletion(true);

    }

}
```

### WCMapper

```java
package icu.shaoyayu.hadoop.hbase.mr;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * @author shaoyayu
 * @date 2020/7/15 11:43
 * @E_Mail
 * @Version 1.0.0
 * @readme ：
 */
public class WCMapper extends Mapper<LongWritable,Text, Text, IntWritable>{

    @Override
    protected void map(LongWritable key, Text value, Context context)
            throws IOException, InterruptedException {
        String[] splits = value.toString().split(" ");
        //第二种切割方法
//    new StringTokenizer(value.toString()," ");
        for (String string : splits) {
            context.write(new Text(string), new IntWritable(1));
        }
    }
}
```

### WCReducer

```java
package icu.shaoyayu.hadoop.hbase.mr;

import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.TableReducer;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;

import java.io.IOException;

/**
 * @author shaoyayu
 * @date 2020/7/15 11:45
 * @E_Mail
 * @Version 1.0.0
 * @readme ：
 */
public class WCReducer extends TableReducer<Text, IntWritable, ImmutableBytesWritable>{

    @Override
    protected void reduce(Text key, Iterable<IntWritable> iter,
                          Context context)
            throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable intWritable : iter) {
            sum+=intWritable.get();
        }
        Put put = new Put(key.toString().getBytes());
        put.add("cf".getBytes(), "cf".getBytes(), String.valueOf(sum).getBytes());
        context.write(null, put);
    }
}
```



## 源码分析：TableMapReduceUtil

### initTableReducerJob（）

TableMapReduceUtil.initTableReducerJob(String table,
    Class<? extends TableReducer> reducer, Job job,
    Class partitioner, String quorumAddress, String serverClass,
    String serverImpl, boolean addDependencyJars)

```java
/ ** 
    *在提交TableReduce作业之前使用此功能。 
    *将适当地设置JobConf。 
    * 
    * @param table输出表。 
    * @param reducer要使用的reducer类。 
    * @param job当前要调整的作业。确保传递的作业*具有所有必要的HBase配置。 
    * @param partitioner要使用的Partitioner。传递<code> null </ code>以使用*默认分区程序。 
    * @param quorumAddress要写入的远程集群； 
    *输出到<code> hbase-site.xml </ code>中指定的集群的默认值为null。 
    *将此字符串设置为备用远程集群的zookeeper集成
    *当您需要减少写入非默认集群的集群时；例如在集群之间复制表时，源将由<code> hbase-site.xml </code>指定，
    *并且该参数将具有远程集群的集合地址。要传递的格式特别。 
    *传递<code> hbase.zookeeper.quorum
    * hbase.zookeeper.client.port
    * zookeeper.znode.parent 
    * </ code>，例如<code> server，server2，server3：2181：/ hbase </ code>。 
    * @param serverClass重新定义了hbase.regionserver.class 
    * @param serverImpl重新定义了hbase.regionserver.impl 
    * @param addDependencyJars通过分布式缓存（tmpjars）为任何已配置的作业类上载HBase jar和jars。 
    * @throws IOException当确定区域计数失败时。 
    * /
public static void initTableReducerJob(String table,
    Class<? extends TableReducer> reducer, Job job,
    Class partitioner, String quorumAddress, String serverClass,
    String serverImpl, boolean addDependencyJars) throws IOException {

    Configuration conf = job.getConfiguration();
    HBaseConfiguration.merge(conf, HBaseConfiguration.create(conf));
    //重点在这个地方设置输出到HBase里面去
    job.setOutputFormatClass(TableOutputFormat.class);
    if (reducer != null) job.setReducerClass(reducer);
    conf.set(TableOutputFormat.OUTPUT_TABLE, table);
    conf.setStrings("io.serializations", conf.get("io.serializations"),
        MutationSerialization.class.getName(), ResultSerialization.class.getName());
    // If passed a quorum/ensemble address, pass it on to TableOutputFormat.
    if (quorumAddress != null) {
      // Calling this will validate the format
      ZKConfig.validateClusterKey(quorumAddress);
      conf.set(TableOutputFormat.QUORUM_ADDRESS,quorumAddress);
    }
    if (serverClass != null && serverImpl != null) {
      conf.set(TableOutputFormat.REGION_SERVER_CLASS, serverClass);
      conf.set(TableOutputFormat.REGION_SERVER_IMPL, serverImpl);
    }
    job.setOutputKeyClass(ImmutableBytesWritable.class);
    job.setOutputValueClass(Writable.class);
    if (partitioner == HRegionPartitioner.class) {
      job.setPartitionerClass(HRegionPartitioner.class);
      int regions = MetaTableAccessor.getRegionCount(conf, TableName.valueOf(table));
      if (job.getNumReduceTasks() > regions) {
        job.setNumReduceTasks(regions);
      }
    } else if (partitioner != null) {
      job.setPartitionerClass(partitioner);
    }

    if (addDependencyJars) {
      addDependencyJars(job);
    }

    initCredentials(job);
  }
```

#### TableOutputFormat

```java
/ **
    *创建一个新的记录作者。
    *
    *请注意，基线javadoc给人的印象是只有一个
    *每个工作{@link RecordWriter}，但在HBase中，如果我们给您一个新的
    * RecordWriter每次调用此方法。 完成后，您必须关闭返回的RecordWriter。
    *否则将丢失写入。
    *
    * @param context当前任务上下文。
    * @return新创建的writer实例。
    * @throws IOException创建写入器时失败。
    * @throws InterruptedException作业取消时。
    * /
  @Override
  public RecordWriter<KEY, Mutation> getRecordWriter(TaskAttemptContext context)
  throws IOException, InterruptedException {
    return new TableRecordWriter();
  }
```

#### TableRecordWriter

```java
/**
   * 将reducer输出写入HBase表。
   */
  protected class TableRecordWriter
  extends RecordWriter<KEY, Mutation> {

    private Connection connection;
    private BufferedMutator mutator;

    /**
     * @throws IOException
     *
     */
    public TableRecordWriter() throws IOException {
      String tableName = conf.get(OUTPUT_TABLE);
      this.connection = ConnectionFactory.createConnection(conf);
      this.mutator = connection.getBufferedMutator(TableName.valueOf(tableName));
      LOG.info("Created table instance for "  + tableName);
    }
    /**
     * Closes the writer, in this case flush table commits.
     *
     * @param context  The context.
     * @throws IOException When closing the writer fails.
     * @see RecordWriter#close(TaskAttemptContext)
     */
    @Override
    public void close(TaskAttemptContext context) throws IOException {
      try {
        if (mutator != null) {
          mutator.close();
        }
      } finally {
        if (connection != null) {
          connection.close();
        }
      }
    }

/ **
      *将一个键/值对写入表中。
      *
      * @param键键。
      * @param value值。
      * @throws IOException写入失败时。
      * @请参见RecordWriter＃write（Object，Object）
      * /
    @Override
    public void write(KEY key, Mutation value)
    throws IOException {
      if (!(value instanceof Put) && !(value instanceof Delete)) {
        throw new IOException("Pass a Delete or a Put");
      }
      mutator.mutate(value);
    }
  }
```



### initTableMapperJob()

```java
/ **
  *在提交Multi TableMap作业之前使用它。 它将适当设置
  *完成工作。
  *
  * @param scans要读取的{@link Scan}对象的列表。
  * @param mapper要使用的mapper类。
  * @param outputKeyClass输出键的类。
  * @param outputValueClass输出值的类。
  * @param job当前要调整的作业。 确保通过的工作正在进行
  *所有必需的HBase配置。
  * @param addDependencyJars上传HBase jar和任何
  *通过分布式缓存（tmpjars）配置作业类。
  * @param initCredentials是否初始化作业的hbase身份验证凭据
  * @throws IOException设置细节时失败。
  * /
public static void initTableMapperJob(List<Scan> scans,
    Class<? extends TableMapper> mapper,
    Class<?> outputKeyClass,
    Class<?> outputValueClass, Job job,
    boolean addDependencyJars,
    boolean initCredentials) throws IOException {
    //设置Hbase为输入对象
  job.setInputFormatClass(MultiTableInputFormat.class);
  if (outputValueClass != null) {
    job.setMapOutputValueClass(outputValueClass);
  }
  if (outputKeyClass != null) {
    job.setMapOutputKeyClass(outputKeyClass);
  }
  job.setMapperClass(mapper);
  Configuration conf = job.getConfiguration();
  HBaseConfiguration.merge(conf, HBaseConfiguration.create(conf));
  List<String> scanStrings = new ArrayList<>();

  for (Scan scan : scans) {
    scanStrings.add(convertScanToString(scan));
  }
  job.getConfiguration().setStrings(MultiTableInputFormat.SCANS,
    scanStrings.toArray(new String[scanStrings.size()]));

  if (addDependencyJars) {
    addDependencyJars(job);
  }

  if (initCredentials) {
    initCredentials(job);
  }
}
```



#### MultiTableInputFormat

在父类`MultiTableInputFormatBase`中

```java
/ **
  *计算将用作地图任务输入的分割。 的
  *分割数与表中的区域数匹配。
  *
  * @param context当前作业上下文。
  * @return输入拆分列表。
  * @throws IOException创建拆分列表时失败。
  * @see org.apache.hadoop.mapreduce.InputFormat＃getSplits（org.apache.hadoop.mapreduce.JobContext）
  * /
@Override
public List<InputSplit> getSplits(JobContext context) throws IOException {
  if (scans.isEmpty()) {
    throw new IOException("No scans were provided.");
  }
  Map<TableName, List<Scan>> tableMaps = new HashMap<>();
  for (Scan scan : scans) {
    byte[] tableNameBytes = scan.getAttribute(Scan.SCAN_ATTRIBUTES_TABLE_NAME);
    if (tableNameBytes == null)
      throw new IOException("A scan object did not have a table name");
    TableName tableName = TableName.valueOf(tableNameBytes);
    List<Scan> scanList = tableMaps.get(tableName);
    if (scanList == null) {
      scanList = new ArrayList<>();
      tableMaps.put(tableName, scanList);
    }
    scanList.add(scan);
  }
  List<InputSplit> splits = new ArrayList<>();
  Iterator iter = tableMaps.entrySet().iterator();
  // Make a single Connection to the Cluster and use it across all tables.
  try (Connection conn = ConnectionFactory.createConnection(context.getConfiguration())) {
    while (iter.hasNext()) {
      Map.Entry<TableName, List<Scan>> entry = (Map.Entry<TableName, List<Scan>>) iter.next();
      TableName tableName = entry.getKey();
      List<Scan> scanList = entry.getValue();
      try (Table table = conn.getTable(tableName);
           RegionLocator regionLocator = conn.getRegionLocator(tableName)) {
        RegionSizeCalculator sizeCalculator = new RegionSizeCalculator(
            regionLocator, conn.getAdmin());
        Pair<byte[][], byte[][]> keys = regionLocator.getStartEndKeys();
        for (Scan scan : scanList) {
          if (keys == null || keys.getFirst() == null || keys.getFirst().length == 0) {
            throw new IOException("Expecting at least one region for table : "
                + tableName.getNameAsString());
          }
          int count = 0;
          byte[] startRow = scan.getStartRow();
          byte[] stopRow = scan.getStopRow();
          for (int i = 0; i < keys.getFirst().length; i++) {
            if (!includeRegionInSplit(keys.getFirst()[i], keys.getSecond()[i])) {
              continue;
            }
            if ((startRow.length == 0 || keys.getSecond()[i].length == 0 ||
                Bytes.compareTo(startRow, keys.getSecond()[i]) < 0) &&
                (stopRow.length == 0 || Bytes.compareTo(stopRow,
                    keys.getFirst()[i]) > 0)) {
              byte[] splitStart = startRow.length == 0 ||
                  Bytes.compareTo(keys.getFirst()[i], startRow) >= 0 ?
                  keys.getFirst()[i] : startRow;
              byte[] splitStop = (stopRow.length == 0 ||
                  Bytes.compareTo(keys.getSecond()[i], stopRow) <= 0) &&
                  keys.getSecond()[i].length > 0 ?
                  keys.getSecond()[i] : stopRow;
              HRegionLocation hregionLocation = regionLocator.getRegionLocation(
                  keys.getFirst()[i], false);
              String regionHostname = hregionLocation.getHostname();
              HRegionInfo regionInfo = hregionLocation.getRegionInfo();
              String encodedRegionName = regionInfo.getEncodedName();
              long regionSize = sizeCalculator.getRegionSize(
                  regionInfo.getRegionName());
              TableSplit split = new TableSplit(table.getName(),
                  scan, splitStart, splitStop, regionHostname,
                  encodedRegionName, regionSize);
              splits.add(split);
              if (LOG.isDebugEnabled()) {
                LOG.debug("getSplits: split -> " + (count++) + " -> " + split);
              }
            }
          }
        }
      }
    }
  }
  return splits;
}
```

默认的切片是RowKey的大小

