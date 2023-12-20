---
displayed_sidebar: English
---

# to_base64

## 描述

将字符串转换为Base64编码的字符串。此函数是[from_base64](from_base64.md)的逆函数。

## 语法

```Haskell
to_base64(str);
```

## 参数

`str`：要编码的字符串。必须是VARCHAR类型。

## 返回值

返回VARCHAR类型的值。如果输入为NULL，则返回NULL。如果输入为空，则返回错误。

此函数只接受一个字符串输入。输入多于一个字符串将导致错误。

## 示例

```Plain
mysql> select to_base64("starrocks");
+------------------------+
| to_base64('starrocks') |
+------------------------+
| c3RhcnJvY2tz           |
+------------------------+
1 row in set (0.00 sec)

mysql> select to_base64(123);
+----------------+
| to_base64(123) |
+----------------+
| MTIz           |
+----------------+
```