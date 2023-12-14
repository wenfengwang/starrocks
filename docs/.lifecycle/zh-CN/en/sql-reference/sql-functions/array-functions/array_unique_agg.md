---
displayed_sidebar: "Chinese"
---

# array_unique_agg

## 描述

将数组列中的不同值（包括`NULL`）聚合到一个数组中（多行到一行）。

## 语法

```Haskell
ARRAY_UNIQUE_AGG(col)
```

## 参数

- `col`: 您要聚合值的列。支持的数据类型是数组。

## 返回值

返回数组类型的值。

## 使用说明

- 数组中元素的顺序是随机的。
- 返回的数组中元素的数据类型与输入列中元素的数据类型相同。
- 如果输入为空且没有group-by列，则返回`NULL`。

## 示例

以以下数据表为例：

```plaintext
mysql> select * from array_unique_agg_example;
+------+--------------+
| a    | b            |
+------+--------------+
|    2 | [1,null,2,4] |
|    2 | [1,null,3]   |
|    1 | [1,1,2,3]    |
|    1 | [2,3,4]      |
+------+--------------+
```

示例1：对列`a`中的值进行分组，并将列`b`中的不同值聚合到一个数组中。

```plaintext
mysql> select a, array_unique_agg(b) from array_unique_agg_example group by a;
+------+---------------------+
| a    | array_unique_agg(b) |
+------+---------------------+
|    1 | [4,1,2,3]           |
|    2 | [4,1,2,3,null]      |
+------+---------------------+
```

示例2：使用WHERE子句聚合列`b`中的值。如果没有数据满足筛选条件，则返回`NULL`值。

```plaintext
mysql> select array_unique_agg(b) from array_unique_agg_example where a < 0;
+---------------------+
| array_unique_agg(b) |
+---------------------+
| NULL                |
+---------------------+
```

## 关键词

ARRAY_UNIQUE_AGG, ARRAY