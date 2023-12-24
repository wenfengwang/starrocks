---
displayed_sidebar: English
---

# 恢复

## 描述

将数据恢复到指定的数据库、表或分区。目前，StarRocks仅支持将数据恢复到OLAP表。有关更多信息，请参见[数据备份和恢复](../../../administration/Backup_and_restore.md)。

RESTORE是一个异步操作。您可以使用[SHOW RESTORE](../data-manipulation/SHOW_RESTORE.md)来检查RESTORE作业的状态，或者使用[CANCEL RESTORE](../data-definition/CANCEL_RESTORE.md)来取消RESTORE作业。

> **注意**
>
> - 只有具有ADMIN权限的用户才能恢复数据。
> - 在每个数据库中，每次只允许运行一个BACKUP或RESTORE作业。否则，StarRocks会返回错误。

## 语法

```SQL
RESTORE SNAPSHOT <db_name>.<snapshot_name>
FROM <repository_name>
[ ON ( <table_name> [ PARTITION ( <partition_name> [, ...] ) ]
    [ AS <table_alias>] [, ...] ) ]
PROPERTIES ("key"="value", ...)
```

## 参数

| **参数**   | **描述**                                              |
| --------------- | ------------------------------------------------------------ |
| db_name         | 要将数据恢复到的数据库名称。           |
| snapshot_name   | 数据快照的名称。                                  |
| repository_name | 存储库名称。                                             |
| ON              | 要恢复的表的名称。如果未指定此参数，则将恢复整个数据库。 |
| PARTITION       | 要恢复的分区的名称。如果未指定此参数，则将恢复整个表。您可以使用[SHOW PARTITIONS](../data-manipulation/SHOW_PARTITIONS.md)查看分区名称。 |
| PROPERTIES      | RESTORE操作的属性。有效键：<ul><li>`backup_timestamp`：备份时间戳。**必填**。您可以使用[SHOW SNAPSHOT](../data-manipulation/SHOW_SNAPSHOT.md)查看备份时间戳。</li><li>`replication_num`：指定要恢复的副本数。默认值：`3`。</li><li>`meta_version`：此参数仅用作恢复StarRocks早期版本备份的临时解决方案。备份数据的最新版本已包含`meta version`，因此无需指定此参数。</li><li>`timeout`：任务超时。单位：秒。默认值：`86400`。</li></ul> |

## 例子

示例1：将快照`snapshot_label1`中的表`backup_tbl`从存储库`example_repo`还原到数据库`example_db`，备份时间戳为`2018-05-04-16-45-08`，并恢复一个副本。

```SQL
RESTORE SNAPSHOT example_db.snapshot_label1
FROM example_repo
ON (backup_tbl)
PROPERTIES
(
    "backup_timestamp"="2018-05-04-16-45-08",
    "replication_num"="1"
);
```

示例2：将存储库`example_repo`中`snapshot_label2`的表`backup_tbl`的分区`p1`和`p2`以及表`backup_tbl2`还原到数据库`example_db`，并将表`backup_tbl2`重命名为`new_tbl`。备份时间戳为`2018-05-04-17-11-01`。默认情况下恢复三个副本。

```SQL
RESTORE SNAPSHOT example_db.snapshot_label2
FROM example_repo
ON (
    backup_tbl PARTITION (p1, p2),
    backup_tbl2 AS new_tbl
)
PROPERTIES
(
    "backup_timestamp"="2018-05-04-17-11-01"
);
```
