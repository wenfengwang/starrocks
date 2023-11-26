---
displayed_sidebar: "Japanese"
---

# Microsoft Azure Storageへの認証

v3.0以降、StarRocksは以下のシナリオでMicrosoft Azure Storage（Azure Blob StorageまたはAzure Data Lake Storage）と統合することができます。

- Azure Storageからのデータのバッチロード
- Azure Storageへのデータのバックアップとリストア
- Azure StorageのParquetおよびORCファイルのクエリ
- Azure Storageの[Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)、および[Delta Lake](../data_source/catalog/deltalake_catalog.md)テーブルのクエリ

StarRocksは以下のタイプのAzure Storageアカウントをサポートしています。

- Azure Blob Storage
- Azure Data Lake Storage Gen1
- Azure Data Lake Storage Gen2

このトピックでは、Hiveカタログ、ファイル外部テーブル、およびブローカーロードを使用して、これらのタイプのAzure Storageアカウントを使用してStarRocksがAzure Storageとどのように統合されるかを示す例を示します。例のパラメータについての情報は、[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[ファイル外部テーブル](../data_source/file_external_table.md)、および[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

## Blob Storage

StarRocksは、Blob Storageへのアクセスに次の認証方法のいずれかを使用することができます。

- 共有キー
- SASトークン

> **注意**
>
> Blob Storageからデータをロードするか、ファイルを直接クエリする場合は、データにアクセスするためにwasbまたはwasbsプロトコルを使用する必要があります：
>
> - ストレージアカウントがHTTP経由でアクセスを許可している場合は、wasbプロトコルを使用し、ファイルパスを`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`として書きます。
> - ストレージアカウントがHTTPS経由でアクセスを許可している場合は、wasbsプロトコルを使用し、ファイルパスを`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`として書きます。

### 共有キー

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで、`azure.blob.storage_account`および`azure.blob.shared_key`を次のように設定します：

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

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで、`azure.blob.storage_account`、`azure.blob.shared_key`、およびファイルパス（`path`）を次のように設定します：

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

#### ブローカーロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで、`azure.blob.storage_account`、`azure.blob.shared_key`、およびファイルパス（`DATA INFILE`）を次のように設定します：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*")
    INTO TABLE test_ingestion_2
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
);
```

### SASトークン

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで、`azure.blob.account_name`、`azure.blob.container_name`、および`azure.blob.sas_token`を次のように設定します：

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

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで、`azure.blob.account_name`、`azure.blob.container_name`、`azure.blob.sas_token`、およびファイルパス（`path`）を次のように設定します：

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

#### ブローカーロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで、`azure.blob.account_name`、`azure.blob.container_name`、`azure.blob.sas_token`、およびファイルパス（`DATA INFILE`）を次のように設定します：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*")
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

StarRocksは、Data Lake Storage Gen1へのアクセスに次の認証方法のいずれかを使用することができます。

- 管理されたサービスアイデンティティ
- サービスプリンシパル

> **注意**
>
> Data Lake Storage Gen1からデータをロードするか、ファイルをクエリする場合は、データにアクセスするためにadlプロトコルを使用する必要があります。ファイルパスは`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`として書きます。

### 管理されたサービスアイデンティティ

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで、`azure.adls1.use_managed_service_identity`を次のように設定します：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://10.1.0.18:9083",
    "azure.adls1.use_managed_service_identity" = "true"
);
```

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで、`azure.adls1.use_managed_service_identity`およびファイルパス（`path`）を次のように設定します：

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

#### ブローカーロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで、`azure.adls1.use_managed_service_identity`およびファイルパス（`DATA INFILE`）を次のように設定します：

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

### サービスプリンシパル

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで、`azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、および`azure.adls1.oauth2_endpoint`を次のように設定します：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_azure
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://10.1.0.18:9083",
    "azure.adls1.oauth2_client_id" = "<application_client_id>",
    "azure.adls1.oauth2_credential" = "<application_client_credential>",
    "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
);
```

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで、`azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、`azure.adls1.oauth2_endpoint`、およびファイルパス（`path`）を次のように設定します：

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

#### ブローカーロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで、`azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、`azure.adls1.oauth2_endpoint`、およびファイルパス（`DATA INFILE`）を次のように設定します：

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

StarRocksは、Data Lake Storage Gen2へのアクセスに次の認証方法のいずれかを使用することができます。

- 管理アイデンティティ
- 共有キー
- サービスプリンシパル

> **注意**
>
> Data Lake Storage Gen2からデータをロードするか、ファイルをクエリする場合は、データにアクセスするためにabfsまたはabfssプロトコルを使用する必要があります：
>
> - ストレージアカウントがHTTP経由でアクセスを許可している場合は、abfsプロトコルを使用し、ファイルパスを`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`として書きます。
> - ストレージアカウントがHTTPS経由でアクセスを許可している場合は、abfssプロトコルを使用し、ファイルパスを`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`として書きます。

### 管理アイデンティティ

開始する前に、StarRocksクラスタが展開されている仮想マシン（VM）を編集し、これらのVMに管理アイデンティティを追加する必要があります。また、管理アイデンティティがストレージアカウント内のデータを読み取るために承認された役割（**Storage Blob Data Reader**）に関連付けられていることを確認してください。

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで、`azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、および`azure.adls2.oauth2_client_id`を次のように設定します：

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

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで、`azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id`、およびファイルパス（`path`）を次のように設定します：

```SQL
CREATE EXTERNAL TABLE external_table_azure
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "abfs[s]://<container>@<storage_account>.dfs.core.windows.net/<file_name>",
    "format" = "ORC",
    "azure.adls2.oauth2_use_managed_identity" = "true",
    "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
    "azure.adls2.oauth2_client_id" = "<service_client_id>"
);
```

#### ブローカーロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで、`azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id`、およびファイルパス（`DATA INFILE`）を次のように設定します：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("adfs[s]://<container>@<storage_account>.dfs.core.windows.net/<file_name>")
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

### 共有キー

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで、`azure.adls2.storage_account`および`azure.adls2.shared_key`を次のように設定します：

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

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで、`azure.adls2.storage_account`、`azure.adls2.shared_key`、およびファイルパス（`path`）を次のように設定します：

```SQL
CREATE EXTERNAL TABLE external_table_azure
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "abfs[s]://<container>@<storage_account>.dfs.core.windows.net/<file_name>",
    "format" = "ORC",
    "azure.adls2.storage_account" = "<storage_account_name>",
    "azure.adls2.shared_key" = "<shared_key>"
);
```

#### ブローカーロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで、`azure.adls2.storage_account`、`azure.adls2.shared_key`、およびファイルパス（`DATA INFILE`）を次のように設定します：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("adfs[s]://<container>@<storage_account>.dfs.core.windows.net/<file_name>")
    INTO TABLE target_table
    FORMAT AS "parquet"
)
WITH BROKER
(
    "azure.adls2.storage_account" = "<storage_account_name>",
    "azure.adls2.shared_key" = "<shared_key>"
);
```

### サービスプリンシパル

開始する前に、サービスプリンシパルを作成し、サービスプリンシパルに役割を割り当てるためのロール割り当てを作成し、そのロール割り当てをストレージアカウントに追加する必要があります。これにより、このサービスプリンシパルがストレージアカウント内のデータに正常にアクセスできることが保証されます。

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで、`azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、および`azure.adls2.oauth2_client_endpoint`を次のように設定します：

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

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで、`azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint`、およびファイルパス（`path`）を次のように設定します：

```SQL
CREATE EXTERNAL TABLE external_table_azure
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "abfs[s]://<container>@<storage_account>.dfs.core.windows.net/<file_name>",
    "format" = "ORC",
    "azure.adls2.oauth2_client_id" = "<service_client_id>",
    "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
    "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
);
```

#### ブローカーロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで、`azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint`、およびファイルパス（`DATA INFILE`）を次のように設定します：

```SQL
LOAD LABEL test_db.label000
(
    DATA INFILE("adfs[s]://<container>@<storage_account>.dfs.core.windows.net/<file_name>")
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
