---
displayed_sidebar: Chinese
---

# GCS 認証情報の設定

## 認証方法の紹介

StarRocks はバージョン 3.0 から、以下の認証方法を通じて Google Cloud Storage（GCS）にアクセスすることをサポートしています：

- VM ベースの認証と認可

  Google Cloud Compute Engine にバインドされたクレデンシャルを使用して GCS を認証および認可します。

- Service Account ベースの認証と認可

  Service Account を使用して GCS を認証および認可します。

- Impersonation ベースの認証と認可

  Service Account または VM インスタンスを使用して別の Service Account を模倣し、GCS の認証および認可を実現します。

## 適用シナリオ

StarRocks は以下のシナリオで GCS との統合をサポートしています：

- GCS からのバッチデータのインポート。
- GCS へのデータのバックアップ、または GCS からのデータのリストア。
- GCS 内の Parquet または ORC 形式のデータファイルのクエリ。
- GCS 内の [Hive](../data_source/catalog/hive_catalog.md)、[Iceberg](../data_source/catalog/iceberg_catalog.md)、[Hudi](../data_source/catalog/hudi_catalog.md)、または [Delta Lake](../data_source/catalog/deltalake_catalog.md) テーブルのクエリ。

このドキュメントでは [Hive catalog](../data_source/catalog/hive_catalog.md)、[ファイル外部テーブル](../data_source/file_external_table.md)、および [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を例に、StarRocks が各シナリオで GCS をどのように統合するかを紹介します。以下の例に出現する `StorageCredentialParams` の詳細については、本ドキュメントの「[パラメータ設定](../integrations/authenticate_to_gcs.md#パラメータ設定)」セクションを参照してください。

> **説明**
>
> Broker Load は gs プロトコルを通じて Google GCS にアクセスすることのみをサポートしているため、Google GCS からデータをインポートする際には、ファイルパスに指定された GCS URI が `gs://` をプレフィックスとして使用していることを確認する必要があります。

### 外部カタログ

[CREATE EXTERNAL CATALOG](../sql-reference/sql-statements/data-definition/CREATE_EXTERNAL_CATALOG.md) ステートメントを使用して Hive Catalog `hive_catalog_gcs` を作成し、GCS 内のデータファイルをクエリします：

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

[CREATE EXTERNAL TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントを使用してファイル外部テーブル `external_table_gcs` を作成し、GCS 内のデータファイル `test_file_external_tbl`（GCS にメタデータサービスが設定されていない場合）をクエリします：

```SQL
CREATE EXTERNAL TABLE external_table_gcs
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "gs://test-gcs/test_file_external_tbl",
    "format" = "ORC",
    StorageCredentialParams
);
```

### Broker Load

[LOAD LABEL](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) ステートメントを使用して Broker Load ジョブを作成し、ジョブラベル `test_db.label000` で GCS 内のデータを StarRocks テーブル `target_table` にバッチインポートします：

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

## パラメータ設定

`StorageCredentialParams` は StarRocks が GCS にアクセスするための関連するパラメータ設定を指定するために使用されます。具体的にどのパラメータを含むかは、使用する認証方法によって異なります。

### VM ベースの認証と認可

StarRocks を Google Cloud Platform（GCP）上の VM インスタンスにデプロイし、VM を使用して GCS を認証および認可する場合は、以下のように `StorageCredentialParams` を設定してください：

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true"
```

`StorageCredentialParams` には以下のパラメータが含まれます。

| **パラメータ**                                   | **デフォルト値** | **例** | **説明**                                                 |
| ------------------------------------------ | ---------- | ------------ | -------------------------------------------------------- |
| gcp.gcs.use_compute_engine_service_account | false      | true         | Compute Engine にバインドされた Service Account を直接使用するかどうか。 |

### Service Account ベースの認証と認可

Service Account を使用して GCS を認証および認可する場合は、以下のように `StorageCredentialParams` を設定してください：

```Plain
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

`StorageCredentialParams` には以下のパラメータが含まれます。

| **パラメータ**                               | **デフォルト値** | **例**                                                | **説明**                                                     |
| -------------------------------------- | ---------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""         | "`user@hello.iam.gserviceaccount.com`"                        | Service Account を作成した際に生成された JSON ファイル内の Email。          |
| gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                  | Service Account を作成した際に生成された JSON ファイル内の Private Key ID。 |
| gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | Service Account を作成した際に生成された JSON ファイル内の Private Key。    |

### Impersonation ベースの認証と認可

#### VM インスタンスを使用して Service Account を模倣する

StarRocks を GCP 上の VM インスタンスにデプロイし、その VM インスタンスを使用して別の Service Account を模倣し、StarRocks がその Service Account の GCS アクセス権を継承する場合は、以下のように `StorageCredentialParams` を設定してください：

```Plain
"gcp.gcs.use_compute_engine_service_account" = "true",
"gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
```

`StorageCredentialParams` には以下のパラメータが含まれます。

| **パラメータ**                                   | **デフォルト値** | **例** | **説明**                                                 |
| ------------------------------------------ | ---------- | ------------ | -------------------------------------------------------- |
| gcp.gcs.use_compute_engine_service_account | false      | true         | Compute Engine にバインドされた Service Account を直接使用するかどうか。 |
| gcp.gcs.impersonation_service_account      | ""         | "user@example.com"      | 模倣する対象の Service Account。                         |

#### Service Account を使用して別の Service Account を模倣する

ある Service Account（仮に「Meta Service Account」とします）を使用して別の Service Account（仮に「Data Service Account」とします）を模倣し、StarRocks がその Data Service Account の GCS アクセス権を継承する場合は、以下のように `StorageCredentialParams` を設定してください：

```Plain
"gcp.gcs.service_account_email" = "<meta_google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<meta_google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<meta_google_service_private_key>",
"gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
```

`StorageCredentialParams` には以下のパラメータが含まれます。

| **パラメータ**                               | **デフォルト値** | **例**                                                | **説明**                                                     |
| -------------------------------------- | ---------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""         | "`meta@example.com`"                        | Meta Service Account を作成した際に生成された JSON ファイル内の Email。     |
| gcp.gcs.service_account_private_key_id | ""         | "1234567890abcdef1234567890abcdef12345678"                  | Meta Service Account を作成した際に生成された JSON ファイル内の Private Key ID。 |
| gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | Meta Service Account を作成した際に生成された JSON ファイル内の Private Key。 |
| gcp.gcs.impersonation_service_account  | ""         | "data@example.com"                                                     | 模倣する対象の Data Service Account。                        |
