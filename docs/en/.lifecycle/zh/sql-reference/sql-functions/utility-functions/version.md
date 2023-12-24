---
displayed_sidebar: English
---

# 版本

## 描述

返回 MySQL 数据库的当前版本。

您可以使用 [current_version](current_version.md) 来查询 StarRocks 的版本。

## 语法

```Haskell
VARCHAR version();
```

## 参数

无

## 返回值

返回 VARCHAR 类型的值。

## 例子

```Plain Text
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.1.0     |
+-----------+
1 行受影响 (0.00 秒)
```

## 引用

[current_version](../utility-functions/current_version.md)
