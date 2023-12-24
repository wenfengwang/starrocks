---
displayed_sidebar: English
---

# 创建数据库

## 描述

该语句用于创建数据库。

:::提示

在目标目录上执行此操作需要具有 CREATE DATABASE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。

:::

## 语法

```sql
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## 例子

1. 创建名为 `db_test` 的数据库。

    ```sql
    CREATE DATABASE db_test;
    ```

## 引用

- [显示创建数据库](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [显示数据库](../data-manipulation/SHOW_DATABASES.md)
- [DESC](../Utility/DESCRIBE.md)
- [删除数据库](../data-definition/DROP_DATABASE.md)
