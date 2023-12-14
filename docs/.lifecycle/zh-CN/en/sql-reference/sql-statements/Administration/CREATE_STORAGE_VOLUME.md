---
displayed_sidebar: "Chinese"
---

# 创建存储卷

## 描述

为远程存储系统创建存储卷。此功能从v3.1版开始支持。

存储卷包括远程数据存储的属性和凭据信息。您可以在[共享数据的 StarRocks 集群](../../../deployment/shared_data/s3.md)中创建数据库和云原生表时引用存储卷。

> **注意**
>
> 只有在系统级别具有CREATE STORAGE VOLUME权限的用户才能执行此操作。

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

| **参数**             | **描述**                                                     |
| --------------------- | ------------------------------------------------------------- |
| storage_volume_name  | 存储卷的名称。请注意，您不能创建名为`builtin_storage_volume`的存储卷，因为它用于创建内置存储卷。 |
| TYPE                 | 远程存储系统的类型。有效值：`S3`和`AZBLOB`。`S3`表示AWS S3或与S3兼容的存储系统。`AZBLOB`表示Azure Blob 存储（从v3.1.1版开始支持）。 |
| LOCATIONS            | 存储位置。格式如下：<ul><li>对于AWS S3或与S3协议兼容的存储系统：`s3://<s3_path>`。`<s3_path>`必须是绝对路径，例如，`s3://testbucket/subpath`。</li><li>对于Azure Blob 存储：`azblob://<azblob_path>`。`<azblob_path>`必须是绝对路径，例如，`azblob://testcontainer/subpath`。</li></ul> |
| COMMENT              | 存储卷的注释。                                              |
| PROPERTIES           | 用于指定访问远程存储系统的属性和凭据信息的“key”=“value”对参数。有关详细信息，请参阅[PROPERTIES](#属性)。 |

### PROPERTIES

- 如果您使用AWS S3：

  - 如果您使用AWS SDK的默认身份验证凭据访问S3，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "true"
    ```

  - 如果您使用基于IAM用户的凭据（访问密钥和密码）访问S3，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<access_key>",
    "aws.s3.secret_key" = "<secrete_key>"
    ```

  - 如果您使用实例配置文件访问S3，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true"
    ```

  - 如果您使用假定角色访问S3，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "<role_arn>"
    ```

  - 如果您使用假定角色从外部AWS帐户访问S3，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "<role_arn>",
    "aws.s3.external_id" = "<external_id>"
    ```

- 如果您使用GCP Cloud Storage，请设置以下属性：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例如：us-east-1
  "aws.s3.region" = "<region>",
  
  -- 例如：https://storage.googleapis.com
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secrete_key>"
  ```

- 如果您使用MinIO，请设置以下属性：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例如：us-east-1
  "aws.s3.region" = "<region>",
  
  -- 例如：http://172.26.xx.xxx:39000
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secrete_key>"
  ```

  | **属性**                           | **描述**                                                     |
  | ---------------------------------- | ------------------------------------------------------------- |
  | enabled                            | 是否启用此存储卷。默认：`false`。禁用的存储卷无法被引用。  |
  | aws.s3.region                      | 您的S3存储桶所在的区域，例如，`us-west-2`。                  |
  | aws.s3.endpoint                    | 用于访问您的S3存储桶的端点URL，例如，`https://s3.us-west-2.amazonaws.com`。 |
  | aws.s3.use_aws_sdk_default_behavior | 是否使用AWS SDK的默认身份验证凭据。有效值：`true`和`false`（默认）。 |
  | aws.s3.use_instance_profile         | 是否使用实例配置文件和假定角色作为访问S3的凭据方法。有效值：`true`和`false`（默认）。<ul><li>如果您使用基于IAM用户的凭据（访问密钥和密码）访问S3，必须将此项指定为`false`，并指定`aws.s3.access_key`和`aws.s3.secret_key`。</li><li>如果您使用实例配置文件访问S3，必须将此项指定为`true`。</li><li>如果您使用假定角色访问S3，必须将此项指定为`true`，并指定`aws.s3.iam_role_arn`。</li><li>如果您使用外部AWS帐户，必须将此项指定为`true`，并指定`aws.s3.iam_role_arn`和`aws.s3.external_id`。</li></ul> |
  | aws.s3.access_key                  | 用于访问您的S3存储桶的Access Key ID。                        |
  | aws.s3.secret_key                  | 用于访问您的S3存储桶的Secret Access Key。                    |
  | aws.s3.iam_role_arn                | 具有权限访问您的S3存储桶中数据文件的IAM角色的ARN。            |
  | aws.s3.external_id                 | 用于跨帐户访问您的S3存储桶的AWS帐户的外部ID。               |

- 如果您使用Azure Blob 存储（从v3.1.1版开始支持）：

  - 如果您使用Shared Key访问Azure Blob 存储，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.shared_key" = "<shared_key>"
    ```

  - 如果您使用共享访问签名（SAS）访问Azure Blob 存储，请设置以下属性：

    ```SQL
    "enabled" = "{ true | false }",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.sas_token" = "<sas_token>"
    ```

  > **注意**
  >
  > 创建Azure Blob 存储账户时，必须禁用分层命名空间。

  | **属性**               | **描述**                                                     |
  | ---------------------- | ------------------------------------------------------------- |
  | enabled                | 是否启用此存储卷。默认：`false`。禁用的存储卷无法被引用。 |
  | azure.blob.endpoint    | 您的Azure Blob 存储账户的端点，例如，`https://test.blob.core.windows.net`。 |
  | azure.blob.shared_key  | 用于授权您的Azure Blob 存储请求的共享密钥。                  |
  | azure.blob.sas_token   | 用于授权您的Azure Blob 存储请求的共享访问签名（SAS）。        |

## 示例

示例1：为AWS S3存储桶`defaultbucket`创建名为`my_s3_volume`的存储卷，使用基于IAM用户的凭据（访问密钥和密码）访问S3，并将其启用。

```SQL
CREATE STORAGE VOLUME my_s3_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
属性
(
    "aws.s3.region" = "us-west-2",
    "aws.s3.endpoint" = "https://s3.us-west-2.amazonaws.com",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "xxxxxxxxxx",
    "aws.s3.secret_key" = "yyyyyyyyyy"
);


## 相关的SQL语句

- [修改存储卷](./ALTER_STORAGE_VOLUME.md)
- [删除存储卷](./DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [描述存储卷](./DESC_STORAGE_VOLUME.md)
- [显示存储卷](./SHOW_STORAGE_VOLUMES.md)