---
displayed_sidebar: English
---

# LIKE

## 描述

检查给定表达式是否模糊匹配指定的模式。如果是，则返回 1。否则返回 0。如果输入参数中有任何一个为 NULL，则返回 NULL。

通常使用 LIKE 与百分号（%）和下划线（_）等字符一起。`%` 匹配 0、1 或多个字符。`_` 匹配任何单个字符。

## 语法

```Haskell
BOOLEAN like(VARCHAR expr, VARCHAR pattern);
```

## 参数

- `expr`：字符串表达式。支持的数据类型为 VARCHAR。

- `pattern`：要匹配的模式。支持的数据类型为 VARCHAR。

## 返回值

返回一个 BOOLEAN 值。

## 例子

```Plain Text
mysql> select like("star","star");
+----------------------+
| like('star', 'star') |
+----------------------+
|                    1 |
+----------------------+

mysql> select like("starrocks","star%");
+----------------------+
| like('starrocks', 'star%') |
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