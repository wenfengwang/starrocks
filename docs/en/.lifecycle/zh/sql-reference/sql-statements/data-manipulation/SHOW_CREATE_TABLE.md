---
displayed_sidebar: English
---

# SHOW CREATE TABLE

返回用于创建给定表的 CREATE TABLE 语句。

> **注意**
> 在 v3.0 之前的版本中，SHOW CREATE TABLE 语句要求您拥有表的 `SELECT_PRIV` 权限。从 v3.0 开始，SHOW CREATE TABLE 语句要求您拥有表的 `SELECT` 权限。

从 v3.0 开始，您可以使用 SHOW CREATE TABLE 语句查看由外部目录管理并存储在 Apache Hive™、Apache Iceberg、Apache Hudi 或 Delta Lake 中的表的 CREATE TABLE 语句。

从 v2.5.7 开始，StarRocks 可以在创建表或添加分区时自动设置桶数（BUCKETS）。您不再需要手动设置桶数。有关详细信息，请参阅[确定桶数](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)。

- 如果创建表时指定了桶数，SHOW CREATE TABLE 的输出将显示桶数。
- 如果创建表时没有指定桶数，SHOW CREATE TABLE 的输出不会显示桶数。您可以运行 [SHOW PARTITIONS](SHOW_PARTITIONS.md) 查看每个分区的桶数。

在 v2.5.7 之前的版本中，创建表时需要设置桶数。因此，SHOW CREATE TABLE 默认显示桶数。

## 语法

```SQL
SHOW CREATE TABLE [db_name.]table_name
```

## 参数

|**参数**|**是否必须**|**描述**|
|---|---|---|
|db_name|否|数据库名称。如果未指定此参数，则默认返回当前数据库中给定表的 CREATE TABLE 语句。|
|table_name|是|表名称。|

## 输出

```Plain
+-----------+----------------+
| Table     | Create Table   |                                               
+-----------+----------------+
```

下表描述了该语句返回的参数。

|**参数**|**描述**|
|---|---|
|Table|表名称。|
|Create Table|表的 CREATE TABLE 语句。|

## 示例

### 未指定桶数

创建名为 `example_table` 的表，且 DISTRIBUTED BY 中未指定桶数。

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

执行 SHOW CREATE TABLE 命令，可以看到 `example_table` 的 CREATE TABLE 语句。DISTRIBUTED BY 中没有显示桶数。请注意，如果您在创建表时未指定 PROPERTIES，则默认属性将显示在 SHOW CREATE TABLE 的输出中。

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

创建名为 `example_table1` 的表，并将 DISTRIBUTED BY 中的桶数设置为 10。

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

执行 SHOW CREATE TABLE 命令，可以看到 `example_table1` 的 CREATE TABLE 语句。桶数 (`BUCKETS 10`) 显示在 DISTRIBUTED BY 中。请注意，如果您在创建表时未指定 PROPERTIES，则默认属性将显示在 SHOW CREATE TABLE 的输出中。

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

- [CREATE TABLE](../data-definition/CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [ALTER TABLE](../data-definition/ALTER_TABLE.md)
- [DROP TABLE](../data-definition/DROP_TABLE.md)