---
displayed_sidebar: English
---

# 修改数据库

## 描述

配置指定数据库的属性。

:::提示

此操作需要在目标数据库上拥有**ALTER**权限。您可以遵循[GRANT](../account-management/GRANT.md)指令中的步骤来授予该权限。

:::

1. 以B/K/KB/M/MB/G/GB/T/TB/P/PB单位设置数据库数据配额。

   ```sql
   ALTER DATABASE db_name SET DATA QUOTA quota;
   ```

2. 重命名数据库

   ```sql
   ALTER DATABASE db_name RENAME new_db_name;
   ```

3. 设置数据库副本配额

   ```sql
   ALTER DATABASE db_name SET REPLICA QUOTA quota;
   ```

注意：

```plain
After renaming the database, use REVOKE and GRANT commands to modify the corresponding user permission if necessary.
The database's default data quota and the default replica quota are 2^63-1.
```

## 示例

1. 为指定数据库设置数据配额

   ```SQL
   ALTER DATABASE example_db SET DATA QUOTA 10995116277760;
   -- The above unit is bytes, equivalent to the following statement.
   ALTER DATABASE example_db SET DATA QUOTA 10T;
   ALTER DATABASE example_db SET DATA QUOTA 100G;
   ALTER DATABASE example_db SET DATA QUOTA 200M;
   ```

2. 将数据库example_db重命名为example_db2

   ```SQL
   ALTER DATABASE example_db RENAME example_db2;
   ```

3. 为指定数据库设置副本配额

   ```SQL
   ALTER DATABASE example_db SET REPLICA QUOTA 102400;
   ```

## 参考资料

- [创建数据库](CREATE_DATABASE.md)
- [使用](../data-definition/USE.md)
- [显示数据库](../data-manipulation/SHOW_DATABASES.md)
- [DESC](../Utility/DESCRIBE.md)（描述）
- [删除数据库](../data-definition/DROP_DATABASE.md)
