---
displayed_sidebar: "Chinese"
---

# 使用GCS部署StarRocks

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## 架构

![共享数据架构](../../assets/share_data_arch.png)

## 部署共享数据StarRocks集群

共享数据StarRocks集群的部署与共享无数据StarRocks集群类似。唯一的区别是您需要在共享数据集群中部署CN（而不是BE）。本节仅列出在部署共享数据StarRocks集群时，需要在FE和CN的配置文件**fe.conf**和**cn.conf**中添加的额外FE和CN配置项。有关部署StarRocks集群的详细说明，请参见[部署StarRocks](../../deployment/deploy_manually.md)。

> **注意**
>
> 在进行了本文档下一节中的共享存储配置之后再启动集群。

## 为共享数据StarRocks配置FE节点

在启动集群之前配置FE和CN。以下是示例配置，然后提供每个参数的详细信息。

### GCS的FE示例配置

您可以将共享数据的示例添加到每个FE节点上的`fe.conf`文件中。因为GCS存储是使用[Cloud Storage XML API](https://cloud.google.com/storage/docs/xml-api/overview)访问的，所以参数使用前缀`aws_s3`。

  ```Properties
  run_mode = shared_data

  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = S3

  # 例如，testbucket/subpath

  aws_s3_path = <s3_path>


  # 例如：us-east1
  aws_s3_region = <region>

  # 例如：https://storage.googleapis.com
  aws_s3_endpoint = <endpoint_url>

  aws_s3_access_key = <HMAC access_key>
  aws_s3_secret_key = <HMAC secret_key>
  ```

### 与GCS共享存储相关的所有FE参数

#### run_mode

StarRocks集群的运行模式。有效值：

- `shared_data` 
- `shared_nothing`（默认）。

> **注意**
>
> 无法同时为StarRocks集群采用`shared_data`和`shared_nothing`模式。不支持混合部署。
>
> 在集群部署后不要更改`run_mode`。否则，集群将无法重新启动。不支持从共享无数据集群或共享数据集群进行转换。

#### cloud_native_meta_port

云原生meta服务RPC端口。

- 默认值：`6090`

#### enable_load_volume_from_conf

是否允许StarRocks使用FE配置文件中指定的对象存储相关属性创建默认存储卷。有效值：

- `true`（默认）：如果在创建新的共享数据集群时将此项指定为`true`，StarRocks将使用FE配置文件中的对象存储相关属性创建内置存储卷`builtin_storage_volume`，并将其设置为默认存储卷。但是，如果您尚未指定对象存储相关属性，StarRocks将无法启动。
- `false`：如果在创建新的共享数据集群时将此项指定为`false`，StarRocks将在不创建内置存储卷的情况下直接启动。在创建StarRocks中的任何对象之前，您必须手动创建存储卷并将其设置为默认存储卷。有关更多信息，请参见[创建默认存储卷](#create-default-storage-volume)。

从v3.1.0版本开始支持。

> **注意**
>
> 我们强烈建议您在从v3.0升级现有的共享数据集群时将此项保留为`true`。如果将此项指定为`false`，则在升级前创建的数据库和表将变为只读，您无法向其加载数据。

#### cloud_native_storage_type

您使用的对象存储类型。在共享数据模式下，StarRocks支持将数据存储在Azure Blob（从v3.1.1版本开始支持）和与S3协议兼容的对象存储中（例如AWS S3、Google GCS和MinIO）。有效值：

- `S3`（默认）
- `AZBLOB`

#### aws_s3_path

用于存储数据的S3路径。它由您的S3存储桶名称和其中的子路径（如果有）组成，例如，`testbucket/subpath`。

#### aws_s3_endpoint

用于访问您的S3存储桶的终点，例如，`https://storage.googleapis.com/`。

#### aws_s3_region

您的S3存储桶所在的地区，例如，`us-west-2`。

#### aws_s3_use_instance_profile

用于访问GCS的实例配置和假定角色作为凭据方法。有效值：

- `true`
- `false`（默认）。

如果使用IAM用户凭证（访问密钥和秘密密钥）访问GCS，则必须将此项指定为`false`，并指定`aws_s3_access_key`和`aws_s3_secret_key`。

如果使用实例配置访问GCS，则必须将此项指定为`true`。

如果使用假定角色访问GCS，则必须指定此项为`true`，并指定`aws_s3_iam_role_arn`。

如果使用外部AWS帐户，则还必须指定`aws_s3_external_id`。

#### aws_s3_access_key

用于访问您的GCS存储桶的HMAC访问密钥ID。

#### aws_s3_secret_key

用于访问您的GCS存储桶的HMAC秘密访问密钥。

#### aws_s3_iam_role_arn

具有对您的GCS存储桶中存储数据文件权限的IAM角色的ARN。

#### aws_s3_external_id

用于跨帐户访问您的GCS存储桶的AWS帐户外部ID。

> **注意**
>
> 仅允许在创建共享数据StarRocks集群之后修改与凭证相关的配置项。如果更改了原始存储路径相关的配置项，那么在更改之前创建的数据库和表将变为只读，您无法向其加载数据。

如果希望在集群创建后手动创建默认存储卷，您只需要添加以下配置项：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 为共享数据StarRocks配置CN节点
<SharedDataCNconf />

## 使用您的共享数据StarRocks集群

<SharedDataUseIntro />

以下示例为名为`defaultbucket`的GCS存储桶创建了存储卷`def_volume`，并使用HMAC访问密钥和秘密访问密钥启用了存储卷，并将其设置为默认存储卷：

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "aws.s3.region" = "us-east1",
    "aws.s3.endpoint" = "https://storage.googleapis.com",
    "aws.s3.access_key" = "<HMAC access key>",
    "aws.s3.secret_key" = "<HMAC secret key>"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />