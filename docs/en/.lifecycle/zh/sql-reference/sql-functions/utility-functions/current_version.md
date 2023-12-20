---
displayed_sidebar: English
---

# current_version

## 描述

返回 StarRocks 的当前版本。为了兼容不同客户端，提供了两种语法。

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
| StarRocks 版本 2.1.2    |
+-------------------------+
1 row in set (0.01 sec)
```