---
displayed_sidebar: English
---

# 数据库

## 描述

返回当前数据库的名称。如果没有选定数据库，则返回一个空值。

## 语法

```Haskell
database()
```

## 参数

此函数不需要参数。

## 返回值

返回当前数据库名称的字符串。

## 示例

```sql
-- 选择一个目标数据库。
use db_test

-- 查询当前数据库的名称。
select database();
+------------+
| DATABASE() |
+------------+
| db_test    |
+------------+
```

## 另请参阅

[USE](../../sql-statements/data-definition/USE.md)：切换到目标数据库。