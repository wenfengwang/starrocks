---
displayed_sidebar: English
---

# percentile_hash

## 描述

构造 DOUBLE 值作为 PERCENTILE 值。

## 语法

```Haskell
PERCENTILE_HASH(x);
```

## 参数

`x`：支持的数据类型为 DOUBLE。

## 返回值

返回 PERCENTILE 值。

## 例子

```Plain Text
mysql> select percentile_approx_raw(percentile_hash(234.234), 0.99);
+-------------------------------------------------------+
| percentile_approx_raw(percentile_hash(234.234), 0.99) |
+-------------------------------------------------------+
|                                    234.23399353027344 |
+-------------------------------------------------------+
1 行受影响 (0.00 秒)