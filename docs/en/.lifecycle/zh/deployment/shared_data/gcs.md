---
displayed_sidebar: English
---

# 使用 GCS 部署 StarRocks

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />


## 架构

![共享数据架构](../../assets/share_data_arch.png)

## 部署共享数据 StarRocks 集群

部署共享数据 StarRocks 集群与部署无共享 StarRocks 集群类似。唯一的区别是在共享数据集群中需要部署 CN 而不是 BE。本节只列出了在部署共享数据 StarRocks 集群时，需要在 FE 和 CN 的配置文件 **fe.conf** 和 **cn.conf** 中添加的额外 FE 和 CN 配置项。有关部署 StarRocks 集群的详细说明，请参阅[部署 StarRocks](../../deployment/deploy_manually.md)。

> **注意**
> 请勿在本文档的下一部分配置集群为共享存储之前启动集群。

## 为共享数据 StarRocks 配置 FE 节点

在启动集群之前，请先配置 FE 和 CN。下面提供了一个配置示例，随后会详细介绍每个参数。

### GCS 的 FE 配置示例

您可以将以下 `fe.conf` 的共享数据添加示例添加到每个 FE 节点的 `fe.conf` 文件中。因为 GCS 存储是通过 [Cloud Storage XML API](https://cloud.google.com/storage/docs/xml-api/overview) 访问的，所以参数使用了 `aws_s3` 前缀。

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

### 与 GCS 共享存储相关的所有 FE 参数

#### run_mode

StarRocks 集群的运行模式。有效值：

- `shared_data`
- `shared_nothing`（默认值）。

> **注意**
> 不能同时为 StarRocks 集群采用 `shared_data` 和 `shared_nothing` 模式。不支持混合部署。
> 集群部署后不要更改 `run_mode`，否则集群无法重启。不支持从无共享集群转换为共享数据集群，反之亦然。

#### cloud_native_meta_port

云原生元数据服务 RPC 端口。

- 默认值：`6090`

#### enable_load_volume_from_conf

是否允许 StarRocks 使用 FE 配置文件中指定的对象存储相关属性来创建默认存储卷。有效值：

- `true`（默认值）如果在创建新的共享数据集群时将此项设置为 `true`，StarRocks 将使用 FE 配置文件中的对象存储相关属性创建内置存储卷 `builtin_storage_volume` 并将其设置为默认存储卷。但是，如果您未指定对象存储相关属性，StarRocks 将无法启动。
- `false` 如果在创建新的共享数据集群时将此项设置为 `false`，StarRocks 将直接启动而不创建内置存储卷。您必须手动创建存储卷并在 StarRocks 中创建任何对象之前将其设置为默认存储卷。更多信息请参阅[创建默认存储卷](#use-your-shared-data-starrocks-cluster)。

从 v3.1.0 版本开始支持。

> **警告**
> 我们强烈建议您在从 v3.0 版本升级现有共享数据集群时保留此项为 `true`。如果您将此项设置为 `false`，升级前创建的数据库和表将变为只读，您将无法向其中加载数据。

#### cloud_native_storage_type

您使用的对象存储类型。在共享数据模式下，StarRocks 支持将数据存储在 Azure Blob（从 v3.1.1 版本开始支持）和兼容 S3 协议的对象存储（如 AWS S3、Google GCS 和 MinIO）。有效值：

- `S3`（默认值）
- `AZBLOB`

#### aws_s3_path

用于存储数据的 S3 路径。它由您的 S3 存储桶名称和其下的子路径（如果有）组成，例如 `testbucket/subpath`。

#### aws_s3_endpoint

用于访问您的 S3 存储桶的端点，例如 `https://storage.googleapis.com/`

#### aws_s3_region

您的 S3 存储桶所在的区域，例如 `us-west-2`。

#### aws_s3_use_instance_profile

是否使用实例配置文件和假定角色作为访问 GCS 的凭证方法。有效值：

- `true`
- `false`（默认值）。

如果您使用基于 IAM 用户的凭证（访问密钥和秘密密钥）来访问 GCS，您必须将此项设置为 `false` 并指定 `aws_s3_access_key` 和 `aws_s3_secret_key`。

如果您使用实例配置文件来访问 GCS，您必须将此项设置为 `true`。

如果您使用假定角色来访问 GCS，您必须将此项设置为 `true` 并指定 `aws_s3_iam_role_arn`。

如果您使用外部 AWS 账户，您还必须指定 `aws_s3_external_id`。

#### aws_s3_access_key

用于访问 GCS 存储桶的 HMAC 访问密钥 ID。

#### aws_s3_secret_key

用于访问 GCS 存储桶的 HMAC 秘密访问密钥。

#### aws_s3_iam_role_arn

对存储数据文件的 GCS 存储桶拥有权限的 IAM 角色的 ARN。

#### aws_s3_external_id

用于跨账户访问 GCS 存储桶的 AWS 账户的外部 ID。

> **注意**
> 创建共享数据 StarRocks 集群后，只能修改与凭证相关的配置项。如果您更改了原始存储路径相关的配置项，更改前创建的数据库和表将变为只读，您将无法向其中加载数据。

如果您希望在集群创建后手动创建默认存储卷，只需添加以下配置项：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 为共享数据 StarRocks 配置 CN 节点

<SharedDataCNconf />


## 使用您的共享数据 StarRocks 集群

<SharedDataUseIntro />


以下示例为 GCS 存储桶 `defaultbucket` 创建存储卷 `def_volume`，使用 HMAC 访问密钥和秘密密钥，启用该存储卷，并将其设置为默认存储卷：

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
