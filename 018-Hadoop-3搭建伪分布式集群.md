# 伪分布式集群搭建

### 1、伪分布式搭建

#### 1.1、先配置JDK环境

上传后解压JDK到/usr/java/

```shell
export JAVA_HOME=/usr/java/jdk1.8.0_221
export JRE_HOME=/usr/java/jdk1.8.0_221/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```

#### 1.2、免密登录

```shell
[root@MDNode01 ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
c3:5f:8d:47:39:1d:bf:8b:ac:cb:60:64:25:8b:9f:66 root@MDNode01
The key's randomart image is:
+--[ RSA 2048]----+
|               . |
|               oo|
|         . .  + o|
|       .. +  + ..|
|       .S+  o o. |
|        +o..... .|
|         E.  o . |
|        + o .    |
|           +.    |
+-----------------+
[root@MDNode01 ~]# ll -la
total 56
dr-xr-x---.  4 root root  4096 Nov 14 03:12 .
dr-xr-xr-x. 22 root root  4096 Nov 14 02:49 ..
-rw-------.  1 root root  5413 Sep 16 01:02 .bash_history
-rw-r--r--.  1 root root    18 May 20  2009 .bash_logout
-rw-r--r--.  1 root root   176 May 20  2009 .bash_profile
-rw-r--r--.  1 root root   176 Sep 23  2004 .bashrc
-rw-r--r--.  1 root root   100 Sep 23  2004 .cshrc
-rw-r--r--   1 root root 12288 Aug  9 16:52 .c.txt.swp
drwxr-xr-x   2 root root  4096 Nov 14 02:56 software
drwx------   2 root root  4096 Nov 14 03:12 .ssh
-rw-r--r--.  1 root root   129 Dec  4  2004 .tcshrc
[root@MDNode01 ~]# cd .ssh/
[root@MDNode01 .ssh]# ll
total 8
-rw------- 1 root root 1675 Nov 14 03:12 id_rsa
-rw-r--r-- 1 root root  395 Nov 14 03:12 id_rsa.pub
[root@MDNode01 .ssh]# cat id_rsa.pub > authorized_keys
[root@MDNode01 .ssh]# ll
total 12
-rw-r--r-- 1 root root  395 Nov 14 03:14 authorized_keys
-rw------- 1 root root 1675 Nov 14 03:12 id_rsa
-rw-r--r-- 1 root root  395 Nov 14 03:12 id_rsa.pub
[root@MDNode01 .ssh]# cat authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAt91KeWuzxbO3Tb1A7NhGKJFGdUINEBmfQYw3D7VfTA7wJfr9Na6xcVj9Wy8gwp0+yfx/8nizeNJrePRie8tl8FwUUs06pASuNenMz56m/JGnASpR4itoF9AxyIYwFHVRymCLh15Msd2RbvW2ZqcmxrtoDdOrxijnFKbZJodCoZlJLvBUnlEci0Yf249oQ1b+mF1EBwfMgEhn0aiqcaUbX5h2zbV2rUi2uFy1xdODgsyaO4PW+mkNzleD+AQZdkB5jdZF6165Q4uFS1gX+0NhG3UXfmcRv1IMuYmH5vEJehB6ZOPOxSmhfwlEm9Zfoiw/r1gJhCmW9KXAe4vsQchOnQ== root@MDNode01
[root@MDNode01 .ssh]# cd ..
[root@MDNode01 ~]# ssh MDNode01
The authenticity of host 'mdnode01 (192.168.25.50)' can't be established.
RSA key fingerprint is f7:e2:3a:54:8f:c3:31:07:64:46:1c:0d:48:b4:1b:08.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'mdnode01,192.168.25.50' (RSA) to the list of known hosts.
Last login: Thu Nov 14 02:49:28 2019 from 192.168.25.1
[root@MDNode01 ~]# exit
logout
Connection to MDNode01 closed.
[root@MDNode01 ~]# ssh MDNode01
Last login: Thu Nov 14 03:14:42 2019 from mdnode01
[root@MDNode01 ~]# exit
logout
Connection to MDNode01 closed.
[root@MDNode01 ~]# ssh localhost
The authenticity of host 'localhost (::1)' can't be established.
RSA key fingerprint is f7:e2:3a:54:8f:c3:31:07:64:46:1c:0d:48:b4:1b:08.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (RSA) to the list of known hosts.
Last login: Thu Nov 14 03:14:50 2019 from mdnode01
[root@MDNode01 ~]# exit
logout
Connection to localhost closed.
[root@MDNode01 ~]# ssh localhost
Last login: Thu Nov 14 03:15:23 2019 from localhost
[root@MDNode01 ~]# exit
logout
Connection to localhost closed.
[root@MDNode01 ~]# 
```

#### 1.3、解压Hadoop

```shell
[root@MDNode01 software]# ll
total 402372
-rw-r--r-- 1 root root 216929574 Nov 14 02:55 hadoop-2.7.5.tar.gz
-rw-r--r-- 1 root root 195094741 Nov 14 02:55 jdk-8u221-linux-x64.tar.gz
[root@MDNode01 software]# mkdir -p /opt/mgs
[root@MDNode01 software]# tar -zxf hadoop-2.7.5.tar.gz -C /opt/mgs/
[root@MDNode01 software]# cd /opt/mgs/
[root@MDNode01 mgs]# cd /opt/mgs/
[root@MDNode01 mgs]# ll
total 4
drwxr-xr-x 9 20415 101 4096 Dec 16  2017 hadoop-2.7.5
[root@MDNode01 mgs]# cd hadoop-2.7.5/
[root@MDNode01 hadoop-2.7.5]# ll
total 136
drwxr-xr-x 2 20415 101  4096 Dec 16  2017 bin
drwxr-xr-x 3 20415 101  4096 Dec 16  2017 etc
drwxr-xr-x 2 20415 101  4096 Dec 16  2017 include
drwxr-xr-x 3 20415 101  4096 Dec 16  2017 lib
drwxr-xr-x 2 20415 101  4096 Dec 16  2017 libexec
-rw-r--r-- 1 20415 101 86424 Dec 16  2017 LICENSE.txt
-rw-r--r-- 1 20415 101 14978 Dec 16  2017 NOTICE.txt
-rw-r--r-- 1 20415 101  1366 Dec 16  2017 README.txt
drwxr-xr-x 2 20415 101  4096 Dec 16  2017 sbin
drwxr-xr-x 4 20415 101  4096 Dec 16  2017 share
[root@MDNode01 hadoop-2.7.5]# 
```

#### 1.4、配置HADOOP_HOME变量

```shell
export JAVA_HOME=/usr/java/jdk1.8.0_221
export JRE_HOME=/usr/java/jdk1.8.0_221/jre
export HADOOP_HOME=/opt/mgs/hadoop-2.7.5
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

#### 1.5、配置Hadoop的JDK环境变量

/opt/service/jdk1.8.0_221

```shell
[root@MDNode01 hadoop]# pwd
/opt/mgs/hadoop-2.7.5/etc/hadoop
[root@MDNode01 hadoop]# ll
total 152
-rw-r--r-- 1 20415 101  4436 Dec 16  2017 capacity-scheduler.xml
-rw-r--r-- 1 20415 101  1335 Dec 16  2017 configuration.xsl
-rw-r--r-- 1 20415 101   318 Dec 16  2017 container-executor.cfg
-rw-r--r-- 1 20415 101   774 Dec 16  2017 core-site.xml
-rw-r--r-- 1 20415 101  3670 Dec 16  2017 hadoop-env.cmd
-rw-r--r-- 1 20415 101  4224 Dec 16  2017 hadoop-env.sh
-rw-r--r-- 1 20415 101  2598 Dec 16  2017 hadoop-metrics2.properties
-rw-r--r-- 1 20415 101  2490 Dec 16  2017 hadoop-metrics.properties
-rw-r--r-- 1 20415 101  9683 Dec 16  2017 hadoop-policy.xml
-rw-r--r-- 1 20415 101   775 Dec 16  2017 hdfs-site.xml
-rw-r--r-- 1 20415 101  1449 Dec 16  2017 httpfs-env.sh
-rw-r--r-- 1 20415 101  1657 Dec 16  2017 httpfs-log4j.properties
-rw-r--r-- 1 20415 101    21 Dec 16  2017 httpfs-signature.secret
-rw-r--r-- 1 20415 101   620 Dec 16  2017 httpfs-site.xml
-rw-r--r-- 1 20415 101  3518 Dec 16  2017 kms-acls.xml
-rw-r--r-- 1 20415 101  1527 Dec 16  2017 kms-env.sh
-rw-r--r-- 1 20415 101  1631 Dec 16  2017 kms-log4j.properties
-rw-r--r-- 1 20415 101  5540 Dec 16  2017 kms-site.xml
-rw-r--r-- 1 20415 101 11237 Dec 16  2017 log4j.properties
-rw-r--r-- 1 20415 101   951 Dec 16  2017 mapred-env.cmd
-rw-r--r-- 1 20415 101  1383 Dec 16  2017 mapred-env.sh
-rw-r--r-- 1 20415 101  4113 Dec 16  2017 mapred-queues.xml.template
-rw-r--r-- 1 20415 101   758 Dec 16  2017 mapred-site.xml.template
-rw-r--r-- 1 20415 101    10 Dec 16  2017 slaves
-rw-r--r-- 1 20415 101  2316 Dec 16  2017 ssl-client.xml.example
-rw-r--r-- 1 20415 101  2697 Dec 16  2017 ssl-server.xml.example
-rw-r--r-- 1 20415 101  2191 Dec 16  2017 yarn-env.cmd
-rw-r--r-- 1 20415 101  4567 Dec 16  2017 yarn-env.sh
-rw-r--r-- 1 20415 101   690 Dec 16  2017 yarn-site.xml
[root@MDNode01 hadoop]# 
```



##### 修改vi hadoop-env.sh

```shell
# The only required environment variable is JAVA_HOME.  All others are
# optional.  When running a distributed configuration it is best to
# set JAVA_HOME in this file, so that it is correctly defined on
# remote nodes.

# The java implementation to use.
export JAVA_HOME=/usr/java/jdk1.8.0_221
```

##### 修改 vi mapred-env.sh

```shell
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

export JAVA_HOME=/usr/java/jdk1.8.0_221

export HADOOP_JOB_HISTORYSERVER_HEAPSIZE=1000

export HADOOP_MAPRED_ROOT_LOGGER=INFO,RFA
```

##### 修改 vi yarn-env.sh 

```shell
# some Java parameters
export JAVA_HOME=/usr/java/jdk1.8.0_221
if [ "$JAVA_HOME" != "" ]; then
  #echo "run java in $JAVA_HOME"
  JAVA_HOME=$JAVA_HOME

```

#### 1.6、配置全局

##### core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://MDNode01:9000</value>
    </property>
    <!--配置输出的文件目录，不采用默认的文件路径-->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/var/mgs/hadoop/pseudo</value>
    </property>
</configuration>
```

主节点的配置

##### hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <!--伪分布式只要配置一个副本就可以了-->
        <value>1</value>
    </property>
    <!--配置namenode.secondary的节点和端口号-->
     <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>MDNode01:50090</value>
    </property>
</configuration>

```

##### slaves 

```shell
MDNode01
```

配置DataNode

#### 1.7、格式化【生成镜像文件】

```shell
[root@MDNode01 hadoop]# hdfs namenode -format
19/11/14 04:54:05 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = MDNode01/192.168.25.50
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 2.7.5
STARTUP_MSG:   classpath = /opt/mgs/hadoop-2.7.5/etc/hadoop:/opt/mgs/hadoop-2.7.5/share/hadoop/common/lib/protobuf-java-2.5.0.jar:/opt/mgs/hadoop-
....
19/11/14 04:54:07 INFO common.Storage: Storage directory /var/mgs/hadoop/pseudo/dfs/name has been successfully formatted.
#成功生相对于的数据格式化数据，NameNode的源数据信息
19/11/14 04:54:07 INFO namenode.FSImageFormatProtobuf: Saving image file /var/mgs/hadoop/pseudo/dfs/name/current/fsimage.ckpt_0000000000000000000 using no compression
19/11/14 04:54:07 INFO namenode.FSImageFormatProtobuf: Image file /var/mgs/hadoop/pseudo/dfs/name/current/fsimage.ckpt_0000000000000000000 of size 321 bytes saved in 0 seconds.
19/11/14 04:54:07 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
19/11/14 04:54:07 INFO util.ExitUtil: Exiting with status 0
19/11/14 04:54:07 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at MDNode01/192.168.25.50
************************************************************/
```



**说明**

```shell
[root@MDNode01 hadoop]# cd /var/mgs/hadoop/pseudo/dfs/name/current/
[root@MDNode01 current]# ll
total 16
-rw-r--r-- 1 root root 321 Nov 14 04:54 fsimage_0000000000000000000
-rw-r--r-- 1 root root  62 Nov 14 04:54 fsimage_0000000000000000000.md5
-rw-r--r-- 1 root root   2 Nov 14 04:54 seen_txid
-rw-r--r-- 1 root root 205 Nov 14 04:54 VERSION
```

在自定义的文件夹路径的/dfs/name/current/下面存放着NameNode的相关配置文件输出的信息

```shell
[root@MDNode01 current]# cat VERSION 
#Thu Nov 14 04:54:07 CST 2019
namespaceID=375516717
#集群唯一的ID号，在集群中所有的角色共享，NameNode，SecondaryNameNode ，DataNode都是同一个ID
clusterID=CID-56b9d3a0-f7ce-4a59-b8ef-96c4222e82ce
cTime=0
storageType=NAME_NODE
blockpoolID=BP-1669788457-192.168.25.50-1573678447143
layoutVersion=-63
```

#### 1.8、启动伪分布式集群

```shell
[root@MDNode01 hadoop-2.7.5]# start-dfs.sh 
Starting namenodes on [MDNode01]
MDNode01: starting namenode, logging to /opt/mgs/hadoop-2.7.5/logs/hadoop-root-namenode-MDNode01.out
MDNode01: starting datanode, logging to /opt/mgs/hadoop-2.7.5/logs/hadoop-root-datanode-MDNode01.out
Starting secondary namenodes [MDNode01]
MDNode01: starting secondarynamenode, logging to /opt/mgs/hadoop-2.7.5/logs/hadoop-root-secondarynamenode-MDNode01.out
[root@MDNode01 hadoop-2.7.5]# jps
1605 NameNode
1720 DataNode
1977 Jps
1868 SecondaryNameNode
[root@MDNode01 hadoop-2.7.5]# 
```

#### 1.9、查看相关生存的文件

产生相对于的三个角色的配置文件目录

```shell
[root@MDNode01 dfs]# pwd
/var/mgs/hadoop/pseudo/dfs
[root@MDNode01 dfs]# ll
total 12
drwx------ 3 root root 4096 Nov 14 05:10 data
drwxr-xr-x 3 root root 4096 Nov 14 05:10 name
drwxr-xr-x 3 root root 4096 Nov 14 05:11 namesecondary
[root@MDNode01 dfs]# 
```

```shell
[root@MDNode01 dfs]# cd data/
[root@MDNode01 data]# ll
total 8
drwxr-xr-x 3 root root 4096 Nov 14 05:10 current
-rw-r--r-- 1 root root   13 Nov 14 05:10 in_use.lock
[root@MDNode01 data]# cd current/
[root@MDNode01 current]# ll
total 8
drwx------ 4 root root 4096 Nov 14 05:10 BP-1669788457-192.168.25.50-1573678447143
-rw-r--r-- 1 root root  229 Nov 14 05:10 VERSION
[root@MDNode01 current]# cat VERSION 
#Thu Nov 14 05:10:28 CST 2019
storageID=DS-bc5408ea-44b6-45c2-9866-4b95bcafe10e
clusterID=CID-56b9d3a0-f7ce-4a59-b8ef-96c4222e82ce
cTime=0
datanodeUuid=9e6cd277-f5a5-403e-91b3-21f0a5804625
storageType=DATA_NODE
layoutVersion=-56
```

可以看到的集群ID是和主节点相同的

```shell
[root@MDNode01 current]# cd BP-1669788457-192.168.25.50-1573678447143/
[root@MDNode01 BP-1669788457-192.168.25.50-1573678447143]# ll
total 12
drwxr-xr-x 4 root root 4096 Nov 14 05:10 current
-rw-r--r-- 1 root root  166 Nov 14 05:10 scanner.cursor
drwxr-xr-x 2 root root 4096 Nov 14 05:10 tmp
[root@MDNode01 BP-1669788457-192.168.25.50-1573678447143]# cd current/
[root@MDNode01 current]# ll
total 12
drwxr-xr-x 2 root root 4096 Nov 14 05:10 finalized
drwxr-xr-x 2 root root 4096 Nov 14 05:10 rbw
-rw-r--r-- 1 root root  132 Nov 14 05:10 VERSION
[root@MDNode01 current]# cd finalized/
[root@MDNode01 finalized]# ll
total 0
```

可以看到当前的节点上的Block数据为0，是因为没有数据

**可视化**

![](images\Hadoop可视化端口.png)



使用浏览器访问：

![](images\Hadoop可视化浏览器访问.png)

----



![](images\Hadoop可视化浏览器访问文件夹.png)

----

#### 1.10、创建文件目录

```shell
[root@MDNode01 ~]# hdfs dfs -mkdir -p /user/root
[root@MDNode01 ~]# hdfs dfs -ls /
Found 1 items
#也是一个默认的文件夹
drwxr-xr-x   - root supergroup          0 2019-11-14 05:34 /user
```

![](images\Hadoop可视化浏览器路径文件夹.png)

------

#### 1.11、上传文件

**hdfs dfs常用的命令**

```
[root@MDNode01 software]# hdfs dfs
Usage: hadoop fs [generic options]
	[-appendToFile <localsrc> ... <dst>]
	[-cat [-ignoreCrc] <src> ...]
	[-checksum <src> ...]
	[-chgrp [-R] GROUP PATH...]
	[-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
	[-chown [-R] [OWNER][:[GROUP]] PATH...]
	[-copyFromLocal [-f] [-p] [-l] <localsrc> ... <dst>]
	[-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-count [-q] [-h] <path> ...]
	[-cp [-f] [-p | -p[topax]] <src> ... <dst>]
	[-createSnapshot <snapshotDir> [<snapshotName>]]
	[-deleteSnapshot <snapshotDir> <snapshotName>]
	[-df [-h] [<path> ...]]
	[-du [-s] [-h] <path> ...]
	[-expunge]
	[-find <path> ... <expression> ...]
	[-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-getfacl [-R] <path>]
	[-getfattr [-R] {-n name | -d} [-e en] <path>]
	[-getmerge [-nl] <src> <localdst>]
	[-help [cmd ...]]
	[-ls [-d] [-h] [-R] [<path> ...]]
	[-mkdir [-p] <path> ...]
	[-moveFromLocal <localsrc> ... <dst>]
	[-moveToLocal <src> <localdst>]
	[-mv <src> ... <dst>]
	[-put [-f] [-p] [-l] <localsrc> ... <dst>]
	[-renameSnapshot <snapshotDir> <oldName> <newName>]
	[-rm [-f] [-r|-R] [-skipTrash] <src> ...]
	[-rmdir [--ignore-fail-on-non-empty] <dir> ...]
	[-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
	[-setfattr {-n name [-v value] | -x name} <path>]
	[-setrep [-R] [-w] <rep> <path> ...]
	[-stat [format] <path> ...]
	[-tail [-f] <file>]
	[-test -[defsz] <path>]
	[-text [-ignoreCrc] <src> ...]
	[-touchz <path> ...]
	[-truncate [-w] <length> <path> ...]
	[-usage [cmd ...]]

Generic options supported are
-conf <configuration file>     specify an application configuration file
-D <property=value>            use value for given property
-fs <local|namenode:port>      specify a namenode
-jt <local|resourcemanager:port>    specify a ResourceManager
-files <comma separated list of files>    specify comma separated files to be copied to the map reduce cluster
-libjars <comma separated list of jars>    specify comma separated jar files to include in the classpath.
-archives <comma separated list of archives>    specify comma separated archives to be unarchived on the compute machines.

The general command line syntax is
bin/hadoop command [genericOptions] [commandOptions]
```



测试上传文件是一个Hadoop的压缩包

```shell
[root@MDNode01 software]# hdfs dfs -put hadoop-2.7.5.tar.gz /user/root
[root@MDNode01 software]# hdfs dfs -ls /user/root/
Found 1 items	
-rw-r--r--   1 root supergroup  216929574 2019-11-14 05:45 /user/root/hadoop-2.7.5.tar.gz

[root@MDNode01 subdir0]# pwd
/var/mgs/hadoop/pseudo/dfs/data/current/BP-1669788457-192.168.25.50-1573678447143/current/finalized/subdir0/subdir0
[root@MDNode01 subdir0]# ll -lh
total 209M
-rw-r--r-- 1 root root 128M Nov 14 05:45 blk_1073741825
-rw-r--r-- 1 root root 1.1M Nov 14 05:45 blk_1073741825_1001.meta
-rw-r--r-- 1 root root  79M Nov 14 05:45 blk_1073741826
-rw-r--r-- 1 root root 632K Nov 14 05:45 blk_1073741826_1002.meta
[root@MDNode01 subdir0]# 
```

可视化展示：

![](images\Hadoop文件上传大小的可视化1.png)

文件的默认Block块的大小是128MB

![](images\Hadoop文件上传大小的可视化2.png)

![](images\Hadoop文件上传大小的可视化3.png)

-------------

删除文件

```shell
[root@MDNode01 logs]# hdfs dfs -rm /user/root/hadoop-2.7.5.tar.gz
19/11/14 06:04:11 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 0 minutes, Emptier interval = 0 minutes.
Deleted /user/root/hadoop-2.7.5.tar.gz
```



#### 1.12、关闭集群

```shell
[root@MDNode01 logs]# stop-dfs.sh 
Stopping namenodes on [MDNode01]
MDNode01: stopping namenode
MDNode01: stopping datanode
Stopping secondary namenodes [MDNode01]
MDNode01: stopping secondarynamenode
[root@MDNode01 logs]# jps
2657 Jps
[root@MDNode01 logs]# 
```



