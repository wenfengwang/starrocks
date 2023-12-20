---
displayed_sidebar: English
---

# 数组去重聚合

## 描述

聚合数组列中的不同值（包括NULL）到一个新数组中（多行合并成一行）。

## 语法

```Haskell
ARRAY_UNIQUE_AGG(col)
```

## 参数

- col：你想要聚合的列。支持的数据类型为ARRAY。

## 返回值

返回一个ARRAY类型的值。

## 使用说明

- 数组中元素的顺序是随机的。
- 返回的数组中元素的数据类型与原始输入列中的元素数据类型一致。
- 如果输入为空并且没有分组列，则返回NULL。

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

示例1：对a列的值进行分组，并将b列中的不同值聚合成一个数组。

```plaintext
mysql> select a, array_unique_agg(b) from array_unique_agg_example group by a;
+------+---------------------+
| a    | array_unique_agg(b) |
+------+---------------------+
|    1 | [4,1,2,3]           |
|    2 | [4,1,2,3,null]      |
+------+---------------------+
```

示例2：使用WHERE子句来聚合b列的值。如果没有数据符合过滤条件，则返回一个NULL值。

```plaintext
mysql> select array_unique_agg(b) from array_unique_agg_example where a < 0;
+---------------------+
| array_unique_agg(b) |
+---------------------+
| NULL                |
+---------------------+
```

## 关键词

ARRAY_UNIQUE_AGG，数组去重聚合
