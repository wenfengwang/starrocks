---
displayed_sidebar: English
---

# array_generate

## 描述

返回一个由`start`和`end`指定范围内不同值组成的数组，增量为`step`。

此函数从v3.1开始支持。

## 语法

```Haskell
ARRAY array_generate([start,] end [, step])
```

## 参数

- `start`：可选，起始值。必须是计算结果为TINYINT、SMALLINT、INT、BIGINT或LARGEINT的常量或列。默认值为1。
- `end`：必填，结束值。必须是计算结果为TINYINT、SMALLINT、INT、BIGINT或LARGEINT的常量或列。
- `step`：可选，增量。必须是计算结果为TINYINT、SMALLINT、INT、BIGINT或LARGEINT的常量或列。当`start`小于`end`时，默认值为1。当`start`大于`end`时，默认值为-1。

## 返回值

返回一个数组，其元素的数据类型与输入参数相同。

## 使用说明

- 如果任何输入参数是列，则必须指定该列所属的表。
- 如果任何输入参数是列，则必须指定其他参数。不支持默认值。
- 如果任何输入参数为NULL，则返回NULL。
- 如果`step`为0，则返回一个空数组。
- 如果`start`等于`end`，则返回该值。

## 例子

### 输入参数为常量

```Plain Text
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

### 其中一个输入参数为列

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

```Plain Text
mysql> select array_generate(1,c2,2) from `array_generate`;
+--------------------------+
| array_generate(1, c2, 2) |
+--------------------------+
| [1,3,5]                  |
| [1,3,5,7,9]              |
+--------------------------+
```
