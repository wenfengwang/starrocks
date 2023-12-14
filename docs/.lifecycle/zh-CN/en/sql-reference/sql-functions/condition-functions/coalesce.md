---
displayed_sidebar: "Chinese"
---

# 合并

## 描述

返回输入参数中第一个非空表达式。如果找不到非空表达式，则返回NULL。

## 语法

```Haskell
coalesce(expr1,...);
```

## 参数

`expr1`: 输入表达式，必须评估为兼容的数据类型。

## 返回值

返回值与 `expr1` 的类型相同。

## 示例

```Plain Text
mysql> select coalesce(3,NULL,1,1);
+-------------------------+
| coalesce(3, NULL, 1, 1) |
+-------------------------+
|                       3 |
+-------------------------+
1 row in set (0.00 sec)
```