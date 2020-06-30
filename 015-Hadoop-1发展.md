#  Hadoop介绍

### 什么是Apache Hadoop？

Apache™Hadoop®项目开发了用于可靠，可扩展的分布式计算的开源软件。

Apache Hadoop软件库是一个框架，该框架允许使用简单的编程模型跨计算机集群对大型数据集进行分布式处理。它旨在从单个服务器扩展到数千台机器，每台机器都提供本地计算和存储。库本身不用于依靠硬件来提供高可用性，而是被设计用来检测和处理应用程序层的故障，因此可以在计算机集群的顶部提供高可用性服务，每台计算机都容易出现故障。

#### 该项目包括以下模块：

- **Hadoop Common**：支持其他Hadoop模块的通用实用程序。
- **Hadoop分布式文件系统（HDFS™）**：一种分布式文件系统，可提供对应用程序数据的高吞吐量访问。
- **Hadoop YARN**：用于作业调度和群集资源管理的框架。
- **Hadoop MapReduce**：基于YARN的系统，用于并行处理大数据集。

#### Apache的其他与Hadoop相关的项目包括：

- [**Ambari™**](http://incubator.apache.org/ambari/)：基于Web的工具，用于供应，管理和监视Apache Hadoop集群，其中包括对Hadoop HDFS，Hadoop MapReduce，Hive，HCatalog，HBase，ZooKeeper，Oozie，Pig和Sqoop的支持。Ambari还提供了一个仪表板，用于查看集群健康状况（例如热图）以及以可视方式查看MapReduce，Pig和Hive应用程序的功能以及以用户友好的方式诊断其性能特征的功能。
- [**Avro™**](http://avro.apache.org/)：数据序列化系统。
- [**Cassandra™**](http://cassandra.apache.org/)：可扩展的多主数据库，没有单点故障。
- [**Chukwa™**](http://incubator.apache.org/chukwa/)：一种用于管理大型分布式系统的数据收集系统。
- [**HBase™**](http://hbase.apache.org/)：可扩展的分布式数据库，支持大型表的结构化数据存储。
- [**Hive™**](http://hive.apache.org/)：一种数据仓库基础结构，提供数据汇总和即席查询。
- [**Mahout™**](http://mahout.apache.org/)：可扩展的机器学习和数据挖掘库。
- [**Pig™**](http://pig.apache.org/)：用于并行计算的高级数据流语言和执行框架。
- [**Spark™**](http://spark.incubator.apache.org/)：一种用于Hadoop数据的快速通用计算引擎。Spark提供了一个简单而富有表现力的编程模型，该模型支持广泛的应用程序，包括ETL，机器学习，流处理和图形计算。
- [**Tez™**](http://tez.incubator.apache.org/)：基于Hadoop YARN的通用数据流编程框架，它提供了强大而灵活的引擎来执行任意DAG任务，以处理批处理和交互用例的数据。Hadoop生态系统中的Hive™，Pig™和其他框架以及其他商业软件（例如ETL工具）都采用了Tez，以取代Hadoop™MapReduce作为基础执行引擎。
- [**ZooKeeper™**](http://zookeeper.apache.org/)：针对分布式应用程序的高性能协调服务。