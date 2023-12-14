---
displayed_sidebar: "中文"
---

# 取消恢复

## 描述

取消指定数据库中正在进行的恢复任务。有关更多信息，请参阅[数据备份和恢复](../../../administration/Backup_and_restore.md)。

> **注意**
>
> 如果在提交阶段取消恢复任务，则恢复的数据将损坏并且无法访问。在这种情况下，您只能重新执行恢复操作并等待作业完成。

## 语法

```SQL
CANCEL RESTORE FROM <db_name>
```

## 参数

| **参数**    | **描述**                       |
| ----------- | ------------------------------ |
| db_name     | 该恢复任务所属数据库的名称。   |

## 示例

示例 1：取消 `example_db` 数据库中的恢复任务。

```SQL
CANCEL RESTORE FROM example_db;
```