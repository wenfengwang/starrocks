---
displayed_sidebar: "Chinese"
---

# 显示备份

## 描述

查看指定数据库中的最后一个备份任务。有关更多信息，请参见 [数据备份和还原](../../../administration/Backup_and_restore.md)。

> **注意**
>
> 仅保存最后一个备份任务的信息。 

## 语法

```SQL
SHOW BACKUP [FROM <db_name>]
```

## 参数

| **参数**   | **描述**                           |
| ---------- | ---------------------------------- |
| db_name    | 备份任务所属数据库的名称。        |

## 返回

| **返回**             | **描述**                                                |
| -------------------- | ------------------------------------------------------ |
| JobId                | 唯一的作业标识符。                                      |
| SnapshotName         | 数据快照的名称。                                        |
| DbName               | 备份任务所属数据库的名称。                              |
| State                | 备份任务的当前状态：<ul><li>PENDING：提交作业后的初始状态。</li><li>SNAPSHOTING：创建快照。</li><li>UPLOAD_SNAPSHOT：快照完成，准备上传。</li><li>UPLOADING：上传快照。</li><li>SAVE_META：创建本地元数据文件。</li><li>UPLOAD_INFO：上传元数据文件和备份任务的信息。</li><li>FINISHED：备份任务完成。</li><li>CANCELLED：备份任务失败或取消。</li></ul> |
| BackupObjs           | 备份的对象。                                           |
| CreateTime           | 任务提交时间。                                         |
| SnapshotFinishedTime | 快照完成时间。                                         |
| UploadFinishedTime   | 快照上传完成时间。                                     |
| FinishedTime         | 任务完成时间。                                         |
| UnfinishedTasks      | 创建快照和上传快照阶段中未完成的子任务ID。               |
| Progress             | 快照上传任务的进度。                                 |
| TaskErrMsg           | 错误消息。                                           |
| Status               | 状态信息。                                           |
| Timeout              | 任务超时。单位：秒。                                  |

## 示例

示例 1：查看数据库`example_db`中的最后一个备份任务。

```SQL
SHOW BACKUP FROM example_db;
```