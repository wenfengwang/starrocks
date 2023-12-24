---
displayed_sidebar: English
---

# from_base64

## 描述

解码 Base64 编码的字符串。此函数是 [to_base64](to_base64.md) 的逆函数。

## 语法

```Haskell
from_base64(str);
```

## 参数

`str`：要解码的字符串。它必须是 VARCHAR 类型。

## 返回值

返回 VARCHAR 类型的值。如果输入为 NULL 或无效的 Base64 字符串，则返回 NULL。如果输入为空，则返回错误。

此函数只接受一个字符串。多个输入字符串会导致错误。

## 例子

```Plain Text
mysql> select from_base64("starrocks");
+--------------------------+
| from_base64('starrocks') |
+--------------------------+
| ²֫®$                   |
+--------------------------+
1 行受影响 (0.00 秒)

mysql> select from_base64('c3RhcnJvY2tz');
+-----------------------------+
| from_base64('c3RhcnJvY2tz') |
+-----------------------------+
| starrocks                   |
+-----------------------------+