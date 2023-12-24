---
displayed_sidebar: English
---

# 使用 MinIO 进行共享数据

从 '../../assets/commonMarkdown/sharedDataIntro.md' 导入 SharedDataIntro
从 '../../assets/commonMarkdown/sharedDataCNconf.md' 导入 SharedDataCNconf
从 '../../assets/commonMarkdown/sharedDataUseIntro.md' 导入 SharedDataUseIntro
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## 架构

![共享数据架构](../../assets/share_data_arch.png)

## 部署共享数据 StarRocks 集群

共享数据 StarRocks 集群的部署与无共享 StarRocks 集群类似。唯一的区别是，在共享数据集群中，您需要部署 CN 节点而不是 BE 节点。本节仅列出在部署共享数据 StarRocks 集群时，需要在 FE 和 CN 的配置文件 **fe.conf** 和 **cn.conf** 中添加的额外配置项。有关部署 StarRocks 集群的详细说明，请参见[部署 StarRocks](../../deployment/deploy_manually.md)。

### 为共享数据 StarRocks 配置 FE 节点

在启动 FE 之前，请在 FE 配置文件 **fe.conf** 中添加以下配置项。

#### run_mode

StarRocks 集群的运行模式。有效值：

- `shared_data`
- `shared_nothing`（默认）。

> **注意**
>
> 您不能同时为 StarRocks 集群采用 `shared_data` 和 `shared_nothing` 模式。不支持混合部署。
>
> 在部署集群后，请勿更改 `run_mode`。否则，集群将无法重新启动。不支持从无共享集群转换为共享数据集群，反之亦然。

#### cloud_native_meta_port

云原生元服务 RPC 端口。

- 默认：`6090`

#### enable_load_volume_from_conf

是否允许 StarRocks使用 FE 配置文件中指定的对象存储相关属性创建默认存储卷。有效值：

- `true`（默认）如果在创建新的共享数据集群时将此项指定为 `true`，StarRocks 将使用 FE 配置文件中的对象存储相关属性创建内置存储卷 `builtin_storage_volume`，并将其设置为默认存储卷。但是，如果您未指定对象存储相关属性，StarRocks 将无法启动。
- `false`如果在创建新的共享数据集群时将此项指定为 `false`，StarRocks 将直接启动，而不会创建内置存储卷。在创建 StarRocks 中的任何对象之前，您必须手动创建存储卷并将其设置为默认存储卷。有关详细信息，请参见[创建默认存储卷](#use-your-shared-data-starrocks-cluster)。

从 v3.1.0 版本开始支持。

> **注意**
>
> 我们强烈建议您在升级现有共享数据集群（从 v3.0 版本）时将此项保留为 `true`。如果将此项指定为 `false`，则升级前创建的数据库和表将变为只读状态，无法向其中加载数据。

#### cloud_native_storage_type

您使用的对象存储类型。在共享数据模式下，StarRocks 支持将数据存储在 Azure Blob（从 v3.1.1 版本开始支持），以及兼容 S3 协议的对象存储（如 AWS S3、Google GCP 和 MinIO）。有效值：

- `S3`（默认）
- `AZBLOB`。

> 注意
>
> 如果将此参数指定为 `S3`，则必须添加以 `aws_s3` 为前缀的参数。
>
> 如果将此参数指定为 `AZBLOB`，则必须添加以 `azure_blob` 为前缀的参数。

#### aws_s3_path

用于存储数据的 S3 路径。由您的 S3 存储桶名称和其中的子路径（如果有）组成，例如 `testbucket/subpath`。

#### aws_s3_endpoint

用于访问您的 S3 存储桶的终端节点，例如 `https://s3.us-west-2.amazonaws.com`。

#### aws_s3_region

您的 S3 存储桶所在的地区，例如 `us-west-2`。

#### aws_s3_use_aws_sdk_default_behavior

是否使用 [AWS SDK 默认凭证提供程序链](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html)。有效值：

- `true`
- `false`（默认）。

#### aws_s3_use_instance_profile

是否使用实例配置文件和假定角色作为访问 S3 的凭证方法。有效值：

- `true`
- `false`（默认）。

如果您使用基于 IAM 用户的凭证（访问密钥和秘密密钥）访问 S3，则必须将此项指定为 `false`，并指定 `aws_s3_access_key` 和 `aws_s3_secret_key`。

如果您使用实例配置文件访问 S3，则必须将此项指定为 `true`。

如果您使用假定角色访问 S3，则必须将此项指定为 `true`，并指定 `aws_s3_iam_role_arn`。

如果您使用外部 AWS 账户，还必须指定 `aws_s3_external_id`。

#### aws_s3_access_key

用于访问您的 S3 存储桶的访问密钥 ID。

#### aws_s3_secret_key

用于访问您的 S3 存储桶的秘密访问密钥。

#### aws_s3_iam_role_arn

具有对您的 S3 存储桶中存储数据文件的权限的 IAM 角色的 ARN。

#### aws_s3_external_id

用于跨账户访问您的 S3 存储桶的 AWS 账户的外部 ID。

#### azure_blob_path

用于存储数据的 Azure Blob 存储路径。由存储帐户中的容器名称和其中的子路径（如果有）组成，例如 `testcontainer/subpath`。

#### azure_blob_endpoint

您的 Azure Blob 存储帐户的终结点，例如 `https://test.blob.core.windows.net`。

#### azure_blob_shared_key

用于授权对您的 Azure Blob 存储发出请求的共享密钥。

#### azure_blob_sas_token

用于授权对您的 Azure Blob 存储发出请求的共享访问签名（SAS）。

> **注意**
>
> StarRocks 共享数据集群创建后，只能修改与凭证相关的配置项。如果更改了原始存储路径相关的配置项，那么更改前创建的数据库和表将变为只读状态，无法向其中加载数据。

如果要在创建集群后手动创建默认存储卷，只需添加以下配置项：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 为共享数据 StarRocks 配置 CN 节点

<SharedDataCNconf />

## 使用您的共享数据 StarRocks 集群

<SharedDataUseIntro />

以下示例为 MinIO 存储桶 `defaultbucket` 创建名为 `def_volume` 的存储卷，使用访问密钥和秘密密钥凭证，启用存储卷，并将其设置为默认存储卷：

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
