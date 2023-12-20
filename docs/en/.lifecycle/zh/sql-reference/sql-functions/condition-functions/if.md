---
displayed_sidebar: English
---

# if

## 描述

如果 `expr1` 的计算结果为 TRUE，则返回 `expr2`。否则，返回 `expr3`。

## 语法

```Haskell
if(expr1, expr2, expr3);
```

## 参数

`expr1`：条件。它必须是 BOOLEAN 值。

`expr2` 和 `expr3` 必须在数据类型上兼容。

## 返回值

返回值的类型与 `expr2` 相同。

## 示例

```Plain
mysql> select if(true, 1, 2);
+----------------+
| if(TRUE, 1, 2) |
+----------------+
|              1 |
+----------------+

mysql> select if(false, 2.14, 2);
+--------------------+
| if(FALSE, 2.14, 2) |
+--------------------+
|               2.00 |
+--------------------+
```