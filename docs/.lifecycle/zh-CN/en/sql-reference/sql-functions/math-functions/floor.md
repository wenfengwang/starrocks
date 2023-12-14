---
displayed_sidebar: "Chinese"
---

# floor, dfloor

## 描述

返回不超过 `x` 的最大整数。

## 语法

```SQL
FLOOR(x);
```

## 参数

`x`：支持 DOUBLE 类型。

## 返回值

返回 BIGINT 数据类型的值。

## 示例

```Plaintext
mysql> select floor(3.14);
+-------------+
| floor(3.14) |
+-------------+
|           3 |
+-------------+
1 行受影响 (0.01 秒)
```