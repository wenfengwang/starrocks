---
displayed_sidebar: English
---

# log10、dlog10

## 描述

计算一个数的以 10 为底的对数。

## 语法

```SQL
log10(arg)
```

## 参数

- `arg`：您想要计算其对数的值。只支持 DOUBLE 数据类型。

> **注意**
> 如果 `arg` 被指定为负数或 0，StarRocks 将返回 `NULL`。

## 返回值

返回 DOUBLE 数据类型的值。

## 示例

示例 1：计算 100 的以 10 为底的对数。

```Plain
select log10(100);
+------------+
| log10(100) |
+------------+
|          2 |
+------------+
1 row in set (0.02 sec)
```