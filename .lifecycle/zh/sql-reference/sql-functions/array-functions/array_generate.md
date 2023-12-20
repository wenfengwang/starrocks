---
displayed_sidebar: English
---

# 数组生成

## 描述

返回一个数组，包含由起始值 start 和结束值 end 指定范围内的不同值，以 step 为增量。

该函数从 v3.1 版本开始支持。

## 语法

```Haskell
ARRAY array_generate([start,] end [, step])
```

## 参数

- start：可选，起始值。必须是常量或者能够计算为 TINYINT、SMALLINT、INT、BIGINT 或 LARGEINT 类型的列。默认值是 1。
- end：必需，结束值。必须是常量或者能够计算为 TINYINT、SMALLINT、INT、BIGINT 或 LARGEINT 类型的列。
- step：可选，步进值。必须是常量或者能够计算为 TINYINT、SMALLINT、INT、BIGINT 或 LARGEINT 类型的列。当 start 小于 end 时，默认值是 1；当 start 大于 end 时，默认值是 -1。

## 返回值

返回一个数组，其元素数据类型与输入参数相同。

## 使用须知

- 如果输入参数中有列，必须明确指出该列所属的表格。
- 如果输入参数中有列，必须明确指出其他参数，不支持使用默认值。
- 如果任何输入参数为 NULL，返回结果为 NULL。
- 如果步进值 step 为 0，将返回一个空数组。
- 如果起始值 start 与结束值 end 相等，将返回包含该值的数组。

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

### 输入参数中有一列

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
mysql> select array_generate(1,c2,2) from `array_generate`;
+--------------------------+
| array_generate(1, c2, 2) |
+--------------------------+
| [1,3,5]                  |
| [1,3,5,7,9]              |
+--------------------------+
```
