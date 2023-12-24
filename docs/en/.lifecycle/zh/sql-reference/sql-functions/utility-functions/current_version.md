---
displayed_sidebar: English
---

# current_version

## 描述

返回 StarRocks 的当前版本。提供了两种语法，以便与不同的客户端兼容。

## 语法

```Haskell
current_version();

@@version_comment;
```

## 参数

无

## 返回值

返回 VARCHAR 类型的值。

## 例子

```Plain Text
mysql> select current_version();
+-------------------+
| current_version() |
+-------------------+
| 2.1.2 0782ad7     |
+-------------------+
1 行受影响 (0.00 秒)

mysql> select @@version_comment;
+-------------------------+
| @@version_comment       |
+-------------------------+
| StarRocks version 2.1.2 |
+-------------------------+
1 行受影响 (0.01 秒)