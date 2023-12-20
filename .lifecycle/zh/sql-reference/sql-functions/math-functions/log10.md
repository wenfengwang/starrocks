---
displayed_sidebar: English
---

# log10、dlog10

## 描述

计算一个数的以10为底的对数。

## 语法

```SQL
log10(arg)
```

## 参数

- arg：您想要计算对数的值。只支持DOUBLE数据类型。

> **注意**
> StarRocks返回**NULL**如果`arg`被指定为负数或0。

## 返回值

返回一个DOUBLE数据类型的值。

## 示例

示例1：计算100的以10为底的对数。

```Plain
select log10(100);
+------------+
| log10(100) |
+------------+
|          2 |
+------------+
1 row in set (0.02 sec)
```
