# Hadoop集群的HA

### 1、简介

HA模式解决单点故障问题，

### 2、规划

-------

|       | NN-1 | NN-2 |  DN  |  ZK  | ZKFC | JNN  |
| :---: | :--: | :--: | :--: | :--: | :--: | :--: |
| Node1 |  *   |      |      |      |  *   |  *   |
| Node2 |      |  *   |  *   |  *   |  *   |  *   |
| Node3 |      |      |  *   |  *   |      |  *   |
| Node4 |      |      |  *   |  *   |      |      |

ZK:  zookeeper 

ZKFC:   failover controller【故障转移进程】

#### 2.1、秘钥

给配置ZKFC两个节点做免秘钥登录。

【这里我就不做过多的介绍了】

#### 2.2、架构

在典型的HA群集中，将两个单独的计算机配置为NameNode。在任何时间点，一个NameNode都恰好处于*Active*状态，而另一个Node 处于*Standby*状态。Active NameNode负责集群中的所有客户端操作，而Standby只是充当从属，并保持足够的状态以在必要时提供快速故障转移。

为了使备用节点保持其状态与活动节点同步，两个节点都与一组称为“ JournalNodes”（JN）的单独守护程序进行通信。当活动节点执行任何名称空间修改时，它会持久地将修改记录记录到大多数这些JN中。Standby节点能够从JN读取编辑，并一直在监视它们以查看编辑日志的更改。当“备用节点”看到编辑内容时，会将其应用于自己的名称空间。发生故障转移时，备用数据库将确保在将自身升级为活动状态之前，已从JounalNodes读取所有编辑内容。这样可确保在发生故障转移之前，名称空间状态已完全同步。

为了提供快速的故障转移，备用节点还必须具有有关集群中块位置的最新信息。为了实现这一点，DataNodes被配置了两个NameNodes的位置，并向两者发送块位置信息和心跳信号。

对于HA群集的正确操作至关重要，一次只能有一个NameNode处于活动状态。否则，名称空间状态将在两者之间迅速分散，从而有数据丢失或其他不正确结果的风险。为了确保此属性并防止所谓的“裂脑情况”，JournalNode将仅一次允许单个NameNode成为作者。在故障转移期间，将变为活动状态的NameNode将仅承担写入JournalNodes的角色，这将有效地防止另一个NameNode继续处于活动状态，从而使新的Active可以安全地进行故障转移

#### 2.3、环境

为了部署高可用性群集，您应该准备以下内容：

- **NameNode计算机** -运行活动NameNode和Standby NameNode的计算机应具有彼此等效的硬件，以及与非HA群集中将使用的硬件相同的硬件。
- **JournalNode计算机** -运行JournalNode的计算机。JournalNode守护程序相对较轻，因此可以合理地将这些守护程序与其他Hadoop守护程序（例如NameNodes，JobTracker或YARN ResourceManager）并置在计算机上。**注意：**必须至少有3个JournalNode守护程序，因为必须将编辑日志修改写入大多数JN。这将允许系统容忍单个计算机的故障。您可能还会运行3个以上的JournalNode，但是为了实际增加系统可以容忍的故障数量，您应该运行奇数个JN（即3、5、7等）。请注意，当与N个JournalNode一起运行时，系统最多可以容忍（N-1）/ 2个故障，并继续正常运行。

请注意，在HA群集中，备用NameNode也执行名称空间状态的检查点，因此不必在HA群集中运行Secondary NameNode，CheckpointNode或BackupNode。实际上，这样做将是一个错误。这还允许重新配置未启用HA的HDFS群集的用户启用HA，以重用他们先前专用于次要NameNode的硬件。

#### 2.4、部署方式

与联合身份验证配置类似，高可用性配置向后兼容，并允许现有的单个NameNode配置无需更改即可工作。设计新的配置，以便群集中的所有节点都可以具有相同的配置，而无需根据节点的类型将不同的配置文件部署到不同的计算机。

像HDFS联合身份验证一样，HA群集重用名称服务ID来标识实际上可能由多个HA NameNode组成的单个HDFS实例。此外，HA还添加了一个名为NameNode ID的新抽象。群集中的每个不同的NameNode都有一个不同的NameNode ID来区分它。为了支持所有NameNode的单个配置文件，相关的配置参数后缀有nameservice ID和NameNode ID。



要配置HA NameNode，必须将多个配置选项添加到hdfs-site.xml配置文件中。

设置这些配置的顺序并不重要，但是为dfs.nameservices和dfs.ha.namenodes。[nameservice ID]选择的值将确定后面的密钥。因此，您应该在设置其余配置选项之前决定这些值。

dfs.nameservices-此新名称服务的逻辑名称

 为这个名称服务选择一个逻辑名，例如“mycluster”，然后使用这个逻辑名作为这个配置选项的值。您选择的名称是任意的。它将用于配置和作为集群中绝对HDFS路径的权威组件。 

 使用逗号分隔的NameNode id列表进行配置。datanode将使用它来确定集群中的所有namenode。例如，如果您以前使用“mycluster”作为nameservice ID，并且希望使用“nn1”和“nn2”作为namenode的单独ID，那么您可以这样配置它: 

```xml
<property>
  <name>dfs.ha.namenodes.mycluster</name>
  <value>nn1,nn2</value>
</property>
```

