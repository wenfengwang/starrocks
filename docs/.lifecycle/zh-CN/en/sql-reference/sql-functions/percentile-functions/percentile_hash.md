---
displayed_sidebar: "Chinese"
---

# percentile_hash

## 描述

将DOUBLE值构造为百分位值。

## 语法

```Haskell
PERCENTILE_HASH(x);
```

## 参数

`x`：支持的数据类型为DOUBLE。

## 返回值

返回百分位值。

## 示例

```Plain Text
mysql> select percentile_approx_raw(percentile_hash(234.234), 0.99);
+-------------------------------------------------------+
| percentile_approx_raw(percentile_hash(234.234), 0.99) |
+-------------------------------------------------------+
|                                    234.23399353027344 |
+-------------------------------------------------------+
1 row in set (0.00 sec)
```