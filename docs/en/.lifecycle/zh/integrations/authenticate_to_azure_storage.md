---
displayed_sidebar: English
---

# Microsoft Azure 存储身份验证

从 v3.0 版本开始，StarRocks 可以在以下场景中与 Microsoft Azure 存储（Azure Blob 存储或 Azure Data Lake Storage）集成：

- 从 Azure 存储批量加载数据。
- 从 Azure 存储备份数据并将数据还原到 Azure 存储。
- 查询 Azure 存储中的 Parquet 和 ORC 文件。
- 查询 Azure 存储中的 [Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md) 和 [Delta Lake](../data_source/catalog/deltalake_catalog.md) 表。

StarRocks 支持以下类型的 Azure 存储账号：

- Azure Blob 存储
- Azure Data Lake Storage Gen1
- Azure Data Lake Storage Gen2

本主题以 Hive 目录、文件外部表和 Broker Load 为例，展示了 StarRocks 如何通过这些类型的 Azure 存储账号与 Azure 存储进行集成。有关示例中的参数信息，请参阅 [Hive 目录](../data_source/catalog/hive_catalog.md)、[文件外部表](../data_source/file_external_table.md) 和 [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

## Blob 存储

StarRocks 支持以下认证方式之一来访问 Blob 存储：

- 共享密钥
- SAS 令牌

> **注意**
>
> 当从 Blob 存储加载数据或直接查询文件时，必须使用 wasb 或 wasbs 协议来访问数据：
>
> - 如果您的存储账户允许通过 HTTP 访问，请使用 wasb 协议，并将文件路径写为 `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>`。
> - 如果您的存储账户允许通过 HTTPS 访问，请使用 wasbs 协议，并将文件路径写为 `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>`。

### 共享密钥

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

#### 代理负载

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

### SAS 令牌

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

#### 代理负载

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

StarRocks 支持以下认证方式之一来访问 Data Lake Storage Gen1：

- 托管服务标识
- 服务主体

> **注意**
>
> 当从 Data Lake Storage Gen1 加载数据或查询文件时，必须使用 adl 协议来访问数据，并将文件路径写为 `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。

### 托管服务标识

#### 外部目录

在 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句中配置 `azure.adls1.use_managed_service_identity`，如下所示：

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

在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中配置 `azure.adls1.use_managed_service_identity` 和文件路径 (`path`)，如下所示：

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

#### 代理负载

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

### 服务主体

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
在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中，按照以下方式配置 `azure.adls1.oauth2_client_id` 、 `azure.adls1.oauth2_credential`、 `azure.adls1.oauth2_endpoint`和文件路径 (`path`)：

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

#### 代理负载

在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中，按照以下方式配置 `azure.adls1.oauth2_client_id`、 `azure.adls1.oauth2_credential`、 `azure.adls1.oauth2_endpoint`和文件路径 (`DATA INFILE`)：

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

## Data Lake Storage Gen2

StarRocks 支持使用以下认证方法之一访问 Data Lake Storage Gen2：

- 托管标识
- 共享密钥
- 服务主体

> **注意**
>
> 从 Data Lake Storage Gen2 加载数据或查询文件时，必须使用 abfs 或 abfss 协议来访问数据：
>
> - 如果您的存储帐户允许通过 HTTP 访问，请使用 abfs 协议，并将文件路径写为 `abfs://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>`.
> - 如果您的存储帐户允许通过 HTTPS 访问，请使用 abfss 协议，并将文件路径写为 `abfss://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>`。

### 托管标识

在开始之前，您需要进行以下准备工作：

- 编辑部署了 StarRocks 集群的虚拟机 (VM)。
- 将托管标识添加到这些 VM。
- 确保托管标识与被授权读取存储帐户中数据的角色 (存储 Blob 数据读取者) 相关联。

#### 外部目录

在 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句中，按照以下方式配置 `azure.adls2.oauth2_use_managed_identity`、 `azure.adls2.oauth2_tenant_id`和 `azure.adls2.oauth2_client_id`：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "azure.adls2.oauth2_use_managed_identity" = "true",
    "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
    "azure.adls2.oauth2_client_id" = "<service_client_id>"
);
```

#### 文件外部表

在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中，按照以下方式配置 `azure.adls2.oauth2_use_managed_identity`、 `azure.adls2.oauth2_tenant_id`、 `azure.adls2.oauth2_client_id`和文件路径 (`path`)：

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

#### 代理负载

在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中，按照以下方式配置 `azure.adls2.oauth2_use_managed_identity`、 `azure.adls2.oauth2_tenant_id`、 `azure.adls2.oauth2_client_id`和文件路径 (`DATA INFILE`)：

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

在 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句中，按照以下方式配置 `azure.adls2.storage_account`、 `azure.adls2.shared_key`：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "azure.adls2.storage_account" = "<storage_account_name>",
    "azure.adls2.shared_key" = "<shared_key>"
);
```

#### 文件外部表

在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中，按照以下方式配置 `azure.adls2.storage_account`、 `azure.adls2.shared_key`和文件路径 (`path`)：

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

#### 代理负载

在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中，按照以下方式配置 `azure.adls2.storage_account`、 `azure.adls2.shared_key`和文件路径 (`DATA INFILE`)：

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

在开始之前，您需要创建一个服务主体，创建一个角色分配以将角色分配给服务主体，然后将角色分配添加到您的存储帐户。通过这样做，您可以确保此服务主体可以成功访问存储帐户中的数据。

#### 外部目录

在 [CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) 语句中，按照以下方式配置 `azure.adls2.oauth2_client_id`、 `azure.adls2.oauth2_client_secret`和 `azure.adls2.oauth2_client_endpoint`：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "azure.adls2.oauth2_client_id" = "<service_client_id>",
    "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
    "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
);
```

#### 文件外部表

在 [CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 语句中，按照以下方式配置 `azure.adls2.oauth2_client_id`、 `azure.adls2.oauth2_client_secret`、 `azure.adls2.oauth2_client_endpoint`和文件路径 (`path`)：

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
    "azure.adls2.oauth2_client_id" = "<service_client_id>",
    "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
    "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
);
```

#### 代理负载
在 [LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 语句中，按照以下方式配置 `azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint`和文件路径（`DATA INFILE`）：

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