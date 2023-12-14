---
displayed_sidebar: "中文"
---

# nullif

## 功能

如果参数 `expr1` 与 `expr2` 相等，则返回 NULL，否则返回 `expr1` 的值。

## 语法

```Haskell
nullif(expr1,expr2);
```

## 参数说明

`expr1` 与 `expr2` 必须在数据类型上兼容，否则会报错。

## 返回值说明

返回值的数据类型与 `expr1` 的类型相同。

## 示例

```Plain Text
mysql> select nullif(1,2);
+--------------+
| nullif(1, 2) |
+--------------+
|            1 |
+--------------+

mysql> select nullif(1,1);
+--------------+
| nullif(1, 1) |
+--------------+
|         NULL |
+--------------+
1 row in set (0.01 sec)
```