---
displayed_sidebar: "Chinese"
---

# 认证到Google Cloud Storage

## 认证方法

从v3.0开始，StarRocks支持使用以下一种认证方法之一来访问Google Cloud Storage（GCS）：

- 基于VM的认证

  使用附加到Google Cloud Compute Engine的凭据来认证GCS。

- 基于服务帐号的认证

  使用服务帐号来认证GCS。

- 基于模拟的认证

  使服务帐号或虚拟机（VM）实例模拟另一个服务帐号。

## 场景

StarRocks可以在以下场景中认证到GCS：

- 从GCS批量加载数据。
- 从GCS备份并恢复数据。
- 在GCS中查询Parquet和ORC文件。
- 在GCS中查询[Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)和[Delta Lake](../data_source/catalog/deltalake_catalog.md)表。

在此主题中，[Hive目录](../data_source/catalog/hive_catalog.md)、[文件外部表](../data_source/file_external_table.md)和[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)分别用作示例，以展示StarRocks在不同场景下如何与GCS集成。有关示例中的`StorageCredentialParams`的信息，请参见本主题的"[参数](../integrations/authenticate_to_gcs.md#parameters)"部分。

> **注意**
>
> StarRocks仅支持根据gs协议从GCS加载数据或直接查询文件。因此，当您从GCS加载数据或查询文件时，必须在文件路径中包含`gs`作为前缀。

### 外部目录

使用[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)语句创建名为`hive_catalog_gcs`的Hive目录，如下所示，以便从GCS查询文件：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_gcs
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://34.132.15.127:9083",
    StorageCredentialParams
);
```

### 文件外部表

使用[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)语句创建名为`external_table_gcs`的文件外部表，如下所示，以便从GCS查询名为`test_file_external_tbl`的数据文件，而无需任何元数据存储：

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

### Broker加载

使用[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)语句创建标签为`test_db.label000`的Broker加载作业，以将数据从GCS批量加载到StarRocks表`target_table`：

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

`StorageCredentialParams`表示一组参数，描述了如何使用不同的认证方法认证到GCS。

### 基于VM的认证

如果您的StarRocks集群部署在托管在Google Cloud Platform（GCP）上的VM实例上，并且要使用该VM实例对GCS进行认证，请配置`StorageCredentialParams`如下：

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                                  | **默认值** | **示例值** | **描述**                                |
| ---------------------------------------- | ---------- | ---------- | ------------------------------------- |
| gcp.gcs.use_compute_engine_service_account | false      | true       | 指定是否直接使用绑定到计算引擎的服务帐号。 |

### 基于服务帐号的认证

如果您直接使用服务帐号来认证GCS，请配置`StorageCredentialParams`如下：

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                          | **默认值** | **示例值**                                       | **描述**                                |
| -------------------------------- | ---------- | ----------------------------------------------- | ------------------------------------- |
| gcp.gcs.service_account_email     | ""         | "`user@hello.iam.gserviceaccount.com`"          | 在创建服务帐号时生成的JSON文件中的电子邮件地址。 |
| gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"     | 在创建服务帐号时生成的JSON文件中的私钥ID。 |
| gcp.gcs.service_account_private_key | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 在创建服务帐号时生成的JSON文件中的私钥。  |

### 基于模拟的认证

#### 使VM实例模拟服务帐号

如果您的StarRocks集群部署在GCP上的VM实例上，并且希望使该VM实例模拟服务帐号，从而使StarRocks继承服务帐号的权限以访问GCS，请配置`StorageCredentialParams`如下：

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true",
"gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                                  | **默认值** | **示例值** | **描述**                                |
| ---------------------------------------- | ---------- | ---------- | ------------------------------------- |
| gcp.gcs.use_compute_engine_service_account | false      | true       | 指定是否直接使用绑定到计算引擎的服务帐号。 |
| gcp.gcs.impersonation_service_account     | ""         | "hello"    | 您要模拟的服务帐号。                        |

#### 使服务帐号模拟另一个服务帐号

如果要使一个服务帐号（临时命名为元服务帐号）模拟另一个服务帐号（临时命名为数据服务帐号）并使StarRocks继承数据服务帐号的权限以访问GCS，请配置`StorageCredentialParams`如下：

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
"gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
"gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                          | **默认值** | **示例值**                                       | **描述**                                |
| -------------------------------- | ---------- | ----------------------------------------------- | ------------------------------------- |
| gcp.gcs.service_account_email     | ""         | "`user@hello.iam.gserviceaccount.com`"          | 在创建元服务帐号时生成的JSON文件中的电子邮件地址。 |
| gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"     | 在创建元服务帐号时生成的JSON文件中的私钥ID。 |
| gcp.gcs.service_account_private_key | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 在创建元服务帐号时生成的JSON文件中的私钥。  |
| gcp.gcs.impersonation_service_account | ""         | "hello"                                         | 您要模拟的数据服务帐号。                     |