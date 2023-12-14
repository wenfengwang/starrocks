---
displayed_sidebar: "Chinese"
---

# 恢复

## 描述

此语句用于恢复已删除的数据库、表或分区。

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

1. 只能恢复一段时间前删除的元信息。默认时间为一天（您可以通过fe.conf中的参数配置catalog_trash_expire_second来更改它。）
2. 如果已删除的元信息与一个相同的已创建的元信息相同，先前的元信息将不会被恢复。

## 示例

1. 恢复名为example_db的数据库

    ```sql
    RECOVER DATABASE example_db;
    ```

2. 恢复名为example_tbl的表

    ```sql
    RECOVER TABLE example_db.example_tbl;
    ```

3. 恢复example_tbl表中名为p1的分区

    ```sql
    RECOVER PARTITION p1 FROM example_tbl;
    ```