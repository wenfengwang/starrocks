---
displayed_sidebar: "Chinese"
---

# 取消备份

## 描述

取消指定数据库中正在进行的备份任务。有关更多信息，请参阅[数据备份和恢复](../../../administration/Backup_and_restore.md)。

## 语法

```SQL
CANCEL BACKUP FROM <db_name>
```

## 参数

| **参数**   | **描述**                       |
| ---------- | ------------------------------- |
| db_name    | BACKUP 任务所属的数据库名称。 |

## 示例

示例 1：取消`example_db`数据库下的备份任务。

```SQL
CANCEL BACKUP FROM example_db;
```