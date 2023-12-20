---
displayed_sidebar: English
---

# 查看备份

## 描述

Views the last BACKUP task in a specified database. For more information, see [数据备份与恢复](../../../administration/Backup_and_restore.md).

> **注意**
> StarRocks仅保存最新一次备份任务的信息。

## 语法

```SQL
SHOW BACKUP [FROM <db_name>]
```

## 参数

|参数|说明|
|---|---|
|db_name|BACKUP 任务所属的数据库的名称。|

## 返回值

|返回|说明|
|---|---|
|JobId|唯一的作业 ID。|
|SnapshotName|数据快照的名称。|
|DbName|BACKUP 任务所属的数据库的名称。|
|State|BACKUP 任务的当前状态：PENDING：提交作业后的初始状态。SNAPSHOTING：正在创建快照。UPLOAD_SNAPSHOT：快照完成，准备上传。UPLOADING：正在上传快照。SAVE_META：正在创建本地元数据文件。UPLOAD_INFO：正在上传元数据文件以及备份任务的信息。已完成：备份任务已完成。已取消：备份任务失败或已取消。|
|BackupObjs|备份对象。|
|CreateTime|任务提交时间。|
|SnapshotFinishedTime|快照完成时间。|
|UploadFinishedTime|快照上传完成时间。|
|FinishedTime|任务完成时间。|
|UnfinishedTasks|快照和上传阶段中未完成的子任务 ID。|
|进度|快照上传任务的进度。|
|TaskErrMsg|错误消息。|
|状态|状态信息。|
|超时|任务超时。单位：秒。|

## 示例

示例 1：查看数据库example_db中的最新备份任务。

```SQL
SHOW BACKUP FROM example_db;
```
