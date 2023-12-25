---
displayed_sidebar: Chinese
---

# Hive カタログ

Hive Catalog は External Catalog の一種で、バージョン 2.3 からサポートされています。Hive Catalog を使用すると、以下のことができます:

- 手動でテーブルを作成することなく、Hive Catalog を通じて Hive 内のデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) や非同期マテリアライズドビュー（バージョン 3.1 以降）を使用して、Hive 内のデータを加工・モデリングし、StarRocks にインポートできます。
- StarRocks 側で Hive のデータベースやテーブルを作成・削除したり、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して StarRocks のテーブルデータを Parquet 形式の Hive テーブルに書き込むことができます（バージョン 3.2 以降）。

Hive 内のデータに正常にアクセスするために、StarRocks クラスターは以下の二つの重要なコンポーネントを統合する必要があります:

- 分散ファイルシステム（HDFS）またはオブジェクトストレージ。現在サポートされているオブジェクトストレージには、AWS S3、Microsoft Azure Storage、Google GCS、その他の S3 プロトコル互換のオブジェクトストレージ（例：アリババクラウド OSS、MinIO）が含まれます。

- メタデータサービス。現在サポートされているメタデータサービスには、Hive Metastore（以下 HMS と略称）、AWS Glue が含まれます。

  > **説明**
  >
  > AWS S3 をストレージシステムとして選択した場合、HMS または AWS Glue をメタデータサービスとして選択できます。他のストレージシステムを選択した場合は、HMS をメタデータサービスとして選択する必要があります。

## 使用説明

- StarRocks が Hive データをクエリする際、Parquet、ORC、CSV、Avro、RCFile、SequenceFile のファイルフォーマットをサポートしており、以下の通りです:

  - Parquet ファイルは、SNAPPY、LZ4、ZSTD、GZIP、NO_COMPRESSION の圧縮フォーマットをサポートしています。バージョン 3.1.5 からは、LZO 圧縮フォーマットもサポートしています。
  - ORC ファイルは、ZLIB、SNAPPY、LZO、LZ4、ZSTD、NO_COMPRESSION の圧縮フォーマットをサポートしています。
  - CSV ファイルは、バージョン 3.1.5 から LZO 圧縮フォーマットをサポートしています。

- StarRocks が Hive データをクエリする際、INTERVAL、BINARY、UNION の三つのデータタイプはサポートされていません。さらに、CSV 形式の Hive テーブルについては、StarRocks は MAP、STRUCT データタイプをサポートしていません。
- Hive Catalog は Hive データのクエリのみをサポートし、Hive への書き込み/削除操作はサポートしていません。

## 準備作業

Hive Catalog を作成する前に、StarRocks クラスターが Hive のファイルストレージおよびメタデータサービスに正常にアクセスできることを確認してください。

### AWS IAM

Hive が AWS S3 をファイルストレージとして使用する場合、または AWS Glue をメタデータサービスとして使用する場合、StarRocks クラスターが関連する AWS クラウドリソースにアクセスできるように、適切な認証および承認スキームを選択する必要があります。

以下の認証および承認スキームから選択できます:

- Instance Profile（推奨）
- Assumed Role
- IAM User

StarRocks が AWS 認証および承認にアクセスする詳細については、[AWS 認証方法の設定 - 準備作業](../../integrations/authenticate_to_aws_resources.md#准备工作)を参照してください。

### HDFS

HDFS をファイルストレージとして使用する場合、StarRocks クラスターで以下の設定を行う必要があります:

- （オプション）HDFS クラスターおよび HMS にアクセスするためのユーザー名を設定します。各 FE の **fe/conf/hadoop_env.sh** ファイルおよび各 BE の **be/conf/hadoop_env.sh** ファイルの最初に `export HADOOP_USER_NAME="<user_name>"` を追加して、このユーザー名を設定できます。設定後、各 FE および BE を再起動して設定を有効にします。このユーザー名を設定しない場合、デフォルトでは FE および BE プロセスのユーザー名が使用されます。各 StarRocks クラスターは一つのユーザー名のみを設定できます。
- Hive データをクエリする際、StarRocks クラスターの FE および BE は HDFS クライアントを通じて HDFS クラスターにアクセスします。通常、StarRocks はデフォルト設定で HDFS クライアントを起動し、手動での設定は必要ありません。ただし、以下のシナリオでは手動での設定が必要です:
  - HDFS クラスターが高可用性（High Availability、HA）モードを有効にしている場合、HDFS クラスター内の **hdfs-site.xml** ファイルを各 FE の **$FE_HOME/conf** ディレクトリおよび各 BE の **$BE_HOME/conf** ディレクトリに配置する必要があります。
  - HDFS クラスターが ViewFs を設定している場合、HDFS クラスター内の **core-site.xml** ファイルを各 FE の **$FE_HOME/conf** ディレクトリおよび各 BE の **$BE_HOME/conf** ディレクトリに配置する必要があります。

> **注意**
>
> クエリ時にドメイン名が認識できない（Unknown Host）ためにアクセスに失敗する場合、HDFS クラスター内の各ノードのホスト名と IP アドレスのマッピングを **/etc/hosts** に設定する必要があります。

### Kerberos 認証

HDFS クラスターまたは HMS が Kerberos 認証を有効にしている場合、StarRocks クラスターで以下の設定を行う必要があります:

- 各 FE および各 BE で `kinit -kt keytab_path principal` コマンドを実行し、Key Distribution Center (KDC) から Ticket Granting Ticket (TGT) を取得します。このコマンドを実行するユーザーは、HMS および HDFS にアクセスする権限を持っている必要があります。このコマンドを使用して KDC にアクセスすることには有効期限があるため、cron を使用して定期的に実行する必要があります。
- 各 FE の **$FE_HOME/conf/fe.conf** ファイルおよび各 BE の **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。ここで、`/etc/krb5.conf` は **krb5.conf** ファイルのパスであり、実際のファイルのパスに応じて変更できます。

## Hive カタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "hive",
    GeneralParams,
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### パラメータ説明

#### catalog_name

Hive Catalog の名前。命名規則は以下の通りです:

- 英字 (a-z または A-Z)、数字 (0-9)、アンダースコア (_) で構成され、英字で始まる必要があります。
- 全体の長さは 1023 文字を超えてはいけません。
- カタログ名は大文字と小文字を区別します。

#### コメント

Hive Catalog の説明。このパラメータはオプションです。

#### type

データソースのタイプ。`hive` に設定します。

#### GeneralParams (一般パラメーター)

一般的な設定を指定するパラメータのセットです。

`GeneralParams` には以下のパラメータが含まれます。

| パラメータ                      | 必須  | 説明                                                         |
| ------------------------ | -------- | ------------------------------------------------------------ |
| enable_recursive_listing | いいえ       | StarRocks がテーブルやパーティションディレクトリ（サブディレクトリを含む）内のファイルデータを再帰的に読み取るかどうかを指定します。値の範囲: `true` または `false`。デフォルト値: `false`。`true` は再帰的にトラバースすることを意味し、`false` はテーブルまたはパーティションディレクトリの現在のレベルのファイルデータのみを読み取ることを意味します。|

#### MetastoreParams (メタストアパラメータ)

StarRocks が Hive クラスターのメタデータサービスにアクセスするための関連パラメータ設定です。

##### Hive メタストア

HMS を Hive クラスターのメタデータサービスとして選択する場合、`MetastoreParams` を以下のように設定してください:

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **説明**
>
> Hive データをクエリする前に、すべての HMS ノードのホスト名と IP アドレスのマッピングを **/etc/hosts** に追加する必要があります。そうしないと、クエリを実行する際に StarRocks が HMS にアクセスできない可能性があります。

`MetastoreParams` には以下のパラメータが含まれます。

| パラメータ                | 必須   | 説明                                                         |
| ------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type | はい       | Hive クラスターで使用されるメタデータサービスのタイプ。`hive` に設定します。 |
| hive.metastore.uris | はい       | HMS の URI。形式: `thrift://<HMS IP アドレス>:<HMS ポート番号>`。<br />HMS が高可用性モードを有効にしている場合、複数の HMS アドレスをコンマで区切って記述できます。例: `"thrift://<HMS IP アドレス 1>:<HMS ポート番号 1>,thrift://<HMS IP アドレス 2>:<HMS ポート番号 2>,thrift://<HMS IP アドレス 3>:<HMS ポート番号 3>"`。 |

##### AWS Glue

AWS S3 をストレージシステムとして使用する場合にのみ、AWS Glue を Hive クラスターのメタデータサービスとして選択できます。その場合、`MetastoreParams` を以下のように設定してください:

- Instance Profile を使用した認証と承認

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- Assumed Role を使用した認証と承認

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAM User を使用した認証と承認

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

`MetastoreParams` には以下のパラメータが含まれます。

| パラメータ                          | 必須   | 説明                                                         |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type           | はい       | Hive クラスターで使用されるメタデータサービスのタイプ。`glue` に設定します。 |
| aws.glue.use_instance_profile | はい       | Instance Profile と Assumed Role の二つの認証方式を有効にするかどうかを指定します。値の範囲: `true` または `false`。デフォルト値: `false`。 |
| aws.glue.iam_role_arn         | いいえ       | AWS Glue Data Catalog にアクセスする権限を持つ IAM Role の ARN。Assumed Role 認証方式を使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.region               | はい       | AWS Glue Data Catalog が存在するリージョン。例: `us-west-1`。 |
| aws.glue.access_key           | いいえ       | IAM User の Access Key。IAM User 認証方式を使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | いいえ       | IAM User の Secret Key。IAM User 認証方式を使用する場合は、このパラメータを指定する必要があります。 |


AWS Glue へのアクセスに使用する認証方法の選択と、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS Glue へのアクセス認証パラメータ](../../integrations/authenticate_to_aws_resources.md#aws-glue-へのアクセス認証パラメータ)を参照してください。

#### StorageCredentialParams

StarRocks が Hive クラスターのファイルストレージにアクセスするためのパラメータ設定です。

HDFS をストレージシステムとして使用する場合は、`StorageCredentialParams` を設定する必要はありません。

AWS S3、S3 プロトコル互換のオブジェクトストレージ、Microsoft Azure Storage、または GCS を使用する場合は、`StorageCredentialParams` を設定する必要があります。

##### AWS S3

AWS S3 を Hive クラスターのファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- Instance Profile を使用した認証と認可

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- Assumed Role を使用した認証と認可

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM User を使用した認証と認可

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

`StorageCredentialParams` には以下のパラメータが含まれます。

| パラメータ                        | 必須   | 説明                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい       | Instance Profile と Assumed Role の認証方法を使用するかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`false`。 |
| aws.s3.iam_role_arn         | いいえ       | AWS S3 Bucket にアクセスする権限を持つ IAM Role の ARN。Assumed Role 認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい       | AWS S3 Bucket が存在するリージョン。例：`us-west-1`。                |
| aws.s3.access_key           | いいえ       | IAM User のアクセスキー。IAM User 認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAM User のシークレットキー。IAM User 認証方法を使用する場合は、このパラメータを指定する必要があります。 |

AWS S3 へのアクセスに使用する認証方法の選択と、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS S3 へのアクセス認証パラメータ](../../integrations/authenticate_to_aws_resources.md#aws-s3-へのアクセス認証パラメータ)を参照してください。

##### 阿里云 OSS

阿里云 OSS を Hive クラスターのファイルストレージとして選択する場合は、`StorageCredentialParams` に以下の認証パラメータを設定する必要があります：

```SQL
"aliyun.oss.access_key" = "<user_access_key>",
"aliyun.oss.secret_key" = "<user_secret_key>",
"aliyun.oss.endpoint" = "<oss_endpoint>" 
```

| パラメータ                            | 必須 | 説明                                                         |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aliyun.oss.endpoint             | はい      | 阿里云 OSS のエンドポイント。例：`oss-cn-beijing.aliyuncs.com`。エンドポイントとリージョンの関係を調べるには、[アクセスドメインとデータセンター](https://help.aliyun.com/document_detail/31837.html)を参照してください。    |
| aliyun.oss.access_key           | はい      | 阿里云アカウントまたは RAM ユーザーの AccessKey ID。取得方法については、[AccessKey の取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。                                     |
| aliyun.oss.secret_key           | はい      | 阿里云アカウントまたは RAM ユーザーの AccessKey Secret。取得方法については、[AccessKey の取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。      |

##### S3 プロトコル互換のオブジェクトストレージ

Hive Catalog はバージョン 2.5 から S3 プロトコル互換のオブジェクトストレージをサポートしています。

S3 プロトコル互換のオブジェクトストレージ（例：MinIO）を Hive クラスターのファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

`StorageCredentialParams` には以下のパラメータが含まれます。

| パラメータ                             | 必須   | 説明                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | はい      | SSL 接続を有効にするかどうか。<br />値の範囲：`true` または `false`。デフォルト値：`true`。 |
| aws.s3.enable_path_style_access  | はい      | パススタイルアクセス（Path-Style Access）を有効にするかどうか。<br />値の範囲：`true` または `false`。デフォルト値：`false`。MinIO の場合は `true` に設定する必要があります。<br />パススタイル URL は以下の形式で使用されます：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部（オレゴン）リージョンに `DOC-EXAMPLE-BUCKET1` というバケットを作成し、その中の `alice.jpg` オブジェクトにアクセスしたい場合、以下のパススタイル URL を使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。|
| aws.s3.endpoint                  | はい      | S3 プロトコル互換のオブジェクトストレージにアクセスするためのエンドポイント。 |
| aws.s3.access_key                | はい      | IAM ユーザーのアクセスキー。 |
| aws.s3.secret_key                | はい      | IAM ユーザーのシークレットキー。 |

##### Microsoft Azure ストレージ

Hive Catalog はバージョン 3.0 から Microsoft Azure Storage をサポートしています。

###### Azure Blob Storage

Azure Blob Storage を Hive クラスターのファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- Shared Key を使用した認証と認可

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | パラメータ                   | 必須 | 説明                         |
  | -------------------------- | ------------ | -------------------------------- |
  | azure.blob.storage_account | はい           | Blob Storage アカウントのユーザー名。      |
  | azure.blob.shared_key      | はい           | Blob Storage アカウントの Shared Key。 |

- SAS Token を使用した認証と認可

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | パラメータ                  | 必須 | 説明                                 |
  | ------------------------- | ------------ | ---------------------------------------- |
  | azure.blob.storage_account| はい           | Blob Storage アカウントのユーザー名。              |
  | azure.blob.container      | はい           | Blob コンテナの名前。               |
  | azure.blob.sas_token      | はい           | Blob Storage アカウントへのアクセスに使用する SAS Token。 |

###### Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 を Hive クラスターのファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- Managed Service Identity を使用した認証と認可

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | パラメータ                                 | 必須 | 説明                                                     |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい           | Managed Service Identity 認証方法を有効にするかどうか。`true` に設定します。 |

- Service Principal を使用した認証と認可

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | パラメータ                 | 必須 | 説明                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい           | Service Principal のクライアント（アプリケーション）ID。               |
  | azure.adls1.oauth2_credential | はい           | 新規作成されたクライアント（アプリケーション）シークレット。                         |
  | azure.adls1.oauth2_endpoint   | はい           | Service Principal またはアプリケーションの OAuth 2.0 トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2

Azure Data Lake Storage Gen2 を Hive クラスターのファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- Managed Identity を使用した認証と認可

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | パラメータ                                | 必須 | 説明                                                |
  | --------------------------------------- | ------------ | ------------------------------------------------------- |
  | azure.adls2.oauth2_use_managed_identity | はい           | Managed Identity 認証方法を有効にするかどうか。`true` に設定します。 |
  | azure.adls2.oauth2_tenant_id            | はい           | テナントの ID。                                 |
  | azure.adls2.oauth2_client_id            | はい           | Managed Identity のクライアント（アプリケーション）ID。           |

- Shared Key を使用した認証と認可

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | パラメータ                    | 必須 | 説明                                   |
  | --------------------------- | ------------ | ------------------------------------------ |
  | azure.adls2.storage_account | はい           | Data Lake Storage Gen2 アカウントのユーザー名。      |
  | azure.adls2.shared_key      | はい           | Data Lake Storage Gen2 アカウントの Shared Key。 |

- Service Principal を使用した認証と認可

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | パラメータ                           | 必須 | 説明                                                     |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい           | Service Principal のクライアント（アプリケーション）ID。               |
  | azure.adls2.oauth2_client_secret   | はい           | 新規作成されたクライアント（アプリケーション）シークレット。                         |

  | azure.adls2.oauth2_client_endpoint | はい           | Service Principal または Application の OAuth 2.0 Token Endpoint (v1)。 |

##### Google GCS

Hive Catalog はバージョン 3.0 から Google GCS をサポートしています。

Google GCS を Hive クラスターのファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- VM を使用した認証と認可

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                                   | **デフォルト値** | **例** | **説明**                                                 |
  | ------------------------------------------ | ---------- | ------------ | -------------------------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | false      | true         | Compute Engine に紐付けられた Service Account を直接使用するかどうか。 |

- Service Account を使用した認証と認可

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                               | **デフォルト値** | **例**                                                 | **説明**                                                     |
  | -------------------------------------- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""         | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | Service Account を作成した際に生成された JSON ファイル内の Email。          |
  | gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | Service Account を作成した際に生成された JSON ファイル内の Private Key ID。 |
  | gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | Service Account を作成した際に生成された JSON ファイル内の Private Key。    |

- Impersonation を使用した認証と認可

  - VM インスタンスで Service Account を模倣する場合

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | **パラメータ**                                   | **デフォルト値** | **例** | **説明**                                                     |
    | ------------------------------------------ | ---------- | ------------ | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false      | true         | Compute Engine に紐付けられた Service Account を直接使用するかどうか。     |
    | gcp.gcs.impersonation_service_account      | ""         | "hello"      | 模倣する対象の Service Account。 |

  - ある Service Account（仮に "Meta Service Account" と呼ぶ）が別の Service Account（仮に "Data Service Account" と呼ぶ）を模倣する場合

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | **パラメータ**                               | **デフォルト値** | **例**                                                 | **説明**                                                     |
    | -------------------------------------- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""         | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | Meta Service Account を作成した際に生成された JSON ファイル内の Email。     |
    | gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | Meta Service Account を作成した際に生成された JSON ファイル内の Private Key ID。 |
    | gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | Meta Service Account を作成した際に生成された JSON ファイル内の Private Key。 |
    | gcp.gcs.impersonation_service_account  | ""         | "hello"                                                      | 模倣する対象の Data Service Account。 |

#### MetadataUpdateParams

キャッシュされたメタデータの更新戦略を指定する一連のパラメータです。StarRocks はこの戦略に基づいて Hive のメタデータを更新します。このパラメータ群はオプションです。

StarRocks はデフォルトで[自動非同期更新戦略](#附录理解元数据自动异步更新策略)を採用しており、すぐに使用できます。そのため、通常は `MetadataUpdateParams` を無視して、戦略パラメータの調整は不要です。

しかし、Hive データの更新頻度が高い場合は、これらのパラメータを調整して、自動非同期更新戦略のパフォーマンスを最適化することができます。

| パラメータ                                   | 必須 | 説明                                                         |
| -------------------------------------- | -------- | ------------------------------------------------------------ |
| enable_metastore_cache            | いいえ       | StarRocks が Hive テーブルのメタデータをキャッシュするかどうかを指定します。値の範囲は `true` または `false`。デフォルト値は `true` です。`true` はキャッシュを有効にし、`false` はキャッシュを無効にします。|
| enable_remote_file_cache               | いいえ       | StarRocks が Hive テーブルまたはパーティションのデータファイルのメタデータをキャッシュするかどうかを指定します。値の範囲は `true` または `false`。デフォルト値は `true` です。`true` はキャッシュを有効にし、`false` はキャッシュを無効にします。|
| metastore_cache_refresh_interval_sec   | いいえ       | StarRocks がキャッシュされた Hive テーブルまたはパーティションのメタデータを非同期で更新する間隔。単位は秒。デフォルト値は `7200`、つまり 2 時間です。 |
| remote_file_cache_refresh_interval_sec | いいえ       | StarRocks がキャッシュされた Hive テーブルまたはパーティションのデータファイルのメタデータを非同期で更新する間隔。単位は秒。デフォルト値は `60` です。 |
| metastore_cache_ttl_sec                | いいえ       | StarRocks がキャッシュされた Hive テーブルまたはパーティションのメタデータを自動的に削除するまでの時間。単位は秒。デフォルト値は `86400`、つまり 24 時間です。 |
| remote_file_cache_ttl_sec              | いいえ       | StarRocks がキャッシュされた Hive テーブルまたはパーティションのデータファイルのメタデータを自動的に削除するまでの時間。単位は秒。デフォルト値は `129600`、つまり 36 時間です。 |
| enable_cache_list_names                | いいえ       | StarRocks が Hive パーティション名をキャッシュするかどうかを指定します。値の範囲は `true` または `false`。デフォルト値は `true` です。`true` はキャッシュを有効にし、`false` はキャッシュを無効にします。|

### 例

以下の例では、Hive クラスタ内のデータをクエリするために `hive_catalog_hms` または `hive_catalog_glue` という名前の Hive Catalog を作成しています。

#### HDFS

HDFS をストレージとして使用する場合は、以下のように Hive Catalog を作成できます：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

#### AWS S3

##### Instance Profile を使用した認証と認可の場合

- Hive クラスタが HMS をメタデータサービスとして使用している場合は、以下のように Hive Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Hive クラスタが AWS Glue をメタデータサービスとして使用している場合は、以下のように Hive Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### Assumed Role を使用した認証と認可の場合

- Hive クラスタが HMS をメタデータサービスとして使用している場合は、以下のように Hive Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Hive クラスタが AWS Glue をメタデータサービスとして使用している場合は、以下のように Hive Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### IAM User を使用した認証と認可の場合

- Hive クラスタが HMS をメタデータサービスとして使用している場合は、以下のように Hive Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Hive クラスタが AWS Glue をメタデータサービスとして使用している場合は、以下のように Hive Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "false",
      "aws.glue.access_key" = "<iam_user_access_key>",
      "aws.glue.secret_key" = "<iam_user_secret_key>",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

#### S3 プロトコルに対応するオブジェクトストレージ

MinIO を例に、以下のように Hive Catalog を作成できます：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "aws.s3.enable_ssl" = "true",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```

#### Microsoft Azure ストレージ

##### Azure Blob Storage

- Shared Key を使用して認証および承認する場合、以下のように Hive カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SAS Token を使用して認証および承認する場合、以下のように Hive カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- Managed Service Identity を使用して認証および承認する場合、以下のように Hive カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- Service Principal を使用して認証および承認する場合、以下のように Hive カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- Managed Identity を使用して認証および承認する場合、以下のように Hive カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- Shared Key を使用して認証および承認する場合、以下のように Hive カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- Service Principal を使用して認証および承認する場合、以下のように Hive カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>" 
  );
  ```

#### Google GCS

- VM を使用して認証および承認する場合、以下のように Hive カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- Service Account を使用して認証および承認する場合、以下のように Hive カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- Impersonation を使用して認証および承認する場合

  - VM インスタンスで Service Account を模倣するには、以下のように Hive カタログを作成できます:

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - 一つの Service Account で別の Service Account を模倣するには、以下のように Hive カタログを作成できます:

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_private_key>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Hive カタログを表示する

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) を使用して、現在の StarRocks クラスター内のすべてのカタログを照会できます:

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を使用して、特定の External Catalog の作成ステートメントを照会できます。例えば、以下のコマンドで Hive Catalog `hive_catalog_glue` の作成ステートメントを照会できます:

```SQL
SHOW CREATE CATALOG hive_catalog_glue;
```

## Hive カタログとデータベースを切り替える

以下の方法で目的の Hive カタログとデータベースに切り替えることができます:

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) を使用して現在のセッションに有効な Hive カタログを指定し、その後 [USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用してデータベースを指定します:

  ```SQL
  -- 現在のセッションに有効なカタログを切り替えます:
  SET CATALOG <catalog_name>
  -- 現在のセッションに有効なデータベースを指定します:
  USE <db_name>
  ```

- [USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、セッションを直接目的の Hive カタログの指定されたデータベースに切り替えます:

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Hive カタログを削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) を使用して、特定の External Catalog を削除できます。

例えば、以下のコマンドで Hive Catalog `hive_catalog_glue` を削除できます:

```SQL
DROP CATALOG hive_catalog_glue;
```

## Hive テーブルの構造を表示する

以下の方法で Hive テーブルの構造を表示できます:

- テーブルの構造を表示する

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATE コマンドからテーブルの構造とファイルの保存場所を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Hive テーブルのデータを照会する

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を使用して、指定されたカタログに属する Hive クラスター内のデータベースを表示します:

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [目的の Hive カタログとデータベースに切り替える](#hive-カタログとデータベースを切り替える)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して、目的のデータベース内のテーブルからデータを照会します:

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hive データをインポートする

OLAP テーブルがあり、その名前が `olap_tbl` であるとします。以下のようにしてそのテーブルのデータを変換し、StarRocks にインポートすることができます:

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hive_table
```

## Hive テーブルとビューの権限を付与する

[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) を使用して、特定の Hive カタログ内のすべてのテーブルまたはビューに対するクエリ権限をロールに付与できます。

- 特定の Hive カタログ内のすべてのテーブルに対するクエリ権限をロールに付与する:

  ```SQL
  GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE <role_name>
  ```

- 特定の Hive カタログ内のすべてのビューに対するクエリ権限をロールに付与する:

  ```SQL
  GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE <role_name>
  ```

例えば、以下のコマンドでロール `hive_role_table` を作成し、Hive カタログ `hive_catalog` に切り替え、`hive_catalog` 内のすべてのテーブルとビューのクエリ権限を `hive_role_table` に付与できます:

```SQL
-- ロール hive_role_table を作成します。
CREATE ROLE hive_role_table;

-- データカタログ hive_catalog に切り替えます。
SET CATALOG hive_catalog;

-- hive_catalog 内のすべてのテーブルに対するクエリ権限を hive_role_table に付与します。
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE hive_role_table;

-- hive_catalog 内のすべてのビューに対するクエリ権限を hive_role_table に付与します。
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE hive_role_table;
```

## Hive データベースを作成する

StarRocks の内部データカタログ (Internal Catalog) と同様に、Hive カタログの [CREATE DATABASE](../../administration/privilege_item.md#データカタログ権限-catalog) 権限を持っている場合、[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) を使用してその Hive カタログ内にデータベースを作成できます。この機能はバージョン 3.2 からサポートされています。

> **注記**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) および [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザーやロールに対する権限を付与および取り消すことができます。

[目的の Hive カタログに切り替える](#hive-カタログとデータベースを切り替える) 、その後以下のステートメントで Hive データベースを作成します:

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location` パラメータはデータベースのファイルパスを指定するために使用され、HDFS およびオブジェクトストレージをサポートしています:


- HMSをメタデータサービスとして選択した場合、データベースの作成時に`location`を指定しないと、システムはHMSのデフォルトの`<warehouse_location>/<database_name.db>`をファイルパスとして使用します。
- AWS Glueをメタデータサービスとして選択した場合、`location`パラメータにはデフォルト値がないため、データベースを作成する際にこのパラメータを指定する必要があります。

`prefix`はストレージシステムによって異なります：

| **ストレージシステム**                           | **`Prefix`** **値**                                        |
| -------------------------------------- | ------------------------------------------------------------ |
| HDFS                                   | `hdfs`                                                       |
| Google GCS                             | `gs`                                                         |
| Azure Blob Storage                     | <ul><li>ストレージアカウントがHTTPプロトコルでアクセスをサポートする場合、`prefix`は`wasb`です。</li><li>ストレージアカウントがHTTPSプロトコルでアクセスをサポートする場合、`prefix`は`wasbs`です。</li></ul> |
| Azure Data Lake Storage Gen1           | `adl`                                                        |
| Azure Data Lake Storage Gen2           | <ul><li>ストレージアカウントがHTTPプロトコルでアクセスをサポートする場合、`prefix`は`abfs`です。</li><li>ストレージアカウントがHTTPSプロトコルでアクセスをサポートする場合、`prefix`は`abfss`です。</li></ul> |
| 阿里云 OSS                             | `oss`                                                        |
| 腾讯云 COS                             | `cosn`                                                       |
| 华为云 OBS                             | `obs`                                                        |
| AWS S3およびその他のS3互換ストレージ（例：MinIO） | `s3`                                                         |

## Hiveデータベースの削除

StarRocksの内部データベースと同様に、Hiveデータベースの[DROP](../../administration/privilege_item.md#数据库权限-database)権限を持っている場合、[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md)を使用してHiveデータベースを削除できます。この機能はバージョン3.2からサポートされています。空のデータベースのみ削除がサポートされています。

> **説明**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、ユーザーとロールに権限を付与および取り消すことができます。

データベースを削除しても、HDFSまたはオブジェクトストレージ上の対応するファイルパスは削除されません。

[目的のHive Catalogに切り替え](#切换-hive-catalog-和数据库)て、次のステートメントでHiveデータベースを削除します：

```SQL
DROP DATABASE <database_name>
```

## Hiveテーブルの作成

StarRocksの内部データベースと同様に、Hiveデータベースの[CREATE TABLE](../../administration/privilege_item.md#数据库权限-database)権限を持っている場合、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)または[CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)を使用して、そのHiveデータベースにManaged Tableを作成できます。この機能はバージョン3.2からサポートされています。

> **説明**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、ユーザーとロールに権限を付与および取り消すことができます。

[目的のHive Catalogとデータベースに切り替え](#切换-hive-catalog-和数据库)て、次の構文でHiveのManaged Tableを作成します：

### 構文

```SQL
CREATE TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...
partition_column_definition1,partition_column_definition2...])
[partition_desc]
[PROPERTIES ("key" = "value", ...)]
[AS SELECT query]
```

### パラメータ説明

#### column_definition

`column_definition`の構文は以下の通りです：

```SQL
col_name col_type [COMMENT 'comment']
```

パラメータ説明：

| パラメータ | 説明                                                         |
| -------- | ------------------------------------------------------------ |
| col_name | 列名。                                                       |
| col_type | 列のデータ型。現在サポートされているデータ型は：TINYINT、SMALLINT、INT、BIGINT、FLOAT、DOUBLE、DECIMAL、DATE、DATETIME、CHAR、VARCHAR[(length)]、ARRAY、MAP、STRUCTです。LARGEINT、HLL、BITMAP型はサポートされていません。 |

> **注意**
>
> すべての非パーティション列はデフォルトで`NULL`です（つまり、テーブル作成ステートメントで`DEFAULT "NULL"`を指定します）。パーティション列は最後に宣言する必要があり、`NULL`にはできません。

#### partition_desc

`partition_desc`の構文は以下の通りです：

```SQL
PARTITION BY (par_col1[, par_col2...])
```

現在、StarRocksはIdentity Transformsのみをサポートしています。つまり、各ユニークなパーティション値に対してパーティションが作成されます。

> **注意**
>
> パーティション列は最後に宣言する必要があり、FLOAT、DOUBLE、DECIMAL、DATETIME以外のデータ型をサポートし、`NULL`値はサポートされていません。また、`partition_desc`で宣言されたパーティション列の順序は、`column_definition`で定義された列の順序と一致している必要があります。

#### PROPERTIES

`PROPERTIES`内で`"key" = "value"`の形式でHiveテーブルの属性を宣言できます。

以下は一般的な属性です：

| **属性**          | **説明**                                                     |
| ----------------- | ------------------------------------------------------------ |
| location          | Managed Tableが存在するファイルパス。HMSをメタデータサービスとして使用する場合、`location`パラメータを指定する必要はありません。AWS Glueをメタデータサービスとして使用する場合：<ul><li>現在のデータベースを作成する際に`location`パラメータが指定されている場合、現在のデータベースでテーブルを作成する際に再度`location`パラメータを指定する必要はありません。StarRocksはデフォルトでテーブルを現在のデータベースのファイルパスに作成します。</li><li>現在のデータベースを作成する際に`location`パラメータが指定されていない場合、現在のデータベースでテーブルを作成する際に`location`パラメータを指定する必要があります。</li></ul> |
| file_format       | Managed Tableのファイル形式。現在はParquet形式のみをサポートしています。デフォルト値は`parquet`です。 |
| compression_codec | Managed Tableの圧縮形式。現在、SNAPPY、GZIP、ZSTD、LZ4をサポートしています。デフォルト値は`gzip`です。 |

### 例

1. `unpartition_tbl`という非パーティションテーブルを作成し、`id`と`score`の2つの列を含むようにします：

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. `partition_tbl_1`というパーティションテーブルを作成し、`action`、`id`、`dt`の3つの列を含み、`id`と`dt`をパーティション列として定義します：

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. 元のテーブル`partition_tbl_1`からデータをクエリし、その結果に基づいてパーティションテーブル`partition_tbl_2`を作成し、`id`と`dt`を`partition_tbl_2`のパーティション列として定義します：

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (id, dt)
   AS SELECT * from partition_tbl_1;
   ```

## Hiveテーブルへのデータ挿入

StarRocksの内部テーブルと同様に、Hiveテーブル（Managed TableまたはExternal Table）の[INSERT](../../administration/privilege_item.md#表权限-table)権限を持っている場合、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用してStarRocksテーブルのデータをHiveテーブルに書き込むことができます（現在はParquet形式のHiveテーブルへの書き込みのみをサポートしています）。この機能はバージョン3.2からサポートされています。External Tableへのデータ書き込み機能はデフォルトでオフになっていますが、[システム変数ENABLE_WRITE_HIVE_EXTERNAL_TABLE](../../reference/System_variable.md)を使用してオンにすることができます。

> **説明**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、ユーザーとロールに権限を付与および取り消すことができます。

[目的のHive Catalogとデータベースに切り替え](#切换-hive-catalog-和数据库)て、次の構文でStarRocksテーブルのデータをParquet形式のHiveテーブルに書き込みます：

### 構文

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータを書き込む。
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

> **注意**
>
> パーティション列は`NULL`にできないため、インポート時にパーティション列に値があることを確認する必要があります。

### パラメータ説明

| パラメータ   | 説明                                                         |
| ----------- | ------------------------------------------------------------ |
| INTO        | データをターゲットテーブルに追加します。                     |
| OVERWRITE   | データをターゲットテーブルに上書きします。                   |
| column_name | インポートするターゲット列。一つまたは複数の列を指定できます。複数の列を指定する場合は、カンマ(`,`)で区切る必要があります。指定された列はターゲットテーブルに存在する列でなければならず、パーティション列を含む必要があります。このパラメータはソーステーブルの列名と異なる場合がありますが、順序は一致している必要があります。このパラメータを指定しない場合、デフォルトでターゲットテーブルのすべての列にデータがインポートされます。ソーステーブルの非パーティション列がターゲット列に存在しない場合は、デフォルト値`NULL`が書き込まれます。クエリ結果の列の型がターゲット列の型と一致しない場合、暗黙の型変換が行われます。変換できない場合は、INSERT INTOステートメントで構文解析エラーが発生します。 |
| expression  | 対応する列に値を割り当てるための式。                         |
| DEFAULT     | 対応する列にデフォルト値を割り当てます。                     |
| query       | クエリステートメント。クエリの結果はターゲットテーブルにインポートされます。クエリステートメントは、StarRocksがサポートする任意のSQLクエリ構文を使用できます。 |
| PARTITION   | インポートするターゲットパーティション。ターゲットテーブルのすべてのパーティション列を指定する必要があります。指定されたパーティション列の順序は、テーブル作成時に定義されたパーティション列の順序と異なっても構いません。パーティションを指定する場合、列名(`column_name`)を使用してインポートするターゲット列を指定することはできません。 |

### 例

1. `partition_tbl_1`テーブルに次の3行のデータを挿入します：

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. `partition_tbl_1`テーブルに、指定された列の順序で、簡単な計算を含むSELECTクエリの結果データを挿入します：

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. `partition_tbl_1`テーブルに、自身からデータを読み取るSELECTクエリの結果データを挿入します：

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. `partition_tbl_2`テーブルの`dt='2023-09-01'`、`id=1`のパーティションにSELECTクエリの結果データを挿入します：

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   または

   ```SQL

```
   INSERT INTO partition_tbl_2 partition(dt='2023-09-01',id=1) SELECT 'order';
   ```

5. `partition_tbl_1` テーブルの `dt='2023-09-01'`、`id=1` のパーティションの全ての `action` 列の値を `close` で上書きする:

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT OVERWRITE partition_tbl_1 partition(dt='2023-09-01',id=1) SELECT 'close';
   ```

## Hive テーブルの削除

StarRocks 内のテーブルと同様に、Hive テーブルを削除するには、Hive テーブルの [DROP](../../administration/privilege_item.md#表权限-table) 権限が必要です。この機能は、バージョン3.2からサポートされています。現在、Hive の Managed Table のみがサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) および [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザーとロールに権限を付与および取り消すことができます。

テーブルを削除する場合は、DROP TABLE ステートメントで `FORCE` キーワードを指定する必要があります。この操作は、テーブルに関連するファイルパスを削除しませんが、HDFS やオブジェクトストレージ上のテーブルデータは削除されます。この操作を慎重に実行してください。

[目的の Hive カタログとデータベースに切り替え](#切り替え-hive-catalog-とデータベース)し、次のステートメントで Hive テーブルを削除します:

```SQL
DROP TABLE <table_name> FORCE
```

## メタデータキャッシュの手動または自動更新

### 手動更新

デフォルトでは、StarRocks は Hive のメタデータをキャッシュし、非同期モードでキャッシュされたメタデータを自動的に更新してクエリのパフォーマンスを向上させます。また、Hive テーブルのスキーマ変更やその他のテーブルの更新が行われた場合、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) を使用してテーブルのメタデータを手動で更新し、StarRocks が適切なクエリプランを生成することを保証することもできます。

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

以下の場合にメタデータを手動で更新する必要があります：

- 既存のパーティション内のデータファイルが変更された場合、例：`INSERT OVERWRITE ... PARTITION ...` コマンドを実行した場合。
- Hive テーブルのスキーマが変更された場合。
- Hive テーブルが削除され、同じ名前の Hive テーブルが再作成された場合。
- Hive カタログの作成時に `PROPERTIES` で `"enable_cache_list_names" = "true"` を指定した場合。Hive 側で新しいパーティションが追加された場合、追加されたパーティションをクエリする必要があります。
  > **注意**
  >
  > StarRocks 2.5.5 以降、Hive メタデータキャッシュの定期的なリフレッシュがサポートされています。詳細については、後述の「[メタデータキャッシュの定期的なリフレッシュ](#メタデータキャッシュの定期的なリフレッシュ)」を参照してください。Hive メタデータキャッシュの定期的なリフレッシュ機能を有効にすると、デフォルトでは StarRocks は Hive メタデータキャッシュを 10 分ごとにリフレッシュします。したがって、通常は手動で更新する必要はありません。新しいパーティションをクエリする必要がある場合にのみ、手動で更新する必要があります。

注意: REFRESH EXTERNAL TABLE は、FE にキャッシュされているテーブルとパーティションのみを更新します。

### 自動増分更新

自動非同期更新ポリシーとは異なり、自動増分更新ポリシーでは、FE は定期的に HMS からさまざまなイベントを読み取り、Hive テーブルのメタデータの変更（列の追加/削除、パーティションの追加/削除、パーティションデータの更新など）を自動的に検知します。Hive テーブルのメタデータを手動で更新する必要はありません。
この機能は、HMS に大きな負荷をかけるため、注意して使用してください。データの変更を検知するために、[メタデータキャッシュの定期的なリフレッシュ](#メタデータキャッシュの定期的なリフレッシュ)を使用することをお勧めします。

自動増分更新ポリシーを有効にする手順は以下の通りです：

#### ステップ 1: HMS でイベントリスナーを設定する

HMS 2.x および 3.x のバージョンは、イベントリスナーの設定をサポートしています。ここでは、HMS 3.1.2 のイベントリスナー設定を例に説明します。以下の設定を **$HiveMetastore/conf/hive-site.xml** ファイルに追加し、HMS を再起動します。

```XML
<property>
    <name>hive.metastore.event.db.notification.api.auth</name>
    <value>false</value>
</property>
<property>
    <name>hive.metastore.notifications.add.thrift.objects</name>
    <value>true</value>
</property>
<property>
    <name>hive.metastore.alter.notifications.basic</name>
    <value>false</value>
</property>
<property>
    <name>hive.metastore.dml.events</name>
    <value>true</value>
</property>
<property>
    <name>hive.metastore.transactional.event.listeners</name>
    <value>org.apache.hive.hcatalog.listener.DbNotificationListener</value>
</property>
<property>
    <name>hive.metastore.event.db.listener.timetolive</name>
    <value>172800s</value>
</property>
<property>
    <name>hive.metastore.server.max.message.size</name>
    <value>858993459</value>
</property>
```

設定が完了したら、FE のログファイルで `event id` を検索し、イベント ID を確認してイベントリスナーが正しく設定されているかどうかを確認できます。設定に失敗した場合、すべての `event id` は `0` になります。

#### ステップ 2: StarRocks で自動増分更新ポリシーを有効にする

StarRocks クラスター内の特定の Hive カタログに対して自動増分更新ポリシーを有効にすることもできますし、StarRocks クラスター内のすべての Hive カタログに対して自動増分更新ポリシーを有効にすることもできます。

- 個々の Hive カタログに自動増分更新ポリシーを有効にする場合は、Hive カタログを作成する際に `PROPERTIES` の `enable_hms_events_incremental_sync` パラメータを `true` に設定します。以下に例を示します:

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```
  
- StarRocks クラスター内のすべての Hive カタログに自動増分更新ポリシーを有効にする場合は、各 FE の **$FE_HOME/conf/fe.conf** ファイルに `enable_hms_events_incremental_sync` パラメータを追加し、`true` に設定し、FE を再起動してパラメータの設定を有効にします。

また、各 FE の **$FE_HOME/conf/fe.conf** ファイルで以下のパラメータを調整し、FE を再起動してパラメータの設定を有効にすることもできます。

| パラメーター                         | 説明                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocks が HMS からイベントを読み取る間隔。デフォルト値: `5000`。単位: ミリ秒。 |
| hms_events_batch_size_per_rpc     | StarRocks が一度に読み取るイベントの最大数。デフォルト値: `500`。            |
| enable_hms_parallel_process_evens | StarRocks がイベントを読み取る際に並列処理を行うかどうかを指定します。値の範囲: `true` および `false`。デフォルト値: `true`。`true` の場合、並列処理が有効になります。`false` の場合、並列処理が無効になります。 |
| hms_process_events_parallel_num   | StarRocks が一度に処理するイベントの最大並行数。デフォルト値: `4`。            |

## 付録: メタデータの自動非同期更新ポリシーの理解

自動非同期更新ポリシーは、StarRocks が Hive カタログのメタデータを更新するためのデフォルトのポリシーです。

デフォルトでは（つまり、`enable_metastore_cache` パラメータと `enable_remote_file_cache` パラメータがいずれも `true` に設定されている場合）、クエリが Hive テーブルのパーティションにヒットすると、StarRocks はそのパーティションのメタデータとデータファイルのメタデータを自動的にキャッシュします。キャッシュされたメタデータは遅延更新（Lazy Update）ポリシーを採用しています。

例えば、`table2` という名前の Hive テーブルがあり、そのテーブルのデータが `p1`、`p2`、`p3`、`p4` の4つのパーティションに分散しているとします。クエリが `p1` にヒットすると、StarRocks は `p1` のメタデータと `p1` のデータファイルのメタデータを自動的にキャッシュします。現在のキャッシュメタデータの更新および削除ポリシーは以下のように設定されています：

- `p1` のキャッシュメタデータの非同期更新間隔（`metastore_cache_refresh_interval_sec` パラメータで指定）は2時間です。
- `p1` のデータファイルのキャッシュメタデータの非同期更新間隔（`remote_file_cache_refresh_interval_sec` パラメータで指定）は60秒です。
- `p1` のキャッシュメタデータの自動削除間隔（`metastore_cache_ttl_sec` パラメータで指定）は24時間です。
- `p1` のデータファイルのキャッシュメタデータの自動削除間隔（`remote_file_cache_ttl_sec` パラメータで指定）は36時間です。

以下の図を参照してください。


![タイムライン上のポリシー更新](../../assets/catalog_timeline_zh.png)

StarRocksは以下の方針でキャッシュされたメタデータを更新および廃棄します：

- 別のクエリが`p1`を再ヒットし、現在の時刻が最後の更新から60秒以内である場合、StarRocksは`p1`のキャッシュされたメタデータも`p1`のデータファイルのキャッシュされたメタデータも更新しません。
- 別のクエリが`p1`を再ヒットし、現在の時刻が最後の更新から60秒を超える場合、StarRocksは`p1`のデータファイルのキャッシュされたメタデータを更新します。
- 別のクエリが`p1`を再ヒットし、現在の時刻が最後の更新から2時間を超える場合、StarRocksは`p1`のキャッシュされたメタデータを更新します。
- 最後の更新が終了してから24時間以内に`p1`がアクセスされなかった場合、StarRocksは`p1`のキャッシュされたメタデータを廃棄します。その後、クエリが`p1`を再ヒットすると、`p1`のメタデータが再度キャッシュされます。
- 最後の更新が終了してから36時間以内に`p1`がアクセスされなかった場合、StarRocksは`p1`のデータファイルのキャッシュされたメタデータを廃棄します。その後、クエリが`p1`を再ヒットすると、`p1`のデータファイルのメタデータが再度キャッシュされます。
