---
displayed_sidebar: English
---

# 显示备份

## 描述

查看指定数据库中的最后一个 **BACKUP** 任务。更多信息，请参见[数据备份和恢复](../../../administration/Backup_and_restore.md)。

> **注意**
> 在StarRocks中，仅保存最后一次**BACKUP**任务的信息。

## 语法

```SQL
SHOW BACKUP [FROM <db_name>]
```

## 参数

|**参数**|**说明**|
|---|---|
|db_name|**BACKUP** 任务所属的数据库名称。|

## 返回值

|**返回**|**说明**|
|---|---|
|JobId|唯一的作业ID。|
|SnapshotName|数据快照的名称。|
|DbName|**BACKUP** 任务所属的数据库名称。|
|State|**BACKUP** 任务的当前状态：<ul><li>PENDING：提交作业后的初始状态。</li><li>SNAPSHOTING：正在创建快照。</li><li>UPLOAD_SNAPSHOT：快照完成，准备上传。</li><li>UPLOADING：正在上传快照。</li><li>SAVE_META：正在创建本地元数据文件。</li><li>UPLOAD_INFO：正在上传元数据文件和**BACKUP** 任务的信息。</li><li>FINISHED：**BACKUP** 任务完成。</li><li>CANCELLED：**BACKUP** 任务失败或已取消。</li></ul>|
|BackupObjs|已备份的对象。|
|CreateTime|任务提交时间。|
|SnapshotFinishedTime|快照完成时间。|
|UploadFinishedTime|快照上传完成时间。|
|FinishedTime|任务完成时间。|
|UnfinishedTasks|在SNAPSHOTING和UPLOADING阶段中未完成的子任务ID。|
|Progress|快照上传任务的进度。|
|TaskErrMsg|错误信息。|
|Status|状态信息。|
|Timeout|任务超时时间。单位：秒。|

## 示例

示例 1：查看数据库 `example_db` 中的最后一个 **BACKUP** 任务。

```SQL
SHOW BACKUP FROM example_db;
```