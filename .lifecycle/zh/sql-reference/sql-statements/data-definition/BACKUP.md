---
displayed_sidebar: English
---

# 备份

## 描述

备份指定数据库、表或分区中的数据。目前，StarRocks 仅支持备份 OLAP 表中的数据。更多信息，请参阅[data backup and restoration](../../../administration/Backup_and_restore.md)。

BACKUP 是一个异步操作。您可以使用 [SHOW BACKUP](../data-manipulation/SHOW_BACKUP.md) 命令检查备份作业的状态，或使用 [CANCEL BACKUP](../data-definition/CANCEL_BACKUP.md) 命令取消备份作业。您还可以使用 [SHOW SNAPSHOT](../data-manipulation/SHOW_SNAPSHOT.md) 命令查看快照信息。

> **注意**
- 只有拥有 ADMIN 权限的用户才能进行数据备份。
- 在每个数据库中，同一时间只允许有一个 BACKUP 或 RESTORE 作业在运行。否则，StarRocks 将报错。
- StarRocks 不支持为数据备份指定数据压缩算法。

## 语法

```SQL
BACKUP SNAPSHOT <db_name>.<snapshot_name>
TO <repository_name>
[ ON ( <table_name> [ PARTITION ( <partition_name> [, ...] ) ]
       [, ...] ) ]
[ PROPERTIES ("key"="value" [, ...] ) ]
```

## 参数

|参数|说明|
|---|---|
|db_name|存储要备份的数据的数据库的名称。|
|snapshot_name|指定数据快照的名称。全球独一无二。|
|repository_name|存储库名称。您可以使用 CREATE REPOSITORY 创建存储库。|
|ON|要备份的表的名称。如果不指定该参数，则备份整个数据库。|
|PARTITION|要备份的分区的名称。如果不指定该参数，则备份整个表。|
|属性|数据快照的属性。有效键：类型：备份类型。目前仅支持全量备份FULL。默认值：FULL。timeout：任务超时。单位：第二。默认值：86400。|

## 示例

示例 1：将数据库 example_db 备份到仓库 example_repo。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label1
TO example_repo
PROPERTIES ("type" = "full");
```

示例 2：将 example_db 中的表 example_tbl 备份到仓库 example_repo。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label2
TO example_repo
ON (example_tbl);
```

示例 3：将 example_db 中的表 example_tbl 的分区 p1 和 p2 以及表 example_tbl2 备份到仓库 example_repo。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label3
TO example_repo
ON(
    example_tbl PARTITION (p1, p2),
    example_tbl2
);
```
