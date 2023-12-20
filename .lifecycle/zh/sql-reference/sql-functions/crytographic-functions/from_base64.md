---
displayed_sidebar: English
---

# 解码Base64字符串

## 描述

将Base64编码的字符串解码。这个函数是[to_base64](to_base64.md)函数的反向操作。

## 语法

```Haskell
from_base64(str);
```

## 参数

str：需要解码的字符串。必须是VARCHAR类型。

## 返回值

返回VARCHAR类型的值。如果输入为NULL或不是有效的Base64字符串，将返回NULL。如果输入为空字符串，将报错。

此函数仅接受单个字符串输入。输入多个字符串将引发错误。

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
