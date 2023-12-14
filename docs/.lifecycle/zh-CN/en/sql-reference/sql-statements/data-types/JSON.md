---
displayed_sidebar: "Chinese"
---

# JSON

自 v2.2.0 开始，StarRocks 开始支持 JSON 数据类型。本主题描述JSON的基本概念。它还描述了如何创建JSON列，加载JSON数据，查询JSON数据以及使用JSON函数和运算符构建和处理JSON数据。

## 什么是JSON

JSON是一种轻量级的、为半结构化数据设计的数据交换格式。JSON以分层树结构呈现数据，这种格式在各种数据存储和分析场景中具有灵活性并且易于阅读和编写。JSON支持`NULL`值和以下数据类型：NUMBER，STRING，BOOLEAN，ARRAY以及OBJECT。

有关JSON的更多信息，请访问[JSON网站](http://www.json.org/?spm=a2c63.p38356.0.0.50756b9fVEfwCd)。有关JSON输入和输出语法的信息，请参见[RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4) 上的JSON规范。

StarRocks支持JSON数据的存储、高效查询和分析。StarRocks不直接存储输入文本。相反，它以二进制格式存储JSON数据，以减少解析成本并提高查询效率。

## 使用JSON数据

### 创建JSON列

在创建表时，您可以使用`JSON`关键字将`j`列指定为JSON列。

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

### 加载数据并将数据存储为JSON数据

StarRocks提供以下方法让您将数据加载并存储为JSON数据：

- 方法1：使用`INSERT INTO`将数据写入到表的JSON列。在以下示例中，使用了名为`tj`的表，该表的`j`列是一个JSON列。

```plaintext
INSERT INTO tj (id, j) VALUES (1, parse_json('{"a": 1, "b": true}'));
INSERT INTO tj (id, j) VALUES (2, parse_json('{"a": 2, "b": false}'));
INSERT INTO tj (id, j) VALUES (3, parse_json('{"a": 3, "b": true}'));
INSERT INTO tj (id, j) VALUES (4, json_object('a', 4, 'b', false)); 
```

> parse_json函数可以将STRING数据解释为JSON数据。json_object函数可以构建JSON对象或将现有表转换为JSON文件。更多信息，请参见[parse_json](../../sql-functions/json-functions/json-constructor-functions/parse_json.md) 和 [json_object](../../sql-functions/json-functions/json-constructor-functions/json_object.md)。

- 方法2：使用流加载来加载JSON文件并将文件存储为JSON数据。有关详细信息，请参见[加载JSON数据](../../../loading/StreamLoad.md#load-json-data)。

  - 如果要加载根JSON对象，请将`jsonpaths`设置为`$`。
  - 如果要加载JSON对象的特定值，请将`jsonpaths`设置为`$.a`，其中`a`指定一个键。有关StarRocks支持的JSON路径表达式的更多信息，请参见[JSON路径](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md#json-path-expressions)。

- 方法3：使用代理加载来加载Parquet文件并将文件存储为JSON数据。有关详细信息，请参见[代理加载](../data-manipulation/BROKER_LOAD.md)。

StarRocks支持在加载Parquet文件时进行以下数据类型转换。

| Parquet文件的数据类型                                    | JSON数据类型 |
| ------------------------------------------------------- | ------------ |
| INTEGER (INT8, INT16, INT32, INT64, UINT8, UINT16, UINT32, 和 UINT64) | NUMBER       |
| FLOAT 和 DOUBLE                                          | NUMBER       |
| BOOLEAN                                                   | BOOLEAN      |
| STRING                                                    | STRING       |
| MAP                                                       | OBJECT       |
| STRUCT                                                    | OBJECT       |
| LIST                                                      | ARRAY        |
| 其他数据类型如UNION和TIMESTAMP                           | 不支持       |

- 方法4：使用[Routine](../../../loading/RoutineLoad.md)加载，持续将JSON数据从Kafka加载到StarRocks。

### 查询和处理JSON数据

StarRocks支持对JSON数据进行查询和处理，以及使用JSON函数和运算符。

在以下示例中，使用了名为`tj`的表，该表的`j`列指定为JSON列。

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

示例1：过滤JSON列的数据，以检索满足`id=1`过滤条件的数据。

```plaintext
mysql> select * from tj where id = 1;
+------+---------------------+
| id   |           j         |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
+------+---------------------+
```

示例2：过滤JSON列`j`的数据，以检索满足指定过滤条件的数据。

> `j->'a'` 返回JSON数据。您可以使用第一个示例来比较数据（请注意，在本示例中执行了隐式转换）。或者，您可以使用CAST函数将JSON数据转换为INT，然后再比较数据。

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

示例3：使用CAST函数将表的JSON列中的值转换为BOOLEAN值。然后，过滤JSON列的数据，以检索满足指定过滤条件的数据。

```plaintext
mysql> select * from tj where cast(j->'b' as boolean);
+------+---------------------+
|  id  |          j          |
+------+---------------------+
| 1    | {"a": 1, "b": true} |
| 3    | {"a": 3, "b": true} |
+------+---------------------+
```

示例4：使用CAST函数将表的JSON列中的值转换为BOOLEAN值。然后，过滤JSON列的数据，以检索满足指定过滤条件的数据，并对数据进行算术运算。

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

示例5：使用JSON列作为排序键，对表的数据进行排序。

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

## JSON函数和运算符

您可以使用JSON函数和运算符来构建和处理JSON数据。有关更多信息，请参见[JSON函数和运算符概述](../../sql-functions/json-functions/overview-of-json-functions-and-operators.md)。

## 限制和使用注意事项

- JSON值的最大长度为16MB。

- ORDER BY、GROUP BY 和 JOIN 子句不支持对 JSON 列的引用。 如果要创建对 JSON 列的引用，请在创建引用之前使用 CAST 函数将 JSON 列转换为 SQL 列。 有关更多信息，请参见[cast](../../sql-functions/json-functions/json-query-and-processing-functions/cast.md)。

- JSON 列支持重复键表、主键表和唯一键表。 它们不支持聚合表。

- 不能将 JSON 列用作重复键表、主键表和唯一键表的分区键、分桶键或维度列。 不能在 ORDER BY、GROUP BY 和 JOIN 子句中使用它们。

- StarRocks 允许您使用以下 JSON 比较运算符查询 JSON 数据：`<`、`<=`、`>`、`>=`、`= 、` 和 `!=`。 但它不允许您使用 `IN` 来查询 JSON 数据。