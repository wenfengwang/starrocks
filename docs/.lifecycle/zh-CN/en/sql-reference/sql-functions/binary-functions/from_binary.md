---
displayed_sidebar: "Chinese"
---

# from_binary

## 描述

根据指定的二进制格式（`binary_type`），将二进制值转换为基于VARCHAR的字符串。支持以下二进制格式：`hex`、`encode64` 和 `utf8`。如果未指定`binary_type`，则默认为`hex`。

## 语法

```Haskell
from_binary(binary[, binary_type])
```

## 参数

- `binary`：要转换的输入二进制，必需。

- `binary_type`：转换的二进制格式，可选。

  - `hex`（默认）：`from_binary` 使用 `hex` 方法将输入二进制编码为 VARCHAR 字符串。
  - `encode64`：`from_binary` 使用 `base64` 方法将输入二进制编码为 VARCHAR 字符串。
  - `utf8`：`from_binary` 将输入二进制转换为不经任何转换的 VARCHAR 字符串。

## 返回值

返回VARCHAR字符串。

## 示例

```Plain
mysql> select from_binary(to_binary('ABAB', 'hex'), 'hex');
+----------------------------------------------+
| from_binary(to_binary('ABAB', 'hex'), 'hex') |
+----------------------------------------------+
| ABAB                                         |
+----------------------------------------------+
1 row in set (0.02 sec)

mysql> select from_base64(from_binary(to_binary('U1RBUlJPQ0tT', 'encode64'), 'encode64'));
+-----------------------------------------------------------------------------+
| from_base64(from_binary(to_binary('U1RBUlJPQ0tT', 'encode64'), 'encode64')) |
+-----------------------------------------------------------------------------+
| STARROCKS                                                                   |
+-----------------------------------------------------------------------------+
1 row in set (0.01 sec)

mysql> select from_binary(to_binary('STARROCKS', 'utf8'), 'utf8');
+-----------------------------------------------------+
| from_binary(to_binary('STARROCKS', 'utf8'), 'utf8') |
+-----------------------------------------------------+
| STARROCKS                                           |
+-----------------------------------------------------+
1 row in set (0.01 sec)

```

## 参考

- [to_binary](to_binary_zh.md)
- [BINARY/VARBINARY数据类型](../../sql-statements/data-types/BINARY_zh.md)