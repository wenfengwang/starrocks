---
displayed_sidebar: "Chinese"
---

# 备份与恢复

This article describes how to backup and restore data in StarRocks, or migrate data to a new StarRocks cluster.

StarRocks supports backing up data to a remote storage system in the form of snapshot files, or restoring backed-up data from a remote storage system to any StarRocks cluster. With this feature, you can regularly snapshot the data in the StarRocks cluster, or migrate data between different StarRocks clusters.

StarRocks supports backing up data in the following external storage systems:

- Apache™ Hadoop® (HDFS) cluster
- AWS S3
- Google GCS
- Alibaba Cloud OSS
- Tencent Cloud COS

## Backup Data

StarRocks supports full backup of data at the granularity of databases, tables, or partitions.

When the data volume of a table is large, it is recommended to perform backups separately by partition to reduce the cost of failed retries. If you need to backup the data regularly, it is recommended to define [dynamic partition](../table_design/dynamic_partitioning.md) strategy when creating tables, so that you can regularly back up the data in the newly added partitions in the later operation and maintenance process.

### Create Repository

A repository is used to store backup files in a remote storage system. Before backing up data, you need to create a repository in StarRocks based on the path of the remote storage system. You can create multiple repositories in the same cluster. For detailed usage, see [CREATE REPOSITORY](../sql-reference/sql-statements/data-definition/CREATE_REPOSITORY.md).

- Create a repository in the HDFS cluster

The following example creates a repository `test_repo` in the Apache™ Hadoop® cluster.

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "hdfs://<hdfs_host>:<hdfs_port>/repo_dir/backup"
PROPERTIES(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

- Create a repository in AWS S3

  You can choose to use IAM user credentials (Access Key and Secret Key), Instance Profile, or Assumed Role as security credentials for accessing Amazon S3.

  - The following example uses IAM user credentials as security credentials to create a repository `test_repo` in the Amazon S3 bucket `bucket_s3`.

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.access_key" = "XXXXXXXXXXXXXXXXX",
      "aws.s3.secret_key" = "yyyyyyyyyyyyyyyyyyyyyyyy",
      "aws.s3.endpoint" = "s3.us-east-1.amazonaws.com"
  );
  ```

  - The following example uses Instance Profile as security credentials to create a repository `test_repo` in the Amazon S3 bucket `bucket_s3`.

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-east-1"
  );
  ```

  - The following example uses Assumed Role as security credentials to create a repository `test_repo` in the Amazon S3 bucket `bucket_s3`.

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::xxxxxxxxxx:role/yyyyyyyy",
      "aws.s3.region" = "us-east-1"
  );
  ```

> **Note**
>
> StarRocks only supports creating repositories in AWS S3 using the S3A protocol. Therefore, when creating a repository in AWS S3, you must replace `s3://` with `s3a://` in the `ON LOCATION` parameter.

- Create a repository in Google GCS

The following example creates a repository `test_repo` in the Google GCS bucket `bucket_gcs`.

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "s3a://bucket_gcs/backup"
PROPERTIES(
    "fs.s3a.access.key" = "xxxxxxxxxxxxxxxxxxxx",
    "fs.s3a.secret.key" = "yyyyyyyyyyyyyyyyyyyy",
    "fs.s3a.endpoint" = "storage.googleapis.com"
);
```

> **Note**
>
> StarRocks only supports creating repositories in Google GCS using the S3A protocol. Therefore, when creating a repository in Google GCS, you must replace the prefix of the GCS URI with `s3a://` in the `ON LOCATION` parameter.

- Create a repository in Alibaba Cloud OSS

The following example creates a repository `test_repo` in the Alibaba Cloud OSS bucket `bucket_oss`.

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "oss://bucket_oss/backup"
PROPERTIES(
    "fs.oss.accessKeyId" = "xxxxxxxxxxxxxxxxxxxxxxxxxx",
    "fs.oss.accessKeySecret" = "yyyyyyyyyyyyyyyyyyyy",
    "fs.oss.endpoint" = "oss-cn-zhangjiakou-internal.aliyuncs.com"
);
```

- Create a repository in Tencent Cloud COS

The following example creates a repository `test_repo` in the Tencent Cloud COS bucket `bucket_cos`.

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "cosn://bucket_cos/backup"
PROPERTIES(
    "fs.cosn.userinfo.secretId" = "xxxxxxxxxxxxxxxxx",
    "fs.cosn.userinfo.secretKey" = "yyyyyyyyyyyyyyyy",
    "fs.cosn.bucket.endpoint_suffix" = "cos.ap-beijing.myqcloud.com"
);
```

After the repository is created, you can use [SHOW REPOSITORIES](../sql-reference/sql-statements/data-manipulation/SHOW_REPOSITORIES.md) to view the created repositories. After the data recovery is completed, you can use the [DROP REPOSITORY](../sql-reference/sql-statements/data-definition/DROP_REPOSITORY.md) statement to delete the repository in StarRocks. However, the snapshot data backed up in the remote storage system cannot be directly deleted through StarRocks at present, and you need to manually delete the snapshot path of the backup in the remote storage system.

### Backup Data Snapshot

After creating the data repository, you can use the [BACKUP](../sql-reference/sql-statements/data-definition/BACKUP.md) command to create a data snapshot and back it up to the remote repository.

The following example creates a data snapshot `sr_member_backup` for the table `sr_member` in the database `sr_hub` and backs it up to the repository `test_repo`.

```SQL
BACKUP SNAPSHOT sr_hub.sr_member_backup
TO test_repo
ON (sr_member);
```

Data backup is an asynchronous operation. You can use the [SHOW BACKUP](../sql-reference/sql-statements/data-manipulation/SHOW_BACKUP.md) statement to view the backup job status, or use the [CANCEL BACKUP](../sql-reference/sql-statements/data-definition/CANCEL_BACKUP.md) statement to cancel the backup job.

## Restore or Migrate Data

You can restore the data snapshot backed up to the remote repository to the current or other StarRocks clusters to complete the data recovery or migration.

### (Optional) Create a Repository in the New Cluster

If you need to migrate data to other StarRocks clusters, you need to create a repository in the new cluster with the same **repository name** and **address**. Otherwise, you will not be able to view the previously backed up data snapshots. For more information, see [Create Repository](#create-repository).

### 查看数据库快照

开始恢复或迁移前，您可以通过 [SHOW SNAPSHOT](../sql-reference/sql-statements/data-manipulation/SHOW_SNAPSHOT.md) 查看特定仓库对应的数据快照信息。

以下示例查看仓库 `test_repo` 中的数据快照信息。

```Plain
mysql> SHOW SNAPSHOT ON test_repo;
+------------------+-------------------------+--------+
| Snapshot         | Timestamp               | Status |
+------------------+-------------------------+--------+
| sr_member_backup | 2023-02-07-14-45-53-143 | OK     |
+------------------+-------------------------+--------+
1 row in set (1.16 sec)
```

### 恢复数据快照

通过 [RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md) 语句将远端仓库中的数据快照恢复至当前或其他 StarRocks 集群以恢复或迁移数据。

以下示例将仓库 `test_repo` 中的数据快照 `sr_member_backup`恢复为表 `sr_member`，仅恢复一个数据副本。

```SQL
RESTORE SNAPSHOT sr_hub.sr_member_backup
FROM test_repo
ON (sr_member)
PROPERTIES (
    "backup_timestamp"="2023-02-07-14-45-53-143",
    "replication_num" = "1"
);
```

数据恢复为异步操作。您可以通过 [SHOW RESTORE](../sql-reference/sql-statements/data-manipulation/SHOW_RESTORE.md) 语句查看恢复作业状态，或通过 [CANCEL RESTORE](../sql-reference/sql-statements/data-definition/CANCEL_RESTORE.md) 语句取消恢复作业。

## 配置相关参数

您可以通过在 BE 配置文件 **be.conf** 中修改以下配置项加速备份或还原作业：

| 配置项                   | 说明                                                                             |
| ----------------------- | -------------------------------------------------------------------------------- |
| upload_worker_count     | BE 节点上传任务的最大线程数，用于备份作业。默认值：`1`。增加此配置项的值可以增加上传任务并行度。|
| download_worker_count   | BE 节点下载任务的最大线程数，用于还原作业。默认值：`1`。增加此配置项的值可以增加下载任务并行度。|
| max_download_speed_kbps | BE 节点下载速度上限。默认值：`50000`。单位：KB/s。通常还原作业的下载速度不会超过默认值。如果该速度上限限制了还原作业的性能，您可以根据带宽情况适当增加。|

## 物化视图备份恢复

在备份或还原表（Table）数据期间，StarRocks 会自动备份或还原其中的 [同步物化视图](../using_starrocks/Materialized_view-single_table.md)。

从 v3.2.0 开始，StarRocks 支持在备份和还原数据库（Database）时备份和还原数据库中的 [异步物化视图](../using_starrocks/Materialized_view.md)。

在备份和还原数据库期间，StarRocks 执行以下操作：

- **BACKUP**

1. 遍历数据库以收集所有表和异步物化视图的信息。
2. 调整备份还原队列中表的顺序，确保物化视图的基表位于物化视图之前：
   - 如果基表存在于当前数据库中，StarRocks 将表添加到队列中。
   - 如果基表不存在于当前数据库中，StarRocks 打印警告日志并继续执行备份操作而不阻塞该过程。
3. 按照队列的顺序执行备份任务。

- **RESTORE**

1. 按照备份还原队列的顺序还原表和物化视图。
2. 重新构建物化视图与其基表之间的依赖关系，并重新提交刷新任务调度。

在整个还原过程中遇到的任何错误都不会阻塞该过程。

还原后，您可以使用[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) 检查物化视图的状态。

- 如果物化视图处于 Active 状态，则可以直接使用。
- 如果物化视图处于 Inactive 状态，可能是因为其基表尚未还原。在还原所有基表后，您可以使用[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) 重新激活物化视图。

## 注意事项

- 仅限拥有 ADMIN 权限的用户执行备份与恢复功能。
- 单一数据库内，仅可同时执行一个备份或恢复作业。否则，StarRocks 返回错误。
- 因为备份与恢复操作会占用一定系统资源，建议您在集群业务低峰期进行该操作。
- 目前 StarRocks 不支持在备份数据时使用压缩算法。
- 因为数据备份是通过快照的形式完成的，所以在当前数据快照生成之后导入的数据不会被备份。因此，在快照生成至恢复（迁移）作业完成这段期间导入的数据，需要重新导入至集群。建议您在迁移完成后，对新旧两个集群并行导入一段时间，完成数据和业务正确性校验后，再将业务迁移到新的集群。
- 在恢复作业完成前，被恢复表无法被操作。
- Primary Key 表无法被恢复至 v2.5 之前版本的 StarRocks 集群中。
- 您无需在恢复作业前在新集群中创建需要被恢复表。恢复作业将自动创建该表。
- 如果被恢复表与已有表重名，StarRocks 会首先识别已有表的 Schema。如果 Schema 相同，StarRocks 会覆盖写入已有表。如果 Schema 不同，恢复作业失败。您可以通过 `AS` 关键字重新命名被恢复表，或者删除已有表后重新发起恢复作业。
- 如果恢复作业是一次覆盖操作（指定恢复数据到已经存在的表或分区中），那么从恢复作业的 COMMIT 阶段开始，当前集群上被覆盖的数据有可能不能再被还原。此时如果恢复作业失败或被取消，有可能造成之前的数据损坏且无法访问。这种情况下，只能通过再次执行恢复操作，并等待作业完成。因此，我们建议您，如无必要，不要使用覆盖的方式恢复数据，除非确认当前数据已不再使用。覆盖操作会检查快照和已存在的表或分区的元数据是否相同，包括 Schema 和 Rollup 等信息，如果不同则无法执行恢复操作。
- 目前 StarRocks 暂不支持备份恢复逻辑视图。
- 目前 StarRocks 暂不支持备份恢复用户、权限以及资源组配置相关数据。
- StarRocks 不支持备份恢复表之间的 Colocate Join 关系。