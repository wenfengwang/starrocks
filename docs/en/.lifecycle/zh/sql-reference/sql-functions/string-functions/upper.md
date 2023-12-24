---
displayed_sidebar: English
---

# 大写

## 描述

将字符串转换为大写。

## 语法

```haskell
upper(str)
```

## 参数

- `str`：要转换的字符串。如果 `str` 不是字符串类型，将首先尝试进行隐式转换。

## 返回值

返回一个大写字符串。

## 例子

```plaintext
MySQL [test]> select C_String, upper(C_String) from ex_iceberg_tbl;
+-------------------+-------------------+
| C_String          | upper(C_String)   |
+-------------------+-------------------+
| Hello, StarRocks! | HELLO, STARROCKS! |
| Hello, World!     | HELLO, WORLD!     |
+-------------------+-------------------+