# HDFS入门

### 简介：

在现代的企业环境中，单机容量往往无法存储大量数据，需要跨机器存储。统一管理分布在集群上的文件系统称为分布式文件系统。而一旦在系统中，引入网络，就不可避免地引入了所有网络编程的复杂性，例如挑战之一是如果保证在节点不可用的时候数据不丢失。

传统的网络文件系统（NFS）虽然也称为分布式文件系统，但是其存在一些限制。由于NFS中，文件是存储在单机上，因此无法提供可靠性保证，当很多客户端同时访问NFS Server时，很容易造成服务器压力，造成性能瓶颈。另外如果要对NFS中的文件进行操作，需要首先同步到本地，这些修改在同步到服务端之前，其他客户端是不可见的。某种程度上，NFS不是一种典型的分布式系统，虽然它的文件的确放在远端（单一）的服务器上面。

### 1、存储类型

使用的是字节存储类型，

- 文件线性切割成块，(Block):偏移量 offset(byte,)

中文，GBK，一个中文2个字节，UTF-8是三个字节

- Block分散存储在集群的节点上

- 单一文件Block大小一致，文件与文件可以不一致性

（同一个文件被切割的大小要一致，和另一个文件没有关系）

- Block可以设置副本数，副本无序分散在不同的节点中，副本个数不要超过节点的数量。

单一的文件被切割的文件的Block块默认的情况是有副本的，(数据的安全，高可用，横向扩展)，

- 文件上传可以设置Block的大小个副本数，(资源不够开辟线程)

（默认的大小【128M】，认为的设置最小不能小于1M，在Hadoop中，，，副本数根据Block在节点上可能访问高的，就设置多个副本。）

- 文件上传的文件Block副本数可以调整，大小不变
- 只支持一次写入多次读取，同一时刻只能只有一个写入者。
- 可以append追加数据

### 2、架构模型

[Hadoop的架构是主从架构]

- 文件元数据MetaData，文件数据
  - 元数据
  - 数据本身
- 主[NameNode]节点保存文件元数据:单元点 posix

- 从[DataNode]节点保存文件Block数据，多节点
- DataNode与NameNode保持心跳，由NameNode向DataNode发送消息。
- HdfsClient与NameNode交互元数据信息
- HDFSClient与DataNode交互文件Block数据(cs)
- DataNode利用服务器本地文件系统存储数据块

![](images\HDFS架构客服端访问数据.png)

### 3、设计思想

![](images\HDFS设计思想.png)

### 4、持久化

##### **NameNode**

- 基于内存存储：不会和磁盘发生交互(双向)
  - 只存在内存中
  - 持久化(单向)【内存-->磁盘(拍照内存数据)】

【SQL数据库和NoSql的区别】

- NameNode主要的功能：
  - 接受客服端对额读写服务
  - 收集DataNode汇报的Block信息包括
- NameNode保存metaData信息包括
  - 文件owership(所属者)和permissions(权限)
  - 文件大小，时间
  - (Blocck列表:Block偏移量),位置消息(持久化不存，不会存入磁盘中，信息由DataNode发送过来)
  - Block每个副本位置(由DataNode上报)



##### NodeNode持久化的方式

- NameNode的metadata信息在启动后加载到内存

- metadata存储到磁盘文件，文件名"fsimage"（时时备份快照）【1】

  文件类型是一种二进制的字节码文件。在系统格式化的时候，系统搭建的时候edits先产生数据和fsimage。

  当两者到达一定的时间会做合并。>>>SecondaryNameNode(SNN)

  S SecondaryNameNode 

- Block的位置相信不会保存到fsimage

- edits记录对metadata的操作日志...

  ​	Redis数据库

### 5、DataNode持久化

- 本地磁盘目录存储数据(Block)，文件形式
- 同时存储Block的元数据信息文件
- 启动DN时候会发送DN汇报clock信息
- 通过向NN发送心跳保存联系，3秒一次，如果NN10分钟内没有收到DN的心跳，则认为NN已经挂掉了，并copy其的block到其他的NN上面。

### 6、Hadoop优点

- 高容错性
  - 数据自动保存多个副本
  - 副本就是后，自动恢复。(Blockmanagement的管理策略)
- 适合批处理
  - 移动计算而非数据(规避数据传输,移动数据运算框架)
  - 数据位置暴露给计算框架
- 适合大数据处理
  - GB、TB、甚至PB级以上的数据
  - 百万规模的文件数量
  - 10k+节点，单节点的数据大于10K
- 可以构架在廉价的机器上面运行
  - 通过多副本提高可靠性
  - 提供了容器和恢复机制

### 7、Hadoop缺点

- 低延迟数据访问
  - 比如毫秒级
  - 低延迟与高吞吐率
- 小文件读取
  - 占用NameNode大量内存
  - 寻到时间超过读取时间
- 并发写入、文件随机修改
  - 一个文件只能又一个写者
  - 仅支持append

### 8、Block的放置策略

1. 第一个副本：放置在上传文件的DN，如果是集群外提交，随机选择一台磁盘不满，CPU不忙的节点
2. 第二个副本：放置在另一个机架上
3. 第三个副本：放置在第二个同一个机架的的节点上
4. 更多的副本，随机的节点上面

### 9、Hadoop的HDFS的读写流程

![](images\Hadoop的HDFS写入策略1.png)

![](images\Hadoop的HDFS写入策略2.png)



#### 具体过程描述如下：

1. Client调用DistributedFileSystem对象的create方法，创建一个文件输出流（FSDataOutputStream）对象
2. 通过DistributedFileSystem对象与Hadoop集群的NameNode进行一次RPC远程调用，在HDFS的Namespace中创建一个文件条目（Entry），该条目没有任何的Block
3. 通过FSDataOutputStream对象，向DataNode写入数据，数据首先被写入FSDataOutputStream对象内部的Buffer中，然后数据被分割成一个个Packet数据包
4. 以Packet最小单位，基于Socket连接发送到按特定算法选择的HDFS集群中一组DataNode（正常是3个，可能大于等于1）中的一个节点上，在这组DataNode组成的Pipeline上依次传输Packet
5. 这组DataNode组成的Pipeline反方向上，发送ack，最终由Pipeline中第一个DataNode节点将Pipeline ack发送给Client
6. 完成向文件写入数据，Client在文件输出流（FSDataOutputStream）对象上调用close方法，关闭流
7. 调用DistributedFileSystem对象的complete方法，通知NameNode文件写入成功

#### 更详细的流程：

1. client发起文件上传请求,通过RPC与NameNode建立连接,NameNode检查目标文件是否已经存在,父目录是否存在,并检查用户是否有相应的权限,若检查通过,会为该文件创建一个新的记录,否则的话文件创建失败,客户端得到异常信息
2. client通过请求NameNode,第一个block应该传输到哪些DataNode服务器上
3. NameNode根据配置文件中指定的备份(replica)数量及机架感知原理进行文件分配,返回可用的DataNode的地址 以三台DataNode为例:A B C。注: Hadoop在设计时考虑到数据的安全与高效,数据文件默认在HDFS上存放三份,存储策略为:第一个备份放在客户端相同的datanode上(若客户端在集群外运行,就随机选取一个datanode来存放第一个replica),第二个replica放在与第一个replica不同机架的一个随机datanode上,第三个replica放在与第二个replica相同机架的随机datanode上,如果replica数大于三,则随后的replica在集群中随机存放,Hadoop会尽量避免过多的replica存放在同一个机架上.选取replica存放在同一个机架上.(Hadoop 1.x以后允许replica是可插拔的,意思是说可以定制自己需要的replica分配策略)
4. client请求3台的DataNode的一台A上传数据,(本质是一个RPC调用,建立pipeline),A收到请求会继续调用B,然后B调用C,将整个pipeline建立完成后,逐级返回client
5. client开始往A上传第一个block(先从磁盘读取数据放到一个本地内存缓存),以packet为单位(默认 64K),A收到一个packet就会传给B,B传递给C;A每传一个packet会放入一个应答队列等待应答。注: 如果某个datanode在写数据的时候宕掉了下面这些对用户透明的步骤会被执行:数据被分割成一个个packet数据包在pipeline上一次传输,在pipeline反方向上,逐个发送ack(命令正确应答),最终由pipeline中第一个DataNode节点A将pipeline ack发送给client
   1. 管道线关闭,所有确认队列上的数据会被挪到数据队列的首部重新发送,这样也就确保管道线中宕掉的datanode下流的datanode不会因为宕掉的datanode而丢失数据包
   2. 在还在正常运行datanode上的当前block上做一个标志,这样当宕掉的datanode重新启动以后namenode就会知道该datanode上哪个block是刚才宕机残留下的局部损坏block,从而把他删除掉
   3. 已经宕掉的datanode从管道线中被移除,未写完的block的其他数据继续呗写入到其他两个还在正常运行的datanode中,namenode知道这个block还处在under-replicated状态(即备份数不足的状态)下,然后它会安排一个新的replica从而达到要求的备份数,后续的block写入方法同前面正常时候一样
   4. 有可能管道线中的多个datanode宕掉(一般这种情况很少),但只要dfs.relication.min(默认值为1)个replica被创建,我么就认为该创建成功了,剩余的relica会在以后异步创建以达到指定的replica数
6. 
7. 当一个block传输完成后,client再次发送请求NameNode上传第二个block到服务器

参考[ https://www.cnblogs.com/zengfa/p/9323346.html ]

### 10、HDFS度数据

![](images\Hadoop的HDFS读出策略1.png)

　　HDFS读数据步骤：

1. Client向NameNode发起RPC请求,来确定请求文件block所在的位置
2. NameNode会视情况返回文件的部分或者全部block列表,对于每个block,NameNode都会返回含有该block副本的DataNode地址      
3. 这些返回的DN地址,会按照集群拓扑结构得出DataNode与客户端的距离,然后进行排序,排序两个规则:网络拓扑结构中距离Client的排在前;心跳机制中超时汇报的DN状态为STALE,这样的排在后
4. Clietn选取排序靠前的DataNode来读取block,如果客户端本身就是DataNode,那么将从本地直接获取数据
5. 底层本质是建立Socket Stream(FSDataInputStream) ,重复调用父类DataInputStream的read方法,知道这个块上的数据读取完毕
6. 当读完列表的block后,若文件读取还没有结束,客户端会继续想NameNode获取下一批的block列表
7. 读取完一个Block都会进行checksum验证,如果读取DataNode时出现错误,客户端会通知NameNode,然后再从下一个拥有该block副本的DataNode继续读取。注: 如果在读取过程中DFSInputStream检测到block错误,DFSInputStream也会检查从datanode读取来的数据的校验和,如果发现有数据损坏,它会把坏掉的block报告给namenode同时重新读取其他datanode上的其他block备份
8. read方法是并行的读取block信息,不是一块一块的读取,NameNode只是返回Client请求包含块的DataNode地址,并不是返回请求块的数据
9. 最终读取哎所有的block会合并成一个完整的最终文件

### 11、Hadoop文件权限

HDFS的文件权限POSIX标准，（可移植操作系统接口）

安全模式：

![](images\Hadoop安全模式.png)

### Hadoop-HDFS角色

角色=进程

![](images\角色.png)

