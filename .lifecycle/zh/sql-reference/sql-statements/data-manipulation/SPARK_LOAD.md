---
displayed_sidebar: English
---

# Spark Load

## 描述

Spark Load 通过外部 Spark 资源对导入的数据进行预处理，提升了 StarRocks 大量数据的导入性能，并节约了 StarRocks 集群的计算资源。它主要用于初始迁移和大规模数据导入 StarRocks 的场景。

Spark Load 是一种异步导入方法。用户需要通过 MySQL 协议创建 Spark 类型的导入任务，并通过 SHOW LOAD 命令查看导入结果。

> **注意**
- 只有对 StarRocks 表拥有 **INSERT** 权限的用户才能将数据加载到这些表中。如果您没有 **INSERT** 权限，请按照 [GRANT](../account-management/GRANT.md) 命令提供的说明，授予您用来连接 StarRocks 集群的用户 **INSERT** 权限。
- 当使用 Spark Load 向 StarRocks 表导入数据时，该表的分桶列不能是 DATE、DATETIME 或 DECIMAL 类型。

语法

```sql
LOAD LABEL load_label
(
data_desc1[, data_desc2, ...]
)
WITH RESOURCE resource_name
[resource_properties]
[opt_properties]
```

1. load_label

当前导入批次的标签，数据库内唯一。

语法：

```sql
[database_name.]your_label
```

2. data_desc

用来描述一批导入数据的描述。

语法：

```sql
DATA INFILE
(
"file_path1"[, file_path2, ...]
)
[NEGATIVE]
INTO TABLE `table_name`
[PARTITION (p1, p2)]
[COLUMNS TERMINATED BY "column_separator"]
[FORMAT AS "file_type"]
[(column_list)]
[COLUMNS FROM PATH AS (col2, ...)]
[SET (k1 = func(k2))]
[WHERE predicate]

DATA FROM TABLE hive_external_tbl
[NEGATIVE]
INTO TABLE tbl_name
[PARTITION (p1, p2)]
[SET (k1=f1(xx), k2=f2(xx))]
[WHERE predicate]
```

备注

```plain
file_path:

The file path can be specified to one file, or the * wildcard can be used to specify all files in a directory. Wildcards must match files, not directories.

hive_external_tbl:

hive external table name.
It is required that the columns in the imported starrocks table must exist in the hive external table.
Each load task only supports loading from one Hive external table.
Cannot be used with file_ path mode at the same time.

PARTITION:

If this parameter is specified, only the specified partition will be imported, and the data outside the imported partition will be filtered out.
If not specified, all partitions of table will be imported by default.

NEGATIVE:

If this parameter is specified, it is equivalent to loading a batch of "negative" data. Used to offset the same batch of previously imported data.
This parameter is only applicable when the value column exists and the aggregation type of the value column is SUM only.

column_separator:

Specifies the column separator in the import file. Default is \ t
If it is an invisible character, you need to prefix it with \ \ x and use hexadecimal to represent the separator.
For example, the separator of hive file \ x01 is specified as "\ \ x01"

file_type:

Used to specify the type of imported file. Currently, supported file types are csv, orc, and parquet.

column_list:

Used to specify the correspondence between the columns in the import file and the columns in the table.
When you need to skip a column in the import file, specify the column as a column name that does not exist in the table.

Syntax:
(col_name1, col_name2, ...)

SET:

If specify this parameter, you can convert a column of the source file according to the function, and then import the converted results into table. Syntax is column_name = expression.
Only Spark SQL build_in functions are supported. Please refer to https://spark.apache.org/docs/2.4.6/api/sql/index.html.
Give a few examples to help understand.
Example 1: there are three columns "c1, c2, c3" in the table, and the first two columns in the source file correspond to (c1, c2), and the sum of the last two columns corresponds to C3; then columns (c1, c2, tmp_c3, tmp_c4) set (c3 = tmp_c3 + tmp_c4) needs to be specified;
Example 2: there are three columns "year, month and day" in the table, and there is only one time column in the source file in the format of "2018-06-01 01:02:03".
Then you can specify columns (tmp_time) set (year = year (tmp_time), month = month (tmp_time), day = day (tmp_time)) to complete the import.

WHERE:

Filter the transformed data, and only the data that meets the where condition can be imported. Only the column names in the table can be referenced in the WHERE statement
```

3. resource_name

可以通过 SHOW RESOURCES 命令查看所使用的 Spark 资源的名称。

4. resource_properties

当您有临时需求，比如需要修改 Spark 和 HDFS 的配置，您可以在这里设置参数，这些参数只会对当前的 Spark 加载作业生效，不会影响 StarRocks 集群中现有的配置。

5. opt_properties

用于指定一些特殊的参数。

语法：

```sql
[PROPERTIES ("key"="value", ...)]
```

您可以指定以下参数： timeout：指定导入操作的超时时间，默认为 4 小时，单位为秒。 max_filter_ratio：允许过滤的数据最大比例（例如，由于数据不规范等原因），默认不允许过滤。 strict mode：是否严格限制数据，默认为否。 timezone：指定某些受时区影响的函数的时区，例如 strftime / alignment_timestamp / from_unixtime 等，详细信息请参考【时区】文档。如果未指定，默认使用“亚洲/上海”时区。

6. 导入数据格式示例

整数类型 (TINYINT/SMALLINT/INT/BIGINT/LARGEINT): 1, 1000, 1234 浮点类型 (FLOAT/DOUBLE/DECIMAL): 1.1, 0.23, .356 日期类型 (DATE/DATETIME): 2017-10-03, 2017-06-13 12:34:03（注：对于其他日期格式，可以在导入命令中使用 strftime 或 time_format 函数进行转换） 字符串类型 (CHAR/VARCHAR): "I am a student", "a"

NULL 值表示法：\N

## 示例

1. 从 HDFS 导入一批数据，并指定超时时间和过滤比例。使用名为 my_spark 的 Spark 资源。

   ```sql
   LOAD LABEL example_db.label1
   (
   DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
   INTO TABLE `my_table`
   )
   WITH RESOURCE 'my_spark'
   PROPERTIES
   (
       "timeout" = "3600",
       "max_filter_ratio" = "0.1"
   );
   ```

   其中 hdfs_host 是 NameNode 的主机地址，hdfs_port 是 fs.defaultfs 的端口（默认为 9000）。

2. 从 HDFS 导入一批以逗号为分隔符的“负数”数据，使用通配符 * 指定目录中的所有文件，并设置 Spark 资源的临时参数。

   ```sql
   LOAD LABEL example_db.label3
   (
   DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/*")
   NEGATIVE
   INTO TABLE `my_table`
   COLUMNS TERMINATED BY ","
   )
   WITH RESOURCE 'my_spark'
   (
       "spark.executor.memory" = "3g",
       "broker.username" = "hdfs_user",
       "broker.password" = "hdfs_passwd"
   );
   ```

3. 从 HDFS 导入一批数据，指定分区，并对导入文件的列进行转换，具体如下：

   ```plain
   The table structure is:
   k1 varchar(20)
   k2 int
   
   Assume that the data file has only one line of data:
   
   Adele,1,1
   
   Each column in the data file corresponds to each column specified in the import statement:
   k1,tmp_k2,tmp_k3
   
   The conversion is as follows:
   
   1. k1: no conversion
   2. k2:is the sum of tmp_ k2 and tmp_k3
   
   LOAD LABEL example_db.label6
   (
   DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
   INTO TABLE `my_table`
   PARTITION (p1, p2)
   COLUMNS TERMINATED BY ","
   (k1, tmp_k2, tmp_k3)
   SET (
   k2 = tmp_k2 + tmp_k3
   )
   )
   WITH RESOURCE 'my_spark';
   ```

4. 提取文件路径中的分区字段。

   如有必要，将根据表中定义的字段类型解析文件路径中的分区字段，这类似于 Spark 中的分区发现功能。

   ```sql
   LOAD LABEL example_db.label10
   (
   DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/*/*")
   INTO TABLE `my_table`
   (k1, k2, k3)
   COLUMNS FROM PATH AS (city, utc_date)
   SET (uniq_id = md5sum(k1, city))
   )
   WITH RESOURCE 'my_spark';
   ```

   hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing 该目录包括以下文件：

   [hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv，hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv, ...]

   从文件路径中提取 city 和 utc_date 字段。

5. 过滤待导入的数据，只导入 k1 值大于 10 的列。

   ```sql
   LOAD LABEL example_db.label10
   (
   DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
   INTO TABLE `my_table`
   WHERE k1 > 10
   )
   WITH RESOURCE 'my_spark';
   ```

6. 从 Hive 外部表导入数据，并通过全局词典将源表中的 uuid 列转换为 bitmap 类型。

   ```sql
   LOAD LABEL db1.label1
   (
   DATA FROM TABLE hive_t1
   INTO TABLE tbl1
   SET
   (
   uuid=bitmap_dict(uuid)
   )
   )
   WITH RESOURCE 'my_spark';
   ```
