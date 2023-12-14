```yaml
---
displayed_sidebar: "Chinese"
---

# 取消恢复

## 功能

取消指定数据库中正在进行的恢复任务。有关更多信息，请参见 [备份和恢复](../../../administration/Backup_and_restore.md)。

> **注意**
>
> 如果在 COMMIT 阶段取消恢复作业，已恢复的数据将会损坏，且无法访问。这种情况下，只能通过再次执行恢复操作，并等待作业完成。

## 语法

```SQL
CANCEL RESTORE FROM <db_name>
```

## 参数说明

| **参数** | **说明**               |
| -------- | ---------------------- |
| db_name  | 恢复任务所属数据库名。 |

## 示例

示例一：取消 `example_db` 数据库下的恢复任务。

```SQL
CANCEL RESTORE FROM example_db;
```