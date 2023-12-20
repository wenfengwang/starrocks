---
displayed_sidebar: English
---

# 取消恢复

## 描述

取消指定数据库中正在进行的恢复任务。更多信息，请参见[数据备份与恢复](../../../administration/Backup_and_restore.md)。

> **警告**
> 如果在 COMMIT 阶段取消 **RESTORE** 任务，那么恢复的数据将会损坏并且无法访问。在这种情况下，您只能重新执行**RESTORE**操作并等待作业完成。

## 语法

```SQL
CANCEL RESTORE FROM <db_name>
```

## 参数

|**参数**|**描述**|
|---|---|
|db_name|所属恢复任务的数据库名称。|

## 示例

示例 1：取消数据库 `example_db` 下的恢复任务。

```SQL
CANCEL RESTORE FROM example_db;
```