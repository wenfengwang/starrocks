```yaml
---
displayed_sidebar: "Chinese"
---

# isnotnull

## 描述

检查值是否不是 `NULL`，如果不是 `NULL`，则返回 `1`，如果是 `NULL`，则返回 `0`。

## 语法

```Haskell
ISNOTNULL(v)
```

## 参数

- `v`: 要检查的值。支持所有日期类型。

## 返回值

如果值不是 `NULL`，则返回 `1`，如果值是 `NULL`，则返回 `0`。

## 示例

```plain text
MYSQL > SELECT c1, isnotnull(c1) FROM t1;
+------+--------------+
| c1   | `c1` IS NULL |
+------+--------------+
| NULL |            0 |
|    1 |            1 |
+------+--------------+
```