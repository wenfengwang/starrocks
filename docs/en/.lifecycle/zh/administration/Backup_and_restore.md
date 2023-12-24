---
displayed_sidebar: English
---

# 备份和恢复数据

本主题描述了如何在StarRocks中备份和恢复数据，或将数据迁移到新的StarRocks集群。

StarRocks支持将数据作为快照备份到远程存储系统，并将数据恢复到任何StarRocks集群。

StarRocks支持以下远程存储系统：

- Apache™ Hadoop®（HDFS）集群
- AWS S3
- Google GCS

## 备份数据

StarRocks支持在数据库、表或分区级别进行完全备份。

如果您在表中存储了大量数据，建议您按分区进行数据备份和恢复。这样，您可以降低作业失败时重试的成本。如果需要定期备份增量数据，可以制定一个[动态分区](../table_design/dynamic_partitioning.md)计划（例如，按特定时间间隔）来备份表中的新分区。

### 创建存储库

在备份数据之前，您需要创建一个存储库，用于将数据快照存储在远程存储系统中。您可以在StarRocks集群中创建多个存储库。有关详细说明，请参见[CREATE REPOSITORY](../sql-reference/sql-statements/data-definition/CREATE_REPOSITORY.md)。

- 在HDFS中创建存储库

以下示例在HDFS集群中创建了一个名为`test_repo`的存储库。

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "hdfs://<hdfs_host>:<hdfs_port>/repo_dir/backup"
PROPERTIES(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

- 在AWS S3中创建存储库

  您可以选择IAM用户凭证（Access Key和Secret Key）、实例配置文件或假定角色作为访问AWS S3的凭证方法。

  - 以下示例使用IAM用户凭证作为凭证方法，在名为`bucket_s3`的AWS S3存储桶中创建了一个名为`test_repo`的存储库。

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

  - 以下示例使用实例配置文件作为凭证方法，在名为`bucket_s3`的AWS S3存储桶中创建了一个名为`test_repo`的存储库。

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-east-1"
  );
  ```

  - 以下示例使用假定角色作为凭证方法，在名为`bucket_s3`的AWS S3存储桶中创建了一个名为`test_repo`的存储库。

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
> StarRocks仅支持根据S3A协议在AWS S3中创建存储库。因此，在AWS S3中创建存储库时，必须将`ON LOCATION`中传递的S3 URI中的`s3://`替换为`s3a://`。

- 在Google GCS中创建存储库

以下示例在Google GCS存储桶中创建了一个名为`test_repo`的存储库。

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
> StarRocks仅支持根据S3A协议在Google GCS中创建存储库。因此，在Google GCS中创建存储库时，必须将`ON LOCATION`中传递的GCS URI中的前缀替换为`s3a://`。

存储库创建完成后，您可以通过[SHOW REPOSITORIES](../sql-reference/sql-statements/data-manipulation/SHOW_REPOSITORIES.md)检查存储库。恢复数据后，您可以使用[DROP REPOSITORY](../sql-reference/sql-statements/data-definition/DROP_REPOSITORY.md)在StarRocks中删除存储库。但是，远程存储系统中备份的数据快照无法通过StarRocks删除。您需要在远程存储系统中手动删除它们。

### 备份数据快照

存储库创建完成后，您需要创建数据快照并将其备份到远程存储库中。有关详细说明，请参见[BACKUP](../sql-reference/sql-statements/data-definition/BACKUP.md)。

以下示例为数据库`sr_hub`中的表`sr_member`创建了一个名为`sr_member_backup`的数据快照，并将其备份到了存储库`test_repo`中。

```SQL
BACKUP SNAPSHOT sr_hub.sr_member_backup
TO test_repo
ON (sr_member);
```

备份是一个异步操作。您可以使用[SHOW BACKUP](../sql-reference/sql-statements/data-manipulation/SHOW_BACKUP.md)来检查备份作业的状态，或使用[CANCEL BACKUP](../sql-reference/sql-statements/data-definition/CANCEL_BACKUP.md)来取消备份作业。

## 恢复或迁移数据

您可以将远程存储系统中备份的数据快照恢复到当前或其他StarRocks集群，以恢复或迁移数据。

### （可选）在新集群中创建存储库

要将数据迁移到另一个StarRocks集群，您需要在新集群中创建具有相同**存储库名称**和**位置**的存储库，否则您将无法查看先前备份的数据快照。有关详细信息，请参见[创建存储库](#create-a-repository)。

### 检查快照

在恢复数据之前，您可以使用[SHOW SNAPSHOT](../sql-reference/sql-statements/data-manipulation/SHOW_SNAPSHOT.md)来检查指定存储库中的快照。
以下示例检查 `test_repo` 中的快照信息。

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

您可以使用 [RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md) 语句将远程存储系统中的数据快照恢复到当前或其他 StarRocks 集群。

以下示例在 `test_repo` 上的 `sr_member` 表中还原数据快照 `sr_member_backup`。它只还原一个数据副本。

```SQL
RESTORE SNAPSHOT sr_hub.sr_member_backup
FROM test_repo
ON (sr_member)
PROPERTIES (
    "backup_timestamp"="2023-02-07-14-45-53-143",
    "replication_num" = "1"
);
```

RESTORE 是一个异步操作。您可以使用 [SHOW RESTORE](../sql-reference/sql-statements/data-manipulation/SHOW_RESTORE.md) 来检查 RESTORE 作业的状态，或使用 [CANCEL RESTORE](../sql-reference/sql-statements/data-definition/CANCEL_RESTORE.md) 来取消 RESTORE 作业。

## 配置 BACKUP 或 RESTORE 作业

您可以通过修改 BE 配置文件 **be.conf** 中的以下配置项来优化 BACKUP 或 RESTORE 作业的性能：

| 配置项                | 描述                                                                                   |
| ----------------------- | -------------------------------------------------------------------------------- |
| upload_worker_count     | BE 节点上 BACKUP 作业上传任务的最大线程数。默认值：`1`。增加该配置项的值可以增加上传任务的并发性。 |
| download_worker_count   | BE 节点上 RESTORE 作业下载任务的最大线程数。默认值：`1`。增加该配置项的值可以增加下载任务的并发性。 |
| max_download_speed_kbps | BE 节点下载速度的上限。默认值：`50000`。单位：KB/s。通常，RESTORE 作业中下载任务的速度不会超过默认值。如果此配置限制了 RESTORE 作业的性能，则可以根据带宽增加性能。|

## 物化视图的 BACKUP 和 RESTORE

在对表进行 BACKUP 或 RESTORE 作业时，StarRocks 会自动备份或恢复其[同步物化视图](../using_starrocks/Materialized_view-single_table.md)。

从 v3.2.0 开始，StarRocks 支持在备份和恢复它们所在的数据库时备份和恢复[异步物化视图](../using_starrocks/Materialized_view.md)。

在对数据库进行 BACKUP 和 RESTORE 操作时，StarRocks 会执行以下操作：

- **备份**

1. 遍历数据库以收集所有表和异步物化视图的信息。
2. 调整 BACKUP 和 RESTORE 队列中表的顺序，确保物化视图的基表位于物化视图之前：
   - 如果基表存在于当前数据库中，StarRocks 会将该表添加到队列中。
   - 如果基表不存在于当前数据库中，StarRocks 会打印警告日志，并在不阻塞进程的情况下继续执行 BACKUP 操作。
3. 按队列顺序执行 BACKUP 任务。

- **RESTORE**

1. 按照 BACKUP 和 RESTORE 队列的顺序还原表和物化视图。
2. 重新构建物化视图与其基表之间的依赖关系，并重新提交刷新任务计划。

在整个 RESTORE 过程中遇到的任何错误都不会阻止该进程。

RESTORE 后，您可以使用 [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) 来检查物化视图的状态。

- 如果物化视图处于活动状态，则可以直接使用。
- 如果物化视图处于非活动状态，可能是因为其基表未还原。还原所有基表后，您可以使用 [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) 来重新激活物化视图。

## 使用说明

- 只有具有 ADMIN 权限的用户才能备份或恢复数据。
- 在每个数据库中，每次只允许一个正在运行的 BACKUP 或 RESTORE 作业。否则，StarRocks 会返回错误。
- 由于 BACKUP 和 RESTORE 作业占用了 StarRocks 集群的大量资源，因此建议您在 StarRocks 集群负载较轻的情况下进行数据备份和恢复。
- StarRocks 不支持为数据备份指定数据压缩算法。
- 由于数据是作为快照备份的，因此快照生成时加载的数据不包括在快照中。因此，如果在快照生成后、RESTORE 作业完成之前将数据加载到旧集群中，则还需要将数据加载到要恢复数据的集群中。建议在数据迁移完成后，将数据并行加载到两个集群中一段时间，验证数据和服务的正确性后再将应用迁移到新集群。
- 在 RESTORE 作业完成之前，无法对待恢复的表进行操作。
- 主键表不支持恢复到早于 v2.5 的 StarRocks 集群。
- 在还原之前，您无需在新集群中创建要还原的表。RESTORE 作业会自动创建它。
- 如果存在与要还原表重名的现有表，StarRocks 会首先检查现有表的模式是否与要还原表的模式匹配。如果模式匹配，StarRocks 会使用快照中的数据覆盖现有表。如果模式不匹配，则 RESTORE 作业将失败。您可以使用关键字 `AS` 重命名要还原的表，也可以在还原数据之前删除现有表。
- 如果 RESTORE 作业覆盖了现有数据库、表或分区，则在作业进入 COMMIT 阶段后，无法恢复被覆盖的数据。如果此时 RESTORE 作业失败或被取消，则数据可能已损坏且无法访问。在这种情况下，您只能再次执行 RESTORE 操作并等待作业完成。因此，我们建议您不要通过覆盖来恢复数据，除非您确定当前数据不再使用。覆盖操作首先检查快照与现有数据库、表或分区之间的元数据一致性。如果检测到不一致，则无法执行 RESTORE 操作。
- 目前，StarRocks 不支持备份和恢复逻辑视图。
- 目前，StarRocks 不支持备份和恢复与用户账号、权限和资源组相关的配置数据。
- 目前，StarRocks 不支持备份和恢复表间的 Colocate Join 关系。