---
displayed_sidebar: English
---

# 合并

## 描述

返回输入参数中的第一个非 NULL 表达式。如果找不到非 NULL 表达式，则返回 NULL。

## 语法

```Haskell
coalesce(expr1,...);
```

## 参数

`expr1`：输入表达式，必须计算为兼容的数据类型。

## 返回值

返回值与 `expr1` 的类型相同。

## 例子

```Plain Text
mysql> select coalesce(3,NULL,1,1);
+-------------------------+
| coalesce(3, NULL, 1, 1) |
+-------------------------+
|                       3 |
+-------------------------+
1 行记录受影响 (0.00 秒)