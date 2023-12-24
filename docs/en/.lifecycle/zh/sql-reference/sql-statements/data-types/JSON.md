---
displayed_sidebar: English
---

# JSON

StarRocks 从 v2.2.0 开始支持 JSON 数据类型。本主题描述了 JSON 的基本概念。它还描述了如何创建 JSON 列、加载 JSON 数据、查询 JSON 数据，以及使用 JSON 函数和运算符构建和处理 JSON 数据。

## 什么是 JSON

JSON 是一种轻量级的数据交换格式，专为半结构化数据而设计。JSON 以分层树状结构呈现数据，灵活且易于在各种数据存储和分析场景中进行读写。JSON 支持 `NULL` 值和以下数据类型：NUMBER、STRING、BOOLEAN、ARRAY 和 OBJECT。

有关 JSON 的更多信息，请访问 [JSON 网站](http://www.json.org/?spm=a2c63.p38356.0.0.50756b9fVEfwCd)。有关 JSON 的输入和输出语法的信息，请参阅 [RFC 7159 中的 JSON 规范](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4)。

StarRocks 既支持存储 JSON 数据，也支持对 JSON 数据进行高效的查询和分析。StarRocks 不直接存储输入文本，而是以二进制格式存储 JSON 数据，以降低解析成本并提高查询效率。

## 使用 JSON 数据

### 创建 JSON 列

创建表时，可以使用 `JSON` 关键字将 `j` 列指定为 JSON 列。

```sql
CREATE TABLE `tj` (
    `id` INT(11) NOT NULL COMMENT "",
    `j`  JSON NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`)
PROPERTIES (
    "replication_num" = "3",
    "storage_format" = "DEFAULT"
);
```

### 加载数据并将数据存储为 JSON 数据

StarRocks 提供以下方法来加载数据并将数据存储为 JSON 数据：

- 方法 1：使用 `INSERT INTO` 将数据写入表的 JSON 列。以下示例中，使用了名为 `tj` 的表，该表的 `j` 列是 JSON 列。

```plaintext
INSERT INTO tj (id, j) VALUES (1, parse_json('{"a": 1, "b": true}'));
INSERT INTO tj (id, j) VALUES (2, parse_json('{"a": 2, "b": false}'));
INSERT INTO tj (id, j) VALUES (3, parse_json('{"a": 3, "b": true}'));
INSERT INTO tj (id, j) VALUES (4, json_object('a', 4, 'b', false)); 
```

> parse_json 函数可以将 STRING 数据解释为 JSON 数据。json_object 函数可以构造 JSON 对象或将现有表转换为 JSON 文件。有关详细信息，请参阅 [parse_json](../../sql-functions/json-functions/json-constructor-functions/parse_json.md) 和 [json_object](../../sql-functions/json-functions/json-constructor-functions/json_object.md)。

- 方法 2：使用 Stream Load 加载 JSON 文件并将文件存储为 JSON 数据。有关详细信息，请参阅 [加载 JSON 数据](../../../loading/StreamLoad.md#load-json-data)。

  - 如果要加载根 JSON 对象，请将 `jsonpaths` 设置为 `$`。
  - 如果要加载 JSON 对象的特定值，请将 `jsonpaths` 设置为 `$.a`，其中 `a` 指定一个键。StarRocks 支持的 JSON 路径表达式，请参见 [JSON 路径](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md#json-path-expressions)。

- 方法 3：使用 Broker Load 加载 Parquet 文件并将文件存储为 JSON 数据。有关详细信息，请参阅 [Broker Load](../data-manipulation/BROKER_LOAD.md)。

StarRocks 在加载 Parquet 文件时支持以下数据类型转换。

| Parquet 文件的数据类型                                    | JSON 数据类型 |
| ------------------------------------------------------------ | -------------- |
| INTEGER (INT8、INT16、INT32、INT64、UINT8、UINT16、UINT32 和 UINT64) | NUMBER         |
| FLOAT 和 DOUBLE                                             | NUMBER         |
| BOOLEAN                                                      | BOOLEAN        |
| STRING                                                       | STRING         |
| MAP                                                          | OBJECT         |
| STRUCT                                                       | OBJECT         |
| LIST                                                         | ARRAY          |
| 其他数据类型，如 UNION 和 TIMESTAMP                 | 不支持         |

- 方法 4：使用 [Routine](../../../loading/RoutineLoad.md) load 从 Kafka 连续加载 JSON 数据到 StarRocks。

### 查询和处理 JSON 数据

StarRocks 支持查询和处理 JSON 数据，并支持使用 JSON 函数和算子。

在以下示例中，使用了名为 `tj` 的表，该表的 `j` 列被指定为 JSON 列。

```plaintext
mysql> select * from tj;
+------+----------------------+
| id   |          j           |
+------+----------------------+
| 1    | {"a": 1, "b": true}  |
| 2    | {"a": 2, "b": false} |
| 3    | {"a": 3, "b": true}  |
| 4    | {"a": 4, "b": false} |
+------+----------------------+
```

示例 1：筛选 JSON 列的数据，检索满足 `id=1` 筛选条件的数据。

```plaintext
mysql> select * from tj where id = 1;
+------+---------------------+
| id   |           j         |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
+------+---------------------+
```

示例 2：筛选 JSON 列 `j` 的数据，检索满足指定筛选条件的数据。

> `j->'a'` 返回 JSON 数据。您可以使用第一个示例来比较数据（请注意，在此示例中执行了隐式转换）。或者，您可以使用 CAST 函数将 JSON 数据转换为 INT，然后比较数据。

```plaintext
mysql> select * from tj where j->'a' = 1;
+------+---------------------+
| id   | j                   |
+------+---------------------+
|    1 | {"a": 1, "b": true} |


mysql> select * from tj where cast(j->'a' as INT) = 1;
+------+---------------------+
| id   | j                   |
+------+---------------------+
|    1 | {"a": 1, "b": true} |
+------+---------------------+
```

示例 3：使用 CAST 函数将表的 JSON 列中的值转换为 BOOLEAN 值。然后，筛选 JSON 列的数据，检索满足指定筛选条件的数据。

```plaintext
mysql> select * from tj where cast(j->'b' as boolean);
+------+---------------------+
|  id  |          j          |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
| 3    | {"a": 3, "b": true} |
+------+---------------------+
```

示例 4：使用 CAST 函数将表的 JSON 列中的值转换为 BOOLEAN 值。然后，筛选 JSON 列的数据，检索满足指定筛选条件的数据，并对数据进行算术运算。

```plaintext
mysql> select cast(j->'a' as int) from tj where cast(j->'b' as boolean);
+-----------------------+
|  CAST(j->'a' AS INT)  |
+-----------------------+
|          3            |
|          1            |
+-----------------------+

mysql> select sum(cast(j->'a' as int)) from tj where cast(j->'b' as boolean);
+----------------------------+
| sum(CAST(j->'a' AS INT))  |
+----------------------------+
|              4             |
+----------------------------+
```

示例 5：使用 JSON 列作为排序键对表的数据进行排序。

```plaintext
mysql> select * from tj
    ->        where j->'a' <= parse_json('3')
    ->        order by cast(j->'a' as int);
+------+----------------------+
| id   | j                    |
+------+----------------------+
|    1 | {"a": 1, "b": true}  |
|    2 | {"a": 2, "b": false} |
|    3 | {"a": 3, "b": true}  |
|    4 | {"a": 4, "b": false} |
+------+----------------------+
4 rows in set (0.05 sec)
```

## JSON 函数和运算符

您可以使用 JSON 函数和运算符来构建和处理 JSON 数据。有关更多信息，请参阅 [JSON 函数和运算符概述](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md)。

## 限制和使用说明

- JSON 值的最大长度为 16 MB。

- ORDER BY、GROUP BY 和 JOIN 子句不支持对 JSON 列的引用。如果要创建对 JSON 列的引用，请在创建引用之前使用 CAST 函数将 JSON 列转换为 SQL 列。有关详细信息，请参阅 [cast](../../sql-functions/json-functions/json-query-and-processing-functions/cast.md)。

- JSON 列受到“重复键”、“主键”和“唯一键”表的支持，但在聚合表中不受支持。

- JSON 列不能用作 DUPLICATE KEY、PRIMARY KEY 和 UNIQUE KEY 表的分区键、存储桶键或维度列。它们不能在 ORDER BY、GROUP BY 和 JOIN 子句中使用。

- StarRocks 允许您使用以下 JSON 比较算子来查询 JSON 数据：`<`、`<=`、`>`、`>=`、`=` 和 `!=`。不允许使用 `IN` 来查询 JSON 数据。
