---
displayed_sidebar: English
---

# 取消恢复操作

## 描述

取消一个正在进行的RESTORE任务在指定的数据库中。更多信息，请参阅[data backup and restoration](../../../administration/Backup_and_restore.md)。

> **注意**
> 如果在提交（COMMIT）阶段取消恢复任务，那么已恢复的数据将会损坏并且无法访问。在这种情况下，您只能重新执行**恢复**操作，并等待任务完成。

## 语法

```SQL
CANCEL RESTORE FROM <db_name>
```

## 参数

|参数|说明|
|---|---|
|db_name|RESTORE 任务所属的数据库的名称。|

## 示例

示例 1：取消在数据库example_db中的恢复任务。

```SQL
CANCEL RESTORE FROM example_db;
```
