---
displayed_sidebar: English
---

# hll_hash

## 描述

将值转换为 hll 类型。通常在导入时使用，用于将源数据中的值映射到 StarRocks 表中的 HLL 列类型。

## 语法

```Haskell
HLL_HASH(column_name)
```

## 参数

`column_name`：生成的 HLL 列的名称。

## 返回值

返回一个 HLL 类型的值。

## 示例

```plain
mysql> select hll_cardinality(hll_hash("a"));
+--------------------------------+
| hll_cardinality(hll_hash('a')) |
+--------------------------------+
|                              1 |
+--------------------------------+
```