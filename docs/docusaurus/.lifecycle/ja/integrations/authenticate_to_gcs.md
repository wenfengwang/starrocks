---
displayed_sidebar: "Japanese"
---

# Google Cloud Storageへの認証

## 認証方法

v3.0以降、StarRocksではGoogle Cloud Storage（GCS）へアクセスするための以下の認証方法がサポートされています：

- VMベースの認証

  Google Cloud Compute Engineにアタッチされたクレデンシャルを使用して、GCSに認証します。

- サービスアカウントベースの認証

  サービスアカウントを使用して、GCSに認証します。

- インパーソネーションベースの認証

  サービスアカウントまたは仮想マシン（VM）インスタンスを別のサービスアカウントになりすます。

## シナリオ

StarRocksは、以下のシナリオでGCSへの認証を行うことができます：

- GCSからデータをバッチロードする。
- データをバックアップし、GCSにデータをリストアする。
- GCSでParquetおよびORCファイルをクエリする。
- GCSで[Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)、および[Delta Lake](../data_source/catalog/deltalake_catalog.md)のテーブルをクエリする。

このトピックでは、[Hive catalog](../data_source/catalog/hive_catalog.md)、[file external table](../data_source/file_external_table.md)、および[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)の使用例を示し、StarRocksが異なるシナリオでGCSと統合する方法を示します。「例」セクションの`StorageCredentialParams`に関する詳細は、このトピックの"[パラメータ](../integrations/authenticate_to_gcs.md#parameters)"セクションを参照してください。

> **注意**
>
> StarRocksは、GCSからのデータの読み込みまたはファイルの直接クエリをgsプロトコルに従ってサポートしています。したがって、GCSからデータを読み込むかファイルをクエリする場合は、ファイルパスに`gs`を接頭辞として含める必要があります。

### 外部カタログ

GCSからファイルをクエリするために、以下のように[Hive catalog](../data_source/catalog/hive_catalog.md)という名前のHiveカタログを作成するために[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントを使用します。

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

メタストアなしでGCSからデータファイル`test_file_external_tbl`をクエリするために、以下のように[file external table](../data_source/file_external_table.md)という外部ファイルテーブルを作成するために[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントを使用します。

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

### Brokerロード

StarRocksテーブル`target_table`にデータをバッチロードするために、ラベルが`test_db.label0`のBrokerロードジョブを作成するために、[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントを使用します。

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

StarRocksクラスタがGoogle Cloud Platform（GCP）のVMインスタンスに展開されており、そのVMインスタンスを使用してGCSに認証する場合は、次のように`StorageCredentialParams`を構成します。

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true"
```

次の表は、`StorageCredentialParams`で構成する必要のあるパラメータについて説明しています。

| **パラメータ**                                | **デフォルト値** | **値の例** | **説明**                                           |
| ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |

### サービスアカウントベースの認証

サービスアカウントを直接使用してGCSに認証する場合は、次のように`StorageCredentialParams`を構成します。

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

次の表は、`StorageCredentialParams`で構成する必要のあるパラメータについて説明しています。

| **パラメータ**                          | **デフォルト値** | **値の例**                                       | **説明**                                              |
| -------------------------------------- | ----------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""                | "`user@hello.iam.gserviceaccount.com`"                        | サービスアカウントの作成時に生成されたJSONファイルのメールアドレス。 |
| gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                  | サービスアカウントの作成時に生成されたJSONファイルのプライベートキーID。 |
| gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウントの作成時に生成されたJSONファイルのプライベートキー。 |

### インパーソネーションベースの認証

#### VMインスタンスをサービスアカウントになりすます

StarRocksクラスタがGCPのVMインスタンスに展開されており、そのVMインスタンスをサービスアカウントになりすますことを希望する場合は、GCSへアクセスするためにサービスアカウントから権限を継承するために`StorageCredentialParams`を構成します。

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true",
"gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
```

次の表は、`StorageCredentialParams`で構成する必要のあるパラメータについて説明しています。

| **パラメータ**                                | **デフォルト値** | **値の例** | **説明**                                           |
| ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
| gcp.gcs.impersonation_service_account      | ""                | "hello"               | なりすますことを希望するサービスアカウント。 |

#### サービスアカウントが別のサービスアカウントになりすます

サービスアカウント（一時的にメタサービスアカウントと呼ばれる）が別のサービスアカウント（一時的にデータサービスアカウントと呼ばれる）になり、StarRocksがデータサービスアカウントから権限を継承してGCSにアクセスすることを希望する場合は、`StorageCredentialParams`を構成します。

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
"gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
"gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
```

次の表は、`StorageCredentialParams`で構成する必要のあるパラメータについて説明しています。

| **パラメータ**                          | **デフォルト値** | **値の例**                                       | **説明**                                              |
| -------------------------------------- | ----------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""                | "`user@hello.iam.gserviceaccount.com`"                        | メタサービスアカウントの作成時に生成されたJSONファイルのメールアドレス。 |
| gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                  | メタサービスアカウントの作成時に生成されたJSONファイルのプライベートキーID。 |
| gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | メタサービスアカウントの作成時に生成されたJSONファイルのプライベートキー。 |
| gcp.gcs.impersonation_service_account  | ""                | "hello"                                                     | なりすますことを希望するデータサービスアカウント。 |