---
displayed_sidebar: English
---

# 修改数据库

## 描述

配置指定数据库的属性。

:::提示

此操作需要对目标数据库具有 ALTER 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。

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

```plain text
重命名数据库后，如果有必要，使用 REVOKE 和 GRANT 命令修改相应用户的权限。
数据库的默认数据配额和默认副本配额为2^63-1。
```

## 例子

1. 设置指定数据库的数据配额

    ```SQL
    ALTER DATABASE example_db SET DATA QUOTA 10995116277760;
    -- 以上单位为字节，等同于以下语句。
    ALTER DATABASE example_db SET DATA QUOTA 10T;
    ALTER DATABASE example_db SET DATA QUOTA 100G;
    ALTER DATABASE example_db SET DATA QUOTA 200M;
    ```

2. 将数据库 example_db 重命名为 example_db2

    ```SQL
    ALTER DATABASE example_db RENAME example_db2;
    ```

3. 设置指定数据库的副本配额

    ```SQL
    ALTER DATABASE example_db SET REPLICA QUOTA 102400;
    ```

## 引用

- [创建数据库](CREATE_DATABASE.md)
- [使用](../data-definition/USE.md)
- [显示数据库](../data-manipulation/SHOW_DATABASES.md)
- [DESC](../Utility/DESCRIBE.md)
- [删除数据库](../data-definition/DROP_DATABASE.md)
