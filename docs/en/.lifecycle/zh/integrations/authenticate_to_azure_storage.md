---
displayed_sidebar: English
---

# 对 Microsoft Azure 存储进行身份验证

从 v3.0 开始，StarRocks 可以在以下场景中与 Microsoft Azure 存储（Azure Blob 存储或 Azure Data Lake 存储）集成：

- 从 Azure 存储批量加载数据。
- 从 Azure 存储备份数据并将数据恢复到 Azure 存储。
- 在 Azure 存储中查询 Parquet 和 ORC 文件。
- 在 Azure 存储中查询 [Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md) 和 [Delta Lake](../data_source/catalog/deltalake_catalog.md) 表。

StarRocks 支持以下类型的 Azure 存储账户：

- Azure Blob 存储
- Azure Data Lake 存储 Gen1
- Azure Data Lake 存储 Gen2

本主题将使用 Hive 目录、文件外部表和 Broker Load 作为示例，展示 StarRocks 如何使用这些类型的 Azure 存储账户与 Azure 存储集成。有关示例中参数的信息，请参见 [Hive 目录](../data_source/catalog/hive_catalog.md)、[文件外部表](../data_source/file_external_table.md) 和 [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

## Blob 存储

StarRocks 支持使用以下身份验证方法之一来访问 Blob 存储：

- Shared Key
- SAS Token

> **注意**
> 当您从 Blob 存储加载数据或直接查询文件时，必须使用 wasb 或 wasbs 协议来访问您的数据：
- 如果您的存储账户允许通过 HTTP 访问，请使用 wasb 协议并将文件路径写作 `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>`。
- 如果您的存储账户允许通过 HTTPS 访问，请使用 wasbs 协议并将文件路径写作 `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>`。

### Shared Key

#### 外部目录

在 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句中配置 `azure.blob.storage_account` 和 `azure.blob.shared_key`，如下所示：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
);
```

#### 文件外部表

在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中配置 `azure.blob.storage_account`、`azure.blob.shared_key` 和文件路径 (`path`)，如下所示：

```SQL
CREATE EXTERNAL TABLE external_table_azure
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>",
    "format" = "ORC",
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
);
```

#### Broker Load

在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中配置 `azure.blob.storage_account`、`azure.blob.shared_key` 和文件路径 (`DATA INFILE`)，如下所示：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>")
    INTO TABLE test_ingestion_2
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
);
```

### SAS Token

#### 外部目录

在 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句中配置 `azure.blob.storage_account`、`azure.blob.container` 和 `azure.blob.sas_token`，如下所示：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.container" = "<blob_container_name>",
    "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
);
```

#### 文件外部表

在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中配置 `azure.blob.storage_account`、`azure.blob.container`、`azure.blob.sas_token` 和文件路径 (`path`)，如下所示：

```SQL
CREATE EXTERNAL TABLE external_table_azure
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>",
    "format" = "ORC",
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.container" = "<blob_container_name>",
    "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
);
```

#### Broker Load

在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中配置 `azure.blob.storage_account`、`azure.blob.container`、`azure.blob.sas_token` 和文件路径 (`DATA INFILE`)，如下所示：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>")
    INTO TABLE target_table
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.container" = "<blob_container_name>",
    "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
);
```

## Data Lake Storage Gen1

StarRocks 支持使用以下身份验证方法之一来访问 Data Lake Storage Gen1：

- Managed Service Identity
- Service Principal

> **注意**
> 当您加载数据或查询 Data Lake Storage Gen1 中的文件时，必须使用 adl 协议来访问您的数据，并将文件路径写作 `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。

### Managed Service Identity

#### 外部目录

在 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句中按如下方式配置 `azure.adls1.use_managed_service_identity`：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "azure.adls1.use_managed_service_identity" = "true"
);
```

#### 文件外部表

在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中配置 `azure.adls1.use_managed_service_identity` 和文件路径（`path`），如下所示：

```SQL
CREATE EXTERNAL TABLE external_table_azure
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>",
    "format" = "ORC",
    "azure.adls1.use_managed_service_identity" = "true"
);
```

#### Broker Load

在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中配置 `azure.adls1.use_managed_service_identity` 和文件路径 (`DATA INFILE`)，如下所示：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>")
    INTO TABLE target_table
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.adls1.use_managed_service_identity" = "true"
);
```

### Service Principal

#### 外部目录

在 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句中配置 `azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential` 和 `azure.adls1.oauth2_endpoint`，如下所示：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "azure.adls1.oauth2_client_id" = "<application_client_id>",
    "azure.adls1.oauth2_credential" = "<application_client_credential>",
    "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
);
```

#### 文件外部表

```
在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中配置 `azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint` 和文件路径 (`DATA INFILE`)，如下所示：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("adfs[s]://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>")
    INTO TABLE target_table
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.adls2.oauth2_client_id" = "<service_client_id>",
    "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
    "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
);
```
在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中配置 `azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint` 和文件路径 (`DATA INFILE`)，如下所示：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("abfss://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>")
    INTO TABLE target_table
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.adls2.oauth2_client_id" = "<service_client_id>",
    "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
    "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
);
```