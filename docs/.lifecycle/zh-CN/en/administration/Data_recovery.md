---
displayed_sidebar: "Chinese"
---

# 数据恢复

StarRocks支持对错误删除的数据库/表/分区进行数据恢复。在执行`drop table`或`drop database`之后，StarRocks并不会立即物理删除数据，而是会将其保留在回收站中一段时间（默认为1天）。管理员可以使用`RECOVER`命令来恢复错误删除的数据。

## 相关命令

语法:

~~~sql
-- 1) 恢复数据库
RECOVER DATABASE db_name;
-- 2) 恢复表
RECOVER TABLE [db_name.]table_name;
-- 3) 恢复分区
RECOVER PARTITION partition_name FROM [db_name.]table_name;
~~~

## 注意事项

1. 此操作仅可恢复已删除的元信息。默认时间为1天，可通过`fe.conf`中的`catalog_trash_expire_second`参数进行配置。
2. 如果在删除元信息后创建了相同名称和类型的新元信息，则无法恢复先前已删除的元信息。

## 示例

1. 恢复名为`example_db`的数据库

    ~~~sql
    RECOVER DATABASE example_db;
    ~~~ 2.

2. 恢复名为`example_tbl`的表

    ~~~sql
    RECOVER TABLE example_db.example_tbl;
    ~~~ 3.

3. 恢复表`example_tbl`中名称为`p1`的分区

    ~~~sql
    RECOVER PARTITION p1 FROM example_tbl;
    ~~~