---
displayed_sidebar: English
---

# to_base64

## 描述

将字符串转换成 Base64-**编码**格式的字符串。这个函数是 [from_base64](from_base64.md) 函数的逆操作。

## 语法

```Haskell
to_base64(str);
```

## 参数

str：需要编码的字符串。必须是 VARCHAR 类型。

## 返回值

返回的值为 VARCHAR 类型。如果输入值为 NULL，则返回 NULL。如果输入值为空，将返回一个错误。

此函数仅接受单个字符串作为输入。输入多个字符串将会引发错误。

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
