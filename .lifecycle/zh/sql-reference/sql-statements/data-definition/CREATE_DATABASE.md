---
displayed_sidebar: English
---

# 创建数据库

## 描述

此语句用于创建数据库。

:::提示

进行此操作需要在目标目录上拥有 **CREATE DATABASE** 权限。您可以遵循 [GRANT](../account-management/GRANT.md) 中的指导来授予该权限。

:::

## 语法

```sql
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## 示例

1. 创建数据库 db_test。

   ```sql
   CREATE DATABASE db_test;
   ```

## 参考资料

- [显示创建数据库](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [使用](../data-definition/USE.md)
- [显示数据库](../data-manipulation/SHOW_DATABASES.md)
- [DESC](../Utility/DESCRIBE.md)
- [删除数据库](../data-definition/DROP_DATABASE.md)
