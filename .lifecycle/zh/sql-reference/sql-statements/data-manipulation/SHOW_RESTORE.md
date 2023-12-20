---
displayed_sidebar: English
---

# 显示恢复任务

## 描述

查看指定数据库中最近一次的恢复（RESTORE）任务。欲了解更多信息，请参阅[data backup and restoration](../../../administration/Backup_and_restore.md)。

> **注意**
> StarRocks仅保存最后一次恢复任务的信息。

## 语法

```SQL
SHOW RESTORE [FROM <db_name>]
```

## 参数

|参数|说明|
|---|---|
|db_name|RESTORE 任务所属的数据库的名称。|

## 返回值

|返回|说明|
|---|---|
|JobId|唯一的作业 ID。|
|标签|数据快照的名称。|
|时间戳|备份时间戳。|
|DbName|RESTORE 任务所属的数据库的名称。|
|状态|RESTORE 任务的当前状态：PENDING：提交作业后的初始状态。SNAPSHOTING：正在执行本地快照。DOWNLOAD：正在提交快照下载任务。DOWNLOADING：正在下载快照。COMMIT：提交下载的快照。COMMITTING：正在提交下载的快照。已完成：恢复任务已完成。已取消：恢复任务失败或已取消。|
|AllowLoad|在 RESTORE 任务期间是否允许加载数据。|
|ReplicationNum|要恢复的副本数量。|
|RestoreObjs|恢复的对象（表和分区）。|
|CreateTime|任务提交时间。|
|MetaPreparedTime|本地元数据完成时间。|
|SnapshotFinishedTime|快照完成时间。|
|DownloadFinishedTime|快照下载完成时间。|
|FinishedTime|任务完成时间。|
|UnfinishedTasks|快照、下载和提交阶段中未完成的子任务 ID。|
|进度|快照下载任务的进度。|
|TaskErrMsg|错误消息。|
|状态|状态信息。|
|超时|任务超时。单位：秒。|

## 示例

示例 1：查看数据库 example_db 中最近一次的恢复任务。

```SQL
SHOW RESTORE FROM example_db;
```
