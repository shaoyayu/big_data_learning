# [Hive DDL 中文文档](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)

总览

HiveQL DDL语句在此处记录，包括：

- 创建数据库/架构，表，视图，函数，索引
- 删除数据库/架构，表，视图，索引
- 截断表
- ALTER DATABASE / SCHEMA，表格，视图
- MSCK维修表（或更改表恢复分区）
- 显示数据库/架构，表，TBL属性，视图，分区，函数，索引，列，创建表
- DESCRIBE DATABASE / SCHEMA，表名，视图名，实体化视图名

除SHOW PARTITIONS外，PARTITION语句通常是TABLE语句的选项。

## 关键字，非保留关键字和保留关键字

|           | 所有关键词                                                   |                                                              |
| :-------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 版        | 非保留关键字                                                 | 保留关键字                                                   |
| 蜂巢1.2.0 | `ADD, ADMIN, AFTER, ANALYZE, ARCHIVE, ASC, BEFORE, BUCKET, BUCKETS, CASCADE, CHANGE, CLUSTER, CLUSTERED, CLUSTERSTATUS, COLLECTION, COLUMNS, COMMENT, COMPACT, COMPACTIONS, COMPUTE, CONCATENATE, CONTINUE, DATA, DATABASES, DATETIME, DAY, DBPROPERTIES, DEFERRED, DEFINED, DELIMITED, DEPENDENCY, DESC, DIRECTORIES, DIRECTORY, DISABLE, DISTRIBUTE, ELEM_TYPE, ENABLE, ESCAPED, EXCLUSIVE, EXPLAIN, EXPORT, FIELDS, FILE, FILEFORMAT, FIRST, FORMAT, FORMATTED, FUNCTIONS, HOLD_DDLTIME, HOUR, IDXPROPERTIES, IGNORE, INDEX, INDEXES, INPATH, INPUTDRIVER, INPUTFORMAT, ITEMS, JAR, KEYS, KEY_TYPE, LIMIT, LINES, LOAD, LOCATION, LOCK, LOCKS, LOGICAL, LONG, MAPJOIN, MATERIALIZED, METADATA, MINUS, MINUTE, MONTH, MSCK, NOSCAN, NO_DROP, OFFLINE, OPTION, OUTPUTDRIVER, OUTPUTFORMAT, OVERWRITE, OWNER, PARTITIONED, PARTITIONS, PLUS, PRETTY, PRINCIPALS, PROTECTION, PURGE, READ, READONLY, REBUILD, RECORDREADER, RECORDWRITER, REGEXP, RELOAD, RENAME, REPAIR, REPLACE, REPLICATION, RESTRICT, REWRITE, RLIKE, ROLE, ROLES, SCHEMA, SCHEMAS, SECOND, SEMI, SERDE, SERDEPROPERTIES, SERVER, SETS, SHARED, SHOW, SHOW_DATABASE, SKEWED, SORT, SORTED, SSL, STATISTICS, STORED, STREAMTABLE, STRING, STRUCT, TABLES, TBLPROPERTIES, TEMPORARY, TERMINATED, TINYINT, TOUCH, TRANSACTIONS, UNARCHIVE, UNDO, UNIONTYPE, UNLOCK, UNSET, UNSIGNED, URI, USE, UTC, UTCTIMESTAMP, VALUE_TYPE, VIEW, WHILE, YEAR` | `ALL, ALTER, AND, ARRAY, AS, AUTHORIZATION, BETWEEN, BIGINT, BINARY, BOOLEAN, BOTH, BY, CASE, CAST, CHAR, COLUMN, CONF, CREATE, CROSS, CUBE, CURRENT, CURRENT_DATE, CURRENT_TIMESTAMP, CURSOR, DATABASE, DATE, DECIMAL, DELETE, DESCRIBE, DISTINCT, DOUBLE, DROP, ELSE, END, EXCHANGE, EXISTS, EXTENDED, EXTERNAL, FALSE, FETCH, FLOAT, FOLLOWING, FOR, FROM, FULL, FUNCTION, GRANT, GROUP, GROUPING, HAVING, IF, IMPORT, IN, INNER, INSERT, INT, INTERSECT, INTERVAL, INTO, IS, JOIN, LATERAL, LEFT, LESS, LIKE, LOCAL, MACRO, MAP, MORE, NONE, NOT, NULL, OF, ON, OR, ORDER, OUT, OUTER, OVER, PARTIALSCAN, PARTITION, PERCENT, PRECEDING, PRESERVE, PROCEDURE, RANGE, READS, REDUCE, REVOKE, RIGHT, ROLLUP, ROW, ROWS, SELECT, SET, SMALLINT, TABLE, TABLESAMPLE, THEN, TIMESTAMP, TO, TRANSFORM, TRIGGER, TRUE, TRUNCATE, UNBOUNDED, UNION, UNIQUEJOIN, UPDATE, USER, USING, UTC_TMESTAMP, VALUES, VARCHAR, WHEN, WHERE, WINDOW, WITH` |
| 蜂巢2.0.0 | *删除：* `REGEXP, RLIKE`*添加：* `AUTOCOMMIT, ISOLATION, LEVEL, OFFSET, SNAPSHOT, `事务，工作，写 | *添加：* `COMMIT, ONLY, REGEXP, RLIKE, ROLLBACK, START`      |
| 蜂巢2.1.0 | *添加：* `ABORT, KEY, LAST, NORELY, NOVALIDATE, NULLS, RELY, VALIDATE` | *添加：* `CACHE, CONSTRAINT, FOREIGN, PRIMARY, REFERENCES`   |
| 蜂巢2.2.0 | *添加：* `DETAIL, DOW, EXPRESSION, OPERATOR, QUARTER, SUMMARY, VECTORIZATION, WEEK, YEARS, MONTHS, WEEKS, DAYS, HOURS, MINUTES, SECONDS` | *添加：* `DAYOFWEEK, EXTRACT, FLOOR, INTEGER, PRECISION, VIEWS` |
| 蜂巢3.0.0 | *添加：* `TIMESTAMPTZ, ZONE `                                | *添加：* `TIME, NUMERIC, SYNC`                               |

> 版本信息
>
> REGEXP和RLIKE是Hive 2.0.0之前的非保留关键字，并且是从Hive 2.0.0（[HIVE-11703](https://issues.apache.org/jira/browse/HIVE-11703)）开始的保留关键字。

## 创建/删除/更改/使用数据库

### 创建数据库

```
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name`` ``[COMMENT database_comment]`` ``[LOCATION hdfs_path]`` ``[MANAGEDLOCATION hdfs_path]`` ``[WITH DBPROPERTIES (property_name=property_value, ...)];
```

SCHEMA和DATABASE的用法是可互换的–它们含义相同。在Hive 0.6（[HIVE-675](https://issues.apache.org/jira/browse/HIVE-675)）中添加了CREATE DATABASE 。在Hive 0.7（[HIVE-1836](https://issues.apache.org/jira/browse/HIVE-1836)）中添加了WITH DBPROPERTIES子句。

MANAGEDLOCATION已添加到Hive 4.0.0（[HIVE-22995](https://issues.apache.org/jira/browse/HIVE-22995)）中的数据库中。LOCATION现在引用外部表的默认目录，而MANAGEDLOCATION引用托管表的默认目录。它建议将MANAGEDLOCATION放在metastore.warehouse.dir中，以便所有托管表都具有公共根，该根位于公共管理策略所在的位置。它可以与metastore.warehouse.tenant.colocation 一起使用， 以使其指向仓库根目录之外的目录，以具有基于租户的公共根目录，可以在其中设置配额和其他策略。 

### 删除数据库

```sql
DROP (DATABASE|SCHEMA) [IF EXISTS] database_name [RESTRICT|CASCADE];
```

SCHEMA和DATABASE的用法是可互换的–它们含义相同。在Hive 0.6（[HIVE-675](https://issues.apache.org/jira/browse/HIVE-675)）中添加了DROP DATABASE 。默认行为是RESTRICT，如果数据库不为空，则DROP DATABASE将失败。要将表也拖放到数据库中，请使用DROP DATABASE ... CASCADE。在Hive 0.8（[HIVE-2090](https://issues.apache.org/jira/browse/HIVE-2090)）中添加了对RESTRICT和CASCADE的支持。

### 修改数据库

```sql
ALTER (DATABASE|SCHEMA) database_name SET DBPROPERTIES (property_name=property_value, ...);   -- (Note: SCHEMA added in Hive 0.14.0)
 
ALTER (DATABASE|SCHEMA) database_name SET OWNER [USER|ROLE] user_or_role;   -- (Note: Hive 0.13.0 and later; SCHEMA added in Hive 0.14.0)
  
ALTER (DATABASE|SCHEMA) database_name SET LOCATION hdfs_path; -- (Note: Hive 2.2.1, 2.4.0 and later)
 
ALTER (DATABASE|SCHEMA) database_name SET MANAGEDLOCATION hdfs_path; -- (Note: Hive 4.0.0 and later)
```

SCHEMA和DATABASE的用法是可互换的–它们含义相同。在Hive 0.14（[HIVE-6601](https://issues.apache.org/jira/browse/HIVE-6601)）中添加了ALTER SCHEMA 。

ALTER DATABASE ... SET LOCATION语句不会将数据库当前目录的内容移动到新指定的位置。它不会更改与指定数据库下任何表/分区关联的位置。它仅更改默认的父目录，在该目录中将为此数据库添加新表。此行为类似于更改表目录不会将现有分区移动到其他位置。

ALTER DATABASE ... SET MANAGEDLOCATION语句不会将数据库的托管表目录的内容移动到新指定的位置。它不会更改与指定数据库下任何表/分区关联的位置。它仅更改默认的父目录，在该目录中将为此数据库添加新表。此行为类似于更改表目录不会将现有分区移动到其他位置。

关于数据库的其他元数据无法更改。 

### 使用数据库

```sql
USE database_name;``USE DEFAULT;
```

USE为所有后续的HiveQL语句设置当前数据库。要恢复为默认数据库，请使用关键字“ `default`”代替数据库名称。要检查当前正在使用哪个数据库：（从[Hive 0.13.0开始](https://issues.apache.org/jira/browse/HIVE-4144)）。`SELECT current_database()`

`USE database_name`是在Hive 0.6（[HIVE-675](https://issues.apache.org/jira/browse/HIVE-675)）中添加的。



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

- 表名和列名不区分大小写，但SerDe和属性名区分大小写。

  - 在Hive 0.12和更早版本中，表和列名称中仅允许使用字母数字和下划线字符。
  - 在蜂巢0.13后来，列名可以包含任何[的Unicode](http://en.wikipedia.org/wiki/List_of_Unicode_characters)字符（见[HIVE-6013](https://issues.apache.org/jira/browse/HIVE-6013)），然而，点（**。**和结肠癌）（ ：**）**产量错误的查询，所以它们在蜂房1.2.0不允许的（见[HIVE-10120](https://issues.apache.org/jira/browse/HIVE-10120)）。反引号（```）中指定的任何列名均按字面意义处理。在反引号字符串中，使用双反引号（````）表示反引号字符。反引号还可以将保留关键字用于表和列标识符。
  - 要恢复到0.13.0之前的行为并将列名称限制为字母数字和下划线字符，请将配置属性设置`hive.support.quoted.identifiers`为`none`。在此配置中，带反引号的名称被解释为正则表达式。有关详细信息，请参阅[列名称中支持带引号的标识符](https://issues.apache.org/jira/secure/attachment/12618321/QuotedIdentifier.html)。

- 表和列注释是字符串文字（单引号）。

- 没有[EXTERNAL子句](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-ExternalTables)创建的表称为*[托管表，](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-ManagedandExternalTables)*因为Hive管理其数据。要确定表是托管表还是外部表，请在[DESCRIBE EXTENDED table_name](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-DescribeTable/View/Column)的输出中查找tableType 。

- 

  TBLPROPERTIES子句允许您使用自己的元数据键/值对标记表定义。还存在一些预定义的表属性，例如由Hive自动添加和管理的last_modified_user和last_modified_time。其他预定义的表属性包括：

  - TBLPROPERTIES（“ comment” =“ *table_comment* ”）
  - TBLPROPERTIES（“ hbase.table.name” =“ *table_name* ”）–请参见[HBase集成](https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration#HBaseIntegration-Usage)。
  - 版本0.13.0+（[HIVE-6406](https://issues.apache.org/jira/browse/HIVE-6406)）中的TBLPROPERTIES（“ immutable” =“ true”）或（“ immutable” =“ false” ）–请参阅[从查询将数据插入到Hive表中](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-InsertingdataintoHiveTablesfromqueries)。
  - TBLPROPERTIES（“ orc.compress” =“ ZLIB”）或（“ orc.compress” =“ SNAPPY”）或（“ orc.compress” =“ NONE”）和其他ORC属性–请参见[ORC文件](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-HiveQLSyntax)。
  - TBLPROPERTIES（“ transactional” =“ true”）或（“ transactional” =“ false”）在版本0.14.0+中，默认值为“ false” –请参见[Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-TableProperties)。
  - TBLPROPERTIES（“ NO_AUTO_COMPACTION” =“ true”）或（“ NO_AUTO_COMPACTION” =“ false”），默认值为“ false” –请参阅[Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-TableProperties)。
  - TBLPROPERTIES（“ compactor.mapreduce.map.memory.mb” =“ *mapper_memory”*）–请参阅[Hive Transactions ](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-TableProperties)。 
  - TBLPROPERTIES（“ compactorthreshold.hive.compactor.delta.num.threshold” =“ *threshold_num* ”）– 请参阅[Hive Transactions ](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-TableProperties)。 
  - TBLPROPERTIES（“ compactorthreshold.hive.compactor.delta.pct.threshold” =“ *threshold_pct* ”）– 请参阅[Hive Transactions ](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-TableProperties)。 
  - 1.2.0+（[HIVE-9118](https://issues.apache.org/jira/browse/HIVE-9118)）版本中的TBLPROPERTIES（“ auto.purge” =“ true”）或（“ auto.purge” =“ false”）–请参见[LanguageManual DDL＃Drop表](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-DropTable)，[LanguageManual DDL＃Drop分区](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-DropPartitions)，[LanguageManual DDL＃Truncate Table](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-TruncateTable)，然后[插入Overwrite](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-InsertOverwrite)。
  - 版本0.6.0+（[HIVE-1329](https://issues.apache.org/jira/browse/HIVE-1329)）中的TBLPROPERTIES（“ EXTERNAL” =“ TRUE”）–将托管表更改为外部表，反之亦然，将“ FALSE”更改为外部表。
    - 从Hive 2.4.0（[HIVE-16324](https://issues.apache.org/jira/browse/HIVE-16324)）开始，属性'EXTERNAL'的值被解析为布尔值（不区分大小写的真或假），而不是区分大小写的字符串比较。
  - 如果在外部表上设置了4.0.0+版（[HIVE-19981](https://issues.apache.org/jira/browse/HIVE-19981)）中的TBLPROPERTIES（“ external.table.purge” =“ true”），也会删除该数据。

- 若要为表指定数据库，请在CREATE TABLE语句之前发出[USE database_name](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-UseDatabase)语句（在[Hive 0.6](https://issues.apache.org/jira/browse/HIVE-675)及更高版本中），或使用数据库名称限定表名（`database_name.table.name`在[Hive 0.7](https://issues.apache.org/jira/browse/HIVE-1517)及更高版本中使用“ ” ）。
  关键字“ `default`”可用于默认数据库。

有关表注释，表属性和SerDe属性的更多信息，请参见下面的[更改表](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterTable)。

有关原始和复杂数据类型的详细信息，请参见[类型系统](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-TypeSystem)和[配置单元](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types)数据类型。

#### 托管表和外部表

默认情况下，Hive创建托管表，其中文件，元数据和统计信息由内部Hive进程管理。有关托管表和外部表之间差异的详细信息，请参阅[托管表与外部表](https://cwiki.apache.org/confluence/display/Hive/Managed+vs.+External+Tables)。

#### 储存格式

Hive支持内置和自定义开发的文件格式。有关压缩表存储的详细信息，请参见[CompressedStorage ](https://cwiki.apache.org/confluence/display/Hive/CompressedStorage)。以下是Hive内置的一些格式：

 

| 储存格式                  | 描述                                                         |
| :------------------------ | :----------------------------------------------------------- |
| 储存格式                  | 描述                                                         |
| 储存者                    | 以非本地表格式存储。创建或链接到非本地表，例如由[HBase](https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration)或[Druid](https://cwiki.apache.org/confluence/display/Hive/Druid+Integration)或[Accumulo ](https://cwiki.apache.org/confluence/display/Hive/AccumuloIntegration)支持的表。 有关此选项的更多信息，请参见[StorageHandlers ](https://cwiki.apache.org/confluence/display/Hive/StorageHandlers)。 |
| INPUTFORMAT和OUTPUTFORMAT | 在file_format中，以将相应的InputFormat和OutputFormat类的名称指定为字符串文字。  例如，“ org.apache.hadoop.hive.contrib.fileformat.base64.Base64TextInputFormat”。  对于LZO压缩，要使用的值为 “ INPUTFORMAT“ com.hadoop.mapred.DeprecatedLzoTextInputFormat” OUTPUTFORMAT“ [org.apache.hadoop.hive.ql.io](http://org.apache.hadoop.hive.ql.io/) .HiveIgnoreKeyTextOutputFormat”  （请参见[LZO压缩](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LZO)）。 |
| 储存为PARQUET             | 在[Hive 0.13.0及更高版本中](https://cwiki.apache.org/confluence/display/Hive/Parquet#Parquet-Hive0.13andlater)以[Parquet ](https://cwiki.apache.org/confluence/display/Hive/Parquet)列存储格式存储为Parquet格式；使用行格式SERDE ...存储为INPUTFORMAT ... OUTPUTFORMAT语法...在[蜂巢0.10，0.11，0.12 ](https://cwiki.apache.org/confluence/display/Hive/Parquet#Parquet-Hive0.10-0.12)。 |
| 储存为AVRO                | 在[Hive 0.14.0和更高版本中](https://issues.apache.org/jira/browse/HIVE-6806)以Avro格式存储（请参阅[Avro SerDe ](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe)）。 |
| 储存为JSONFILE            | 在Hive 4.0.0及更高版本中以Json文件格式存储。                 |
| 储存为ORC                 | 存储为[ORC文件格式](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-HiveQLSyntax)。支持ACID交易和基于成本的优化器（CBO）。存储列级元数据。 |
| 存储为序列文件            | 存储为压缩的序列文件。                                       |
| 存储为文本文件            | 存储为纯文本文件。TEXTFILE是默认文件格式，除非配置参数[hive.default.fileformat ](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.default.fileformat)具有不同的设置。使用DELIMITED子句读取定界文件。通过使用'ESCAPED BY'子句（例如ESCAPED BY'\'）对定界字符启用转义。 如果要使用可以包含这些定界字符的数据，则需要转义。  也可以使用'NULL DEFINED AS'子句指定自定义NULL格式（默认为'\ N'）。 |
| 存储为RCFILE              | 存储为[记录列文件](https://en.wikipedia.org/wiki/RCFile)格式。 |

#### 行格式和SerDe

您可以使用自定义SerDe或本机SerDe创建表。如果未指定ROW FORMAT或指定ROW FORMAT DELIMITED，则使用本机SerDe。
使用SERDE子句使用自定义SerDe创建表。有关SerDes的更多信息，请参见：

- [蜂巢SerDe](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)
- [SerDe](https://cwiki.apache.org/confluence/display/Hive/SerDe)
- [HCatalog存储格式](https://cwiki.apache.org/confluence/display/Hive/HCatalog+StorageFormats)

您必须为使用本机SerDe的表指定列列表。有关允许的列类型，请参阅《用户指南》的“ [类型”](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types)部分。
可以指定使用自定义SerDe的表的列列表，但是Hive将查询SerDe以确定该表的实际列列表。

有关SerDes的常规信息，请参阅  《开发人员指南》中的[Hive SerDe ](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)。 有关输入和输出处理的详细信息，另请参见 [SerDe ](https://cwiki.apache.org/confluence/display/Hive/SerDe)。

要更改表的SerDe或SERDEPROPERTIES，请使用下面的[语言手册DDL＃Add SerDe Properties中](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AddSerDeProperties)所述的ALTER TABLE语句。

#### 分区表

可以使用PARTITIONED BY子句创建分区表。一个表可以具有一个或多个分区列，并为分区列中的每个不同值组合创建一个单独的数据目录。此外，可以使用CLUSTERED BY列对表或分区进行存储，并且可以通过SORT BY列在该存储区中对数据进行排序。这样可以提高某些查询的性能。

如果在创建分区表时收到以下错误消息：“ FAILED：语义分析错误：分区列中重复的列”，则意味着您正在尝试将分区列包含在表本身的数据中。您可能确实定义了该列。但是，您创建的分区将创建一个可查询的伪列，因此您必须将表列重命名为其他名称（用户不应在其上查询！）。

例如，假设原始未分区表具有三列：id，date和name。

```sql
id     int,
date   date,
name   varchar
```

现在您要按日期分区。您的Hive定义可以使用“ dtDontQuery”作为列名，以便可以将“ date”用于分区（和查询）。

```sql
create table table_name (
  id                int,
  dtDontQuery       string,
  name              string
)
partitioned by (date string)
```

现在，您的用户仍将查询“ where date = '...'”，但第二列dtDontQuery将保留原始值。

这是创建分区表的示例语句：

```sql
CREATE TABLE page_view(viewTime INT, userid BIGINT,
     page_url STRING, referrer_url STRING,
     ip STRING COMMENT 'IP Address of the User')
 COMMENT 'This is the page view table'
 PARTITIONED BY(dt STRING, country STRING)
 STORED AS SEQUENCEFILE;
```

上面的语句使用viewTime，userid，page_url，referrer_url和ip列（包括注释）创建page_view表。该表也被分区，数据存储在序列文件中。假定文件中的数据格式由ctrl-A字段分隔，由换行符行分隔。

```sql
CREATE TABLE page_view(viewTime INT, userid BIGINT,
     page_url STRING, referrer_url STRING,
     ip STRING COMMENT 'IP Address of the User')
 COMMENT 'This is the page view table'
 PARTITIONED BY(dt STRING, country STRING)
 ROW FORMAT DELIMITED
   FIELDS TERMINATED BY '\001'
STORED AS SEQUENCEFILE;
```

上面的语句使您可以创建与上一个表相同的表。

在前面的示例中，数据存储在<hive.metastore.warehouse.dir> / page_view中。`hive.metastore.warehouse.dir`在Hive配置文件hive-site.xml中为密钥指定一个值。

#### 外部表

EXTERNAL关键字使您可以创建表并提供LOCATION，以便Hive对此表不使用默认位置。如果您已经生成了数据，这将很方便。删除EXTERNAL表时，*不会*从文件系统中删除表中的数据。启动Hive 4.0.0（ [![img](https://issues.apache.org/jira/secure/viewavatar?size=xsmall&avatarId=21146&avatarType=issuetype)HIVE-19981](https://issues.apache.org/jira/browse/HIVE-19981) - 管理的表转换为外部表由HiveStrictManagedMigration工具应设置为删除数据时，表被删除 **解决** ）设置表属性external.table.purge = true，也会删除数据。

EXTERNAL表指向其存储的任何HDFS位置，而不是存储在configuration属性指定的文件夹中`hive.metastore.warehouse.dir`。

例如：

```sql
CREATE EXTERNAL TABLE page_view(viewTime INT, userid BIGINT,
     page_url STRING, referrer_url STRING,
     ip STRING COMMENT 'IP Address of the User',
     country STRING COMMENT 'country of origination')
 COMMENT 'This is the staging page view table'
 ROW FORMAT DELIMITED FIELDS TERMINATED BY '\054'
 STORED AS TEXTFILE
 LOCATION '<hdfs_location>';
```

您可以使用上面的语句创建一个page_view表，该表指向任何要存储的HDFS位置。但是您仍然必须确保按照上面的CREATE语句中指定的方式对数据进行定界。

有关创建外部表的另一个示例，请参见教程中的“ [加载数据](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-LoadingData) ”。

#### 选择创建表（CTAS）

还可以通过一个create-table-as-select（CTAS）语句中的查询结果来创建和填充表。CTAS创建的表是原子表，这意味着直到填充所有查询结果后，其他用户才能看到该表。因此，其他用户将看不到带有查询完整结果的表，或者根本看不到该表。

CTAS有两部分，SELECT部分可以是HiveQL支持的任何[SELECT语句](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select)。CTAS的CREATE部分从SELECT部分获取结果模式，并使用其他表属性（例如SerDe和存储格式）创建目标表。

从Hive 3.2.0开始，CTAS语句可以为目标表（[HIVE-20241](https://issues.apache.org/jira/browse/HIVE-20241)）定义分区规范。

CTAS具有以下限制：

- 目标表不能是外部表。
- 目标表不能是列表存储表。

```sql
CREATE TABLE new_key_value_store
   ROW FORMAT SERDE "org.apache.hadoop.hive.serde2.columnar.ColumnarSerDe"
   STORED AS RCFile
   AS
SELECT (key % 1024) new_key, concat(key, value) key_value_pair
FROM key_value_store
SORT BY new_key, key_value_pair;
```

上面的CTAS语句使用从SELECT语句的结果派生的架构（new_key DOUBLE，key_value_pair STRING）创建目标表new_key_value_store。如果SELECT语句未指定列别名，则列名称将自动分配给_col0，_col1和_col2等。此外，新目标表是使用特定的SerDe和独立于源表中存储表的存储格式创建的SELECT语句。

从[Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-1180)开始，SELECT语句可以包含一个或多个公用表表达式（CTE），如[SELECT语法](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select#LanguageManualSelect-SelectSyntax)所示。有关示例，请参见[Common Table Expression](https://cwiki.apache.org/confluence/display/Hive/Common+Table+Expression#CommonTableExpression-CTEinViews,CTAS,andInsertStatements)。

能够从一个表到另一个表选择数据是Hive最强大的功能之一。Hive在执行查询时处理数据从源格式到目标格式的转换。

#### 创建像表

LIKE形式的CREATE TABLE允许您精确地复制现有的表定义（而不复制其数据）。与CTAS相比，以下语句创建了一个新的empty_key_value_store表，该表的定义与表名以外的所有详细信息都与现有key_value_store完全匹配。新表不包含任何行。

```sql
CREATE TABLE empty_key_value_store
LIKE key_value_store [TBLPROPERTIES (property_name=property_value, ...)];
```

在Hive 0.8.0之前，CREATE TABLE LIKE view_name将复制该视图。在Hive 0.8.0和更高版本中，CREATE TABLE LIKE view_name通过采用view_name模式（字段和分区列）使用SerDe和文件格式的默认值来创建表。

#### 桶排序表

```sql
CREATE TABLE page_view(viewTime INT, userid BIGINT,
     page_url STRING, referrer_url STRING,
     ip STRING COMMENT 'IP Address of the User')
 COMMENT 'This is the page view table'
 PARTITIONED BY(dt STRING, country STRING)
 CLUSTERED BY(userid) SORTED BY(viewTime) INTO 32 BUCKETS
 ROW FORMAT DELIMITED
   FIELDS TERMINATED BY '\001'
   COLLECTION ITEMS TERMINATED BY '\002'
   MAP KEYS TERMINATED BY '\003'
 STORED AS SEQUENCEFILE;
```

在上面的示例中，page_view表是由userid存储（分类）的，在每个存储区中，数据按viewTime的升序排序。这样的组织允许用户对聚集列（在本例中为userid）进行高效采样。排序属性允许内部运算符在评估查询时利用众所周知的数据结构，从而提高效率。如果任何列是列表或地图，则可以使用MAP KEYS和COLLECTION ITEMS关键字。

CLUSTERED BY和SORTED BY创建命令不会影响将数据插入表的方式，而只会影响数据的读取方式。这意味着用户必须注意正确地插入数据，方法是将减速器的数量指定为等于存储桶的数量，并在查询中使用CLUSTER BY和SORT BY命令。

还有一个[创建和填充存储桶表](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL+BucketedTables)的示例。

#### 倾斜的桌子

>  版本信息
>
> 自Hive 0.10.0起（[HIVE-3072](https://issues.apache.org/jira/browse/HIVE-3072)和[HIVE-3649](https://issues.apache.org/jira/browse/HIVE-3649)）。有关在Hive 0.10.0和0.11.0中实现列表存储的其他JIRA票证，请参见[HIVE-3026](https://issues.apache.org/jira/browse/HIVE-3026)。

> 设计文件
>
> 有关更多信息，请阅读[偏斜联接优化](https://cwiki.apache.org/confluence/display/Hive/Skewed+Join+Optimization)和[列表桶](https://cwiki.apache.org/confluence/display/Hive/ListBucketing)设计文档。

此功能可用于提高一列或多列具有[偏斜](https://cwiki.apache.org/confluence/display/Hive/Skewed+Join+Optimization)值的表的性能。通过指定经常出现的值（严重偏斜），Hive会自动将它们拆分成单独的文件（或[列表存储的](https://cwiki.apache.org/confluence/display/Hive/ListBucketing)情况下为[目录](https://cwiki.apache.org/confluence/display/Hive/ListBucketing)），并在查询过程中考虑到这一事实，以便它可以跳过或包括整个文件（或如果可能，则在[列表存储的](https://cwiki.apache.org/confluence/display/Hive/ListBucketing)情况下访问[目录](https://cwiki.apache.org/confluence/display/Hive/ListBucketing)）。

可以在创建表的过程中在每个表级别上指定。

下面的示例显示具有三个偏斜值的一列，还可以选择带有STORED AS DIRECTORIES子句，该子句指定列表存储。

```sql
CREATE TABLE list_bucket_single (key STRING, value STRING)
  SKEWED BY (key) ON (1,5,6) [STORED AS DIRECTORIES];
```

这是带有两个倾斜列的表的示例。

```sql
CREATE TABLE list_bucket_multiple (col1 STRING, col2 int, col3 STRING)
  SKEWED BY (col1, col2) ON (('s1',1), ('s3',3), ('s13',13), ('s78',78)) [STORED AS DIRECTORIES];
```

有关相应的ALTER TABLE语句，请参见下面的[LanguageManual DDL＃Alter Table歪斜或存储为目录](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterTableSkewedorStoredasDirectories)。

#### 临时表

> 版本信息
>
> 从Hive 0.14.0（[HIVE-7090](https://issues.apache.org/jira/browse/HIVE-7090)）开始。

已创建为临时表的表仅对当前会话可见。数据将存储在用户的临时目录中，并在会话结束时删除。

如果使用数据库中已经存在的永久表的数据库/表名创建临时表，则在该会话内，对该表的任何引用都将解析为该临时表，而不是永久表。在不删除临时表或将其重命名为不冲突的名称的情况下，用户将无法访问该会话中的原始表。

临时表具有以下限制：

- 不支持分区列。
- 不支持创建索引。

小号在tarting [蜂巢1.1.0 ](https://issues.apache.org/jira/browse/HIVE-7313)牛逼，他临时表存储策略可以设置为`memory`， `ssd`或 `default`与[hive.exec.temporary.table.storage](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.exec.temporary.table.storage)配置参数（见 [HDFS存储类型和存储策略](http://hadoop.apache.org/docs/r2.6.0/hadoop-project-dist/hadoop-hdfs/ArchivalStorage.html#Storage_Types_and_Storage_Policies)）。

例：

```sql
CREATE TEMPORARY TABLE list_bucket_multiple (col1 STRING, col2 int, col3 STRING);
```

#### 交易表

> 版本信息
>
> 从Hive 4.0（[HIVE-18453](https://issues.apache.org/jira/browse/HIVE-18453)）开始。

支持使用ACID语义进行操作的表。请参阅[此](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)有关事务表的更多细节。

```sql
CREATE TRANSACTIONAL TABLE transactional_table_test(key string, value string) PARTITIONED BY(ds string) STORED AS ORC;
```

#### 约束条件

> 版本信息
>
> 从Hive 2.1.0（[HIVE-13290](https://issues.apache.org/jira/browse/HIVE-13290)）开始。

Hive包括对未验证的主键和外键约束的支持。存在约束时，某些SQL工具会生成更有效的查询。由于未验证这些约束，因此上游系统需要在将数据加载到Hive中之前确保数据完整性。

```sql
create table pk(id1 integer, id2 integer,
  primary key(id1, id2) disable novalidate);
 
create table fk(id1 integer, id2 integer,
  constraint c1 foreign key(id1, id2) references pk(id2, id1) disable novalidate);
```

> 版本信息
>
> 从Hive 3.0.0开始（[HIVE-16575](https://issues.apache.org/jira/browse/HIVE-16575)， [HIVE-18726](https://issues.apache.org/jira/browse/HIVE-18726)， [HIVE-18953](https://issues.apache.org/jira/browse/HIVE-18953)）。

Hive包括对UNIQUE，NOT NULL，DEFAULT和CHECK约束的支持。除了UNIQUE之外，还强制执行所有三种类型的约束。

```sql
create table constraints1(id1 integer UNIQUE disable novalidate, id2 integer NOT NULL,
  usr string DEFAULT current_user(), price double CHECK (price > 0 AND price <= 1000));
 
create table constraints2(id1 integer, id2 integer,
  constraint c1_unique UNIQUE(id1) disable novalidate);
 
create table constraints3(id1 integer, id2 integer,
  constraint c1_check CHECK(id1 + id2 > 0));
```

不支持对复杂数据类型（例如map，struct，array）使用DEFAULT。

### 删除表

```sql
DROP TABLE [IF EXISTS] table_name [PURGE];     -- (Note: PURGE available in Hive 0.14.0 and later)
```

DROP TABLE删除该表的元数据和数据。如果已配置垃圾桶（并且未指定PURGE），则数据实际上将移动到.Trash / Current目录。元数据完全丢失。

删除EXTERNAL表时，将*不会*从文件系统中删除表中的数据。启动Hive 4.0.0（ [![img](https://issues.apache.org/jira/secure/viewavatar?size=xsmall&avatarId=21146&avatarType=issuetype)HIVE-19981](https://issues.apache.org/jira/browse/HIVE-19981) - 管理的表转换为外部表由HiveStrictManagedMigration工具应设置为删除数据时，表被删除 **解决** ）设置表属性external.table.purge = true，也会删除数据。

删除视图引用的表时，不会发出警告（视图被悬吊为无效，必须由用户删除或重新创建）。

否则，将表信息从元存储中删除，并且将原始数据删除，就像通过“ hadoop dfs -rm”一样。在许多情况下，这导致表数据被移动到用户主目录中的.Trash文件夹中。因此，错误地删除表的用户可以通过以下方式恢复丢失的数据：重新创建具有相同架构的表，重新创建任何必要的分区，然后使用Hadoop手动将数据移回原位。该解决方案依赖于基础实施，因此可能会随着时间的推移或在整个安装过程中发生变化。强烈建议用户不要随意删除表。

> 版本信息：PURGE
>
> [HIVE-7100](https://issues.apache.org/jira/browse/HIVE-7100)在版本0.14.0中添加了PURGE选项。

```sql
TRUNCATE [TABLE] table_name [PARTITION partition_spec];
 
partition_spec:
  : (partition_column = partition_col_value, partition_column = partition_col_value, ...)
```

从表或分区中删除所有行。如果启用了文件系统“ [废纸will”](https://issues.apache.org/jira/browse/HIVE-14626)，则这些[行将被废纸](https://issues.apache.org/jira/browse/HIVE-14626)，否则将被删除（自Hive 2.2.0和[HIVE-14626起](https://issues.apache.org/jira/browse/HIVE-14626)）。当前目标表应该是本机/托管表，否则将引发异常。用户可以指定partial partition_spec来一次截断多个分区，而省略partition_spec将截断表中的所有分区。

从HIVE 2.3.0（[HIVE-15880](https://issues.apache.org/jira/browse/HIVE-15880)）开始，如果表属性“ auto.purge”（请参见上面的[TBLPROPERTIES ](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-listTableProperties)）设置为“ true”，则对它发出TRUNCATE TABLE命令时，表的数据不会移到“已删除邮件”并且在TRUNCATE错误的情况下无法检索。这仅适用于托管表（请参阅[托管表](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-ManagedandExternalTables)）。如果未为托管表设置“ auto.purge”属性或将其设置为false，则可以关闭此行为。

从Hive 4.0（[HIVE-23183](https://issues.apache.org/jira/browse/HIVE-23183)）开始，TABLE令牌是可选的，以前的版本需要它。

## 更改表/分区/列

Alter table语句使您可以更改现有表的结构。您可以添加列/分区，更改SerDe，添加表和SerDe属性或重命名表本身。同样，alter table partition语句允许您更改命名表中特定分区的属性。

### 修改表

#### 重命名表

```sql
ALTER TABLE table_name RENAME TO new_table_name;
```

此语句使您可以将表的名称更改为其他名称。

从0.6版开始，[托管表](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-managedTable)上的重命名将移动其HDFS位置。重命名已从版本2.2.0（[HIVE-14909](https://issues.apache.org/jira/browse/HIVE-14909)）更改，以便仅在创建表时没有[LOCATION子句](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-CreateTable)且在其数据库目录下移动了托管表的HDFS位置。0.6之前的Hive版本只是在不移动HDFS位置的情况下重命名了元存储中的表。

#### 更改表属性

```sql
ALTER TABLE table_name SET TBLPROPERTIES table_properties;
 
table_properties:
  : (property_name = property_value, property_name = property_value, ... )
```

您可以使用此语句将自己的元数据添加到表中。当前，last_modified_user和last_modified_time属性由Hive自动添加和管理。用户可以将自己的属性添加到此列表。您可以执行DESCRIBE EXTENDED TABLE以获取此信息。

有关更多信息，请参见上面的“创建表”中的[TBLPROPERTIES子句](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-listTableProperties)。

```sql
ALTER TABLE table_name SET TBLPROPERTIES ('comment' = new_comment);
```

#### 添加SerDe属性

```sql
ALTER TABLE table_name [PARTITION partition_spec] SET SERDE serde_class_name [WITH SERDEPROPERTIES serde_properties];
 
ALTER TABLE table_name [PARTITION partition_spec] SET SERDEPROPERTIES serde_properties;
 
serde_properties:
  : (property_name = property_value, property_name = property_value, ... )
```

这些语句使您能够更改表的SerDe或将用户定义的元数据添加到表的SerDe对象。

当Hive初始化表以序列化和反序列化数据时，SerDe属性将传递到表的SerDe。因此，用户可以在此处存储其自定义SerDe所需的任何信息。有关更多信息，请参考《[SerDe文档》](https://cwiki.apache.org/confluence/display/Hive/SerDe)和《[Hive SerDe](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)》，有关在CREATE TABLE语句中设置表的SerDe和SERDEPROPERTIES的详细信息，请参见上述[LanguageManual DDL＃Row格式，存储格式和SerDe](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-RowFormat,StorageFormat,andSerDe)。

请注意这两个`property_name`和`property_value`必须加引号。

例：

```sql
ALTER TABLE table_name SET SERDEPROPERTIES ('field.delim' = ',');
```

#### 删除SerDe属性

> 版本信息
>
> 从Hive 4.0.0（[HIVE-21952](https://issues.apache.org/jira/browse/HIVE-18842)）开始支持Remove SerDe Properties 。

```sql
ALTER TABLE table_name [PARTITION partition_spec] UNSET SERDEPROPERTIES (property_name, ... );
```

这些语句使您可以将用户定义的元数据删除到表的SerDe对象。

注意  property_name 必须加引号。

例：

```sql
ALTER TABLE table_name UNSET SERDEPROPERTIES ('field.delim');
```

#### 更改表存储属性

```sql
ALTER TABLE table_name CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name, ...)]
  INTO num_buckets BUCKETS;
```

这些语句更改表的物理存储属性。

注意：这些命令只会修改Hive的元数据，*不会*重组或重新格式化现有数据。用户应确保实际数据布局符合元数据定义。

#### 倾斜或存储为目录的变更表

> 版本信息
>
> 自Hive 0.10.0起（[HIVE-3072](https://issues.apache.org/jira/browse/HIVE-3072)和[HIVE-3649](https://issues.apache.org/jira/browse/HIVE-3649)）。有关在Hive 0.10.0和0.11.0中实现列表存储的其他JIRA票证，请参见[HIVE-3026](https://issues.apache.org/jira/browse/HIVE-3026)。

可以使用ALTER TABLE语句更改表的SKEWED和STORED AS DIRECTORIES选项。有关相应的CREATE TABLE语法，请参见上面的[LanguageManual DDL＃Skewed Tables](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-SkewedTables)。

##### 修改表倾斜

```sql
ALTER TABLE table_name SKEWED BY (col_name1, col_name2, ...)
  ON ([(col_name1_value, col_name2_value, ...) [, (col_name1_value, col_name2_value), ...]
  [STORED AS DIRECTORIES];
```

STORED AS DIRECTORIES选项确定[倾斜的](https://cwiki.apache.org/confluence/display/Hive/Skewed+Join+Optimization)表是否使用[列表存储](https://cwiki.apache.org/confluence/display/Hive/ListBucketing)功能，该功能为倾斜的值创建子目录。

##### 更改表未存储为目录

```sql
ALTER TABLE table_name NOT STORED AS DIRECTORIES;
```

这将更改列表存储的位置图。

#### 更改表约束

> 版本信息
>
> 从Hive版本[2.1.0开始](https://issues.apache.org/jira/browse/HIVE-13290)。

可以通过ALTER TABLE语句添加或删除表约束。

```sql
ALTER TABLE table_name ADD CONSTRAINT constraint_name PRIMARY KEY (column, ...) DISABLE NOVALIDATE;
ALTER TABLE table_name ADD CONSTRAINT constraint_name FOREIGN KEY (column, ...) REFERENCES table_name(column, ...) DISABLE NOVALIDATE RELY;
ALTER TABLE table_name ADD CONSTRAINT constraint_name UNIQUE (column, ...) DISABLE NOVALIDATE;
ALTER TABLE table_name CHANGE COLUMN column_name column_name data_type CONSTRAINT constraint_name NOT NULL ENABLE;
ALTER TABLE table_name CHANGE COLUMN column_name column_name data_type CONSTRAINT constraint_name DEFAULT default_value ENABLE;
ALTER TABLE table_name CHANGE COLUMN column_name column_name data_type CONSTRAINT constraint_name CHECK check_expression ENABLE;
 
ALTER TABLE table_name DROP CONSTRAINT constraint_name;
```

#### 其他Alter Table语句

有关[更改表](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterEitherTableorPartition)的更多DDL语句，请参见下面的[LanguageManual DDL＃Alter表或分区](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterEitherTableorPartition)。

### 旧分区

可以通过使用ALTER TABLE语句中的PARTITION子句来添加，重命名，交换（移动），删除或（未存档）分区，如下所述。要使metastore知道直接添加到HDFS的分区，可以使用metastore check命令（[MSCK ](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-RecoverPartitions(MSCKREPAIRTABLE))），或者在Amazon EMR上可以使用ALTER TABLE的RECOVER PARTITIONS选项。有关[更改分区](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterEitherTableorPartition)的更多方法，请参见下面的[LanguageManual DDL＃Alter表或](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterEitherTableorPartition)分区。

> 版本1.2+
>
> 从Hive 1.2（[HIVE-10307 ](https://issues.apache.org/jira/browse/HIVE-10307)）开始，如果将属性[hive.typecheck.on.insert ](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.typecheck.on.insert)设置为true（默认值），则会对分区规范中指定的分区值进行类型检查，转换和规范化以符合其列类型。值可以是数字文字。

#### **添加分区**

```sql
ALTER TABLE table_name ADD [IF NOT EXISTS] PARTITION partition_spec [LOCATION 'location'][, PARTITION partition_spec [LOCATION 'location'], ...];
 
partition_spec:
  : (partition_column = partition_col_value, partition_column = partition_col_value, ...)
```

您可以使用ALTER TABLE ADD PARTITION将分区添加到表中。分区值仅在为字符串时才应加引号。该位置必须是数据文件所在的目录。（ADD PARTITION会更改表元数据，但不会加载数据。如果分区位置中不存在数据，查询将不会返回任何结果。）如果表的partition_spec已经存在，则会引发错误。您可以使用IF NOT EXISTS跳过该错误。

> 版本0.7
>
> 尽管在单个ALTER TABLE中具有多个partition_spec是正确的语法，但是如果在版本0.7中执行此操作，则分区方案将失败。也就是说，每个指定分区的查询将始终仅使用第一个分区。

具体而言，以下示例将在Hive 0.7中静默失败且没有错误，并且无论您指定哪个分区，所有查询都只会进入dt ='2008-08-08'分区。

**例：**

```sql
ALTER TABLE page_view ADD PARTITION (dt='2008-08-08', country='us') location '/path/to/us/part080808'
                          PARTITION (dt='2008-08-09', country='us') location '/path/to/us/part080809';
```

在Hive 0.8和更高版本中，您可以在单个ALTER TABLE语句中添加多个分区，如上例所示。

在Hive 0.7中，如果要添加许多分区，应使用以下形式：

```sql
ALTER TABLE table_name ADD PARTITION (partCol = 'value1') location 'loc1';
ALTER TABLE table_name ADD PARTITION (partCol = 'value2') location 'loc2';
...
ALTER TABLE table_name ADD PARTITION (partCol = 'valueN') location 'locN';
```

##### **动态分区**

> 版本信息
>
> 从Hive 0.9开始。

```sql
ALTER TABLE table_name PARTITION partition_spec RENAME TO PARTITION partition_spec;
```

该语句使您可以更改分区列的值。用例之一是，您可以使用此语句来规范您的旧分区列值，使其符合其类型。在这种情况下，即使将属性[hive.typecheck.on.insert](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.typecheck.on.insert)设置为true（默认值）也无法为旧*partition_spec中*的列值启用类型转换和规范化，这允许您以旧字符串形式指定任何旧数据。*partition_spec*。

#### 交换分区

可以在表之间交换（移动）分区。

> 版本信息
>
> 从Hive 0.12开始（[HIVE-4095](https://issues.apache.org/jira/browse/HIVE-4095)）。在蜂巢版本支持多分区 [1.2.2 ， 1.3.0，和2.0.0+](https://issues.apache.org/jira/browse/HIVE-11745)。

```sql
-- Move partition from table_name_1 to table_name_2
ALTER TABLE table_name_2 EXCHANGE PARTITION (partition_spec) WITH TABLE table_name_1;
-- multiple partitions
ALTER TABLE table_name_2 EXCHANGE PARTITION (partition_spec, partition_spec2, ...) WITH TABLE table_name_1;
```

该语句使您可以将一个分区中的数据从一个表移动到另一个具有相同架构并且还没有该分区的表。
有关此功能的更多详细信息，请参阅[Exchange分区](https://cwiki.apache.org/confluence/display/Hive/Exchange+Partition)和[HIVE-4095 ](https://issues.apache.org/jira/browse/HIVE-4095)。

#### 发现分区

在Hive Metastore中自动发现并同步分区的元数据。 

创建外部分区表后，将自动添加“ discover.partitions” =“ true”表属性。对于托管分区表，可以手动添加“ discover.partitions”表属性。当Hive Metastore Service（HMS）在远程服务模式下启动时，后台线程（PartitionManagementTask）每300秒（通过metastore.partition.management.task.frequency配置可配置）定期进行调度，以查找带有“ discover.partitions”表的表属性设置为true，并在同步模式下执行msck修复。如果该表是事务性表，则在执行msck修复之前，将获得该表的“排他锁”。使用此表属性，不再需要手动运行“ MSCK REPAIR TABLE table_name SYNC PARTITIONS”。 

> 版本信息
>
> 从Hive 4.0.0（[HIVE-20707](https://issues.apache.org/jira/browse/HIVE-20707)）开始。

#### 恢复分区（MSCK修复表）

Hive将每个表的分区列表存储在其元存储中。但是，如果将新分区直接添加到HDFS（例如通过使用`hadoop fs -put`命令）或从HDFS中删除，则除非用户`ALTER TABLE table_name ADD/DROP PARTITION`在每个新添加的分区上运行命令，否则元存储（因此Hive）将不会意识到分区信息的这些更改。或分别删除的分区。

但是，用户可以使用带有修复表选项的metastore check命令运行：

```sql
MSCK [REPAIR] TABLE table_name [ADD/DROP/SYNC PARTITIONS];
```

它将有关分区的元数据更新到Hive元存储中，以获取尚不存在此类元数据的分区。MSC命令的默认选项是“添加分区”。使用此选项，它将把HDFS上存在但元存储中不存在的所有分区添加到元存储中。DROP PARTITIONS选项将从已经从HDFS中删除的metastore中删除分区信息。SYNC PARTITIONS选项等效于调用ADD和DROP PARTITIONS。有关更多详细信息，请参见[HIVE-874](https://issues.apache.org/jira/browse/HIVE-874)和[HIVE-17824](https://issues.apache.org/jira/browse/HIVE-17824)。如果存在大量未跟踪的分区，则可以批量运行MSCK REPAIR TABLE，以避免OOME（内存不足错误）。通过为属性提供配置的批次大小**hive.msck.repair.batch.size**可以在内部批量运行。该属性的默认值为零，这意味着它将立即执行所有分区。不带REPAIR选项的MSCK命令可用于查找有关元数据不匹配元存储的详细信息。

Amazon Elastic MapReduce（EMR）版本的Hive上的等效命令为：

```sql
ALTER TABLE table_name RECOVER PARTITIONS;
```

从Hive 1.3开始，如果在HDFS上找到分区值中包含禁止字符的目录，则MSCK将引发异常。在客户端上使用hive.msck.path.validation设置可以更改此行为；“跳过”将仅跳过目录。无论如何，“ ignore”将尝试创建分区（旧行为）。这可能会或可能不会。

#### 删除分区

```sql
ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec[, PARTITION partition_spec, ...]
  [IGNORE PROTECTION] [PURGE];            -- (Note: PURGE available in Hive 1.2.0 and later, IGNORE PROTECTION not available 2.0.0 and later)
```

您可以使用ALTER TABLE DROP PARTITION删除表的分区。这将删除该分区的数据和元数据。如果配置了垃圾桶，则实际上将数据移动到.Trash / Current目录，除非指定了PURGE，但元数据完全丢失（请参见上面的[LanguageManual DDL＃Drop表](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-DropTable)）。

> 版本信息：保护
>
> IGNORE PROTECTION在2.0.0及更高版本中不再可用。通过使用Hive可用的几个安全选项之一来代替此功能（请参阅[基于SQL Standard的Hive授权](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization)）。有关详细信息，请参见[HIVE-11145](https://issues.apache.org/jira/browse/HIVE-11145)。

对于受[NO_DROP CASCADE](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterTable/PartitionProtections)保护的表，可以使用谓词IGNORE PROTECTION删除指定的分区或一组分区（例如，在两个Hadoop群集之间拆分表时）：

```sql
ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec IGNORE PROTECTION;
```

无论保护状态如何，上述命令都会删除该分区。

> 版本信息：PURGE
>
> [HIVE-10934](https://issues.apache.org/jira/browse/HIVE-10934)将PURGE选项添加到1.2.1版的ALTER TABLE中。



如果指定了PURGE，则分区数据不会进入.Trash / Current目录，因此在DROP错误的情况下无法检索该分区数据：

```sql
ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec PURGE;   -- (Note: Hive 1.2.0 and later)
```

还可以使用表属性auto.purge 指定清除选项（请参见上面的[TBLPROPERTIES](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-listTableProperties)）。

在Hive 0.7.0或更高版本中，如果不存在分区，则DROP返回错误，除非指定了IF EXISTS或将配置变量[hive.exec.drop.ignorenonexistent ](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.exec.drop.ignorenonexistent)设置为true。

```
ALTER TABLE page_view DROP PARTITION (dt=``'2008-08-08'``, country=``'us'``);
```

#### （A）成绩档案

```
ALTER TABLE table_name ARCHIVE PARTITION partition_spec;``ALTER TABLE table_name UNARCHIVE PARTITION partition_spec;
```

归档是一项功能，可将分区的文件移动到Hadoop存档（HAR）中。请注意，只有文件数会减少；HAR不提供任何压缩。有关更多信息，请参见[语言手动存档](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Archiving)。



### 更改表或分区

#### 更改表/分区文件格式

```sql
ALTER TABLE table_name [PARTITION partition_spec] SET FILEFORMAT file_format;
```

该语句更改表（或分区）的文件格式。有关可用的file_format选项，请参见上面有关[CREATE TABLE](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-CreateTable)的部分。该操作仅更改表元数据。现有数据的任何转换都必须在Hive之外进行。

#### 更改表/分区位置

```sql
ALTER TABLE table_name [PARTITION partition_spec] SET LOCATION ``"new location"``;
```

#### 更改表/分区触摸

```sql
ALTER TABLE table_name TOUCH [PARTITION partition_spec];
```

TOUCH读取元数据，然后将其写回。这具有导致执行前/执行后钩子触发的效果。一个示例用例是，如果您有一个钩子记录所有修改过的表/分区，以及一个外部脚本，该脚本可以直接更改HDFS上的文件。由于脚本修改了配置单元之外的文件，因此该钩子不会记录修改。外部脚本可以调用TOUCH触发钩子，并将所述表或分区标记为已修改。

另外，如果我们加入可靠的上次修改时间，以后可能会很有用。然后触摸也会更新该时间。

请注意，TOUCH不会创建表或分区（如果尚不存在）。（请参阅[LanguageManual DDL＃Create表](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-CreateTable)。）

#### 更改表/分区保护

> 版本信息
>
> 从Hive 0.7.0起（[HIVE-1413](https://issues.apache.org/jira/browse/HIVE-1413)）。在HIVE 0.8.0（[HIVE-2605](https://issues.apache.org/jira/browse/HIVE-2605)）中添加了NO_DROP的CASCADE子句。
>
> 此功能已在Hive 2.0.0中删除。通过使用Hive可用的几个安全选项之一来代替此功能（请参阅[基于SQL Standard的Hive授权](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization)）。有关详细信息，请参见[HIVE-11145 ](https://issues.apache.org/jira/browse/HIVE-11145)。

```sql
ALTER TABLE table_name [PARTITION (partition_key = 'partition_value' [, ...])]
  COMPACT 'compaction_type'[AND WAIT]
  [WITH OVERWRITE TBLPROPERTIES ("property"="value" [, ...])];
```

通常，在使用[Hive事务](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)时，您不需要请求压缩，因为系统会检测到对它们的需求并启动压缩。但是，如果关闭了表的压缩，或者您想在系统无法选择的时间压缩表，则ALTER TABLE可以启动压缩。默认情况下，该语句将排队请求压缩并返回。要查看压缩的进度，请使用[SHOW COMPACTIONS](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-ShowCompactions)。从Hive [2.2.0开始](https://issues.apache.org/jira/browse/HIVE-15920)，可以指定“ AND WAIT”以包含操作块，直到压缩完成为止。

compaction_type可以是MAJOR或MINOR。有关更多信息，请参见[Hive Transactions中](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-BasicDesign)的“基本设计”部分。

#### 更改表/分区并置

> 版本信息
>
> 在Hive版本[0.8.0中，](https://issues.apache.org/jira/browse/HIVE-1950) RCFile添加了对使用串联命令对小型RCFile进行快速块级合并的支持。在Hive版本[0.14.0中，](https://issues.apache.org/jira/browse/HIVE-7509)添加的ORC文件支持使用串联命令对小型ORC文件进行快速条带级别合并。

```sql
ALTER TABLE table_name [PARTITION (partition_key = 'partition_value' [, ...])] CONCATENATE;
```

如果表或分区包含许多小的RCFiles或ORC文件，则上述命令会将它们合并为更大的文件。对于RCFile，合并发生在块级别，而对于ORC文件，合并发生在条带级别，从而避免了对数据进行解压缩和解码的开销。

#### 更改表/分区更新列

> 版本信息
>
> 在Hive版本[3.0.0中](https://issues.apache.org/jira/browse/HIVE-15995)，添加了此命令，以使用户将Serde存储的架构信息同步到metastore。

```sql
ALTER TABLE table_name [PARTITION (partition_key = 'partition_value' [, ...])] UPDATE COLUMNS;
```

带有自我描述表模式的Serdes的表在现实中可能与存储在Hive Metastore中的模式不同。例如，当用户使用架构URL或架构文字创建Avro存储的表时，该架构将被插入HMS，然后无论在URL或Serde中文字如何更改，该架构都不会在HMS中更改。特别是在与其他Apache组件集成时，这可能会导致问题。

更新列功能为用户提供了一种方法，使在SERDE中进行的任何模式更改都可以同步到HMS。它适用于表和分区级别，并且显然仅适用于HMS未跟踪其架构的表（请参见metastore.serdes.using.metastore.for.schema）。在这些后面的Serde类型上使用命令将导致错误。

### 更改列

#### 列名规则

列名不区分大小写。

> 版本信息
>
> 在Hive版本0.12.0和更早版本中，列名称只能包含字母数字和下划线字符。
>
> 在蜂巢释放0.13.0及更高版本，默认情况下，列名可以反引号内指定的（```）和含有任何[的Unicode](http://en.wikipedia.org/wiki/List_of_Unicode_characters)字符（[HIVE-6013](https://issues.apache.org/jira/browse/HIVE-6013)），然而，点（**。**和结肠癌）（ ：**）**产量错误的查询。在以反引号分隔的字符串中，所有字符均按字面意义处理，但双反引号（````）表示一个反引号字符。可以通过将设置`hive.support.quoted.identifiers`为来使用0.13.0之前的行为`none`，在这种情况下，反引号会将其解释为正则表达式。有关详细信息，请参见[列名称中的带引号的标识符](https://issues.apache.org/jira/secure/attachment/12618321/QuotedIdentifier.html)。
>
> 反引号可以使列名和表名使用保留关键字。

#### 更改列名称/类型/位置/注释

```sql
ALTER TABLE table_name [PARTITION partition_spec] CHANGE [COLUMN] col_old_name col_new_name column_type`` ``[COMMENT col_comment] [FIRST|AFTER column_name] [CASCADE|RESTRICT];
```

此命令将允许用户更改列的名称，[数据类型](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types)，注释或位置，或它们的任意组合。在[Hive 0.14.0](https://issues.apache.org/jira/browse/HIVE-7971)和更高版本中可以使用PARTITION子句；有关用法，请参见[升级Pre-Hive 0.13.0小数列](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-UpgradingPre-Hive0.13.0DecimalColumns)。还提供了针对Hive 0.13的补丁（请参阅[HIVE-7971](https://issues.apache.org/jira/browse/HIVE-7971)）。

[Hive 1.1.0中](https://issues.apache.org/jira/browse/HIVE-8839)提供了CASCADE | RESTRICT子句。使用CASCADE命令更改表更改列可更改表元数据的列，并将相同的更改级联到所有分区元数据。RESTRICT是默认设置，仅将列更改限制为表元数据。

> !!! 无论表或分区的[保护模式](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterTable/PartitionProtections)如何，ALTER TABLE CHANGE COLUMN CASCADE子句都将覆盖表分区的列元数据。谨慎使用。

> ! column change命令将仅修改Hive的元数据，而不会修改数据。用户应确保表/分区的实际数据布局符合元数据定义。

例：

```sql
CREATE TABLE test_change (a int, b int, c int);
 
// First change column a's name to a1.
ALTER TABLE test_change CHANGE a a1 INT;
 
// Next change column a1's name to a2, its data type to string, and put it after column b.
ALTER TABLE test_change CHANGE a1 a2 STRING AFTER b;
// The new table's structure is:  b int, a2 string, c int.
  
// Then change column c's name to c1, and put it as the first column.
ALTER TABLE test_change CHANGE c c1 INT FIRST;
// The new table's structure is:  c1 int, b int, a2 string.
  
// Add a comment to column a1
ALTER TABLE test_change CHANGE a1 a1 INT COMMENT 'this is column a1';
```

#### 添加/替换列

```sql
ALTER TABLE table_name 
  [PARTITION partition_spec]                 -- (Note: Hive 0.14.0 and later)
  ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...)
  [CASCADE|RESTRICT]                         -- (Note: Hive 1.1.0 and later)
```

使用ADD COLUMNS，您可以将新列添加到现有列的末尾但在分区列之前。Avro支持的表，[Hive 0.14](https://issues.apache.org/jira/browse/HIVE-7446)和更高版本也支持此功能。

替换列将删除所有现有列，并添加新的列集。这仅适用于具有本机SerDe（DynamicSerDe，MetadataTypedColumnsetSerDe，LazySimpleSerDe和ColumnarSerDe）的表。有关更多信息，请参考[Hive SerDe](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide#DeveloperGuide-HiveSerDe)。替换列也可以用于删除列。例如，“ `ALTER TABLE test_change REPLACE COLUMNS (a int, b int);`”将从test_change的架构中删除列“ c”。

在[Hive 0.14.0 ](https://issues.apache.org/jira/browse/HIVE-7971)和更高版本中可以使用PARTITION子句；有关用法，请参见[升级Pre-Hive 0.13.0小数列](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-UpgradingPre-Hive0.13.0DecimalColumns)。

[Hive 1.1.0中](https://issues.apache.org/jira/browse/HIVE-8839)提供了CASCADE | RESTRICT子句。使用CASCADE命令使用ALTER TABLE ADD | REPLACE COLUMNS更改表的元数据的列，并将相同的更改级联到所有分区元数据。RESTRICT是默认设置，仅将列更改限制为表元数据。

> !!!无论表或分区的[保护模式](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterTable/PartitionProtections)如何，ALTER TABLE ADD或REPLACE COLUMNS CASCADE都会覆盖表分区的列元数据。谨慎使用。

> column change命令将仅修改Hive的元数据，而不会修改数据。用户应确保表/分区的实际数据布局符合元数据定义。

#### 部分分区规范

从Hive 0.14（[HIVE-8411](https://issues.apache.org/jira/browse/HIVE-8411)）开始，用户可以为上述某些alter column语句提供部分分区规范，类似于动态分区。因此，不必为每个需要更改的分区发出一个alter column语句：

```sql
ALTER TABLE foo PARTITION (ds='2008-04-08', hr=11) CHANGE COLUMN dec_column_name dec_column_name DECIMAL(38,18);
ALTER TABLE foo PARTITION (ds='2008-04-08', hr=12) CHANGE COLUMN dec_column_name dec_column_name DECIMAL(38,18);
...
```

...您可以使用带有部分分区规范的单个ALTER语句一次更改许多现有分区：

```sql
// hive.exec.dynamic.partition needs to be set to true to enable dynamic partitioning with ALTER PARTITION
SET hive.exec.dynamic.partition = true;
  
// This will alter all existing partitions in the table with ds='2008-04-08' -- be sure you know what you are doing!
ALTER TABLE foo PARTITION (ds='2008-04-08', hr) CHANGE COLUMN dec_column_name dec_column_name DECIMAL(38,18);
 
// This will alter all existing partitions in the table -- be sure you know what you are doing!
ALTER TABLE foo PARTITION (ds, hr) CHANGE COLUMN dec_column_name dec_column_name DECIMAL(38,18);
```

与动态分区类似，必须将[hive.exec.dynamic.partition](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.exec.dynamic.partition)设置为true才能在ALTER PARTITION期间使用部分分区规范。以下操作支持此功能：

- 变更栏
- 添加栏
- 更换栏
- 文件格式
- Serde属性

## 创建/拖放/更改视图

### 建立视图

```sql
CREATE VIEW [IF NOT EXISTS] [db_name.]view_name [(column_name [COMMENT column_comment], ...) ]
  [COMMENT view_comment]
  [TBLPROPERTIES (property_name = property_value, ...)]
  AS SELECT ...;
```

CREATE VIEW创建具有给定名称的视图。如果已经存在具有相同名称的表或视图，则会引发错误。您可以使用IF NOT EXISTS跳过该错误。

如果没有提供列名，则视图的列名将自动从定义的SELECT表达式派生。（如果SELECT包含非别名的标量表达式，例如x + y，则将以_C0，_C1等形式生成结果视图列名称。）重命名列时，也可以选择提供列注释。（注释不会自动从基础列继承。）

如果视图的定义SELECT表达式无效，则CREATE VIEW语句将失败。

请注意，视图是一个纯逻辑对象，没有关联的存储。当查询引用视图时，将评估视图的定义，以生成一组行以供查询进一步处理。（这是一个概念性描述；实际上，作为查询优化的一部分，Hive可以将视图的定义与查询的结合起来，例如将查询中的过滤器向下推到视图中。）

创建视图时，将冻结视图的架构；对基础表的后续更改（例如，添加列）将不会反映在视图的架构中。如果基础表以不兼容的方式被删除或更改，则随后查询无效视图的尝试将失败。

视图是只读的，不能用作LOAD / INSERT / ALTER的目标。有关更改元数据的信息，请参见[ALTER VIEW](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterViewProperties)。

一个视图可能包含ORDER BY和LIMIT子句。如果参照查询还包含这些条款，查询级别子句进行评估**后**视图条款（和之后在查询的任何其它操作）。例如，如果视图指定LIMIT 5，并且引用查询执行为（从v LIMIT 10中选择*），那么最多将返回5行。

从[Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-1180)开始 ，视图的select语句可以包含一个或多个通用表表达式（CTE），如[SELECT语法](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select#LanguageManualSelect-SelectSyntax)所示 。有关CREATE VIEW语句中CTE的示例，请参见 [公共表表达式](https://cwiki.apache.org/confluence/display/Hive/Common+Table+Expression#CommonTableExpression-CTEinViews,CTAS,andInsertStatements)。

例：

```sql

CREATE VIEW onion_referrers(url COMMENT 'URL of Referring page')
  COMMENT 'Referrers to The Onion website'
  AS
  SELECT DISTINCT referrer_url
  FROM page_view
  WHERE page_url='http://www.theonion.com';
```

使用[SHOW CREATE TABLE](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-ShowCreateTable)显示创建视图的CREATE VIEW语句。从Hive 2.2.0开始，[SHOW VIEWS](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-ShowViews)显示数据库中的视图列表。

> 版本信息
>
> 最初，视图的文件格式被硬编码为SequenceFile。Hive 2.1.0（[HIVE-13736](https://issues.apache.org/jira/browse/HIVE-13736)）使视图使用[hive.default.fileformat](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.default.fileformat)和[hive.default.fileformat.managed](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.default.fileformat.managed)属性遵循与表和索引相同的默认值。 [ ](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.default.fileformat) 

### 放下视图

```sql
DROP VIEW [IF EXISTS] [db_name.]view_name;
```

DROP VIEW删除指定视图的元数据。（在视图上使用DROP TABLE是非法的。）

删除由其他视图引用的视图时，不会发出警告（从属视图悬空为无效，必须由用户删除或重新创建）。

在Hive 0.7.0或更高版本中，如果不存在该视图，则DROP返回错误，除非指定了IF EXISTS或将配置变量[hive.exec.drop.ignorenonexistent](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.exec.drop.ignorenonexistent)设置为true。

例：

```sql
DROP VIEW onion_referrers;
```

### 更改视图属性

```sql
ALTER VIEW [db_name.]view_name SET TBLPROPERTIES table_properties;
 
table_properties:
  : (property_name = property_value, property_name = property_value, ...)
```

与ALTER TABLE一样，您可以使用此语句将自己的元数据添加到视图中。

### 将视图更改为选择

> 版本信息
>
> 从[Hive 0.11开始](https://issues.apache.org/jira/browse/HIVE-3834)。

```sql
ALTER VIEW [db_name.]view_name AS select_statement;
```

将视图另存为选择会更改视图的定义，该视图必须存在。语法与CREATE VIEW相似，其效果与CREATE或REPLACE VIEW相同。

注意：该视图必须已经存在，并且如果该视图具有分区，则无法将其替换为“选择时更改视图”。

## 创建/拖放/更改实体化视图

> 版本信息
>
> 物化视图支持仅在Hive 3.0及更高版本中可用。

本节介绍了Hive的物化视图语法。有关Hive中的物化视图支持和用法的更多信息，请参见[此处](https://cwiki.apache.org/confluence/display/Hive/Materialized+views)。

### 创建实例化视图

```sql
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [db_name.]materialized_view_name
  [DISABLE REWRITE]
  [COMMENT materialized_view_comment]
  [PARTITIONED ON (col_name, ...)]
  [CLUSTERED ON (col_name, ...) | DISTRIBUTED ON (col_name, ...) SORTED ON (col_name, ...)]
  [
    [ROW FORMAT row_format]
    [STORED AS file_format]
      | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]
AS SELECT ...;
```

CREATE MATERIALIZED VIEW创建具有给定名称的视图。如果已经存在具有相同名称的表，视图或实例化视图，则会引发错误。您可以使用IF NOT EXISTS跳过该错误。

实例化视图列的名称将自动从定义的SELECT表达式中派生。

如果视图的定义SELECT表达式无效，则CREATE MATERIALIZED VIEW语句将失败。

默认情况下，实例化视图已启用，以供查询优化器在创建视图时自动重写。

> 版本信息
>
> 从Hive 3.2.0（[HIVE-14493](https://issues.apache.org/jira/browse/HIVE-14493)）开始支持PARTITIONED ON 。

> 版本信息
>
> 从Hive 4.0.0（[HIVE-18842](https://issues.apache.org/jira/browse/HIVE-18842)）开始支持CLUSTERED / DISTRIBUTED / SORTED ON 。

### 删除物化视图

```sql
DROP MATERIALIZED VIEW [db_name.]materialized_view_name;
```

DROP MATERIALIZED VIEW删除此实例化视图的元数据和数据。

### 更改物化视图

创建实例化视图后，优化器将能够利用其定义语义来使用实例化视图自动重写传入的查询，从而加快查询的执行速度。 

用户可以有选择地启用/禁用实例化视图以进行重写。回想一下，默认情况下，实例化视图已启用，可在创建时进行重写。若要更改该行为，可以使用以下语句：

```sql
ALTER MATERIALIZED VIEW [db_name.]materialized_view_name ENABLE|DISABLE REWRITE;
```

## 创建/删除/更改索引

> 从Hive 0.7开始。
>
> 从3.0开始删除索引！请参阅 [索引设计文档](https://cwiki.apache.org/confluence/display/Hive/IndexDev)

本节简要介绍了Hive索引，在这里对其进行了更全面的介绍：

- [Hive索引概述](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Indexing)
- [索引设计文件](https://cwiki.apache.org/confluence/display/Hive/IndexDev)

在Hive 0.12.0和更早版本中，对于CREATE INDEX和DROP INDEX语句，索引名称区分大小写。但是，ALTER INDEX需要使用小写字母创建的索引名称（请参阅 [HIVE-2752](https://issues.apache.org/jira/browse/HIVE-2752)）。通过使所有HiveQL语句的索引名称不区分大小写，此错误已在[Hive 0.13.0中](https://issues.apache.org/jira/browse/HIVE-2752)修复。对于0.13.0之前的版本，最佳实践是对所有索引名称使用小写字母。

### 创建索引

```sql
CREATE INDEX index_name
  ON TABLE base_table_name (col_name, ...)
  AS index_type
  [WITH DEFERRED REBUILD]
  [IDXPROPERTIES (property_name=property_value, ...)]
  [IN TABLE index_table_name]
  [
     [ ROW FORMAT ...] STORED AS ...
     | STORED BY ...
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (...)]
  [COMMENT "index comment"];
```

CREATE INDEX使用给定的列列表作为键在表上创建索引。请参阅[索引](https://cwiki.apache.org/confluence/display/Hive/IndexDev#IndexDev-CREATEINDEX)设计文档中的CREATE INDEX 。

### 删除索引

```sql
DROP INDEX [IF EXISTS] index_name ON table_name;
```

DROP INDEX删除索引，以及删除索引表。

在Hive 0.7.0或更高版本中，如果不存在索引，则DROP返回错误，除非指定了IF EXISTS或将配置变量[hive.exec.drop.ignorenonexistent](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.exec.drop.ignorenonexistent) 设置为true。

### 变更索引

```sql
ALTER INDEX index_name ON table_name [PARTITION partition_spec] REBUILD;
```

ALTER INDEX ... REBUILD生成使用WITH WITH DEFERRED REBUILD子句创建的索引，或重建先前生成的索引。如果指定了PARTITION，则仅重建该分区

## 创建/删除宏

版本信息

> 版本信息
>
> 从[Hive 0.12.0开始。](https://issues.apache.org/jira/browse/HIVE-2655)
>
> Bug修复：
>
> - 在[Hive 1.3.0和2.0.0](https://issues.apache.org/jira/browse/HIVE-11432)之前，当处理同一行时多次使用HiveQL宏时，即使参数不同，Hive也会为所有调用返回相同的结果。（请参阅[HIVE-11432](https://issues.apache.org/jira/browse/HIVE-11432)。）
> - 在[Hive 1.3.0和2.0.0 ](https://issues.apache.org/jira/browse/HIVE-12277)之前，如果在处理同一行时使用了多个宏，则ORDER BY子句可能会给出错误的结果。（请参阅[HIVE-12277](https://issues.apache.org/jira/browse/HIVE-12277)。） 
> - 在[Hive 2.1.0 ](https://issues.apache.org/jira/browse/HIVE-13372)之前，如果在处理同一行时使用了多个宏，则后一个宏的结果将被第一个宏的结果覆盖。（请参阅[HIVE-13372](https://issues.apache.org/jira/browse/HIVE-13372)。）

Hive 0.12.0在HiveQL中引入了宏，在此之前只能在Java中创建宏。

### 创建临时宏

```sql
CREATE TEMPORARY MACRO macro_name([col_name col_type, ...]) expression;
```

CREATE TEMPORARY MACRO使用给定的可选列列表作为表达式的输入来创建宏。在当前会话的持续时间内存在宏。

**例子：**

```sql
CREATE TEMPORARY MACRO fixed_number() ``42``;``CREATE TEMPORARY MACRO string_len_plus_two(x string) length(x) + ``2``;``CREATE TEMPORARY MACRO simple_add (x ``int``, y ``int``) x + y;
```

### 删除临时宏

```sql
DROP TEMPORARY MACRO [IF EXISTS] macro_name;
```

如果该函数不存在，则DROP TEMPORARY MACRO会返回错误，除非指定了IF EXISTS。

## 创建/删除/重新加载功能

### 临时功能

#### 创建临时功能

```sql
CREATE TEMPORARY FUNCTION function_name AS class_name;
```

该语句使您可以创建由class_name实现的函数。只要会话持续，就可以在Hive查询中使用此功能。您可以使用Hive的类路径中的任何类。您可以通过执行“ ADD JAR”语句将jar添加到类路径。请参阅CLI部分[Hive Interactive Shell命令](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveInteractiveShellCommands)（包括[Hive资源](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveResources)）以获取有关如何从Hive类路径添加/删除文件的更多信息。使用此功能，您可以注册用户定义功能（UDF）。

另请参阅[Hive插件](https://cwiki.apache.org/confluence/display/Hive/HivePlugins)以获取有关创建自定义UDF的常规信息。

#### 掉落临时功能

您可以按以下方式注销UDF：

```sql
DROP TEMPORARY FUNCTION [IF EXISTS] function_name;
```

在Hive 0.7.0或更高版本中，如果该函数不存在，则DROP返回错误，除非指定了IF EXISTS或将配置变量[hive.exec.drop.ignorenonexistent](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.exec.drop.ignorenonexistent) 设置为true。

### 永久函数

在Hive 0.13或更高版本中，可以将函数注册到元存储，因此可以在查询中引用它们，而不必在每个会话中都创建临时函数。

#### 创建函数

> 版本信息
>
> 从Hive 0.13.0（[HIVE-6047](https://issues.apache.org/jira/browse/HIVE-6047)）开始。

```sql
CREATE FUNCTION [db_name.]function_name AS class_name
  [USING JAR|FILE|ARCHIVE 'file_uri' [, JAR|FILE|ARCHIVE 'file_uri'] ];
```

该语句使您可以创建由class_name实现的函数。可以使用USING子句指定需要添加到环境中的jar，文件或档案；当Hive会话首次引用该功能时，这些资源将被添加到环境中，就像发出了[ADD JAR / FILE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli#LanguageManualCli-HiveResources)一样。如果Hive不在本地模式下，则资源位置必须是非本地URI，例如HDFS位置。

该功能将被添加到指定的数据库，或者在创建功能时添加到当前数据库。可以通过完全限定函数名称（db_name.function_name）来引用该函数，或者如果该函数位于当前数据库中，则可以不带限定地对其进行引用。

#### 删除函数

> 版本信息
>
> 从Hive 0.13.0（[HIVE-6047](https://issues.apache.org/jira/browse/HIVE-6047)）开始。

```sql
DROP FUNCTION [IF EXISTS] function_name;
```

如果该函数不存在，则DROP返回错误，除非指定了IF EXISTS或将配置变量[hive.exec.drop.ignorenonexistent](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.exec.drop.ignorenonexistent) 设置为true。

#### 重新加载函数

> 版本信息
>
> 从Hive 1.2.0（[HIVE-2573](https://issues.apache.org/jira/browse/HIVE-2573)）开始。

```sql
RELOAD (FUNCTIONS|FUNCTION);
```

从[HIVE-2573](https://issues.apache.org/jira/browse/HIVE-2573)开始，如果在一个Hive CLI会话中创建永久功能是在创建功能之前启动的，则它们可能不会反映在HiveServer2或其他Hive CLI会话中。在HiveServer2或HiveCLI会话中发出RELOAD FUNCTIONS将允许它拾取对永久功能的任何更改，而这些更改可能是由其他HiveCLI会话完成的。由于向后兼容的原因，RELOAD FUNCTION; 也被接受。



## 创建/删除/授予/撤消角色和权限

[Hive不建议使用的授权模式/旧模式](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173) 具有有关以下DDL语句的信息：

- [创建角色](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173#Hivedeprecatedauthorizationmode/LegacyMode-Create/DropRole)
- [授予角色](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173#Hivedeprecatedauthorizationmode/LegacyMode-Grant/RevokeRoles)
- [撤销角色](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173#Hivedeprecatedauthorizationmode/LegacyMode-Grant/RevokeRoles)
- [授予权限](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173#Hivedeprecatedauthorizationmode/LegacyMode-Grant/RevokePrivileges)
- [撤消privilege_type](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173#Hivedeprecatedauthorizationmode/LegacyMode-Grant/RevokePrivileges)
- [删除角色](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173#Hivedeprecatedauthorizationmode/LegacyMode-Create/DropRole)
- [显示角色授予](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173#Hivedeprecatedauthorizationmode/LegacyMode-ViewingGrantedRoles)
- [显示授权](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173#Hivedeprecatedauthorizationmode/LegacyMode-ViewingGrantedPrivileges)

有关Hive 0.13.0和更高版本中[基于SQL标准的授权](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization)，请参阅以下DDL语句：

- 角色管理命令
  - [创建角色](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-CreateRole)
  - [授予角色](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-GrantRole)
  - [撤销角色](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-RevokeRole)
  - [删除角色](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-DropRole)
  - [显示角色](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-ShowRoles)
  - [显示角色授予](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-ShowRoleGrant)
  - [显示当前角色](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-ShowCurrentRoles)
  - [设定角色](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-SetRole)
  - [显示原理](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-ShowPrincipals)
- 对象权限命令
  - [授予权限](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-Grant)
  - [撤消privilege_type](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-Revoke)
  - [显示授权](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-ShowGrant)

## 显示

这些语句提供了一种查询Hive元存储库的方法，以获取该Hive系统可访问的现有数据和元数据。

### 显示数据库

```sql
SHOW (DATABASES|SCHEMAS) [LIKE 'identifier_with_wildcards'];
```

SHOW DATABASES或SHOW SCHEMAS列出了元存储中定义的所有数据库。SCHEMAS和DATABASES的用法是可互换的–它们意味着同一件事。

可选的LIKE子句允许使用正则表达式过滤数据库列表。对于任何字符，正则表达式中的通配符只能是“ *”或“ |” 供选择。示例为'employees'，'emp *'，'emp * | * ees'，所有这些都将匹配名为'employees'的数据库。

> 版本信息：SHOW DATABASES
>
> 从4.0.0版本开始，我们仅接受SQL类型，例如表达式，其中任何字符都包含'％'，而单个字符包含'_'。示例为'employees'，'emp％'，'emplo_ees'，所有这些都将匹配名为'employees'的数据库。

### 显示表/视图/材料化视图/分区/索引

#### 显示表格

```sql
SHOW TABLES [IN database_name] ['identifier_with_wildcards'];
```

`SHOW TABLES`列出当前数据库（或使用`IN`子句明确命名的数据库）中的所有基本表和视图，其名称与可选的正则表达式匹配。对于任何字符，正则表达式中的通配符只能是“ *”或“ |” 供选择。示例为“ page_view”，“ page_v *”，“ * view | page *”，所有这些都将与“ page_view”表匹配。匹配表按字母顺序列出。如果在metastore中没有找到匹配的表，这不是错误。如果未给出正则表达式，则会列出所选数据库中的所有表。

#### 显示视图

> 版本信息
>
> 通过[HIVE-14558](https://issues.apache.org/jira/browse/HIVE-14558)在Hive 2.2.0中引入。

```sql
SHOW VIEWS [IN/FROM database_name] [LIKE 'pattern_with_wildcards'];
```

SHOW VIEWS列出当前数据库中的所有视图（或使用INor FROM子句明确命名的视图），其名称与可选的正则表达式匹配。对于任何字符，正则表达式中的通配符只能是“ *”或“ |” 供选择。示例为“ page_view”，“ page_v *”，“ * view | page *”，所有这些都将与“ page_view”视图匹配。匹配的视图按字母顺序列出。如果在metastore中找不到匹配的视图，这不是错误。如果未给出正则表达式，则会列出所选数据库中的所有视图。

例子

```sql
SHOW VIEWS;                                -- show all views in the current database
SHOW VIEWS 'test_*';                       -- show all views that start with "test_"
SHOW VIEWS '*view2';                       -- show all views that end in "view2"
SHOW VIEWS LIKE 'test_view1|test_view2';   -- show views named either "test_view1" or "test_view2"
SHOW VIEWS FROM test1;                     -- show views from database test1
SHOW VIEWS IN test1;                       -- show views from database test1 (FROM and IN are same)
SHOW VIEWS IN test1 "test_*";              -- show views from database test2 that start with "test_"
```

#### 显示实例化视图

```sql
SHOW MATERIALIZED VIEWS [IN/FROM database_name] [LIKE 'pattern_with_wildcards’];
```

`SHOW MATERIALIZED VIEWS`列出当前数据库中的所有视图（或使用`IN`or `FROM`子句明确命名的视图），其名称与可选的正则表达式匹配。它还显示有关实例化视图的其他信息，例如，是否启用重写以及实例化视图的刷新模式。对于任何字符，正则表达式中的通配符只能是“ *”或“ |” 供选择。如果未给出正则表达式，则会列出所选数据库中的所有实例化视图。

#### 显示分区

```sql
SHOW PARTITIONS table_name;
```

SHOW PARTITIONS列出了给定基表的所有现有分区。分区按字母顺序列出。

> 版本信息
>
> 从Hive 0.6开始，SHOW PARTITIONS可以过滤分区列表，如下所示。

还可以指定分区规范的各个部分以过滤结果列表。

例子：

```sql
SHOW PARTITIONS table_name PARTITION(ds='2010-03-03');            -- (Note: Hive 0.6 and later)
SHOW PARTITIONS table_name PARTITION(hr='12');                    -- (Note: Hive 0.6 and later)
SHOW PARTITIONS table_name PARTITION(ds='2010-03-03', hr='12');   -- (Note: Hive 0.6 and later)
```

> 版本信息
>
> 从Hive 0.13.0开始，SHOW PARTITIONS可以指定数据库（[HIVE-5912](https://issues.apache.org/jira/browse/HIVE-5912)）。

```sql
SHOW PARTITIONS [db_name.]table_name [PARTITION(partition_spec)];   -- (Note: Hive 0.13.0 and later)
```

例：

```sql
SHOW PARTITIONS databaseFoo.tableBar PARTITION(ds='2010-03-03', hr='12');   -- (Note: Hive 0.13.0 and later)
```

> 版本信息
>
> 从Hive 4.0.0开始，SHOW PARTITIONS可以选择使用WHERE / ORDER BY / LIMIT子句来过滤/排序/限制结果列表（[HIVE-22458](https://issues.apache.org/jira/browse/HIVE-22458)）。这些子句的工作方式与SELECT语句中的工作方式类似。 

```sql
SHOW PARTITIONS [db_name.]table_name [PARTITION(partition_spec)] [WHERE where_condition] [ORDER BY col_list] [LIMIT rows];   -- (Note: Hive 4.0.0 and later)
```

例：

```sql
SHOW PARTITIONS databaseFoo.tableBar LIMIT 10;                                                               -- (Note: Hive 4.0.0 and later)
SHOW PARTITIONS databaseFoo.tableBar PARTITION(ds='2010-03-03') LIMIT 10;                                    -- (Note: Hive 4.0.0 and later)
SHOW PARTITIONS databaseFoo.tableBar PARTITION(ds='2010-03-03') ORDER BY hr DESC LIMIT 10;                   -- (Note: Hive 4.0.0 and later)
SHOW PARTITIONS databaseFoo.tableBar PARTITION(ds='2010-03-03') WHERE hr >= 10 ORDER BY hr DESC LIMIT 10;    -- (Note: Hive 4.0.0 and later)
SHOW PARTITIONS databaseFoo.tableBar WHERE hr >= 10 AND ds='2010-03-03' ORDER BY hr DESC LIMIT 10;           -- (Note: Hive 4.0.0 and later)
```

注意：请使用*hr> = 10*  而不是*hr-10> = 0*  来过滤结果，因为Metastore不会将后者谓词下推到基础存储中。

#### 显示表/分区扩展

```sql
SHOW TABLE EXTENDED [IN|FROM database_name] LIKE 'identifier_with_wildcards' [PARTITION(partition_spec)];
```

SHOW TABLE EXTENDED将列出与给定正则表达式匹配的所有表的信息。如果存在分区规范，则用户不能将正则表达式用作表名。该命令的输出包括基本表信息和文件系统信息，例如totalNumberFiles，totalFileSize，maxFileSize，minFileSize，lastAccessTime和lastUpdateTime。如果存在分区，它将输出给定分区的文件系统信息，而不是表的文件系统信息。

例

```shell
hive> show table extended like part_table;
OK
tableName:part_table
owner:thejas
location:file:/tmp/warehouse/part_table
inputformat:org.apache.hadoop.mapred.TextInputFormat
outputformat:org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat
columns:struct columns { i32 i}
partitioned:true
partitionColumns:struct partition_columns { string d}
totalNumberFiles:1
totalFileSize:2
maxFileSize:2
minFileSize:2
lastAccessTime:0
lastUpdateTime:1459382233000
```

#### 显示表属性

> 版本信息
>
> 从Hive 0.10.0开始。

```sql
SHOW TBLPROPERTIES tblname;
SHOW TBLPROPERTIES tblname("foo");
```

第一种形式列出了所讨论表的所有表属性，每行一个，由制表符分隔。该命令的第二种形式仅输出所要属性的值。

有关更多信息，请参见上面的“创建表”中的[TBLPROPERTIES子句](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-listTableProperties)。

#### 显示创建表

> 版本信息
>
> 从[Hive 0.10开始](https://issues.apache.org/jira/browse/HIVE-967)。

```sql
SHOW CREATE TABLE ([db_name.]table_name|view_name);
```

SHOW CREATE TABLE显示创建给定表的CREATE TABLE语句，或创建给定视图的CREATE VIEW语句。

#### 显示索引

> 版本信息
>
> 从Hive 0.7开始。
>
> 从3.0开始删除索引！请参阅 [索引设计文档](https://cwiki.apache.org/confluence/display/Hive/IndexDev)

```sql
SHOW [FORMATTED] (INDEX|INDEXES) ON table_with_index [(FROM|IN) db_name];
```

SHOW INDEXES显示特定列上的所有索引以及有关它们的信息：索引名，表名，用作键的列名，索引表名，索引类型和注释。如果使用了FORMATTED关键字，那么将为每列打印列标题。

### 显示列

>  版本信息
>
> 从[Hive 0.10开始](https://issues.apache.org/jira/browse/HIVE-2909)。

```sql
SHOW COLUMNS (FROM|IN) table_name [(FROM|IN) db_name];
```

SHOW COLUMNS显示表中的所有列，包括分区列。

> 版本信息
>
> ```
> 显示列（FROM | IN）table_name [（FROM | IN）db_name]  [ LIKE 'pattern_with_wildcards'];
> ```
>
> 由[HIVE-18373](https://issues.apache.org/jira/browse/HIVE-18373)在Hive 3.0中添加。

SHOW COLUMNS 列出表中所有名称与可选正则表达式匹配的列。对于任何字符，正则表达式中的通配符只能是“ *”或“ |” 供选择。示例为“ cola”，“ col *”，“ * a | col *”，所有这些都与“ cola”列匹配。匹配列按字母顺序列出。如果在表中找不到匹配的列，这不是错误。如果未提供正则表达式，则会列出所选表中的所有列。

例子

```sql
-- SHOW COLUMNS
CREATE DATABASE test_db;
USE test_db;
CREATE TABLE foo(col1 INT, col2 INT, col3 INT, cola INT, colb INT, colc INT, a INT, b INT, c INT);
  
-- SHOW COLUMNS basic syntax
SHOW COLUMNS FROM foo;                            -- show all column in foo
SHOW COLUMNS FROM foo "*";                        -- show all column in foo
SHOW COLUMNS IN foo "col*";                       -- show columns in foo starting with "col"                 OUTPUT col1,col2,col3,cola,colb,colc
SHOW COLUMNS FROM foo '*c';                       -- show columns in foo ending with "c"                     OUTPUT c,colc
SHOW COLUMNS FROM foo LIKE "col1|cola";           -- show columns in foo either col1 or cola                 OUTPUT col1,cola
SHOW COLUMNS FROM foo FROM test_db LIKE 'col*';   -- show columns in foo starting with "col"                 OUTPUT col1,col2,col3,cola,colb,colc
SHOW COLUMNS IN foo IN test_db LIKE 'col*';       -- show columns in foo starting with "col" (FROM/IN same)  OUTPUT col1,col2,col3,cola,colb,colc
  
-- Non existing column pattern resulting in no match
SHOW COLUMNS IN foo "nomatch*";
SHOW COLUMNS IN foo "col+";                       -- + wildcard not supported
SHOW COLUMNS IN foo "nomatch";
```

### 显示函数

```sql
SHOW FUNCTIONS [LIKE "<pattern>"];
```

SHOW FUNCTIONS列出了所有用户定义和内置的函数，如果用LIKE指定，则用正则表达式过滤。

### 显示授予的角色和特权

[Hive不建议使用的授权模式/旧式模式](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173)包含有关以下SHOW语句的信息：

- [显示角色授予](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173#Hivedeprecatedauthorizationmode/LegacyMode-ViewingGrantedRoles)
- [显示授权](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=45876173#Hivedeprecatedauthorizationmode/LegacyMode-ViewingGrantedPrivileges)

在Hive 0.13.0和更高版本中，[基于SQL标准的授权](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization)具有以下SHOW语句：

- [显示角色授予](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-ShowRoleGrant)
- [显示授权](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-ShowGrant)
- [显示当前角色](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-ShowCurrentRoles)
- [显示角色](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-ShowRoles)
- [显示原理](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization#SQLStandardBasedHiveAuthorization-ShowPrincipals)

### 显示锁

```sql
SHOW LOCKS <table_name>;
SHOW LOCKS <table_name> EXTENDED;
SHOW LOCKS <table_name> PARTITION (<partition_spec>);
SHOW LOCKS <table_name> PARTITION (<partition_spec>) EXTENDED;
SHOW LOCKS (DATABASE|SCHEMA) database_name;     
-- (Note: Hive 0.13.0 and later; SCHEMA added in Hive 0.14.0)
```

SHOW LOCKS显示表或分区上的锁。有关锁的信息，请参见[Hive并发模型](https://cwiki.apache.org/confluence/display/Hive/Locking)。

Hive 0.13（对于DATABASE（请参阅[HIVE-2093](https://issues.apache.org/jira/browse/HIVE-2093)）和Hive 0.14（对于SCHEMA）（请参见[HIVE-6601](https://issues.apache.org/jira/browse/HIVE-6601)））支持SHOW LOCK（DATABASE | SCHEMA ）。SCHEMA和DATABASE是可互换的–它们含义相同。

使用[Hive事务时](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)，SHOW LOCKS返回此信息（请参阅[HIVE-6460](https://issues.apache.org/jira/browse/HIVE-6460)）：

- 数据库名称
- 表名
- 分区名称（如果表已分区）
- 锁所处的状态，可以是：
  - “已获得” –请求者持有锁
  - “正在等待” –请求者正在等待锁
  - “中止” –锁已超时，但尚未清除
- 如果此锁处于“正在等待”状态，则锁定此锁的ID
- 锁的类型，可以是：
  - “独占” –没有人可以同时持有该锁（主要是通过DDL操作（例如放置表）获得的）
  - “ shared_read” –任意数量的其他shared_read锁可以同时锁定同一资源（通过读取获得；令人困惑的是，插入操作还获得了shared_read锁）
  - “ shared_write” –任意数量的shared_read锁可以同时锁定同一资源，但不允许其他的shared_write锁（通过更新和删除获得）
- 与该锁关联的交易的ID（如果有）
- 上次此锁的持有者发送心跳信号表明它仍然存在
- 获取锁的时间（如果已获取）
- 请求锁定的Hive用户
- 用户正在运行的主机
- 代理信息–一个字符串，可以帮助识别发出锁定请求的实体。对于SQL客户端，这是查询ID，对于流式客户端，例如，它可能是风暴螺栓ID。

### 显示conf

> 版本信息
>
> 从[Hive 0.14.0开始](https://issues.apache.org/jira/browse/HIVE-6037)。

```sql
SHOW CONF <configuration_name>;
```

SHOW CONF返回指定[配置属性](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties)的描述。

- 默认值
- 必填类型
- 描述

请注意，SHOW CONF不显示配置属性的*当前值*。对于当前属性设置，请在CLI或HiveQL脚本（请参阅[命令](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Commands)）或Beeline（请参见[Beeline Hive命令](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-BeelineHiveCommands)）中使用“ set”命令。

### 显示TRANSACTIONS

> 版本信息
>
> 从[Hive 0.13.0开始](https://issues.apache.org/jira/browse/HIVE-6460)（请参阅[Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)）。

```sql
SHOW TRANSACTIONS;
```

当使用[Hive事务](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)时，SHOW TRANSACTIONS供管理员使用。它返回系统中所有当前打开和中止的事务的列表，包括以下信息：

- 交易编号
- 交易状态
- 开始交易的用户
- 开始交易的机器
- 事务开始的时间戳（自[Hive 2.2.0起](https://issues.apache.org/jira/browse/HIVE-11957)）
- 最后一次心跳的时间戳（自[Hive 2.2.0起](https://issues.apache.org/jira/browse/HIVE-11957)） 

### 显示COMPACTIONS

> 版本信息
>
> 从[Hive 0.13.0开始](https://issues.apache.org/jira/browse/HIVE-6460)（请参阅[Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-BasicDesign)）。

```sql
SHOW COMPACTIONS;
```

当使用[Hive事务](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)时，[SHOW COMPACTIONS](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-SHOWCOMPACTIONS)返回当前正在[压缩](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-BasicDesign)或计划进行压缩的所有表和分区的列表，包括以下信息：

- “ CompactionId”-唯一的内部ID（从[Hive 3.0开始](https://issues.apache.org/jira/browse/HIVE-16084)）
- “数据库”-Hive数据库名称
- “表格”-表格名称
- “分区”-分区名称（如果表已分区）
- “类型”-无论是大压实还是小压实
- “状态”-压缩所处的状态，可以是：
  - “启动” –在队列中等待压缩
  - “工作” –被压缩
  - “准备清洗” –压缩已经完成，旧文件计划清洗
  - “失败” –作业失败。Metastore日志将具有更多详细信息。
  - “成功” –好的
  - “尝试” –启动程序尝试安排压缩，但失败。Metastore日志将具有更多信息。
- “ Worker”-执行压缩的工作线程的线程ID（仅在工作状态下）
- “开始时间”-压实开始的时间（仅当处于工作状态或准备清洁时）
- “ Duration（ms）”-压缩所花费的时间（从[Hive 2.2开始](https://issues.apache.org/jira/browse/HIVE-15337)） 
- “ HadoopJobId”-提交的Hadoop作业的ID（从[Hive 2.2开始](https://issues.apache.org/jira/browse/HIVE-15337)）

压缩会自动启动，但也可以使用[ALTER TABLE COMPACT语句](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-AlterTable/PartitionCompact)手动启动。

## 描述

- [描述数据库](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-DescribeDatabase)
- 描述表/视图/材料化视图/列
  - [显示列统计](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-DisplayColumnStatistics)
- [描述分区](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-DescribePartition)
- [Hive 2.0+：语法更改](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-Hive2.0+:SyntaxChange)

### 描述数据库

> 版本信息
>
> 从Hive 0.7开始。

```sql
DESCRIBE DATABASE [EXTENDED] db_name;
DESCRIBE SCHEMA [EXTENDED] db_name;   -- (Note: Hive ``1.1``.``0` `and later)
```

DESCRIBE DATABASE显示数据库的名称，其注释（如果已设置注释）及其在文件系统上的根位置。SCHEMA和DATABASE的用法是可互换的–它们含义相同。在Hive 1.1.0（[HIVE-8803](https://issues.apache.org/jira/browse/HIVE-8803)）中添加了DESCRIBE SCHEMA 。

EXTENDED还显示了[数据库属性](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-CreateDatabase)。

### 描述表/视图/材料化视图/列

describe table / view /物化视图/ column语法有两种格式，具体取决于是否指定了数据库。

如果未指定数据库，则在点后提供可选的列信息：

```sql
DESCRIBE [EXTENDED|FORMATTED] 
  table_name[.col_name ( [.field_name] | [.'$elem$'] | [.'$key$'] | [.'$value$'] )* ];
                                        -- (Note: Hive 1.x.x and 0.x.x only. See "Hive 2.0+: New Syntax" below)
```

如果指定了数据库，则在空格后提供可选的列信息：

```sql
DESCRIBE [EXTENDED|FORMATTED] 
  [db_name.]table_name[ col_name ( [.field_name] | [.'$elem$'] | [.'$key$'] | [.'$value$'] )* ];
                                        -- (Note: Hive 1.x.x and 0.x.x only. See "Hive 2.0+: New Syntax" below)
```

DESCRIBE显示列的列表，包括给定表的分区列。如果指定了EXTENDED关键字，则它将以Thrift序列化形式显示表的所有元数据。通常，这仅对调试有用，对一般用途不起作用。如果指定了FORMATTED关键字，则它将以表格格式显示元数据。

注意：仅当加载数据时收集了统计信息（请参阅“ [新创建的表”](https://cwiki.apache.org/confluence/display/Hive/StatsDev#StatsDev-NewlyCreatedTables)）并且使用Hive CLI代替Thrift客户端或Beeline 时，DESCRIBE EXTENDED才会显示行数。[HIVE-6285](https://issues.apache.org/jira/browse/HIVE-6285)将解决此问题。尽管ANALYZE TABLE在数据加载后收集统计信息（请参阅[现有表](https://cwiki.apache.org/confluence/display/Hive/StatsDev#StatsDev-ExistingTables)），但它当前不提供有关行数的信息。

如果表具有复杂的列，则可以通过指定table_name.complex_col_name（对于结构元素的field_name，对于数组元素的'$ elem $'，对于地图键的'$ key $'，以及“ $ value $”代表地图价值）。您可以递归指定此参数以浏览复杂的列类型。

对于视图，可以使用DESCRIBE EXTENDED或FORMATTED来检索视图的定义。提供了两个相关的属性：用户指定的原始视图定义，以及Hive在内部使用的扩展定义。

对于实例化视图，DESCRIBE EXTENDED或FORMATTED提供有关是否启用重写以及是否将给定的实例化视图视为针对其所使用的源表中的数据进行自动重写的最新信息。

> 版本信息-分区和非分区列
>
> 在Hive 0.10.0和更早版本中，在显示DESCRIBE TABLE的列时，分区列和非分区列之间没有区别。从Hive 0.12.0开始，它们分别显示。
>
> 在Hive 0.13.0和更高版本中，如果需要，配置参数[hive.display.partition.cols.separately](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.display.partition.cols.separately)允许您使用旧的行为（[HIVE-6689](https://issues.apache.org/jira/browse/HIVE-6689)）。有关示例，请参阅[HIVE-6689补丁中](https://issues.apache.org/jira/secure/attachment/12635956/HIVE-6689.2.patch)的测试用例。

> Hive 0.10.0中已修复的错误-数据库限定符
>
> 表名称的数据库限定符在Hive 0.7.0中引入，但在DESCRIBE中被破坏，直到在Hive 0.10.0中修复了错误（[HIVE-1977](https://issues.apache.org/jira/browse/HIVE-1977)）。

> Hive 0.13.0中已修复的错误-带引号的标识符
>
> 在Hive 0.13.0之前，DESCRIBE不接受表标识符周围的反引号（`），因此DESCRIBE不能用于名称与保留关键字匹配的表（[HIVE-2949](https://issues.apache.org/jira/browse/HIVE-2949)和[HIVE-6187](https://issues.apache.org/jira/browse/HIVE-6187)）。从0.13.0开始，当配置参数[hive.support.quoted.identifiers](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.support.quoted.identifiers)的默认值为“ `column`”（[HIVE-6013](https://issues.apache.org/jira/browse/HIVE-6013)）时，将逐字处理反引号内指定的所有标识符。唯一的例外是双反引号（``）表示单个反引号字符。

#### **显示列统计**

> 版本信息
>
> 从Hive 0.14.0开始；参见[HIVE-7050](https://issues.apache.org/jira/browse/HIVE-7050)和[HIVE-7051](https://issues.apache.org/jira/browse/HIVE-7051)。（从[Hive 0.10.0开始，](https://issues.apache.org/jira/browse/HIVE-1362)可以使用ANALYZE TABLE的FOR COLUMNS选项。）

ANALYZE TABLE *table_name列的* COMPUTE STATISTICS将为指定表中的所有列（如果表已分区，则为所有分区）计算列统计信息。要查看收集的列统计信息，可以使用以下语句：

```sql
DESCRIBE FORMATTED [db_name.]table_name column_name;                              -- (Note: Hive 0.14.0 and later)
DESCRIBE FORMATTED [db_name.]table_name column_name PARTITION (partition_spec);   -- (Note: Hive 0.14.0 to 1.x.x)
                                                                                  -- (see "Hive 2.0+: New Syntax" below)
```

有关ANALYZE TABLE命令的更多信息，请参见[Hive：现有表中的统计](https://cwiki.apache.org/confluence/display/Hive/StatsDev#StatsDev-ExistingTables)信息。

### 描述分区

describe分区语法有两种格式，具体取决于是否指定了数据库。

如果未指定数据库，则在点后提供可选的列信息：

```sql
DESCRIBE [EXTENDED|FORMATTED] table_name[.column_name] PARTITION partition_spec;
                                        -- (Note: Hive 1.x.x and 0.x.x only. See "Hive 2.0+: New Syntax" below)
```

如果指定了数据库，则在空格后提供可选的列信息：

```sql
DESCRIBE [EXTENDED|FORMATTED] [db_name.]table_name [column_name] PARTITION partition_spec;
                                        -- (Note: Hive 1.x.x and 0.x.x only. See "Hive 2.0+: New Syntax" below)
```

该语句列出了给定分区的元数据。输出与DESCRIBE table_name相似。当前，在准备计划时不使用与特定分区关联的列信息。作为蜂巢1.2（的[HIVE-10307](https://issues.apache.org/jira/browse/HIVE-10307)），在指定的分区列值*partition_spec*是类型验证，转换并当正规化为列类型[hive.typecheck.on.insert](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.typecheck.on.insert)被设置为true（默认）。这些值可以是数字文字。

例：

```shell
hive> show partitions part_table;
OK
d=abc
 
 
hive> DESCRIBE extended part_table partition (d='abc');
OK
i                       int                                        
d                       string                                     
                  
# Partition Information         
# col_name              data_type               comment            
                  
d                       string                                     
                  
Detailed Partition Information  Partition(values:[abc], dbName:default, tableName:part_table, createTime:1459382234, lastAccessTime:0, sd:StorageDescriptor(cols:[FieldSchema(name:i, type:int, comment:null), FieldSchema(name:d, type:string, comment:null)], location:file:/tmp/warehouse/part_table/d=abc, inputFormat:org.apache.hadoop.mapred.TextInputFormat, outputFormat:org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat, compressed:false, numBuckets:-1, serdeInfo:SerDeInfo(name:null, serializationLib:org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe, parameters:{serialization.format=1}), bucketCols:[], sortCols:[], parameters:{}, skewedInfo:SkewedInfo(skewedColNames:[], skewedColValues:[], skewedColValueLocationMaps:{}), storedAsSubDirectories:false), parameters:{numFiles=1, COLUMN_STATS_ACCURATE=true, transient_lastDdlTime=1459382234, numRows=1, totalSize=2, rawDataSize=1})  
Time taken: 0.325 seconds, Fetched: 9 row(s)
 
 
hive> DESCRIBE formatted part_table partition (d='abc');
OK
# col_name              data_type               comment            
                  
i                       int                                        
                  
# Partition Information         
# col_name              data_type               comment            
                  
d                       string                                     
                  
# Detailed Partition Information                
Partition Value:        [abc]                   
Database:               default                 
Table:                  part_table              
CreateTime:             Wed Mar 30 16:57:14 PDT 2016    
LastAccessTime:         UNKNOWN                 
Protect Mode:           None                    
Location:               file:/tmp/warehouse/part_table/d=abc    
Partition Parameters:           
        COLUMN_STATS_ACCURATE   true               
        numFiles                1                  
        numRows                 1                  
        rawDataSize             1                  
        totalSize               2                  
        transient_lastDdlTime   1459382234         
                  
# Storage Information           
SerDe Library:          org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe      
InputFormat:            org.apache.hadoop.mapred.TextInputFormat        
OutputFormat:           org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat      
Compressed:             No                      
Num Buckets:            -1                      
Bucket Columns:         []                      
Sort Columns:           []                      
Storage Desc Params:            
        serialization.format    1                  
Time taken: 0.334 seconds, Fetched: 35 row(s)
```

### Hive 2.0+：语法更改

> Hive 2.0+：新语法
>
> 从Hive 2.0发行版开始，describe table命令的语法更改向后不兼容。有关详细信息，请参见[HIVE-12184](https://issues.apache.org/jira/browse/HIVE-12184)。

```sql
DESCRIBE [EXTENDED | FORMATTED]
    [db_name.]table_name [PARTITION partition_spec] [col_name ( [.field_name] | [.'$elem$'] | [.'$key$'] | [.'$value$'] )* ];
```

警告：新语法可能会破坏当前脚本。

- 它不再接受DOT分隔的table_name和column_name。它们必须以空格分隔。DB和TABLENAME是DOT分隔的。column_name仍可以包含复杂数据类型的DOT。
- 可选的partition_spec必须出现在table_name之后，但在可选的column_name之前。在以前的语法中，column_name出现在table_name和partition_spec之间。

**例子**

```sql
DESCRIBE FORMATTED default.src_table PARTITION (part_col = 100) columnA;
DESCRIBE default.src_thrift lintString.$elem$.myint;
```

## Abort

- [Abort Transactions](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AbortTransactions)

### Abort Transactions

> 版本信息
>
> 从[Hive 1.3.0和2.1.0开始](https://issues.apache.org/jira/browse/HIVE-12634)（请参阅[Hive Transactions ](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions#HiveTransactions-BasicDesign)）。

```sql
ABORT TRANSACTIONS transactionID [ transactionID ...];
```

ABORT TRANSACTIONS会从Hive元存储库中清除指定的事务ID，以便用户无需直接与该元存储库进行交互即可删除悬空或失败的事务。在Hive 1.3.0和2.1.0（[HIVE-12634](https://issues.apache.org/jira/browse/HIVE-12634)）中添加了ABORT TRANSACTIONS 。

```sql
ABORT TRANSACTIONS 0000007 0000008 0000010 0000015;
```

该命令可以与[SHOW TRANSACTIONS](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=82706445#LanguageManualDDL-ShowTransactions)一起使用。后者可以帮助找出要清除的候选交易ID。

# 预定查询

文档可在“ [计划查询” ](https://cwiki.apache.org/confluence/display/Hive/Scheduled+Queries)页面上找到。

# HCatalog和WebHCat DDL

有关HCatalog和WebHCat中DDL的信息，请参阅：

- [HCatalog手册](https://cwiki.apache.org/confluence/display/Hive/HCatalog)中的[HCatalog ](https://cwiki.apache.org/confluence/display/Hive/HCatalog)[DDL](https://cwiki.apache.org/confluence/display/Hive/HCatalog+CLI#HCatalogCLI-HCatalogDDL)
- [WebHCat手册](https://cwiki.apache.org/confluence/display/Hive/WebHCat)中的[WebHCat ](https://cwiki.apache.org/confluence/display/Hive/WebHCat)[DDL资源](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference+AllDDL)













