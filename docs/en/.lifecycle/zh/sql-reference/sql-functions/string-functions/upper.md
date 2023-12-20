---
displayed_sidebar: English
---

# upper

## 描述

将字符串转换为大写形式。

## 语法

```haskell
upper(str)
```

## 参数

- `str`：要转换的字符串。如果 `str` 不是字符串类型，将会尝试进行隐式类型转换。

## 返回值

返回转换为大写的字符串。

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