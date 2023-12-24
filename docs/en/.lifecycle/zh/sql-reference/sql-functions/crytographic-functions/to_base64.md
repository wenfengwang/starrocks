---
displayed_sidebar: English
---

# to_base64

## 描述

将字符串转换为 Base64 编码的字符串。此函数是 [from_base64](from_base64.md) 的反函数。

## 语法

```Haskell
to_base64(str);
```

## 参数

`str`：要编码的字符串。它必须是 VARCHAR 类型。

## 返回值

返回 VARCHAR 类型的值。如果输入为 NULL，则返回 NULL。如果输入为空，则返回错误。

此函数只接受一个字符串。多于一个输入字符串会导致错误。

## 例子

```Plain Text
mysql> select to_base64("starrocks");
+------------------------+
| to_base64('starrocks') |
+------------------------+
| c3RhcnJvY2tz           |
+------------------------+
1 行受影响 (0.00 秒)

mysql> select to_base64(123);
+----------------+
| to_base64(123) |
+----------------+
| MTIz           |
+----------------+