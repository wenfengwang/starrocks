---
displayed_sidebar: English
---

# 修改数据库

## 描述

配置指定数据库的属性。

:::tip

此操作需要在目标数据库上的 **ALTER** 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明来授予此权限。

:::

1. 以 B/K/KB/M/MB/G/GB/T/TB/P/PB 单位设置数据库数据配额。

   ```sql
   ALTER DATABASE db_name SET DATA QUOTA quota;
   ```

2. 重命名数据库

   ```sql
   ALTER DATABASE db_name RENAME TO new_db_name;
   ```

3. 设置数据库副本配额

   ```sql
   ALTER DATABASE db_name SET REPLICA QUOTA quota;
   ```

注意：

```plain
重命名数据库后，如有必要，请使用 REVOKE 和 GRANT 命令修改相应的用户权限。
数据库的默认数据配额和默认副本配额是 2^63-1。
```

## 示例

1. 设置指定数据库的数据配额

   ```SQL
   ALTER DATABASE example_db SET DATA QUOTA 10995116277760;
   -- 上述单位是字节，等同于以下语句。
   ALTER DATABASE example_db SET DATA QUOTA 10T;
   ALTER DATABASE example_db SET DATA QUOTA 100G;
   ALTER DATABASE example_db SET DATA QUOTA 200M;
   ```

2. 将数据库 example_db 重命名为 example_db2

   ```SQL
   ALTER DATABASE example_db RENAME TO example_db2;
   ```

3. 设置指定数据库的副本配额

   ```SQL
   ALTER DATABASE example_db SET REPLICA QUOTA 102400;
   ```

## 参考资料

- [CREATE DATABASE](CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
- [DESCRIBE](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)