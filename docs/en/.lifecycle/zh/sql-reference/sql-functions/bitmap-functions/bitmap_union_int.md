---
displayed_sidebar: English
---

# bitmap_union_int

## 描述

计算 TINYINT、SMALLINT 和 INT 类型的列中不同值的数量，返回 COUNT (DISTINCT expr) 的总和。

## 语法

```Haskell
BIGINT bitmap_union_int(expr)
```

### 参数

`expr`：列表达式。支持的列类型为 TINYINT、SMALLINT 和 INT。

## 返回值

返回一个 BIGINT 类型的值。

## 例子

```Plaintext
mysql> select bitmap_union_int(k1) from tbl1;
+------------------------+
| bitmap_union_int(`k1`) |
+------------------------+
|                      2 |
+------------------------+