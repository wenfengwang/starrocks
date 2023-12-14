```yaml
---
displayed_sidebar: "Chinese"
---

# log2

## 描述

计算一个数的以2为底的对数。

## 语法

```SQL
log2(arg)
```

## 参数

- `arg`: 您要计算对数的值。仅支持DOUBLE数据类型。

> **注意**
>
> 如果指定的`arg`是负数或者0，StarRocks会返回NULL。

## 返回值

返回一个DOUBLE数据类型的值。

## 示例

示例1：计算8的以2为底的对数。

```Plain
mysql> select log2(8);
+---------+
| log2(8) |
+---------+
|       3 |
+---------+
1行记录(0.00秒)
```