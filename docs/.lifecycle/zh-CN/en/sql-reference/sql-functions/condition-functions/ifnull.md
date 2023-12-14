---
displayed_sidebar: "中文"
---

# ifnull

## 描述

若 `expr1` 为 NULL，则返回 expr2。若 `expr1` 不为 NULL，则返回 `expr1`。

## 语法

```Haskell
ifnull(expr1,expr2);
```

## 参数

`expr1` 和 `expr2` 必须在数据类型上兼容。

## 返回值

返回值与 `expr1` 的类型相同。

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
1 行记录 (0.01 秒)
```