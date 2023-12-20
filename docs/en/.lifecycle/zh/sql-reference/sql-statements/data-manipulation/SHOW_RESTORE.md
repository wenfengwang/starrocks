---
displayed_sidebar: English
---

# SHOW RESTORE

## 描述

查看指定数据库中的最后一个 RESTORE 任务。更多信息，请参见[数据备份与恢复](../../../administration/Backup_and_restore.md)。

> **注意**
> 在StarRocks中，仅保存最后一次RESTORE任务的信息。

## 语法

```SQL
SHOW RESTORE [FROM <db_name>]
```

## 参数

|**参数**|**说明**|
|---|---|
|db_name|RESTORE 任务所属的数据库名称。|

## 返回值

|**返回**|**说明**|
|---|---|
|JobId|唯一作业ID。|
|Label|数据快照的名称。|
|Timestamp|备份时间戳。|
|DbName|RESTORE 任务所属的数据库名称。|
|State|RESTORE 任务的当前状态：<ul><li>PENDING：提交作业后的初始状态。</li><li>SNAPSHOTING：正在执行本地快照。</li><li>DOWNLOAD：提交快照下载任务。</li><li>DOWNLOADING：正在下载快照。</li><li>COMMIT：准备提交下载的快照。</li><li>COMMITTING：正在提交下载的快照。</li><li>FINISHED：RESTORE 任务已完成。</li><li>CANCELLED：RESTORE 任务失败或已取消。</li></ul>|
|AllowLoad|在 RESTORE 任务期间是否允许加载数据。|
|ReplicationNum|要恢复的副本数。|
|RestoreObjs|恢复的对象（表和分区）。|
|CreateTime|任务提交时间。|
|MetaPreparedTime|本地元数据准备完成时间。|
|SnapshotFinishedTime|快照完成时间。|
|DownloadFinishedTime|快照下载完成时间。|
|FinishedTime|任务完成时间。|
|UnfinishedTasks|SNAPSHOTING、DOWNLOADING 和 COMMITTING 阶段中未完成的子任务ID。|
|Progress|快照下载任务的进度。|
|TaskErrMsg|错误信息。|
|Status|状态信息。|
|Timeout|任务超时时间。单位：秒。|

## 示例

示例 1：查看数据库 `example_db` 中的最后一个 RESTORE 任务。

```SQL
SHOW RESTORE FROM example_db;
```