---
displayed_sidebar: English
---

# 如果

## 描述

如果表达式 expr1 的结果为 TRUE，则返回 expr2；否则，返回 expr3。

## 语法

```Haskell
if(expr1,expr2,expr3);
```

## 参数

expr1：条件表达式。它必须是布尔值（BOOLEAN）。

expr2 和 expr3 必须是数据类型兼容的。

## 返回值

函数的返回值类型将与 expr2 的类型一致。

## 示例

```Plain
mysql> select if(true,1,2);
+----------------+
| if(TRUE, 1, 2) |
+----------------+
|              1 |
+----------------+

mysql> select if(false,2.14,2);
+--------------------+
| if(FALSE, 2.14, 2) |
+--------------------+
|               2.00 |
+--------------------+
```
