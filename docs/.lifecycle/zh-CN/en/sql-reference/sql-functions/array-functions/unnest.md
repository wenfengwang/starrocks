---
displayed_sidebar: "Chinese"
---

# unnest

## 描述

UNNEST是一个表函数，它接受一个数组，并将该数组中的元素转换为表的多行。 这种转换也被称为“扁平化”。

您可以使用Lateral Join与UNNEST来实现常见转换，例如，从STRING、ARRAY或BITMAP转换为多行。 有关更多信息，请参阅[Lateral join](../../../using_starrocks/Lateral_join.md)。

从v2.5开始，UNNEST可以接受多个数组参数。 这些数组可以具有不同的类型和长度（元素数量）。 如果这些数组具有不同的长度，则以最大长度为准，这意味着对于小于此长度的数组将添加null值。 有关更多信息，请参阅[示例2](#示例2-unnest接受多个参数)。

## 语法

```Haskell
unnest(array0[, array1 ...])
```

## 参数

`array`：您想要转换的数组。 必须是一个数组或可以求值为ARRAY数据类型的表达式。 您可以指定一个或多个数组或数组表达式。

## 返回值

返回从数组转换而来的多行。 返回值的类型取决于数组中元素的类型。

有关数组中支持的元素类型，请参阅[ARRAY](../../sql-statements/data-types/Array.md)。

## 使用说明

- UNNEST是一个表函数。 必须与Lateral Join一起使用，但不需要显式指定关键字Lateral Join。
- 如果数组表达式评估为NULL或为空，则不会返回任何行。
- 如果数组中的元素为NULL，则该元素返回NULL。

## 示例

### 示例1：UNNEST接受一个参数

```SQL
-- 创建具有ARRAY列的student_score表。
CREATE TABLE student_score
(
    `id` bigint(20) NULL COMMENT "",
    `scores` ARRAY<int> NULL COMMENT ""
)
DUPLICATE KEY (id)
DISTRIBUTED BY HASH(`id`);

-- 向此表中插入数据。
INSERT INTO student_score VALUES
(1, [80,85,87]),
(2, [77, null, 89]),
(3, null),
(4, []),
(5, [90,92]);

-- 从此表中查询数据。
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

-- 使用UNNEST将scores列扁平化为多行。
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

[80,85,87] 对应于 `id = 1` 被转换为三行。

[77,null,89] 对应于 `id = 2` 保留了空值。

`id = 3` 和 `id = 4` 的`scores`都为NULL和空，因此被跳过。

### 示例2：UNNEST接受多个参数

```SQL
-- 创建example_table表，其中type和scores列的类型不同。
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

-- 使用UNNEST将type和scores转换为多行。
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

`UNNEST`中的`type`和`scores`的类型和长度不同。

`type`是一个VARCHAR列，而`scores`是一个ARRAY列。 使用split()函数将`type`转换为ARRAY。

对于`id = 1`，将`type`转换为["typeA"，"typeB"]，含有两个元素。

对于`id = 2`，将`type`转换为["typeA"，"typeB"，"typeC"]，含有三个元素。

为了确保对于每个`id`有一致数量的行，向["typeA"，"typeB"]添加了一个null元素。