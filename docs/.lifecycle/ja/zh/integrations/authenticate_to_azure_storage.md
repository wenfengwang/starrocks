---
displayed_sidebar: Chinese
---

# Microsoft Azure Storage 認証情報の設定

StarRocks はバージョン 3.0 から、以下のシナリオで Microsoft Azure Storage（Azure Blob Storage または Azure Data Lake Storage）との統合をサポートしています:

- Azure Storage からのデータのバルクインポート。
- Azure Storage へのデータのバックアップ、または Azure Storage からのデータのリストア。
- Azure Storage にある Parquet または ORC 形式のデータファイルのクエリ。
- Azure Storage にある [Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)、または [Delta Lake](../data_source/catalog/deltalake_catalog.md) テーブルのクエリ。

StarRocks は以下のタイプの Azure ストレージアカウントを介して Azure Storage にアクセスすることをサポートしています:

- Azure Blob Storage
- Azure Data Lake Storage Gen1
- Azure Data Lake Storage Gen2

このドキュメントでは、Hive catalog、ファイル外部テーブル、Broker Load を例に、StarRocks が異なるタイプのストレージアカウントを通じて Azure Storage にアクセスする方法を説明します。以下の例に出てくるパラメータの詳細については、[Hive catalog](../data_source/catalog/hive_catalog.md)、[ファイル外部テーブル](../data_source/file_external_table.md)、および [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

## Blob Storage

StarRocks は以下の認証方法を通じて Blob Storage にアクセスすることをサポートしています:

- Shared Key
- SAS Token

> **説明**
>
> Blob Storage からデータをインポートする場合や Blob Storage 内のデータファイルを直接クエリする場合は、ファイルプロトコルとして wasb または wasbs を使用してターゲットデータにアクセスする必要があります:
>
> - ストレージアカウントが HTTP プロトコルを介してアクセスをサポートしている場合は、wasb ファイルプロトコルを使用し、ファイルパスの形式は `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/` です。
> - ストレージアカウントが HTTPS プロトコルを介してアクセスをサポートしている場合は、wasbs ファイルプロトコルを使用し、ファイルパスの形式は `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/` です。

### Shared Key 認証に基づく認証

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントで、`azure.blob.storage_account` と `azure.blob.shared_key` を以下のように設定します:

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

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントで、`azure.blob.storage_account`、`azure.blob.shared_key`、およびファイルパス (`path`) を以下のように設定します:

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

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントで、`azure.blob.storage_account`、`azure.blob.shared_key`、およびファイルパス (`DATA INFILE`) を以下のように設定します:

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

### SAS Token 認証に基づく認証

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントで、`azure.blob.storage_account`、`azure.blob.container`、および `azure.blob.sas_token` を以下のように設定します:

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

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントで、`azure.blob.storage_account`、`azure.blob.container`、`azure.blob.sas_token`、およびファイルパス (`path`) を以下のように設定します:

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

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントで、`azure.blob.storage_account`、`azure.blob.container`、`azure.blob.sas_token`、およびファイルパス (`DATA INFILE`) を以下のように設定します:

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

StarRocks は以下の認証方法を通じて Data Lake Storage Gen1 にアクセスすることをサポートしています:

- Managed Service Identity
- Service Principal

> **説明**
>
> Azure Data Lake Storage Gen1 からデータをインポートする場合や Azure Data Lake Storage Gen1 内のデータファイルを直接クエリする場合は、ファイルプロトコルとして adl を使用してターゲットデータにアクセスする必要があります。ファイルパスの形式は `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>` です。

### Managed Service Identity 認証に基づく認証

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントで、`azure.adls1.use_managed_service_identity` を以下のように設定します:

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "azure.adls1.use_managed_service_identity" = "true"
);
```

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントで、`azure.adls1.use_managed_service_identity` およびファイルパス (`path`) を以下のように設定します:

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

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントで、`azure.adls1.use_managed_service_identity` およびファイルパス (`DATA INFILE`) を以下のように設定します:

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

### Service Principal 認証に基づく認証

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントで、`azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、および `azure.adls1.oauth2_endpoint` を以下のように設定します:

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

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントでは、以下のように `azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、`azure.adls1.oauth2_endpoint` およびファイルパス (`path`) を設定します：

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

#### Broker Load

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントでは、以下のように `azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、`azure.adls1.oauth2_endpoint` およびファイルパス (`DATA INFILE`) を設定します：

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

StarRocks は以下の認証方法を使用して Data Lake Storage Gen2 にアクセスすることをサポートしています：

- Managed Identity
- Shared Key
- Service Principal

> **説明**
>
> Data Lake Storage Gen2 からデータをインポートする場合や Azure Data Lake Storage Gen2 内のデータファイルを直接クエリする場合、ファイルプロトコルとして abfs または abfss を使用してターゲットデータにアクセスする必要があります：
>
> - ストレージアカウントが HTTP プロトコルを介してアクセスをサポートしている場合は、abfs ファイルプロトコルを使用し、ファイルパスの形式は `abfs://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>` です。
> - ストレージアカウントが HTTPS プロトコルを介してアクセスをサポートしている場合は、abfss ファイルプロトコルを使用し、ファイルパスの形式は `abfss://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>` です。

### Managed Identity 認証に基づく認証

Managed Identity 認証を選択する場合、以下の準備作業を事前に完了しておく必要があります：

- 認証要件に基づいて、StarRocks がデプロイされている VM を編集します。
- これらの VM に Managed Identity を追加します。
- 追加された Managed Identity が **Storage Blob Data Reader** ロール（このロールにはストレージアカウント内のデータを読み取る権限があります）にバインドされていることを確認します。

#### External Catalog

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントでは、以下のように `azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id` を設定します：

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

#### 外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントでは、以下のように `azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id` およびファイルパス (`path`) を設定します：

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

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントでは、以下のように `azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id` およびファイルパス (`DATA INFILE`) を設定します：

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

### Shared Key 認証に基づく認証

#### External Catalog

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントでは、以下のように `azure.adls2.storage_account` と `azure.adls2.shared_key` を設定します：

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

#### 外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントでは、以下のように `azure.adls2.storage_account`、`azure.adls2.shared_key` およびファイルパス (`path`) を設定します：

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

#### Broker Load

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントでは、以下のように `azure.adls2.storage_account`、`azure.adls2.shared_key` およびファイルパス (`DATA INFILE`) を設定します：

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

### Service Principal 認証に基づく認証

Service Principal 認証を選択する場合、事前に Service Principal を作成し、Role Assignment を作成してストレージアカウントに追加する必要があります。これにより、作成した Service Principal を介してストレージアカウント内のデータに正常にアクセスできることが保証されます。

#### External Catalog

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントでは、以下のように `azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint` を設定します：

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

#### 外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントでは、以下のように `azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint` およびファイルパス (`path`) を設定します：

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

#### Broker Load


[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 文において、以下のように `azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint` およびファイルパス (`DATA INFILE`) を設定します：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("adls[s]://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>")
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
