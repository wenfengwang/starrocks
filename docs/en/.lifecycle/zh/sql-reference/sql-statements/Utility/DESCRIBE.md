---
displayed_sidebar: English
---

# DESC

## 描述

您可以使用此语句执行以下操作：

- 查看存储在 StarRocks 集群中的表的架构，包括表的[排序键](../../../table_design/Sort_key.md)和[物化视图](../../../using_starrocks/Materialized_view.md)类型。
- 查看存储在如 Apache Hive™ 等外部数据源中的表的架构。请注意，此操作仅在 StarRocks 2.4 及更高版本中支持。

## 语法

```SQL
DESC[RIBE] [catalog_name.][db_name.]table_name [ALL];
```

## 参数

|**参数**|**是否必填**|**描述**|
|---|---|---|
|catalog_name|否|内部目录或外部目录的名称。<ul><li>如果将此参数值设置为内部目录的名称，即 `default_catalog`，则可以查看存储在 StarRocks 集群中的表的架构。</li><li>如果将此参数值设置为外部目录的名称，则可以查看存储在外部数据源中的表的架构。</li></ul>|
|db_name|否|数据库名称。|
|table_name|是|表名称。|
|ALL|否|<ul><li>如果指定此关键字，则可以查看存储在 StarRocks 集群中的表的排序键类型、物化视图和架构。如果未指定此关键字，则只能查看表架构。</li><li>查看存储在外部数据源中的表的架构时，不应指定此关键字。</li></ul>|

## 输出

```Plain
+-----------+---------------+-------+------+------+-----+---------+-------+
| IndexName | IndexKeysType | Field | Type | Null | Key | Default | Extra |
+-----------+---------------+-------+------+------+-----+---------+-------+
```

下表描述了此语句返回的参数。

|**参数**|**描述**|
|---|---|
|IndexName|表名称。查看存储在外部数据源中的表的架构时，不返回此参数。|
|IndexKeysType|表的排序键类型。查看存储在外部数据源中的表的架构时，不返回此参数。|
|Field|列名称。|
|Type|列的数据类型。|
|Null|列值是否可以为 NULL。<ul><li>`yes`：表示值可以为 NULL。</li><li>`no`：表示值不能为 NULL。</li></ul>|
|Key|列是否用作排序键。<ul><li>`true`：表示列用作排序键。</li><li>`false`：表示列不用作排序键。</li></ul>|
|Default|列的数据类型的默认值。如果数据类型没有默认值，则返回 NULL。|
|Extra|<ul><li>查看存储在 StarRocks 集群中的表的架构时，此字段显示列的相关信息：<ul><li>列使用的聚合函数，如 `SUM` 和 `MIN`。</li><li>列上是否创建了布隆过滤器索引。如果是，`Extra` 的值为 `BLOOM_FILTER`。</li></ul></li><li>查看存储在外部数据源中的表的架构时，此字段显示列是否为分区列。如果列是分区列，`Extra` 的值为 `partition key`。</li></ul>|

> 注意：关于物化视图在输出中的显示信息，请参见示例 2。

## 示例

示例 1：查看存储在 StarRocks 集群中的 `example_table` 表的架构。

```SQL
DESC example_table;
```

或

```SQL
DESC default_catalog.example_db.example_table;
```

上述语句的输出如下。

```Plain
+-------+---------------+------+-------+---------+-------+
| Field | Type          | Null | Key   | Default | Extra |
+-------+---------------+------+-------+---------+-------+
| k1    | TINYINT       | Yes  | true  | NULL    |       |
| k2    | DECIMAL(10,2) | Yes  | true  | 10.5    |       |
| k3    | CHAR(10)      | Yes  | false | NULL    |       |
| v1    | INT           | Yes  | false | NULL    |       |
+-------+---------------+------+-------+---------+-------+
```

示例 2：查看存储在 StarRocks 集群中的 `sales_records` 表的架构、排序键类型和物化视图。在以下示例中，基于 `sales_records` 创建了一个物化视图 `store_amt`。

```Plain
DESC db1.sales_records ALL;

+---------------+---------------+-----------+--------+------+-------+---------+-------+
| IndexName     | IndexKeysType | Field     | Type   | Null | Key   | Default | Extra |
+---------------+---------------+-----------+--------+------+-------+---------+-------+
| sales_records | DUP_KEYS      | record_id | INT    | Yes  | true  | NULL    |       |
|               |               | seller_id | INT    | Yes  | true  | NULL    |       |
|               |               | store_id  | INT    | Yes  | true  | NULL    |       |
|               |               | sale_date | DATE   | Yes  | false | NULL    | NONE  |
|               |               | sale_amt  | BIGINT | Yes  | false | NULL    | NONE  |
|               |               |           |        |      |       |         |       |
| store_amt     | AGG_KEYS      | store_id  | INT    | Yes  | true  | NULL    |       |
|               |               | sale_amt  | BIGINT | Yes  | false | NULL    | SUM   |
+---------------+---------------+-----------+--------+------+-------+---------+-------+
```

示例 3：查看存储在 Hive 集群中的 `hive_table` 表的架构。

```Plain
DESC hive_catalog.hive_db.hive_table;

+-------+----------------+------+-------+---------+---------------+ 
| Field | Type           | Null | Key   | Default | Extra         | 
+-------+----------------+------+-------+---------+---------------+ 
| id    | INT            | Yes  | false | NULL    |               | 
| name  | VARCHAR(65533) | Yes  | false | NULL    |               | 
| date  | DATE           | Yes  | false | NULL    | partition key | 
+-------+----------------+------+-------+---------+---------------+
```

## 参考资料

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)