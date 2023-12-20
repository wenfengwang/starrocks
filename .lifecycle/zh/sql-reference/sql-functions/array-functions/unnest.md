---
displayed_sidebar: English
---

# 扁平化表函数

## 描述

UNNEST是一种表函数，它能够将数组中的元素转换成表格的多个行。这种转换也被称作“扁平化”。

您可以使用**横向连接（Lateral Join）**搭配UNNEST来实现常见的转换，比如将**STRING**、**ARRAY**或**BITMAP**转换为多行。更多详情，请参考[横向连接](../../../using_starrocks/Lateral_join.md)。

从v2.5版本开始，UNNEST能够接受数量不定的数组参数。这些数组在类型和长度（元素个数）上可以不同。如果数组长度不一，将以最长的数组为准，这意味着较短的数组将被填充空值。更多信息，请参考[示例2](#example-2-unnest-takes-multiple-parameters)。

## 语法

```Haskell
unnest(array0[, array1 ...])
```

## 参数

array：您想要转换的数组。它必须是数组或能够评估为ARRAY数据类型的表达式。您可以指定一个或多个数组或数组表达式。

## 返回值

返回由数组转换而来的多个行。返回值的类型取决于数组中元素的类型。

关于数组中支持的元素类型，请参考[ARRAY](../../sql-statements/data-types/Array.md)。

## 使用说明

- UNNEST是一个表函数。它必须与横向连接（Lateral Join）一起使用，但不需要显式指出"Lateral Join"关键词。
- 如果数组表达式计算结果为NULL或者是空的，将不会返回任何行。
- 如果数组中的元素是NULL，那么对应的元素返回NULL。

## 示例

### 示例1：UNNEST接受单一参数

```SQL
-- Create table student_score where scores is an ARRAY column.
CREATE TABLE student_score
(
    `id` bigint(20) NULL COMMENT "",
    `scores` ARRAY<int> NULL COMMENT ""
)
DUPLICATE KEY (id)
DISTRIBUTED BY HASH(`id`);

-- Insert data into this table.
INSERT INTO student_score VALUES
(1, [80,85,87]),
(2, [77, null, 89]),
(3, null),
(4, []),
(5, [90,92]);

-- Query data from this table.
SELECT * FROM student_score ORDER BY id;
+------+--------------+
| id   | scores       |
+------+--------------+
|    1 | [80,85,87]   |
|    2 | [77,null,89] |
|    3 | NULL         |
|    4 | []           |
|    5 | [90,92]      |
+------+--------------+

-- Use UNNEST to flatten the scores column into multiple rows.
SELECT id, scores, unnest FROM student_score, unnest(scores);
+------+--------------+--------+
| id   | scores       | unnest |
+------+--------------+--------+
|    1 | [80,85,87]   |     80 |
|    1 | [80,85,87]   |     85 |
|    1 | [80,85,87]   |     87 |
|    2 | [77,null,89] |     77 |
|    2 | [77,null,89] |   NULL |
|    2 | [77,null,89] |     89 |
|    5 | [90,92]      |     90 |
|    5 | [90,92]      |     92 |
+------+--------------+--------+
```

id=1对应的[80,85,87]被转换成三行。

id=2对应的[77,null,89]保留了null值。

id=3和id=4对应的scores为NULL和空值，因此被忽略。

### 示例2：UNNEST接受多个参数

```SQL
-- Create table example_table where the type and scores columns vary in type.
CREATE TABLE example_table (
id varchar(65533) NULL COMMENT "",
type varchar(65533) NULL COMMENT "",
scores ARRAY<int> NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(id)
COMMENT "OLAP"
DISTRIBUTED BY HASH(id)
PROPERTIES (
"replication_num" = "3");

-- Insert data into the table.
INSERT INTO example_table VALUES
("1", "typeA;typeB", [80,85,88]),
("2", "typeA;typeB;typeC", [87,90,95]);

-- Query data from the table.
SELECT * FROM example_table;
+------+-------------------+------------+
| id   | type              | scores     |
+------+-------------------+------------+
| 1    | typeA;typeB       | [80,85,88] |
| 2    | typeA;typeB;typeC | [87,90,95] |
+------+-------------------+------------+

-- Use UNNEST to convert type and scores into multiple rows.
SELECT id, unnest.type, unnest.scores
FROM example_table, unnest(split(type, ";"), scores) as unnest(type,scores);
+------+-------+--------+
| id   | type  | scores |
+------+-------+--------+
| 1    | typeA |     80 |
| 1    | typeB |     85 |
| 1    | NULL  |     88 |
| 2    | typeA |     87 |
| 2    | typeB |     90 |
| 2    | typeC |     95 |
+------+-------+--------+
```

UNNEST中的type和scores在类型和长度上有所不同。

type是一个VARCHAR类型的列，而scores是一个ARRAY类型的列。使用split()函数将type转换为ARRAY。

对于id=1，type被转换为["typeA","typeB"]，包含两个元素。

对于id=2，type被转换为["typeA","typeB","typeC"]，包含三个元素。

为了保持每个id对应的行数一致，在["typeA","typeB"]中添加了一个null元素。
