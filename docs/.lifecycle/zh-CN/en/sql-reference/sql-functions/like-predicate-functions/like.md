---
displayed_sidebar: "Chinese"
---

# like

## 描述

检查给定的表达式是否模糊匹配指定的模式。如果是，则返回1。否则返回0。如果输入参数中的任何一个为NULL，则返回NULL。

LIKE通常与字符一起使用，比如百分号（%）和下划线（_）。`%`表示匹配0个、1个或多个字符。`_`表示匹配任何单个字符。

## 语法

```Haskell
BOOLEAN like(VARCHAR expr, VARCHAR pattern);
```

## 参数

- `expr`：字符串表达式。支持的数据类型为VARCHAR。

- `pattern`：要匹配的模式。支持的数据类型为VARCHAR。

## 返回值

返回一个BOOLEAN值。

## 示例

```Plain Text
mysql> select like("star","star");
+----------------------+
| like('star', 'star') |
+----------------------+
|                    1 |
+----------------------+

mysql> select like("starrocks","star%");
+----------------------+
| like('star', 'star') |
+----------------------+
|                    1 |
+----------------------+

mysql> select like("starrocks","star_");
+----------------------------+
| like('starrocks', 'star_') |
+----------------------------+
|                          0 |
+----------------------------+
```