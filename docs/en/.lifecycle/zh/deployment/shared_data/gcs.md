---
displayed_sidebar: English
---

# 使用 GCS 部署 StarRocks

从 '../../assets/commonMarkdown/sharedDataIntro.md' 导入 SharedDataIntro
从 '../../assets/commonMarkdown/sharedDataCNconf.md' 导入 SharedDataCNconf
从 '../../assets/commonMarkdown/sharedDataUseIntro.md' 导入 SharedDataUseIntro
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />

## 架构

![共享数据架构](../../assets/share_data_arch.png)

## 部署共享数据 StarRocks 集群

共享数据 StarRocks 集群的部署方式与无共享 StarRocks 集群类似。唯一的区别是，您需要在共享数据集群中部署 CN 而不是 BE。本小节仅列出部署共享数据 StarRocks 集群时，需要在 FE 和 CN 的配置文件 **fe.conf** 和 **cn.conf** 中添加的额外 FE 和 CN 配置项。有关部署 StarRocks 集群的详细说明，请参见[部署 StarRocks](../../deployment/deploy_manually.md)。

> **注意**
>
> 在本文档的下一节中，在为共享存储配置群集之前，不要启动群集。

## 配置共享数据 StarRocks 的 FE 节点

在启动集群之前，请先配置 FE 和 CN。下面提供了一个示例配置，然后提供了每个参数的详细信息。

### GCS 的 FE 配置示例

可以将共享数据的示例添加到每个 FE 节点的 `fe.conf` 文件中。由于 GCS 存储是使用 [Cloud Storage XML API](https://cloud.google.com/storage/docs/xml-api/overview) 访问的，参数使用前缀 `aws_s3`。

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
- `shared_nothing` （默认）。

> **注意**
>
> 不能同时为 StarRocks 集群采用 `shared_data` 和 `shared_nothing` 模式。不支持混合部署。
>
> 集群部署后，请勿更改 `run_mode`。否则，集群将无法重新启动。不支持从无共享集群转换为共享数据集群，反之亦然。

#### cloud_native_meta_port

云原生元服务 RPC 端口。

- 默认：`6090`

#### enable_load_volume_from_conf

是否允许 StarRocks使用 FE 配置文件中指定的对象存储相关属性创建默认存储卷。有效值：

- `true`（默认）如果在创建新的共享数据集群时将此项指定为 `true`，StarRocks 将使用 FE 配置文件中的对象存储相关属性创建内置存储卷 `builtin_storage_volume`，并将其设置为默认存储卷。但是，如果您没有指定对象存储相关属性，StarRocks 将无法启动。
- `false`如果在创建新的共享数据集群时将此项指定为 `false`，StarRocks 将直接启动，无需创建内置存储卷。在创建 StarRocks 中的任何对象之前，您必须手动创建存储卷并将其设置为默认存储卷。有关详细信息，请参见[创建默认存储卷](#use-your-shared-data-starrocks-cluster)。

从 v3.1.0 开始支持。

> **注意**
>
> 我们强烈建议您在升级现有共享数据群集时保留此项为 `true`。如果将此项指定为 `false`，则升级前创建的数据库和表将变为只读，并且无法将数据加载到其中。

#### cloud_native_storage_type

您使用的对象存储类型。在共享数据模式下，StarRocks 支持将数据存储在 Azure Blob（从 v3.1.1 开始支持）和兼容 S3 协议的对象存储（如 AWS S3、Google GCS、MinIO 等）中。有效值：

- `S3`（默认）
- `AZBLOB`

#### aws_s3_path

用于存储数据的 S3 路径。由您的 S3 存储桶名称和其中的子路径（如果有）组成，例如 `testbucket/subpath`。

#### aws_s3_endpoint

用于访问您的 S3 存储桶的终端节点，例如 `https://storage.googleapis.com/`

#### aws_s3_region

您的 S3 存储桶所在的区域，例如 `us-west-2`。

#### aws_s3_use_instance_profile

是否使用实例配置文件和假定角色作为访问 GCS 的凭据方法。有效值：

- `true`
- `false`（默认）。

如果您使用 IAM 用户凭据（访问密钥和私有密钥）访问 GCS，则必须将此项指定为 `false`，并指定 `aws_s3_access_key` 和 `aws_s3_secret_key`。

如果使用实例配置文件访问 GCS，则必须将此项指定为 `true`。

如果使用假定角色访问 GCS，则必须将此项指定为 `true`，并指定 `aws_s3_iam_role_arn`。

如果您使用外部 AWS 账户，则还必须指定 `aws_s3_external_id`。

#### aws_s3_access_key

用于访问您的 GCS 存储桶的 HMAC 访问密钥 ID。

#### aws_s3_secret_key

用于访问您的 GCS 存储桶的 HMAC 私有访问密钥。

#### aws_s3_iam_role_arn

具有权限访问您的 GCS 存储桶中数据文件的 IAM 角色的 ARN。

#### aws_s3_external_id

用于跨账户访问您的 GCS 存储桶的 AWS 账户的外部 ID。

> **注意**
>
> StarRocks 共享数据集群创建成功后，只能修改与凭据相关的配置项。如果更改了原有的存储路径相关配置项，则更改前创建的数据库和表将变为只读状态，无法将数据加载到其中。

如果要在集群创建后手动创建默认存储卷，只需添加以下配置项：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 配置共享数据 StarRocks 的 CN 节点

<SharedDataCNconf />

## 使用您的共享数据 StarRocks 集群

<SharedDataUseIntro />

以下示例使用 HMAC 访问密钥和私有密钥为 GCS 存储桶`defaultbucket`创建存储卷`def_volume`，启用存储卷，并将其设置为默认存储卷：

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
