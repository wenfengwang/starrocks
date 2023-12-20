---
displayed_sidebar: English
---

# DESC

## 描述

您可以使用此语句执行以下操作：

- 查看存储在您的 StarRocks 集群中的表架构，以及表的[排序键](../../../table_design/Sort_key.md)类型和[物化视图](../../../using_starrocks/Materialized_view.md)。
- 查看存储在以下外部数据源中的表架构，例如 Apache Hive™。请注意，此操作只能在 StarRocks 2.4 及更高版本中执行。

## 语法

```SQL
DESC[RIBE] [catalog_name.][db_name.]table_name [ALL];
```

## 参数

|参数|必填|说明|
|---|---|---|
| catalog_name |否|内部目录或外部目录的名称。如果将该参数的值设置为内部目录的名称（default_catalog），您可以查看存储在 StarRocks 集群中的表的架构。如果将该参数的值设置为外部目录的名称，则可以查看外部数据源中存储的表的架构。|
|db_name|否|数据库名称。|
|table_name|是|表名称。|
|ALL|否|如果指定该关键字，您可以查看存储在 StarRocks 集群中的表的排序键类型、物化视图和架构。如果未指定此关键字，则仅查看表模式。当您查看存储在外部数据源中的表的架构时，请勿指定此关键字。|

## 输出

```Plain
+-----------+---------------+-------+------+------+-----+---------+-------+
| IndexName | IndexKeysType | Field | Type | Null | Key | Default | Extra |
+-----------+---------------+-------+------+------+-----+---------+-------+
```

下表描述了此语句返回的参数。

|参数|说明|
|---|---|
|IndexName|表名称。如果您查看存储在外部数据源中的表的架构，则不会返回此参数。|
| IndexKeyStype |表的排序键的类型。如果您查看存储在外部数据源中的表的架构，则不会返回此参数。|
|字段|列名。|
|类型|列的数据类型。|
| null |列值是否可以为null。 yes：表示该值可以为NULL。 no：表示值不能为NULL。 |
|Key|该列是否用作排序键。 true：表示该列用作排序键。 false：表示该列不作为排序键。 |
|默认值|列的数据类型的默认值。如果数据类型没有默认值，则将返回空。|
| Extra |如果您看到了Starrocks群集中存储的表格的架构，则此字段显示有关列的以下信息：列使用的汇总函数，例如sum和min。是否在列上创建布隆过滤器索引。如果是，则 Extra 的值为 BLOOM_FILTER。如果您看到存储在外部数据源中的表的架构，此字段将显示该列是否为分区列。如果该列是分区列，则Extra的值为分区键。 |

> 注意：有关物化视图在输出中的显示信息，请参见示例 2。

## 示例

示例 1：查看存储在您的 StarRocks 集群中的 example_table 表的架构。

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

示例 2：查看存储在您的 StarRocks 集群中的 sales_records 表的架构、排序键类型和物化视图。在以下示例中，基于 sales_records 创建了一个物化视图 store_amt。

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

示例 3：查看存储在您的 Hive 集群中的 hive_table 表的架构。

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

- [创建数据库](../data-definition/CREATE_DATABASE.md)
- [显示创建数据库](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [使用](../data-definition/USE.md)
- [显示数据库](../data-manipulation/SHOW_DATABASES.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)
