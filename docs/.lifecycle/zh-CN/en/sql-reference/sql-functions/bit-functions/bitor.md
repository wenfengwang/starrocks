---
displayed_sidebar: "Chinese"
---

# bitor

## 描述

返回两个数字表达式的按位 OR。

## 语法

```Haskell
BITOR(x,y);
```

## 参数

- `x`: 该表达式必须评估为以下任一数据类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: 该表达式必须评估为以下任一数据类型：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` 和 `y`必须具有相同的数据类型。

## 返回值

返回值与`x`具有相同的类型。 如果任何一个值为 NULL，则结果为 NULL。

## 示例

```Plain Text
mysql> select bitor(3,0);
+-------------+
| bitor(3, 0) |
+-------------+
|           3 |
+-------------+
1 行受影响 (0.00 秒)
```