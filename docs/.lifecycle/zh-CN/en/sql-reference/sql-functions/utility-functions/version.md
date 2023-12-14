---
displayed_sidebar: "Chinese"
---

# 版本

## 描述

返回当前的MySQL数据库版本。

您可以使用[current_version](current_version.md)来查询StarRocks版本。

## 语法

```Haskell
VARCHAR version();
```

## 参数

无

## 返回值

返回一个VARCHAR类型的值。

## 例子

```Plain Text
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.1.0     |
+-----------+
1 row in set (0.00 sec)
```

## 参考

[current_version](../utility-functions/current_version.md)