---
displayed_sidebar: "Japanese"
---

# Microsoft Azure Storageへの認証

v3.0以降、StarRocksでは、次のシナリオにおいてMicrosoft Azure Storage（Azure Blob StorageまたはAzure Data Lake Storage）と連携することができます。

- Azure Storageからのデータのバッチロード。
- Azure Storageへのデータのバックアップおよびリストア。
- Azure Storage内のParquetとORCファイルのクエリ。
- Azure Storage内の[Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)、および[Delta Lake](../data_source/catalog/deltalake_catalog.md)テーブルのクエリ。

StarRocksは次の種類のAzure Storageアカウントをサポートしています。

- Azure Blob Storage
- Azure Data Lake Storage Gen1
- Azure Data Lake Storage Gen2

このトピックでは、Hiveカタログ、ファイル外部テーブル、およびブローカーロードを使用して、これらの種類のAzure Storageアカウントを使用してStarRocksがAzure Storageと連携する方法を示す例を使用しています。例のパラメータに関する情報については、[Hive catalog](../data_source/catalog/hive_catalog.md)、[File external table](../data_source/file_external_table.md)、および[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

## Blob Storage

StarRocksでは、Blob Storageへアクセスするために、次の認証方法のいずれかを使用することができます。

- 共有キー
- SAS トークン

> **注意**
>
> Blob Storageからデータをロードしたり、ファイルを直接クエリする場合は、データにアクセスするためにwasbプロトコルまたはwasbsプロトコルを使用する必要があります。
>
> - ストレージアカウントがHTTP経由でアクセスを許可している場合は、wasbプロトコルを使用し、ファイルパスを `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>` と書きます。
> - ストレージアカウントがHTTPS経由でアクセスを許可している場合は、wasbsプロトコルを使用し、ファイルパスを `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>` と書きます。

### 共有キー

#### 外部カタログ

次のように、[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントの`azure.blob.storage_account`および`azure.blob.shared_key`を構成します：

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

次のように、[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントの`azure.blob.storage_account`、`azure.blob.shared_key`、およびファイルパス(`path`)を構成します：

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

次のように、[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントの`azure.blob.storage_account`、`azure.blob.shared_key`、およびファイルパス(`DATA INFILE`)を構成します：

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

### SAS トークン

#### 外部カタログ

次のように、[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントの`azure.blob.account_name`、`azure.blob.container_name`、および`azure.blob.sas_token`を構成します：

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

次のように、[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントの`azure.blob.account_name`、`azure.blob.container_name`、`azure.blob.sas_token`、およびファイルパス(`path`)を構成します：

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

次のように、[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントの`azure.blob.account_name`、`azure.blob.container_name`、`azure.blob.sas_token`、およびファイルパス(`DATA INFILE`)を構成します：

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

StarRocksでは、Data Lake Storage Gen1へアクセスするために、次の認証方法の一つを使用することができます。

- 管理されたサービスアイデンティティ
- サービスプリンシパル

> **注意**
>
> Data Lake Storage Gen1からデータをロードしたりファイルをクエリする場合は、データにアクセスするためにadlプロトコルを使用し、ファイルパスを `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>` と書きます。

### 管理されたサービスアイデンティティ

#### 外部カタログ

次のように、[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントの`azure.adls1.use_managed_service_identity`を構成します：

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

次のように、[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントの`azure.adls1.use_managed_service_identity`とファイルパス(`path`)を構成します：

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

次のように、[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントの`azure.adls1.use_managed_service_identity`とファイルパス(`DATA INFILE`)を構成します：

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

次のように、[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントの`azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、および`azure.adls1.oauth2_endpoint`を構成します：

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

#### ファイル外部テーブル

[CROSS REFERENCE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントの中で、次のように`azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、`azure.adls1.oauth2_endpoint`、およびファイルパス(`path`)を以下のように設定します。

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

#### ブローカー・ロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントの中で、次のように`azure.adls1.oauth2_client_id`、`azure.adls1.oauth2_credential`、`azure.adls1.oauth2_endpoint`、およびファイルパス(`DATA INFILE`)を以下のように設定します。

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

## データレイク・ストレージGen2

StarRocksでは、次のいずれかの認証方法を使用してデータレイク・ストレージGen2にアクセスすることができます。

- 管理された ID
- 共有キー
- サービス プリンシパル

> **注意**
>
> データレイク・ストレージGen2からデータをロードしたりファイルをクエリする場合、データにアクセスするためにabfsまたはabfssプロトコルを使用する必要があります。
>
> - ストレージ アカウントがHTTP経由でのアクセスを許可している場合、abfsプロトコルを使用してファイル パスを`abfs://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>`として書き込んでください。
> - ストレージ アカウントがHTTPS経由でのアクセスを許可している場合、abfssプロトコルを使用してファイルパスを`abfss://<container>@<storage_account>.dfs.core.windows.net/<path>/<file_name>`として書き込んでください。

### 管理された ID

開始する前に、次の準備を行う必要があります。

- StarRocksクラスタが展開されている仮想マシン(VM)を編集します。
- これらのVMに管理された ID を追加します。
- 管理された ID がストレージ アカウント内のデータを読み取る権限がある(**Storage Blob Data Reader**)役割に関連付けられていることを確認します。

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントの中で、次のように`azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、および`azure.adls2.oauth2_client_id`を設定します。

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

[CROSS REFERENCE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントの中で、次のように`azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id`、およびファイルパス(`path`)を以下のように設定します。

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

#### ブローカー・ロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントの中で、次のように`azure.adls2.oauth2_use_managed_identity`、`azure.adls2.oauth2_tenant_id`、`azure.adls2.oauth2_client_id`、およびファイルパス(`DATA INFILE`)を以下のように設定します。

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

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントの中で、次のように`azure.adls2.storage_account`と`azure.adls2.shared_key`を設定します。

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

[CROSS REFERENCE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントの中で、次のように`azure.adls2.storage_account`、`azure.adls2.shared_key`、およびファイルパス(`path`)を以下のように設定します。

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

#### ブローカー・ロード

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントの中で、次のように`azure.adls2.storage_account`、`azure.adls2.shared_key`、およびファイルパス(`DATA INFILE`)を以下のように設定します。

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

開始する前に、サービス プリンシパルを作成し、サービス プリンシパルに役割を割り当て、その後その役割をストレージ アカウントに追加する必要があります。これにより、このサービス プリンシパルがストレージ アカウント内のデータに正常にアクセスできることが確認できます。

#### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントの中で、次のように`azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、および`azure.adls2.oauth2_client_endpoint`を設定します。

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

[CROSS REFERENCE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントの中で、次のように`azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint`、およびファイルパス(`path`)を以下のように設定します。
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

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメント内で、`azure.adls2.oauth2_client_id`、`azure.adls2.oauth2_client_secret`、`azure.adls2.oauth2_client_endpoint`、およびファイルパス（`DATA INFILE`）を次のように設定します。

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