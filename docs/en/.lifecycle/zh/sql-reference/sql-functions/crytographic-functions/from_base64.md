---
displayed_sidebar: English
---

# from_base64

## 描述

解码一个Base64编码的字符串。此函数是[to_base64](to_base64.md)的逆函数。

## 语法

```Haskell
from_base64(str);
```

## 参数

`str`：要解码的字符串。必须是VARCHAR类型。

## 返回值

返回VARCHAR类型的值。如果输入为NULL或无效的Base64字符串，则返回NULL。如果输入为空，则返回错误。

此函数只接受一个字符串输入。输入多于一个字符串将导致错误。

## 示例

```Plain
mysql> select from_base64("starrocks");
+--------------------------+
| from_base64('starrocks') |
+--------------------------+
| ²֫®$                       |
+--------------------------+
1 row in set (0.00 sec)

mysql> select from_base64('c3RhcnJvY2tz');
+-----------------------------+
| from_base64('c3RhcnJvY2tz') |
+-----------------------------+
| starrocks                   |
+-----------------------------+
```