---
displayed_sidebar: English
---

# IF

## 描述

如果 `expr1` 评估为 TRUE，则返回 `expr2`。否则，返回 `expr3`。

## 语法

```Haskell
if(expr1,expr2,expr3);
```

## 参数

`expr1`：条件。它必须是布尔值。

`expr2`和`expr3`的数据类型必须兼容。

## 返回值

返回值与 `expr2` 的类型相同。

## 例子

```Plain Text
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