---
displayed_sidebar: English
---

# 版本

## 描述

返回当前 MySQL 数据库的版本。

您可以使用[current_version](current_version.md)来查询StarRocks的版本。

## 语法

```Haskell
VARCHAR version();
```

## 参数

无

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

```Plain
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.1.0     |
+-----------+
1 row in set (0.00 sec)
```

## 参考资料

[current_version](../utility-functions/current_version.md)
