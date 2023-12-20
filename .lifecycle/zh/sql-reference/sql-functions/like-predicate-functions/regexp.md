---
displayed_sidebar: English
---

# 正则表达式

## 描述：

检查给定的表达式是否与指定的正则表达式模式相匹配。如果匹配，则返回1；如果不匹配，则返回0。如果任一输入参数为NULL，则返回NULL。

regexp() 支持比[like()](like.md)更复杂的匹配条件。

## 语法：

```Haskell
BOOLEAN regexp(VARCHAR expr, VARCHAR pattern);
```

## 参数：

- expr：字符串表达式。支持的数据类型为 VARCHAR。

- pattern：匹配的模式。支持的数据类型为 VARCHAR。

## 返回值：

返回布尔类型的值。

## 示例：

```Plain
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

## 关键词：

regexp, regular expression
