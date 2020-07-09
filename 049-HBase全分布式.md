# HBase全分布式部署

## 1、HDFS上的独立HBase

在独立的hbase上有时有用的变体是，所有守护程序都在一个JVM中运行，而不是持久化到本地文件系统，而是持久化到HDFS实例。

当您打算使用简单的部署概要文件时，可以考虑使用此概要文件，虽然负载很轻，但是数据必须在节点间来回移动。写入要复制数据的HDFS可确保后者。

要配置此独立变体，请编辑hbase-site.xml 设置hbase.rootdir 以指向HDFS实例中的目录，然后将hbase.cluster.distributed设置 为false。例如：

```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://namenode.example.org:8020/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>false</value>
  </property>
</configuration>
```

## 2、完全分布

默认情况下，HBase以独立模式运行。提供了独立模式和伪分布式模式都是为了进行小型测试。对于生产环境，建议使用分布式模式。在分布式模式下，HBase守护程序的多个实例在群集中的多个服务器上运行。

就像在伪分布式模式下一样，完全分布式配置需要将hbase.cluster.distributed属性设置 为true。通常，将hbase.rootdir其配置为指向高度可用的HDFS文件系统。

此外，还配置了群集，以便多个群集节点可以注册为RegionServer，ZooKeeper QuorumPeers和备份HMaster服务器。这些配置基础都在quickstart-full-distributed中进行了演示。

分布式RegionServer
通常，您的群集将包含都运行在不同服务器上的多个RegionServer，以及主和备份Master和ZooKeeper守护程序。该CONF / regionservers在主服务器上的文件中包含主机，其RegionServers与该集群相关的列表。每个主机位于单独的行上。当主服务器启动或停止时，此文件中列出的所有主机都将启动和停止其RegionServer进程。

ZooKeeper和HBase
有关HBase的ZooKeeper设置说明，请参见ZooKeeper部分。

例子2.例子分布式HBase集群

这是分布式HBase群集的*conf / hbase-site.xml*。用于实际工作的群集将包含更多自定义配置参数。大多数HBase配置指令都有默认值，除非在*hbase-site.xml中*覆盖该值，否则将使用默认值。有关更多信息，请参见“ [配置文件](https://hbase.apache.org/book.html#config.files) ”。

```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://namenode.example.org:8020/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>node-a.example.com,node-b.example.com,node-c.example.com</value>
  </property>
</configuration>
```

这是一个示例*conf / regionservers*文件，其中包含应在集群中运行RegionServer的节点的列表。这些节点需要安装HBase，并且需要使用与主服务器相同的*conf /*目录内容。

```java
node-a.example.com
node-b.example.com
node-c.example.com
```

这是示例conf / backup-masters文件，其中包含应运行备份Master实例的每个节点的列表。除非主主实例不可用，否则备份主实例将处于空闲状态。

```java
node-b.example.com
node-c.example.com
```

过程：HDFS客户端配置

1. 值得注意的是，如果您在Hadoop集群上进行了HDFS客户端配置更改（例如HDFS客户端的配置指令），而不是服务器端配置，则必须使用以下方法之一使HBase能够查看和使用这些配置更改：
   1. 的一个指针添加`HADOOP_CONF_DIR`到`HBASE_CLASSPATH`环境变量 *hbase-env.sh*。
   2. 在*$ {HBASE_HOME} / conf*下添加*hdfs-site.xml*（或*hadoop-site.xml*）或更佳的符号链接的 *副本*，或
   3. 如果只有一小部分HDFS客户端配置，请将其添加到*hbase-site.xml*。

这样的HDFS客户端配置的示例是`dfs.replication`。例如，如果要以5的复制因子运行，除非您执行上述操作以使配置可用于HBase，否则HBase将创建默认值为3的文件。

## 3、完全分布式安装

### 3.1、hbase-env.sh

这里需要配置好JAVA_HOME变量和指定自己的Zookeeper集群，把HBASE_MANAGES_ZK指定为false

### 3.2、hbase-site.xml

```xml
<property>
    <name>hbase.rootdir</name>
    <value>hdfs://mycluster/hbase</value>
</property>
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
</property>
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>hadoopNode02,hadoopNode03,hadoopNode04</value>
</property>
```

### 3.3、regionservers

```shell
hadoopNode02
hadoopNode03
hadoopNode04
```

### 3.4、backup-masters

```shell
hadoopNode04
```

### 3.5、拷贝hdfs.site.xml

在*$ {HBASE_HOME} / conf*下添加*hdfs-site.xml*（或*hadoop-site.xml*）或更佳的符号链接的 *副本*，

### 3.6、分发HBase

```shell
> scp -r /opt/mgs/hbase hadoopNode02:/opt/mgs/hbase
> scp -r /opt/mgs/hbase hadoopNode03:/opt/mgs/hbase
> scp -r /opt/mgs/hbase hadoopNode04:/opt/mgs/hbase
```

### 3.7、修改profile

添加HBASE_HOME变量

source一下

## 4、启动集群

启动集群之前确保自己的hdfs集群必须启动起来。

```shell
> start-hbase.sh
# 单节点启动 
> hbase-daemon.sh start master
```



## 5、Zookeeper里面的数据

```shell
> zkCli.sh

```

