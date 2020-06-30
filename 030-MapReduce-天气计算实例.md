# MapReduce天气查询实列

## 天气统计案例

```
2000-01-01	16	29
2000-01-02	14	40
2000-01-03	23	35
2000-01-04	18	25
2000-01-05	14	33
2000-01-06	14	-4
......
2000-01-18	23	26
2000-01-19	10	-5
```

找出每个月中最高天气的两天

### 提交作业类

WeatherApp.class

```java
package icu.shaoyayu.hadoop.weather;

import icu.shaoyayu.hadoop.weather.entity.WeatherMapOutputKeyClass;
import icu.shaoyayu.hadoop.weather.mapper.WeatherMapper;
import icu.shaoyayu.hadoop.weather.reduce.WeatherReduce;
import icu.shaoyayu.hadoop.weather.util.WeatherGroupingComparator;
import icu.shaoyayu.hadoop.weather.util.WeatherPartitioner;
import icu.shaoyayu.hadoop.weather.util.WeatherSortComparator;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * @author 邵涯语
 * @date 2020/4/17 16:41
 * @Version :
 */
public class WeatherApp {

    private static final Log LOG = LogFactory.getLog(WeatherApp.class.getName());

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        //初始换配置
        Configuration configuration = new Configuration(true);

        //获取作业的实列
        Job job = Job.getInstance(configuration);
        //设置启动类
        job.setJarByClass(WeatherApp.class);

        /**
         *  在public class JobContextImpl implements JobContext 中有配置
         *  在JobContextImpl中提到获取
         *   conf.getClass(INPUT_FORMAT_CLASS_ATTR, TextInputFormat.class);
         *  TextInputFormat.class是默认的配置，当然要是可以配置
         *  job.setInputFormatClass(MyInputFormatClass.class);
         */

        /**
         * 准备一个我们自己的mapper类 默认的是Mapper类
         */
        job.setMapperClass(WeatherMapper.class);

        /**
         * map输出的key，要实现序列化和反序列化接口
         */
        job.setMapOutputKeyClass(WeatherMapOutputKeyClass.class);

        /**
         * 设置一个输出的value的类型
         */
        job.setMapOutputValueClass(IntWritable.class);

        /**
         * 设置一个分区器
         */
        job.setPartitionerClass(WeatherPartitioner.class);

        /**
         * 设置一个排序比较累
         */
        job.setSortComparatorClass(WeatherSortComparator.class);

        /**
         * 提交作业等待完成
         *
         */
        job.waitForCompletion(true);

        /**
         * 设置一个Combiner
         * job.setCombinerClass(WeatherCombiner.class);
         */

        //==========================Reduce阶段==============================

        /**
         * 分组比较器
         */
        job.setGroupingComparatorClass(WeatherGroupingComparator.class);

        job.setReducerClass(WeatherReduce.class);

        //设置文件输入路径
        Path InputPath = new Path("/data/weather/input/");
        FileInputFormat.setInputPaths(job,InputPath);


        //设置输出路径
        Path outputPath = new Path("data/weather/output");
        //如果路劲存在，递归删除路径
        if (outputPath.getFileSystem(configuration).exists(outputPath)){
            outputPath.getFileSystem(configuration).delete(outputPath,true);
        }
        FileOutputFormat.setOutputPath(job,outputPath);

        //设置两个Reduce的数量
        job.setNumReduceTasks(2);

    }

}
```



### 自定义Mapper输出的key对象

WeatherMapOutputKeyClass.class

```java
package icu.shaoyayu.hadoop.weather.entity;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

/**
 * @author 邵涯语
 * @date 2020/4/17 17:36
 * @Version :
 */
public class WeatherMapOutputKeyClass implements WritableComparable<WeatherMapOutputKeyClass> {

    private int year;
    private int month;
    private int day;
    private int temperature;

    public int getYear() {
        return year;
    }

    public void setYear(int year) {
        this.year = year;
    }

    public int getMonth() {
        return month;
    }

    public void setMonth(int month) {
        this.month = month;
    }

    public int getDay() {
        return day;
    }

    public void setDay(int day) {
        this.day = day;
    }

    public int getTemperature() {
        return temperature;
    }

    public void setTemperature(int temperature) {
        this.temperature = temperature;
    }

    /**
     * Comparison method
     * 排序的方法，默认的是正序的排序
     * @param keyClass
     * @return
     */
    @Override
    public int compareTo(WeatherMapOutputKeyClass keyClass) {
        int sizeDetermination = Integer.compare(this.year,keyClass.year);
        if (sizeDetermination==0){
            //相等的时候判定月
            sizeDetermination = Integer.compare(this.month,keyClass.month);
            if (sizeDetermination==0){
                return Integer.compare(this.day,keyClass.day);
            }else {
                return sizeDetermination;
            }
        }
        return sizeDetermination;
    }

    /**
     * Serialization method
     * @param out
     * @throws IOException
     */
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeInt(this.year);
        out.writeInt(this.month);
        out.writeInt(this.day);
        out.writeInt(this.temperature);
    }

    /**
     * Deserialization method
     * @param in
     * @throws IOException
     */
    @Override
    public void readFields(DataInput in) throws IOException {
        this.year = in.readInt();
        this.month = in.readInt();
        this.day = in.readInt();
        this.temperature = in.readInt();
    }

    @Override
    public String toString() {
        return year +"-"+ month +"-"+ day ;
    }
}
```





### 自定义Mapper类

WeatherMapper.class

```java
package icu.shaoyayu.hadoop.weather.mapper;

import icu.shaoyayu.hadoop.weather.entity.WeatherMapOutputKeyClass;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.MapTask;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.util.StringUtils;

import java.io.IOException;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

/**
 * @author 邵涯语
 * @date 2020/4/18 11:14
 * @Version :
 * 默认的输入格式化类还是TextInputFormat
 */
public class WeatherMapper extends Mapper<LongWritable, Text, WeatherMapOutputKeyClass, IntWritable> {

    WeatherMapOutputKeyClass mWeatherKeyClass = new WeatherMapOutputKeyClass();
    IntWritable mLatitudeValue =  new IntWritable();

    /**
     * 重写map的方法
     * @param key
     * @param value
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        /*
        2000-01-01	16	29
        2000-01-02	14	40
        2000-01-03	23	35
        2000-01-04	18	25
        2000-01-05	14	33
        2000-01-06	14	-4
        2000-01-07	4	24
         */

        try {
            String[] sts = StringUtils.split("\t");
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            Date date = sdf.parse(sts[0]);
            Calendar cal = Calendar.getInstance();
            cal.setTime(date);
            //对时间赋值
            mWeatherKeyClass.setYear(cal.get(Calendar.YEAR));
            mWeatherKeyClass.setMonth(cal.get(Calendar.MONTH)+1);
            mWeatherKeyClass.setDay(cal.get(Calendar.DAY_OF_MONTH));
            int temperature = Integer.parseInt(sts[sts.length-1].substring(0,sts[sts.length-1].length()-1));
            mWeatherKeyClass.setTemperature(temperature);
            mLatitudeValue.set(temperature);
            //输出
            context.write(mWeatherKeyClass,mLatitudeValue);
        } catch (ParseException e) {
            e.printStackTrace();
        }

    }
}
```

### 自定义分区器

WeatherPartitioner.class

```java
package icu.shaoyayu.hadoop.weather.util;

import icu.shaoyayu.hadoop.weather.entity.WeatherMapOutputKeyClass;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.mapreduce.Partitioner;

/**
 * @author 邵涯语
 * @date 2020/4/18 12:00
 * @Version :
 */
public class WeatherPartitioner extends Partitioner<WeatherMapOutputKeyClass, IntWritable> {
    @Override
    public int getPartition(WeatherMapOutputKeyClass keyClass, IntWritable intWritable, int numPartitions) {

        return keyClass.hashCode()%numPartitions;

    }
}
```

### 自定义一个排序类

WeatherSortComparator.class

```java
package icu.shaoyayu.hadoop.weather.util;

import icu.shaoyayu.hadoop.weather.entity.WeatherMapOutputKeyClass;
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

/**
 * @author 邵涯语
 * @date 2020/4/18 12:08
 * @Version :
 */
public class WeatherSortComparator extends WritableComparator {

    /**
     * 实例化
     */
    public WeatherSortComparator(){
        super(WeatherMapOutputKeyClass.class,true);
    }

    /**
     * 比较,按照年月做正序温度做倒序
     * @param a
     * @param b
     * @return
     */
    @Override
    public int compare(WritableComparable a, WritableComparable b) {

        WeatherMapOutputKeyClass keyClass1 = (WeatherMapOutputKeyClass) a;
        WeatherMapOutputKeyClass keyClass2 = (WeatherMapOutputKeyClass) b;

        int contrast = Integer.compare(keyClass1.getYear(),keyClass2.getYear());
        //比较年
        if (contrast==0){
            contrast = Integer.compare(keyClass1.getMonth(),keyClass2.getMonth());
            //比较月份
            if (contrast==0){
                //温度进行倒序比较
                return -Integer.compare(keyClass1.getTemperature(),keyClass2.getTemperature());
            }else {
                return contrast;
            }
        }else {
            return contrast;
        }

    }
}
```



### 自定义一个分组器

WeatherGroupingComparator.class

```JAVA
package icu.shaoyayu.hadoop.weather.util;

import icu.shaoyayu.hadoop.weather.entity.WeatherMapOutputKeyClass;
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

/**
 * @author 邵涯语
 * @date 2020/4/18 13:00
 * @Version :
 */
public class WeatherGroupingComparator extends WritableComparator {

    public WeatherGroupingComparator(){
        super(WeatherMapOutputKeyClass.class,true);
    }

    @Override
    public int compare(WritableComparable a, WritableComparable b) {
        WeatherMapOutputKeyClass keyClass1 = (WeatherMapOutputKeyClass) a;
        WeatherMapOutputKeyClass keyClass2 = (WeatherMapOutputKeyClass) b;

        int contrast = Integer.compare(keyClass1.getYear(),keyClass2.getYear());
        //比较年
        if (contrast==0){
            return  Integer.compare(keyClass1.getMonth(),keyClass2.getMonth());
        }else {
            return contrast;
        }
    }
}
```

### 自定义一个Reduce

```java
package icu.shaoyayu.hadoop.weather.reduce;


import icu.shaoyayu.hadoop.weather.entity.WeatherMapOutputKeyClass;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * @author 邵涯语
 * @date 2020/4/18 13:08
 * @Version :
 */
public class WeatherReduce extends Reducer<WeatherMapOutputKeyClass, IntWritable, Text, IntWritable> {

    Text mRKey = new Text();
    IntWritable mRValue = new IntWritable();

    /**
     * 重写Reduce方法
     * @param key
     * @param values
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void reduce(WeatherMapOutputKeyClass key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        //values 分为每个月组的数据
        int flg = 0;
        int day = 0;
        for (IntWritable value : values) {
            if (flg==0){
                mRKey.set(key.toString());
                mRValue.set(key.getTemperature());
                flg++;
                day = key.getDay();
                context.write(mRKey,mRValue);
            }
            if (flg!=0 && day!=key.getDay()){
                mRKey.set(key.toString());
                mRValue.set(key.getTemperature());
                context.write(mRKey,mRValue);
                break;
            }
        }
    }
}
```

