# [DML语言手册](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML)

> 翻译版
>
> 原文地址：https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML

### 将文件加载到表中

Hive在将数据加载到表中时不会进行任何转换。加载操作当前是纯复制/移动操作，可将数据文件移动到与Hive表相对应的位置。

##### 句法

```sql
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]
 
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)] [INPUTFORMAT 'inputformat' SERDE 'serde'] (3.0 or later)
```

##### 概要

Hive 3.0之前的加载操作是纯复制/移动操作，可将数据文件移动到与Hive表相对应的位置。

- 文件路径

  可以是：

  - 相对路径，例如 `project/data1`
  - 绝对路径，例如 `/user/hive/project/data1`
  - 具有方案和（可选）权限的完整URI，例如 `hdfs://namenode:9000/user/hive/project/data1`

- 加载到的目标可以是表或分区。如果表已分区，则必须通过为所有分区列指定值来指定表的特定分区。

- *filepath*可以引用文件（在这种情况下，Hive会将文件移至表中），也可以是目录（在这种情况下，Hive会将目录中的所有文件移至表中）。无论哪种情况，文件*路径都会*寻址一组文件。

- 如果指定了关键字LOCAL，则：

  - load命令将在本地文件系统中查找文件*路径*。如果指定了相对路径，它将相对于用户的当前工作目录进行解释。用户也可以为本地文件指定完整的URI-例如：`file:///user/hive/project/data1`
  - load命令将尝试将*filepath*寻址的所有文件复制到目标文件系统。通过查看表的location属性可以推断目标文件系统。然后，复制的数据文件将被移到表中。
  - 注意：如果对HiveServer2实例运行此命令，则本地路径是指HiveServer2实例上的路径。HiveServer2必须具有适当的权限才能访问该文件。

- 如果

  未

  指定关键字LOCAL ，则Hive将使用

  filepath

  的完整URI（如果已指定），或将应用以下规则：

  - 如果未指定scheme或Authority，则Hive将使用hadoop配置变量`fs.default.name`中的方案和Authority，该变量指定Namenode URI。
  - 如果该路径不是绝对路径，则Hive将相对于 `/user/`
  - Hive会将文件*路径*寻址的文件*移动*到表（或分区）中

- 如果使用OVERWRITE关键字，则目标表（或分区）的内容将被删除，并由*filepath*引用的文件替换；否则，*filepath*引用的文件将被添加到表中。

Hive 3.0及更高版本支持其他加载操作，因为Hive在内部将加载重写为INSERT AS SELECT。

- 但是，如果表具有分区，则load命令没有分区，则负载将转换为INSERT AS SELECT，并假定最后一组列为分区列。如果文件不符合预期的架构，它将引发错误。
- 如果表是存储分区的，则适用以下规则：
  - 在严格模式下：启动INSERT AS SELECT作业。
  - 在非严格模式下：如果文件名符合命名约定（如果文件属于存储桶0，则应将其命名为000000_0或000000_0_copy_1，或者如果其属于存储桶2，则名称应类似于000002_0或000002_0_copy_3，依此类推。 ），那么它将是纯复制/移动操作，否则它将启动INSERT AS SELECT作业。
- 如果每个文件都符合架构，则文件*路径*可以包含子目录。
- *inputformat*可以是任何Hive输入格式，例如文本，ORC等。
- *serde*可以是关联的Hive SERDE。
- 无论*inputformat*和*SERDE*区分大小写。

这种模式的示例：

```sql
CREATE TABLE tab1 (col1 int, col2 int) PARTITIONED BY (col3 int) STORED AS ORC;
LOAD DATA LOCAL INPATH 'filepath' INTO TABLE tab1;
```

此处，缺少分区信息，否则将产生错误，但是，如果位于文件*路径中*的文件符合表架构，使得每一行以分区列结尾，则负载将被重写为INSERT AS SELECT工作。

未压缩的数据应如下所示：

（1,2,3），（2,3,4），（4,5,3）等。

##### 笔记

- *文件路径*不能包含子目录（如上所述，Hive 3.0或更高版本除外）。
- 如果未给出关键字LOCAL，则文件*路径*必须引用与表（或分区）位置相同的文件系统中的文件。
- Hive进行了一些最少的检查，以确保要加载的文件与目标表匹配。当前，它会检查表是否以sequencefile格式存储，正在加载的文件也是sequencefile，反之亦然。
- 在版本0.13.0（[HIVE-6048](https://issues.apache.org/jira/browse/HIVE-6048)）中修复了一个阻止在文件名包含“ +”字符时加载文件的错误。
- 如果您的数据文件已压缩，请阅读[CompressedStorage](https://cwiki.apache.org/confluence/display/Hive/CompressedStorage)。

### 从查询将数据插入Hive表

可以使用insert子句将查询结果插入表中。

##### 句法

```sql
Standard syntax:
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...) [IF NOT EXISTS]] select_statement1 FROM from_statement;
INSERT INTO TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1 FROM from_statement;
 
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
```

##### 概要

- 

  INSERT OVERWRITE将覆盖表或分区中的任何现有数据

  - 除非`IF NOT EXISTS`为分区提供（从Hive [0.9.0开始](https://issues.apache.org/jira/browse/HIVE-2612)）。
  - 从Hive 2.3.0（[HIVE-15880](https://issues.apache.org/jira/browse/HIVE-15880)）开始，如果表具有[TBLPROPERTIES](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-listTableProperties)（“ auto.purge” =“ true”），则对该表运行INSERT OVERWRITE查询时，该表的先前数据不会移至“已删除邮件”。此功能仅适用于托管表（请参阅[托管表](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ManagedandExternalTables)），并且在未设置“ auto.purge”属性或将其设置为false时将关闭此功能。

- INSERT INTO将追加到表或分区，使现有数据保持不变。（注意：INSERT INTO语法仅从版本0.8开始可用。）

  - 从Hive [0.13.0开始](https://issues.apache.org/jira/browse/HIVE-6406)，可以通过使用[TBLPROPERTIES](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-CreateTable)创建 表来使其***不可变\***[（“ immutable” =“ true”）](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-CreateTable)。默认值为“ immutable” =“ false”。
    如果已经存在任何数据，则不允许对不可变表执行INSERT INTO行为，尽管如果不可变表为空，则INSERT INTO仍然有效。INSERT OVERWRITE的行为不受“不可变”表属性的影响。
    不可变表可以防止意外更新，因为脚本将数据加载到该表中会导致错误地多次运行。对不可变表的第一次插入成功，而随后的插入失败，导致表中只有一组数据， 

- 可以对表或分区进行插入。如果表已分区，则必须通过为所有分区列指定值来指定表的特定分区。如果[hive.typecheck.on.insert](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.typecheck.on.insert)设置为true，那么将验证，转换和标准化这些值以使其符合其列类型（从Hive [0.12.0](https://issues.apache.org/jira/browse/HIVE-5297)开始）。 
- 可以在同一查询中指定多个insert子句（也称为*Multi Table Insert*）。
- 每个select语句的输出将写入所选的表（或分区）。当前，OVERWRITE关键字是强制性的，它表示将所选表或分区的内容替换为对应的select语句的输出。
- 输出格式和序列化类由表的元数据确定（通过表上的DDL命令指定）。
- 从[Hive 0.14开始](https://issues.apache.org/jira/browse/HIVE-5317)，如果表具有实现AcidOutputFormat的OutputFormat，并且系统配置为使用实现ACID 的[事务](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)管理器，则将对该表禁用INSERT OVERWRITE。这是为了避免用户无意间覆盖交易历史记录。通过使用[TRUNCATE TABLE](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-TruncateTable)（用于未分区的表）或[DROP PARTITION](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-DropPartitions)后接INSERT INTO，可以实现相同的功能。
- 从Hive [1.1.0开始](https://issues.apache.org/jira/browse/HIVE-9353)，TABLE关键字是可选的。
- 从Hive [1.2.0开始，](https://issues.apache.org/jira/browse/HIVE-9481)每个INSERT INTO T都可以采用像INSERT INTO T（z，x，c1）这样的列列表。有关示例，请参见 [HIVE-9481](https://issues.apache.org/jira/browse/HIVE-9481)的说明。

##### 笔记

- 多表插入可最大程度地减少所需的数据扫描次数。Hive可以通过只扫描一次输入数据（并应用不同的查询运算符）到输入数据来将数据插入到多个表中。
- 从[Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-1180)开始 ，select语句可以包含一个或多个通用表表达式（CTE），如[SELECT语法](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select#LanguageManualSelect-SelectSyntax)所示 。有关示例，请参见 [Common Table Expression](https://cwiki.apache.org/confluence/display/Hive/Common+Table+Expression#CommonTableExpression-CTEinViews,CTAS,andInsertStatements)。

##### 动态分区插入

> 版本信息
>
> 此信息反映了Hive 0.12中的情况；在Hive 0.6中添加了动态分区插入。

在动态分区插入中，用户可以提供部分分区规范，这意味着只需在PARTITION子句中指定分区列名称的列表。列值是可选的。如果给出了分区列的值，我们称其为静态分区，否则为动态分区。每个动态分区列都有一个来自select语句的对应输入列。这意味着动态分区的创建取决于输入列的值。动态分区列必须在SELECT语句中的列中**最后指定**，并**以**它们在PARTITION（）子句中出现的**顺序指定**。自Hive 3.0.0（[HIVE-19083](https://issues.apache.org/jira/browse/HIVE-19083)），无需指定动态分区列。如果未指定，Hive将自动生成分区规范。

默认情况下，在Hive 0.9.0之前禁用动态分区插入，而在Hive [0.9.0](https://issues.apache.org/jira/browse/HIVE-2835)和更高版本中默认启用。这些是动态分区插入的相关配置属性：

| 配置属性                                   | 默认     | 注意                                                         |
| :----------------------------------------- | :------- | :----------------------------------------------------------- |
| `hive.exec.dynamic.partition`              | `true`   | 需要设置`true`为启用动态分区插入                             |
| `hive.exec.dynamic.partition.mode`         | `strict` | 在`strict`模式下，用户必须至少指定一个静态分区，以防用户意外覆盖所有分区；在`nonstrict`模式下，允许所有分区都是动态的 |
| `hive.exec.max.dynamic.partitions.pernode` | 100      | 每个映射器/还原器节点中允许创建的最大动态分区数              |
| `hive.exec.max.dynamic.partitions`         | 1000     | 总共允许创建的最大动态分区数                                 |
| `hive.exec.max.created.files`              | 100000   | MapReduce作业中所有映射器/还原器创建的最大HDFS文件数         |
| `hive.error.on.empty.partition`            | `false`  | 如果动态分区插入生成空结果，是否引发异常                     |

###### 例

```sql
FROM page_view_stg pvs
INSERT OVERWRITE TABLE page_view PARTITION(dt='2008-06-08', country)
       SELECT pvs.viewTime, pvs.userid, pvs.page_url, pvs.referrer_url, null, null, pvs.ip, pvs.cnt
```

在这里，`country`分区将由`SELECT`子句的最后一列（即`pvs.cnt`）动态创建。请注意，未使用该名称。在`nonstrict`模式下，`dt`分区也可以动态创建。

###### 附加文件

- 设计文件
  - [原始设计文件](https://issues.apache.org/jira/secure/attachment/12437909/dp_design.txt)
  - [HIVE-936](https://issues.apache.org/jira/browse/HIVE-936)
- [教程：动态分区插入](https://cwiki.apache.org/confluence/display/Hive/Tutorial#Tutorial-Dynamic-PartitionInsert)
- HCatalog动态分区
  - [与猪一起使用](https://cwiki.apache.org/confluence/display/Hive/HCatalog+DynamicPartitions#HCatalogDynamicPartitions-UsagewithPig)
  - [MapReduce的用法](https://cwiki.apache.org/confluence/display/Hive/HCatalog+DynamicPartitions#HCatalogDynamicPartitions-UsagefromMapReduce)

### 通过查询将数据写入文件系统

通过使用上面语法的一些细微变化，可以将查询结果插入文件系统目录：

##### 句法

```sql
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
```

##### 概要

- 目录可以是完整的URI。如果未指定scheme或Authority，则Hive将使用hadoop配置变量`fs.default.name`中的方案和Authority，该变量指定Namenode URI。
- 如果使用LOCAL关键字，则Hive会将数据写入本地文件系统上的目录。
- 写入文件系统的数据被序列化为文本，列之间用^ A分隔，行之间用换行符分隔。如果任何列都不是原始类型，那么这些列将序列化为JSON格式。

##### 笔记

- 对目录，本地目录和表（或分区）的INSERT OVERWRITE语句可以在同一查询中一起使用。
- 对HDFS文件系统目录执行INSERT OVERWRITE语句是从Hive提取大量数据的最佳方法。Hive可以从map-reduce作业中并行写入HDFS目录。
- 如您所料，该目录已被覆盖；换句话说，如果指定的路径存在，它将被破坏并替换为输出。
- 从Hive [0.11.0开始](https://issues.apache.org/jira/browse/HIVE-3682)，可以指定使用的分隔符；在早期版本中，它始终是^ A字符（\ 001）。但是，仅Hive 0.11.0至1.1.0的LOCAL写入支持自定义分隔符–此错误已在1.2.0版中修复（请参见 [HIVE-5672](https://issues.apache.org/jira/browse/HIVE-5672)）。
- 在[Hive 0.14中](https://issues.apache.org/jira/browse/HIVE-5317)，插入到[ACID](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)兼容表中将在选择和插入期间停用向量化。这将自动完成。仍然可以使用矢量化查询已插入数据的ACID表。

### 从SQL向表中插入值

INSERT ... VALUES语句可用于直接从SQL将数据插入表中。

> 版本信息
>
> INSERT ... VALUES从[Hive 0.14](https://issues.apache.org/jira/browse/HIVE-5317)开始可用。

##### 句法

```sql
Standard Syntax:
INSERT INTO TABLE tablename [PARTITION (partcol1[=val1], partcol2[=val2] ...)] VALUES values_row [, values_row ...]
  
Where values_row is:
( value [, value ...] )
where a value is either null or any valid SQL literal
```

##### 概要

- VALUES子句中列出的每一行都插入到表*tablename中*。
- 必须为表中的每一列提供值。尚不支持允许用户仅将值插入某些列的标准SQL语法。为了模仿标准SQL，可以为用户不希望为其分配值的列提供空值。
- 支持动态分区的方式与[INSERT ... SELECT相同](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-DynamicPartitionInserts)。
- 如果要插入的表支持[ACID，](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)并且正在使用支持[ACID](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)的事务管理器，则此操作将在成功完成后自动提交。
- Hive不支持复杂类型（数组，映射，结构，联合）的文字，因此无法在INSERT INTO ... VALUES子句中使用它们。这意味着用户无法使用INSERT INTO ... VALUES子句将数据插入到复杂数据类型列中。

##### 例子

```sql
CREATE TABLE students (name VARCHAR(64), age INT, gpa DECIMAL(3, 2))
  CLUSTERED BY (age) INTO 2 BUCKETS STORED AS ORC;
 
INSERT INTO TABLE students
  VALUES ('fred flintstone', 35, 1.28), ('barney rubble', 32, 2.32);
 
 
CREATE TABLE pageviews (userid VARCHAR(64), link STRING, came_from STRING)
  PARTITIONED BY (datestamp STRING) CLUSTERED BY (userid) INTO 256 BUCKETS STORED AS ORC;
 
INSERT INTO TABLE pageviews PARTITION (datestamp = '2014-09-23')
  VALUES ('jsmith', 'mail.com', 'sports.com'), ('jdoe', 'mail.com', null);
 
INSERT INTO TABLE pageviews PARTITION (datestamp)
  VALUES ('tjohnson', 'sports.com', 'finance.com', '2014-09-23'), ('tlee', 'finance.com', null, '2014-09-21');
  
INSERT INTO TABLE pageviews
  VALUES ('tjohnson', 'sports.com', 'finance.com', '2014-09-23'), ('tlee', 'finance.com', null, '2014-09-21');
```

### 更新资料

> 版本信息
>
> 更新从[Hive 0.14](https://issues.apache.org/jira/browse/HIVE-5317)开始可用。
>
> 更新只能在支持ACID的表上执行。有关详细信息，请参见[Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)。

##### 句法

```sql
Standard Syntax:``UPDATE tablename SET column = value [, column = value ...] [WHERE expression]
```

##### 概要

- 引用的列必须是要更新的表的列。
- 分配的值必须是Hive在select子句中支持的表达式。因此，支持算术运算符，UDF，强制转换，文字等。不支持子查询。
- 仅匹配WHERE子句的行将被更新。
- 分区列无法更新。
- 存储桶列无法更新。
- 在Hive 0.14中，成功完成此操作后，更改将自动提交。

##### 笔记

- 矢量化将被关闭以进行更新操作。这是自动的，不需要用户采取任何措施。非更新操作不受影响。仍然可以使用矢量化查询更新的表。
- 在版本0.14中，建议您在进行更新时设置 [hive.optimize.sort.dynamic.partition](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.optimize.sort.dynamic.partition) = false，因为这会产生更有效的执行计划。

### 删除

> 版本信息
>
> 从[Hive 0.14](https://issues.apache.org/jira/browse/HIVE-5317)开始可以使用DELETE 。
>
> 删除只能在支持ACID的表上执行。有关详细信息，请参见[Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)。

##### 句法

```sql
Standard Syntax:
DELETE FROM tablename [WHERE expression]
```

##### 概要

- 仅匹配WHERE子句的行将被删除。
- 在Hive 0.14中，成功完成此操作后，更改将自动提交。

##### 笔记

- 矢量化将被关闭以进行删除操作。这是自动的，不需要用户采取任何措施。非删除操作不受影响。带有删除数据的表仍可以使用矢量化查询。
- 在版本0.14中，建议您在执行删除操作时设置 [hive.optimize.sort.dynamic.partition](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.optimize.sort.dynamic.partition) = false，因为这会产生更有效的执行计划。

### Merge

> 版本信息
>
> 从[Hive 2.2](https://issues.apache.org/jira/browse/HIVE-10924)开始可以使用MERGE 。
>
> 只能在支持ACID的表上执行合并。有关详细信息，请参见[Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)。

##### 句法

```sql
Standard Syntax:
MERGE INTO <target table> AS T USING <source expression/table> AS S
ON <boolean expression1>
WHEN MATCHED [AND <boolean expression2>] THEN UPDATE SET <set clause list>
WHEN MATCHED [AND <boolean expression3>] THEN DELETE
WHEN NOT MATCHED [AND <boolean expression4>] THEN INSERT VALUES<value list>
```

##### 概要

- [合并](https://en.wikipedia.org/wiki/Merge_(SQL))允许基于与源表的联接结果在目标表上执行操作。
- 在Hive 2.2中，成功完成此操作后，更改将自动提交。

##### 性能说明

如果ON子句与源中的多行匹配目标中的一行，则SQL Standard要求引发错误。该检查的计算量很大，并且可能会严重影响MERGE语句的整体运行时间。  [hive.merge.cardinality.check](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.merge.cardinality.check) = false可用于禁用该检查，[后果](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.merge.cardinality.check)自负。如果禁用该检查 ，但是该语句具有这种交叉联接效果，则可能导致数据损坏。

##### 笔记

- 1、2或3 WHEN子句可以出现；每种类型最多1个：UPDATE / DELETE / INSERT。
- 未匹配时必须是最后的WHEN子句。
- 如果同时存在UPDATE和DELETE子句，则语句中的第一个子句必须包含[AND <布尔表达式>]。
- 矢量化将关闭以进行合并操作。这是自动的，不需要用户采取任何措施。非删除操作不受影响。带有删除数据的表仍可以使用矢量化查询。

##### 例子

- 看 [这里](https://community.hortonworks.com/articles/97113/hive-acid-merge-by-example.html)。

 