---
displayed_sidebar: English
---

# 判断是否为空

## 描述

检查某个值是否为NULL，如果是NULL，则返回1；如果不是NULL，则返回0。

## 语法

```Haskell
ISNULL(v)
```

## 参数

- v：需要检查的值。支持所有数据类型。

## 返回值

若值为NULL，则返回1；若值不为NULL，则返回0。

## 示例

```plain
MYSQL > SELECT c1, isnull(c1) FROM t1;
+------+--------------+
| c1   | `c1` IS NULL |
+------+--------------+
| NULL |            1 |
|    1 |            0 |
+------+--------------+
```
