---
displayed_sidebar: English
---

# Spark Load

## 描述

Spark Load 通过外部 Spark 资源对导入的数据进行预处理，提高了 StarRocks 大量数据的导入性能，节省了 StarRocks 集群的计算资源。它主要用于初次迁移和大量数据导入 StarRocks 的场景。

Spark Load 是一种异步导入方法。用户需要通过 MySQL 协议创建 Spark 类型的导入任务，并通过 `SHOW LOAD` 查看导入结果。

> **注意**
- 您只能以对 StarRocks 表具有 INSERT 权限的用户身份将数据加载到这些 StarRocks 表中。如果您没有 INSERT 权限，请按照 [GRANT](../account-management/GRANT.md) 中提供的说明向用于连接到您的 StarRocks 集群的用户授予 INSERT 权限。
- 使用 Spark Load 向 StarRocks 表加载数据时，StarRocks 表的桶列不能为 DATE、DATETIME 或 DECIMAL 类型。

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

1.load_label

当前导入批次的标签。在数据库中唯一。

语法：

```sql
[database_name.]your_label
```

2.data_desc

用于描述一批导入的数据。

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

注释

```plain
file_path:

文件路径可以指定为一个文件，或者使用 * 通配符指定目录中的所有文件。通配符必须匹配文件，而不是目录。

hive_external_tbl:

Hive 外部表名称。
要求导入 StarRocks 表的列必须存在于 Hive 外部表中。
每个加载任务只支持从一个 Hive 外部表加载。
不能与 file_path 模式同时使用。

PARTITION:

如果指定了此参数，只会导入指定的分区，导入分区之外的数据将被过滤掉。
如果未指定，默认导入表的所有分区。

NEGATIVE:

如果指定了此参数，相当于加载一批“负”数据。用于抵消之前导入的同一批数据。
此参数仅在值列存在且值列的聚合类型仅为 SUM 时适用。

column_separator:

指定导入文件中的列分隔符。默认为 \t
如果是不可见字符，需要用 \\x 前缀并使用十六进制表示分隔符。
例如，指定 Hive 文件的分隔符 \x01 为 "\\x01"

file_type:

用于指定导入文件的类型。目前支持的文件类型有 csv、orc 和 parquet。

column_list:

用于指定导入文件中的列与表中列的对应关系。
当需要跳过导入文件中的某列时，将该列指定为表中不存在的列名。

语法：
(col_name1, col_name2, ...)

SET:

如果指定了此参数，可以根据函数转换源文件中的列，然后将转换结果导入表中。语法为 column_name = expression。
仅支持 Spark SQL 内置函数。请参考 https://spark.apache.org/docs/2.4.6/api/sql/index.html。
举几个例子以帮助理解。
示例 1：表中有三列 "c1, c2, c3"，源文件中的前两列对应 (c1, c2)，最后两列的和对应 C3；则需要指定列 (c1, c2, tmp_c3, tmp_c4) SET (c3 = tmp_c3 + tmp_c4)；
示例 2：表中有三列 "year, month, day"，源文件中只有一个时间列，格式为 "2018-06-01 01:02:03"。
则可以指定列 (tmp_time) SET (year = year(tmp_time), month = month(tmp_time), day = day(tmp_time)) 来完成导入。

WHERE:

过滤转换后的数据，只有满足 where 条件的数据才能被导入。WHERE 语句中只能引用表中的列名
```

3.resource_name

可以通过 `SHOW RESOURCES` 命令查看使用的 Spark 资源名称。

4.resource_properties

当您有临时需求，例如修改 Spark 和 HDFS 配置时，可以在此处设置参数，该参数仅在该特定的 Spark 加载作业中生效，不会影响 StarRocks 集群中的现有配置。

5.opt_properties

用于指定一些特殊参数。

语法：

```sql
[PROPERTIES ("key"="value", ...)]
```

您可以指定以下参数：
timeout：指定导入操作的超时时间。默认超时为 4 小时。单位为秒。
max_filter_ratio：允许过滤的最大数据比例（由于非标准数据等原因）。默认为零容忍。
strict_mode：是否严格限制数据。默认为 false。
timezone：指定一些受时区影响的函数的时区，如 strftime/alignment_timestamp/from_unixtime 等，请参考 [时区](../time-zone/time-zone.md) 文档。如果未指定，默认使用 "Asia/Shanghai" 时区。

6.导入数据格式示例

int (TINYINT/SMALLINT/INT/BIGINT/LARGEINT): 1, 1000, 1234
float (FLOAT/DOUBLE/DECIMAL): 1.1, 0.23, .356
date (DATE/DATETIME): 2017-10-03, 2017-06-13 12:34:03
（注：对于其他日期格式，可以在导入命令中使用 strftime 或 time_format 函数进行转换）
字符串类 (CHAR/VARCHAR): "I am a student", "a"

NULL 值：\N

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

   其中 hdfs_host 是 NameNode 的主机，hdfs_port 是 fs.defaultfs 端口（默认为 9000）

2. 从 HDFS 导入一批“负数”数据，指定分隔符为逗号，使用通配符 * 指定目录下的所有文件，并指定 Spark 资源的临时参数。

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

3. 从 HDFS 导入一批数据，指定分区，并对导入文件的列进行一些转换，如下：

   ```plain
   表结构为：
   k1 varchar(20)
   k2 int
   
   假设数据文件只有一行数据：
   
   Adele,1,1
   
   数据文件中的每列对应导入语句中指定的每列：
   k1, tmp_k2, tmp_k3
   
   转换如下：
   
   1. k1：无转换
   2. k2：是 tmp_k2 和 tmp_k3 的和
   
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

4. 提取文件路径中的分区字段

   如有必要，将根据表中定义的字段类型解析文件路径中的分区字段，类似于 Spark 中的 Partition Discovery 功能

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

   `hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing` 目录包含以下文件：

   `[hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv, hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv, ...]`

   提取文件路径中的 city 和 utc_date 字段

5. 过滤要导入的数据。仅可导入 k1 值大于 10 的列。

   ```sql
   LOAD LABEL example_db.label10
   (
   DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
   INTO TABLE `my_table`
   WHERE k1 > 10
   )
   WITH RESOURCE 'my_spark';
   ```

6. 从 Hive 外部表导入，并通过全局字典将源表中的 uuid 列转换为 bitmap 类型。

   ```sql
   LOAD LABEL db1.label1
   (
   DATA FROM TABLE hive_t1
   INTO TABLE tbl1
   SET
   (
   uuid = bitmap_dict(uuid)
   )
   )
   WITH RESOURCE 'my_spark';
   ```