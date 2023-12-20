---
displayed_sidebar: English
---

# 创建存储库

## 描述

在远程存储系统中创建一个存储库，用于存储数据快照，以便[备份和恢复数据](../../../administration/Backup_and_restore.md)。

> **警告**
> 只有具有 ADMIN 权限的用户才能创建存储库。

有关删除存储库的详细说明，请参阅 [DROP REPOSITORY](../data-definition/DROP_REPOSITORY.md)。

## 语法

```SQL
CREATE [READ ONLY] REPOSITORY <repository_name>
WITH BROKER
ON LOCATION "<repository_location>"
PROPERTIES ("key"="value", ...)
```

## 参数

|**参数**|**说明**|
|---|---|
|READ ONLY|创建只读存储库。请注意，您只能从只读存储库恢复数据。当为两个集群创建相同的存储库以进行数据迁移时，可以为新集群创建一个只读存储库，并仅授予其 RESTORE 权限。|
|repository_name|存储库名称。|
|repository_location|远程存储系统中存储库的位置。|
|PROPERTIES|访问远程存储系统的认证方式。|

**PROPERTIES**：

StarRocks 支持在 HDFS、AWS S3 和 Google GCS 中创建存储库。

- 对于 HDFS：
  - "username": 您想用来访问 HDFS 集群 NameNode 的账户的用户名。
  - "password": 您想用来访问 HDFS 集群 NameNode 的账户的密码。

- 对于 AWS S3：
  - "aws.s3.use_instance_profile": 是否允许使用实例配置文件和假定角色作为访问 AWS S3 的认证方式。默认值：`false`。

    - 如果您使用基于 IAM 用户的认证（Access Key 和 Secret Key）来访问 AWS S3，您不需要指定此参数，而需要指定 "aws.s3.access_key"、"aws.s3.secret_key" 和 "aws.s3.endpoint"。
    - 如果您使用实例配置文件来访问 AWS S3，您需要将此参数设置为 `true` 并指定 "aws.s3.region"。
    - 如果您使用假定角色来访问 AWS S3，您需要将此参数设置为 `true` 并指定 "aws.s3.iam_role_arn" 和 "aws.s3.region"。

  - "aws.s3.access_key": 您可以用来访问 Amazon S3 存储桶的 Access Key ID。
  - "aws.s3.secret_key": 您可以用来访问 Amazon S3 存储桶的 Secret Access Key。
  - "aws.s3.endpoint": 您可以用来访问 Amazon S3 存储桶的 endpoint。
  - "aws.s3.iam_role_arn": 拥有对存储数据文件的 AWS S3 存储桶权限的 IAM 角色的 ARN。如果您希望使用假定角色作为访问 AWS S3 的认证方式，您必须指定此参数。然后，StarRocks 在使用 Hive 目录分析您的 Hive 数据时会扮演此角色。
  - "aws.s3.region": 您的 AWS S3 存储桶所在的区域。例如：`us-west-1`。

> **注意**
> StarRocks 仅支持根据 S3A 协议在 AWS S3 中创建存储库。因此，当您在 AWS S3 中创建存储库时，您必须将 `ON LOCATION` 中传递的 S3 URI 的 `s3://` 替换为 `s3a://`。

- 对于 Google GCS：
  - "fs.s3a.access.key": 您可以用来访问 Google GCS 存储桶的 Access Key。
  - "fs.s3a.secret.key": 您可以用来访问 Google GCS 存储桶的 Secret Key。
  - "fs.s3a.endpoint": 您可以用来访问 Google GCS 存储桶的 endpoint。

> **注意**
> StarRocks 仅支持根据 S3A 协议在 Google GCS 中创建存储库。因此，当您在 Google GCS 中创建存储库时，您必须将 `ON LOCATION` 中传递的 GCS URI 的前缀替换为 `s3a://`。

## 示例

示例 1：在 Apache™ Hadoop® 集群中创建名为 `hdfs_repo` 的存储库。

```SQL
CREATE REPOSITORY hdfs_repo
WITH BROKER
ON LOCATION "hdfs://x.x.x.x:yyyy/repo_dir/backup"
PROPERTIES(
    "username" = "xxxxxxxx",
    "password" = "yyyyyyyy"
);
```

示例 2：在 Amazon S3 存储桶 `bucket_s3` 中创建名为 `s3_repo` 的只读存储库。

```SQL
CREATE READ ONLY REPOSITORY s3_repo
WITH BROKER
ON LOCATION "s3a://bucket_s3/backup"
PROPERTIES(
    "aws.s3.access_key" = "XXXXXXXXXXXXXXXXX",
    "aws.s3.secret_key" = "yyyyyyyyyyyyyyyyy",
    "aws.s3.endpoint" = "s3.us-east-1.amazonaws.com"
);
```

示例 3：在 Google GCS 存储桶 `bucket_gcs` 中创建名为 `gcs_repo` 的存储库。

```SQL
CREATE REPOSITORY gcs_repo
WITH BROKER
ON LOCATION "s3a://bucket_gcs/backup"
PROPERTIES(
    "fs.s3a.access.key" = "xxxxxxxxxxxxxxxxxxxx",
    "fs.s3a.secret.key" = "yyyyyyyyyyyyyyyyyyyy",
    "fs.s3a.endpoint" = "storage.googleapis.com"
);
```