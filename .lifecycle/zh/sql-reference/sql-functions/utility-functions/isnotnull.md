---
displayed_sidebar: English
---

# 非空检查

## 描述

判断指定值是否非空（不是NULL），非空时返回1，为空时返回0。

## 语法

```Haskell
ISNOTNULL(v)
```

## 参数

- v：需进行检查的值。支持所有数据类型。

## 返回值

非空时返回1，为空时返回0。

## 示例

```plain
MYSQL > SELECT c1, isnotnull(c1) FROM t1;
+------+--------------+
| c1   | `c1` IS NULL |
+------+--------------+
| NULL |            0 |
|    1 |            1 |
+------+--------------+
```
