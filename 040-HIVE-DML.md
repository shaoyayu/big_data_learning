# Hive DML

删除表中的数据：

truncate table table_name 删除表的数据

### 从查询将数据插入Hive表

可以使用insert子句将查询结果插入表中。

```sql
Hive extension (multiple inserts):
FROM from_statement
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...) [IF NOT EXISTS]] select_statement1
[INSERT OVERWRITE TABLE tablename2 [PARTITION ... [IF NOT EXISTS]] select_statement2]
[INSERT INTO TABLE tablename2 [PARTITION ...] select_statement2] ...;

FROM from_statement
INSERT INTO TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1
[INSERT INTO TABLE tablename2 [PARTITION ...] select_statement2]
[INSERT OVERWRITE TABLE tablename2 [PARTITION ... [IF NOT EXISTS]] select_statement2] ...;
  
Hive extension (dynamic partition inserts):
INSERT OVERWRITE TABLE tablename PARTITION (partcol1[=val1], partcol2[=val2] ...) select_statement FROM from_statement;
INSERT INTO TABLE tablename PARTITION (partcol1[=val1], partcol2[=val2] ...) select_statement FROM from_statement;

//例如
FROM TABLE
INSERT INTO table1
FROM *
INSERT INTO table2
FROM *
INSERT INTO table3


FROM page_view_stg pvs
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country)
       SELECT pvs.viewTime, pvs.userid, pvs.page_url, pvs.referrer_url, null, null, pvs.ip, pvs.cnt
```

##### 概要

- INSERT OVERWRITE将覆盖表或分区中的任何现有数据
  - 除非`IF NOT EXISTS`为分区提供（从Hive [0.9.0开始](https://issues.apache.org/jira/browse/HIVE-2612)）。
  - 从Hive 2.3.0（[HIVE-15880](https://issues.apache.org/jira/browse/HIVE-15880)）开始，如果表具有[TBLPROPERTIES](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-listTableProperties)（“ auto.purge” =“ true”），则对该表运行INSERT OVERWRITE查询时，该表的先前数据不会移至“已删除邮件”。此功能仅适用于托管表（请参阅[托管表](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ManagedandExternalTables)），并且在未设置“ auto.purge”属性或将其设置为false时将关闭此功能。
- INSERT INTO将追加到表或分区，使现有数据保持不变。（注意：INSERT INTO语法仅从版本0.8开始可用。）
  - 从Hive 0.13.0开始，可以通过使用TBLPROPERTIES创建 表来使其*不可变*（“ immutable” =“ true”）。默认值为“ immutable” =“ false”。
    如果已经存在任何数据，则不允许在不可变表中执行INSERT INTO行为，尽管如果不可变表为空，则INSERT INTO仍然有效。INSERT OVERWRITE的行为不受“不可变”表属性的影响。
    不可变表可以防止意外更新，因为脚本将数据加载到该表中会导致该表被错误地多次运行。对不可变表的第一次插入成功，而随后的插入失败，导致表中只有一组数据， 

- 可以对表或分区进行插入。如果表已分区，则必须通过为所有分区列指定值来指定表的特定分区。如果[hive.typecheck.on.insert](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.typecheck.on.insert)设置为true，那么将验证，转换和标准化这些值以使其符合其列类型（从Hive [0.12.0](https://issues.apache.org/jira/browse/HIVE-5297)开始）。 
- 可以在同一查询中指定多个insert子句（也称为Multi Table Insert）。
- 每个select语句的输出将写入所选的表（或分区）。当前，OVERWRITE关键字是强制性的，它表示将所选表或分区的内容替换为对应的select语句的输出。
- 输出格式和序列化类由表的元数据确定（通过表上的DDL命令指定）。
- 从[Hive 0.14开始](https://issues.apache.org/jira/browse/HIVE-5317)，如果表具有实现AcidOutputFormat的OutputFormat，并且系统配置为使用实现ACID 的[事务](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)管理器，则将对该表禁用INSERT OVERWRITE。这是为了避免用户无意间覆盖交易历史记录。通过使用[TRUNCATE TABLE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-TruncateTable)（用于未分区的表）或[DROP PARTITION](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-DropPartitions)后接INSERT INTO，可以实现相同的功能。
- 从Hive [1.1.0开始](https://issues.apache.org/jira/browse/HIVE-9353)，TABLE关键字是可选的。
- 从Hive 1.2.0开始，每个INSERT INTO T都可以采用列列表，例如INSERT INTO T（z，x，c1）。有关示例，请参见 HIVE-9481的说明。

### 通过查询将数据写入文件系统

可以使用上面语法的一些细微变化将查询结果插入文件系统目录：

##### 句法

```shell
Standard syntax:
INSERT OVERWRITE [LOCAL] DIRECTORY directory1
  [ROW FORMAT row_format] [STORED AS file_format] (Note: Only available starting with Hive 0.11.0)
  SELECT ... FROM ...
 
Hive extension (multiple inserts):
FROM from_statement
INSERT OVERWRITE [LOCAL] DIRECTORY directory1 select_statement1
[INSERT OVERWRITE [LOCAL] DIRECTORY directory2 select_statement2] ...
 
  
row_format
  : DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]] [COLLECTION ITEMS TERMINATED BY char]
        [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
        [NULL DEFINED AS char] (Note: Only available starting with Hive 0.13)
        
//例 OVERWRITE会删除掉当前文件夹下面的所有数据，
INSERT OVERWRITE LOCAL directory '/root/data/'
SELECT * FROM table_name;
```



### 序列化和反序列化

#### hive SerDe

Hive SerDe-Serializer and Deserializer

- SerDe 用以做序列化和反序列化
- 构建在数据存储和执行引擎之间，在两者之间实现解耦
- Hive通过 ROW  FORMAT DELIMITED 以及SERDE 进行呢内容读写

```sql
CREATE TABLE logtbl(
	host STRING,
	identity STRING,
	t_user STRING,
	time STRING,
	request STRING,
	frferer STRING,
	agent STRING)
	ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
	WITH SERDEPROPERTIES(
		"input.reger"  = "([^ ]*) ([^ ]*) ([^ ]*) \\[(.*)\\] \"(.*)\" (-|[0-9]*) (-|[0-9]*)"
	)
	SEORED AS TEXTFILE;
```

文档类型，

```
192.168.57.4 - - [29/Feb/2016:18:14:35 +0800] "GET /bg-upper.png HTTP/1.1" 304 -
```

然后将日志导入系统Hive中。

```
load data local inpath '[linx系统中的文件地址]' into table logtbl;
```



### hive beeline

Beeline 要与HiveServer2配合使用

服务端启动hiveserver2
客户的通过beeline两种方式连接到hive
1、beeline -u jdbc:hive2://localhost:10000/default -n root
2、beeline
beeline> !connect jdbc:hive2://<host>:<port>/<db>;auth=noSasl root 123

默认 用户名、密码不验证

#### HiveService2

[HiveServer2](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Overview)（HS2）是一个服务器接口，使远程客户端可以对Hive执行查询并检索结果（[此处](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Overview)有更详细的介绍）。当前的实现基于Thrift RPC，是[HiveServer](https://cwiki.apache.org/confluence/display/Hive/HiveServer)的改进版本，并支持多客户端并发和身份验证。它旨在为JDBC和ODBC等开放API客户端提供更好的支持。

 两种启动HiveService2的方式

1、 在元数据节点上启动 hiveservice2，同样是一个阻塞的窗口

```
[root@hadoopNode03 ~]# hiveserver2 
```

在其他节点上使用

两种连接hiveservice2的方式
- beeline -u jdbc:hive2://[元数据主机]:10000/default -n root
- beeline
  beeline >!connect jdbc hive2://[元数据主机]:10000/[数据库名] root 123

```
[root@hadoopNode04 ~]# beeline 
Beeline version 1.2.2 by Apache Hive
beeline> !connect jdbc:hive2://hadoopNode03:10000/default root 123
Connecting to jdbc:hive2://hadoopNode03:10000/default
Connected to: Apache Hive (version 1.2.2)
Driver: Hive JDBC (version 1.2.2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
0: jdbc:hive2://hadoopNode03:10000/default> show tables;
+-----------+--+
| tab_name  |
+-----------+--+
| psn       |
| psn_2     |
| tb_1      |
+-----------+--+
3 rows selected (2.564 seconds)
0: jdbc:hive2://hadoopNode03:10000/default> 
```

当前没有做用户验证，所以使用任何的用户名和密码都是可以连接的，

```
beeline -u jdbc:hive2//hadoopNode03:10000/default [用户名] [密码]
```

2、第二种启动。

在元数据节点上面启动元数据服务，在其他的服务节点上面启动hiveservice2服务

```
[root@hadoopNode03 ~]# hive --service metastore
Starting Hive Metastore Server
```

启动hiveservice2 

```
hiveservice
```

```
beeline> !connect jdbc:hive2://hadoopNode04:10000/default root 123
Connecting to jdbc:hive2://hadoopNode04:10000/default
Connected to: Apache Hive (version 1.2.2)
Driver: Hive JDBC (version 1.2.2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
0: jdbc:hive2://hadoopNode04:10000/default> show tables;
+-----------+--+
| tab_name  |
+-----------+--+
| psn       |
| psn_2     |
| tb_1      |
+-----------+--+
```

在beeline中，尽量不要做增删改查找，当然可以将数据上传到hiveservice2的节点上面
在启动beeline添加一个 -n root 的用户指定

- 1 上传文件到hiveservice2的节点中。
- 2 执行 beeline -u jdbc:hive2://hadoopNode04:10000/default -n root 
- 3 使用load data 的方式上传数据



