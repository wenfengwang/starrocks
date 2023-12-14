---
displayed_sidebar: "Chinese"
---

# hll_hash

## 描述

将一个值转换为hll类型。通常在导入时使用，将源数据中的值映射到StarRocks表中的HLL列类型。

## 语法

```Haskell
HLL_HASH(column_name)
```

## 参数

`column_name`：生成的HLL列的名称。

## 返回值

返回一个HLL类型的值。

## 示例

```plain text
mysql> select hll_cardinality(hll_hash("a"));
+--------------------------------+
| hll_cardinality(hll_hash('a')) |
+--------------------------------+
|                              1 |
+--------------------------------+
```