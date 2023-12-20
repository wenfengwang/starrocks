---
displayed_sidebar: English
---

# 数据库

## 描述

返回当前数据库的名称。如果没有选定数据库，将返回一个空值。

## 语法

```Haskell
database()
```

## 参数

此函数不需要任何参数。

## 返回值

返回一个字符串，内容为当前数据库的名称。

## 示例

```sql
-- Select a destination database.
use db_test

-- Query the name of the current database.
select database();
+------------+
| DATABASE() |
+------------+
| db_test    |
+------------+
```

## 另请参阅

[USE](../../sql-statements/data-definition/USE.md)：切换至指定的目标数据库。
