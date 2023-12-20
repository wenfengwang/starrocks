---
displayed_sidebar: English
---

# hll_hash

## 描述

将某个值转换为 hll 类型。这通常用于在导入过程中，把源数据的值映射为 StarRocks 表中的 HLL 列类型。

## 语法

```Haskell
HLL_HASH(column_name)
```

## 参数

column_name：所生成的 HLL 列的名称。

## 返回值

返回一个 hll 类型的值。

## 示例

```plain
mysql> select hll_cardinality(hll_hash("a"));
+--------------------------------+
| hll_cardinality(hll_hash('a')) |
+--------------------------------+
|                              1 |
+--------------------------------+
```
