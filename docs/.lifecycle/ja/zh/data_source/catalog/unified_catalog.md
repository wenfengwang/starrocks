---
displayed_sidebar: Chinese
---

# 統合カタログ

Unified Catalog は External Catalog の一種で、バージョン 3.2 からサポートされています。Unified Catalog を使用すると、Apache Hive™、Apache Iceberg、Apache Hudi、Delta Lake などの複数のデータソースを統合データソースとして扱い、テーブルデータをインポートすることなく直接操作できます。これには以下が含まれます:

- 手動でテーブルを作成する必要がなく、Unified Catalog を通じて Hive、Iceberg、Hudi、Delta Lake などのデータソース内のデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) や非同期マテリアライズドビュー（バージョン 2.5 以上）を使用して、Hive、Iceberg、Hudi、Delta Lake などのデータソース内のデータを加工し、StarRocks にインポートします。
- StarRocks 側で Hive、Iceberg のデータベースやテーブルを作成または削除します。

統合データソース内のデータに正常にアクセスするために、StarRocks クラスターは以下の2つの重要なコンポーネントを統合する必要があります:

- 分散ファイルシステム（HDFS）またはオブジェクトストレージ。現在サポートされているオブジェクトストレージには、AWS S3、Microsoft Azure Storage、Google GCS、その他の S3 プロトコル互換のオブジェクトストレージ（例：アリババクラウド OSS、MinIO）があります。

- メタデータサービス。現在サポートされているメタデータサービスには、Hive Metastore（以下、HMS）、AWS Glue があります。

  > **説明**
  >
  > AWS S3 をストレージシステムとして選択した場合、メタデータサービスとして HMS または AWS Glue を選択できます。他のストレージシステムを選択した場合は、HMS のみをメタデータサービスとして選択できます。

## 使用制限

Unified Catalog は現在、一つのストレージシステムと一つのメタデータサービスのみをサポートしています。したがって、Unified Catalog を通じてアクセスするすべてのデータソースが同じストレージシステムとメタデータサービスを使用していることを確認する必要があります。

## 使用説明

- Unified Catalog がサポートするファイル形式とデータタイプについては、[Hive catalog](../catalog/hive_catalog.md)、[Iceberg catalog](../catalog/iceberg_catalog.md)、[Hudi catalog](../catalog/hudi_catalog.md)、[Delta Lake catalog](../catalog/deltalake_catalog.md) のドキュメント内「使用説明」セクションを参照してください。

- 一部の操作は特定のテーブル形式にのみ使用できます。例えば、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) と [DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) は現在 Hive と Iceberg のテーブルのみをサポートしており、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) は Hive と Hudi のテーブルのみをサポートしています。

  Unified Catalog 内で [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) を使用してテーブルを作成する場合、`ENGINE` パラメータを使用してテーブル形式（Hive または Iceberg）を指定する必要があります。

## 準備作業

Unified Catalog を作成する前に、StarRocks クラスターが使用するファイルストレージおよびメタデータサービスに正常にアクセスできることを確認してください。

### AWS IAM

AWS S3 をファイルストレージとして使用する場合、または AWS Glue をメタデータサービスとして使用する場合、StarRocks クラスターが関連する AWS クラウドリソースにアクセスできるように、適切な認証および認可スキームを選択する必要があります。StarRocks が AWS にアクセスするための認証および認可に関する詳細は、[AWS 認証方法の設定 - 準備作業](../../integrations/authenticate_to_aws_resources.md#準備作業)を参照してください。

### HDFS

HDFS をファイルストレージとして使用する場合、StarRocks クラスターに以下の設定が必要です:

- （オプション）HDFS クラスターおよび HMS にアクセスするためのユーザー名を設定します。各 FE の **fe/conf/hadoop_env.sh** ファイルおよび各 BE の **be/conf/hadoop_env.sh** ファイルの最初に `export HADOOP_USER_NAME="<user_name>"` を追加してユーザー名を設定できます。設定後、各 FE および BE を再起動して設定を有効にします。このユーザー名を設定しない場合、デフォルトでは FE および BE プロセスのユーザー名が使用されます。各 StarRocks クラスターは一つのユーザー名のみを設定できます。
- データをクエリする際、StarRocks クラスターの FE および BE は HDFS クライアントを通じて HDFS クラスターにアクセスします。通常、StarRocks はデフォルトの設定で HDFS クライアントを起動し、手動での設定は必要ありません。ただし、以下のシナリオでは手動での設定が必要です：
  - HDFS クラスターが HA（High Availability）モードを有効にしている場合、HDFS クラスターの **hdfs-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに配置する必要があります。
  - HDFS クラスターが ViewFs を設定している場合、HDFS クラスターの **core-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに配置する必要があります。

> **注意**
>
> クエリ実行時にドメイン名が認識できない（Unknown Host）ためにアクセスに失敗する場合は、HDFS クラスターの各ノードのホスト名と IP アドレスのマッピングを **/etc/hosts** に設定する必要があります。

### Kerberos 認証

HDFS クラスターまたは HMS が Kerberos 認証を有効にしている場合、StarRocks クラスターに以下の設定が必要です:

- 各 FE および各 BE で `kinit -kt keytab_path principal` コマンドを実行し、Key Distribution Center (KDC) から Ticket Granting Ticket (TGT) を取得します。コマンドを実行するユーザーは HMS および HDFS にアクセスする権限を持っている必要があります。このコマンドを使用して KDC にアクセスすることには有効期限があるため、cron を使用して定期的に実行する必要があります。
- 各 FE の **$FE_HOME/conf/fe.conf** および各 BE の **$BE_HOME/conf/be.conf** に `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。ここで、`/etc/krb5.conf` は krb5.conf ファイルのパスであり、実際のファイルのパスに応じて変更できます。

## Unified Catalog の作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "unified",
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### パラメータ説明

#### catalog_name

Unified Catalog の名前。命名規則は以下の通りです：

- 英字 (a-z または A-Z)、数字 (0-9)、アンダースコア (_) のみを含み、英字で始まる必要があります。
- 全体の長さは 1023 文字を超えてはいけません。
- Catalog 名は大文字小文字を区別します。

#### コメント

Unified Catalog の説明。このパラメータはオプションです。

#### type

データソースのタイプ。`unified` に設定します。

#### MetastoreParams

StarRocks がメタデータサービスにアクセスするためのパラメータ設定。

##### Hive metastore

HMS をメタデータサービスとして選択する場合、以下のように `MetastoreParams` を設定してください：

```SQL
"unified.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **説明**
>
> データをクエリする前に、すべての HMS ノードのホスト名と IP アドレスのマッピングを **/etc/hosts** に追加する必要があります。そうしないと、クエリを実行する際に StarRocks が HMS にアクセスできない可能性があります。

`MetastoreParams` には以下のパラメータが含まれます。

| パラメータ                   | 必須 | 説明                                                         |
| ---------------------- | ---- | ------------------------------------------------------------ |
| unified.metastore.type | はい  | メタデータサービスのタイプ。`hive` に設定します。            |
| hive.metastore.uris    | はい  | HMS の URI。形式：`thrift://<HMS IP アドレス>:<HMS ポート番号>`。HMS が HA モードを有効にしている場合、複数の HMS アドレスをコンマ (`,`) で区切って指定できます。例：`"thrift://<HMS IP アドレス 1>:<HMS ポート番号 1>,thrift://<HMS IP アドレス 2>:<HMS ポート番号 2>,thrift://<HMS IP アドレス 3>:<HMS ポート番号 3>"`。 |

##### AWS Glue

AWS Glue をメタデータサービスとして選択する場合（AWS S3 をストレージシステムとして使用している場合のみサポートされます）、以下のように `MetastoreParams` を設定してください：

- Instance Profile を使用した認証と認可

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- Assumed Role を使用した認証と認可

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAM User を使用した認証と認可

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

`MetastoreParams` には以下のパラメータが含まれます。

| パラメータ                          | 必須 | 説明                                                         |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| unified.metastore.type        | はい  | メタデータサービスのタイプ。`glue` に設定します。            |
| aws.glue.use_instance_profile | はい  | Instance Profile と Assumed Role の認証方式を指定します。値は `true` または `false`。デフォルトは `false` です。 |

| aws.glue.iam_role_arn         | 否       | AWS Glue Data Catalog にアクセスする権限を持つ IAM Role の ARN。Assume Role 認証方式で AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.region               | はい       | AWS Glue Data Catalog が存在するリージョン。例：`us-west-1`。        |
| aws.glue.access_key           | 否       | IAM User の Access Key。IAM User 認証方式で AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | 否       | IAM User の Secret Key。IAM User 認証方式で AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |

AWS Glue へのアクセスに使用する認証方式の選択方法および AWS IAM コンソールでのアクセス制御ポリシーの設定方法については、[AWS Glue への認証パラメータ](../../integrations/authenticate_to_aws_resources.md#aws-glue-への認証パラメータ)を参照してください。

#### StorageCredentialParams

StarRocks がファイルストレージにアクセスするためのパラメータ設定です。

HDFS をストレージシステムとして使用する場合は、`StorageCredentialParams` を設定する必要はありません。

AWS S3、アリババクラウド OSS、S3 プロトコル互換のオブジェクトストレージ、Microsoft Azure Storage、または GCS を使用する場合は、`StorageCredentialParams` を設定する必要があります。

##### AWS S3

AWS S3 をファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- Instance Profile を使用した認証と認可

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- Assume Role を使用した認証と認可

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
| aws.s3.use_instance_profile | はい       | Instance Profile と Assume Role の二つの認証方式を使用するかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`false`。 |
| aws.s3.iam_role_arn         | 否       | AWS S3 Bucket にアクセスする権限を持つ IAM Role の ARN。Assume Role 認証方式で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい       | AWS S3 Bucket が存在するリージョン。例：`us-west-1`。                |
| aws.s3.access_key           | 否       | IAM User の Access Key。IAM User 認証方式で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | 否       | IAM User の Secret Key。IAM User 認証方式で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3 へのアクセスに使用する認証方式の選択方法および AWS IAM コンソールでのアクセス制御ポリシーの設定方法については、[AWS S3 への認証パラメータ](../../integrations/authenticate_to_aws_resources.md#aws-s3-への認証パラメータ)を参照してください。

##### アリババクラウド OSS

アリババクラウド OSS をファイルストレージとして選択する場合は、`StorageCredentialParams` に以下の認証パラメータを設定する必要があります：

```SQL
"aliyun.oss.access_key" = "<user_access_key>",
"aliyun.oss.secret_key" = "<user_secret_key>",
"aliyun.oss.endpoint" = "<oss_endpoint>" 
```

| パラメータ                            | 必須 | 説明                                                         |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aliyun.oss.endpoint             | はい      | アリババクラウド OSS のエンドポイント。例：`oss-cn-beijing.aliyuncs.com`。エンドポイントとリージョンの関係を調べるには、[アクセスドメインとデータセンター](https://help.aliyun.com/document_detail/31837.html)を参照してください。 |
| aliyun.oss.access_key           | はい      | アリババクラウドアカウントまたは RAM ユーザーの AccessKey ID。取得方法については、[AccessKey の取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。 |
| aliyun.oss.secret_key           | はい      | アリババクラウドアカウントまたは RAM ユーザーの AccessKey Secret。取得方法については、[AccessKey の取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。 |

##### S3 プロトコル互換のオブジェクトストレージ

S3 プロトコル互換のオブジェクトストレージ（例：MinIO）をファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

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
| aws.s3.enable_path_style_access  | はい      | パススタイルアクセス（Path-Style Access）を有効にするかどうか。<br />値の範囲：`true` または `false`。デフォルト値：`false`。MinIO では `true` に設定する必要があります。<br />パススタイル URL は以下の形式を使用します：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部（オレゴン）リージョンに `DOC-EXAMPLE-BUCKET1` というバケットを作成し、その中の `alice.jpg` オブジェクトにアクセスしたい場合は、以下のパススタイル URL を使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。|
| aws.s3.endpoint                  | はい      | S3 プロトコル互換のオブジェクトストレージにアクセスするためのエンドポイント。 |
| aws.s3.access_key                | はい      | IAM User の Access Key。 |
| aws.s3.secret_key                | はい      | IAM User の Secret Key。 |

##### Microsoft Azure ストレージ

###### Azure Blob Storage

Azure Blob Storage をファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

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
  | azure.blob.container      | はい           | データが存在する Blob コンテナの名前。               |
  | azure.blob.sas_token      | はい           | Blob Storage アカウントにアクセスするための SAS Token。 |

###### Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 をファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- Managed Service Identity を使用した認証と認可

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | パラメータ                                 | 必須 | 説明                                                     |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい           | Managed Service Identity 認証方式を有効にするかどうかを指定します。`true` に設定します。 |

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

Azure Data Lake Storage Gen2 をファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- Managed Identity を使用した認証と認可

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                          | **必須** | **説明**                                                |
  | --------------------------------------- | -------- | ------------------------------------------------------- |
  | azure.adls2.oauth2_use_managed_identity | はい     | Managed Identity 認証方式を使用するかどうかを指定します。`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | はい     | データが属するテナントのIDです。                         |
  | azure.adls2.oauth2_client_id            | はい     | Managed Identity のクライアント（アプリケーション）IDです。 |

- Shared Key を使用した認証と認可

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                      | **必須** | **説明**                                       |
  | --------------------------- | -------- | ------------------------------------------ |
  | azure.adls2.storage_account | はい     | Data Lake Storage Gen2 アカウントのユーザー名です。 |
  | azure.adls2.shared_key      | はい     | Data Lake Storage Gen2 アカウントのShared Keyです。 |

- Service Principal を使用した認証と認可

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                           | **必須** | **説明**                                                         |
  | ---------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい     | Service Principal のクライアント（アプリケーション）IDです。               |
  | azure.adls2.oauth2_client_secret   | はい     | 新しく作成されたクライアント（アプリケーション）シークレットです。                         |
  | azure.adls2.oauth2_client_endpoint | はい     | Service Principal またはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

##### Google GCS

Google GCS をファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- VM を使用した認証と認可

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                                   | **デフォルト値** | **値の例** | **説明**                                                 |
  | ------------------------------------------ | ------------ | ---------- | -------------------------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | false        | true       | Compute Engine に紐づけられたService Accountを直接使用するかどうか。 |

- Service Account を使用した認証と認可

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                               | **デフォルト値** | **値の例**                                                 | **説明**                                                     |
  | -------------------------------------- | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""           | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | Service Account を作成した際に生成されたJSONファイル内のEmailです。 |
  | gcp.gcs.service_account_private_key_id | ""           | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | Service Account を作成した際に生成されたJSONファイル内のPrivate Key IDです。 |
  | gcp.gcs.service_account_private_key    | ""           | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | Service Account を作成した際に生成されたJSONファイル内のPrivate Keyです。 |

- Impersonation を使用した認証と認可

  - VM インスタンスでService Accountを模倣

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | **パラメータ**                                   | **デフォルト値** | **値の例** | **説明**                                                     |
    | ------------------------------------------ | ------------ | ---------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false        | true       | Compute Engine に紐づけられたService Accountを直接使用するかどうか。     |
    | gcp.gcs.impersonation_service_account      | ""           | "hello"    | 模倣する対象のService Accountです。 |

  - あるService Account（仮に「Meta Service Account」とします）で別のService Account（仮に「Data Service Account」とします）を模倣

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | **パラメータ**                               | **デフォルト値** | **値の例**                                                 | **説明**                                                     |
    | -------------------------------------- | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""           | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | Meta Service Account を作成した際に生成されたJSONファイル内のEmailです。 |
    | gcp.gcs.service_account_private_key_id | ""           | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | Meta Service Account を作成した際に生成されたJSONファイル内のPrivate Key IDです。 |
    | gcp.gcs.service_account_private_key    | ""           | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | Meta Service Account を作成した際に生成されたJSONファイル内のPrivate Keyです。 |
    | gcp.gcs.impersonation_service_account  | ""           | "hello"                                                      | 模倣する対象のData Service Accountです。 |

#### MetadataUpdateParams（メタデータ更新パラメータ）

キャッシュされたメタデータの更新戦略を指定するパラメータのセットです。このグループのパラメータはオプションです。StarRocksは、この戦略に基づいてキャッシュされたHive、Hudi、Delta Lakeのメタデータを更新します。Hive、Hudi、Delta Lakeのメタデータキャッシュ更新の詳細については、[Hive catalog](../../data_source/catalog/hive_catalog.md)、[Hudi catalog](../../data_source/catalog/hudi_catalog.md)、[Delta Lake catalog](../../data_source/catalog/deltalake_catalog.md)を参照してください。

StarRocksはデフォルトで自動非同期更新戦略を採用しており、すぐに使用できます。したがって、通常は `MetadataUpdateParams` を無視し、戦略パラメータのチューニングは不要です。

Hive、Hudi、Delta Lakeのデータ更新頻度が高い場合は、これらのパラメータをチューニングして、自動非同期更新戦略のパフォーマンスを最適化できます。

| パラメータ                                 | 必須 | 説明                                                         |
| -------------------------------------- | ---- | ------------------------------------------------------------ |
| enable_metastore_cache                 | いいえ | StarRocksがHive、Hudi、Delta Lakeのテーブルメタデータをキャッシュするかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`true`。`true` はキャッシュを有効にし、`false` はキャッシュを無効にします。 |
| enable_remote_file_cache               | いいえ | StarRocksがHive、Hudi、Delta Lakeのテーブルまたはパーティションのデータファイルのメタデータをキャッシュするかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`true`。`true` はキャッシュを有効にし、`false` はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | いいえ | StarRocksがキャッシュされたHive、Hudi、Delta Lakeのテーブルまたはパーティションのメタデータを非同期で更新する時間間隔。単位：秒。デフォルト値：`7200`、つまり2時間。 |
| remote_file_cache_refresh_interval_sec | いいえ | StarRocksがキャッシュされたHive、Hudi、Delta Lakeのテーブルまたはパーティションのデータファイルのメタデータを非同期で更新する時間間隔。単位：秒。デフォルト値：`60`。 |
| metastore_cache_ttl_sec                | いいえ | StarRocksがキャッシュされたHive、Hudi、Delta Lakeのテーブルまたはパーティションのメタデータを自動的に削除する時間間隔。単位：秒。デフォルト値：`86400`、つまり24時間。 |
| remote_file_cache_ttl_sec              | いいえ | StarRocksがキャッシュされたHive、Hudi、Delta Lakeのテーブルまたはパーティションのデータファイルのメタデータを自動的に削除する時間間隔。単位：秒。デフォルト値：`129600`、つまり36時間。 |

### 例

以下の例では、統合データソース内のデータをクエリするために、`unified_catalog_hms` または `unified_catalog_glue` という名前のUnified Catalogを作成しています。

#### HDFS

HDFSをストレージとして使用する場合、以下のようにUnified Catalogを作成できます：

```SQL
CREATE EXTERNAL CATALOG unified_catalog_hms
PROPERTIES
(
    "type" = "unified",
    "unified.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

#### AWS S3

##### Instance Profileを使用した認証と認可の場合

- HMSをメタデータサービスとして使用する場合は、以下のようにUnified Catalogを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMRがAWS Glueをメタデータサービスとして使用する場合は、以下のようにUnified Catalogを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_glue
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2"
  );
  ```
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### Assumed Role を使用した認証と認可の場合

- HMS をメタデータサービスとして使用する場合は、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR が AWS Glue をメタデータサービスとして使用している場合は、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_glue
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### IAM User を使用した認証と認可の場合

- HMS をメタデータサービスとして使用する場合は、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR が AWS Glue をメタデータサービスとして使用している場合は、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_glue
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "glue",
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

#### S3 プロトコル互換のオブジェクトストレージ

MinIO を例に、以下のように Unified Catalog を作成できます：

```SQL
CREATE EXTERNAL CATALOG unified_catalog_hms
PROPERTIES
(
    "type" = "unified",
    "unified.metastore.type" = "hive",
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

- Shared Key を使用した認証と認可の場合、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.enable_ssl" = "true",
      "aws.s3.enable_path_style_access" = "true",
      "aws.s3.endpoint" = "<s3_endpoint>",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>"
  );
  ```

- SAS Token を使用した認証と認可の場合、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- Managed Service Identity を使用した認証と認可の場合、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- Service Principal を使用した認証と認可の場合、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- Managed Identity を使用した認証と認可の場合、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- Shared Key を使用した認証と認可の場合、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- Service Principal を使用した認証と認可の場合、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>" 
  );
  ```

#### Google GCS

- VM を使用した認証と認可の場合、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- Service Account を使用した認証と認可の場合、以下のように Unified Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- Impersonation を使用した認証と認可の場合

  - VM インスタンスで Service Account を模倣する場合、以下のように Unified Catalog を作成できます：

    ```SQL
    CREATE EXTERNAL CATALOG unified_catalog_hms
    PROPERTIES
    (
        "type" = "unified",
        "unified.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - ある Service Account で別の Service Account を模倣する場合、以下のように Unified Catalog を作成できます：

    ```SQL
    CREATE EXTERNAL CATALOG unified_catalog_hms
    PROPERTIES
    (
        "type" = "unified",
        "unified.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_private_key>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Unified Catalog の表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) コマンドを使用して、現在の StarRocks クラスターにあるすべてのカタログを照会できます：

```SQL
SHOW CATALOGS;
```

[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を使用して、任意の External Catalog の作成ステートメントを照会することもできます。例えば、以下のコマンドで Unified Catalog `unified_catalog_glue` の作成ステートメントを照会します：

```SQL
SHOW CREATE CATALOG unified_catalog_glue;
```

## Unified Catalog とデータベースを切り替える

以下の方法で目的の Unified Catalog とデータベースに切り替えることができます：

- まず [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) で現在のセッションに有効な Unified Catalog を指定し、次に [USE](../../sql-reference/sql-statements/data-definition/USE.md) でデータベースを指定します：

  ```SQL
  -- 現在のセッションに有効な Catalog を切り替える：
  SET CATALOG <catalog_name>
  -- 現在のセッションに有効なデータベースを指定する：
  USE <db_name>
  ```

- [USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、セッションを直接目的の Unified Catalog の指定データベースに切り替えます：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Unified Catalog を削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) を使用して、任意の External Catalog を削除できます。

例えば、以下のコマンドで Unified Catalog `unified_catalog_glue` を削除します：

```SQL
DROP CATALOG unified_catalog_glue;
```

## Unified Catalog 内のテーブル構造を表示する

以下の方法で Hive テーブルのテーブル構造を表示できます：

- テーブル構造を表示する

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATE コマンドからテーブル構造とテーブルファイルの位置を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Unified Catalog 内のテーブルデータを照会する

以下の操作で Unified Catalog 内のデータを照会できます：

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を使用して、指定された Unified Catalog に属するデータソース内のデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [目的の Unified Catalog とデータベースに切り替える](#unified-catalog-とデータベースを切り替える)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して、目的のデータベース内の目的のテーブルを照会します：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hive、Iceberg、Hudi、または Delta Lake からデータをインポートする

[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して、Hive、Iceberg、Hudi、または Delta Lake のテーブルから StarRocks の Unified Catalog 下のテーブルにデータをインポートできます。

例えば、以下のコマンドで Hive テーブル `hive_table` のデータを StarRocks の Unified Catalog `unified_catalog` 下のデータベース `test_database` のテーブル `test_table` にインポートします：

```SQL
INSERT INTO unified_catalog.test_database.test_table SELECT * FROM hive_table
```

## Unified Catalog 内でデータベースを作成する

StarRocks の内部データディレクトリ (Internal Catalog) と同様に、Unified Catalog の [CREATE DATABASE](../../administration/privilege_item.md#データディレクトリ権限-catalog) 権限を持っていれば、[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) を使用してその Unified Catalog 内にデータベースを作成できます。

> **説明**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザーやロールに対する権限を付与および取り消すことができます。

現在は Hive データベースと Iceberg データベースの作成のみをサポートしています。

[目的の Unified Catalog に切り替え](#unified-catalog-とデータベースを切り替える)て、以下のステートメントでデータベースを作成します：

```SQL
CREATE DATABASE <database_name>
[properties ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location` パラメータはデータベースのファイルパスを指定するために使用され、HDFS とオブジェクトストレージをサポートしています：

- HMS をメタデータサービスとして選択した場合、データベースを作成する際に `location` を指定しないと、システムは HMS のデフォルト `<warehouse_location>/<database_name.db>` をファイルパスとして使用します。
- AWS Glue をメタデータサービスとして選択した場合、`location` パラメータにはデフォルト値がないため、データベースを作成する際にこのパラメータを指定する必要があります。

`prefix` はストレージシステムによって異なります：

| **ストレージシステム**                           | **`Prefix`** **の値**                                        |
| -------------------------------------- | ------------------------------------------------------------ |
| HDFS                                   | `hdfs`                                                       |
| Google GCS                             | `gs`                                                         |
| Azure Blob Storage                     | <ul><li>HTTP プロトコルを介してアクセスできるストレージアカウントの場合、`prefix` は `wasb` です。</li><li>HTTPS プロトコルを介してアクセスできるストレージアカウントの場合、`prefix` は `wasbs` です。</li></ul> |
| Azure Data Lake Storage Gen1           | `adl`                                                        |
| Azure Data Lake Storage Gen2           | <ul><li>HTTP プロトコルを介してアクセスできるストレージアカウントの場合、`prefix` は `abfs` です。</li><li>HTTPS プロトコルを介してアクセスできるストレージアカウントの場合、`prefix` は `abfss` です。</li></ul> |
| 阿里云 OSS                             | `oss`                                                        |
| 腾讯云 COS                             | `cosn`                                                       |
| 华为云 OBS                             | `obs`                                                        |
| AWS S3 及び S3 互換ストレージ（例：MinIO）   | `s3`                                                         |

## Unified Catalog 内からデータベースを削除する

StarRocks の内部データベースと同様に、Unified Catalog 内のデータベースの [DROP](../../administration/privilege_item.md#データベース権限-database) 権限を持っていれば、[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md) を使用してそのデータベースを削除できます。空のデータベースのみ削除が可能です。

> **説明**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザーやロールに対する権限を付与および取り消すことができます。

現在は Hive データベースと Iceberg データベースの削除のみをサポートしています。

データベースを削除しても、HDFS やオブジェクトストレージ上の対応するファイルパスは削除されません。

[目的の Unified Catalog に切り替え](#unified-catalog-とデータベースを切り替える)て、以下のステートメントでデータベースを削除します：

```SQL
DROP DATABASE <database_name>
```

## Unified Catalog 内でテーブルを作成する

StarRocks の内部データベースと同様に、Unified Catalog 内のデータベースの [CREATE TABLE](../../administration/privilege_item.md#データベース権限-database) 権限を持っていれば、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) または [CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) を使用してそのデータベース下にテーブルを作成できます。

> **説明**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザーやロールに対する権限を付与および取り消すことができます。

現在は Hive テーブルと Iceberg テーブルの作成のみをサポートしています。

[目的の Unified Catalog とデータベースに切り替え](#unified-catalog-とデータベースを切り替える)て、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) を使用して Hive テーブルまたは Iceberg テーブルを作成します：

```SQL
CREATE TABLE <table_name>
(column_definition1[, column_definition2, ...]
ENGINE = {|hive|iceberg}
[partition_desc]
```

Hive テーブルと Iceberg テーブルの作成に関する詳細情報は、[Hive テーブルの作成](../catalog/hive_catalog.md#hive-テーブルを作成する)と[Iceberg テーブルの作成](../catalog/iceberg_catalog.md#iceberg-テーブルを作成する)を参照してください。

例えば、以下のステートメントで Hive テーブル `hive_table` を作成します：

```SQL
CREATE TABLE hive_table
(
    action varchar(65533),
    id int,
    dt date
)
ENGINE = hive
PARTITION BY (id,dt);
```

## Unified Catalog 内のテーブルにデータを挿入する

StarRocks の内部テーブルと同様に、Unified Catalog 内のテーブルの [INSERT](../../administration/privilege_item.md#テーブル権限-table) 権限を持っていれば、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して StarRocks のテーブルデータをそのテーブルに書き込むことができます（現在は Parquet 形式の Unified Catalog テーブルへの書き込みのみをサポートしています）。

> **説明**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザーやロールに対する権限を付与および取り消すことができます。

現在は Hive テーブルと Iceberg テーブルへのデータ挿入のみをサポートしています。

[目的の Unified Catalog とデータベースに切り替え](#unified-catalog-とデータベースを切り替える)て、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して Hive テーブルまたは Iceberg テーブルにデータを挿入します：

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータを書き込む。
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

Hive テーブルと Iceberg テーブルへのデータ挿入に関する詳細情報は、[Hive テーブルへのデータ挿入](../catalog/hive_catalog.md#hive-テーブルにデータを挿入する)と[Iceberg テーブルへのデータ挿入](../catalog/iceberg_catalog.md#iceberg-テーブルにデータを挿入する)を参照してください。

例えば、以下のステートメントで Hive テーブル `hive_table` に以下のデータを挿入します：

```SQL
INSERT INTO hive_table
VALUES
    ("buy", 1, "2023-09-01"),
    ("sell", 2, "2023-09-02"),
    ("buy", 3, "2023-09-03");
```

## Unified Catalog 内からテーブルを削除する


同 StarRocks の内部テーブルと同様に、Unified Catalog の内部テーブルの [DROP](../../administration/privilege_item.md#表权限-table) 権限を持っている場合は、[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) を使用してそのテーブルを削除できます。

> **説明**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) コマンドを使用して、ユーザーやロールに対する権限を付与または取り消すことができます。

現在、Hive テーブルと Iceberg テーブルの削除のみをサポートしています。

[目的の Unified Catalog とデータベースに切り替え](#切换-unified-catalog-和数据库)てから、[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) を使用して Hive テーブルまたは Iceberg テーブルを削除します。

```SQL
DROP TABLE <table_name>
```

Hive テーブルと Iceberg テーブルの削除に関する詳細は、[Hive テーブルの削除](../catalog/hive_catalog.md#删除-hive-表)と[Iceberg テーブルの削除](../catalog/iceberg_catalog.md#删除-iceberg-表)を参照してください。

例えば、以下のステートメントを使用して Hive テーブル `hive_table` を削除します：

```SQL
DROP TABLE hive_table FORCE
```

