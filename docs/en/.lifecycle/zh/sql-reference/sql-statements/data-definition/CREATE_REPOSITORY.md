---
displayed_sidebar: English
---

# 创建存储库

## 描述

在远程存储系统中创建一个存储库，用于存储[备份和恢复数据](../../../administration/Backup_and_restore.md)的数据快照。

> **注意**
>
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

| **参数**       | **描述**                                              |
| ------------------- | ------------------------------------------------------------ |
| READ ONLY           | 创建一个只读存储库。请注意，您只能从只读存储库还原数据。在为两个集群创建相同的存储库以迁移数据时，您可以为新集群创建一个只读存储库，并且仅授予 RESTORE 权限。|
| repository_name     | 存储库名称。                                             |
| repository_location | 存储库在远程存储系统中的位置。     |
| PROPERTIES          | 用于访问远程存储系统的凭证方法。 |

**PROPERTIES**：

StarRocks 支持在 HDFS、AWS S3 和 Google GCS 中创建存储库。

- 对于 HDFS：
  - "username"：要用于访问 HDFS 集群NameNode的帐户的用户名。
  - "password"：要用于访问 HDFS 集群NameNode的帐户的密码。

- 对于 AWS S3：
  - "aws.s3.use_instance_profile"：是否允许实例配置文件和假定角色作为访问 AWS S3 的凭证方法。默认值： `false`. 

    - 如果您使用基于 IAM 用户的凭证（访问密钥和秘密密钥）访问 AWS S3，则无需指定此参数，并且您需要指定 "aws.s3.access_key"、"aws.s3.secret_key" 和 "aws.s3.endpoint"。
    - 如果您使用实例配置文件访问 AWS S3，则需要将此参数设置为 `true` 并指定 "aws.s3.region"。
    - 如果您使用假定角色访问 AWS S3，则需要将此参数设置为 `true` 并指定 "aws.s3.iam_role_arn" 和 "aws.s3.region"。
  
  - "aws.s3.access_key"：您可以用它来访问 Amazon S3 存储桶的访问密钥 ID。
  - "aws.s3.secret_key"：您可以用它来访问 Amazon S3 存储桶的秘密访问密钥。
  - "aws.s3.endpoint"：您可以用它来访问 Amazon S3 存储桶的终端节点。
  - "aws.s3.iam_role_arn"：具有权限访问 AWS S3 存储桶中数据文件的 IAM 角色的 ARN。如果您要使用假定角色作为访问 AWS S3 的凭证方法，则必须指定此参数。然后，StarRocks 在使用 Hive 目录分析您的 Hive 数据时将扮演此角色。
  - "aws.s3.region"：您的 AWS S3 存储桶所在的地区。示例： `us-west-1`。

> **注意**
>
> StarRocks 仅支持根据 S3A 协议在 AWS S3 中创建存储库。因此，当您在 AWS S3 中创建存储库时，您必须将 `ON LOCATION` 中传递的 S3 URI 替换为 `s3a://`。

- 对于 Google GCS：
  - "fs.s3a.access.key"：您可以用它来访问 Google GCS 存储桶的访问密钥。
  - "fs.s3a.secret.key"：您可以用它来访问 Google GCS 存储桶的秘密访问密钥。
  - "fs.s3a.endpoint"：您可以用它来访问 Google GCS 存储桶的终端节点。

> **注意**
>
> StarRocks 仅支持根据 S3A 协议在 Google GCS 中创建存储库。因此，当您在 Google GCS 中创建存储库时，您必须将 `ON LOCATION` 中传递的 GCS URI 中的前缀替换为 `s3a://`。

## 例子

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

示例 2：在 Amazon S3 存储桶中创建名为 `s3_repo` 的只读存储库 `bucket_s3`。

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

示例 3：在 Google GCS 存储桶中创建名为 `gcs_repo` 的存储库 `bucket_gcs`。

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
