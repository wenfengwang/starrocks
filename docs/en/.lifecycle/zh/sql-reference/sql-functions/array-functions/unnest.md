---
displayed_sidebar: English
---

# UNNEST

## 描述

UNNEST 是一个表函数，它接受一个数组，并将该数组中的元素转换为表的多行。这种转换也被称为“扁平化”。

您可以使用 Lateral Join 结合 UNNEST 来实现常见的转换，例如，从 STRING、ARRAY 或 BITMAP 转换为多行。有关详细信息，请参阅 [Lateral join](../../../using_starrocks/Lateral_join.md)。

从 v2.5 开始，UNNEST 可以接受可变数量的数组参数。这些数组的类型和长度（元素个数）可以各不相同。如果数组的长度不同，则取最大长度为准，这意味着对于长度小于最大长度的数组，将会添加 null 值。有关详细信息，请参阅 [示例 2](#example-2-unnest-takes-multiple-parameters)。

## 语法

```Haskell
unnest(array0[, array1 ...])
```

## 参数

`array`：要转换的数组。它必须是一个数组或可以计算为 ARRAY 数据类型的表达式。您可以指定一个或多个数组或数组表达式。

## 返回值

返回从数组转换而来的多行。返回值的类型取决于数组中元素的类型。

有关数组中支持的元素类型，请参阅 [ARRAY](../../sql-statements/data-types/Array.md)。

## 使用说明

- UNNEST 是一个表函数。它必须与 Lateral Join 一起使用，但不需要显式指定 Lateral Join 关键字。
- 如果数组表达式的计算结果为 NULL 或为空，则不会返回任何行。
- 如果数组中的元素为 NULL，则对应的行也将返回 NULL。

## 例子

### 示例 1：UNNEST 接受一个参数

```SQL
-- 创建一个名为 student_score 的表，其中 scores 是一个 ARRAY 列。
CREATE TABLE student_score
(
    `id` bigint(20) NULL COMMENT "",
    `scores` ARRAY<int> NULL COMMENT ""
)
DUPLICATE KEY (id)
DISTRIBUTED BY HASH(`id`);

-- 向该表中插入数据。
INSERT INTO student_score VALUES
(1, [80,85,87]),
(2, [77, null, 89]),
(3, null),
(4, []),
(5, [90,92]);

-- 从该表中查询数据。
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

[80,85,87] 对应于 `id = 1`，被转换为三行。

[77，空，89] 对应于 `id = 2`，保留了 null 值。

`id = 3` 和 `id = 4` 对应的 `scores` 是 NULL 和空，将被跳过。

### 示例 2：UNNEST 接受多个参数

```SQL
-- 创建一个名为 example_table 的表，其中 type 和 scores 列的类型各不相同。
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

-- 向该表中插入数据。
INSERT INTO example_table VALUES
("1", "typeA;typeB", [80,85,88]),
("2", "typeA;typeB;typeC", [87,90,95]);

-- 从该表中查询数据。
SELECT * FROM example_table;
+------+-------------------+------------+
| id   | type              | scores     |
+------+-------------------+------------+
| 1    | typeA;typeB       | [80,85,88] |
| 2    | typeA;typeB;typeC | [87,90,95] |
+------+-------------------+------------+

-- 使用 UNNEST 将 type 和 scores 转换为多行。
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

`type` 和 `scores` 在 `UNNEST` 中的类型和长度各不相同。

`type` 是一个 VARCHAR 列，而 `scores` 是一个 ARRAY 列。split() 函数用于将 `type` 转换为 ARRAY。

对于 `id = 1`，`type` 被转换为 ["typeA","typeB"]，它包含两个元素。

对于 `id = 2`，`type` 被转换为 ["typeA","typeB","typeC"]，它包含三个元素。

为了确保每个 `id` 的行数一致，对于 ["typeA","typeB"]，添加了一个 null 元素。
