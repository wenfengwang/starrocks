---
displayed_sidebar: English
---

# 使用 MinIO 进行共享数据

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/sharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />


## 架构

![共享数据架构](../../assets/share_data_arch.png)

## 部署共享数据 StarRocks 集群

部署共享数据 StarRocks 集群与部署无共享 StarRocks 集群类似。唯一的区别是在共享数据集群中，您需要部署 CN 而不是 BE。本节仅列出在部署共享数据 StarRocks 集群时，需要在 FE 和 CN 的配置文件 **fe.conf** 和 **cn.conf** 中添加的额外 FE 和 CN 配置项。有关部署 StarRocks 集群的详细说明，请参阅[部署 StarRocks](../../deployment/deploy_manually.md)。

### 为共享数据 StarRocks 配置 FE 节点

在启动 FE 之前，在 FE 配置文件 **fe.conf** 中添加以下配置项。

#### run_mode

StarRocks 集群的运行模式。有效值：

- `shared_data`
- `shared_nothing`（默认值）。

> **注意**
> 您不能同时采用 `shared_data` 和 `shared_nothing` 模式部署 StarRocks 集群。不支持混合部署。
> 集群部署后请勿更改 `run_mode`。否则，集群将无法重启。不支持从无共享集群转换为共享数据集群，反之亦然。

#### cloud_native_meta_port

云原生元数据服务 RPC 端口。

- 默认值：`6090`

#### enable_load_volume_from_conf

是否允许 StarRocks 使用 FE 配置文件中指定的对象存储相关属性来创建默认存储卷。有效值：

- `true`（默认值）如果在创建新的共享数据集群时指定此项为 `true`，StarRocks 将使用 FE 配置文件中的对象存储相关属性创建内置存储卷 `builtin_storage_volume`，并将其设置为默认存储卷。但是，如果您没有指定对象存储相关属性，StarRocks 将无法启动。
- `false` 如果在创建新的共享数据集群时指定此项为 `false`，StarRocks 将直接启动而不创建内置存储卷。您必须在 StarRocks 中创建任何对象之前，手动创建一个存储卷并将其设置为默认存储卷。有关详细信息，请参阅[创建默认存储卷](#use-your-shared-data-starrocks-cluster)。

从 v3.1.0 版本开始支持。

> **警告**
> 我们强烈建议您在从 v3.0 版本升级现有共享数据集群时将此项保留为 `true`。如果指定为 `false`，升级前创建的数据库和表将变为只读，您将无法加载数据。

#### cloud_native_storage_type

您使用的对象存储类型。在共享数据模式下，StarRocks 支持将数据存储在 Azure Blob（从 v3.1.1 版本开始支持）和兼容 S3 协议的对象存储（如 AWS S3、Google GCP 和 MinIO）。有效值：

- `S3`（默认值）
- `AZBLOB`。

> 注意
> 如果您将此参数指定为 `S3`，则必须添加以 `aws_s3` 为前缀的参数。
> 如果您将此参数指定为 `AZBLOB`，则必须添加以 `azure_blob` 为前缀的参数。

#### aws_s3_path

用于存储数据的 S3 路径。它由您的 S3 存储桶名称和其下的子路径（如果有）组成，例如 `testbucket/subpath`。

#### aws_s3_endpoint

用于访问您的 S3 存储桶的终端节点，例如 `https://s3.us-west-2.amazonaws.com`。

#### aws_s3_region

您的 S3 存储桶所在的区域，例如 `us-west-2`。

#### aws_s3_use_aws_sdk_default_behavior

是否使用 [AWS SDK 默认凭证提供程序链](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html)。有效值：

- `true`
- `false`（默认值）。

#### aws_s3_use_instance_profile

是否使用实例配置文件和假定角色作为访问 S3 的凭证方法。有效值：

- `true`
- `false`（默认值）。

如果您使用基于 IAM 用户的凭证（Access Key 和 Secret Key）来访问 S3，您必须将此项指定为 `false`，并指定 `aws_s3_access_key` 和 `aws_s3_secret_key`。

如果您使用实例配置文件来访问 S3，您必须将此项指定为 `true`。

如果您使用假定角色来访问 S3，您必须将此项指定为 `true`，并指定 `aws_s3_iam_role_arn`。

如果您使用外部 AWS 账户，您还必须指定 `aws_s3_external_id`。

#### aws_s3_access_key

用于访问您的 S3 存储桶的 Access Key ID。

#### aws_s3_secret_key

用于访问您的 S3 存储桶的 Secret Access Key。

#### aws_s3_iam_role_arn

对存储数据文件的 S3 存储桶拥有权限的 IAM 角色的 ARN。

#### aws_s3_external_id

用于跨账户访问您的 S3 存储桶的外部 AWS 账户的外部 ID。

#### azure_blob_path

用于存储数据的 Azure Blob 存储路径。它由您的存储账户中的容器名称和容器下的子路径（如果有）组成，例如 `testcontainer/subpath`。

#### azure_blob_endpoint

您的 Azure Blob 存储账户的终端节点，例如 `https://test.blob.core.windows.net`。

#### azure_blob_shared_key

用于授权对您的 Azure Blob 存储请求的 Shared Key。

#### azure_blob_sas_token

用于授权对您的 Azure Blob 存储请求的 Shared Access Signatures (SAS)。

> 注意
> 创建共享数据 StarRocks 集群后，只能修改与凭证相关的配置项。如果您更改了原始存储路径相关的配置项，更改前创建的数据库和表将变为只读，您将无法加载数据。

如果集群创建后您想要手动创建默认存储卷，只需要添加以下配置项：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 为共享数据 StarRocks 配置 CN 节点

<SharedDataCNconf />


## 使用您的共享数据 StarRocks 集群

<SharedDataUseIntro />


以下示例为 MinIO 存储桶 `defaultbucket` 创建存储卷 `def_volume`，使用 Access Key 和 Secret Key 凭证，启用该存储卷，并将其设置为默认存储卷：

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "aws.s3.region" = "us-west-2",
    "aws.s3.endpoint" = "https://hostname.domainname.com:portnumber",
    "aws.s3.access_key" = "xxxxxxxxxx",
    "aws.s3.secret_key" = "yyyyyyyyyy"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />
