---
displayed_sidebar: English
---

# JSON

StarRocks 自 v2.2.0 版本起开始支持 JSON 数据类型。本主题将介绍 JSON 的基本概念，并说明如何创建 JSON 列、加载 JSON 数据、查询 JSON 数据，以及如何使用 JSON 函数和运算符来构建和处理 JSON 数据。

## 什么是 JSON？

JSON 是一种轻量级的数据交换格式，专门为处理半结构化数据而设计。JSON 以分层的树状结构呈现数据，这种结构在各种数据存储和分析场景中都表现出了灵活性和易读易写的特点。JSON 支持 NULL 值以及以下数据类型：NUMBER、STRING、BOOLEAN、ARRAY 和 OBJECT。

想要了解更多关于 JSON 的信息，请访问[JSON 官方网站](http://www.json.org/?spm=a2c63.p38356.0.0.50756b9fVEfwCd)。关于 JSON 输入和输出语法的详细信息，请参见[RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4) 中的 JSON 规范。

StarRocks 支持 JSON 数据的存储以及高效的查询和分析。StarRocks 不会直接存储输入文本，而是以二进制格式存储 JSON 数据，这样做可以降低解析成本并提升查询效率。

## 如何使用 JSON 数据

### 创建 JSON 列

在创建表时，您可以使用 JSON 关键词来指定某个列（如 j 列）为 JSON 类型的列。

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

### 加载数据并以 JSON 数据形式存储

StarRocks 提供了以下几种方法来加载数据并将其以 JSON 数据的形式存储：

- 方法一：使用 INSERT INTO 语句将数据写入表的 JSON 列。以下是一个示例，其中使用了名为 tj 的表，该表的 j 列被定义为 JSON 列。

```plaintext
INSERT INTO tj (id, j) VALUES (1, parse_json('{"a": 1, "b": true}'));
INSERT INTO tj (id, j) VALUES (2, parse_json('{"a": 2, "b": false}'));
INSERT INTO tj (id, j) VALUES (3, parse_json('{"a": 3, "b": true}'));
INSERT INTO tj (id, j) VALUES (4, json_object('a', 4, 'b', false)); 
```

> parse_json 函数能够将 STRING 数据解析为 JSON 数据。json_object 函数可以构造一个 JSON 对象或将现有表转换成 JSON 格式文件。欲了解更多信息，请参见 [parse_json](../../sql-functions/json-functions/json-constructor-functions/parse_json.md) 和 [json_object](../../sql-functions/json-functions/json-constructor-functions/json_object.md)。

- 方法二：使用 Stream Load 来加载 JSON 文件并将文件存储为 JSON 数据。更多信息请参见[加载 JSON 数据](../../../loading/StreamLoad.md#load-json-data)。

  - 如果您想要加载一个根 JSON 对象，请将 jsonpaths 设置为 "$"。
  - 如果您想要加载 JSON 对象中的特定值，请将 `jsonpaths` 设置为 `$.a`，其中 `a` 表示一个键名。关于 StarRocks 支持的 JSON 路径表达式的更多信息，请参见 [JSON 路径](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md#json-path-expressions)。

- 方法3：使用 Broker Load 来加载一个 Parquet 文件并将该文件存储为 JSON 数据。更多信息请参见 [Broker Load](../data-manipulation/BROKER_LOAD.md)。

在加载 Parquet 文件时，StarRocks 支持以下数据类型的转换：

|Parquet 文件的数据类型|JSON 数据类型|
|---|---|
|整数（INT8、INT16、INT32、INT64、UINT8、UINT16、UINT32 和 UINT64）|数字|
|浮点型和双精度型|数字|
|布尔值|布尔值|
|字符串|字符串|
|地图|物体|
|结构|对象|
|列表|数组|
|其他数据类型，例如 UNION 和 TIMESTAMP|不支持|

- 方法 4: 使用 [Routine](../../../loading/RoutineLoad.md) load 将来自 Kafka 的 JSON 数据持续性地加载到 StarRocks 中。

### 查询和处理 JSON 数据

StarRocks 支持对 JSON 数据的查询和处理，并使用 JSON 函数和运算符。

在以下示例中，使用了名为 tj 的表，并将该表的 j 列设置为 JSON 列。

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

示例 1：过滤 JSON 列的数据，提取出满足 id=1 过滤条件的数据。

```plaintext
mysql> select * from tj where id = 1;
+------+---------------------+
| id   |           j         |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
+------+---------------------+
```

示例 2：过滤 JSON 列 j 的数据，提取出满足特定过滤条件的数据。

> j->'a' 返回 JSON 数据。您可以使用第一个示例进行数据比较（请注意，本示例中进行了隐式转换）。或者，您也可以使用 CAST 函数将 JSON 数据转换为 INT 类型，然后进行数据比较。

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

示例 3：使用 CAST 函数将表中 JSON 列的值转换为 BOOLEAN 类型。然后，过滤 JSON 列的数据，提取出满足特定过滤条件的数据。

```plaintext
mysql> select * from tj where cast(j->'b' as boolean);
+------+---------------------+
|  id  |          j          |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
| 3    | {"a": 3, "b": true} |
+------+---------------------+
```

示例 4：使用 CAST 函数将表中 JSON 列的值转换为 BOOLEAN 类型。然后，过滤 JSON 列的数据，提取出满足特定过滤条件的数据，并对这些数据进行算术运算。

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

示例 5：使用 JSON 列作为排序键，对表中的数据进行排序。

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

您可以使用 JSON 函数和运算符来构建和处理 JSON 数据。欲了解更多信息，请参阅 [JSON 函数和运算符概览](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md)。

## 限制和使用注意事项

- JSON 值的最大长度为 16 MB。

- ORDER BY、GROUP BY 和 JOIN 子句不支持对 JSON 列的引用。如果需要对 JSON 列创建引用，请在创建引用之前使用 CAST 函数将 JSON 列转换为 SQL 列。更多信息请参见 [cast](../../sql-functions/json-functions/json-query-and-processing-functions/cast.md) 函数。

- Duplicate Key、Primary Key 和 Unique Key 类型的表支持 JSON 列。而 Aggregate 类型的表不支持 JSON 列。

- JSON 列不能用作 Duplicate Key、Primary Key 和 Unique Key 类型表的分区键、桶键或维度列。它们也不能在 ORDER BY、GROUP BY 和 JOIN 子句中使用。

- StarRocks 允许您使用以下 JSON 比较运算符来查询 JSON 数据：<、<=、>、>=、= 和 !=。但不支持使用 IN 运算符来查询 JSON 数据。
