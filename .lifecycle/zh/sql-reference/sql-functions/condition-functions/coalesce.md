---
displayed_sidebar: English
---

# 聚合

## 描述

返回输入参数中第一个非空（NULL）的表达式。如果所有表达式均为 NULL，则返回 NULL。

## 语法

```Haskell
coalesce(expr1,...);
```

## 参数

expr1：输入表达式，必须计算得出兼容的数据类型。

## 返回值

返回值类型与 expr1 相同。

## 示例

```Plain
mysql> select coalesce(3,NULL,1,1);
+-------------------------+
| coalesce(3, NULL, 1, 1) |
+-------------------------+
|                       3 |
+-------------------------+
1 row in set (0.00 sec)
```
