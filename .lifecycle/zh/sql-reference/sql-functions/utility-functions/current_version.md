---
displayed_sidebar: English
---

# 当前版本

## 描述

返回 StarRocks 的当前版本信息。为了适配不同客户端，提供了两种语法形式。

## 语法

```Haskell
current_version();

@@version_comment;
```

## 参数

无

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

```Plain
mysql> select current_version();
+-------------------+
| current_version() |
+-------------------+
| 2.1.2 0782ad7     |
+-------------------+
1 row in set (0.00 sec)

mysql> select @@version_comment;
+-------------------------+
| @@version_comment       |
+-------------------------+
| StarRocks version 2.1.2 |
+-------------------------+
1 row in set (0.01 sec)
```
