---
displayed_sidebar: English
---

# 创建数据库

## 描述

此语句用于创建数据库。

:::tip

此操作需要在目标目录上的 CREATE DATABASE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明来授予此权限。

:::

## 语法

```sql
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## 示例

1. 创建名为 `db_test` 的数据库。

   ```sql
   CREATE DATABASE db_test;
   ```

## 参考资料

- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)