---
displayed_sidebar: English
---

# Google Cloud Storage への認証

## 認証方法

v3.0以降、StarRocksは以下の認証方法のいずれかを使用してGoogle Cloud Storage(GCS)にアクセスすることができます：

- VMベースの認証

  Google Cloud Compute Engineに添付されたクレデンシャルを使用してGCSを認証します。

- サービスアカウントベースの認証

  サービスアカウントを使用してGCSを認証します。

- 権限代行ベースの認証

  サービスアカウントまたは仮想マシン(VM)インスタンスが別のサービスアカウントの権限を代行します。

## シナリオ

StarRocksは以下のシナリオでGCSに認証できます：

- GCSからバッチデータをロードします。
- GCSからデータをバックアップし、GCSにデータを復元します。
- GCS内のParquetおよびORCファイルをクエリします。
- GCS内の[Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)、[Delta Lake](../data_source/catalog/deltalake_catalog.md)テーブルをクエリします。

このトピックでは、[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[ファイル外部テーブル](../data_source/file_external_table.md)、および[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を例に、StarRocksがさまざまなシナリオでGCSとどのように統合するかを示します。例における`StorageCredentialParams`については、このトピックの「[パラメータ](../integrations/authenticate_to_gcs.md#parameters)」セクションを参照してください。

> **注記**
>
> StarRocksは、gsプロトコルに基づいてのみ、GCSからデータをロードしたりファイルを直接クエリしたりすることをサポートしています。そのため、GCSからデータをロードしたりファイルをクエリしたりする際には、ファイルパスに`gs`をプレフィックスとして含める必要があります。

### 外部カタログ

GCSからファイルをクエリするために、[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントを使用して、`hive_catalog_gcs`という名前のHiveカタログを以下のように作成します：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_gcs
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    StorageCredentialParams
);
```

### ファイル外部テーブル

メタストアを使用せずにGCSから`test_file_external_tbl`という名前のデータファイルをクエリするために、[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントを使用して、`external_table_gcs`という名前のファイル外部テーブルを以下のように作成します：

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

### Broker Load

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントを使用して、`test_db.label000`というラベルのBroker Loadジョブを作成し、GCSからStarRocksテーブル`target_table`にバッチでデータをロードします：

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

## パラメータ

`StorageCredentialParams`は、異なる認証方法を使用してGCSに認証する方法を記述するパラメータセットを表します。

### VMベースの認証

StarRocksクラスタがGoogle Cloud Platform(GCP)上のVMインスタンスにデプロイされており、そのVMインスタンスを使用してGCSを認証する場合、`StorageCredentialParams`を以下のように設定します：

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true"
```

以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

| **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                              |
| ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineに紐づけられたサービスアカウントを直接使用するかどうかを指定します。 |

### サービスアカウントベースの認証

直接サービスアカウントを使用してGCSを認証する場合、`StorageCredentialParams`を以下のように設定します：

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

| **パラメータ**                          | **デフォルト値** | **値の例**                                       | **説明**                                              |
| -------------------------------------- | ----------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""                | "`user@hello.iam.gserviceaccount.com`"                        | サービスアカウント作成時に生成されたJSONファイル内のメールアドレス。 |
| gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                  | サービスアカウント作成時に生成されたJSONファイル内のプライベートキーID。 |
| gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n" | サービスアカウント作成時に生成されたJSONファイル内のプライベートキー。 |

### 権限代行ベースの認証

#### VMインスタンスがサービスアカウントの権限を代行する

StarRocksクラスタがGCP上のVMインスタンスにデプロイされており、そのVMインスタンスがサービスアカウントの権限を代行してStarRocksがGCSへのアクセス権を継承する場合、`StorageCredentialParams`を以下のように設定します：

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true",
"gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
```

以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

| **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                              |
| ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineに紐づけられたサービスアカウントを直接使用するかどうかを指定します。 |
| gcp.gcs.impersonation_service_account      | ""                | "hello"               | 代行するサービスアカウント。            |

#### サービスアカウントが別のサービスアカウントの権限を代行する

メタサービスアカウントとしてのサービスアカウントがデータサービスアカウントとしての別のサービスアカウントの権限を代行し、StarRocksがGCSへのアクセス権を継承する場合、`StorageCredentialParams`を以下のように設定します：

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
"gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
"gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
```

以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

| **パラメータ**                          | **デフォルト値** | **値の例**                                       | **説明**                                              |
| -------------------------------------- | ----------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""                | "`user@hello.iam.gserviceaccount.com`"                        | メタサービスアカウント作成時に生成されたJSONファイル内のメールアドレス。 |
| gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                  | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキーID。 |
| gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n" | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキー。 |
| gcp.gcs.impersonation_service_account  | ""                | "hello"                                                     | 代行するデータサービスアカウント。       |
