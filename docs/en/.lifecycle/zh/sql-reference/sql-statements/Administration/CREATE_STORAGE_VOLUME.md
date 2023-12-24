---
displayed_sidebar: English
---

# 创建存储卷

## 描述

创建远程存储系统的存储卷。此功能从 v3.1 版本开始支持。

存储卷包括远程数据存储的属性和凭据信息。在[共享数据 StarRocks 集群](../../../deployment/shared_data/s3.md)中创建数据库和云原生表时，可以引用存储卷。

> **注意**
>
> 只有在 SYSTEM 级别具有 CREATE STORAGE VOLUME 权限的用户才能执行此操作。

## 语法

```SQL
CREATE STORAGE VOLUME [IF NOT EXISTS] <storage_volume_name>
TYPE = { S3 | AZBLOB }
LOCATIONS = ('<remote_storage_path>')
[ COMMENT '<comment_string>' ]
PROPERTIES
("key" = "value",...)
```

## 参数

| **参数**       | **描述**                                              |
| ------------------- | ------------------------------------------------------------ |
| storage_volume_name | 存储卷的名称。请注意，不能创建名为 `builtin_storage_volume` 的存储卷，因为它用于创建内置存储卷。 |
| 类型                | 远程存储系统的类型。有效值： `S3` 和 `AZBLOB`。 `S3` 表示 AWS S3 或兼容 S3 的存储系统。 `AZBLOB` 表示 Azure Blob 存储（从 v3.1.1 开始支持）。 |
| 地点           | 存储位置。格式如下：<ul><li>对于 AWS S3 或兼容 S3 的存储系统： `s3://<s3_path>`. `<s3_path>` 必须是绝对路径，例如 `s3://testbucket/subpath`.</li><li>对于 Azure Blob 存储： `azblob://<azblob_path>`. `<azblob_path>` 必须是绝对路径，例如 `azblob://testcontainer/subpath`.</li></ul> |
| 评论             | 存储卷的注释。                           |
| 属性          | 用于指定访问远程存储系统的属性和凭据信息的 `"key" = "value"` 对。详细信息，请参阅 [PROPERTIES](#properties) 。

### 属性

- 如果使用 AWS S3：

  - 如果使用 AWS SDK 的默认身份验证凭证访问 S3，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "true"
    ```

  - 如果使用基于 IAM 用户的凭证（访问密钥和私有密钥）访问 S3，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<access_key>",
    "aws.s3.secret_key" = "<secrete_key>"
    ```

  - 如果使用实例配置文件访问 S3，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true"
    ```

  - 如果使用假定角色访问 S3，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "<role_arn>"
    ```

  - 如果使用假定角色从外部 AWS 账户访问 S3，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "<role_arn>",
    "aws.s3.external_id" = "<external_id>"
    ```

- 如果使用 GCP Cloud Storage，请设置以下属性：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例如：us-east-1
  "aws.s3.region" = "<region>",
  
  -- 例如：https://storage.googleapis.com
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secrete_key>"
  ```

- 如果使用 MinIO，请设置以下属性：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例如：us-east-1
  "aws.s3.region" = "<region>",
  
  -- 例如：http://172.26.xx.xxx:39000
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secrete_key>"
  ```

  | **属性**                        | **描述**                                              |
  | ----------------------------------- | ------------------------------------------------------------ |
  | 启用                             | 是否启用此存储卷。默认值： `false`。禁用的存储卷无法被引用。 |
  | aws.s3.region                       | 您的 S3 存储桶所在的区域，例如 `us-west-2`。 |
  | aws.s3.endpoint                     | 用于访问您的 S3 存储桶的终端节点 URL，例如 `https://s3.us-west-2.amazonaws.com`。 |
  | aws.s3.use_aws_sdk_default_behavior | 是否使用 AWS SDK 的默认身份验证凭证。有效值： `true` 和 `false`（默认值）。 |
  | aws.s3.use_instance_profile         | 是否使用实例配置文件和假定角色作为访问 S3 的凭证方法。有效值： `true` 和 `false`（默认值）。<ul><li>如果使用基于 IAM 用户的凭证（访问密钥和私有密钥）访问 S3，则必须将此项指定为 `false`，并指定 `aws.s3.access_key` 和 `aws.s3.secret_key`。</li><li>如果使用实例配置文件访问 S3，则必须将此项指定为 `true`。</li><li>如果使用假定角色访问 S3，则必须将此项指定为 `true`，并指定 `aws.s3.iam_role_arn`。</li><li>如果使用外部 AWS 账户，则必须将此项指定为 `true`，并指定 `aws.s3.iam_role_arn` 和 `aws.s3.external_id`。</li></ul> |
  | aws.s3.access_key                   | 用于访问您的 S3 存储桶的访问密钥 ID。             |
  | aws.s3.secret_key                   | 用于访问您的 S3 存储桶的秘密访问密钥。         |
  | aws.s3.iam_role_arn                 | 具有权限访问您的 S3 存储桶中存储数据文件的 IAM 角色的 ARN。 |
  | aws.s3.external_id                  | 用于跨账户访问您的 S3 存储桶的 AWS 账户的外部 ID。 |

- 如果使用 Azure Blob 存储（从 v3.1.1 开始支持）：

  - 如果使用共享密钥访问 Azure Blob 存储，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.shared_key" = "<shared_key>"
    ```

  - 如果使用共享访问签名（SAS）访问 Azure Blob 存储，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.sas_token" = "<sas_token>"
    ```

  > **注意**
  >
  > 创建 Azure Blob 存储帐户时，必须禁用分层命名空间。

  | **属性**          | **描述**                                              |
  | --------------------- | ------------------------------------------------------------ |
  | 启用               | 是否启用此存储卷。默认值： `false`。禁用的存储卷无法被引用。 |
  | azure.blob.endpoint   | 您的 Azure Blob 存储帐户的终结点，例如 `https://test.blob.core.windows.net`。 |
  | azure.blob.shared_key | 用于授权请求您的 Azure Blob 存储的共享密钥。 |
  | azure.blob.sas_token  | 用于授权请求您的 Azure Blob 存储的共享访问签名（SAS）。 |

## 例子

示例 1：为 AWS S3 存储桶 `defaultbucket` 创建存储卷 `my_s3_volume`，使用基于 IAM 用户的凭证（访问密钥和私有密钥）访问 S3，并启用它。

```SQL
CREATE STORAGE VOLUME my_s3_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "aws.s3.region" = "us-west-2",
    "aws.s3.endpoint" = "https://s3.us-west-2.amazonaws.com",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "xxxxxxxxxx",
    "aws.s3.secret_key" = "yyyyyyyyyy"
);
```

## 相关 SQL 语句

- [更改存储卷](./ALTER_STORAGE_VOLUME.md)
- [丢弃存储卷](./DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC 存储卷](./DESC_STORAGE_VOLUME.md)
- [显示存储卷](./SHOW_STORAGE_VOLUMES.md)
