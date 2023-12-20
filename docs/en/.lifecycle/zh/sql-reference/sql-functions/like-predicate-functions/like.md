---
displayed_sidebar: English
---

# like

## 描述

检查给定表达式是否模糊匹配指定模式。如果是，则返回1。否则，返回0。如果任何输入参数为NULL，则返回NULL。

LIKE通常与百分号（%）和下划线（_）等字符一起使用。`%`可以匹配0、1或多个字符。`_`可以匹配任何单个字符。

## 语法

```Haskell
BOOLEAN like(VARCHAR expr, VARCHAR pattern);
```

## 参数

- `expr`：字符串表达式。支持的数据类型是VARCHAR。

- `pattern`：要匹配的模式。支持的数据类型是VARCHAR。

## 返回值

返回一个BOOLEAN值。

## 示例

```Plain
mysql> select like("star","star");
+----------------------+
| like('star', 'star') |
+----------------------+
|                    1 |
+----------------------+

mysql> select like("starrocks","star%");
+--------------------------+
| like('starrocks', 'star%') |
+--------------------------+
|                        1 |
+--------------------------+

mysql> select like("starrocks","star_");
+----------------------------+
| like('starrocks', 'star_') |
+----------------------------+
|                          0 |
+----------------------------+
```