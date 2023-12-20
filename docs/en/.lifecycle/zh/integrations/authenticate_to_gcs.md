---
displayed_sidebar: English
---

# 对 Google Cloud Storage 进行身份验证

## 身份验证方法

从 v3.0 版本开始，StarRocks 支持使用以下几种身份验证方法之一来访问 Google Cloud Storage (GCS)：

- 基于虚拟机的身份验证

  使用附加在 Google Cloud Compute Engine 上的凭证来认证 GCS。

- 基于服务账户的身份验证

  使用服务账户来认证 GCS。

- 基于模拟的身份验证

  使服务账户或虚拟机 (VM) 实例模拟另一个服务账户。

## 应用场景

StarRocks 可以在以下场景中对 GCS 进行身份验证：

- 从 GCS 批量加载数据。
- 从 GCS 备份数据并将数据恢复到 GCS。
- 在 GCS 中查询 Parquet 和 ORC 文件。
- 查询 GCS 中的 [Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md) 和 [Delta Lake](../data_source/catalog/deltalake_catalog.md) 表。

在本主题中，[Hive 目录](../data_source/catalog/hive_catalog.md)、[文件外部表](../data_source/file_external_table.md) 和 [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 被用作示例，展示 StarRocks 在不同场景下如何与 GCS 集成。有关示例中 `StorageCredentialParams` 的信息，请参见本主题的“[参数](../integrations/authenticate_to_gcs.md#parameters)”部分。

> **注意**
> StarRocks 仅支持根据 gs 协议从 GCS 加载数据或直接查询文件。因此，当您从 GCS 加载数据或查询文件时，必须在文件路径中包含 `gs` 作为前缀。

### 外部目录

使用 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句创建一个名为 `hive_catalog_gcs` 的 Hive 目录，如下所示，以便从 GCS 查询文件：

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

使用 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句创建一个名为 `external_table_gcs` 的文件外部表，如下所示，以便在没有任何元数据存储的情况下从 GCS 查询名为 `test_file_external_tbl` 的数据文件：

```SQL
CREATE EXTERNAL TABLE external_table_gcs
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "gs://test-gcs/test_file_external_tbl",
    "format" = "ORC",
    StorageCredentialParams
);
```

### Broker Load

使用 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句创建一个标签为 `test_db.label000` 的 Broker Load 任务，以便将 GCS 中的数据批量加载到 StarRocks 表 `target_table`：

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

`StorageCredentialParams` 代表了一组参数，描述了如何使用不同的身份验证方法对 GCS 进行认证。

### 基于虚拟机的身份验证

如果您的 StarRocks 集群部署在 Google Cloud Platform (GCP) 上的 VM 实例上，并且您希望使用该 VM 实例来认证 GCS，请如下配置 `StorageCredentialParams`：

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true"
```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

|**参数**|**默认值**|**值示例**|**说明**|
|---|---|---|---|
|gcp.gcs.use_compute_engine_service_account|false|true|指定是否直接使用绑定到 Compute Engine 的服务账户。|

### 基于服务账户的身份验证

如果您直接使用服务账户来认证 GCS，请如下配置 `StorageCredentialParams`：

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

|**参数**|**默认值**|**值示例**|**说明**|
|---|---|---|---|
|gcp.gcs.service_account_email|""|"`user@hello.iam.gserviceaccount.com`"|创建服务账户时生成的 JSON 文件中的电子邮件地址。|
|gcp.gcs.service_account_private_key_id|""|"61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"|创建服务账户时生成的 JSON 文件中的私钥 ID。|
|gcp.gcs.service_account_private_key|""|"-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"|创建服务账户时生成的 JSON 文件中的私钥。|

### 基于模拟的身份验证

#### 让 VM 实例模拟服务账户

如果您的 StarRocks 集群部署在 GCP 托管的 VM 实例上，并且您想让该 VM 实例模拟服务账户，以便 StarRocks 继承服务账户的权限来访问 GCS，请如下配置 `StorageCredentialParams`：

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true",
"gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

|**参数**|**默认值**|**值示例**|**说明**|
|---|---|---|---|
|gcp.gcs.use_compute_engine_service_account|false|true|指定是否直接使用绑定到 Compute Engine 的服务账户。|
|gcp.gcs.impersonation_service_account|""|"hello"|您想要模拟的服务账户。|

#### 让一个服务账户模拟另一个服务账户

如果您想让一个服务账户（暂称为元服务账户）模拟另一个服务账户（暂称为数据服务账户），并让 StarRocks 继承数据服务账户的权限来访问 GCS，请如下配置 `StorageCredentialParams`：

```Plain
"gcp.gcs.service_account_email" = "<meta_google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<meta_google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<meta_google_service_private_key>",
"gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

|**参数**|**默认值**|**值示例**|**说明**|
|---|---|---|---|
|gcp.gcs.service_account_email|""|"user@hello.iam.gserviceaccount.com"|创建元服务账户时生成的 JSON 文件中的电子邮件地址。|
|gcp.gcs.service_account_private_key_id|""|"61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"|创建元服务账户时生成的 JSON 文件中的私钥 ID。|
|gcp.gcs.service_account_private_key|""|"-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"|创建元服务账户时生成的 JSON 文件中的私钥。|
|gcp.gcs.impersonation_service_account|""|"hello"|您想要模拟的数据服务账户。|