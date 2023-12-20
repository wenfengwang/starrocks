---
displayed_sidebar: English
---

# 对数2

## 描述

计算一个数的以2为底的对数。

## 语法

```SQL
log2(arg)
```

## 参数

- arg：想要计算对数的数值。只支持DOUBLE数据类型。

> **注意**
> StarRocks返回**NULL**如果`arg`被指定为负数或0。

## 返回值

返回的值为DOUBLE数据类型。

## 示例

示例1：计算8的以2为底的对数。

```Plain
mysql> select log2(8);
+---------+
| log2(8) |
+---------+
|       3 |
+---------+
1 row in set (0.00 sec)
```
