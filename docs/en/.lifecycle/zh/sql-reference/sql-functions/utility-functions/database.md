---
displayed_sidebar: English
---

# 数据库

## 描述

返回当前数据库的名称。如果没有选择数据库，则返回空值。

## 语法

```Haskell
database()
```

## 参数

此函数不需要参数。

## 返回值

以字符串形式返回当前数据库的名称。

## 例子

```sql
-- 选择目标数据库。
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
