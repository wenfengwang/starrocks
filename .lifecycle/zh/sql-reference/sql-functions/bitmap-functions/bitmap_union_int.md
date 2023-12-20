---
displayed_sidebar: English
---

# 位图合并整型

## 描述

计算 TINYINT、SMALLINT 和 INT 类型列中不同值的数量，返回 COUNT(DISTINCT expr) 各项之和。

## 语法

```Haskell
BIGINT bitmap_union_int(expr)
```

### 参数

expr：列表达式。支持的列类型包括 TINYINT、SMALLINT 和 INT。

## 返回值

返回一个 BIGINT 类型的值。

## 示例

```Plaintext
mysql> select bitmap_union_int(k1) from tbl1;
+------------------------+
| bitmap_union_int(`k1`) |
+------------------------+
|                      2 |
+------------------------+
```
