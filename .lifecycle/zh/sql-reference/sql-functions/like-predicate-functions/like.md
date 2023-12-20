---
displayed_sidebar: English
---

# LIKE

## 描述

检查给定的表达式是否模糊匹配特定的模式。如果匹配，则返回1。如果不匹配，则返回0。如果任何输入参数为NULL，则返回NULL。

LIKE通常与如百分号（%）和下划线（_）这样的字符一起使用。%可以匹配0个、1个或多个字符。_可以匹配任意单个字符。

## 语法

```Haskell
BOOLEAN like(VARCHAR expr, VARCHAR pattern);
```

## 参数

- expr：字符串表达式。支持的数据类型为VARCHAR。

- pattern：要匹配的模式。支持的数据类型为VARCHAR。

## 返回值

返回一个布尔值。

## 示例

```Plain
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
