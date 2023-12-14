```yaml
---
displayed_sidebar: "中文"
---

# 显示还原信息

## 描述

查看在指定数据库中的最后一个还原任务。有关更多信息，请参见[数据备份和恢复](../../../administration/Backup_and_restore.md)。

> **注意**
>
> 只有最后一个还原任务的信息才会保存在StarRocks中。

## 语法

```SQL
SHOW RESTORE [FROM <db_name>]
```

## 参数

| **参数**      | **描述**                                            |
| ------------- | ---------------------------------------------------- |
| db_name       | 还原任务所属数据库的名称。                           |

## 返回

| **返回**                | **描述**                                                      |
| ----------------------- | ------------------------------------------------------------ |
| JobId                   | 唯一的作业ID。                                                 |
| Label                   | 数据快照的名称。                                               |
| Timestamp               | 备份时间戳。                                                   |
| DbName                  | 还原任务所属数据库的名称。                                     |
| State                   | 还原任务的当前状态：<ul><li>PENDING：提交作业后的初始状态。</li><li>SNAPSHOTING：执行本地快照。</li><li>DOWNLOAD：提交快照下载任务。</li><li>DOWNLOADING：下载快照。</li><li>COMMIT：提交已下载的快照。</li><li>COMMITTING：提交已下载的快照。</li><li>FINISHED：还原任务已完成。</li><li>CANCELLED：还原任务失败或已取消。</li></ul> |
| AllowLoad               | 是否允许在还原任务期间加载数据。                                 |
| ReplicationNum          | 待还原的副本数量。                                             |
| RestoreObjs             | 已还原的对象（表和分区）。                                     |
| CreateTime              | 任务提交时间。                                                 |
| MetaPreparedTime        | 本地元数据完成时间。                                           |
| SnapshotFinishedTime    | 快照完成时间。                                                 |
| DownloadFinishedTime    | 快照下载完成时间。                                             |
| FinishedTime            | 任务完成时间。                                                 |
| UnfinishedTasks         | 快照下载任务和提交任务阶段中未完成的子任务ID。                   |
| Progress                | 快照下载任务的进度。                                           |
| TaskErrMsg              | 错误消息。                                                     |
| Status                  | 状态信息。                                                     |
| Timeout                 | 任务超时时间。单位：秒。                                       |

## 示例

示例 1：查看数据库`example_db`中的最后一个还原任务。

```SQL
SHOW RESTORE FROM example_db;
```