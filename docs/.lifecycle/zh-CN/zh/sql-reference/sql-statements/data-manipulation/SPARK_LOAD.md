---
displayed_sidebar: "Chinese"
---

# SPARK LOAD

## 功能

Spark load 通过外部的 Spark 资源实现对导入数据的预处理，提高 StarRocks 大数据量的导入性能并且节省 StarRocks 集群的计算资源。主要用于初次迁移，大数据量导入 StarRocks 的场景。

Spark load 是一种异步导入方式，用户需要通过 MySQL 协议创建 Spark 类型导入任务，并通过 `SHOW LOAD` 查看导入结果。

> **注意**
>
> - Spark Load 操作需要目标表的 INSERT 权限。如果您的用户账号没有 INSERT 权限，请参考 [GRANT](../account-management/GRANT.md) 给用户赋权。
> - 使用 Spark Load 导入数据至 StarRocks 表时，不支持该表分桶列的数据类型为 DATE、DATETIME 或者 DECIMAL。

## 语法

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

当前导入批次的标签。在一个 database 内唯一。

语法：

```sql
[database_name.]your_label
```

2.data_desc

用于描述一批导入数据。

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
......(omitted)

```

### 从HDFS导入数据到指定分区并进行列转换

从 HDFS 导入一批数据，指定分区，并对导入文件的列做一些转化，如下：

```plain text
表结构为：
k1 varchar(20)
k2 int

假设数据文件只有一行数据：

Adele,1,1

数据文件中各列，对应导入语句中指定的各列：
k1,tmp_k2,tmp_k3

转换如下：

1. k1: 不变换
2. k2：是tmp_k2和tmp_k3数据之和

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

### 提取文件路径中的分区字段

如果需要，则会根据表中定义的字段类型解析文件路径中的分区字段（partitioned fields），类似 Spark 中 Partition Discovery 的功能。

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

`hdfs://hdfs_host: hdfs_port/user/starRocks/data/input/dir/city = beijing` 目录下包括如下文件：

`[hdfs://hdfs_host: hdfs_port/user/starRocks/data/input/dir/city = beijing/utc_date = 2019-06-26/0000.csv, hdfs://hdfs_host: hdfs_port/user/starRocks/data/input/dir/city = beijing/utc_date = 2019-06-26/0001.csv, ...]`

则提取文件路径的中的 `city` 和 `utc_date` 字段。

### 对导入数据进行过滤

对待导入数据进行过滤，k1 值大于 10 的列才能被导入。

```sql
LOAD LABEL example_db.label10
(
DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
INTO TABLE `my_table`
WHERE k1 > 10
)
WITH RESOURCE 'my_spark';
```

### 从 Hive 外表导入并构建全局字典

从 hive 外部表导入，并将源表中的 uuid 列通过全局字典转化为 bitmap 类型。

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