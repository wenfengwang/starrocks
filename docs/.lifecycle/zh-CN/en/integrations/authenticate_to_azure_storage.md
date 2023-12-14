---
displayed_sidebar: "英文"
---

# 身份验证到Microsoft Azure Storage

从v3.0开始，StarRocks可以在以下场景中集成Microsoft Azure Storage（Azure Blob Storage或Azure Data Lake Storage）：

- 从Azure Storage批量加载数据。
- 备份数据并将数据恢复到Azure Storage。
- 查询Azure Storage中的Parquet和ORC文件。
- 查询Azure Storage中的[Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)和[Delta Lake](../data_source/catalog/deltalake_catalog.md)表。

StarRocks支持以下类型的Azure存储帐户：

- Azure Blob Storage
- Azure Data Lake Storage Gen1
- Azure Data Lake Storage Gen2

在本主题中，以Hive目录、文件外部表和Broker Load为例，展示了StarRocks如何使用这些类型的Azure存储帐户与Azure Storage集成。有关示例中参数的信息，请参见[Hive目录](../data_source/catalog/hive_catalog.md)、[文件外部表](../data_source/file_external_table.md)和[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

## Blob Storage

StarRocks支持以下身份验证方法之一来访问Blob Storage：

- 共享密钥（Shared Key）
- SAS令牌（SAS Token）

> **注意**
>
> 当您从Blob Storage加载数据或直接查询文件时，必须使用wasb或wasbs协议来访问您的数据：
>
> - 如果您的存储帐户允许通过HTTP访问，请使用wasb协议，并将文件路径编写为`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>`。
> - 如果您的存储帐户允许通过HTTPS访问，请使用wasbs协议，并将文件路径编写为`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>`。

### 共享密钥

#### 外部目录

在[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)语句中，配置`azure.blob.storage_account`和`azure.blob.shared_key`如下：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://10.1.0.18:9083",
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
);
```

#### 文件外部表

在[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)语句中，配置`azure.blob.storage_account`、`azure.blob.shared_key`和文件路径（`path`）如下：

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

在[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)语句中，配置`azure.blob.storage_account`、`azure.blob.shared_key`和文件路径（`DATA INFILE`）如下：

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

### SAS令牌

#### 外部目录

在[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)语句中，配置`azure.blob.account_name`、`azure.blob.container_name`和`azure.blob.sas_token`如下：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://10.1.0.18:9083",
    "azure.blob.account_name" = "<blob_storage_account_name>",
    "azure.blob.container_name" = "<blob_container_name>",
    "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
);
```

#### 文件外部表

在[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)语句中，配置`azure.blob.account_name`、`azure.blob.container_name`、`azure.blob.sas_token`和文件路径（`path`）如下：

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
    "azure.blob.account_name" = "<blob_storage_account_name>",
    "azure.blob.container_name" = "<blob_container_name>",
    "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
);
```

#### Broker load

在[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)语句中，配置`azure.blob.account_name`、`azure.blob.container_name`、`azure.blob.sas_token`和文件路径（`DATA INFILE`）如下：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>")
    INTO TABLE target_table
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.blob.account_name" = "<blob_storage_account_name>",
    "azure.blob.container_name" = "<blob_container_name>",
    "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
);
```

## Data Lake Storage Gen1

StarRocks支持以下身份验证方法之一来访问Data Lake Storage Gen1：

- 托管服务身份（Managed Service Identity）
- 服务主体（Service Principal）

> **注意**
>
> 当您从Data Lake Storage Gen1加载数据或查询文件时，必须使用adl协议来访问您的数据，并将文件路径编写为`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。

### 托管服务身份

#### 外部目录

在[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)语句中，配置`azure.adls1.use_managed_service_identity`如下：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://10.1.0.18:9083",
    "azure.adls1.use_managed_service_identity" = "true"
);
```

#### 文件外部表

在[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)语句中，配置`azure.adls1.use_managed_service_identity`和文件路径（`path`）如下：

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

在[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)语句中，配置`azure.adls1.use_managed_service_identity`和文件路径（`DATA INFILE`）如下：

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

### 服务主体

#### 外部目录

在[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)语句中，配置`azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`和`azure.adls1.oauth2_endpoint`如下：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://10.1.0.18:9083",
    "azure.adls1.oauth2_client_id" = "<application_client_id>",
```
    "azure.adls1.oauth2_credential" = "<application_client_credential>",
    "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
);
```

#### 外部表格

在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中，按照以下方式配置 `azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential` 、`azure.adls1.oauth2_endpoint` 和文件路径 (`path`)：

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
    "azure.adls1.oauth2_client_id" = "<application_client_id>",
    "azure.adls1.oauth2_credential" = "<application_client_credential>",
    "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
);
```

#### Broker 载入

在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中，按照以下方式配置 `azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、`azure.adls1.oauth2_endpoint` 和文件路径 (`DATA INFILE`)：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>")
    INTO TABLE target_table
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.adls1.oauth2_client_id" = "<application_client_id>",
    "azure.adls1.oauth2_credential" = "<application_client_credential>",
    "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
);
```

## 数据湖存储 Gen2

StarRocks 支持以下一种或多种认证方式来访问数据湖存储 Gen2：

- 托管标识
- 共享密钥
- 服务主体

> **注意**
>
> 当您从数据湖存储 Gen2 加载数据或查询文件时，您必须使用 abfs 或 abfss 协议来访问您的数据：
>
> - 如果您的存储帐户允许通过 HTTP 访问，请使用 abfs 协议，并将文件路径写为 `abfs://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>`。
> - 如果您的存储帐户允许通过 HTTPS 访问，请使用 abfss 协议，并将文件路径写为 `abfss://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>`。

### 托管标识

在开始前，您需要进行以下准备工作：

- 编辑您部署 StarRocks 集群的虚拟机 (VM)。
- 将托管标识添加到这些 VM。
- 确保托管标识与被授权读取存储帐户中数据的角色 (**Storage Blob Data Reader**) 相关联。

#### 外部目录

在 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句中，按照以下方式配置 `azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id` 和 `azure.adls2.oauth2_client_id`：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://10.1.0.18:9083",
    "azure.adls2.oauth2_use_managed_identity" = "true",
    "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
    "azure.adls2.oauth2_client_id" = "<service_client_id>"
);
```

#### 文件外部表格

在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中，按照以下方式配置 `azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id` 和文件路径 (`path`)：

```SQL
CREATE EXTERNAL TABLE external_table_azure
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "abfs[s]://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>",
    "format" = "ORC",
    "azure.adls2.oauth2_use_managed_identity" = "true",
    "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
    "azure.adls2.oauth2_client_id" = "<service_client_id>"
);
```

#### Broker Load

在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中，按照以下方式配置 `azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id` 和文件路径 (`DATA INFILE`)：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("adfs[s]://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>")
    INTO TABLE target_table
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.adls2.oauth2_use_managed_identity" = "true",
    "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
    "azure.adls2.oauth2_client_id" = "<service_client_id>"
);
```

### 共享密钥

#### 外部目录

在 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句中，按照以下方式配置 `azure.adls2.storage_account` 和 `azure.adls2.shared_key`：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://10.1.0.18:9083",
    "azure.adls2.storage_account" = "<storage_account_name>",
    "azure.adls2.shared_key" = "<shared_key>"
);
```

#### 文件外部表格

在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中，按照以下方式配置 `azure.adls2.storage_account`、`azure.adls2.shared_key` 和文件路径 (`path`)：

```SQL
CREATE EXTERNAL TABLE external_table_azure
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "abfs[s]://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>",
    "format" = "ORC",
    "azure.adls2.storage_account" = "<storage_account_name>",
    "azure.adls2.shared_key" = "<shared_key>"
);
```

#### Broker 载入

在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中，按照以下方式配置 `azure.adls2.storage_account`、`azure.adls2.shared_key` 和文件路径 (`DATA INFILE`)：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("adfs[s]://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>")
    INTO TABLE target_table
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.adls2.storage_account" = "<storage_account_name>",
    "azure.adls2.shared_key" = "<shared_key>"
);
```

### 服务主体

在开始前，您需要创建一个服务主体，创建角色分配以将角色分配给服务主体，然后将角色分配添加到您的存储帐户。通过这样的方式，您可以确保该服务主体能够成功访问存储帐户中的数据。

#### 外部目录

在 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句中，按照以下方式配置 `azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret` 和 `azure.adls2.oauth2_client_endpoint`：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://10.1.0.18:9083",
    "azure.adls2.oauth2_client_id" = "<service_client_id>",
    "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
    "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
);
```

#### 文件外部表格

在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中，按照以下方式配置 `azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint` 和文件路径 (`path`)：

```SQL
CREATE EXTERNAL TABLE external_table_azure
(
```sql
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "abfs[s]://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>",
    "format" = "ORC",
    "azure.adls2.oauth2_client_id" = "<service_client_id>",
    "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
    "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
);
```

#### Broker Load

在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 声明中，如下配置`azure.adls2.oauth2_client_id`，`azure.adls2.oauth2_client_secret`，`azure.adls2.oauth2_client_endpoint`，以及文件路径 (`DATA INFILE`)：

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