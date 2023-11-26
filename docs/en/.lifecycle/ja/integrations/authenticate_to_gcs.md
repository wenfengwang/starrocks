---
displayed_sidebar: "Japanese"
---

# Google Cloud Storageへの認証

## 認証方法

v3.0以降、StarRocksはGoogle Cloud Storage（GCS）へのアクセスに以下の認証方法のいずれかを使用することができます。

- VMベースの認証

  Google Cloud Compute Engineにアタッチされた認証情報を使用してGCSに認証します。

- サービスアカウントベースの認証

  サービスアカウントを使用してGCSに認証します。

- 偽装ベースの認証

  サービスアカウントまたは仮想マシン（VM）インスタンスを他のサービスアカウントに偽装させます。

## シナリオ

StarRocksは以下のシナリオでGCSに認証することができます。

- GCSからデータをバッチロードする。
- GCSからデータをバックアップおよびリストアする。
- GCSのParquetおよびORCファイルをクエリする。
- GCSの[Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)、および[Delta Lake](../data_source/catalog/deltalake_catalog.md)テーブルをクエリする。

このトピックでは、異なるシナリオでStarRocksがGCSと統合する方法を示すために、[Hiveカタログ](../data_source/catalog/hive_catalog.md)、[ファイル外部テーブル](../data_source/file_external_table.md)、および[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を例として使用します。例の`StorageCredentialParams`に関する詳細については、このトピックの"[パラメータ](../integrations/authenticate_to_gcs.md#parameters)"セクションを参照してください。

> **注意**
>
> StarRocksは、GCSからデータをロードしたりファイルを直接クエリする場合、gsプロトコルのみをサポートしています。したがって、GCSからデータをロードしたりファイルをクエリする場合は、ファイルパスに`gs`を接頭辞として含める必要があります。

### 外部カタログ

以下のように、[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md)ステートメントを使用して、GCSからファイルをクエリするためのHiveカタログ`hive_catalog_gcs`を作成します。

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

以下のように、[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントを使用して、メタストアなしでGCSから名前が`test_file_external_tbl`のデータファイルをクエリするためのファイル外部テーブル`external_table_gcs`を作成します。

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

以下のように、[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)ステートメントを使用して、GCSからデータをバッチロードしてStarRocksテーブル`target_table`に格納するためのラベルが`test_db.label000`のBroker Loadジョブを作成します。

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

`StorageCredentialParams`は、さまざまな認証方法でGCSに認証する方法を記述するパラメータセットを表します。

### VMベースの認証

StarRocksクラスタがGoogle Cloud Platform（GCP）上のVMインスタンスに展開され、そのVMインスタンスを使用してGCSに認証する場合は、`StorageCredentialParams`を次のように設定します。

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true"
```

次の表に、`StorageCredentialParams`で設定する必要のあるパラメータを示します。

| **パラメータ**                                 | **デフォルト値** | **値の例** | **説明**                                                     |
| --------------------------------------------- | ---------------- | ----------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account    | false            | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |

### サービスアカウントベースの認証

サービスアカウントを直接使用してGCSに認証する場合は、`StorageCredentialParams`を次のように設定します。

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

次の表に、`StorageCredentialParams`で設定する必要のあるパラメータを示します。

| **パラメータ**                             | **デフォルト値** | **値の例**                                                | **説明**                                                     |
| ----------------------------------------- | ---------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email             | ""               | "`user@hello.iam.gserviceaccount.com`"                     | サービスアカウントの作成時に生成されたJSONファイル内のメールアドレスです。 |
| gcp.gcs.service_account_private_key_id    | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | サービスアカウントの作成時に生成されたJSONファイル内のプライベートキーIDです。 |
| gcp.gcs.service_account_private_key       | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウントの作成時に生成されたJSONファイル内のプライベートキーです。 |

### 偽装ベースの認証

#### VMインスタンスをサービスアカウントに偽装する

StarRocksクラスタがGCP上のVMインスタンスに展開され、そのVMインスタンスをサービスアカウントに偽装させ、StarRocksがサービスアカウントからGCSにアクセスするための特権を継承する場合は、`StorageCredentialParams`を次のように設定します。

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true",
"gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
```

次の表に、`StorageCredentialParams`で設定する必要のあるパラメータを示します。

| **パラメータ**                                 | **デフォルト値** | **値の例** | **説明**                                                     |
| --------------------------------------------- | ---------------- | ----------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account    | false            | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
| gcp.gcs.impersonation_service_account         | ""               | "hello"     | 偽装したいサービスアカウントです。                            |

#### サービスアカウントが別のサービスアカウントに偽装する

サービスアカウント（一時的にメタサービスアカウントと呼ばれる）が別のサービスアカウント（一時的にデータサービスアカウントと呼ばれる）に偽装し、StarRocksがデータサービスアカウントからGCSにアクセスするための特権を継承する場合は、`StorageCredentialParams`を次のように設定します。

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
"gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
"gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
```

次の表に、`StorageCredentialParams`で設定する必要のあるパラメータを示します。

| **パラメータ**                             | **デフォルト値** | **値の例**                                                | **説明**                                                     |
| ----------------------------------------- | ---------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email             | ""               | "`user@hello.iam.gserviceaccount.com`"                     | メタサービスアカウントの作成時に生成されたJSONファイル内のメールアドレスです。 |
| gcp.gcs.service_account_private_key_id    | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーIDです。 |
| gcp.gcs.service_account_private_key       | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーです。 |
| gcp.gcs.impersonation_service_account     | ""               | "hello"                                                  | 偽装したいデータサービスアカウントです。                    |
