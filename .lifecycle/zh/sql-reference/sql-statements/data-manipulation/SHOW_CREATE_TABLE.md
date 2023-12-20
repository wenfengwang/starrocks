---
displayed_sidebar: English
---

# 显示创建表命令

返回创建指定表的 CREATE TABLE 语句。

> **注意**
> 在 v3.0 之前的版本中，执行 `SHOW CREATE TABLE` 命令需要您对该表拥有 `SELECT_PRIV` 权限。从 v3.0 开始，执行 `SHOW CREATE TABLE` 命令需要您对该表拥有 `SELECT` 权限。

从 v3.0 版本开始，您可以使用 SHOW CREATE TABLE 命令查看由外部目录管理并存储在 Apache Hive™、Apache Iceberg、Apache Hudi 或 Delta Lake 中的表的 CREATE TABLE 语句。

从 v2.5.7 版本开始，StarRocks 在创建表或添加分区时可以自动设置桶数（BUCKETS）。您不再需要手动设置桶数。更多详细信息，请参见[Determine the number of buckets](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)。

- 如果在创建表时指定了桶数，SHOW CREATE TABLE 的输出将会显示桶数。
- 如果在创建表时没有指定桶数，SHOW CREATE TABLE 的输出将不会显示桶数。您可以执行 [SHOW PARTITIONS](SHOW_PARTITIONS.md) 命令来查看每个分区的桶数。

在 v2.5.7 之前的版本中，创建表时必须设置桶数。因此，SHOW CREATE TABLE 默认会显示桶数。

## 语法

```SQL
SHOW CREATE TABLE [db_name.]table_name
```

## 参数

|参数|必填|说明|
|---|---|---|
|db_name|否|数据库名称。如果未指定此参数，则默认返回当前数据库中给定表的 CREATE TABLE 语句。|
|table_name|是|表名称。|

## 输出

```Plain
+-----------+----------------+
| Table     | Create Table   |                                               
+-----------+----------------+
```

下表描述了此语句返回的参数。

|参数|说明|
|---|---|
|表|表名称。|
|创建表|表的CREATE TABLE 语句。|

## 示例

### 未指定桶数

在 DISTRIBUTED BY 中未指定桶数，创建名为 example_table 的表。

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

执行 SHOW CREATE TABLE 命令，显示 example_table 的 CREATE TABLE 语句。DISTRIBUTED BY 中不会显示桶数。请注意，如果在创建表时未指定 PROPERTIES，那么默认属性会在 SHOW CREATE TABLE 的输出中显示。

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

### 指定桶数

在 DISTRIBUTED BY 中设置桶数为 10，创建名为 example_table1 的表。

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

执行 SHOW CREATE TABLE 命令，显示 example_table1 的 CREATE TABLE 语句。DISTRIBUTED BY 中会显示桶数（BUCKETS 10）。请注意，如果在创建表时未指定 PROPERTIES，那么默认属性会在 SHOW CREATE TABLE 的输出中显示。

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

## 参考资料

- [创建表](../data-definition/CREATE_TABLE.md)
- [显示表格](../data-manipulation/SHOW_TABLES.md)
- [修改表](../data-definition/ALTER_TABLE.md)
- [DROP TABLE](../data-definition/DROP_TABLE.md)
