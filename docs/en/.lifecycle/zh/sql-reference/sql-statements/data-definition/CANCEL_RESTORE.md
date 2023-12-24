---
displayed_sidebar: English
---

# 取消还原

## 描述

取消指定数据库中正在进行的还原任务。更多信息，请参见 [数据备份和恢复](../../../administration/Backup_and_restore.md)。

> **注意**
>
> 如果在提交阶段取消还原任务，则恢复的数据将损坏且无法访问。在这种情况下，您只能重新执行还原操作并等待作业完成。

## 语法

```SQL
CANCEL RESTORE FROM <db_name>
```

## 参数

| **参数** | **描述**                                        |
| ------------- | ------------------------------------------------------ |
| db_name       | 还原任务所属的数据库的名称。 |

## 例子

示例 1：取消数据库中的还原任务 `example_db`。

```SQL
CANCEL RESTORE FROM example_db;