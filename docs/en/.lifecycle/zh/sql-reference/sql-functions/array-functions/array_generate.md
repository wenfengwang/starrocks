---
displayed_sidebar: English
---

# array_generate

## 描述

返回一个由 `start` 和 `end` 指定范围内的不重复值组成的数组，增量为 `step`。

该函数从 v3.1 版本开始支持。

## 语法

```Haskell
ARRAY array_generate([start,] end [, step])
```

## 参数

- `start`：可选，起始值。它必须是常量或计算结果为 TINYINT、SMALLINT、INT、BIGINT 或 LARGEINT 的列。默认值为 1。
- `end`：必需，结束值。它必须是常量或计算结果为 TINYINT、SMALLINT、INT、BIGINT 或 LARGEINT 的列。
- `step`：可选，增量。它必须是常量或计算结果为 TINYINT、SMALLINT、INT、BIGINT 或 LARGEINT 的列。当 `start` 小于 `end` 时，默认值为 1。当 `start` 大于 `end` 时，默认值为 -1。

## 返回值

返回一个数组，其元素的数据类型与输入参数相同。

## 使用说明

- 如果输入参数是列，则必须指定该列所属的表。
- 如果任何输入参数是列，则必须指定其他参数。不支持默认值。
- 如果任何输入参数为 NULL，返回 NULL。
- 如果 `step` 为 0，则返回空数组。
- 如果 `start` 等于 `end`，则返回该值。

## 示例

### 输入参数为常量

```Plain
mysql> select array_generate(9);
+---------------------+
| array_generate(9)   |
+---------------------+
| [1,2,3,4,5,6,7,8,9] |
+---------------------+

select array_generate(9,12);
+-----------------------+
| array_generate(9, 12) |
+-----------------------+
| [9,10,11,12]          |
+-----------------------+

select array_generate(9,6);
+----------------------+
| array_generate(9, 6) |
+----------------------+
| [9,8,7,6]            |
+----------------------+

select array_generate(9,6,-1);
+--------------------------+
| array_generate(9, 6, -1) |
+--------------------------+
| [9,8,7,6]                |
+--------------------------+

select array_generate(3,3);
+----------------------+
| array_generate(3, 3) |
+----------------------+
| [3]                  |
+----------------------+
```

### 输入参数中包含列

```sql
CREATE TABLE `array_generate`
(
  `c1` TINYINT,
  `c2` SMALLINT,
  `c3` INT
)
ENGINE = OLAP
DUPLICATE KEY(`c1`)
DISTRIBUTED BY HASH(`c1`);

INSERT INTO `array_generate` VALUES
(1, 6, 3),
(2, 9, 4);
```

```Plain
mysql> select array_generate(1, c2, 2) from `array_generate`;
+--------------------------+
| array_generate(1, c2, 2) |
+--------------------------+
| [1,3,5]                  |
| [1,3,5,7,9]              |
+--------------------------+
```