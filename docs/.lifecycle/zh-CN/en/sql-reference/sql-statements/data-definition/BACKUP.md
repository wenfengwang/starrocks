---
displayed_sidebar: "Chinese"
---

# 备份

## 描述

在指定的数据库、表或分区中备份数据。目前，StarRocks仅支持在OLAP表中备份数据。有关更多信息，请参见[数据备份和恢复](../../../administration/Backup_and_restore.md)。

备份是一项异步操作。您可以使用[SHOW BACKUP](../data-manipulation/SHOW_BACKUP.md)检查备份作业的状态，或使用[CANCEL BACKUP](../data-definition/CANCEL_BACKUP.md)取消备份作业。您可以使用[SHOW SNAPSHOT](../data-manipulation/SHOW_SNAPSHOT.md)查看快照信息。

> **注意**
>
> - 只有具有ADMIN权限的用户才能备份数据。
> - 在每个数据库中，每次只允许运行一个备份或恢复作业。否则，StarRocks将返回错误。
> - StarRocks不支持为数据备份指定数据压缩算法。

## 语法

```SQL
BACKUP SNAPSHOT <db_name>.<snapshot_name>
TO <repository_name>
[ ON ( <table_name> [ PARTITION ( <partition_name> [, ...] ) ]
       [, ...] ) ]
[ PROPERTIES ("key"="value" [, ...] ) ]
```

## 参数

| **参数**         | **描述**                                                     |
| --------------- | ------------------------------------------------------------ |
| db_name         | 存储要备份数据的数据库名称。                                   |
| snapshot_name   | 为数据快照指定一个名称。全局唯一。                           |
| repository_name | 仓库名称。您可以使用[CREATE REPOSITORY](../data-definition/CREATE_REPOSITORY.md)创建仓库。 |
| ON              | 要备份的表的名称。如果未指定此参数，则备份整个数据库。        |
| PARTITION       | 要备份的分区的名称。如果未指定此参数，则备份整个表。          |
| PROPERTIES      | 数据快照的属性。有效的键：`type`：备份类型。当前仅支持全备份`FULL`。默认值：`FULL`。`timeout`：任务超时。单位：秒。默认值：`86400`。 |

## 示例

示例1：备份数据库`example_db`到仓库`example_repo`。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label1
TO example_repo
PROPERTIES ("type" = "full");
```

示例2：备份`example_db`中的表`example_tbl`到`example_repo`。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label2
TO example_repo
ON (example_tbl);
```

示例3：备份`example_db`中`example_tbl`的分区`p1`和`p2`及表`example_tbl2`到`example_repo`。

```SQL
BACKUP SNAPSHOT example_db.snapshot_label3
TO example_repo
ON(
    example_tbl PARTITION (p1, p2),
    example_tbl2
);
```