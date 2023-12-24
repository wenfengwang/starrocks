---
displayed_sidebar: English
---

# 火花加载

## 描述

Spark Load 通过外部 Spark 资源对导入的数据进行预处理，提高了大量 StarRocks 数据的导入性能，并节省了 StarRocks 集群的计算资源。主要用于初次迁移和大量数据导入 StarRocks 的场景。

Spark Load 是一种异步导入方法。用户需要通过 MySQL 协议创建 Spark 类型的导入任务，并通过`SHOW LOAD`查看导入结果。

> **注意**
>
> - 只有对 StarRocks 表具有 INSERT 权限的用户才能将数据加载到 StarRocks 表中。如果没有 INSERT 权限，请按照 [GRANT](../account-management/GRANT.md) 中的说明，为连接到 StarRocks 集群的用户授予 INSERT 权限。
> - 使用 Spark Load 将数据加载到 StarRocks 表时，StarRocks 表的分桶列不能是 DATE、DATETIME 或 DECIMAL 类型。

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

当前导入批次的标签，在数据库中是唯一的。

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

注意

```plain text
file_path:

文件路径可以指定为一个文件，也可以使用 * 通配符指定目录中的所有文件。通配符必须匹配文件，而不是目录。

hive_external_tbl:

hive 外部表名称。
导入到 starrocks 表中的列必须存在于 hive 外部表中。
每个加载任务仅支持从一个 Hive 外部表加载。
不能与 file_ path 模式同时使用。

PARTITION:

如果指定了此参数，则只导入指定的分区，并过滤掉导入分区之外的数据。
如果未指定，默认情况下将导入表的所有分区。

NEGATIVE:

如果指定了此参数，相当于加载一批“负”数据。用于抵消先前导入数据的同一批次。
此参数仅适用于值列存在且值列的聚合类型仅为 SUM 时。

column_separator:

指定导入文件中的列分隔符。默认为 \ t
如果是不可见字符，则需要使用 \ \ x 前缀，并使用十六进制表示分隔符。
例如，hive 文件的分隔符 \ x01 被指定为 "\ \ x01"

file_type:

用于指定导入文件的类型。当前支持的文件类型为 csv、orc 和 parquet。

column_list:

用于指定导入文件中的列与表中的列之间的对应关系。
当需要跳过导入文件中的某一列时，将该列指定为表中不存在的列名。

语法：
(col_name1, col_name2, ...)

SET:

如果指定此参数，可以根据函数转换源文件的列，然后将转换后的结果导入表中。语法为 column_name = expression。
仅支持 Spark SQL 内置函数。请参考 https://spark.apache.org/docs/2.4.6/api/sql/index.html。
给出一些示例以帮助理解。
示例 1：表中有三列“c1、c2、c3”，源文件中的前两列对应于（c1、c2），最后两列的和对应于 C3；然后需要指定列（c1、c2、tmp_c3、tmp_c4）set (c3 = tmp_c3 + tmp_c4)；
示例 2：表中有三列“year、month 和 day”，源文件中只有一个时间列，格式为“2018-06-01 01:02:03”。
然后可以指定列（tmp_time）set (year = year (tmp_time), month = month (tmp_time), day = day (tmp_time)) 完成导入。

WHERE:

过滤转换后的数据，只有满足 WHERE 条件的数据才能被导入。WHERE 语句中只能引用表中的列名。
```

3.resource_name

使用的 spark 资源的名称可以通过`SHOW RESOURCES`命令查看。

4.resource_properties

当您有临时需求时，例如修改 Spark 和 HDFS 配置，可以在此处设置参数，该参数仅在该特定的 Spark 加载作业中生效，不会影响 StarRocks 集群中的现有配置。

5.opt_properties

用于指定一些特殊参数。

语法：

```sql
[PROPERTIES ("key"="value", ...)]
```

您可以指定以下参数：
timeout：指定导入操作的超时时间。默认超时为 4 小时。以秒为单位。
max_filter_ratio：可过滤的最大允许数据占比（例如由于非标准数据等原因）。默认为零容忍。
strict mode：是否严格限制数据。默认为 false。
timezone：指定部分受时区影响的函数的时区，例如 strftime/alignment_timestamp/from_unixtime 等。有关详细信息，请参阅[时区]文档。如果未指定，则使用“亚洲/上海”时区。

6.导入数据格式示例

int（TINYINT/SMALLINT/INT/BIGINT/LARGEINT）：1、1000、1234
float（FLOAT/DOUBLE/DECIMAL）：1.1、0.23、.356
date（DATE/DATETIME）：2017-10-03、2017-06-13 12:34:03。
（注意：对于其他日期格式，可以使用 strftime 或 time_format 函数在导入命令中进行转换）string 类（CHAR/VARCHAR）：“我是学生”，“a”

NULL 值：\N

## 例子

1. 从 HDFS 导入一批数据，并指定超时时间和过滤比例。使用名称 my_spark 的 spark 资源。

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

    其中 hdfs_host 是 namenode 的主机，hdfs_port 是 fs.defaultfs 端口（默认为 9000）

2. 从 HDFS 导入一批“负”数据，指定分隔符为逗号，使用通配符 * 指定目录中的所有文件，并指定 spark 资源的临时参数。

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

3. 从 HDFS 导入一批数据，指定分区，并对导入文件的列进行一些转换，如下所示：

    ```plain text
    表结构为：
    k1 varchar(20)
    k2 int
    
    假设数据文件只有一行数据：
    
    Adele,1,1
    
    数据文件中的每一列都对应导入语句中指定的每一列：
    k1,tmp_k2,tmp_k3
    
    转换如下：
    
    1. k1：无需转换
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

    如有必要，文件路径中的分区字段会根据表中定义的字段类型进行解析，类似于 Spark 中的分区发现功能

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

    `hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing`  该目录包含以下文件：

    `[hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv, hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv, ...]`

    提取文件路径中的 city 和 utc_date 字段

5. 筛选要导入的数据。只有 k1 值大于 10 的列才能被导入。

    ```sql
    LOAD LABEL example_db.label10
    (
    DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
    INTO TABLE `my_table`
    WHERE k1 > 10
    )
    WITH RESOURCE 'my_spark';
    ```

6. 从 hive 外部表导入，并通过全局字典将源表中的 uuid 列转换为位图类型。

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
