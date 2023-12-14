---
displayed_sidebar: "Chinese"
---

# 使用 Azure Blob 存储共享数据

导入 SharedDataIntro 自 ../../assets/commonMarkdown/sharedDataIntro.md
SharedDataCNconf 是从 ../../assets/commonMarkdown/sharedDataCNconf.md
SharedDataUseIntro 导入自 ../../assets/commonMarkdown/sharedDataUseIntro.md
从 ../../assets/commonMarkdown/sharedDataUse.md 导入 SharedDataUseIntro

<SharedDataIntro />

## 架构

![Shared-data Architecture](../../assets/share_data_arch.png)

## 部署共享数据 StarRocks 集群

部署共享数据 StarRocks 集群与共享无存储 StarRocks 集群类似。唯一的区别是在共享数据集群中，您需要部署 CN（存储节点） 而不是 BE（执行节点）。本章节仅列出在部署共享数据 StarRocks 集群时，您需要在 FE 和 CN 的配置文件 **fe.conf** 和 **cn.conf** 中添加的附加 FE 和 CN 配置项。有关部署 StarRocks 集群的详细说明，请参见 [手动部署 StarRocks](../../deployment/deploy_manually.md)。

> **注意**
>
> 直到配置本文件中的共享存储后，才启动集群。

## 配置共享数据 StarRocks 的 FE 节点

在启动集群之前，请配置 FE 和 CN。以下是示例配置，然后提供每个参数的详细信息。

### Azure Blob 存储的 FE 示例配置

您可以将示例共享数据添加到您的 `fe.conf` 中，在每个 FE 节点上都可以添加到 `fe.conf` 文件中。


  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = AZBLOB

  # 例如，testcontainer/subpath
  azure_blob_path = <blob_path>

  # 例如，https://test.blob.core.windows.net
  azure_blob_endpoint = <endpoint_url>

  azure_blob_shared_key = <shared_key>
  ```

- 如果您使用共享访问签名（SAS）访问 Azure Blob 存储，则添加以下配置项：

  ```Properties
  run_mode = shared_data
  cloud_native_meta_port = <meta_port>
  cloud_native_storage_type = AZBLOB

  # 例如，testcontainer/subpath
  azure_blob_path = <blob_path>

  # 例如，https://test.blob.core.windows.net
  azure_blob_endpoint = <endpoint_url>

  azure_blob_sas_token = <sas_token>
  ```

> **注意**
>
> 在创建 Azure Blob 存储账户时，必须禁用分层命名空间。

### 与 Azure Blob 存储相关的所有 FE 参数


#### run_mode

　StarRocks 集群的运行模式。有效值：

- `shared_data`
- `shared_nothing`（默认）。

> **注意**
>
> 对于 StarRocks 集群，您不能同时采用 `shared_data` 和 `shared_nothing` 模式。不支持混合部署。
>
> 不要在集群部署之后更改 `run_mode`。否则，集群将无法重新启动。共享无存储集群和共享数据集群之间的相互转换均不受支持。

#### cloud_native_meta_port

云原生 meta 服务 RPC 端口。

- 默认值：`6090`

#### enable_load_volume_from_conf

是否允许 StarRocks 使用 FE 配置文件中指定的与对象存储相关的属性创建默认存储卷。有效值：

- `true`（默认）如果在创建新的共享数据集群时将此项指定为 `true`，StarRocks 将使用 FE 配置文件中指定的与对象存储相关的属性创建内置存储卷 `builtin_storage_volume`，并将其设置为默认存储卷。但是，如果您尚未指定与对象存储相关的属性，StarRocks 无法启动。
- `false`如果在创建新的共享数据集群时将此项指定为 `false`，StarRocks 将直接启动而不创建内置存储卷。在创建 StarRocks 中的任何对象之前，您必须手动创建存储卷并将其设置为默认存储卷。有关更多信息，请参见 [创建默认存储卷](#创建默认存储卷)。

从 v3.1.0 版本开始支持。

> **注意**
>
> 我们强烈建议您在从 v3.0 升级现有共享数据集群时，将此项设置为 `true`。如果将此项设置为 `false`，则在升级前创建的数据库和表将变为只读，您无法向其加载数据。

#### cloud_native_storage_type

您使用的对象存储类型。在共享数据模式下，StarRocks 支持在 Azure Blob（从 v3.1.1 版本开始支持）中存储数据，以及与 S3 协议兼容的对象存储（例如 AWS S3、Google GCP 和 MinIO）。有效值：

- `S3`（默认）
- `AZBLOB`。

> **注意**
>
> 如果将此参数指定为 `S3`，则必须添加以 `aws_s3` 为前缀的参数。
>
> 如果将此参数指定为 `AZBLOB`，则必须添加以 `azure_blob` 为前缀的参数。

#### azure_blob_path

用于存储数据的 Azure Blob 存储路径。它由您存储帐户中的容器名称和容器下的子路径（如果有）组成，例如 `testcontainer/subpath`。

#### azure_blob_endpoint

您的 Azure Blob 存储账户的终结点，例如 `https://test.blob.core.windows.net`。

#### azure_blob_shared_key

用于授权对您的 Azure Blob 存储发出请求的共享密钥。

#### azure_blob_sas_token

用于授权对您的 Azure Blob 存储发出请求的共享访问签名（SAS）。

> **注意**
>
> 仅可以在创建共享数据 StarRocks 集群后修改与凭据相关的配置项。如果更改了原始存储路径相关的配置项，那么在更改前创建的数据库和表将变为只读状态，您无法向其加载数据。

如果您想在创建集群后手动创建默认存储卷，只需添加以下配置项目：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 配置共享数据 StarRocks 的 CN 节点

<SharedDataCNconf />

## 使用共享数据 StarRocks 集群

<SharedDataUseIntro />

下面的示例为 Azure Blob 存储桶 `defaultbucket` 创建了一个存储卷 `def_volume`，并启用了共享密钥访问，然后将其作为默认存储卷：

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = AZBLOB
LOCATIONS = ("azblob://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.shared_key" = "<shared_key>"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />