---
displayed_sidebar: "Chinese"
---

# Spark Load

## 描述

Spark Load通过外部spark资源对导入的数据进行预处理，提高了大量StarRocks数据的导入性能，并节约了StarRocks集群的计算资源。它主要用于初始迁移和大量数据导入到StarRocks的场景中。

Spark Load是一种异步导入方法。用户需要通过MySQL协议创建Spark类型的导入任务，并通过`SHOW LOAD`查看导入结果。

> **注意**
>
> - 您只能以在拥有INSERT表权限的用户身份将数据加载到StarRocks表中。如果您没有INSERT权限，请按照[GRANT](../account-management/GRANT.md)中提供的说明，将INSERT权限授予您用于连接到StarRocks集群的用户。
> - 当使用Spark Load加载数据到StarRocks表时，StarRocks表的分桶列不能是DATE、DATETIME或DECIMAL类型。

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

当前导入批次的标签。在数据库内唯一。

语法:

```sql
[database_name.]your_label
```

2.data_desc

用于描述导入数据批次。

语法:

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

可以将文件路径指定为一个文件，或者可以使用通配符*来指定目录中的所有文件。通配符必须匹配文件，不能匹配目录。

hive_external_tbl:

hive外部表名称。
导入的StarRocks表中必须存在于hive外部表中的列。
每个加载任务只支持从一个Hive外部表加载数据。
不能与file_ path模式同时使用。

PARTITION:

如果指定了此参数，只导入指定的分区，并过滤掉导入分区之外的数据。
如果未指定，默认将导入表的所有分区。

NEGATIVE:

如果指定了此参数，相当于加载一批“负面”数据。用于抵消先前导入数据的同一批次。
此参数仅在值列存在且值列的聚合类型仅为SUM时适用。

column_separator:

指定导入文件中的列分隔符。默认为\ t
如果是不可见字符，需要使用\ \ x作为前缀，并用十六进制表示分隔符。
例如，hive文件\ x01的分隔符指定为"\ x01"

file_type:

用于指定导入文件的类型。当前支持的文件类型为csv、orc和parquet。

column_list:

用于指定导入文件中的列与表中的列的对应关系。
当需要跳过导入文件中的列时，指定该列为表中不存在的列名。

语法:
(col_name1, col_name2, ...)

SET:

如果指定此参数，可以根据函数转换源文件的列，然后将转换后的结果导入到表中。语法为column_name = expression。
仅支持Spark SQL内置函数。请参阅https://spark.apache.org/docs/2.4.6/api/sql/index.html。
给出一些示例以帮助理解。
示例1:表中有三列“c1、c2、c3”，源文件的前两列对应于(c1、c2)，最后两列的和对应C3；然后需要指定(columns (c1、c2、tmp_c3、tmp_c4) set (c3 = tmp_c3 + tmp_c4))；
示例2:表中有三列“year、month和day”，源文件中只有一个格式为“2018-06-01 01:02:03”的时间列。
然后可以指定(columns (tmp_time) set (year = year(tmp_time), month = month(tmp_time), day = day(tmp_time))来完成导入。

WHERE:

过滤转换后的数据，只有满足where条件的数据才能被导入。在WHERE语句中只能引用表中的列名
```

3.resource_name

所使用的spark资源的名称可以通过`SHOW RESOURCES`命令进行查看。

4.resource_properties

当您有临时需求，比如修改Spark和HDFS配置时，可以在这里设置参数，这仅在此特定spark加载作业中生效，并不会影响StarRocks集群中的现有配置。

5.opt_properties

用于指定一些特殊参数。

语法:

```sql
[PROPERTIES ("key"="value", ...)]
```

您可以指定以下参数:
timeout:         指定导入操作的超时时间。默认超时时间为4小时。单位为秒。
max_filter_ratio:可以过滤的最大允许数据比例(由于非标准数据等原因)。默认为零容差。
strict mode:     是否严格限制数据。默认为false。
timezone:         指定受时区影响的某些函数的时区，例如strftime/alignment_timestamp/from_unixtime等。有关详情，请参阅[时区] 文档。如果未指定，将使用“Asia / Shanghai”时区。

6.导入数据格式示例

int (TINYINT/SMALLINT/INT/BIGINT/LARGEINT): 1、1000、1234
float (FLOAT/DOUBLE/DECIMAL): 1.1、0.23、.356
date (DATE/DATETIME) :2017-10-03、2017-06-13 12:34:03。
（注意：对于其他日期格式，您可以使用strftime或time_format函数在导入命令中进行转换）字符串类（CHAR/VARCHAR）: "I am a student"、"a"

NULL value: \ N

## 示例

1. 从HDFS导入一批数据，并指定超时时间和过滤比例。使用名为my_ spark资源的spark。

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

    这里hdfs_host是namenode的主机，hdfs_port是fs.defaultfs端口(默认9000)

2. 从HDFS导入一批“负面”数据，指定分隔符为逗号，使用通配符*指定目录中的所有文件，并指定spark资源的临时参数。

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

3. 从HDFS导入一批数据，指定分区，并对导入文件的列进行一些转换，如下：

    ```plain text
    表结构如下:
    k1 varchar(20)
    k2 int
    
    假设数据文件中只有一行数据:
    
    Adele,1,1
    
    数据文件中的每一列分别对应导入语句中指定的每一列:
    k1,tmp_k2,tmp_k3
    
    转换规则如下:
    
    1. k1: 无转换
    2. k2: tmp_k2和tmp_k3的和
    
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

如果有必要，文件路径中的分区字段将根据表中定义的字段类型进行解析，类似于Spark中的Partition Discovery功能

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

`hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing` 该目录包括以下文件：

`[hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv, hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv, ...]`

文件路径中的city和utc_date字段将被提取

5. 过滤要导入的数据。只能导入k1值大于10的列。

```sql
LOAD LABEL example_db.label10
(
DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
INTO TABLE `my_table`
WHERE k1 > 10
)
WITH RESOURCE 'my_spark';
```

6. 从Hive外部表导入，并通过全局字典将源表中的uuid列转换为位图类型。

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