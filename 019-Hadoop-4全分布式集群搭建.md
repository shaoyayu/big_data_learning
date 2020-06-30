# 全分布式集群搭建

### 1、系统配置

#### 1.1、时间

#### 1.2、JDK

#### 1.3、免秘钥登录

【我做的是节点之间免密登录，可以给主节点做单向的免秘钥登录】

【服务器时间必须统一】

【关闭防火墙，关闭安全机制】

---

这里我就不做过多的笔记，后面还有很多的配置等着，10几分钟左右的事情。

### 2、修改配置文件：

这是基于伪分布式文件上面更改的：

#### core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://MDNode01:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <!--修改文件保存的路径-->
        <value>/var/mgs/hadoop/full</value>
    </property>
</configuration>

```



#### hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <!--因为我搭建的只有3台DataNode服务器，副本数设置为2-->
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <!--指向另一台节点的线程上运行-->
        <value>MDNode02:50090</value>
    </property>
</configuration>
```

#### slaves

```
#配置从节点
MDNode02
MDNode03
Mdnode04
```

#### 分发Hadoop数据包

```
[root@MDNode01 opt]# pwd
/opt
[root@MDNode01 opt]# ll
total 4
drwxr-xr-x 3 root root 4096 Nov 14  2019 mgs
#将Hadoop分发到不同的节点上面
[root@MDNode01 opt]# scp -r mgs/ MDNode04:`pwd`
```

#### 分发HADOOP_HOME的配置

### 3、格式集群

```shell
[root@MDNode01 hadoop-2.7.5]# hdfs namenode -format
```

```
19/11/13 23:52:54 INFO namenode.FSImage: Allocated new BlockPoolId: BP-987421903-192.168.25.50-1573660374725
19/11/13 23:52:54 INFO common.Storage: Storage directory /var/mgs/hadoop/full/dfs/name has been successfully formatted.
19/11/13 23:52:55 INFO namenode.FSImageFormatProtobuf: Saving image file /var/mgs/hadoop/full/dfs/name/current/fsimage.ckpt_0000000000000000000 using no compression
19/11/13 23:52:55 INFO namenode.FSImageFormatProtobuf: Image file /var/mgs/hadoop/full/dfs/name/current/fsimage.ckpt_0000000000000000000 of size 321 bytes saved in 0 seconds.
19/11/13 23:52:55 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
19/11/13 23:52:55 INFO util.ExitUtil: Exiting with status 0
19/11/13 23:52:55 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at MDNode01/192.168.25.50
************************************************************/
```

在info后面的时候可以发现配置完成

```
[root@MDNode01 hadoop-2.7.5]# cd /var/mgs/hadoop/full/dfs/
[root@MDNode01 dfs]# ll
total 4
drwxr-xr-x 3 root root 4096 Nov 13 23:52 name
[root@MDNode01 dfs]# cd name
[root@MDNode01 name]# ll
total 4
drwxr-xr-x 2 root root 4096 Nov 13 23:52 current
[root@MDNode01 name]# cd current/
[root@MDNode01 current]# ll
total 16
-rw-r--r-- 1 root root 321 Nov 13 23:52 fsimage_0000000000000000000
-rw-r--r-- 1 root root  62 Nov 13 23:52 fsimage_0000000000000000000.md5
-rw-r--r-- 1 root root   2 Nov 13 23:52 seen_txid
-rw-r--r-- 1 root root 204 Nov 13 23:52 VERSION
[root@MDNode01 current]# cat VERSION 
#Wed Nov 13 23:52:54 CST 2019
namespaceID=330427894
clusterID=CID-c042ece5-6da9-4158-9fc2-527bf4688fbc
cTime=0
storageType=NAME_NODE
blockpoolID=BP-987421903-192.168.25.50-1573660374725
layoutVersion=-63
[root@MDNode01 current]# 
```

可以看到当期的节点是一个主节点上面生成的目录，其他的子节点上面在没有启动的时候不会生成配置文件

### 4、启动集群

```
[root@MDNode01 hadoop-2.7.5]# start-dfs.sh 
Starting namenodes on [MDNode01]
MDNode01: starting namenode, logging to /opt/mgs/hadoop-2.7.5/logs/hadoop-root-namenode-MDNode01.out
Mdnode04: starting datanode, logging to /opt/mgs/hadoop-2.7.5/logs/hadoop-root-datanode-MDNode04.out
MDNode03: starting datanode, logging to /opt/mgs/hadoop-2.7.5/logs/hadoop-root-datanode-MDNode03.out
MDNode02: starting datanode, logging to /opt/mgs/hadoop-2.7.5/logs/hadoop-root-datanode-MDNode02.out
Starting secondary namenodes [MDNode02]
MDNode02: starting secondarynamenode, logging to /opt/mgs/hadoop-2.7.5/logs/hadoop-root-secondarynamenode-MDNode02.out
```

可以看到节点启动正常，【在集群自己所有机器都做相互之间的免秘钥登录在哪个节点都可以启动】

接下来我们查看所有的java进程，

```
[root@MDNode01 current]# jps
1305 NameNode
1387 Jps
[root@MDNode01 current]# 
```

```
[root@MDNode02 hadoop-2.7.5]# jps
1872 Jps
1719 SecondaryNameNode
1599 DataNode
[root@MDNode02 hadoop-2.7.5]# 

```

```
[root@MDNode03 hadoop-2.7.5]# jps
1434 DataNode
1518 Jps
[root@MDNode03 hadoop-2.7.5]# 
```

```
[root@MDNode04 hadoop-2.7.5]# jps
1507 Jps
1423 DataNode
[root@MDNode04 hadoop-2.7.5]# 
```

OK，

不懂的节点出错建议去查看自己节点的日志。

![](images\全分布式集群可视化1.png)

这样就可以观察到不同节点的数据了。

创建一个默认的目录：

```
[root@MDNode01 current]# hdfs dfs -mkdir -p /user/root
```

### 5、上传文件

```
[root@MDNode01 software]# for i in `seq 100000`;do echo "hello shaoyayu $i" >> test.txt;done
[root@MDNode01 software]# ll -lh
total 395M
-rw-r--r-- 1 root root 207M Nov 14  2019 hadoop-2.7.5.tar.gz
-rw-r--r-- 1 root root 187M Nov 14  2019 jdk-8u221-linux-x64.tar.gz
-rw-r--r-- 1 root root 2.0M Nov 14 00:21 test.txt
#指定上传的大小是1M=1048576个字节。默认的是128M的大小，路径默认的存放在/user/root的目录下面
[root@MDNode01 software]# hdfs dfs -D dfs.blocksize=1048576 -put test.txt 
```

可视化观察：

![](images\全分布式集群可视化2.png)

![](images\全分布式集群可视化3.png)

可以查看到文件本的副本数量，和存放的位置和文件的大小。

可以去存放的节点之上找到相对于的副本

```
[root@MDNode02 subdir0]# pwd
/var/mgs/hadoop/full/dfs/data/current/BP-987421903-192.168.25.50-1573660374725/current/finalized/subdir0/subdir0
[root@MDNode02 subdir0]# ls -lh
total 2.1M
-rw-r--r-- 1 root root  1.0M Nov 14 00:26 blk_1073741825
-rw-r--r-- 1 root root  8.1K Nov 14 00:26 blk_1073741825_1001.meta
-rw-r--r-- 1 root root 1016K Nov 14 00:26 blk_1073741826
-rw-r--r-- 1 root root  8.0K Nov 14 00:26 blk_1073741826_1002.meta
[root@MDNode02 subdir0]# 
```

```
[root@MDNode03 hadoop-2.7.5]# cd /var/mgs/hadoop/full/dfs/data/current/BP-987421903-192.168.25.50-1573660374725/current/finalized
[root@MDNode03 finalized]# cd subdir0/subdir0/
[root@MDNode03 subdir0]# ls -lh
total 2.1M
-rw-r--r-- 1 root root  1.0M Nov 14 00:26 blk_1073741825
-rw-r--r-- 1 root root  8.1K Nov 14 00:26 blk_1073741825_1001.meta
-rw-r--r-- 1 root root 1016K Nov 14 00:26 blk_1073741826
-rw-r--r-- 1 root root  8.0K Nov 14 00:26 blk_1073741826_1002.meta
[root@MDNode03 subdir0]#
```

这就可以找到相对于的节点上的数据了

vi + blk_1073741825

```
......
hello shaoyayu 50458
hello shaoyayu 50459
hello shaoyayu 50460
hello shaoyayu 50461
h
```

 vi + blk_1073741826

```
ello shaoyayu 50462
hello shaoyayu 50463
......
hello shaoyayu 100000
```

可以看到文件在50416的位置后面已经被切割了，严格按照物理切割