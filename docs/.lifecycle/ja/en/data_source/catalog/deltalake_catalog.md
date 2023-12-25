---
displayed_sidebar: English
---

# Delta Lake カタログ

Delta Lake カタログは、データを取り込むことなく Delta Lake からクエリを行うことができる外部カタログの一種です。

また、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して Delta Lake カタログに基づいて Delta Lake からデータを直接変換し、ロードすることもできます。StarRocks は v2.5 以降、Delta Lake カタログをサポートしています。

Delta Lake クラスター上で SQL ワークロードを成功させるためには、StarRocks クラスターが以下の二つの重要なコンポーネントと統合する必要があります：

- 分散ファイルシステム（HDFS）またはオブジェクトストレージ（例：AWS S3、Microsoft Azure Storage、Google GCS、その他の S3 互換ストレージシステム、例えば MinIO）

- メタストア（例：Hive メタストアや AWS Glue）

  > **注記**
  >
  > ストレージとして AWS S3 を選択する場合、メタストアとして HMS または AWS Glue を使用できます。他のストレージシステムを選択する場合は、HMS をメタストアとしてのみ使用できます。

## 使用上の注意

- StarRocks がサポートする Delta Lake のファイル形式は Parquet です。Parquet ファイルは、SNAPPY、LZ4、ZSTD、GZIP、および NO_COMPRESSION の圧縮形式をサポートしています。
- StarRocks がサポートしていない Delta Lake のデータ型は MAP と STRUCT です。

## 統合の準備

Delta Lake カタログを作成する前に、StarRocks クラスターが Delta Lake クラスターのストレージシステムおよびメタストアと統合できるようにしてください。

### AWS IAM

Delta Lake クラスターがストレージとして AWS S3 を使用するか、メタストアとして AWS Glue を使用する場合、適切な認証方法を選択し、StarRocks クラスターが関連する AWS クラウドリソースにアクセスできるように必要な準備を行ってください。

推奨される認証方法は以下の通りです：

- インスタンスプロファイル
- アサムドロール
- IAM ユーザー

上記の認証方法の中で、インスタンスプロファイルが最も広く使用されています。

詳細については、[AWS IAM での認証準備](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)を参照してください。

### HDFS

ストレージとして HDFS を選択する場合、StarRocks クラスターを以下のように設定してください：

- （オプション）HDFS クラスターおよび Hive メタストアにアクセスするために使用するユーザー名を設定します。デフォルトでは、StarRocks は FE および BE プロセスのユーザー名を使用して HDFS クラスターおよび Hive メタストアにアクセスします。`export HADOOP_USER_NAME="<user_name>"` を各 FE の **fe/conf/hadoop_env.sh** ファイルおよび各 BE の **be/conf/hadoop_env.sh** ファイルの先頭に追加することでユーザー名を設定することもできます。これらのファイルにユーザー名を設定した後、各 FE および各 BE を再起動してパラメータ設定を有効にします。StarRocks クラスターごとに設定できるユーザー名は一つだけです。
- Delta Lake データをクエリする際、StarRocks クラスターの FE と BE は HDFS クライアントを使用して HDFS クラスターにアクセスします。ほとんどの場合、StarRocks はデフォルトの設定を使用して HDFS クライアントを起動し、クラスターの設定を変更する必要はありません。ただし、以下の状況では StarRocks クラスターを設定する必要があります：

  - HDFS クラスターで高可用性（HA）が有効になっている場合：HDFS クラスターの **hdfs-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに追加します。
  - View File System（ViewFs）が HDFS クラスターで有効になっている場合：HDFS クラスターの **core-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに追加します。

> **注記**
>
> クエリを送信した際に未知のホストを示すエラーが返される場合は、HDFS クラスターノードのホスト名と IP アドレスのマッピングを **/etc/hosts** に追加する必要があります。

### Kerberos 認証

HDFS クラスターや Hive メタストアで Kerberos 認証が有効になっている場合、StarRocks クラスターを以下のように設定します：

- 各 FE および各 BE で `kinit -kt keytab_path principal` コマンドを実行して、Key Distribution Center（KDC）から Ticket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFS クラスターおよび Hive メタストアへのアクセス権が必要です。KDC へのアクセスは時間に敏感なため、このコマンドを定期的に実行するために cron を使用する必要があります。
- 各 FE の **$FE_HOME/conf/fe.conf** ファイルおよび各 BE の **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。この例では、`/etc/krb5.conf` は **krb5.conf** ファイルの保存パスです。必要に応じてパスを変更してください。

## Delta Lake カタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "deltalake",
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### パラメータ

#### catalog_name

Delta Lake カタログの名前です。命名規則は以下の通りです：

- 名前には文字、数字（0-9）、およびアンダースコア（_）を含めることができます。最初の文字は必ず文字である必要があります。
- 名前は大文字と小文字を区別し、長さは 1023 文字を超えることはできません。

#### comment

Delta Lake カタログの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。値を `deltalake` に設定します。

#### MetastoreParams

StarRocks がデータソースのメタストアと統合する方法に関するパラメータのセットです。

##### Hive メタストア

データソースのメタストアとして Hive メタストアを選択する場合、`MetastoreParams` を以下のように設定します：

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> Delta Lake データをクエリする前に、Hive メタストアノードのホスト名と IP アドレスのマッピングを `/etc/hosts` に追加する必要があります。そうしないと、クエリを開始する際に StarRocks が Hive メタストアにアクセスできない可能性があります。

以下の表は、`MetastoreParams` で設定する必要があるパラメータを説明しています。

| パラメータ             | 必須 | 説明                                                          |
| ------------------- | ---- | ------------------------------------------------------------ |
| hive.metastore.type | はい  | Delta Lake クラスターで使用するメタストアのタイプです。値を `hive` に設定します。 |
| hive.metastore.uris | はい  | Hive メタストアの URI です。形式：`thrift://<metastore_IP_address>:<metastore_port>`。<br />Hive メタストアで高可用性（HA）が有効な場合、複数のメタストア URI を指定し、それらをコンマ（,）で区切ることができます。例：`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

データソースのメタストアとして AWS Glue を選択する場合（ストレージとして AWS S3 を選択した場合にのみサポートされます）、インスタンスプロファイルベースの認証方法を選択するために `MetastoreParams` を以下のように設定します：

  ```SQL
  -- ここには具体的な設定が必要です。文書には具体的な設定が記載されていないため、この部分は空白です。
  ```

  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 想定されるロールベース認証方法を選択するには、`MetastoreParams`を以下のように設定します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAMユーザーベース認証方法を選択するには、`MetastoreParams`を以下のように設定します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

以下の表は、`MetastoreParams`で設定する必要があるパラメータを説明しています。

| パラメーター                     | 必須 | 説明                                                  |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type           | はい      | Delta Lakeクラスターで使用するメタストアのタイプ。`glue`に設定します。 |
| aws.glue.use_instance_profile | はい      | インスタンスプロファイルベース認証方法または想定されるロールベース認証方法を有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。 |
| aws.glue.iam_role_arn         | いいえ       | AWS Glue Data Catalogに対する権限を持つIAMロールのARN。想定されるロールベース認証方法を使用してAWS Glueにアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.region               | はい      | AWS Glue Data Catalogが存在するリージョン。例: `us-west-1`。 |
| aws.glue.access_key           | いいえ       | AWS IAMユーザーのアクセスキー。IAMユーザーベース認証方法を使用してAWS Glueにアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | いいえ       | AWS IAMユーザーのシークレットキー。IAMユーザーベース認証方法を使用してAWS Glueにアクセスする場合、このパラメータを指定する必要があります。 |

AWS Glueへのアクセス方法を選択し、AWS IAMコンソールでアクセスコントロールポリシーを設定する方法については、[AWS Glueへのアクセスのための認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocksがストレージシステムと統合するためのパラメータセットです。このパラメータセットはオプションです。

HDFSをストレージとして使用する場合、`StorageCredentialParams`を設定する必要はありません。

AWS S3、その他のS3互換ストレージシステム、Microsoft Azure Storage、またはGoogle GCSをストレージとして使用する場合は、`StorageCredentialParams`を設定する必要があります。

##### AWS S3

Delta LakeクラスターのストレージとしてAWS S3を選択する場合、以下のいずれかのアクションを実行します。

- インスタンスプロファイルベース認証方法を選択するには、`StorageCredentialParams`を以下のように設定します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 想定されるロールベース認証方法を選択するには、`StorageCredentialParams`を以下のように設定します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーベース認証方法を選択するには、`StorageCredentialParams`を以下のように設定します。

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

| パラメーター                   | 必須 | 説明                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい      | インスタンスプロファイルベース認証方法または想定されるロールベース認証方法を有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | いいえ       | AWS S3バケットに対する権限を持つIAMロールのARN。想定されるロールベース認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい      | AWS S3バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | いいえ       | IAMユーザーのアクセスキー。IAMユーザーベース認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAMユーザーのシークレットキー。IAMユーザーベース認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3へのアクセス方法を選択し、AWS IAMコンソールでアクセスコントロールポリシーを設定する方法については、[AWS S3へのアクセスのための認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

Delta Lakeカタログは、v2.5以降のS3互換ストレージシステムをサポートしています。

MinIOなどのS3互換ストレージシステムをDelta Lakeクラスターのストレージとして選択する場合、`StorageCredentialParams`を以下のように設定して、統合が成功するようにします。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

| パラメーター                        | 必須 | 説明                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | はい      | SSL接続を有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access  | はい      | パス形式のアクセスを有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。MinIOの場合、値を`true`に設定する必要があります。<br />パス形式のURLは、次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部（オレゴン）リージョンに`DOC-EXAMPLE-BUCKET1`という名前のバケットを作成し、そのバケット内の`alice.jpg`オブジェクトにアクセスする場合、次のパス形式のURLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | はい      | AWS S3ではなく、S3互換ストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key                | はい      | IAMユーザーのアクセスキー。 |
| aws.s3.secret_key                | はい      | IAMユーザーのシークレットキー。 |

##### Microsoft Azure Storage

Delta Lakeカタログは、v3.0以降のMicrosoft Azure Storageをサポートしています。

###### Azure Blob Storage

Delta LakeクラスターのストレージとしてBlob Storageを選択する場合、以下のいずれかのアクションを実行します。

- 共有キー認証方法を選択するには、`StorageCredentialParams`を以下のように設定します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。


  | **パラメーター**              | **必須** | **説明**                              |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | はい          | Blob Storage アカウントのユーザー名です。   |
  | azure.blob.shared_key      | はい          | Blob Storage アカウントの共有キーです。 |

- SAS トークン認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターの説明です。

  | **パラメーター**             | **必須** | **説明**                                              |
  | ------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.storage_account| はい          | Blob Storage アカウントのユーザー名です。                   |
  | azure.blob.container      | はい          | データを格納する blob コンテナーの名前です。        |
  | azure.blob.sas_token      | はい          | Blob Storage アカウントへのアクセスに使用される SAS トークンです。 |

###### Azure Data Lake Storage Gen1

Delta Lake クラスターのストレージとして Data Lake Storage Gen1 を選択する場合、以下のアクションのいずれかを実行します。

- 管理対象サービス ID の認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターの説明です。

  | **パラメーター**                            | **必須** | **説明**                                              |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい          | 管理対象サービス ID の認証方法を有効にするかどうかを指定します。値は `true` に設定します。 |

- サービス プリンシパルの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターの説明です。

  | **パラメーター**                 | **必須** | **説明**                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい          | サービス プリンシパルのクライアント (アプリケーション) ID です。        |
  | azure.adls1.oauth2_credential | はい          | 作成された新しいクライアント (アプリケーション) シークレットの値です。    |
  | azure.adls1.oauth2_endpoint   | はい          | サービス プリンシパルまたはアプリケーションの OAuth 2.0 トークン エンドポイント (v1) です。 |

###### Azure Data Lake Storage Gen2

Delta Lake クラスターのストレージとして Data Lake Storage Gen2 を選択する場合、以下のアクションのいずれかを実行します。

- マネージド ID の認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターの説明です。

  | **パラメーター**                           | **必須** | **説明**                                              |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | はい          | マネージド ID 認証方法を有効にするかどうかを指定します。値は `true` に設定します。 |
  | azure.adls2.oauth2_tenant_id            | はい          | データにアクセスするテナントの ID です。          |
  | azure.adls2.oauth2_client_id            | はい          | マネージド ID のクライアント (アプリケーション) ID です。         |

- 共有キーの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターの説明です。

  | **パラメーター**               | **必須** | **説明**                                              |
  | --------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account | はい          | Data Lake Storage Gen2 ストレージ アカウントのユーザー名です。 |
  | azure.adls2.shared_key      | はい          | Data Lake Storage Gen2 ストレージ アカウントの共有キーです。 |

- サービス プリンシパルの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターの説明です。

  | **パラメーター**                      | **必須** | **説明**                                              |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい          | サービス プリンシパルのクライアント (アプリケーション) ID です。        |
  | azure.adls2.oauth2_client_secret   | はい          | 作成された新しいクライアント (アプリケーション) シークレットの値です。    |
  | azure.adls2.oauth2_client_endpoint | はい          | サービス プリンシパルまたはアプリケーションの OAuth 2.0 トークン エンドポイント (v1) です。 |

##### Google GCS

Delta Lake カタログは、v3.0 以降 Google GCS をサポートしています。

Delta Lake クラスターのストレージとして Google GCS を選択する場合、以下のアクションのいずれかを実行します。

- VM ベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターの説明です。

  | **パラメーター**                              | **デフォルト値** | **値の例** | **説明**                                              |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engine にバインドされているサービス アカウントを直接使用するかどうかを指定します。 |

- サービス アカウント ベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>",
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターの説明です。

  | **パラメーター**                          | **デフォルト値** | **値の例**                                        | **説明**                                              |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントの作成時に生成された JSON ファイル内のメールアドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウントの作成時に生成された JSON ファイル内のプライベートキー ID です。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウントの作成時に生成された JSON ファイル内のプライベートキーです。 |

- 偽装ベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  - VM インスタンスがサービスアカウントを偽装する場合：
  
    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    以下の表は、`StorageCredentialParams` で設定する必要があるパラメーターの説明です。

    | **パラメーター**                              | **デフォルト値** | **値の例** | **説明**                                              |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineに紐づけられたサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | なりすますサービスアカウントを指定します。            |

  - サービスアカウント（一時的にメタサービスアカウントと呼ばれる）が別のサービスアカウント（一時的にデータサービスアカウントと呼ばれる）になりすますように設定します：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

    | **パラメーター**                          | **デフォルト値** | **値の例**                                        | **説明**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウント作成時に生成されたJSONファイル内のメールアドレス。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキーID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキー。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | なりすますデータサービスアカウント。       |

#### MetadataUpdateParams

StarRocksがDelta Lakeのキャッシュされたメタデータをどのように更新するかについてのパラメーターセットです。このパラメーターセットはオプションです。

StarRocksはデフォルトで[自動非同期更新ポリシー](#appendix-understand-metadata-automatic-asynchronous-update)を実装しています。

ほとんどの場合、`MetadataUpdateParams`を無視して、そのポリシーパラメーターを調整する必要はありません。なぜなら、これらのパラメーターのデフォルト値は既に最適なパフォーマンスを提供しているからです。

しかし、Delta Lakeでのデータ更新頻度が高い場合は、これらのパラメーターを調整して、自動非同期更新のパフォーマンスをさらに最適化することができます。

> **注記**
>
> ほとんどの場合、Delta Lakeデータが1時間以下の粒度で更新される場合、データ更新頻度は高いと考えられます。

| パラメーター                              | 必須 | 説明                                                  |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | No       | StarRocksがDelta Lakeテーブルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| enable_remote_file_cache               | No       | StarRocksがDelta Lakeテーブルまたはパーティションの下層データファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | No       | StarRocksが自身にキャッシュされたDelta Lakeテーブルまたはパーティションのメタデータを非同期で更新する時間間隔。単位: 秒。デフォルト値: `7200`（2時間）。 |
| remote_file_cache_refresh_interval_sec | No       | StarRocksが自身にキャッシュされたDelta Lakeテーブルまたはパーティションの下層データファイルのメタデータを非同期で更新する時間間隔。単位: 秒。デフォルト値: `60`。 |
| metastore_cache_ttl_sec                | No       | StarRocksが自身にキャッシュされたDelta Lakeテーブルまたはパーティションのメタデータを自動的に破棄する時間間隔。単位: 秒。デフォルト値: `86400`（24時間）。 |
| remote_file_cache_ttl_sec              | No       | StarRocksが自身にキャッシュされたDelta Lakeテーブルまたはパーティションの下層データファイルのメタデータを自動的に破棄する時間間隔。単位: 秒。デフォルト値: `129600`（36時間）。 |

### 例

以下の例は、使用するメタストアのタイプに応じて、`deltalake_catalog_hms`または`deltalake_catalog_glue`という名前のDelta Lakeカタログを作成し、Delta Lakeクラスターからデータをクエリする方法を示しています。

#### HDFS

HDFSをストレージとして使用する場合、以下のコマンドを実行します：

```SQL
CREATE EXTERNAL CATALOG deltalake_catalog_hms
PROPERTIES
(
    "type" = "deltalake",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

#### AWS S3

##### インスタンスプロファイルベースの認証を選択した場合

- Delta LakeクラスターでHiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Delta LakeクラスターでAWS Glueを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_glue
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 想定されるロールベースの認証を選択した場合

- Delta LakeクラスターでHiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Delta LakeクラスターでAWS Glueを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_glue
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### IAMユーザーベースの認証を選択した場合

- Delta LakeクラスターでHiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Delta LakeクラスターでAWS Glueを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_glue
  PROPERTIES
  (
      "type" = "deltalake",
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

#### S3互換ストレージシステム

MinIOを例に、以下のコマンドを実行します。

```SQL
CREATE EXTERNAL CATALOG deltalake_catalog_hms
PROPERTIES
(
    "type" = "deltalake",
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

- 共有キー認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SASトークン認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- マネージドサービスアイデンティティ認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- サービスプリンシパル認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- マネージドアイデンティティ認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 共有キー認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- サービスプリンシパル認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMベースの認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- サービスアカウントベースの認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 代理ベースの認証方法を選択する場合:

  - VMインスタンスがサービスアカウントを代理する場合、以下のコマンドを実行します。

    ```SQL
    CREATE EXTERNAL CATALOG deltalake_catalog_hms
    PROPERTIES
    (
        "type" = "deltalake",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - サービスアカウントが別のサービスアカウントを代理する場合、以下のコマンドを実行します。

    ```SQL
    CREATE EXTERNAL CATALOG deltalake_catalog_hms
    PROPERTIES
    (
        "type" = "deltalake",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_private_key>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Delta Lakeカタログの表示

現在のStarRocksクラスター内のすべてのカタログを照会するには、[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用します。

```SQL
SHOW CATALOGS;
```

外部カタログの作成ステートメントを照会するには、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用することもできます。以下の例では、`deltalake_catalog_glue`という名前のDelta Lakeカタログの作成ステートメントをクエリします。

```SQL
SHOW CREATE CATALOG deltalake_catalog_glue;
```

## Delta Lakeカタログとその中のデータベースに切り替える

Delta Lakeカタログとその中のデータベースに切り替えるには、以下の方法のいずれかを使用できます。

- 現在のセッションでDelta Lakeカタログを指定するには、[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用し、その後アクティブなデータベースを指定するには[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用します。

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替えます:
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定します:
  USE <db_name>
  ```

- Delta Lakeカタログとその中のデータベースに直接切り替えるには、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用します。

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Delta Lakeカタログを削除する

外部カタログを削除するには、[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用します。

以下の例では、`deltalake_catalog_glue`という名前のDelta Lakeカタログを削除します。

```SQL
DROP CATALOG deltalake_catalog_glue;
```

## Delta Lakeテーブルのスキーマを表示する

Delta Lakeテーブルのスキーマを表示するには、以下の構文のいずれかを使用します。

- スキーマを表示する

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Delta Lakeテーブルをクエリする


1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を使用して、Delta Lake クラスター内のデータベースを表示します。

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Delta Lake カタログとその中のデータベースに切り替える](#switch-to-a-delta-lake-catalog-and-a-database-in-it)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して、指定されたデータベース内の目的のテーブルをクエリします。

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Delta Lake からデータを読み込む

`olap_tbl` という名前の OLAP テーブルがある場合、以下のようにデータを変換してロードできます。

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM deltalake_table
```

## メタデータキャッシュの手動更新または自動更新

### 手動更新

デフォルトでは、StarRocks は Delta Lake のメタデータをキャッシュし、非同期モードでメタデータを自動更新してパフォーマンスを向上させます。さらに、Delta Lake テーブルにスキーマ変更やテーブル更新が行われた後、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) を使用してメタデータを手動で更新することもできます。これにより、StarRocks はできるだけ早く最新のメタデータを取得し、適切な実行プランを生成することができます。

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自動増分更新

自動非同期更新ポリシーとは異なり、自動増分更新ポリシーでは、StarRocks クラスターの FE が Hive メタストアからイベント（列の追加、パーティションの削除、データの更新など）を読み取ります。StarRocks はこれらのイベントに基づいて FE にキャッシュされたメタデータを自動的に更新します。つまり、Delta Lake テーブルのメタデータを手動で更新する必要はありません。

自動増分更新を有効にするには、以下の手順に従ってください。

#### ステップ 1: Hive メタストアのイベントリスナーを設定する

Hive メタストア v2.x および v3.x はイベントリスナーの設定をサポートしています。ここでは、Hive メタストア v3.1.2 用のイベントリスナー設定を例として使用します。以下の設定項目を **$HiveMetastore/conf/hive-site.xml** ファイルに追加し、その後 Hive メタストアを再起動します。

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

FE のログファイルで `event id` を検索して、イベントリスナーが正常に設定されているかどうかを確認できます。設定が失敗した場合、`event id` の値は `0` になります。

#### ステップ 2: StarRocks で自動増分更新を有効にする

自動増分更新は、StarRocks クラスター内の単一の Delta Lake カタログまたはすべての Delta Lake カタログに対して有効にできます。

- 単一の Delta Lake カタログに対して自動増分更新を有効にするには、Delta Lake カタログを作成する際に `PROPERTIES` で `enable_hms_events_incremental_sync` パラメータを `true` に設定します。

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.uris" = "thrift://102.168.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```

- すべての Delta Lake カタログに対して自動増分更新を有効にするには、各 FE の `$FE_HOME/conf/fe.conf` ファイルに `"enable_hms_events_incremental_sync" = "true"` を追加し、各 FE を再起動してパラメータ設定を有効にします。

ビジネス要件に基づいて、各 FE の `$FE_HOME/conf/fe.conf` ファイルで以下のパラメータを調整し、各 FE を再起動してパラメータ設定を有効にすることもできます。

| パラメータ                         | 説明                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocks が Hive メタストアからイベントを読み取る時間間隔。デフォルト値: `5000`。単位: ミリ秒。 |
| hms_events_batch_size_per_rpc     | StarRocks が一度に読み取ることができるイベントの最大数。デフォルト値: `500`。 |
| enable_hms_parallel_process_evens | StarRocks がイベントを読み取る際に、イベントを並行して処理するかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `true`。`true` は並列処理を有効にし、`false` は並列処理を無効にします。 |
| hms_process_events_parallel_num   | StarRocks が並行して処理できるイベントの最大数。デフォルト値: `4`。 |

## 付録: メタデータ自動非同期更新についての理解

自動非同期更新は、StarRocks が Delta Lake カタログのメタデータを更新するために使用するデフォルトのポリシーです。

デフォルトでは（つまり、`enable_metastore_cache` と `enable_remote_file_cache` パラメータが両方とも `true` に設定されている場合）、クエリが Delta Lake テーブルのパーティションにヒットすると、StarRocks はそのパーティションのメタデータとそのパーティションの基になるデータファイルのメタデータを自動的にキャッシュします。キャッシュされたメタデータは、遅延更新ポリシーによって更新されます。

例えば、`table2` という名前の Delta Lake テーブルがあり、`p1`、`p2`、`p3`、`p4` の 4 つのパーティションがあります。クエリが `p1` にヒットし、StarRocks は `p1` のメタデータと `p1` の基になるデータファイルのメタデータをキャッシュします。キャッシュされたメタデータを更新および破棄するデフォルトの時間間隔は以下の通りです。

- キャッシュされたメタデータを非同期的に更新する時間間隔（`metastore_cache_refresh_interval_sec` パラメータによって指定）は `p1` に対して 2 時間です。
- 基になるデータファイルのキャッシュされたメタデータを非同期的に更新する時間間隔（`remote_file_cache_refresh_interval_sec` パラメータによって指定）は `p1` に対して 60 秒です。
- キャッシュされたメタデータを自動的に破棄する時間間隔（`metastore_cache_ttl_sec` パラメータによって指定）は `p1` に対して 24 時間です。
- 基になるデータファイルのキャッシュされたメタデータを自動的に破棄する時間間隔（`remote_file_cache_ttl_sec` パラメータによって指定）は `p1` に対して 36 時間です。

以下の図は、タイムライン上の時間間隔をわかりやすく示しています。

![キャッシュされたメタデータの更新と破棄のタイムライン](../../assets/catalog_timeline.png)

その後、StarRocks は以下のルールに従ってメタデータを更新または破棄します。

- 別のクエリが `p1` に再びヒットし、最後の更新からの現在の時間が 60 秒未満である場合、StarRocks は `p1` のキャッシュされたメタデータや `p1` の基になるデータファイルのキャッシュされたメタデータを更新しません。
- 別のクエリが`p1`を再び参照し、最後の更新から現在までの時間が60秒を超えた場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを更新します。
- 別のクエリが`p1`を再び参照し、最後の更新から現在までの時間が2時間を超えた場合、StarRocksは`p1`のキャッシュされたメタデータを更新します。
- `p1`が最後の更新から24時間以内にアクセスされていない場合、StarRocksは`p1`のキャッシュされたメタデータを破棄します。メタデータは次のクエリ時に再度キャッシュされます。
- `p1`が最後の更新から36時間以内にアクセスされていない場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを破棄します。メタデータは次のクエリ時に再度キャッシュされます。
