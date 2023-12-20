---
displayed_sidebar: English
---

# 幂、power、dpow、fpow

## 描述

返回 x 的 y 次幂的结果。

## 语法

```Haskell
POW(x,y);POWER(x,y);
```

## 参数

x：支持双精度浮点数（DOUBLE）数据类型。

y：支持双精度浮点数（DOUBLE）数据类型。

## 返回值

返回一个双精度浮点数（DOUBLE）数据类型的值。

## 示例

```Plain
mysql> select pow(2,2);
+-----------+
| pow(2, 2) |
+-----------+
|         4 |
+-----------+
1 row in set (0.00 sec)

mysql> select power(4,3);
+-------------+
| power(4, 3) |
+-------------+
|          64 |
+-------------+
1 row in set (0.00 sec)
```
