---
displayed_sidebar: English
---

# 正则表达式

## 描述

检查给定的表达式是否与指定的正则表达式`pattern`匹配。如果匹配，则返回1。否则返回0。如果任何输入参数为NULL，则返回NULL。

regexp()支持比[like()](like.md)更复杂的匹配条件。

## 语法

```Haskell
BOOLEAN regexp(VARCHAR expr, VARCHAR pattern);
```

## 参数

- `expr`：字符串表达式。支持的数据类型为VARCHAR。

- `pattern`：要匹配的模式。支持的数据类型为VARCHAR。

## 返回值

返回一个BOOLEAN值。

## 例子

```Plain Text
mysql> select regexp("abc123","abc*");
+--------------------------+
| regexp('abc123', 'abc*') |
+--------------------------+
|                        1 |
+--------------------------+
1 row in set (0.06 sec)

select regexp("abc123","xyz*");
+--------------------------+
| regexp('abc123', 'xyz*') |
+--------------------------+
|                        0 |
+--------------------------+
```

## 关键字

正则表达式，常规