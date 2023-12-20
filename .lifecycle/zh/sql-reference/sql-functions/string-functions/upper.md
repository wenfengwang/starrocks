---
displayed_sidebar: English
---

# 将字符串转为大写。

## 描述

将一个字符串全部转换成大写字母。

## 语法

```haskell
upper(str)
```

## 参数

- str：需要转换的字符串。如果str不是字符串类型，将会首先尝试进行隐式类型转换。

## 返回值

返回转换后的全大写字符串。

## 示例

```plaintext
MySQL [test]> select C_String, upper(C_String) from ex_iceberg_tbl;
+-------------------+-------------------+
| C_String          | upper(C_String)   |
+-------------------+-------------------+
| Hello, StarRocks! | HELLO, STARROCKS! |
| Hello, World!     | HELLO, WORLD!     |
+-------------------+-------------------+
```
