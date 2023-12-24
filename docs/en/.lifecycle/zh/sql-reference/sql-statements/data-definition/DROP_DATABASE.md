---
displayed_sidebar: English
---

# 删除数据库

## 描述

在 StarRocks 中删除数据库。

> **注意**
>
> 此操作需要对目标数据库具有 DROP 权限。

## 语法

```sql
DROP DATABASE [IF EXISTS] <db_name> [FORCE]
```

请注意以下几点：

- 执行 DROP DATABASE 删除数据库后，您可以使用 RECOVER 语句在指定的保留期内（默认保留期为一天）恢复已删除的数据库，但是与数据库一起删除的管道（从 v3.2 开始支持）无法恢复。
- 如果执行 `DROP DATABASE FORCE` 删除数据库，则直接删除该数据库，而不检查其中是否存在未完成的活动，且无法恢复。通常不建议执行此操作。
- 如果删除数据库，则属于该数据库的所有管道（从 v3.2 开始支持）将与数据库一起被删除。

## 例子

1. 删除数据库 db_text。

    ```sql
    DROP DATABASE db_test;
    ```

## 引用

- [创建数据库](../data-definition/CREATE_DATABASE.md)
- [显示创建数据库](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [显示数据库](../data-manipulation/SHOW_DATABASES.md)
