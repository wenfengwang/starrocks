---
displayed_sidebar: English
---

# UNNEST

## 描述

UNNEST 是一个表函数，它接受一个数组并将该数组中的元素转换为表的多行。这种转换也被称为“扁平化”。

您可以使用 Lateral Join 与 UNNEST 来实现常见的转换，例如，将 STRING、ARRAY 或 BITMAP 转换为多行。有关更多信息，请参见[横向连接](../../../using_starrocks/Lateral_join.md)。

从 v2.5 版本开始，UNNEST 可以接受多个数组参数。这些数组在类型和长度（元素数量）上可以不同。如果数组长度不同，则以最长的数组长度为准，这意味着会向长度较短的数组中添加 null 值。更多信息请参见[示例 2](#example-2-unnest-takes-multiple-parameters)。

## 语法

```Haskell
unnest(array0[, array1 ...])
```

## 参数

`array`：您想要转换的数组。它必须是一个数组或能够计算为 ARRAY 数据类型的表达式。您可以指定一个或多个数组或数组表达式。

## 返回值

返回从数组转换而来的多行。返回值的类型取决于数组中元素的类型。

有关数组中支持的元素类型，请参见 [ARRAY](../../sql-statements/data-types/Array.md)。

## 使用说明

- UNNEST 是一个表函数。它必须与 Lateral Join 一起使用，但不需要显式指定 Lateral Join 关键字。
- 如果数组表达式计算结果为 NULL 或它是空的，则不会返回任何行。
- 如果数组中的某个元素为 NULL，则该元素返回 NULL。

## 示例

### 示例 1：UNNEST 使用一个参数

```SQL
-- 创建 student_score 表，其中 scores 是一个 ARRAY 列。
CREATE TABLE student_score
(
    `id` bigint(20) NULL COMMENT "",
    `scores` ARRAY<int> NULL COMMENT ""
)
DUPLICATE KEY (`id`)
DISTRIBUTED BY HASH(`id`);

-- 向该表中插入数据。
INSERT INTO student_score VALUES
(1, [80,85,87]),
(2, [77, null, 89]),
(3, null),
(4, []),
(5, [90,92]);

-- 从该表查询数据。
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

-- 使用 UNNEST 将 scores 列扁平化为多行。
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

id=1 对应的 [80,85,87] 转换成三行。

id=2 对应的 [77,null,89] 保留了 null 值。

id=3 和 id=4 对应的 `scores` 为 NULL 和空数组，这些情况被跳过。

### 示例 2：UNNEST 使用多个参数

```SQL
-- 创建 example_table 表，其中 type 和 scores 列的类型不同。
CREATE TABLE example_table (
id varchar(65533) NULL COMMENT "",
type varchar(65533) NULL COMMENT "",
scores ARRAY<int> NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY (id)
COMMENT "OLAP"
DISTRIBUTED BY HASH(id)
PROPERTIES (
"replication_num" = "3");

-- 向表中插入数据。
INSERT INTO example_table VALUES
("1", "typeA;typeB", [80,85,88]),
("2", "typeA;typeB;typeC", [87,90,95]);

-- 从表中查询数据。
SELECT * FROM example_table;
+------+-------------------+------------+
| id   | type              | scores     |
+------+-------------------+------------+
| 1    | typeA;typeB       | [80,85,88] |
| 2    | typeA;typeB;typeC | [87,90,95] |
+------+-------------------+------------+

-- 使用 UNNEST 将 type 和 scores 转换为多行。
SELECT id, unnest.type, unnest.scores
FROM example_table, unnest(split(type, ";"), scores) AS unnest(type, scores);
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

UNNEST 中的 `type` 和 `scores` 在类型和长度上有所不同。

`type` 是一个 VARCHAR 列，而 `scores` 是一个 ARRAY 列。使用 split() 函数将 `type` 转换为 ARRAY。

对于 id=1，`type` 转换为 ["typeA","typeB"]，它有两个元素。

对于 id=2，`type` 转换为 ["typeA","typeB","typeC"]，它有三个元素。

为了确保每个 id 的行数一致，向 ["typeA","typeB"] 中添加了一个 null 元素。