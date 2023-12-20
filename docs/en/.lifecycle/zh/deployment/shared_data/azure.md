---
displayed_sidebar: English
---

# 使用 Azure Blob 存储共享数据

import SharedDataIntro from '../../assets/commonMarkdown/sharedDataIntro.md'
import SharedDataCNconf from '../../assets/commonMarkdown/sharedDataCNconf.md'
import SharedDataUseIntro from '../../assets/commonMarkdown/SharedDataUseIntro.md'
import SharedDataUse from '../../assets/commonMarkdown/sharedDataUse.md'

<SharedDataIntro />


## 架构

![共享数据架构](../../assets/share_data_arch.png)

## 部署共享数据 StarRocks 集群

部署共享数据 StarRocks 集群的过程与部署无共享 StarRocks 集群类似。唯一的区别是您需要在共享数据集群中部署 CN 而不是 BE。本节仅列出在部署共享数据 StarRocks 集群时，您需要在 FE 和 CN 的配置文件 **fe.conf** 和 **cn.conf** 中添加的额外 FE 和 CN 配置项。有关部署 StarRocks 集群的详细说明，请参阅[部署 StarRocks](../../deployment/deploy_manually.md)。

> **注意**
> 在本文档的下一部分配置集群为共享存储之前，请勿启动集群。

## 为共享数据 StarRocks 配置 FE 节点

在启动集群之前，请先配置 FE 和 CN。下面提供了一个配置示例，随后会详细介绍每个参数。

### Azure Blob 存储的 FE 配置示例

您可以将以下共享数据配置示例添加到每个 FE 节点的 `fe.conf` 文件中。

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

- 如果您使用共享访问签名 (SAS) 来访问 Azure Blob 存储，请添加以下配置项：

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

> **警告**
> 创建 Azure Blob 存储账户时，必须禁用分层命名空间。

### 与 Azure Blob 存储共享存储相关的所有 FE 参数

#### run_mode

StarRocks 集群的运行模式。有效值：

- `shared_data`
- `shared_nothing`（默认值）。

> **注意**
> 您不能同时为 StarRocks 集群采用 `shared_data` 和 `shared_nothing` 模式。不支持混合部署。
> 集群部署后请勿更改 `run_mode`，否则集群将无法重启。不支持将无共享集群转换为共享数据集群，反之亦然。

#### cloud_native_meta_port

云原生元服务 RPC 端口。

- 默认值：`6090`

#### enable_load_volume_from_conf

是否允许 StarRocks 使用 FE 配置文件中指定的对象存储相关属性来创建默认存储卷。有效值：

- `true`（默认值）如果在创建新的共享数据集群时将此项指定为 `true`，StarRocks 将使用 FE 配置文件中的对象存储相关属性来创建内置存储卷 `builtin_storage_volume` 并将其设置为默认存储卷。但是，如果您未指定对象存储相关属性，StarRocks 将无法启动。
- `false` 如果在创建新的共享数据集群时将此项指定为 `false`，StarRocks 将直接启动而不创建内置存储卷。在 StarRocks 中创建任何对象之前，您必须手动创建存储卷并将其设置为默认存储卷。有关详细信息，请参阅[创建默认存储卷](#use-your-shared-data-starrocks-cluster)。

从 v3.1.0 版本开始支持。

> **警告**
> 我们强烈建议您在从 v3.0 版本升级现有共享数据集群时保留此项为 `true`。如果您将此项指定为 `false`，升级前创建的数据库和表将变为只读，您将无法向其中加载数据。

#### cloud_native_storage_type

您使用的对象存储类型。在共享数据模式下，StarRocks 支持将数据存储在 Azure Blob（从 v3.1.1 版本开始支持）和兼容 S3 协议的对象存储（如 AWS S3、Google GCP 和 MinIO）。有效值：

- `S3`（默认值）
- `AZBLOB`。

> **注意**
> 如果您将此参数指定为 `S3`，则必须添加以 `aws_s3` 为前缀的参数。
> 如果您将此参数指定为 `AZBLOB`，则必须添加以 `azure_blob` 为前缀的参数。

#### azure_blob_path

用于存储数据的 Azure Blob 存储路径。它由您的存储账户中的容器名称和该容器下的子路径（如果有）组成，例如 `testcontainer/subpath`。

#### azure_blob_endpoint

您的 Azure Blob 存储账户的终结点，例如 `https://test.blob.core.windows.net`。

#### azure_blob_shared_key

用于授权 Azure Blob 存储请求的共享密钥。

#### azure_blob_sas_token

用于授权 Azure Blob 存储请求的共享访问签名 (SAS)。

> **注意**
> 创建共享数据 StarRocks 集群后，只能修改与凭证相关的配置项。如果您更改了原始存储路径相关的配置项，更改前创建的数据库和表将变为只读，您将无法向其中加载数据。

如果您在集群创建后想要手动创建默认存储卷，只需要添加以下配置项：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 为共享数据 StarRocks 配置 CN 节点

<SharedDataCNconf />


## 使用您的共享数据 StarRocks 集群

<SharedDataUseIntro />


以下示例创建了一个名为 `def_volume` 的存储卷，用于具有共享密钥访问权限的 Azure Blob 存储桶 `defaultbucket`，启用该存储卷，并将其设置为默认存储卷：

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
