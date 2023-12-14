```yaml
---
displayed_sidebar: "Chinese"
---

# 取消备份

## 功能

取消指定数据库中正在进行的备份任务。有关更多信息，请参见[备份和恢复](../../../administration/Backup_and_restore.md)。

## 语法

```SQL
CANCEL BACKUP FROM <db_name>
```

## 参数说明

| **参数** | **说明**               |
| -------- | ---------------------- |
| db_name  | 备份任务所属数据库名称。 |

## 示例

示例一：取消`example_db`数据库中的备份任务。

```SQL
CANCEL BACKUP FROM example_db;
```