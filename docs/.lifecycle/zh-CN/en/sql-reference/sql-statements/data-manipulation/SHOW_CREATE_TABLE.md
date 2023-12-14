---
displayed_sidebar: "Chinese"
---

# 显示创建表

返回用于创建给定表的CREATE TABLE语句。

> **注意**
>
> 在v3.0之前的版本中，SHOW CREATE TABLE语句要求你对该表具有`SELECT_PRIV`权限。从v3.0开始，SHOW CREATE TABLE语句要求你对该表具有`SELECT`权限。

从v3.0开始，您可以使用SHOW CREATE TABLE语句查看由外部目录管理并存储在Apache Hive™、Apache Iceberg、Apache Hudi或Delta Lake中的表的CREATE TABLE语句。

从v2.5.7开始，StarRocks在创建表或添加分区时可以自动设置桶的数量（BUCKETS）。您无需手动设置桶的数量。有关详细信息，请参阅[确定桶的数量](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)。

- 如果在创建表时指定了桶的数量，则SHOW CREATE TABLE的输出将显示桶的数量。
- 如果在创建表时未指定桶的数量，则SHOW CREATE TABLE的输出将不显示桶的数量。您可以运行[SHOW PARTITIONS](SHOW_PARTITIONS.md)来查看每个分区的桶数量。

在v2.5.7之前的版本中，创建表时需要设置桶的数量。因此，SHOW CREATE TABLE默认情况下将显示桶的数量。

## 语法

```SQL
显示创建表 [db_name.]table_name
```

## 参数

| **参数**    | **是否必需** | **描述**                                                     |
| ----------- | ------------ | ------------------------------------------------------------ |
| db_name     | 否           | 数据库名称。如果未指定此参数，则默认返回当前数据库中给定表的CREATE TABLE语句。 |
| table_name  | 是           | 表名称。                                                      |

## 输出

```Plain
+-----------+----------------+
| 表        | 创建表         |                                               
+-----------+----------------+
```

以下表格描述了该语句返回的参数。

| **参数**     | **描述**              |
| ------------ | --------------------- |
| 表           | 表名称                |
| 创建表      | 表的CREATE TABLE语句 |

## 示例

### 未指定桶的数量

在DISTRIBUTED BY中未指定桶的数量，创建名为`example_table`的表。

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

运行SHOW CREATE TABLE来显示`example_table`的CREATE TABLE语句。DISTRIBUTED BY中不显示桶的数量。请注意，如果在创建表时未指定PROPERTIES，则在SHOW CREATE TABLE的输出中显示默认属性。

```Plain
SHOW CREATE TABLE example_table\G
*************************** 1. row ***************************
       表: example_table
创建表: CREATE TABLE `example_table` (
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

### 指定了桶的数量

在DISTRIBUTED BY中将桶的数量设置为10，创建名为`example_table1`的表。

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

运行SHOW CREATE TABLE来显示`example_table1`的CREATE TABLE语句。在DISTRIBUTED BY中显示桶的数量（`BUCKETS 10`）。请注意，如果在创建表时未指定PROPERTIES，则在SHOW CREATE TABLE的输出中显示默认属性。

```plain
SHOW CREATE TABLE example_table1\G
*************************** 1. row ***************************
       表: example_table1
创建表: CREATE TABLE `example_table1` (
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

## 参考

- [CREATE TABLE](../data-definition/CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [ALTER TABLE](../data-definition/ALTER_TABLE.md)
- [DROP TABLE](../data-definition/DROP_TABLE.md)