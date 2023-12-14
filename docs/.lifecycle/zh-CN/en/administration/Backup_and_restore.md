---
displayed_sidebar: "中文"
---

# 数据备份与恢复

本主题介绍如何在 StarRocks 中备份和恢复数据，或将数据迁移到新的 StarRocks 集群。

StarRocks 可以将数据备份为快照存储到远程存储系统，并将数据还原到任何 StarRocks 集群中。

StarRocks 支持以下远程存储系统：

- Apache™ Hadoop®（HDFS）集群
- AWS S3
- Google GCS

## 备份数据

StarRocks 可以在数据库、表或分区的粒度级别上进行完全备份。

如果您在表中存储了大量数据，我们建议您按分区进行数据备份和还原。这样，在作业失败时可以降低重试的成本。如果您需要定期备份增量数据，可以为表制定一项[动态分区](../table_design/dynamic_partitioning.md)计划（例如按一定时间间隔），每次仅备份新的分区。

### 创建存储库

在备份数据之前，您需要创建一个存储库，用于将数据快照存储到远程存储系统中。您可以在 StarRocks 集群中创建多个存储库。有关详细说明，请参见[CREATE REPOSITORY](../sql-reference/sql-statements/data-definition/CREATE_REPOSITORY.md)。

- 在 HDFS 中创建存储库

以下示例在 HDFS 集群中创建名为 `test_repo` 的存储库。

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

  您可以选择 IAM 用户凭证（访问密钥和秘密密钥）、实例配置文件或假定角色作为访问 AWS S3 的凭据方法。

  - 以下示例使用 IAM 用户凭证作为凭证方法，在 AWS S3 存储桶 `bucket_s3` 中创建名为 `test_repo` 的存储库。

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

  - 以下示例使用实例配置文件作为凭证方法，在 AWS S3 存储桶 `bucket_s3` 中创建名为 `test_repo` 的存储库。

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-east-1"
  );
  ```

  - 以下示例使用假定角色作为凭证方法，在 AWS S3 存储桶 `bucket_s3` 中创建名为 `test_repo` 的存储库。

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
>
> StarRocks 仅支持按照 S3A 协议在 AWS S3 中创建存储库。因此，在 AWS S3 中创建存储库时，必须将 `ON LOCATION` 中传递的 S3 URI 中的 `s3://` 替换为 `s3a://`。

- 在 Google GCS 中创建存储库

以下示例在 Google GCS 存储桶 `bucket_gcs` 中创建名为 `test_repo` 的存储库。

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
>
> StarRocks 仅支持按照 S3A 协议在 Google GCS 中创建存储库。因此，在 Google GCS 中创建存储库时，必须将 `ON LOCATION` 中传递的 GCS URI 的前缀替换为 `s3a://`。

存储库创建后，您可以通过[SHOW REPOSITORIES](../sql-reference/sql-statements/data-manipulation/SHOW_REPOSITORIES.md)检查存储库。还原数据后，您可以使用[DROP REPOSITORY](../sql-reference/sql-statements/data-definition/DROP_REPOSITORY.md)在 StarRocks 中删除存储库。但是，通过 StarRocks 备份在远程存储系统中的快照无法通过 StarRocks 删除，您需要在远程存储系统中手动删除它们。

### 备份数据快照

存储库创建后，您需要创建一个数据快照并将其备份到远程存储库中。有关详细说明，请参见[BACKUP](../sql-reference/sql-statements/data-definition/BACKUP.md)。

以下示例为数据库 `sr_hub` 中表 `sr_member` 创建一个名为 `sr_member_backup` 的数据快照，并将其备份到存储库 `test_repo` 中。

```SQL
BACKUP SNAPSHOT sr_hub.sr_member_backup
TO test_repo
ON (sr_member);
```

备份是一个异步操作。您可以使用[SHOW BACKUP](../sql-reference/sql-statements/data-manipulation/SHOW_BACKUP.md)检查 BACKUP 作业的状态，或使用[CANCEL BACKUP](../sql-reference/sql-statements/data-definition/CANCEL_BACKUP.md)取消 BACKUP 作业。

## 还原或迁移数据

您可以将备份在远程存储系统中的数据快照还原到当前的 StarRocks 集群或其他 StarRocks 集群中，以还原或迁移数据。

### （可选）在新集群中创建存储库

要将数据迁移到另一个 StarRocks 集群，您需要在新集群中使用相同的**存储库名称**和**位置**创建一个存储库，否则您将无法查看先前备份的数据快照。有关详细信息，请参见[创建存储库](#创建存储库)。

### 检查快照

还原数据之前，您可以使用[SHOW SNAPSHOT](../sql-reference/sql-statements/data-manipulation/SHOW_SNAPSHOT.md)查看特定存储库中的快照。

以下示例检查 `test_repo` 中的快照信息。

```Plain
mysql> SHOW SNAPSHOT ON test_repo;
+------------------+-------------------------+--------+
| 快照             | 时间戳                  | 状态   |
+------------------+-------------------------+--------+
| sr_member_backup | 2023-02-07-14-45-53-143 | OK     |
+------------------+-------------------------+--------+
1 行记录（用时 1.16 秒）
```

### 通过快照还原数据

您可以使用[RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md)语句将远程存储系统中的数据快照还原到当前的 StarRocks 集群或其他 StarRocks 集群中。

以下示例将 `test_repo` 中的数据快照 `sr_member_backup` 在表 `sr_member` 上还原。仅还原一个数据副本。

```SQL
RESTORE SNAPSHOT sr_hub.sr_member_backup
FROM test_repo
ON (sr_member)
PROPERTIES (
    "backup_timestamp"="2023-02-07-14-45-53-143",
    "replication_num" = "1"
);
```

RESTORE 是一个异步操作。您可以使用[SHOW RESTORE](../sql-reference/sql-statements/data-manipulation/SHOW_RESTORE.md)检查 RESTORE 作业的状态，或使用[CANCEL RESTORE](../sql-reference/sql-statements/data-definition/CANCEL_RESTORE.md)取消 RESTORE 作业。

## 配置备份或还原作业

您可以通过修改 BE 配置文件 **be.conf** 中的以下配置项来优化备份或还原作业的性能：

| 配置项               | 描述                                                                             |
| ----------------------- | -------------------------------------------------------------------------------- |
| upload_worker_count     | BE 节点上备份作业的上传任务的最大线程数。默认为 `1`。增加此配置项的值可以增加上传任务的并发性。 |
| download_worker_count   | BE 节点上还原作业的下载任务的最大线程数。默认为 `1`。增加此配置项的值可以增加下载任务的并发性。 |
| max_download_speed_kbps | BE 节点上的下载速度上限。默认为 `50000`。单位：KB/s。通常，还原作业中的下载任务速度不会超过默认值。如果该配置限制了还原作业的性能，您可以根据带宽将其增加。|

## 物化视图备份和还原

在对表进行备份或还原作业时，StarRocks 会自动备份或还原其[同步物化视图](../using_starrocks/Materialized_view-single_table.md)。

从 v3.2.0 版本开始，StarRocks 支持在备份和还原数据库时备份和还原 [异步物化视图](../using_starrocks/Materialized_view.md)。

在执行数据库的 BACKUP 和 RESTORE 操作时，StarRocks 执行以下操作:

- **BACKUP**

1. 遍历数据库以收集所有表和异步物化视图的信息。
2. 调整 BACKUP 和 RESTORE 队列中表的顺序，确保物化视图的基础表在物化视图之前:
   - 如果基础表存在于当前数据库中，StarRocks 将表添加到队列中。
   - 如果基础表不存在于当前数据库中，StarRocks 会打印警告日志，并继续执行 BACKUP 操作而不阻塞该过程。
3. 按队列顺序执行 BACKUP 任务。

- **RESTORE**

1. 按照 BACKUP 和 RESTORE 队列的顺序还原表和物化视图。
2. 重建物化视图与其基础表之间的依赖关系，并重新提交刷新任务调度。

在 RESTORE 过程中遇到的任何错误都不会阻塞该过程。

RESTORE 后，您可以使用 [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) 来检查物化视图的状态。

- 如果物化视图处于活动状态，则可以直接使用。
- 如果物化视图处于非活动状态，可能是因为其基础表尚未恢复。在所有基础表都恢复之后，可以使用 [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) 来重新激活物化视图。

## 使用注意事项

- 仅具有 ADMIN 权限的用户可以备份或还原数据。
- 在每个数据库中，一次只允许运行一个 BACKUP 或 RESTORE 作业。否则，StarRocks 会返回错误。
- 由于 BACKUP 和 RESTORE 作业占用 StarRocks 集群的许多资源，您可以在 StarRocks 集群的负载不重时备份和还原数据。
- StarRocks 不支持为数据备份指定数据压缩算法。
- 由于数据是作为快照进行备份的，在生成快照时加载的数据不包括在快照中。因此，如果在生成快照后并且 RESTORE 作业完成之前将数据加载到旧集群中，还需要将数据加载到还原数据的集群中。建议在数据迁移完成后的一段时间内同时在两个集群中加载数据，然后在验证数据和服务的正确性后将应用程序迁移到新集群。
- 在 RESTORE 作业完成之前，无法操作要还原的表。
- 主键表不能还原到早于 v2.5 的 StarRocks 集群。
- 在还原之前，无需在新集群中创建要还原的表。RESTORE 作业会自动创建它。
- 如果存在一个与要还原表重复名称的现有表，StarRocks 首先检查现有表的模式是否与要还原的表的模式匹配。如果模式匹配，StarRocks 将使用快照中的数据覆盖现有表。如果模式不匹配，则 RESTORE 作业失败。您可以使用关键字 `AS` 重命名要还原的表，或者在还原数据之前删除现有表。
- 如果 RESTORE 作业覆盖了现有数据库、表或分区，则作业进入 COMMIT 阶段后覆盖的数据无法进行还原。如果 RESTORE 作业在此时失败或被取消，数据可能会损坏并且无法访问。在这种情况下，您只能再次执行 RESTORE 操作并等待作业完成。因此，我们建议您不要进行覆盖式的数据还原，除非确定当前数据不再使用。覆盖操作首先检查快照和现有数据库、表或分区之间的元数据一致性。如果检测到不一致性，无法执行 RESTORE 操作。
- 目前，StarRocks不支持备份和还原逻辑视图。
- 目前，StarRocks不支持备份和还原与用户帐户、权限和资源组相关的配置数据。
- 目前，StarRocks不支持备份和还原表之间的 Colocate Join 关系。