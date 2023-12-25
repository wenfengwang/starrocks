---
displayed_sidebar: English
---

# Microsoft Azure Storage への認証

v3.0 以降、StarRocks は以下のシナリオで Microsoft Azure Storage（Azure Blob Storage または Azure Data Lake Storage）と統合できます：

- Azure Storage からバッチでデータをロードします。
- Azure Storage からデータをバックアップし、Azure Storage へデータをリストアします。
- Azure Storage 内の Parquet および ORC ファイルをクエリします。
- Azure Storage 内の [Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)、[Delta Lake](../data_source/catalog/deltalake_catalog.md) テーブルをクエリします。

StarRocks は以下のタイプの Azure Storage アカウントをサポートしています：

- Azure Blob Storage
- Azure Data Lake Storage Gen1
- Azure Data Lake Storage Gen2

このトピックでは、Hive カタログ、ファイル外部テーブル、Broker Load を例に、これらのタイプの Azure Storage アカウントを使用して StarRocks が Azure Storage とどのように統合するかを示します。例のパラメータについては、[Hive カタログ](../data_source/catalog/hive_catalog.md)、[ファイル外部テーブル](../data_source/file_external_table.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

## Blob Storage

StarRocks は Blob Storage にアクセスするために以下の認証方法のいずれかを使用できます：

- Shared Key
- SAS Token

> **注記**
>
> Blob Storage からデータをロードする場合やファイルを直接クエリする場合は、wasb または wasbs プロトコルを使用してデータにアクセスする必要があります：
>
> - ストレージアカウントが HTTP 経由でのアクセスを許可している場合、wasb プロトコルを使用し、ファイルパスを `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>` と書きます。
> - ストレージアカウントが HTTPS 経由でのアクセスを許可している場合、wasbs プロトコルを使用し、ファイルパスを `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>` と書きます。

### Shared Key

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントで `azure.blob.storage_account` と `azure.blob.shared_key` を以下のように設定します：

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

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントで `azure.blob.storage_account`、`azure.blob.shared_key`、およびファイルパス（`path`）を以下のように設定します：

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

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントで `azure.blob.storage_account`、`azure.blob.shared_key`、およびファイルパス（`DATA INFILE`）を以下のように設定します：

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

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントで `azure.blob.storage_account`、`azure.blob.container`、および `azure.blob.sas_token` を以下のように設定します：

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

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントで `azure.blob.storage_account`、`azure.blob.container`、`azure.blob.sas_token`、およびファイルパス（`path`）を以下のように設定します：

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

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントで `azure.blob.storage_account`、`azure.blob.container`、`azure.blob.sas_token`、およびファイルパス（`DATA INFILE`）を以下のように設定します：

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

StarRocks は Data Lake Storage Gen1 へのアクセスに以下の認証方法を使用できます：

- Managed Service Identity（管理されたサービス ID）
- Service Principal（サービス プリンシパル）

> **注記**
>
> Data Lake Storage Gen1 からデータをロードしたりファイルをクエリしたりする場合は、adl プロトコルを使用してデータにアクセスし、ファイルパスを `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>` と書きます。

### Managed Service Identity（管理されたサービス ID）

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントで `azure.adls1.use_managed_service_identity` を以下のように設定します：

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

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントで `azure.adls1.use_managed_service_identity` およびファイルパス（`path`）を以下のように設定します：

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

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントで `azure.adls1.use_managed_service_identity` およびファイルパス（`DATA INFILE`）を以下のように設定します：

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

### Service Principal（サービス プリンシパル）

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントで `azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、および `azure.adls1.oauth2_endpoint` を以下のように設定します：

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


`azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、`azure.adls1.oauth2_endpoint`、およびファイルパス(`path`)を[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで次のように構成します。

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

`azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、`azure.adls1.oauth2_endpoint`、およびファイルパス(`DATA INFILE`)を[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで次のように構成します。

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

StarRocksは、Data Lake Storage Gen2へのアクセスに以下の認証方法のいずれかを使用することをサポートしています：

- Managed Identity
- Shared Key
- Service Principal

> **注記**
>
> Data Lake Storage Gen2からデータをロードするかファイルをクエリする際には、abfsまたはabfssプロトコルを使用してデータにアクセスする必要があります：
>
> - ストレージアカウントがHTTP経由でのアクセスを許可している場合は、abfsプロトコルを使用し、ファイルパスを`abfs://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>`と書きます。
> - ストレージアカウントがHTTPS経由でのアクセスを許可している場合は、abfssプロトコルを使用し、ファイルパスを`abfss://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>`と書きます。

### Managed Identity

開始する前に、以下の準備を行う必要があります：

- StarRocksクラスタがデプロイされている仮想マシン(VM)を編集します。
- これらのVMにManaged Identityを追加します。
- Managed Identityがストレージアカウント内のデータを読み取る権限を持つロール（**Storage Blob Data Reader**）に関連付けられていることを確認します。

#### 外部カタログ

`azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id`を[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで次のように構成します。

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

#### ファイル外部テーブル

`azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id`、およびファイルパス(`path`)を[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで次のように構成します。

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

`azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id`、およびファイルパス(`DATA INFILE`)を[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで次のように構成します。

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("abfs[s]://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>")
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

### Shared Key

#### 外部カタログ

`azure.adls2.storage_account`と`azure.adls2.shared_key`を[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで次のように構成します。

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

#### ファイル外部テーブル

`azure.adls2.storage_account`、`azure.adls2.shared_key`、およびファイルパス(`path`)を[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで次のように構成します。

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

`azure.adls2.storage_account`、`azure.adls2.shared_key`、およびファイルパス(`DATA INFILE`)を[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで次のように構成します。

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("abfs[s]://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>")
    INTO TABLE target_table
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.adls2.storage_account" = "<storage_account_name>",
    "azure.adls2.shared_key" = "<shared_key>"
);
```

### Service Principal

開始する前に、サービスプリンシパルを作成し、サービスプリンシパルにロールを割り当てるためのロール割り当てを作成し、そのロール割り当てをストレージアカウントに追加する必要があります。これにより、サービスプリンシパルがストレージアカウント内のデータに正常にアクセスできることを確認できます。

#### 外部カタログ

`azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint`を[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで次のように構成します。

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

#### ファイル外部テーブル

`azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint`、およびファイルパス(`path`)を[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで次のように構成します。

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


[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントで、`azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint`、およびファイルパス (`DATA INFILE`) を以下のように設定します：

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

