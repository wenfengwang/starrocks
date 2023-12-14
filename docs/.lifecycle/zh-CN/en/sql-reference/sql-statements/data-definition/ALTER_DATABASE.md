---
displayed_sidebar: "Chinese"
---

# ALTER DATABASE

## 描述

配置指定数据库的属性。(仅限管理员)

1. 设置数据库数据配额，单位为B/K/KB/M/MB/G/GB/T/TB/P/PB。

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
重命名数据库后，如有必要，使用REVOKE和GRANT命令修改相应的用户权限。
数据库的默认数据配额和默认副本配额均为2^63-1。
```

## 示例

1. 设置指定数据库数据配额

    ```SQL
    ALTER DATABASE example_db SET DATA QUOTA 10995116277760;
    -- 以上单位为字节，等同于以下语句。
    ALTER DATABASE example_db SET DATA QUOTA 10T;
    ALTER DATABASE example_db SET DATA QUOTA 100G;
    ALTER DATABASE example_db SET DATA QUOTA 200M;
    ```

2. 将数据库example_db重命名为example_db2

    ```SQL
    ALTER DATABASE example_db RENAME example_db2;
    ```

3. 设置指定数据库副本配额

    ```SQL
    ALTER DATABASE example_db SET REPLICA QUOTA 102400;
    ```

## 参考

- [创建数据库](CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [显示数据库](../data-manipulation/SHOW_DATABASES.md)
- [DESC](../Utility/DESCRIBE.md)
- [删除数据库](../data-definition/DROP_DATABASE.md)