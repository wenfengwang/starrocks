---
displayed_sidebar: "Japanese"
---

# Google Cloud Storageへの認証

## 認証方法

v3.0以降、StarRocksはGoogle Cloud Storage（GCS）へのアクセスに以下の認証方法のいずれかを使用することをサポートしています。

- VMベースの認証

   Google Cloud Compute Engineにアタッチされた資格情報を使用してGCSに認証します。

- サービスアカウントベースの認証

   サービスアカウントを使用してGCSに認証します。

- 偽装ベースの認証

   サービスアカウントまたは仮想マシン（VM）インスタンスを他のサービスアカウントに偽装させます。

## シナリオ

StarRocksは次の状況でGCSに認証できます。

- GCSからのデータのバッチロード
- GCSへのデータのバックアップおよびリストア
- GCS内のParquetやORCファイルのクエリ
- GCSの[Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)、および[Delta Lake](../data_source/catalog/deltalake_catalog.md)テーブルのクエリ。

このトピックでは、GCSとのさまざまなシナリオでStarRocksがどのように統合されるかを示すために、[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[file external table](../data_source/file_external_table.md)、および[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)の情報を含めています。「StorageCredentialParams」の例の詳細については、このトピックの"[Parameters](../integrations/authenticate_to_gcs.md#parameters)"セクションを参照してください。

> **注意**
>
> StarRocksは、GCSからデータをロードしたりファイルを直接クエリしたりする場合、gsプロトコルに従う必要があります。そのため、GCSからデータをロードしたりファイルをクエリする場合は、ファイルパスに`gs`をプレフィックスとして含める必要があります。

### 外部カタログ

次のように、Hiveカタログとして`hive_catalog_gcs`という名前のカタログを作成するために、[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントを使用してGCSからファイルをクエリすることができます。

```SQL
CREATE EXTERNAL CATALOG hive_catalog_gcs
PROPERTIES
(
    "type" = "hive", 
    "hive.metastore.uris" = "thrift://34.132.15.127:9083",
    StorageCredentialParams
);
```

### ファイル外部テーブル

次のように、メタストアなしでGCSから`test_file_external_tbl`という名前のデータファイルをクエリするために、[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントを使用して`external_table_gcs`という名前のファイル外部テーブルを作成できます。

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

### ブローカーロード

次のように、StarRocksテーブル`target_table`にデータをバッチロードするために、ラベルが`test_db.label000`のブローカーロードジョブを作成するために、[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントを使用できます。

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

`StorageCredentialParams`は、異なる認証方法でGCSに認証する方法を記述したパラメータセットを表します。

### VMベースの認証

StarRocksクラスタがGoogle Cloud Platform（GCP）でホストされるVMインスタンスに展開されており、そのVMインスタンスを使用してGCSに認証したい場合は、`StorageCredentialParams`を以下のように設定します。

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true"
```

次の表には、`StorageCredentialParams`で構成する必要があるパラメータの詳細が記載されています。

| **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                                     |
| ------------------------------------------ | ----------------- | ----------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false             | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |

### サービスアカウントベースの認証

サービスアカウントを直接使用してGCSに認証する場合、`StorageCredentialParams`を以下のように設定します。

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

次の表には、`StorageCredentialParams`で構成する必要があるパラメータの詳細が記載されています。

| **パラメータ**                          | **デフォルト値** | **値の例**                                     | **説明**                                                     |
| -------------------------------------- | ----------------- | --------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""                | "`user@hello.iam.gserviceaccount.com`"        | サービスアカウントの作成時に生成されたJSONファイル内のメールアドレスです。 |
| gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"  | サービスアカウントの作成時に生成されたJSONファイル内のプライベートキーIDです。 |
| gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウントの作成時に生成されたJSONファイル内のプライベートキーです。 |

### 偽装ベースの認証

#### VMインスタンスがサービスアカウントを偽装する

StarRocksクラスタがGCP上のVMインスタンスに展開され、そのVMインスタンスを特定のサービスアカウントに偽装させる場合、つまりStarRocksがGCSへのアクセス権をサービスアカウントから継承する場合は、`StorageCredentialParams`を以下のように設定します。

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true",
"gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
```

次の表には、`StorageCredentialParams`で構成する必要があるパラメータの詳細が記載されています。

| **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                                     |
| ------------------------------------------ | ----------------- | ----------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false             | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
| gcp.gcs.impersonation_service_account      | ""                | "hello"     | 偽装したいサービスアカウントです。                           |

#### サービスアカウントが別のサービスアカウントを偽装する

サービスアカウント（一時的にメタサービスアカウントとして命名）が別のサービスアカウント（一時的にデータサービスアカウントとして命名）を偽装し、StarRocksがデータサービスアカウントからアクセス権を継承する場合は、`StorageCredentialParams`を以下のように設定します。

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
"gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
"gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
```

次の表には、`StorageCredentialParams`で構成する必要があるパラメータの詳細が記載されています。

| **パラメータ**                          | **デフォルト値** | **値の例**                                     | **説明**                                                     |
| -------------------------------------- | ----------------- | --------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""                | "`user@hello.iam.gserviceaccount.com`"        | メタサービスアカウントの作成時に生成されたJSONファイル内の電子メールアドレスです。 |
| gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"  | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーIDです。 |
| gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーです。 |
| gcp.gcs.impersonation_service_account  | ""                | "hello"     | 偽装したいデータサービスアカウントです。      |