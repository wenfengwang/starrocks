---
displayed_sidebar: English
---

# 恢复

## 描述

该语句用于恢复已删除的数据库、表或分区。

语法：

1. 恢复数据库

    ```sql
    RECOVER DATABASE <db_name>
    ```

2. 恢复表

    ```sql
    RECOVER TABLE [<db_name>.]<table_name>
    ```

3. 恢复分区

    ```sql
    RECOVER PARTITION partition_name FROM [<db_name>.]<table_name>
    ```

注意：

1. 它只能恢复一段时间前删除的元信息。默认时间为一天。（您可以通过在 fe.conf 中配置 catalog_trash_expire_second 参数来进行更改。）
2. 如果删除的元信息已经被创建相同的元信息替代，先前的元信息将不会被恢复。

## 例子

1. 恢复名为 example_db 的数据库

    ```sql
    RECOVER DATABASE example_db;
    ```

2. 恢复名为 example_tbl 的表

    ```sql
    RECOVER TABLE example_db.example_tbl;
    ```

3. 从 example_tbl 表中恢复名为 p1 的分区

    ```sql
    RECOVER PARTITION p1 FROM example_tbl;
    ```
