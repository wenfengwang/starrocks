---
displayed_sidebar: English
---

# 删除数据库

## 描述

在 StarRocks 中删除一个数据库。

> **注意**
> 此操作需要对目标数据库有 **DROP** 权限。

## 语法

```sql
DROP DATABASE [IF EXISTS] <db_name> [FORCE]
```

请注意以下几点：

- 执行 `DROP DATABASE` 删除数据库后，在指定的保留期限内（默认保留期限为1天）可以使用 [RECOVER](../data-definition/RECOVER.md) 语句恢复被删除的数据库，但是与数据库一起删除的管道（从 v3.2 版本开始支持）无法恢复。
- 如果执行 `DROP DATABASE FORCE` 删除数据库，数据库将会被直接删除，不会检查数据库中是否有未完成的活动，且无法恢复。通常不推荐使用此操作。
- 删除数据库时，所有属于该数据库的管道（从 v3.2 版本开始支持）也会随之被删除。

## 示例

1. 删除数据库 db_test。

   ```sql
   DROP DATABASE db_test;
   ```

## 参考资料

- [CREATE DATABASE](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [DESCRIBE](../Utility/DESCRIBE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)