---
displayed_sidebar: Chinese
---

# Iceberg Catalog

Iceberg Catalog は External Catalog の一種です。StarRocks はバージョン2.4から Iceberg Catalog をサポートしています。以下のことが可能です：

- 手動でテーブルを作成することなく、Iceberg Catalog を通じて Iceberg のデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) や非同期マテリアライズドビュー（バージョン2.5以上）を使用して、Iceberg のデータを加工・モデリングし、StarRocks にインポートできます。
- StarRocks 側で Iceberg のデータベースやテーブルを作成・削除したり、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して StarRocks のテーブルデータを Parquet 形式の Iceberg テーブルに書き込むことができます（バージョン3.1以上）。

Iceberg のデータに正常にアクセスするために、StarRocks クラスタは以下の2つの重要なコンポーネントを統合する必要があります：

- 分散ファイルシステム（HDFS）またはオブジェクトストレージ。現在サポートされているオブジェクトストレージには、AWS S3、Microsoft Azure Storage、Google GCS、その他の S3 プロトコル互換のオブジェクトストレージ（例：アリババクラウド OSS、ファーウェイクラウド OBS、テンセントクラウド COS、火山エンジン TOS、キングソフトクラウド KS3、MinIO、Ceph S3 など）が含まれます。

- メタデータサービス。現在サポートされているメタデータサービスには、Hive Metastore（以下、HMS）、AWS Glue、Tabular があります。

  > **説明**
  >
  > - AWS S3 をストレージシステムとして選択した場合、HMS または AWS Glue をメタデータサービスとして選択できます。他のストレージシステムを選択した場合は、HMS のみをメタデータサービスとして選択できます。
  > - Tabular をメタデータサービスとして使用する場合は、Iceberg の REST Catalog を使用する必要があります。

## 使用説明

- StarRocks が Iceberg データをクエリする際、Parquet および ORC ファイル形式をサポートしています。その中で：

  - Parquet ファイルは SNAPPY、LZ4、ZSTD、GZIP、NO_COMPRESSION の圧縮形式をサポートしています。
  - ORC ファイルは ZLIB、SNAPPY、LZO、LZ4、ZSTD、NO_COMPRESSION の圧縮形式をサポートしています。

- Iceberg Catalog は v1 テーブルデータのクエリをサポートしています。バージョン3.0からは ORC 形式の v2 テーブルデータのクエリをサポートし、バージョン3.1からは Parquet 形式の v2 テーブルデータのクエリをサポートしています。

## 準備作業

Iceberg Catalog を作成する前に、StarRocks クラスタが Iceberg のファイルストレージおよびメタデータサービスに正常にアクセスできることを確認してください。

### AWS IAM

Iceberg が AWS S3 をファイルストレージとして使用する場合、または AWS Glue をメタデータサービスとして使用する場合、StarRocks クラスタが関連する AWS クラウドリソースにアクセスできるように、適切な認証および承認スキームを選択する必要があります。

以下の認証および承認スキームを選択できます：

- Instance Profile（推奨）
- Assumed Role
- IAM User

StarRocks が AWS にアクセスするための認証および承認の詳細については、[AWS 認証方法の設定 - 準備作業](../../integrations/authenticate_to_aws_resources.md#準備作業)を参照してください。

### HDFS

HDFS をファイルストレージとして使用する場合は、StarRocks クラスタで以下の設定を行う必要があります：

- （オプション）HDFS クラスタおよび HMS にアクセスするためのユーザー名を設定します。各 FE の **fe/conf/hadoop_env.sh** ファイルおよび各 BE の **be/conf/hadoop_env.sh** ファイルの最初に `export HADOOP_USER_NAME="<user_name>"` を追加して、このユーザー名を設定できます。設定後、各 FE および BE を再起動して設定を有効にします。このユーザー名を設定しない場合、デフォルトでは FE および BE プロセスのユーザー名が使用されます。各 StarRocks クラスタは1つのユーザー名のみを設定できます。
- Iceberg データをクエリする際、StarRocks クラスタの FE および BE は HDFS クライアントを通じて HDFS クラスタにアクセスします。通常、StarRocks はデフォルトの設定に従って HDFS クライアントを起動するため、手動での設定は不要です。ただし、以下のシナリオでは手動での設定が必要です：
  - HDFS クラスタが高可用性（High Availability、HA）モードを有効にしている場合は、HDFS クラスタの **hdfs-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに配置する必要があります。
  - HDFS クラスタが ViewFs を設定している場合は、HDFS クラスタの **core-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに配置する必要があります。

> **注意**
>
> クエリ時にドメイン名が認識できない（Unknown Host）ためにアクセスに失敗する場合は、HDFS クラスタの各ノードのホスト名と IP アドレスのマッピングを **/etc/hosts** に設定する必要があります。

### Kerberos 認証

HDFS クラスタまたは HMS が Kerberos 認証を有効にしている場合、StarRocks クラスタで以下の設定を行う必要があります：

- 各 FE および各 BE で `kinit -kt keytab_path principal` コマンドを実行し、Key Distribution Center（KDC）から Ticket Granting Ticket（TGT）を取得します。このコマンドを実行するユーザーは HMS および HDFS にアクセスする権限を持っている必要があります。このコマンドを使用して KDC にアクセスすることには有効期限があるため、cron を使用して定期的にこのコマンドを実行する必要があります。
- 各 FE の **$FE_HOME/conf/fe.conf** ファイルおよび各 BE の **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。ここで、`/etc/krb5.conf` は **krb5.conf** ファイルのパスであり、実際のファイルパスに応じて変更できます。

## Iceberg Catalog の作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "iceberg",
    MetastoreParams,
    StorageCredentialParams
)
```

### パラメータ説明

#### catalog_name

Iceberg Catalog の名前。命名規則は以下の通りです：

- 英字 (a-z または A-Z)、数字 (0-9)、アンダースコア (_) のみを含むことができ、英字で始まる必要があります。
- 全体の長さは 1023 文字を超えることはできません。
- Catalog 名は大文字と小文字を区別します。

#### comment

Iceberg Catalog の説明。このパラメータはオプションです。

#### type

データソースのタイプ。`iceberg` に設定します。

#### MetastoreParams

StarRocks が Iceberg クラスタのメタデータサービスにアクセスするためのパラメータ設定。

##### HMS

HMS を Iceberg クラスタのメタデータサービスとして選択する場合は、以下のように `MetastoreParams` を設定してください：

```SQL
"iceberg.catalog.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **説明**
>
> Iceberg データをクエリする前に、すべての HMS ノードのホスト名と IP アドレスのマッピングを **/etc/hosts** に追加する必要があります。そうしないと、クエリを開始したときに StarRocks が HMS にアクセスできない可能性があります。

`MetastoreParams` には以下のパラメータが含まれます。

| パラメータ                         | 必須 | 説明                                                         |
| ----------------------------------- | ---- | ------------------------------------------------------------ |
| iceberg.catalog.type                | はい | Iceberg クラスタで使用されるメタデータサービスのタイプ。`hive` に設定します。 |
| hive.metastore.uris                 | はい | HMS の URI。形式：`thrift://<HMS IP アドレス>:<HMS ポート番号>`。<br />HMS が高可用性モードを有効にしている場合は、複数の HMS アドレスをカンマで区切って記述できます。例：`"thrift://<HMS IP アドレス 1>:<HMS ポート番号 1>,thrift://<HMS IP アドレス 2>:<HMS ポート番号 2>,thrift://<HMS IP アドレス 3>:<HMS ポート番号 3>"`。 |

##### AWS Glue

AWS Glue を Iceberg クラスタのメタデータサービスとして選択する場合（AWS S3 をストレージシステムとして使用する場合のみサポートされます）、以下のように `MetastoreParams` を設定してください：

- Instance Profile を使用した認証と承認

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- Assumed Role を使用した認証と承認

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAM User を使用した認証と承認

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

`MetastoreParams` には以下のパラメータが含まれます。

| パラメータ                      | 必須 | 説明                                                         |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| iceberg.catalog.type          | はい | Iceberg クラスタで使用されるメタデータサービスのタイプ。`glue` に設定します。 |
| aws.glue.use_instance_profile | はい | Instance Profile と Assumed Role の2つの認証方式を有効にするかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`false`。 |
| aws.glue.iam_role_arn         | いいえ | AWS Glue Data Catalog にアクセスする権限を持つ IAM Role の ARN。Assumed Role 認証方式を使用して AWS Glue にアクセスする場合は、このパラメータを指定する必要があります。 |
| aws.glue.region               | はい | AWS Glue Data Catalog が存在するリージョン。例：`us-west-1`。 |
| aws.glue.access_key           | いいえ | IAM User の Access Key。IAM User 認証方式を使用して AWS Glue にアクセスする場合は、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | いいえ | IAM User の Secret Key。IAM User 認証方式を使用して AWS Glue にアクセスする場合は、このパラメータを指定する必要があります。 |

AWS Glue へのアクセスに使用する認証方式の選択方法や、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS Glue へのアクセス認証パラメータ](../../integrations/authenticate_to_aws_resources.md#AWS-Glueへのアクセス認証パラメータ)を参照してください。

##### Tabular

Tabular をメタデータサービスとして使用する場合は、メタデータサービスのタイプを REST (`"iceberg.catalog.type" = "rest"`) に設定する必要があります。以下のように `MetastoreParams` を設定してください：

```SQL
"iceberg.catalog.type" = "rest",
"iceberg.catalog.uri" = "<rest_server_api_endpoint>",
"iceberg.catalog.credential" = "<credential>",
"iceberg.catalog.warehouse" = "<identifier_or_path_to_warehouse>"
```

`MetastoreParams` には以下のパラメータが含まれます。

| パラメータ                   | 必須 | 説明                                                         |

| -------------------------- | ------ | ------------------------------------------------------------ |
| iceberg.catalog.type       | はい      | Iceberg クラスターが使用するメタデータサービスのタイプ。`rest`に設定します。           |
| iceberg.catalog.uri        | はい      | Tabular サービスのエンドポイントURI。例: `https://api.tabular.io/ws`。      |
| iceberg.catalog.credential | はい      | Tabular サービスの認証情報。                                        |
| iceberg.catalog.warehouse  | いいえ      | カタログのウェアハウスの位置または識別子。例: `s3://my_bucket/warehouse_location` または `sandbox`。 |

例えば、`tabular` という名前の Iceberg カタログを作成し、メタデータサービスとして Tabular を使用するには：

```SQL
CREATE EXTERNAL CATALOG tabular
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "rest",
    "iceberg.catalog.uri" = "https://api.tabular.io/ws",
    "iceberg.catalog.credential" = "t-5Ii8e3FIbT9m0:aaaa-3bbbbbbbbbbbbbbbbbbb",
    "iceberg.catalog.warehouse" = "sandbox"
);
```

#### StorageCredentialParams

StarRocks が Iceberg クラスターのファイルストレージにアクセスするためのパラメーター設定。

注意：

- HDFS をストレージシステムとして使用する場合は、`StorageCredentialParams` の設定は不要で、このセクションをスキップできます。AWS S3、S3互換オブジェクトストレージ、Microsoft Azure Storage、または GCS を使用する場合は、`StorageCredentialParams` を設定する必要があります。

- Tabular をメタデータサービスとして使用する場合は、`StorageCredentialParams` の設定は不要で、このセクションをスキップできます。HMS または AWS Glue をメタデータサービスとして使用する場合は、`StorageCredentialParams` を設定する必要があります。

##### AWS S3

AWS S3 を Iceberg クラスターのファイルストレージとして選択した場合、以下のように `StorageCredentialParams` を設定してください：

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

`StorageCredentialParams` には以下のパラメーターが含まれます。

| パラメーター                     | 必須 | 説明                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい       | Instance Profile と Assumed Role の認証方式を指定します。値は `true` または `false`。デフォルトは `false`。 |
| aws.s3.iam_role_arn         | いいえ       | AWS S3 バケットにアクセスする権限を持つ IAM Role の ARN。Assumed Role 認証方式を使用する場合はこのパラメーターを指定する必要があります。 |
| aws.s3.region               | はい       | AWS S3 バケットが存在するリージョン。例: `us-west-1`。                |
| aws.s3.access_key           | いいえ       | IAM ユーザーのアクセスキー。IAM ユーザー認証方式を使用する場合はこのパラメーターを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAM ユーザーのシークレットキー。IAM ユーザー認証方式を使用する場合はこのパラメーターを指定する必要があります。 |

AWS S3 へのアクセスに使用する認証方式の選択方法と、AWS IAM コンソールでのアクセス制御ポリシーの設定方法については、[AWS S3 への認証パラメーター](../../integrations/authenticate_to_aws_resources.md#AWS-S3-への認証パラメーター)を参照してください。

##### 阿里云 OSS

阿里云 OSS を Iceberg クラスターのファイルストレージとして選択した場合、`StorageCredentialParams` に以下の認証パラメーターを設定する必要があります：

```SQL
"aliyun.oss.access_key" = "<user_access_key>",
"aliyun.oss.secret_key" = "<user_secret_key>",
"aliyun.oss.endpoint" = "<oss_endpoint>" 
```

| パラメーター                        | 必須 | 説明                                                         |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aliyun.oss.endpoint             | はい      | 阿里云 OSS のエンドポイント。例: `oss-cn-beijing.aliyuncs.com`。エンドポイントとリージョンの関係を調べるには、[アクセスドメインとデータセンター](https://help.aliyun.com/document_detail/31837.html)を参照してください。    |
| aliyun.oss.access_key           | はい      | 阿里云アカウントまたは RAM ユーザーの AccessKey ID。取得方法については、[AccessKey の取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。                                     |
| aliyun.oss.secret_key           | はい      | 阿里云アカウントまたは RAM ユーザーの AccessKey Secret。取得方法については、[AccessKey の取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。     |

##### S3互換オブジェクトストレージ

Iceberg カタログはバージョン 2.5 から S3互換オブジェクトストレージをサポートしています。

S3互換オブジェクトストレージ（例: MinIO）を Iceberg クラスターのファイルストレージとして選択した場合、以下のように `StorageCredentialParams` を設定してください：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

`StorageCredentialParams` には以下のパラメーターが含まれます。

| パラメーター                          | 必須   | 説明                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | はい      | SSL 接続を有効にするかどうか。<br />値は `true` または `false`。デフォルトは `true`。 |
| aws.s3.enable_path_style_access  | はい      | パススタイルアクセス（Path-Style Access）を有効にするかどうか。<br />値は `true` または `false`。デフォルトは `false`。MinIO を使用する場合は `true` に設定する必要があります。<br />パススタイルURLは以下のフォーマットを使用します：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部（オレゴン）リージョンに `DOC-EXAMPLE-BUCKET1` という名前のバケットを作成し、そのバケット内の `alice.jpg` オブジェクトにアクセスしたい場合、以下のパススタイルURLを使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。|
| aws.s3.endpoint                  | はい      | S3互換オブジェクトストレージにアクセスするためのエンドポイント。 |
| aws.s3.access_key                | はい      | IAM ユーザーのアクセスキー。 |
| aws.s3.secret_key                | はい      | IAM ユーザーのシークレットキー。 |

##### Microsoft Azure Storage

Iceberg カタログはバージョン 3.0 から Microsoft Azure Storage をサポートしています。

###### Azure Blob Storage

Azure Blob Storage を Iceberg クラスターのファイルストレージとして選択した場合、以下のように `StorageCredentialParams` を設定してください：

- Shared Key を使用した認証と認可

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` には以下のパラメーターが含まれます。

  | パラメーター                   | 必須 | 説明                         |
  | -------------------------- | ------------ | -------------------------------- |
  | azure.blob.storage_account | はい           | Blob Storage アカウントのユーザー名。      |
  | azure.blob.shared_key      | はい           | Blob Storage アカウントの Shared Key。 |

- SAS Token を使用した認証と認可

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams` には以下のパラメーターが含まれます。

  | パラメーター                  | 必須 | 説明                                 |
  | ------------------------- | ------------ | ---------------------------------------- |
  | azure.blob.storage_account| はい           | Blob Storage アカウントのユーザー名。              |
  | azure.blob.container      | はい           | Blob コンテナの名前。               |
  | azure.blob.sas_token      | はい           | Blob Storage アカウントにアクセスするための SAS Token。 |

###### Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 を Iceberg クラスターのファイルストレージとして選択した場合、以下のように `StorageCredentialParams` を設定してください：

- Managed Service Identity を使用した認証と認可

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` には以下のパラメーターが含まれます。

  | パラメーター                                 | 必須 | 説明                                                     |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい           | Managed Service Identity 認証方式を有効にするかどうか。`true` に設定します。 |

- Service Principal を使用した認証と認可

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams` には以下のパラメーターが含まれます。

  | パラメーター                 | 必須 | 説明                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい           | Service Principal のクライアント（アプリケーション）ID。               |
  | azure.adls1.oauth2_credential | はい           | 新規作成されたクライアント（アプリケーション）シークレット。                         |
  | azure.adls1.oauth2_endpoint   | はい           | Service Principal またはアプリケーションの OAuth 2.0 トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2

Azure Data Lake Storage Gen2 を Iceberg クラスターのファイルストレージとして選択した場合、以下のように `StorageCredentialParams` を設定してください：

- Managed Identity を使用した認証と認可

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  `StorageCredentialParams` には以下のパラメーターが含まれます。

  | パラメーター                                | 必須 | 説明                                                |
  | --------------------------------------- | ------------ | ------------------------------------------------------- |
  | azure.adls2.oauth2_use_managed_identity | はい           | Managed Identity 認証方式を有効にするかどうか。`true` に設定します。 |
  | azure.adls2.oauth2_tenant_id            | はい           | テナントの ID。                                 |
  | azure.adls2.oauth2_client_id            | はい           | Managed Identity のクライアント（アプリケーション）ID。           |

- Shared Key を使用した認証と認可

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                    | **必須** | **説明**                                   |
  | --------------------------- | ------------ | ------------------------------------------ |
  | azure.adls2.storage_account | はい           | Data Lake Storage Gen2 のアカウント名。      |
  | azure.adls2.shared_key      | はい           | Data Lake Storage Gen2 のアカウントの Shared Key。 |

- Service Principal を使用した認証と認可

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                           | **必須** | **説明**                                                     |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい           | Service Principal のクライアント（アプリケーション）ID。               |
  | azure.adls2.oauth2_client_secret   | はい           | 新規作成されたクライアント（アプリケーション）の Secret。                         |
  | azure.adls2.oauth2_client_endpoint | はい           | Service Principal またはアプリケーションの OAuth 2.0 トークンエンドポイント（v1）。 |

##### Google GCS

Iceberg Catalog はバージョン 3.0 から Google GCS をサポートしています。

Iceberg クラスタのファイルストレージとして Google GCS を選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- VM を使用した認証と認可

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                                   | **デフォルト値** | **値の例** | **説明**                                                 |
  | ------------------------------------------ | ---------- | ------------ | -------------------------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | false      | true         | Compute Engine に紐づけられた Service Account を直接使用するかどうか。 |

- Service Account を使用した認証と認可

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                               | **デフォルト値** | **値の例**                                                 | **説明**                                                     |
  | -------------------------------------- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""         | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | Service Account を作成した際に生成された JSON ファイル内の Email。          |
  | gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | Service Account を作成した際に生成された JSON ファイル内の Private Key ID。 |
  | gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | Service Account を作成した際に生成された JSON ファイル内の Private Key。    |

- Impersonation を使用した認証と認可

  - VM インスタンスで Service Account を模倣する

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | **パラメータ**                                   | **デフォルト値** | **値の例** | **説明**                                                     |
    | ------------------------------------------ | ---------- | ------------ | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false      | true         | Compute Engine に紐づけられた Service Account を直接使用するかどうか。     |
    | gcp.gcs.impersonation_service_account      | ""         | "hello"      | 模倣する対象の Service Account。 |

  - ある Service Account（仮に「Meta Service Account」とします）で別の Service Account（仮に「Data Service Account」とします）を模倣する

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | **パラメータ**                               | **デフォルト値** | **値の例**                                                 | **説明**                                                     |
    | -------------------------------------- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""         | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | Meta Service Account を作成した際に生成された JSON ファイル内の Email。     |
    | gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | Meta Service Account を作成した際に生成された JSON ファイル内の Private Key ID。 |
    | gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | Meta Service Account を作成した際に生成された JSON ファイル内の Private Key。 |
    | gcp.gcs.impersonation_service_account  | ""         | "hello"                                                      | 模倣する対象の Data Service Account。 |

### 例

以下の例では、`iceberg_catalog_hms` または `iceberg_catalog_glue` という名前の Iceberg Catalog を作成し、Iceberg クラスタ内のデータをクエリします。

#### HDFS

HDFS をストレージとして使用する場合、以下のように Iceberg Catalog を作成できます：

```SQL
CREATE EXTERNAL CATALOG iceberg_catalog_hms
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

#### AWS S3

##### Instance Profile を使用した認証と認可の場合

- Iceberg クラスタが HMS をメタデータサービスとして使用している場合、以下のように Iceberg Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"

  );
  ```

- Amazon EMR Iceberg クラスタが AWS Glue をメタデータサービスとして使用している場合、以下のように Iceberg Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_glue
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### Assumed Role を使用した認証と認可の場合

- Iceberg クラスタが HMS をメタデータサービスとして使用している場合、以下のように Iceberg Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Iceberg クラスタが AWS Glue をメタデータサービスとして使用している場合、以下のように Iceberg Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_glue
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### IAM User を使用した認証と認可の場合

- Iceberg クラスタが HMS をメタデータサービスとして使用している場合、以下のように Iceberg Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Iceberg クラスタが AWS Glue をメタデータサービスとして使用している場合、以下のように Iceberg Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_glue
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "glue",
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

#### S3 プロトコルに互換性のあるオブジェクトストレージ

MinIO を例に、以下のように Iceberg Catalog を作成できます：

```SQL
CREATE EXTERNAL CATALOG iceberg_catalog_hms
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "hive",
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

- Shared Key を使用した認証と認可の場合、以下のように Iceberg Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SAS Token を使用した認証と認可の場合、以下のように Iceberg Catalog を作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- Managed Service Identityを使用して認証および承認を行う場合、以下のようにIceberg Catalogを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- Service Principalを使用して認証および承認を行う場合、以下のようにIceberg Catalogを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- Managed Identityを使用して認証および承認を行う場合、以下のようにIceberg Catalogを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- Shared Keyを使用して認証および承認を行う場合、以下のようにIceberg Catalogを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- Service Principalを使用して認証および承認を行う場合、以下のようにIceberg Catalogを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMを使用して認証および承認を行う場合、以下のようにIceberg Catalogを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- Service Accountを使用して認証および承認を行う場合、以下のようにIceberg Catalogを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- Impersonationを使用して認証および承認を行う場合：

  - VMインスタンスを使用してService Accountを模倣する場合、以下のようにIceberg Catalogを作成できます：

    ```SQL
    CREATE EXTERNAL CATALOG iceberg_catalog_hms
    PROPERTIES
    (
        "type" = "iceberg",
        "iceberg.catalog.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    );
    ```

  - あるService Accountを使用して別のService Accountを模倣する場合、以下のようにIceberg Catalogを作成できます：

    ```SQL
    CREATE EXTERNAL CATALOG iceberg_catalog_hms
    PROPERTIES
    (
        "type" = "iceberg",
        "iceberg.catalog.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Icebergカタログを表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスター内のすべてのカタログを照会できます：

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、特定のExternal Catalogの作成ステートメントを照会することもできます。例えば、以下のコマンドを使用してIceberg Catalog `iceberg_catalog_glue`の作成ステートメントを照会します：

```SQL
SHOW CREATE CATALOG iceberg_catalog_glue;
```

## Icebergカタログとデータベースを切り替える

以下の方法で目的のIcebergカタログとデータベースに切り替えることができます：

- まず[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して現在のセッションに有効なIcebergカタログを指定し、次に[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してデータベースを指定します：

  ```SQL
  -- 現在のセッションに有効なカタログを切り替えます：
  SET CATALOG <catalog_name>
  -- 現在のセッションに有効なデータベースを指定します：
  USE <db_name>
  ```

- [USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、セッションを直接目的のIcebergカタログの指定されたデータベースに切り替えます：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Icebergカタログを削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、特定のExternal Catalogを削除できます。

例えば、以下のコマンドを使用してIceberg Catalog `iceberg_catalog_glue`を削除します：

```SQL
DROP CATALOG iceberg_catalog_glue;
```

## Icebergテーブルの構造を表示する

以下の方法でIcebergテーブルの構造を表示できます：

- テーブルの構造を表示する

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEコマンドからテーブルの構造とファイルの保存場所を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Icebergテーブルのデータを照会する

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、指定されたカタログに属するIcebergクラスター内のデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [目的のIcebergカタログとデータベースに切り替える](#icebergカタログとデータベースを切り替える)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、目的のデータベース内の目的のテーブルを照会します：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Icebergデータベースを作成する

StarRocksの内部データディレクトリ（Internal Catalog）と同様に、Iceberg Catalogの[CREATE DATABASE](../../administration/privilege_item.md#データディレクトリ権限-catalog)権限を持っている場合、[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)を使用してそのIceberg Catalog内にデータベースを作成できます。この機能はバージョン3.1からサポートされています。

> **説明**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、ユーザーとロールに権限を付与および取り消すことができます。

[目的のIcebergカタログに切り替え](#icebergカタログとデータベースを切り替える)、次に以下のステートメントでIcebergデータベースを作成します：

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>/")]
```

`location`パラメータを使用して、データベースの具体的なファイルパスを設定できます。HDFSおよびオブジェクトストレージがサポートされています。`location`パラメータを指定しない場合、StarRocksは現在のIceberg Catalogのデフォルトパスにデータベースを作成します。

`prefix`はストレージシステムによって異なります：

| **ストレージシステム**                           | **`Prefix`の値**                                        |
| -------------------------------------- | ------------------------------------------------------------ |
| HDFS                                   | `hdfs`                                                       |
| Google GCS                             | `gs`                                                         |
| Azure Blob Storage                     | <ul><li>HTTPプロトコルを介してアクセスできるストレージアカウントの場合、`prefix`は`wasb`です。</li><li>HTTPSプロトコルを介してアクセスできるストレージアカウントの場合、`prefix`は`wasbs`です。</li></ul> |
| Azure Data Lake Storage Gen1           | `adl`                                                        |
| Azure Data Lake Storage Gen2           | <ul><li>HTTPプロトコルを介してアクセスできるストレージアカウントの場合、`prefix`は`abfs`です。</li><li>HTTPSプロトコルを介してアクセスできるストレージアカウントの場合、`prefix`は`abfss`です。</li></ul> |
| 阿里云 OSS                             | `oss`                                                        |
| 腾讯云 COS                             | `cosn`                                                       |
| 华为云 OBS                             | `obs`                                                        |
| AWS S3およびS3互換ストレージ（例：MinIO）   | `s3`                                                         |

## Icebergデータベースを削除する

StarRocksの内部データベースと同様に、Icebergデータベースの[DROP](../../administration/privilege_item.md#データベース権限-database)権限を持っている場合、[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md)を使用してそのIcebergデータベースを削除できます。この機能はバージョン3.1からサポートされています。空のデータベースのみ削除できます。

> **説明**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、ユーザーとロールに権限を付与および取り消すことができます。

データベースを削除しても、HDFSまたはオブジェクトストレージ上の対応するファイルパスは削除されません。

[目的のIcebergカタログに切り替え](#icebergカタログとデータベースを切り替える)、次に以下のステートメントでIcebergデータベースを削除します：

```SQL
DROP DATABASE <database_name>;
```

## Icebergテーブルを作成する

StarRocksの内部データベースと同様に、Icebergデータベースの[CREATE TABLE](../../administration/privilege_item.md#データベース権限-database)権限を持っている場合、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)または[CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)を使用してそのIcebergデータベースにテーブルを作成できます。この機能はバージョン3.1からサポートされています。

> **説明**
>

> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)の操作を使用して、ユーザーとロールに権限を付与および取り消すことができます。

[ターゲットのIcebergカタログとデータベースに切り替える](#切り替え-icebergカタログ-データベース)。次の構文を使用してIcebergテーブルを作成します：

### 構文

```SQL
CREATE TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...
partition_column_definition1,partition_column_definition2...])
[partition_desc]
[PROPERTIES ("key" = "value", ...)]
[AS SELECT query]
```

### パラメータの説明

#### column_definition

`column_definition`の構文は次のとおりです：

```SQL
col_name col_type [COMMENT 'comment']
```

パラメータの説明：

| **パラメータ**   | **説明**                                                 |
| --------------- | ------------------------------------------------------- |
| col_name        | 列名。                                                    |
| col_type        | 列のデータ型。現在、次のデータ型がサポートされています：TINYINT、SMALLINT、INT、BIGINT、FLOAT、DOUBLE、DECIMAL、DATE、DATETIME、CHAR、VARCHAR[(length)]、ARRAY、MAP、STRUCT。LARGEINT、HLL、BITMAPの型はサポートされていません。 |

> **注意**
>
> すべての非パーティション列はデフォルト値として`NULL`を持ちます（つまり、テーブル作成ステートメントで`DEFAULT "NULL"`を指定します）。パーティション列は最後に宣言する必要があり、`NULL`にすることはできません。

#### partition_desc

`partition_desc`の構文は次のとおりです：

```SQL
PARTITION BY (par_col1[, par_col2...])
```

現在、StarRocksは[Identity Transforms](https://iceberg.apache.org/spec/#partitioning)のみをサポートしています。つまり、ユニークなパーティション値ごとにパーティションが作成されます。

> **注意**
>
> パーティション列は最後に宣言する必要があり、FLOAT、DOUBLE、DECIMAL、DATETIME以外のデータ型をサポートし、`NULL`値はサポートされていません。

#### PROPERTIES

`PROPERTIES`では、`"key" = "value"`の形式でIcebergテーブルのプロパティを宣言できます。詳細については、[Icebergテーブルのプロパティ](https://iceberg.apache.org/docs/latest/configuration/)を参照してください。

以下は一般的なIcebergテーブルのプロパティのいくつかです：

| **プロパティ**       | **説明**                                                     |
| ----------------- | ------------------------------------------------------------ |
| location          | Icebergテーブルのファイルパス。HMSをメタデータサービスとして使用する場合、`location`パラメータを指定する必要はありません。AWS Glueをメタデータサービスとして使用する場合：<ul><li>現在のデータベースを作成するときに`location`パラメータを指定した場合、現在のデータベースの作成時に`location`パラメータを指定する必要はありません。StarRocksは、テーブルを現在のデータベースのファイルパスに作成します。</li><li>現在のデータベースを作成するときに`location`パラメータを指定しなかった場合、現在のデータベースの作成時に`location`パラメータを指定する必要があります。</li></ul> |
| file_format       | Icebergテーブルのファイル形式。現在、Parquet形式のみサポートされています。デフォルト値：`parquet`。 |
| compression_codec | Icebergテーブルの圧縮形式。現在、SNAPPY、GZIP、ZSTD、LZ4がサポートされています。デフォルト値：`gzip`。 |

### 例

1. `unpartition_tbl`という非パーティションテーブルを作成し、`id`と`score`の2つの列を含めます。以下のようになります：

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. `partition_tbl_1`というパーティションテーブルを作成し、`action`、`id`、`dt`の3つの列を含め、`id`と`dt`をパーティション列として定義します。以下のようになります：

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. 元のテーブル`partition_tbl_1`のデータをクエリし、クエリの結果に基づいて`partition_tbl_2`というパーティションテーブルを作成します。`id`と`dt`を`partition_tbl_2`のパーティション列として定義します：

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (id, dt)
   AS SELECT * from partition_tbl_1;
   ```

## Icebergテーブルにデータを挿入する

StarRocks内のテーブルと同様に、Icebergテーブルに[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)権限がある場合、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用してStarRocksテーブルのデータをIcebergテーブルに書き込むことができます（現在、Parquet形式のIcebergテーブルにのみ書き込むことができます）。この機能は、バージョン3.1以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)の操作を使用して、ユーザーとロールに権限を付与および取り消すことができます。

[ターゲットのIcebergカタログとデータベースに切り替える](#切り替え-icebergカタログ-データベース)、次の構文を使用してStarRocksテーブルのデータをParquet形式のIcebergテーブルに書き込みます：

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
> パーティション列は`NULL`を許可しないため、インポート時にパーティション列に値があることを保証する必要があります。

### パラメータの説明

| パラメータ     | 説明                                                         |
| ----------- | ------------------------------------------------------------ |
| INTO        | データをターゲットテーブルに追加します。                                       |
| OVERWRITE   | データをターゲットテーブルに上書きします。                                       |
| column_name | インポートするターゲット列。1つまたは複数の列を指定できます。複数の列を指定する場合は、カンマ（`,`）で区切る必要があります。指定した列は、ターゲットテーブルに存在する列でなければならず、パーティション列を含まなければなりません。このパラメータは、ソーステーブルの列名と異なる場合がありますが、順序は一致している必要があります。パラメータを指定しない場合、デフォルトでターゲットテーブルのすべての列にデータがインポートされます。ソーステーブルの非パーティション列のいずれかがターゲット列に存在しない場合、デフォルト値`NULL`が書き込まれます。クエリの結果の列の型がターゲット列の型と一致しない場合、暗黙的な変換が行われます。変換できない場合、INSERT INTOステートメントは構文解析エラーが発生します。 |
| expression  | 対応する列に値を割り当てるための式。                                   |
| DEFAULT     | 対応する列にデフォルト値を割り当てます。                                         |
| query       | クエリ文。クエリの結果はターゲットテーブルにインポートされます。クエリ文は、StarRocksがサポートする任意のSQLクエリ構文をサポートしています。 |
| PARTITION   | インポートするターゲットパーティション。ターゲットテーブルのすべてのパーティション列を指定する必要があります。指定したパーティション列の順序は、テーブル作成時に定義されたパーティション列の順序と一致しなくても構いません。列名（`column_name`）を使用してインポートするターゲット列を指定することはできません。 |

### 例

1. テーブル`partition_tbl_1`に次の3行のデータを挿入します：

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. 指定した列の順序でテーブル`partition_tbl_1`にSELECTクエリの結果データを挿入します：

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. テーブル`partition_tbl_1`からデータを読み取り、その結果に基づいてテーブル`partition_tbl_1`にデータを挿入します：

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. テーブル`partition_tbl_2`の`dt='2023-09-01'`、`id=1`のパーティションにSELECTクエリの結果データを挿入します：

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT INTO partition_tbl_2 partition(dt='2023-09-01',id=1) SELECT 'order';
   ```

5. テーブル`partition_tbl_1`の`dt='2023-09-01'`、`id=1`のパーティションのすべての`action`列の値を`close`に上書きします：

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT OVERWRITE partition_tbl_1 partition(dt='2023-09-01',id=1) SELECT 'close';
   ```

## Icebergテーブルの削除

StarRocks内のテーブルと同様に、Icebergテーブルを削除するには、Icebergテーブルの[DROP](../../administration/privilege_item.md#表の権限-table)権限が必要です。この機能は、バージョン3.1以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)の操作を使用して、ユーザーとロールに権限を付与および取り消すことができます。

テーブルを削除する操作は、対応するHDFSまたはオブジェクトストレージ上のファイルパスとデータを削除しません。

テーブルを強制的に削除する（`FORCE`キーワードを追加）と、HDFSまたはオブジェクトストレージ上のデータが削除されますが、対応するファイルパスは削除されません。

[ターゲットのIcebergカタログとデータベースに切り替える](#切り替え-icebergカタログ-データベース)、次のステートメントを使用してIcebergテーブルを削除します：

```SQL
DROP TABLE <table_name> [FORCE];
```

## メタデータキャッシュの設定

Icebergのメタデータファイルは、AWS S3またはHDFSに保存される場合があります。StarRocksは、デフォルトでIcebergメタデータをメモリにキャッシュします。クエリの高速化のために、StarRocksはメモリとディスクの2段階のメタデータキャッシュメカニズムを提供し、最初のクエリでキャッシュがトリガーされ、後続のクエリではキャッシュが優先されます。キャッシュに対応するメタデータがない場合は、直接リモートストレージにアクセスします。

StarRocksは、最も最近使用された（LRU）ポリシーを使用してデータをキャッシュし、削除します。基本的な原則は次のとおりです：

- メモリからメタデータを読み取ります。メモリに見つからない場合は、ディスクから読み取ります。ディスクから読み取ったメタデータはメモリにロードされます。ディスクのキャッシュにヒットしない場合は、リモートストレージからメタデータを取得し、メモリにキャッシュします。
- メモリから削除されたメタデータはディスクに書き込まれます。ディスクから削除されたメタデータは破棄されます。

次のFE設定項目を使用してIcebergメタデータのキャッシュ方法を設定できます：

| **設定項目**                                       | **単位** | **デフォルト値**                                           | **説明**                                                     |
| ------------------------------------------------ | -------- | ---------------------------------------------------- | ------------------------------------------------------------ |

```
| enable_iceberg_metadata_disk_cache               | なし     | `false`                                              | ディスクキャッシュを有効にするかどうか。                     |
| iceberg_metadata_cache_disk_path                 | なし     | `StarRocksFE.STARROCKS_HOME_DIR + "/caches/iceberg"` | ディスクキャッシュのメタデータファイルの位置。                 |
| iceberg_metadata_disk_cache_capacity             | バイト   | `2147483648`、すなわち 2 GB                          | ディスクキャッシュにおけるメタデータの最大容量。               |
| iceberg_metadata_memory_cache_capacity           | バイト   | `536870912`、すなわち 512 MB                         | メモリキャッシュにおけるメタデータの最大容量。                 |
| iceberg_metadata_memory_cache_expiration_seconds | 秒       | `86500`                                              | メモリキャッシュのメタデータが最後にアクセスされてからの有効期限。 |
| iceberg_metadata_disk_cache_expiration_seconds   | 秒       | `604800`、すなわち1週間                              | ディスクキャッシュのメタデータが最後にアクセスされてからの有効期限。 |
| iceberg_metadata_cache_max_entry_size            | バイト   | `8388608`、すなわち 8 MB                             | キャッシュされる単一ファイルの最大サイズ。これにより、あまりに大きなファイルが他のファイルのスペースを圧迫するのを防ぎます。このサイズを超えるファイルはキャッシュされず、クエリがヒットした場合は直接リモートのメタデータファイルにアクセスします。 |
