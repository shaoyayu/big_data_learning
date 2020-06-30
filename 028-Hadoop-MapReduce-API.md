# MapReduce api实战

### 配置pmx

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>icu.shaoyayu.hadoop</groupId>
  <artifactId>mapReduceApi</artifactId>
  <version>1.0</version>
  <packaging>jar</packaging>
  <name>mapReduceApi</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <!--定义hadoop版本-->
    <hadoop.version>2.7.5</hadoop.version>
  </properties>

  <dependencies>
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

  <build>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <!-- clean lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#clean_Lifecycle -->
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- default lifecycle, jar packaging: see https://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-jar-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
        <!-- site lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#site_Lifecycle -->
        <plugin>
          <artifactId>maven-site-plugin</artifactId>
          <version>3.7.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-project-info-reports-plugin</artifactId>
          <version>3.0.0</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>

```

### 环境配置

跟Hdfs的API一样，将配置文件拷贝到本地

### 程序入口

```java
package icu.shaoyayu.hadoop;

import icu.shaoyayu.hadoop.map.MyMapper;
import icu.shaoyayu.hadoop.reduce.MyReducer;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * @author shaoyayu
 *
 * 计算出每个时间段出现的行为次数最多的
 */
public class App {
    public static void main( String[] args ) throws IOException, ClassNotFoundException, InterruptedException {
        //获取配置文件
        Configuration config = new Configuration(true);
        //拿到作业
        Job job = Job.getInstance(config);
        job.setJobName("myJob_1");
        //设置启动的类
        job.setJarByClass(App.class);

        //定义一个hdfs的输入源作为输入
        Path inputPath = new Path("/user/root/user/mgs/tianmao/tianchi_mobile_recommend_train_user.csv");
        //可以定义多个数据源作为输入
        FileInputFormat.addInputPath(job,inputPath);

        //只能存在一个输出的数据源
        Path outputPath = new Path("/user/root/user/mgs/outputTianMao");
        //因为输出的路径不能存在，需要删除
        if (outputPath.getFileSystem(config).exists(outputPath)){
            outputPath.getFileSystem(config).delete(outputPath,true);
        }
        FileOutputFormat.setOutputPath(job,outputPath);

        //设置Mapper环境的类
        job.setMapperClass(MyMapper.class);
        //告诉后面的反序列化是哪个类
        job.setMapOutputKeyClass(IntWritable.class);
        job.setMapOutputValueClass(Text.class);
        //设置Reduce环境的类
        job.setReducerClass(MyReducer.class);

        job.waitForCompletion(true);

    }
}
```

### MyMap类

```java
package icu.shaoyayu.hadoop.map;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * @author 邵涯语
 * @date 2020/4/10 22:26
 * @Version :
 * <KEYIN, VALUEIN, 输入类型相关的 在每一行的split的到的数据类型有关
 * KEYOUT, VALUEOUT>  输出给Reduce的数据类型
 */
public class MyMapper extends Mapper<Object, Text, IntWritable, Text> {

    private Text word = new Text();

    /**
     * map方法会被多次调用
     * @param key   字符串的偏移量，
     * @param value 行的数据
     * @param context   上下文
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void map(Object key, Text value, Context context) throws IOException, InterruptedException {
        //按照一定的方法切割字符串
        String[] split = value.toString().split(",");
        if (split.length!=6){
            return;
        }
        //取出最后一个时间小时值
        String[] times = split[split.length-1].split(" ");
        //第一行存在没有值时间
        if (times.length!=2){
            return;
        }
        //取出时间
        IntWritable time = new IntWritable(Integer.valueOf(times[1]));
        word.set(split[0]+","+split[2]);
        context.write(time, word);

    }
}
```

### MyReduce类

```java
package icu.shaoyayu.hadoop.reduce;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * @author 邵涯语
 * @date 2020/4/10 22:27
 * @Version :
 * <Text, IntWritable, 这个地方的输入来自map阶段的输出
 * Text, IntWritable>   自定义的输出类型
 */
public class MyReducer extends Reducer<IntWritable, Text, IntWritable, IntWritable> {

    private IntWritable result = new IntWritable();

    @Override
    protected void reduce(IntWritable key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (Text val : values) {
            sum = sum+1;
        }
        result.set(sum);
        context.write(key, result);
    }
}

```

### 运行

打包成jar防盗对于的节点上面执行

```
hadoop jar [jar报名] [入口程序包名.类名]
```

### Mapper类源码

```java
/**
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.apache.hadoop.mapreduce;

import java.io.IOException;

import org.apache.hadoop.classification.InterfaceAudience;
import org.apache.hadoop.classification.InterfaceStability;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.RawComparator;
import org.apache.hadoop.io.compress.CompressionCodec;
import org.apache.hadoop.mapreduce.task.MapContextImpl;

/** 
 * Maps input key/value pairs to a set of intermediate key/value pairs.  
 * 
 * <p>Maps are the individual tasks which transform input records into a 
 * intermediate records. The transformed intermediate records need not be of 
 * the same type as the input records. A given input pair may map to zero or 
 * many output pairs.</p> 
 * 
 * <p>The Hadoop Map-Reduce framework spawns one map task for each 
 * {@link InputSplit} generated by the {@link InputFormat} for the job.
 * <code>Mapper</code> implementations can access the {@link Configuration} for 
 * the job via the {@link JobContext#getConfiguration()}.
 * 
 * <p>The framework first calls 
 * {@link #setup(org.apache.hadoop.mapreduce.Mapper.Context)}, followed by
 * {@link #map(Object, Object, org.apache.hadoop.mapreduce.Mapper.Context)}
 * for each key/value pair in the <code>InputSplit</code>. Finally 
 * {@link #cleanup(org.apache.hadoop.mapreduce.Mapper.Context)} is called.</p>
 * 
 * <p>All intermediate values associated with a given output key are 
 * subsequently grouped by the framework, and passed to a {@link Reducer} to  
 * determine the final output. Users can control the sorting and grouping by 
 * specifying two key {@link RawComparator} classes.</p>
 *
 * <p>The <code>Mapper</code> outputs are partitioned per 
 * <code>Reducer</code>. Users can control which keys (and hence records) go to 
 * which <code>Reducer</code> by implementing a custom {@link Partitioner}.
 * 
 * <p>Users can optionally specify a <code>combiner</code>, via 
 * {@link Job#setCombinerClass(Class)}, to perform local aggregation of the 
 * intermediate outputs, which helps to cut down the amount of data transferred 
 * from the <code>Mapper</code> to the <code>Reducer</code>.
 * 
 * <p>Applications can specify if and how the intermediate
 * outputs are to be compressed and which {@link CompressionCodec}s are to be
 * used via the <code>Configuration</code>.</p>
 *  
 * <p>If the job has zero
 * reduces then the output of the <code>Mapper</code> is directly written
 * to the {@link OutputFormat} without sorting by keys.</p>
 * 
 * <p>Example:</p>
 * <p><blockquote><pre>
 * public class TokenCounterMapper 
 *     extends Mapper&lt;Object, Text, Text, IntWritable&gt;{
 *    
 *   private final static IntWritable one = new IntWritable(1);
 *   private Text word = new Text();
 *   
 *   public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
 *     StringTokenizer itr = new StringTokenizer(value.toString());
 *     while (itr.hasMoreTokens()) {
 *       word.set(itr.nextToken());
 *       context.write(word, one);
 *     }
 *   }
 * }
 * </pre></blockquote>
 *
 * <p>Applications may override the
 * {@link #run(org.apache.hadoop.mapreduce.Mapper.Context)} method to exert
 * greater control on map processing e.g. multi-threaded <code>Mapper</code>s 
 * etc.</p>
 * 
 * @see InputFormat
 * @see JobContext
 * @see Partitioner  
 * @see Reducer
 */
@InterfaceAudience.Public
@InterfaceStability.Stable
public class Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT> {

  /**
   * The <code>Context</code> passed on to the {@link Mapper} implementations.
   */
  public abstract class Context
    implements MapContext<KEYIN,VALUEIN,KEYOUT,VALUEOUT> {
  }
  
  /**
   * Called once at the beginning of the task.
   */
  protected void setup(Context context
                       ) throws IOException, InterruptedException {
    // NOTHING
  }

  /**
   * Called once for each key/value pair in the input split. Most applications
   * should override this, but the default is the identity function.
   */
  @SuppressWarnings("unchecked")
  protected void map(KEYIN key, VALUEIN value, 
                     Context context) throws IOException, InterruptedException {
    context.write((KEYOUT) key, (VALUEOUT) value);
  }

  /**
   * Called once at the end of the task.
   */
  protected void cleanup(Context context
                         ) throws IOException, InterruptedException {
    // NOTHING
  }
  
  /**
   * Expert users can override this method for more complete control over the
   * execution of the Mapper.
   * @param context
   * @throws IOException
   */
  public void run(Context context) throws IOException, InterruptedException {
    setup(context);
    try {
      while (context.nextKeyValue()) {
        map(context.getCurrentKey(), context.getCurrentValue(), context);
      }
    } finally {
      cleanup(context);
    }
  }
}
```

### Reduce类源码

```java
/**
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.apache.hadoop.mapreduce;

import java.io.IOException;

import org.apache.hadoop.classification.InterfaceAudience;
import org.apache.hadoop.classification.InterfaceStability;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.mapreduce.task.annotation.Checkpointable;

import java.util.Iterator;

/** 
 * Reduces a set of intermediate values which share a key to a smaller set of
 * values.  
 * 
 * <p><code>Reducer</code> implementations 
 * can access the {@link Configuration} for the job via the 
 * {@link JobContext#getConfiguration()} method.</p>

 * <p><code>Reducer</code> has 3 primary phases:</p>
 * <ol>
 *   <li>
 *   
 *   <b id="Shuffle">Shuffle</b>
 *   
 *   <p>The <code>Reducer</code> copies the sorted output from each 
 *   {@link Mapper} using HTTP across the network.</p>
 *   </li>
 *   
 *   <li>
 *   <b id="Sort">Sort</b>
 *   
 *   <p>The framework merge sorts <code>Reducer</code> inputs by 
 *   <code>key</code>s 
 *   (since different <code>Mapper</code>s may have output the same key).</p>
 *   
 *   <p>The shuffle and sort phases occur simultaneously i.e. while outputs are
 *   being fetched they are merged.</p>
 *      
 *   <b id="SecondarySort">SecondarySort</b>
 *   
 *   <p>To achieve a secondary sort on the values returned by the value 
 *   iterator, the application should extend the key with the secondary
 *   key and define a grouping comparator. The keys will be sorted using the
 *   entire key, but will be grouped using the grouping comparator to decide
 *   which keys and values are sent in the same call to reduce.The grouping 
 *   comparator is specified via 
 *   {@link Job#setGroupingComparatorClass(Class)}. The sort order is
 *   controlled by 
 *   {@link Job#setSortComparatorClass(Class)}.</p>
 *   
 *   
 *   For example, say that you want to find duplicate web pages and tag them 
 *   all with the url of the "best" known example. You would set up the job 
 *   like:
 *   <ul>
 *     <li>Map Input Key: url</li>
 *     <li>Map Input Value: document</li>
 *     <li>Map Output Key: document checksum, url pagerank</li>
 *     <li>Map Output Value: url</li>
 *     <li>Partitioner: by checksum</li>
 *     <li>OutputKeyComparator: by checksum and then decreasing pagerank</li>
 *     <li>OutputValueGroupingComparator: by checksum</li>
 *   </ul>
 *   </li>
 *   
 *   <li>   
 *   <b id="Reduce">Reduce</b>
 *   
 *   <p>In this phase the 
 *   {@link #reduce(Object, Iterable, org.apache.hadoop.mapreduce.Reducer.Context)}
 *   method is called for each <code>&lt;key, (collection of values)&gt;</code> in
 *   the sorted inputs.</p>
 *   <p>The output of the reduce task is typically written to a 
 *   {@link RecordWriter} via 
 *   {@link Context#write(Object, Object)}.</p>
 *   </li>
 * </ol>
 * 
 * <p>The output of the <code>Reducer</code> is <b>not re-sorted</b>.</p>
 * 
 * <p>Example:</p>
 * <p><blockquote><pre>
 * public class IntSumReducer&lt;Key&gt; extends Reducer&lt;Key,IntWritable,
 *                                                 Key,IntWritable&gt; {
 *   private IntWritable result = new IntWritable();
 * 
 *   public void reduce(Key key, Iterable&lt;IntWritable&gt; values,
 *                      Context context) throws IOException, InterruptedException {
 *     int sum = 0;
 *     for (IntWritable val : values) {
 *       sum += val.get();
 *     }
 *     result.set(sum);
 *     context.write(key, result);
 *   }
 * }
 * </pre></blockquote>
 * 
 * @see Mapper
 * @see Partitioner
 */
@Checkpointable
@InterfaceAudience.Public
@InterfaceStability.Stable
public class Reducer<KEYIN,VALUEIN,KEYOUT,VALUEOUT> {

  /**
   * The <code>Context</code> passed on to the {@link Reducer} implementations.
   */
  public abstract class Context 
    implements ReduceContext<KEYIN,VALUEIN,KEYOUT,VALUEOUT> {
  }

  /**
   * Called once at the start of the task.
   */
  protected void setup(Context context
                       ) throws IOException, InterruptedException {
    // NOTHING
  }

  /**
   * This method is called once for each key. Most applications will define
   * their reduce class by overriding this method. The default implementation
   * is an identity function.
   */
  @SuppressWarnings("unchecked")
  protected void reduce(KEYIN key, Iterable<VALUEIN> values, Context context
                        ) throws IOException, InterruptedException {
    for(VALUEIN value: values) {
      context.write((KEYOUT) key, (VALUEOUT) value);
    }
  }

  /**
   * Called once at the end of the task.
   */
  protected void cleanup(Context context
                         ) throws IOException, InterruptedException {
    // NOTHING
  }

  /**
   * Advanced application writers can use the 
   * {@link #run(org.apache.hadoop.mapreduce.Reducer.Context)} method to
   * control how the reduce task works.
   */
  public void run(Context context) throws IOException, InterruptedException {
    setup(context);
    try {
      while (context.nextKey()) {
        reduce(context.getCurrentKey(), context.getValues(), context);
        // If a back up store is used, reset it
        Iterator<VALUEIN> iter = context.getValues().iterator();
        if(iter instanceof ReduceContext.ValueIterator) {
          ((ReduceContext.ValueIterator<VALUEIN>)iter).resetBackupStore();        
        }
      }
    } finally {
      cleanup(context);
    }
  }
}
```

