---
displayed_sidebar: "Japanese"
---

# Delta Lake カタログ

Delta Lake カタログは、データの取り込みなしで Delta Lake からデータをクエリできるようにする外部カタログの一種です。

また、Delta Lake カタログに基づいて [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して直接データを変換およびロードすることもできます。StarRocks は、v2.5 以降で Delta Lake カタログをサポートしています。

Delta Lake クラスタでの SQL ワークロードの成功を確保するためには、StarRocks クラスタが次の 2 つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム (HDFS) または AWS S3、Microsoft Azure Storage、Google GCS、またはその他の S3 互換ストレージシステム（たとえば、MinIO）のようなオブジェクトストレージ
- Hive メタストアまたは AWS Glue のようなメタストア

  > **注記**
  >
  > ストレージとして AWS S3 を選択した場合、メタストアとして HMS または AWS Glue を使用できます。他のストレージシステムを選択した場合、メタストアとして HMS のみを使用できます。

## 使用上の注意

- StarRocks がサポートする Delta Lake のファイル形式は Parquet です。Parquet ファイルは、SNAPPY、LZ4、ZSTD、GZIP、および NO_COMPRESSION の圧縮形式をサポートしています。
- StarRocks がサポートしない Delta Lake のデータ型は MAP および STRUCT です。

## 統合の準備

Delta Lake カタログを作成する前に、StarRocks クラスタが Delta Lake クラスタのストレージシステムとメタストアと統合できることを確認してください。

### AWS IAM

Delta Lake クラスタがストレージとして AWS S3 を使用する場合またはメタストアとして AWS Glue を使用する場合、適切な認証方法を選択し、関連する AWS クラウドリソースに StarRocks クラスタがアクセスできるように必要な準備を行ってください。

次の認証方法が推奨されます。

- インスタンスプロファイル
- 仮定されるロール
- IAM ユーザー

上記の 3 つの認証方法のうち、インスタンスプロファイルが最も一般的に使用されています。

詳細については、[AWS IAM での認証の準備](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)を参照してください。

### HDFS

ストレージとして HDFS を選択した場合、StarRocks クラスタを次のように構成してください。

- (オプション) HDFS クラスタと Hive メタストアにアクセスするために使用されるユーザー名を設定します。デフォルトでは、StarRocks は FE プロセスと BE プロセスのユーザー名を使用して HDFS クラスタと Hive メタストアにアクセスします。各 FE の **fe/conf/hadoop_env.sh** ファイルと各 BE の **be/conf/hadoop_env.sh** ファイルの先頭に `export HADOOP_USER_NAME="<user_name>"` を追加してユーザー名を設定することもできます。これらのファイルでユーザー名を設定した後、各 FE と各 BE を再起動してパラメータ設定を有効にします。StarRocks クラスタごとに 1 つのユーザー名のみを設定できます。
- Delta Lake データをクエリする場合、StarRocks クラスタの FE と BE は HDFS クライアントを使用して HDFS クラスタにアクセスします。ほとんどの場合、この目的を達成するために StarRocks クラスタを構成する必要はありません。StarRocks はデフォルトの設定で HDFS クライアントを起動します。次の状況では、StarRocks クラスタを構成する必要があります。

  - HDFS クラスタで高可用性 (HA) が有効になっている場合: HDFS クラスタの **hdfs-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスと各 BE の **$BE_HOME/conf** パスに追加します。
  - HDFS クラスタで View File System (ViewFs) が有効になっている場合: HDFS クラスタの **core-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスと各 BE の **$BE_HOME/conf** パスに追加します。

> **注記**
>
> クエリを送信する際に不明なホストを示すエラーが返される場合は、HDFS クラスタノードのホスト名と IP アドレスのマッピングを **/etc/hosts** パスに追加する必要があります。

### Kerberos 認証

HDFS クラスタまたは Hive メタストアで Kerberos 認証が有効になっている場合、StarRocks クラスタを次のように構成してください。

- 各 FE と各 BE で `kinit -kt keytab_path principal` コマンドを実行して、Key Distribution Center (KDC) から Ticket Granting Ticket (TGT) を取得します。このコマンドを実行するには、HDFS クラスタと Hive メタストアにアクセスする権限が必要です。このコマンドを使用して KDC にアクセスする際は、時間に敏感です。そのため、このコマンドを定期的に実行するために cron を使用する必要があります。
- 各 FE の **$FE_HOME/conf/fe.conf** ファイルと各 BE の **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。この例では、`/etc/krb5.conf` は **krb5.conf** ファイルの保存パスです。必要に応じてパスを変更できます。

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

Delta Lake カタログの名前です。命名規則は次のとおりです。

- 名前には、文字、数字 (0-9)、およびアンダースコア (_) を含めることができます。ただし、文字で始める必要があります。
- 名前は大文字と小文字を区別し、長さが 1023 文字を超えることはできません。

#### comment

Delta Lake カタログの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。値を `deltalake` に設定します。

#### MetastoreParams

データソースのメタストアとの統合方法に関するパラメータのセットです。

##### Hive メタストア

データソースのメタストアとして Hive メタストアを選択した場合、`MetastoreParams` を次のように構成します。

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> Delta Lake データをクエリする前に、Hive メタストアノードのホスト名と IP アドレスのマッピングを `/etc/hosts` パスに追加する必要があります。そうしないと、クエリを開始するときに StarRocks が Hive メタストアにアクセスできなくなる場合があります。

次の表に、`MetastoreParams` で構成する必要のあるパラメータについて説明します。

| パラメータ名             | 必須 | 説明                                                         |
| ----------------------- | ---- | ------------------------------------------------------------ |
| hive.metastore.type     | Yes  | Delta Lake クラスタで使用するメタストアのタイプです。値を `hive` に設定します。 |
| hive.metastore.uris     | Yes  | Hive メタストアの URI です。形式: `thrift://<metastore_IP_address>:<metastore_port>`。<br />Hive メタストアで高可用性 (HA) が有効になっている場合、複数のメタストア URI を指定し、カンマ (`,`) で区切って指定できます。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"` のように指定します。 |

##### AWS Glue

データソースのメタストアとして AWS Glue を選択した場合（ストレージとして AWS S3 を選択した場合のみサポートされます）、次のいずれかの操作を実行します。

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams` を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 仮定されるロールベースの認証方法を選択する場合、`MetastoreParams` を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAM ユーザーベースの認証方法を選択する場合、`MetastoreParams` を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

次の表に、`MetastoreParams` で構成する必要のあるパラメータについて説明します。

| パラメータ名                     | 必須 | 説明                                                         |
| ------------------------------- | ---- | ------------------------------------------------------------ |
| hive.metastore.type             | Yes  | Delta Lake クラスタで使用するメタストアのタイプです。値を `glue` に設定します。 |
| aws.glue.use_instance_profile   | Yes  | インスタンスプロファイルベースの認証方法と仮定されるロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。 |
| aws.glue.iam_role_arn           | No   | AWS Glue データカタログに特権を持つ IAM ロールの ARN。AWS Glue へのアクセスに仮定されるロールベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.region                 | Yes  | AWS Glue データカタログが存在するリージョン。例: `us-west-1`。 |
| aws.glue.access_key             | No   | AWS IAM ユーザーのアクセスキー。AWS S3 へのアクセスに IAM ユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key             | No   | AWS IAM ユーザーのシークレットキー。AWS S3 へのアクセスに IAM ユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |

AWS Glue へのアクセス方法の選択および AWS IAM コンソールでのアクセス制御ポリシーの構成方法については、[AWS Glue へのアクセスのための認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocks がストレージシステムと統合する方法に関するパラメータのセットです。このパラメータセットはオプションです。

ストレージとして HDFS を使用する場合、`StorageCredentialParams` を構成する必要はありません。

ストレージとして AWS S3、他の S3 互換ストレージシステム、Microsoft Azure Storage、または Google GCS を使用する場合、`StorageCredentialParams` を構成する必要があります。

##### AWS S3

Delta Lake クラスタのストレージとして AWS S3 を選択した場合、次のいずれかの操作を実行します。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 仮定されるロールベースの認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM ユーザーベースの認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

| パラメータ名                   | 必須 | 説明                                                         |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile   | Yes  | インスタンスプロファイルベースの認証方法と仮定されるロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn           | No   | AWS S3 バケットに特権を持つ IAM ロールの ARN。AWS S3 へのアクセスに仮定されるロールベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.s3.region                 | Yes  | AWS S3 バケットが存在するリージョン。例: `us-west-1`。       |
| aws.s3.access_key             | No   | IAM ユーザーのアクセスキー。IAM ユーザーベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key             | No   | IAM ユーザーのシークレットキー。IAM ユーザーベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3 へのアクセス方法の選択および AWS IAM コンソールでのアクセス制御ポリシーの構成方法については、[AWS S3 へのアクセスのための認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3 互換ストレージシステム

Delta Lake カタログは、v2.5 以降で S3 互換ストレージシステム（たとえば、MinIO）をサポートしています。

S3 互換ストレージシステムをストレージとして選択した場合（たとえば、MinIO）、次のように `StorageCredentialParams` を構成して、正常な統合を確保してください。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

| パラメータ名                      | 必須 | 説明                                                         |
| -------------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | Yes  | SSL 接続を有効にするかどうかを指定します。<br />有効な値: `true`、`false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access  | Yes  | パス形式のアクセスを有効にするかどうかを指定します。<br />有効な値: `true`、`false`。デフォルト値: `false`。MinIO の場合、値を `true` に設定する必要があります。<br />パス形式の URL は次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、US West (Oregon) リージョンで `DOC-EXAMPLE-BUCKET1` というバケットを作成し、そのバケットの `alice.jpg` オブジェクトにアクセスする場合、次のパス形式の URL を使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | Yes  | AWS S3 ではなく、S3 互換ストレージシステムに接続するために使用されるエンドポイントです。 |
| aws.s3.access_key                | Yes  | IAM ユーザーのアクセスキーです。                             |
| aws.s3.secret_key                | Yes  | IAM ユーザーのシークレットキーです。                           |

##### Microsoft Azure Storage

Delta Lake カタログは、v3.0 以降で Microsoft Azure Storage をサポートしています。

###### Azure Blob Storage

Delta Lake クラスタのストレージとして Blob Storage を選択した場合、次のいずれかの操作を実行します。

- 共有キー認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**              | **必須** | **説明**                                                     |
  | ---------------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.storage_account   | Yes      | Blob Storage アカウントのユーザー名です。                      |
  | azure.blob.shared_key        | Yes      | Blob Storage アカウントの共有キーです。                        |

- SAS トークン認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**        | **必須** | **説明**                                                     |
  | ---------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.account_name | Yes      | Blob Storage アカウントのユーザー名です。                      |
  | azure.blob.container_name | Yes      | データを格納する Blob コンテナの名前です。                    |
  | azure.blob.sas_token    | Yes      | Blob Storage アカウントにアクセスするために使用される SAS トークンです。 |

###### Azure Data Lake Storage Gen1

Delta Lake クラスタのストレージとして Data Lake Storage Gen1 を選択した場合、次のいずれかの操作を実行します。

- Managed Service Identity 認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                            | **必須** | **説明**                                                     |
  | -------------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity     | Yes      | Managed Service Identity 認証方法を有効にするかどうかを指定します。値を `true` に設定します。 |

- Service Principal 認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                 | **必須** | **説明**                                                     |
  | ------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id    | Yes      | サービスプリンシパルのクライアント (アプリケーション) ID です。 |
  | azure.adls1.oauth2_credential   | Yes      | 作成時に生成された新しいクライアント (アプリケーション) シークレットの値です。 |
  | azure.adls1.oauth2_endpoint     | Yes      | サービスプリンシパルまたはアプリケーションの OAuth 2.0 トークンエンドポイント (v1) です。 |

###### Azure Data Lake Storage Gen2

Delta Lake クラスタのストレージとして Data Lake Storage Gen2 を選択した場合、次のいずれかの操作を実行します。

- Managed Identity 認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                             | **必須** | **説明**                                                     |
  | --------------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity        | Yes      | Managed Identity 認証方法を有効にするかどうかを指定します。値を `true` に設定します。 |
  | azure.adls2.oauth2_tenant_id                   | Yes      | アクセスするデータのテナントの ID です。                     |
  | azure.adls2.oauth2_client_id                   | Yes      | 管理対象 ID のクライアント (アプリケーション) ID です。        |

- Shared Key 認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**              | **必須** | **説明**                                                     |
  | ---------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.storage_account  | Yes      | Data Lake Storage Gen2 ストレージアカウントのユーザー名です。 |
  | azure.adls2.shared_key       | Yes      | Data Lake Storage Gen2 ストレージアカウントの共有キーです。   |

- Service Principal 認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                   | **必須** | **説明**                                                     |
  | ----------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id        | Yes      | サービスプリンシパルのクライアント (アプリケーション) ID です。 |
  | azure.adls2.oauth2_client_secret    | Yes      | 作成時に生成された新しいクライアント (アプリケーション) シークレットの値です。 |
  | azure.adls2.oauth2_client_endpoint  | Yes      | サービスプリンシパルまたはアプリケーションの OAuth 2.0 トークンエンドポイント (v1) です。 |

##### Google GCS

Delta Lake カタログは、v3.0 以降で Google GCS をサポートしています。

Delta Lake クラスタのストレージとして Google GCS を選択した場合、次のいずれかの操作を実行します。

- VM ベースの認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                              | **デフォルト値** | **値の例** | **説明**                                                     |
  | --------------------------------------------- | ---------------- | ----------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account     | false            | true        | Compute Engine にバインドされたサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択する場合、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>",
  ```

  次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                          | **デフォルト値** | **値の例**                                                | **説明**                                                     |
  | ---------------------------------------- | ---------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email             | ""               | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントの JSON ファイル作成時に生成されたメールアドレスです。 |
  | gcp.gcs.service_account_private_key_id    | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | サービスアカウントの JSON ファイル作成時に生成されたプライベートキー ID です。 |
  | gcp.gcs.service_account_private_key       | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウントの JSON ファイル作成時に生成されたプライベートキーです。 |

- 偽装ベースの認証方法を選択する場合、次のいずれかの操作を実行します。

  - VM インスタンスがサービスアカウントを偽装する場合:

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

    | **パラメータ名**                              | **デフォルト値** | **値の例** | **説明**                                                     |
    | --------------------------------------------- | ---------------- | ----------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account     | false            | true        | Compute Engine にバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account          | ""               | "hello"     | 偽装するサービスアカウントです。                              |

  - サービスアカウント（一時的にメタサービスアカウントと呼ばれる）が別のサービスアカウント（一時的にデータサービスアカウントと呼ばれる）を偽装する場合:

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表に、`StorageCredentialParams` で構成する必要のあるパラメータについて説明します。

    | **パラメータ名**                          | **デフォルト値** | **値の例**                                                | **説明**                                                     |
    | ---------------------------------------- | ---------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email             | ""               | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントの JSON ファイル作成時に生成されたメールアドレスです。 |
    | gcp.gcs.service_account_private_key_id    | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | サービスアカウントの JSON ファイル作成時に生成されたプライベートキー ID です。 |
    | gcp.gcs.service_account_private_key       | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウントの JSON ファイル作成時に生成されたプライベートキーです。 |
    | gcp.gcs.impersonation_service_account     | ""               | "hello"                                                  | 偽装するデータサービスアカウントです。                        |

#### MetadataUpdateParams

StarRocks が Delta Lake のキャッシュされたメタデータを更新する方法に関するパラメータのセットです。このパラメータセットはオプションです。

StarRocks はデフォルトで [自動非同期更新ポリシー](#appendix-understand-metadata-automatic-asynchronous-update) を実装しています。

ほとんどの場合、`MetadataUpdateParams` を無視し、このパラメータのデフォルト値を使用して、すぐに使用できるパフォーマンスを提供します。

ただし、Delta Lake のデータ更新の頻度が高い場合は、これらのパラメータを調整して自動非同期更新のパフォーマンスをさらに最適化することができます。

> **注記**
>
> ほとんどの場合、Delta Lakeデータが1時間以下の粒度で更新される場合、データの更新頻度は高いと見なされます。

| パラメーター                            | 必須     | 説明                                                         |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | いいえ   | StarRocksがDelta Lakeテーブルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。値 `true` はキャッシュを有効にし、値 `false` はキャッシュを無効にします。 |
| enable_remote_file_cache               | いいえ   | StarRocksがDelta Lakeテーブルまたはパーティションの基礎データファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。値 `true` はキャッシュを有効にし、値 `false` はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | いいえ   | StarRocksが自身にキャッシュされたDelta Lakeテーブルまたはパーティションのメタデータを非同期で更新する時間間隔。単位: 秒。デフォルト値: `7200` (2時間)。 |
| remote_file_cache_refresh_interval_sec | いいえ   | StarRocksが自身にキャッシュされたDelta Lakeテーブルまたはパーティションの基礎データファイルのメタデータを非同期で更新する時間間隔。単位: 秒。デフォルト値: `60`。 |
| metastore_cache_ttl_sec                | いいえ   | StarRocksが自動的に破棄する自身にキャッシュされたDelta Lakeテーブルまたはパーティションのメタデータの時間間隔。単位: 秒。デフォルト値: `86400` (24時間)。 |
| remote_file_cache_ttl_sec              | いいえ   | StarRocksが自動的に破棄する自身にキャッシュされたDelta Lakeテーブルまたはパーティションの基礎データファイルのメタデータの時間間隔。単位: 秒。デフォルト値: `129600` (36時間)。 |

### 例

以下の例では、Delta Lakeクラスターからデータをクエリするために、`deltalake_catalog_hms`または`deltalake_catalog_glue`という名前のDelta Lakeカタログを作成します。

#### HDFS

ストレージとしてHDFSを使用する場合、次のようなコマンドを実行します:

```SQL
CREATE EXTERNAL CATALOG deltalake_catalog_hms
PROPERTIES
(
    "type" = "deltalake",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
);
```

#### AWS S3

##### インスタンスプロファイルベースの認証情報を選択した場合

- Delta LakeクラスターでHiveメタストアを使用する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Delta LakeクラスターでAWS Glueを使用する場合、次のようなコマンドを実行します:

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

##### 偽装ロールベースの認証情報を選択した場合

- Delta LakeクラスターでHiveメタストアを使用する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Delta LakeクラスターでAWS Glueを使用する場合、次のようなコマンドを実行します:

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

##### IAMユーザーベースの認証情報を選択した場合

- Delta LakeクラスターでHiveメタストアを使用する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Delta LakeクラスターでAWS Glueを使用する場合、次のようなコマンドを実行します:

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

#### S3互換のストレージシステム

MinIOを例に使用します。次のようなコマンドを実行します:

```SQL
CREATE EXTERNAL CATALOG deltalake_catalog_hms
PROPERTIES
(
    "type" = "deltalake",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://34.132.15.127:9083",
    "aws.s3.enable_ssl" = "true",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```

#### Microsoft Azure Storage

##### Azure Blob Storage

- 共有キー認証方式を選択した場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SASトークン認証方式を選択した場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- マネージドサービスアイデンティティ認証方式を選択した場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- サービスプリンシパル認証方式を選択した場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- マネージドアイデンティティ認証方式を選択した場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 共有キー認証方式を選択した場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- サービスプリンシパル認証方式を選択した場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "glue",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMベースの認証方式を選択した場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- サービスアカウントベースの認証方式を選択した場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 偽装ベースの認証方式を選択した場合:

  - VMインスタンスがサービスアカウントを偽装する場合、次のようなコマンドを実行します:

    ```SQL
    CREATE EXTERNAL CATALOG deltalake_catalog_hms
    PROPERTIES
    (
        "type" = "deltalake",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - サービスアカウントが他のサービスアカウントを偽装する場合、次のようなコマンドを実行します:

    ```SQL
    CREATE EXTERNAL CATALOG deltalake_catalog_hms
    PROPERTIES
    (
        "type" = "deltalake",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Delta Lakeカタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスター内のすべてのカタログをクエリできます:

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリできます。次の例では、Delta Lakeカタログ`deltalake_catalog_glue`の作成ステートメントをクエリします:

```SQL
SHOW CREATE CATALOG deltalake_catalog_glue;
```

## Delta Lakeカタログとその中のデータベースに切り替える

Delta Lakeカタログとその中のデータベースに切り替えるには、次のいずれかの方法を使用できます:

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションでDelta Lakeカタログを指定し、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します:

  ```SQL
  -- 現在のセッションで指定のカタログに切り替える:
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定する:
  USE <db_name>
  ```

- 直接[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、Delta Lakeカタログとその中のデータベースに切り替えます:

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Delta Lakeカタログの削除

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、外部カタログを削除できます。

次の例では、Delta Lakeカタログ`deltalake_catalog_glue`を削除します:

```SQL
DROP Catalog deltalake_catalog_glue;
```

## Delta Lakeテーブルのスキーマの表示

次の構文のいずれかを使用して、Delta Lakeテーブルのスキーマを表示できます:

- スキーマの表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Delta Lakeテーブルのクエリ

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Delta Lakeクラスター内のデータベースを表示します:

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Delta Lakeカタログとその中のデータベースに切り替える](#delta-lakeカタログとその中のデータベースに切り替える)。

3. 指定したデータベース内の宛先テーブルをクエリするために[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用します:

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Delta Lakeからデータをロードする

OLAPテーブル`olap_tbl`があると仮定し、次のようにデータを変換してロードできます:

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM deltalake_table
```

## メタデータキャッシュの手動または自動更新

### 手動更新

デフォルトでは、StarRocksはDelta Lakeカタログのメタデータをキャッシュし、非同期モードでメタデータを更新してパフォーマンスを向上させます。さらに、スキーマの変更やテーブルの更新が行われた場合、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)を使用してメタデータを手動で更新し、StarRocksが最新のメタデータをできるだけ早く取得し、適切な実行計画を生成できるようにすることもできます:

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自動増分更新

自動増分更新は、StarRocksクラスターのFEがHiveメタストアからイベント（列の追加、パーティションの削除、データの更新など）を読み取ることができるようにするためのポリシーです。StarRocksはこれらのイベントに基づいてFEにキャッシュされたメタデータを自動的に更新できます。つまり、メタデータの手動更新は不要です。

自動増分更新を有効にするには、次の手順に従います:

#### ステップ1: Hiveメタストアにイベントリスナーを設定する

Hiveメタストアのバージョンv2.xおよびv3.xの両方は、イベントリスナーを設定することができます。この手順では、Hiveメタストアv3.1.2で使用されるイベントリスナーの設定を例に説明します。次の構成項目を**$HiveMetastore/conf/hive-site.xml**ファイルに追加し、Hiveメタストアを再起動します:

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

FEのログファイルで`event id`を検索して、イベントリスナーが正常に設定されているかどうかを確認できます。設定に失敗した場合、`event id`の値は`0`です。

#### ステップ2: StarRocksで自動増分更新を有効にする

StarRocksで単一のDelta LakeカタログまたはすべてのDelta Lakeカタログに対して自動増分更新を有効にすることができます。

- 単一のDelta Lakeカタログに対して自動増分更新を有効にするには、Delta Lakeカタログを作成する際に`enable_hms_events_incremental_sync`パラメーターを`true`に設定します:

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

- すべてのDelta Lakeカタログに対して自動増分更新を有効にするには、各FEの`$FE_HOME/conf/fe.conf`ファイルに`"enable_hms_events_incremental_sync" = "true"`を追加し、各FEを再起動してパラメーター設定を有効にします。

また、ビジネス要件に基づいて各FEの`$FE_HOME/conf/fe.conf`ファイルで次のパラメーターを調整し、各FEを再起動してパラメーター設定を有効にすることもできます。

| パラメーター                         | 説明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocksがHiveメタストアからイベントを読み取る時間間隔。デフォルト値: `5000`。単位: ミリ秒。 |
| hms_events_batch_size_per_rpc     | StarRocksが一度に読み取ることができるイベントの最大数。デフォルト値: `500`。 |
| enable_hms_parallel_process_evens | StarRocksがイベントを読み取りながらイベントを並列で処理するかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。値 `true` は並列処理を有効にし、値 `false` は並列処理を無効にします。 |
| hms_process_events_parallel_num   | StarRocksが並列で処理できるイベントの最大数。デフォルト値: `4`。 |

## 付録: メタデータの自動非同期更新の理解

自動非同期更新は、StarRocksがDelta Lakeカタログのメタデータを更新するために使用するデフォルトのポリシーです。

デフォルトでは（つまり、`enable_metastore_cache`パラメーターと`enable_remote_file_cache`パラメーターがともに`true`に設定されている場合）、クエリがDelta Lakeテーブルのパーティションにヒットすると、StarRocksはそのパーティションのメタデータとパーティションの基礎データファイルのメタデータを自動的にキャッシュします。キャッシュされたメタデータは遅延更新ポリシーを使用して更新されます。

例えば、`table2`という名前のDelta Lakeテーブルがあり、`p1`、`p2`、`p3`、`p4`の4つのパーティションがあります。クエリが`p1`にヒットし、StarRocksは`p1`のメタデータと`p1`の基礎データファイルのメタデータをキャッシュします。次のようなデフォルトの更新および破棄の時間間隔を仮定します:

- `p1`のキャッシュされたメタデータを非同期で更新する時間間隔（`metastore_cache_refresh_interval_sec`パラメーターで指定）は2時間です。
- `p1`の基礎データファイルのキャッシュされたメタデータを非同期で更新する時間間隔（`remote_file_cache_refresh_interval_sec`パラメーターで指定）は60秒です。
- `p1`のキャッシュされたメタデータを自動的に破棄する時間間隔（`metastore_cache_ttl_sec`パラメーターで指定）は24時間です。
- `p1`の基礎データファイルのキャッシュされたメタデータを自動的に破棄する時間間隔（`remote_file_cache_ttl_sec`パラメーターで指定）は36時間です。

以下の図は、理解しやすくするためにタイムライン上の時間間隔を示しています。

![メタデータの更新と破棄のタイムライン](../../assets/catalog_timeline.png)

その後、StarRocksは次のルールに従ってメタデータを更新または破棄します:

- もう一度`p1`にヒットするクエリがあり、最後の更新からの現在の時間が60秒未満の場合、StarRocksは`p1`のキャッシュされたメタデータまたは`p1`の基礎データファイルのキャッシュされたメタデータを更新しません。
- もう一度`p1`にヒットするクエリがあり、最後の更新からの現在の時間が60秒以上の場合、StarRocksは`p1`の基礎データファイルのキャッシュされたメタデータを更新します。
- もう一度`p1`にヒットするクエリがあり、最後の更新からの現在の時間が2時間以上の場合、StarRocksは`p1`のキャッシュされたメタデータを更新します。
- 最後の更新から24時間以内に`p1`がアクセスされない場合、StarRocksは`p1`のキャッシュされたメタデータを破棄します。メタデータは次のクエリでキャッシュされます。
- 最後の更新から36時間以内に`p1`がアクセスされない場合、StarRocksは`p1`の基礎データファイルのキャッシュされたメタデータを破棄します。メタデータは次のクエリでキャッシュされます。
