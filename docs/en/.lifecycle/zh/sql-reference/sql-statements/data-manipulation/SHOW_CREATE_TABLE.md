---
displayed_sidebar: English
---

# 显示创建表

返回用于创建给定表的 CREATE TABLE 语句。

> **注意**
>
> 在 v3.0 之前的版本中，SHOW CREATE TABLE 语句要求您对表具有 `SELECT_PRIV` 权限。从 v3.0 开始，SHOW CREATE TABLE 语句要求您具有对表的 `SELECT` 权限。

从 v3.0 开始，您可以使用 SHOW CREATE TABLE 语句查看由外部目录管理并存储在 Apache Hive™、Apache Iceberg、Apache Hudi 或 Delta Lake 中的表的 CREATE TABLE 语句。

从 v2.5.7 开始，StarRocks 可以在创建表或添加分区时自动设置存储桶（BUCKETS）的数量。您不再需要手动设置存储桶的数量。有关详细信息，请参阅 [确定存储桶数量](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)。

- 如果在创建表时指定了存储桶的数量，则 SHOW CREATE TABLE 的输出将显示存储桶的数量。
- 如果在创建表时未指定存储桶的数量，则 SHOW CREATE TABLE 的输出将不会显示存储桶的数量。您可以运行 [SHOW PARTITIONS](SHOW_PARTITIONS.md) 来查看每个分区的存储桶数量。

在 v2.5.7 之前的版本中，创建表时需要设置存储桶的数量。因此，SHOW CREATE TABLE 默认显示存储桶的数量。

## 语法

```SQL
SHOW CREATE TABLE [db_name.]table_name
```

## 参数

| **参数** | **必填** | **描述**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       | 否           | 数据库名称。如果未指定此参数，则默认返回当前数据库中给定表的 CREATE TABLE 语句。 |
| table_name    | 是          | 表名。                                              |

## 输出

```Plain
+-----------+----------------+
| Table     | Create Table   |                                               
+-----------+----------------+
```

该语句返回的参数说明如下表所示。

| **参数** | **描述**                          |
| ------------- | ---------------------------------------- |
| Table         | 表名。                          |
| Create Table  | 表的 CREATE TABLE 语句。 |

## 例子

### 未指定存储桶编号

创建一个名为 `example_table` 的表，其中在 DISTRIBUTED BY 中未指定存储桶编号。

```SQL
CREATE TABLE example_table
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 CHAR(10) REPLACE,
    v2 INT SUM
)
ENGINE = olap
AGGREGATE KEY(k1, k2)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(k1);
```

运行 SHOW CREATE TABLE 来显示 `example_table` 的 CREATE TABLE 语句。在 DISTRIBUTED BY 中不显示存储桶编号。请注意，如果在创建表时未指定 PROPERTIES，则默认属性将显示在 SHOW CREATE TABLE 的输出中。

```Plain
SHOW CREATE TABLE example_table\G
*************************** 1. row ***************************
       Table: example_table
Create Table: CREATE TABLE `example_table` (
  `k1` tinyint(4) NULL COMMENT "",
  `k2` decimal64(10, 2) NULL DEFAULT "10.5" COMMENT "",
  `v1` char(10) REPLACE NULL COMMENT "",
  `v2` int(11) SUM NULL COMMENT ""
) ENGINE=OLAP 
AGGREGATE KEY(`k1`, `k2`)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(`k1`)
PROPERTIES (
"replication_num" = "3",
"in_memory" = "false",
"enable_persistent_index" = "false",
"replicated_storage" = "true",
"compression" = "LZ4"
);
```

### 指定了存储桶编号

创建一个名为 `example_table1` 的表，并在 DISTRIBUTED BY 中将存储桶编号设置为 10。

```SQL
CREATE TABLE example_table1
(
    k1 TINYINT,
    k2 DECIMAL(10, 2) DEFAULT "10.5",
    v1 CHAR(10) REPLACE,
    v2 INT SUM
)
ENGINE = olap
AGGREGATE KEY(k1, k2)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(k1) BUCKETS 10;
```

运行 SHOW CREATE TABLE 来显示 `example_table1` 的 CREATE TABLE 语句。存储桶编号 (`BUCKETS 10`) 显示在 DISTRIBUTED BY 中。请注意，如果在创建表时未指定 PROPERTIES，则默认属性将显示在 SHOW CREATE TABLE 的输出中。

```plain
SHOW CREATE TABLE example_table1\G
*************************** 1. row ***************************
       Table: example_table1
Create Table: CREATE TABLE `example_table1` (
  `k1` tinyint(4) NULL COMMENT "",
  `k2` decimal64(10, 2) NULL DEFAULT "10.5" COMMENT "",
  `v1` char(10) REPLACE NULL COMMENT "",
  `v2` int(11) SUM NULL COMMENT ""
) ENGINE=OLAP 
AGGREGATE KEY(`k1`, `k2`)
COMMENT "my first starrocks table"
DISTRIBUTED BY HASH(`k1`) BUCKETS 10 
PROPERTIES (
"replication_num" = "3",
"in_memory" = "false",
"enable_persistent_index" = "false",
"replicated_storage" = "true",
"compression" = "LZ4"
);
```

## 引用

- [CREATE TABLE](../data-definition/CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [ALTER TABLE](../data-definition/ALTER_TABLE.md)
- [DROP TABLE](../data-definition/DROP_TABLE.md)
