---
displayed_sidebar: English
---

# 取消备份

## 说明

取消一个正在进行的**备份**任务在指定的数据库中。欲了解更多信息，请参阅[data backup and restoration](../../../administration/Backup_and_restore.md)。

## 语法

```SQL
CANCEL BACKUP FROM <db_name>
```

## 参数

|参数|说明|
|---|---|
|db_name|BACKUP 任务所属的数据库的名称。|

## 示例

示例 1：取消 example_db 数据库中的备份任务。

```SQL
CANCEL BACKUP FROM example_db;
```
