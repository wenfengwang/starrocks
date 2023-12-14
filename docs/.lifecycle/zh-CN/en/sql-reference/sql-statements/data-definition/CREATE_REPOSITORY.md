---
displayed_sidebar: "Chinese"
---

# 创建存储库

## 描述

在远程存储系统中创建一个仓库，用于存储[备份和恢复数据](../../../administration/Backup_and_restore.md)的数据快照。

> **注意**
>
> 只有具有ADMIN权限的用户才能创建仓库。

有关删除仓库的详细说明，请参阅[DROP REPOSITORY](../data-definition/DROP_REPOSITORY.md)。

## 语法

```SQL
CREATE [READ ONLY] REPOSITORY <repository_name>
WITH BROKER
ON LOCATION "<repository_location>"
PROPERTIES ("key"="value", ...)
```

## 参数

| **参数**             | **描述**                                                     |
| ------------------- | ------------------------------------------------------------ |
| READ ONLY           | 创建一个只读仓库。请注意，您只能从只读仓库中恢复数据。当为两个集群创建相同的仓库以迁移数据时，您可以为新集群创建一个只读仓库，并且只授予恢复权限。|
| repository_name     | 仓库名称。                                                   |
| repository_location | 仓库在远程存储系统中的位置。                                 |
| PROPERTIES          | 访问远程存储系统的凭据方法。                                |

**PROPERTIES**:

StarRocks支持在HDFS、AWS S3和Google GCS中创建仓库。

- 对于HDFS:
  - "username": 要用于访问HDFS集群的NameNode的帐户的用户名。
  - "password": 要用于访问HDFS集群的NameNode的帐户的密码。

- 对于AWS S3:
  - "aws.s3.use_instance_profile": 是否允许使用实例配置文件和假定角色作为访问AWS S3的凭据方法。默认值：`false`。

    - 如果您使用基于IAM用户的凭据（访问密钥和秘密密钥）来访问AWS S3，则不需要指定此参数，并且您需要指定"aws.s3.access_key"、"aws.s3.secret_key"和"aws.s3.endpoint"。
    - 如果您使用实例配置文件访问AWS S3，则需要将此参数设置为`true`，并指定"aws.s3.region"。
    - 如果您使用假定角色访问AWS S3，则需要将此参数设置为`true`，并指定"aws.s3.iam_role_arn"和"aws.s3.region"。
  
  - "aws.s3.access_key": 用于访问Amazon S3存储桶的访问密钥ID。
  - "aws.s3.secret_key": 用于访问Amazon S3存储桶的秘密访问密钥。
  - "aws.s3.endpoint": 用于访问Amazon S3存储桶的终端点。
  - "aws.s3.iam_role_arn": 具有权限在您的数据文件存储的AWS S3存储桶上的IAM角色的ARN。如果您希望使用假定角色作为访问AWS S3的凭据方法，必须指定此参数。然后，StarRocks在使用Hive目录分析您的Hive数据时将假定此角色。
  - "aws.s3.region": 您的AWS S3存储桶所在的区域。示例：`us-west-1`。

> **注意**
>
> StarRocks仅支持根据S3A协议在AWS S3中创建仓库。因此，在AWS S3中创建仓库时，您必须将`ON LOCATION`中作为仓库位置传递的S3 URI中的`s3://`替换为`s3a://`。

- 对于Google GCS:
  - "fs.s3a.access.key": 用于访问Google GCS存储桶的访问密钥。
  - "fs.s3a.secret.key": 用于访问Google GCS存储桶的秘密访问密钥。
  - "fs.s3a.endpoint": 用于访问Google GCS存储桶的终端点。

> **注意**
>
> StarRocks仅支持根据S3A协议在Google GCS中创建仓库。因此，在Google GCS中创建仓库时，您必须将作为仓库位置传递的GCS URI中的前缀替换为`s3a://`。

## 示例

示例1：在Apache™ Hadoop®集群中创建名为`hdfs_repo`的仓库。

```SQL
CREATE REPOSITORY hdfs_repo
WITH BROKER
ON LOCATION "hdfs://x.x.x.x:yyyy/repo_dir/backup"
PROPERTIES(
    "username" = "xxxxxxxx",
    "password" = "yyyyyyyy"
);
```

示例2：在Amazon S3存储桶`bucket_s3`中创建名为`s3_repo`的只读仓库。

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

示例3：在Google GCS存储桶`bucket_gcs`中创建名为`gcs_repo`的仓库。

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