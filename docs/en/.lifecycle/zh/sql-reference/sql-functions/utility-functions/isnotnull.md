---
displayed_sidebar: English
---

# isnotnull

## 描述

检查值是否不为`NULL`，如果不为`NULL`，则返回`1`；如果为`NULL`，则返回`0`。

## 语法

```Haskell
ISNOTNULL(v)
```

## 参数

- `v`：要检查的值。支持所有数据类型。

## 返回值

如果不为`NULL`则返回`1`，如果为`NULL`则返回`0`。

## 示例

```plain
MYSQL > SELECT c1, isnotnull(c1) FROM t1;
+------+--------------+
| c1   | isnotnull(c1) |
+------+--------------+
| NULL |            0 |
|    1 |            1 |
+------+--------------+
```