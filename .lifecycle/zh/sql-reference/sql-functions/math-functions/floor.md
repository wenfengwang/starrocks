---
displayed_sidebar: English
---

# 地板函数，floor

## 描述

返回不超过 x 的最大整数。

## 语法

```SQL
FLOOR(x);
```

## 参数

x：支持 DOUBLE 类型。

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
1 row in set (0.01 sec)
```
