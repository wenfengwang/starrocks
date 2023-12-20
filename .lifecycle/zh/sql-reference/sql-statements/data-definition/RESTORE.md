---
displayed_sidebar: English
---

# 数据恢复

## 描述

将数据恢复到指定的数据库、表或分区。目前，StarRocks 仅支持恢复数据至 OLAP 表。更多信息，请参考[data backup and restoration](../../../administration/Backup_and_restore.md)。

RESTORE 是一项异步操作。您可以使用 [SHOW RESTORE](../data-manipulation/SHOW_RESTORE.md) 命令来检查 RESTORE 作业的状态，或使用 [CANCEL RESTORE](../data-definition/CANCEL_RESTORE.md) 命令来取消 RESTORE 作业。

> **注意**
- 只有拥有 ADMIN 权限的用户才能执行数据恢复。
- 在每个数据库中，同一时间只允许执行一个 BACKUP 或 RESTORE 作业。否则，StarRocks 将报错。

## 语法

```SQL
RESTORE SNAPSHOT <db_name>.<snapshot_name>
FROM <repository_name>
[ ON ( <table_name> [ PARTITION ( <partition_name> [, ...] ) ]
    [ AS <table_alias>] [, ...] ) ]
PROPERTIES ("key"="value", ...)
```

## 参数

|参数|说明|
|---|---|
|db_name|数据恢复到的数据库的名称。|
|snapshot_name|数据快照的名称。|
|repository_name|存储库名称。|
|ON|要恢复的表的名称。如果未指定此参数，则恢复整个数据库。|
|PARTITION|要恢复的分区的名称。如果不指定该参数，则恢复整个表。您可以使用 SHOW PARTITIONS 查看分区名称。|
|属性|RESTORE 操作的属性。有效键：backup_timestamp：备份时间戳。必需的。您可以使用 SHOW SNAPSHOT 查看备份时间戳。replication_num：指定要恢复的副本数量。默认值：3.meta_version：该参数仅作为恢复早期版本StarRocks备份的数据的临时解决方案。备份数据的最新版本已包含meta版本，无需指定。timeout：任务超时时间。单位：第二。默认值：86400。|

## 示例

示例 1：从 example_repo 仓库中，将 snapshot_label1 快照中的表 backup_tbl 恢复到数据库 example_db，备份时间戳为 2018-05-04-16-45-08。恢复一个副本。

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

示例 2：从 example_repo 仓库中，将 snapshot_label2 快照中的表 backup_tbl 以及表 backup_tbl2 的分区 p1 和 p2 恢复到数据库 example_db，并将 backup_tbl2 重命名为 new_tbl。备份时间戳为 2018-05-04-17-11-01。默认情况下恢复三个副本。

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
