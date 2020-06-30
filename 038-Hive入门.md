# HIVE的入门

https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL

## 创建/删除/更改/使用数据库

### 创建数据库

```
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
  [COMMENT database_comment]
  [LOCATION hdfs_path]
  [WITH DBPROPERTIES (property_name=property_value, ...)];
```

SCHEMA和DATABASE的用法是可互换的–它们意味着同一件事。在Hive 0.6（[HIVE-675](https://issues.apache.org/jira/browse/HIVE-675)）中添加了CREATE DATABASE 。在Hive 0.7（[HIVE-1836](https://issues.apache.org/jira/browse/HIVE-1836)）中添加了WITH DBPROPERTIES子句。

### 删除数据库

```
DROP (DATABASE|SCHEMA) [IF EXISTS] database_name [RESTRICT|CASCADE];
```

SCHEMA和DATABASE的用法是可互换的–它们意味着同一件事。在Hive 0.6（[HIVE-675](https://issues.apache.org/jira/browse/HIVE-675)）中添加了DROP DATABASE 。默认行为是RESTRICT，如果数据库不为空，则DROP DATABASE将失败。要将表也拖放到数据库中，请使用DROP DATABASE ... CASCADE。在Hive 0.8（[HIVE-2090](https://issues.apache.org/jira/browse/HIVE-2090)）中添加了对RESTRICT和CASCADE的支持。

### 修改数据库

```
ALTER (DATABASE|SCHEMA) database_name SET DBPROPERTIES (property_name=property_value, ...);   -- (Note: SCHEMA added in Hive 0.14.0)
 
ALTER (DATABASE|SCHEMA) database_name SET OWNER [USER|ROLE] user_or_role;   -- (Note: Hive 0.13.0 and later; SCHEMA added in Hive 0.14.0)
  
ALTER (DATABASE|SCHEMA) database_name SET LOCATION hdfs_path; -- (Note: Hive 2.2.1, 2.4.0 and later)
```

SCHEMA和DATABASE的用法是可互换的–它们意味着同一件事。在Hive 0.14（[HIVE-6601](https://issues.apache.org/jira/browse/HIVE-6601)）中添加了ALTER SCHEMA 。

ALTER DATABASE ... SET LOCATION语句不会将数据库当前目录的内容移动到新指定的位置。它不会更改与指定数据库下任何表/分区关联的位置。它仅更改默认父目录，在该目录中将为此数据库添加新表。此行为类似于更改表目录不会将现有分区移动到其他位置。

关于数据库的其他元数据无法更改。 

### 使用数据库

```
USE database_name;
USE DEFAULT;
```

USE为所有后续的HiveQL语句设置当前数据库。要恢复为默认数据库，请使用关键字“ default”代替数据库名称。要检查当前正在使用哪个数据库：（从Hive 0.13.0开始）。SELECT current_database()

USE database_name在Hive 0.6（HIVE-675）中添加。



## 创建/删除/截断表

### 建立表格

```sql
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- (Note: TEMPORARY available in Hive 0.14.0 and later)
  [(col_name data_type [column_constraint_specification] [COMMENT col_comment], ... [constraint_specification])]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
  [SKEWED BY (col_name, col_name, ...)                  -- (Note: Available in Hive 0.10.0 and later)]
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format] 
   [STORED AS file_format]
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]   -- (Note: Available in Hive 0.6.0 and later)
  [AS select_statement];   -- (Note: Available in Hive 0.5.0 and later; not supported for external tables)
 
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name
  LIKE existing_table_or_view_name
  [LOCATION hdfs_path];
 
data_type
  : primitive_type
  | array_type
  | map_type
  | struct_type
  | union_type  -- (Note: Available in Hive 0.7.0 and later)
 
primitive_type
  : TINYINT
  | SMALLINT
  | INT
  | BIGINT
  | BOOLEAN
  | FLOAT
  | DOUBLE
  | DOUBLE PRECISION -- (Note: Available in Hive 2.2.0 and later)
  | STRING
  | BINARY      -- (Note: Available in Hive 0.8.0 and later)
  | TIMESTAMP   -- (Note: Available in Hive 0.8.0 and later)
  | DECIMAL     -- (Note: Available in Hive 0.11.0 and later)
  | DECIMAL(precision, scale)  -- (Note: Available in Hive 0.13.0 and later)
  | DATE        -- (Note: Available in Hive 0.12.0 and later)
  | VARCHAR     -- (Note: Available in Hive 0.12.0 and later)
  | CHAR        -- (Note: Available in Hive 0.13.0 and later)
 
array_type
  : ARRAY < data_type >
 
map_type
  : MAP < primitive_type, data_type >
 
struct_type
  : STRUCT < col_name : data_type [COMMENT col_comment], ...>
 
union_type
   : UNIONTYPE < data_type, data_type, ... >  -- (Note: Available in Hive 0.7.0 and later)
 
row_format
  : DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]] [COLLECTION ITEMS TERMINATED BY char]
        [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
        [NULL DEFINED AS char]   -- (Note: Available in Hive 0.13 and later)
  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]
 
file_format:
  : SEQUENCEFILE
  | TEXTFILE    -- (Default, depending on hive.default.fileformat configuration)
  | RCFILE      -- (Note: Available in Hive 0.6.0 and later)
  | ORC         -- (Note: Available in Hive 0.11.0 and later)
  | PARQUET     -- (Note: Available in Hive 0.13.0 and later)
  | AVRO        -- (Note: Available in Hive 0.14.0 and later)
  | JSONFILE    -- (Note: Available in Hive 4.0.0 and later)
  | INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname
 
column_constraint_specification:
  : [ PRIMARY KEY|UNIQUE|NOT NULL|DEFAULT [default_value]|CHECK  [check_expression] ENABLE|DISABLE NOVALIDATE RELY/NORELY ]
 
default_value:
  : [ LITERAL|CURRENT_USER()|CURRENT_DATE()|CURRENT_TIMESTAMP()|NULL ] 
 
constraint_specification:
  : [, PRIMARY KEY (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
    [, PRIMARY KEY (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
    [, CONSTRAINT constraint_name FOREIGN KEY (col_name, ...) REFERENCES table_name(col_name, ...) DISABLE NOVALIDATE 
    [, CONSTRAINT constraint_name UNIQUE (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
    [, CONSTRAINT constraint_name CHECK [check_expression] ENABLE|DISABLE NOVALIDATE RELY/NORELY ]
```

CREATE TABLE创建具有给定名称的表。如果已经存在具有相同名称的表或视图，则会引发错误。您可以使用IF NOT EXISTS跳过该错误。

#### 托管表和外部表

默认情况下，Hive创建托管表，其中文件，元数据和统计信息由内部Hive进程管理。有关托管表和外部表之间差异的详细信息，请参阅[托管表与外部表](https://cwiki.apache.org/confluence/display/Hive/Managed+vs.+External+Tables)。

#### 储存格式

Hive支持内置和自定义开发的文件格式。有关压缩表存储的详细信息，请参见[CompressedStorage ](https://cwiki.apache.org/confluence/display/Hive/CompressedStorage)。以下是Hive内置的一些格式：

| 储存格式                  | 描述                                                         |
| :------------------------ | :----------------------------------------------------------- |
| 存储为文本文件            | 存储为纯文本文件。TEXTFILE是默认文件格式，除非配置参数[hive.default.fileformat ](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.default.fileformat)具有不同的设置。使用DELIMITED子句读取定界文件。通过使用'ESCAPED BY'子句（例如ESCAPED BY'\'）对定界符启用转义 如果要使用可以包含这些定界符的数据，则需要转义。  也可以使用'NULL DEFINED AS'子句指定自定义NULL格式（默认为'\ N'）。 |
| 存储为序列文件            | 存储为压缩的序列文件。                                       |
| 储存为ORC                 | 存储为[ORC文件格式](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-HiveQLSyntax)。支持ACID交易和基于成本的优化器（CBO）。存储列级元数据。 |
| 储存为PARQUET             | 在[Hive 0.13.0及更高版本中](https://cwiki.apache.org/confluence/display/Hive/Parquet#Parquet-Hive0.13andlater)以[Parquet ](https://cwiki.apache.org/confluence/display/Hive/Parquet)列存储格式存储为Parquet格式；使用行格式SERDE ...存储为INPUTFORMAT ... OUTPUTFORMAT语法...在[蜂巢0.10，0.11，0.12 ](https://cwiki.apache.org/confluence/display/Hive/Parquet#Parquet-Hive0.10-0.12)。 |
| 储存为AVRO                | 在[Hive 0.14.0和更高版本中](https://issues.apache.org/jira/browse/HIVE-6806)以Avro格式存储（请参阅[Avro SerDe ](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe)）。 |
| 存储为RCFILE              | 存储为[记录列文件](https://en.wikipedia.org/wiki/RCFile)格式。 |
| 储存为JSONFILE            | 在Hive 4.0.0及更高版本中以Json文件格式存储。                 |
| 储存者                    | 以非本地表格式存储。创建或链接到非本地表，例如由[HBase](https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration)或[Druid](https://cwiki.apache.org/confluence/display/Hive/Druid+Integration)或[Accumulo ](https://cwiki.apache.org/confluence/display/Hive/AccumuloIntegration)支持的表。 有关此选项的更多信息，请参见[StorageHandlers ](https://cwiki.apache.org/confluence/display/Hive/StorageHandlers)。 |
| INPUTFORMAT和OUTPUTFORMAT | 在file_format中将相应的InputFormat和OutputFormat类的名称指定为字符串文字。  例如，“ org.apache.hadoop.hive.contrib.fileformat.base64.Base64TextInputFormat”。  对于LZO压缩，要使用的值为 “ INPUTFORMAT“ com.hadoop.mapred.DeprecatedLzoTextInputFormat” OUTPUTFORMAT“ org.apache.hadoop.hive.ql.io .HiveIgnoreKeyTextOutputFormat”  （请参见LZO压缩）。 |



案列：

```shell
DESC table_name; //简单查询表结构
DESC FORMATTED table_name; //详细查询表结构
//创建表

CREATE TABLE psn(
id int,
name string,
likes array<string>,
address map<string,string>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
COLLECTION ITEMS TERMINATED BY '-'
MAP KEYS TERMINATED BY ':';

hive> desc formatted psn;
OK
# col_name            	data_type           	comment             
	 	 
id                  	int                 	                    
name                	string              	                    
likes               	array<string>       	                    
address             	map<string,string>  	                    
	 	 
# Detailed Table Information	 	 
Database:           	default             	 
Owner:              	root                	 
CreateTime:         	Sat Apr 25 22:15:57 CST 2020	 
LastAccessTime:     	UNKNOWN             	 
Protect Mode:       	None                	 
Retention:          	0                   	 
Location:           	hdfs://mycluster/user/hive/warehouse/psn	 
Table Type:         	MANAGED_TABLE       	 
Table Parameters:	 	 
	transient_lastDdlTime	1587824157          
	 	 
# Storage Information	 	 
SerDe Library:      	org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe	 
InputFormat:        	org.apache.hadoop.mapred.TextInputFormat	 
OutputFormat:       	org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat	 
Compressed:         	No                  	 
Num Buckets:        	-1                  	 
Bucket Columns:     	[]                  	 
Sort Columns:       	[]                  	 
Storage Desc Params:	 	 
	colelction.delim    	-                   
	field.delim         	,                   
	mapkey.delim        	:                   
	serialization.format	,                   
Time taken: 0.427 seconds, Fetched: 32 row(s)

```

使用系统的默认分隔 ^A  '\u001'

```
CREATE TABLE psn(
id int,
name string,
likes array<string>,
address map<string,string>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\001'
COLLECTION ITEMS TERMINATED BY '\002'
MAP KEYS TERMINATED BY '\003';
```



## 将文件加载到表中

```sql
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]
 
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)] [INPUTFORMAT 'inputformat' SERDE 'serde'] (3.0 or later)
```

data

```text
1,小明1,lol-book-movie,beijing:shangxuetang-shanghai:pudong
2,小明2,lol-book-movie,beijing:shangxuetang-shanghai:pudong
3,小明3,lol-book-movie,beijing:shangxuetang-shanghai:pudong
4,小明4,lol-book-movie,beijing:shangxuetang-shanghai:pudong
5,小明5,lol-movie,beijing:shangxuetang-shanghai:pudong
6,小明6,lol-movie,beijing:shangxuetang-shanghai:pudong
7,小明7,lol-movie,beijing:shangxuetang-shanghai:pudong
```

插入数据

```sh
hive> LOAD DATA LOCAL INPATH '/root/data/data' INTO TABLE psn;
Loading data to table default.psn
Table default.psn stats: [numFiles=1, totalSize=419]
OK
Time taken: 12.044 seconds
hive> select * from psn;
OK
1	小明1	["lol","book","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
2	小明2	["lol","book","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
3	小明3	["lol","book","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
4	小明4	["lol","book","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
5	小明5	["lol","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
6	小明6	["lol","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
7	小明7	["lol","movie"]	{"beijing":"shangxuetang","shanghai":"pudong"}
Time taken: 1.674 seconds, Fetched: 7 row(s)
```

#### 三种表

**内部表**

Table Type:         	MANAGED_TABLE 

内部表的创建方式和普通的创建方式是一样的，他是默认的存储的hdfs的路径也是默认的位置；

删除的时候会把源数据信息和hdfs文件存储的数据路径都一起删除。

**临时表**

```sql
CREATE TEMPORARY TABLE psn(
id int,
name string,
likes array<string>,
address map<string,string>
)
```

这张表只是存在当前的会话中，会话退出后，表就不存在了。

**外部表**

Table Type:         	EXTERNAT_TABLE 

需要上传的hdfs的数据

```sql
CREATE EXTERNAL TABLE psn(
id int,
name string,
likes array<string>,
address map<string,string>
)
LOCATION 'hdfs的存储路劲[如果指定文件，则是文件，如实文件夹，则是文件夹下面的所有文件]'
```

删除的时候。会把源数据信息删除，但是不会删除hdfs的数据信息。

### 文件格式问题

读时检查

如果发现文件导入进去的格式不正确，则会产生空值数据。

