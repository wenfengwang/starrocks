---
displayed_sidebar: English
---

# isnull

## 描述

检查值是否为`NULL`，如果是`NULL`则返回`1`，不是`NULL`则返回`0`。

## 语法

```Haskell
ISNULL(v)
```

## 参数

- `v`：要检查的值。支持所有数据类型。

## 返回值

如果值为`NULL`则返回1，不为`NULL`则返回0。

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