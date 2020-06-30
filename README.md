# Hadoop 大数据学习笔记

> 欢迎大家讨论交流
>
> 可能会有长时间一次更新的情况，请耐心等待。谢谢

![](images/hadoop-logo.jpg)

**Apache Hadoop**是一款支持数据密集型[分布式](https://zh.wikipedia.org/w/index.php?title=分佈式&action=edit&redlink=1)应用程序并以Apache 2.0许可协议发布的[开源](https://zh.wikipedia.org/wiki/开源)[软件框架](https://zh.wikipedia.org/wiki/軟體框架)。它支持在商品硬件构建的大型集群上运行的应用程序。Hadoop是根据[谷歌公司](https://zh.wikipedia.org/wiki/谷歌公司)发表的[MapReduce](https://zh.wikipedia.org/wiki/MapReduce)和[Google文件系统](https://zh.wikipedia.org/wiki/Google檔案系統)的论文自行实现而成。所有的Hadoop模块都有一个基本假设，即硬件故障是常见情况，应该由框架自动处理。

Hadoop框架透明地为应用提供可靠性和数据移动。它实现了名为MapReduce的[编程范式](https://zh.wikipedia.org/wiki/编程范式)：应用程序被分割成许多小部分，而每个部分都能在集群中的任意节点上运行或重新运行。此外，Hadoop还提供了分布式文件系统，用以存储所有计算节点的数据，这为整个集群带来了非常高的带宽。MapReduce和分布式文件系统的设计，使得整个框架能够自动处理节点故障。它使应用程序与成千上万的独立计算的电脑和PB级的数据连接起来。现在普遍认为整个Apache Hadoop“平台”包括Hadoop内核、MapReduce、Hadoop分布式文件系统（HDFS）以及一些相关项目，有Apache Hive和Apache HBase等等。[维基百科](https://zh.wikipedia.org/wiki/Apache_Hadoop)