---
displayed_sidebar: "中文"
---

# ifnull

## 功能

如果 `expr1` 不为 NULL，就返回 `expr1`。如果 `expr1` 为 NULL，就返回 `expr2`。

## 语法

```Haskell
ifnull(expr1,expr2);
```

## 参数说明

`expr1` 和 `expr2` 必须在数据类型上兼容，否则将报错。

## 返回值说明

返回值的数据类型将与 `expr1` 的类型相同。

## 示例

```Plain Text
mysql> select ifnull(2,4);
+--------------+
| ifnull(2, 4) |
+--------------+
|            2 |
+--------------+

mysql> select ifnull(NULL,2);
+-----------------+
| ifnull(NULL, 2) |
+-----------------+
|               2 |
+-----------------+
```