---
displayed_sidebar: "Chinese"
---

# 恢复

## 描述

将数据恢复到指定的数据库、表或分区。当前，StarRocks 仅支持将数据恢复到 OLAP 表中。有关更多信息，请参见[数据备份和恢复](../../../administration/Backup_and_restore.md)。

RESTORE 是一个异步操作。您可以使用 [SHOW RESTORE](../data-manipulation/SHOW_RESTORE.md) 来检查 RESTORE 作业的状态，或者使用 [CANCEL RESTORE](../data-definition/CANCEL_RESTORE.md) 来取消 RESTORE 作业。

> **注意**
>
> - 只有具有 ADMIN 权限的用户才能恢复数据。
> - 每个数据库一次只允许运行一个 BACKUP 或 RESTORE 作业。否则，StarRocks 将返回错误。

## 语法

```SQL
RESTORE SNAPSHOT <db_name>.<snapshot_name>
FROM <repository_name>
[ ON ( <table_name> [ PARTITION ( <partition_name> [, ...] ) ]
    [ AS <table_alias>] [, ...] ) ]
PROPERTIES ("key"="value", ...)
```

## 参数

| **参数**          | **描述**                                                      |
| --------------- | ------------------------------------------------------------ |
| db_name         | 要恢复数据到的数据库名称。                                        |
| snapshot_name   | 数据快照的名称。                                               |
| repository_name | 仓库的名称。                                                   |
| ON              | 要恢复的表的名称。如果未指定此参数，则将恢复整个数据库。                 |
| PARTITION       | 要恢复的分区的名称。如果未指定此参数，则将恢复整个表。您可以使用 [SHOW PARTITIONS](../data-manipulation/SHOW_PARTITIONS.md) 查看分区名称。 |
| PROPERTIES      | 恢复操作的属性。有效的键：<ul><li>`backup_timestamp`: 备份时间戳。**必需**。您可以使用 [SHOW SNAPSHOT](../data-manipulation/SHOW_SNAPSHOT.md) 查看备份时间戳。</li><li>`replication_num`: 指定要恢复的副本数。默认值：`3`。</li><li>`meta_version`: 此参数仅作为恢复早期版本 StarRocks 备份的临时解决方案。最新版本的备份数据已包含 `meta version`，因此您无需指定它。</li><li>`timeout`: 任务超时。单位：秒。默认值：`86400`。</li></ul> |

## 示例

示例 1：将名为 `snapshot_label1` 的快照中的表 `backup_tbl` 从存储库 `example_repo` 恢复到数据库 `example_db`，备份时间戳为 `2018-05-04-16-45-08`，并恢复一个副本。

```SQL
RESTORE SNAPSHOT example_db.snapshot_label1
FROM example_repo
ON ( backup_tbl )
PROPERTIES
(
    "backup_timestamp"="2018-05-04-16-45-08",
    "replication_num" = "1"
);
```

示例 2：将名为 `snapshot_label2` 的快照中的表 `backup_tbl` 的分区 `p1` 和 `p2`，以及表 `backup_tbl2` 从存储库 `example_repo` 恢复到数据库 `example_db`，并将 `backup_tbl2` 重命名为 `new_tbl`。备份时间戳为 `2018-05-04-17-11-01`，默认恢复三个副本。

```SQL
RESTORE SNAPSHOT example_db.snapshot_label2
FROM example_repo
ON(
    backup_tbl PARTITION (p1, p2),
    backup_tbl2 AS new_tbl
)
PROPERTIES
(
    "backup_timestamp"="2018-05-04-17-11-01"
);
```