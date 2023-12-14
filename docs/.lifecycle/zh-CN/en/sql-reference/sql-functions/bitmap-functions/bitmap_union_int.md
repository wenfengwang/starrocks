---
displayed_sidebar: "Chinese"
---

# bitmap_union_int

## 描述

计算TINYINT、SMALLINT和INT类型列中不同值的数量，返回COUNT(DISTINCT expr)的总和。

## 语法

```Haskell
BIGINT bitmap_union_int(expr)
```

### 参数

`expr`: 列表达式。支持的列类型为TINYINT、SMALLINT和INT。

## 返回值

返回一个BIGINT类型的值。

## 示例

```Plaintext
mysql> select bitmap_union_int(k1) from tbl1;
+------------------------+
| bitmap_union_int(`k1`) |
+------------------------+
|                      2 |
+------------------------+
```