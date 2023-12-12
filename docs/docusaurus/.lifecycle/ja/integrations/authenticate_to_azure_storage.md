---
displayed_sidebar: "Japanese"
---

# Microsoft Azure Storageへの認証

v3.0以降、StarRocksでは、次のシナリオでMicrosoft Azure Storage（Azure Blob StorageまたはAzure Data Lake Storage）と統合できます。

- Azure Storageからのデータのバッチロード。
- Azure Storageへのデータのバックアップおよびリストア。
- Azure Storage内のParquetおよびORCファイルのクエリ。
- Azure Storage内の[Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)、および[Delta Lake](../data_source/catalog/deltalake_catalog.md)テーブルのクエリ。

StarRocksでは、次の種類のAzure Storageアカウントがサポートされています。

- Azure Blob Storage
- Azure Data Lake Storage Gen1
- Azure Data Lake Storage Gen2

このトピックでは、Hiveカタログ、ファイル外部テーブル、およびブローカーロードを使用して、StarRocksがこれらの種類のAzure Storageアカウントを使用してAzure Storageと統合する方法を例として示します。例に関するパラメータの詳細については、[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[ファイル外部テーブル](../data_source/file_external_table.md)、および[ブローカーロード](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

## Blob Storage

StarRocksでは、Blob Storageへアクセスするために次のいずれかの認証メソッドを使用できます。

- 共有キー
- SASトークン

> **注意**
>
> Blob Storageからデータをロードしたり、ファイルを直接クエリする場合は、データにアクセスするためにwasbまたはwasbsプロトコルを使用する必要があります。
>
> - ストレージアカウントがHTTP経由でアクセスを許可している場合は、wasbプロトコルを使用し、ファイルパスを`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>`として書きます。
> - ストレージアカウントがHTTPS経由でアクセスを許可している場合は、wasbsプロトコルを使用し、ファイルパスを`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>`として書きます。

### 共有キー

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで、次のように`azure.blob.storage_account`および`azure.blob.shared_key`を設定します。

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

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで、次のように`azure.blob.storage_account`、`azure.blob.shared_key`、およびファイルパス(`path`)を設定します。

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

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで、次のように`azure.blob.storage_account`、`azure.blob.shared_key`、およびファイルパス(`DATA INFILE`)を設定します。

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

### SASトークン

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントで、次のように`azure.blob.account_name`、`azure.blob.container_name`、および`azure.blob.sas_token`を設定します。

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

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントで、次のように`azure.blob.account_name`、`azure.blob.container_name`、`azure.blob.sas_token`、およびファイルパス(`path`)を設定します。

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

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントで、次のように`azure.blob.account_name`、`azure.blob.container_name`、`azure.blob.sas_token`、およびファイルパス(`DATA INFILE`)を設定します。

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

StarRocksでは、Data Lake Storage Gen1へアクセスするために次のいずれかの認証メソッドを使用できます。

- マネージドサービスアイデンティティ
- サービスプリンシパル

> **注意**
>
> Data Lake Storage Gen1からデータをロードしたりファイルをクエリする場合は、データにアクセスするためにadlプロトコルを使用する必要があり、ファイルパスを`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`として書きます。
```
    "azure.adls1.oauth2_credential" = "<application_client_credential>",
    "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
);
```

#### ファイル外部テーブル

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントで、次のように `azure.adls1.oauth2_client_id`, `azure.adls1.oauth2_credential`, `azure.adls1.oauth2_endpoint` とファイルパス (`path`) を設定します。

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

#### ブローカー ロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントで、次のように `azure.adls1.oauth2_client_id`, `azure.adls1.oauth2_credential`, `azure.adls1.oauth2_endpoint` とファイルパス (`DATA INFILE`) を設定します。

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

## データ レイク ストレージ Gen2

StarRocks では、次のいずれかの認証方法を使用して Data Lake Storage Gen2 にアクセスすることができます。

- 管理された ID
- 共有キー
- サービス プリンシパル

> **注意**
>
> Data Lake Storage Gen2 からデータをロードしたりクエリのファイルを実行する場合、データにアクセスするために abfs または abfss プロトコルを使用する必要があります。
>
> - ストレージ アカウントが HTTP 経由でアクセスを許可している場合は、abfs プロトコルを使用し、ファイル パスを `abfs://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>` として指定します。
> - ストレージ アカウントが HTTPS 経由でアクセスを許可している場合は、abfss プロトコルを使用し、ファイル パスを `abfss://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>` として指定します。

### 管理された ID

開始する前に、次の準備を行う必要があります。

- StarRocks クラスターが展開されている仮想マシン (VM) を編集します。
- これらの VM に管理された ID を追加します。
- 管理された ID がストレージ アカウントでデータを読み取る権限がある (**Storage Blob Data Reader** のロールを割り当てられている) ことを確認します。

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントで、次のように `azure.adls2.oauth2_use_managed_identity`, `azure.adls2.oauth2_tenant_id`, `azure.adls2.oauth2_client_id` を設定します。

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

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントで、次のように `azure.adls2.oauth2_use_managed_identity`, `azure.adls2.oauth2_tenant_id`, `azure.adls2.oauth2_client_id` とファイルパス (`path`) を設定します。

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

#### ブローカー ロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントで、次のように `azure.adls2.oauth2_use_managed_identity`, `azure.adls2.oauth2_tenant_id`, `azure.adls2.oauth2_client_id` とファイルパス (`DATA INFILE`) を設定します。

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

### 共有キー

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントで、次のように `azure.adls2.storage_account` と `azure.adls2.shared_key` を設定します。

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

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントで、次のように `azure.adls2.storage_account`, `azure.adls2.shared_key` とファイルパス (`path`) を設定します。

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

#### ブローカー ロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントで、次のように `azure.adls2.storage_account`, `azure.adls2.shared_key` とファイルパス (`DATA INFILE`) を設定します。

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

### サービス プリンシパル

開始する前に、サービス プリンシパルを作成し、ロールを割り当ててストレージ アカウントにロール割り当てを追加し、その結果、このサービス プリンシパルがストレージ アカウント内のデータに正常にアクセスできるようにします。

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントで、次のように `azure.adls2.oauth2_client_id`, `azure.adls2.oauth2_client_secret`, `azure.adls2.oauth2_client_endpoint` を設定します。

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

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントで、次のように `azure.adls2.oauth2_client_id`, `azure.adls2.oauth2_client_secret`, `azure.adls2.oauth2_client_endpoint` とファイルパス (`path`) を設定します。

```SQL
CREATE EXTERNAL TABLE external_table_azure
(
```japanese
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

#### ブローカーの読み込み

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメント内のファイルパス(`DATA INFILE`)と`azure.adls2.oauth2_client_id`,`azure.adls2.oauth2_client_secret`,`azure.adls2.oauth2_client_endpoint`を次のように設定して、`target_table` に `parquet` フォーマットで以下のように設定します:

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