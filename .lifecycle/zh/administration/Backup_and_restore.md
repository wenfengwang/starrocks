---
displayed_sidebar: English
---

# 备份和恢复数据

本主题介绍如何在 StarRocks 中备份和恢复数据，或将数据迁移到新的 StarRocks 集群。

StarRocks 支持将数据作为快照备份到远程存储系统，并能将数据恢复到任意 StarRocks 集群。

StarRocks 支持以下远程存储系统：

- Apache™ Hadoop® (HDFS) 集群
- AWS S3
- Google GCS

## 备份数据

StarRocks 支持以数据库、表或分区为粒度的完整备份。

如果您的表中存储了大量数据，我们建议您按分区进行备份和恢复。这样，您可以减少因作业失败而重试的成本。如果您需要定期备份增量数据，可以为您的表制定[动态分区](../table_design/dynamic_partitioning.md)计划（例如按一定时间间隔），并每次只备份新分区。

### 创建存储库

在备份数据之前，您需要创建一个存储库，该存储库用于在远程存储系统中存放数据快照。您可以在 StarRocks 集群中创建多个存储库。详细指南请参见 [CREATE REPOSITORY](../sql-reference/sql-statements/data-definition/CREATE_REPOSITORY.md)。

- 在 HDFS 中创建存储库

以下示例在 HDFS 集群中创建名为 test_repo 的存储库。

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "hdfs://<hdfs_host>:<hdfs_port>/repo_dir/backup"
PROPERTIES(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

- 在 AWS S3 中创建存储库

  您可以选择基于 IAM 用户的凭证（Access Key 和 Secret Key）、实例配置文件或假定角色作为访问 AWS S3 的凭证方法。

  - 以下示例使用基于 IAM 用户的凭证方法，在 AWS S3 存储桶 bucket_s3 中创建名为 test_repo 的存储库。

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

  - 以下示例使用实例配置文件作为凭证方法，在 AWS S3 存储桶 bucket_s3 中创建名为 test_repo 的存储库。

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-east-1"
  );
  ```

  - 以下示例使用假定角色作为凭证方法，在 AWS S3 存储桶 bucket_s3 中创建名为 test_repo 的存储库。

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

> **注意**
> StarRocks 仅支持根据 S3A 协议在 AWS S3 中创建存储库。因此，当您在 AWS S3 中创建存储库时，必须将 `s3://` 替换为 `s3a://`，并将其作为 `ON LOCATION` 中的存储库位置传递。

- 在 Google GCS 中创建存储库

以下示例在 Google GCS 存储桶 bucket_gcs 中创建名为 test_repo 的存储库。

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

> **注意**
> StarRocks 仅支持根据 S3A 协议在 Google GCS 中创建存储库。因此，当您在 Google GCS 中创建存储库时，必须将 `ON LOCATION` 中传递的存储库位置的 GCS URI 的前缀替换为 `s3a://`。

创建存储库后，您可以通过 [SHOW REPOSITORIES](../sql-reference/sql-statements/data-manipulation/SHOW_REPOSITORIES.md) 命令查看存储库。数据恢复后，您可以在 StarRocks 中使用 [DROP REPOSITORY](../sql-reference/sql-statements/data-definition/DROP_REPOSITORY.md) 命令删除存储库。但是，无法通过 StarRocks 删除远程存储系统中备份的数据快照，您需要在远程存储系统中手动删除它们。

### 备份数据快照

创建存储库后，您需要创建数据快照并将其备份到远程存储库。详细指南请参见 [BACKUP](../sql-reference/sql-statements/data-definition/BACKUP.md)。

以下示例为数据库 sr_hub 中的表 sr_member 创建数据快照 sr_member_backup，并将其备份到存储库 test_repo 中。

```SQL
BACKUP SNAPSHOT sr_hub.sr_member_backup
TO test_repo
ON (sr_member);
```

BACKUP 是一个异步操作。您可以使用 [SHOW BACKUP](../sql-reference/sql-statements/data-manipulation/SHOW_BACKUP.md) 命令检查备份作业的状态，或使用 [CANCEL BACKUP](../sql-reference/sql-statements/data-definition/CANCEL_BACKUP.md) 命令取消备份作业。

## 恢复或迁移数据

您可以将远程存储系统中备份的数据快照恢复到当前或其他 StarRocks 集群，以实现数据的恢复或迁移。

### （可选）在新集群中创建存储库

若要将数据迁移到另一个 StarRocks 集群，您需要在新集群中创建具有相同**存储库名称**和**位置**的存储库，否则您将无法查看先前备份的数据快照。详细信息请参见[创建存储库](#create-a-repository)。

### 检查快照

在恢复数据之前，您可以使用[SHOW SNAPSHOT](../sql-reference/sql-statements/data-manipulation/SHOW_SNAPSHOT.md)命令查看指定存储库中的快照信息。

以下示例检查 test_repo 中的快照信息。

```Plain
mysql> SHOW SNAPSHOT ON test_repo;
+------------------+-------------------------+--------+
| Snapshot         | Timestamp               | Status |
+------------------+-------------------------+--------+
| sr_member_backup | 2023-02-07-14-45-53-143 | OK     |
+------------------+-------------------------+--------+
1 row in set (1.16 sec)
```

### 通过快照恢复数据

您可以使用[RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md)语句将远程存储系统中的数据快照恢复到当前或其他StarRocks集群。

以下示例恢复表 sr_member 上 test_repo 中的数据快照 sr_member_backup。该操作仅恢复一个数据副本。

```SQL
RESTORE SNAPSHOT sr_hub.sr_member_backup
FROM test_repo
ON (sr_member)
PROPERTIES (
    "backup_timestamp"="2023-02-07-14-45-53-143",
    "replication_num" = "1"
);
```

RESTORE 是一个异步操作。您可以使用 [SHOW RESTORE](../sql-reference/sql-statements/data-manipulation/SHOW_RESTORE.md) 命令检查 RESTORE 作业的状态，或使用 [CANCEL RESTORE](../sql-reference/sql-statements/data-definition/CANCEL_RESTORE.md) 命令取消 RESTORE 作业。

## 配置 BACKUP 或 RESTORE 作业

您可以通过修改 BE 配置文件 **be.conf** 中的以下配置项来优化 BACKUP 或 RESTORE 作业的性能：

|配置项|说明|
|---|---|
|upload_worker_count|BE节点上BACKUP作业上传任务的最大线程数。默认：1。增大该配置项的值可以提高上传任务的并发度。|
|download_worker_count|BE节点上RESTORE作业下载任务的最大线程数。默认：1。增大该配置项的值可以提高下载任务的并发度。|
|max_download_speed_kbps|BE节点下载速度上限。默认值：50000。单位：KB/s。通常，RESTORE 作业中的下载任务的速度不会超过默认值。如果此配置限制了 RESTORE 作业的性能，您可以根据您的带宽增加它。|

## 物化视图备份和恢复

在备份或恢复表的作业期间，StarRocks自动备份或恢复其[同步物化视图](../using_starrocks/Materialized_view-single_table.md)。

从 v3.2.0 版本开始，StarRocks支持在备份和恢复[异步物化视图](../using_starrocks/Materialized_view.md)所在的数据库时。

在备份和恢复数据库期间，StarRocks 会执行以下操作：

- **备份**

1. 遍历数据库，收集所有表和异步物化视图的信息。
2. 调整 BACKUP 和 RESTORE 队列中的表顺序，确保物化视图的基础表排在物化视图之前：
   - 如果当前数据库中存在基础表，StarRocks 会将该表加入队列。
   - 如果当前数据库中不存在基础表，StarRocks 会记录一条警告日志，并继续执行 BACKUP 操作，不会阻塞进程。
3. 按照队列的顺序执行 BACKUP 任务。

- **恢复**

1. 按照 BACKUP 和 RESTORE 队列的顺序恢复表和物化视图。
2. 重建物化视图与其基础表之间的依赖关系，并重新提交刷新任务计划。

整个 RESTORE 过程中遇到的任何错误都不会中断进程。

RESTORE 完成后，您可以使用[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)命令检查物化视图的状态。

- 如果物化视图处于活跃状态，可以直接使用。
- 如果物化视图处于非活跃状态，可能是因为其基础表尚未恢复。在所有基础表恢复后，您可以使用 [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) 语句重新激活物化视图。

## 使用须知

- 只有具有 ADMIN 权限的用户才能备份或恢复数据。
- 每个数据库一次只允许运行一个 BACKUP 或 RESTORE 作业。否则，StarRocks 会返回错误。
- 由于 BACKUP 和 RESTORE 作业会占用 StarRocks 集群的大量资源，建议在集群负载较轻时进行数据的备份和恢复。
- StarRocks 不支持为数据备份指定数据压缩算法。
- 由于数据是以快照形式备份的，所以在快照生成之后加载到旧集群中的数据不包含在快照中。因此，如果在快照生成后和 RESTORE 作业完成前向旧集群加载数据，您还需要将数据加载到数据恢复的目标集群中。建议在数据迁移完成后，同时向两个集群并行加载数据一段时间，验证数据和服务的正确性后，再将应用迁移到新集群。
- 在 RESTORE 作业完成之前，不允许对待恢复的表进行操作。
- 主键表无法恢复到 v2.5 之前的 StarRocks 集群。
- 恢复数据前，无需在新集群中预先创建待恢复的表。RESTORE 作业会自动创建它。
- 如果存在与待恢复的表同名的现有表，StarRocks 首先检查现有表的架构是否与待恢复的表的架构匹配。如果架构匹配，StarRocks 会用快照中的数据覆盖现有表。如果架构不匹配，RESTORE 作业将失败。您可以使用 AS 关键字重命名待恢复的表，或在恢复数据之前删除现有表。
- 如果 RESTORE 作业覆盖了现有的数据库、表或分区，一旦作业进入 COMMIT 阶段，被覆盖的数据将无法恢复。如果此时 RESTORE 作业失败或被取消，数据可能会受损且无法访问。在这种情况下，您只能重新执行 RESTORE 操作，并等待作业完成。因此，建议您不要通过覆盖现有数据的方式进行恢复，除非您确定当前数据不再需要。覆盖操作会首先检查快照与现有数据库、表或分区之间的元数据一致性。如果检测到不一致，将无法执行 RESTORE 操作。
- 目前，StarRocks 不支持备份和恢复逻辑视图。
- 目前，StarRocks 不支持备份和恢复与用户账户、权限和资源组相关的配置数据。
- 目前，StarRocks 不支持备份和恢复表间的 Colocate Join 关系。
