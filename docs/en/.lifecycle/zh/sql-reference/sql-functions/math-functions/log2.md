---
displayed_sidebar: English
---

# log2

## 描述

计算一个数的以 2 为底的对数。

## 语法

```SQL
log2(arg)
```

## 参数

- `arg`：要计算对数的值。只支持 DOUBLE 数据类型。

> **注意**
> 如果 `arg` 被指定为负数或0，StarRocks 将返回 `NULL`。

## 返回值

返回 DOUBLE 数据类型的值。

## 示例

示例 1：计算 8 的以 2 为底的对数。

```Plain
mysql> select log2(8);
+---------+
| log2(8) |
+---------+
|       3 |
+---------+
1 row in set (0.00 sec)
```