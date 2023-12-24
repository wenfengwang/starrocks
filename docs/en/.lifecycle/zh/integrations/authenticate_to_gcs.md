---
displayed_sidebar: English
---

# 谷歌云存储身份验证

## 身份验证方法

从 v3.0 开始，StarRocks 支持使用以下身份验证方法之一访问谷歌云存储（GCS）：

- 基于 VM 的身份验证

  使用附加到谷歌云计算引擎的凭据来进行 GCS 身份验证。

- 基于服务帐户的身份验证

  使用服务帐户来进行 GCS 身份验证。

- 基于模拟的身份验证

  使服务帐户或虚拟机（VM）实例模拟另一个服务帐户。

## 场景

StarRocks 可以在以下场景中向 GCS 进行身份验证：

- 从 GCS 批量加载数据。
- 从 GCS 备份数据并将数据还原到 GCS。
- 在 GCS 中查询 Parquet 和 ORC 文件。
- 在 GCS 中查询[Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md) 和[Delta Lake](../data_source/catalog/deltalake_catalog.md)表。

在本主题中，[Hive 目录](../data_source/catalog/hive_catalog.md)、[文件外部表](../data_source/file_external_table.md) 和[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)被用作示例，展示了 StarRocks 在不同场景下如何与 GCS 集成。有关示例中的`StorageCredentialParams`信息，请参阅本主题的“[参数](../integrations/authenticate_to_gcs.md#parameters)”部分。

> **注意**
>
> StarRocks 仅支持根据 gs 协议从 GCS 加载数据或直接查询文件。因此，当您从 GCS 加载数据或查询文件时，必须在文件路径中包含`gs`作为前缀。

### 外部目录

使用[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)语句创建名为`hive_catalog_gcs`的 Hive 目录，以便从 GCS 查询文件，示例如下：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_gcs
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    StorageCredentialParams
);
```

### 文件外部表

使用[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)语句创建名为`external_table_gcs`的文件外部表，以便查询名为`test_file_external_tbl`的数据文件，而无需任何元存储，示例如下：

```SQL
CREATE EXTERNAL TABLE external_table_gcs
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "gs:////test-gcs/test_file_external_tbl",
    "format" = "ORC",
    StorageCredentialParams
);
```

### 代理负载

使用[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)语句创建一个标签为`test_db.label000`的 Broker Load 作业，以便将 GCS 中的数据批量加载到 StarRocks 表`target_table`中，示例如下：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("gs://bucket_gcs/test_brokerload_ingestion/*")
    INTO TABLE target_table
    FORMAT AS "parquet"
)
WITH BROKER
(
    StorageCredentialParams
);
```

## 参数

`StorageCredentialParams`表示一个参数集，该参数集描述了如何使用不同的身份验证方法向 GCS 进行身份验证。

### 基于 VM 的身份验证

如果您的 StarRocks 集群部署在谷歌云平台（GCP）托管的 VM 实例上，并且您希望使用该 VM 实例对 GCS 进行认证，请按以下方式配置`StorageCredentialParams`：

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true"
```

下表描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                              | **默认值** | **值示例** | **描述**                                              |
| ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false             | true                  | 指定是否直接使用绑定到 Compute Engine 的服务帐户。 |

### 基于服务帐户的身份验证

如果直接使用服务帐户对 GCS 进行身份验证，请按以下方式配置`StorageCredentialParams`：

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

下表描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                          | **默认值** | **值示例**                                       | **描述**                                              |
| -------------------------------------- | ----------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""                | "`user@hello.iam.gserviceaccount.com`"                        | 创建服务帐户时生成的 JSON 文件中的电子邮件地址。 |
| gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                  | 创建服务帐户时生成的 JSON 文件中的私钥 ID。 |
| gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 创建服务帐户时生成的 JSON 文件中的私钥。 |

### 基于模拟的身份验证

#### 使 VM 实例模拟服务帐户

如果您的 StarRocks 集群部署在 GCP 上托管的 VM 实例上，并且您希望让该 VM 实例模拟服务帐户，从而让 StarRocks 继承服务帐户访问 GCS 的权限，请按以下方式配置`StorageCredentialParams`：

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true",
"gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
```

下表描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                              | **默认值** | **值示例** | **描述**                                              |
| ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false             | true                  | 指定是否直接使用绑定到 Compute Engine 的服务帐户。 |
| gcp.gcs.impersonation_service_account      | ""                | "hello"               | 要模拟的服务帐户。            |

#### 让一个服务帐户冒充另一个服务帐户

如果您想让一个服务帐户（暂时命名为元服务帐户）冒充另一个服务帐户（暂时命名为数据服务帐户），并让 StarRocks 继承数据服务帐户访问 GCS 的权限，请按以下方式配置`StorageCredentialParams`：

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
"gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
"gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
```

下表描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                          | **默认值** | **值示例**                                       | **描述**                                              |
| -------------------------------------- | ----------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""                | "`user@hello.iam.gserviceaccount.com`"                        | 创建元服务帐户时生成的 JSON 文件中的电子邮件地址。 |
| gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                  | 创建元服务帐户时生成的 JSON 文件中的私钥 ID。 |
| gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 创建元服务帐户时生成的 JSON 文件中的私钥。 |
| gcp.gcs.impersonation_service_account  | ""                | "hello"                                                     | 要模拟的数据服务帐户。       |
