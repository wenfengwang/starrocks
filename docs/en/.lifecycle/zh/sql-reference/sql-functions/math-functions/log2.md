---
displayed_sidebar: English
---

# log2

## 描述

计算一个数字的以 2 为底的对数。

## 语法

```SQL
log2(arg)
```

## 参数

- `arg`：要计算其对数的值。仅支持 DOUBLE 数据类型。

> **注意**
>
> 如果 `arg` 被指定为负数或 0，则 StarRocks 返回 NULL。

## 返回值

返回一个 DOUBLE 数据类型的值。

## 例

示例 1：计算 8 的以 2 为底的对数。

```Plain
mysql> select log2(8);
+---------+
| log2(8) |
+---------+
|       3 |
+---------+
1 行受影响 (0.00 秒)