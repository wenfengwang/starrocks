---
displayed_sidebar: English
---

# 百分位数哈希

## 描述

构造 DOUBLE 类型的数据为百分位数值。

## 语法

```Haskell
PERCENTILE_HASH(x);
```

## 参数

x：支持的数据类型是 DOUBLE。

## 返回值

返回一个百分位数值。

## 示例

```Plain
mysql> select percentile_approx_raw(percentile_hash(234.234), 0.99);
+-------------------------------------------------------+
| percentile_approx_raw(percentile_hash(234.234), 0.99) |
+-------------------------------------------------------+
|                                    234.23399353027344 |
+-------------------------------------------------------+
1 row in set (0.00 sec)
```
