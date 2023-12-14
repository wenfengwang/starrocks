---
displayed_sidebar: "Chinese"
---

# 删除数据库

## 描述

在StarRocks中删除数据库。

> **注意**
>
> 此操作需要对目标数据库拥有DROP权限。

## 语法

```sql
DROP DATABASE [IF EXISTS] <db_name> [FORCE]
```

请注意以下几点：

- 在执行DROP DATABASE删除数据库之后，您可以使用[RECOVER](../data-definition/RECOVER.md)语句在指定的保留期内（默认保留期为一天）恢复已删除的数据库，但是无法恢复已经与数据库一起被删除的管道（从v3.2版本开始支持）。
- 如果您执行`DROP DATABASE FORCE`删除数据库，数据库将直接删除，而不会检查其中是否有未完成的活动，并且无法恢复。通常不建议执行此操作。
- 如果删除数据库，所有属于该数据库的管道（从v3.2版本开始支持）都将与数据库一起被删除。

## 示例

1. 删除数据库db_test。

    ```sql
    DROP DATABASE db_test;
    ```

## 参考

- [创建数据库](../data-definition/CREATE_DATABASE.md)
- [显示创建数据库](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [使用](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)
- [显示数据库](../data-manipulation/SHOW_DATABASES.md)