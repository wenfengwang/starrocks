```yaml
---
displayed_sidebar: "Japanese"
---

# デルタレイク カタログ

デルタレイクカタログは、インジェストなしでデルタレイクデータをクエリできるようにする外部カタログの一種です。

また、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して、デルタレイクカタログに基づいてデルタレイクからデータを直接変換およびロードすることができます。StarRocks は v2.5 以降でデルタレイクカタログをサポートしています。

デルタレイククラスターでの SQL ワークロードの正常な実行を保証するためには、StarRocks クラスターが次の 2 つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム (HDFS) または AWS S3、Microsoft Azure Storage、Google GCS、またはその他の S3 互換ストレージシステム（たとえば MinIO）などのオブジェクトストレージ

- Hive メタストアまたは AWS Glue などのメタストア

  > **注記**
  >
  > ストレージとして AWS S3 を選択した場合、HMS または AWS Glue をメタストアとして使用できます。その他のストレージシステムを選択した場合は、HMS しかメタストアとして使用できません。

## 使用上の注意事項

- StarRocks がサポートするデルタレイクのファイルフォーマットは Parquet です。Parquet ファイルは次の圧縮形式をサポートしています: SNAPPY、LZ4、ZSTD、GZIP、および NO_COMPRESSION。
- StarRocks がサポートしないデルタレイクのデータ型は、MAP および STRUCT です。

## 統合の準備

デルタレイクカタログを作成する前に、StarRocks クラスターがデルタレイククラスターのストレージシステムとメタストアと統合できることを確認してください。

### AWS IAM

デルタレイククラスターがストレージとして AWS S3 またはメタストアとして AWS Glue を使用している場合は、適切な認証メソッドを選択し、StarRocks クラスターが関連する AWS クラウドリソースにアクセスできるようにするために必要な準備を行ってください。

以下の認証メソッドが推奨されています。

- インスタンスプロファイル
- 仮定されたロール
- IAM ユーザー

上記の 3 つの認証メソッドのうち、インスタンスプロファイルが最も広く使用されています。

詳細は [AWS IAM での認証の準備](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam) を参照してください。

### HDFS

ストレージとして HDFS を選択した場合、StarRocks クラスターを次のように構成してください。

- (オプション) HDFS クラスターと Hive メタストアにアクセスするために使用されるユーザー名を設定します。デフォルトでは、StarRocks は FE プロセスと BE プロセスのユーザー名を使用して HDFS クラスターと Hive メタストアにアクセスします。`export HADOOP_USER_NAME="<user_name>"` を各 FE の **fe/conf/hadoop_env.sh** ファイルの先頭に追加し、各 BE の **be/conf/hadoop_env.sh** ファイルの先頭に追加してユーザー名を設定します。これらのファイルにユーザー名を設定した後は、各 FE および各 BE を再起動してパラメータ設定を有効にします。1 つの StarRocks クラスターにつき 1 つのユーザー名のみを設定できます。
- デルタレイクデータをクエリする際、StarRocks クラスターの FE および BE は HDFS クライアントを使用して HDFS クラスターにアクセスします。ほとんどの場合、この目的を達成するために StarRocks クラスターを構成する必要はありません。StarRocks はデフォルトの構成を使用して HDFS クライアントを開始します。次の状況の場合に限り、StarRocks クラスターを構成する必要があります。

  - HDFS クラスターに対してハイアベイラビリティ（HA）が有効になっている場合: HDFS クラスターの **hdfs-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに追加します。
  - HDFS クラスターに対して View File System (ViewFs) が有効になっている場合: HDFS クラスターの **core-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに追加します。

> **注記**
>
> クエリを送信すると不明なホストのエラーが返される場合は、**/etc/hosts** パスに HDFS クラスターノードのホスト名と IP アドレスのマッピングを追加する必要があります。

### ケルベロス認証

HDFS クラスターまたは Hive メタストアに対してケルベロス認証が有効になっている場合は、StarRocks クラスターを次のように構成してください。

- 各 FE および各 BE で `kinit -kt keytab_path principal` コマンドを実行し、Key Distribution Center (KDC) から Ticket Granting Ticket (TGT) を取得します。このコマンドを実行するためには、HDFS クラスターと Hive メタストアにアクセスする権限が必要です。このコマンドを使用して KDC にアクセスする時間に制限があるため、このコマンドを定期的に実行するために cron を使用する必要があります。
- 各 FE の **$FE_HOME/conf/fe.conf** ファイルおよび各 BE の **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。この例では、`/etc/krb5.conf` は **krb5.conf** ファイルの保存パスです。必要に応じてパスを変更できます。

## デルタレイク カタログの作成

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

デルタレイクカタログの名前です。以下の命名規則が適用されます:

- 名前には、文字、数字（0-9）、およびアンダースコア（_）を含めることができます。ただし、文字で始める必要があります。
- 名前は大文字と小文字を区別し、長さを 1023 文字を超えることはできません。

#### comment

デルタレイクカタログの説明です。このパラメータはオプションです。

#### type

データソースの種類です。値を `deltalake` に設定します。

#### MetastoreParams

データソースのメタストアとの統合に関するパラメータのセットです。

##### Hive メタストア

データソースのメタストアとして Hive メタストアを選択した場合、`MetastoreParams` を次のように構成します:

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> デルタレイクデータをクエリする前に、Hive メタストアノードのホスト名と IP アドレスのマッピングを **/etc/hosts** パスに追加する必要があります。そうしないと、クエリを開始すると StarRocks が Hive メタストアにアクセスできない場合があります。

以下の表は、`MetastoreParams` で構成する必要があるパラメータについて説明します。

| パラメータ           | 必須     | 説明                                                        |
| ------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type | はい     | デルタレイククラスターで使用するメタストアの種類。値を `hive` に設定します。 |
| hive.metastore.uris | はい     | Hive メタストアの URI。形式: `thrift://<metastore_IP_address>:<metastore_port>`。<br />Hive メタストアに対してハイアベイラビリティ（HA）が有効になっている場合、複数のメタストア URI を指定し、コンマ（`,`）で区切って指定します。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

データソースのメタストアとして AWS Glue を選択し、ストレージとして AWS S3 を選択した場合は、次のいずれかのアクションを実行します:

- インスタンスプロファイルベースの認証メソッドを選択する場合は、`MetastoreParams` を次のように構成します:

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 仮定されたロールベースの認証メソッドを選択する場合は、`MetastoreParams` を次のように構成します:

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAM ユーザーベースの認証メソッドを選択する場合は、`MetastoreParams` を次のように構成します:

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

以下の表は、`MetastoreParams` で構成する必要があるパラメータについて説明します。

| パラメータ                     | 必須     | 説明                                                        |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type           | はい     | デルタレイククラスターで使用するメタストアの種類。値を `glue` に設定します。 |
| aws.glue.use_instance_profile | はい     | インスタンスプロファイルベースの認証メソッドと仮定されたロールベースの認証メソッドを有効にするかどうかを指定します。有効な値は `true` および `false` です。デフォルト値は `false` です。 |
```
| aws.glue.iam_role_arn         | No       | AWS Glueデータカタログにアクセス権限を持つIAMロールのARN。AWS Glueへのアクセスに前提となるロールベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.region               | Yes      | AWS Glueデータカタログが存在するリージョン。例: `us-west-1`。 |
| aws.glue.access_key           | No       | AWS IAMユーザーのアクセスキー。AWS GlueへのアクセスにIAMユーザーに基づく認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | No       | AWS IAMユーザーのシークレットキー。AWS GlueへのアクセスにIAMユーザーに基づく認証方法を使用する場合は、このパラメータを指定する必要があります。 |

AWS Glueへのアクセス認証方法の選択とAWS IAMコンソールでのアクセス制御ポリシーの設定方法については、[AWS Glueへのアクセス認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocksがストレージシステムと統合する方法に関するパラメータのセット。このパラメータセットはオプションです。

ストレージとしてHDFSを使用する場合、`StorageCredentialParams`を構成する必要はありません。

AWS S3、他のS3互換ストレージシステム、Microsoft Azure Storage、またはGoogle GCSを使用する場合は、`StorageCredentialParams`を構成する必要があります。

##### AWS S3

Delta LakeクラスタのストレージとしてAWS S3を選択する場合、次のいずれかのアクションを実行してください。

- インスタンスプロファイルに基づく認証方法を選択する場合は、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 割り当てられたロールに基づく認証方法を選択する場合は、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーに基づく認証方法を選択する場合は、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

`StorageCredentialParams`で構成する必要があるパラメータに関する情報は、次の表に記載されています。

| パラメータ                   | 必須     | 説明                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes      | インスタンスプロファイルに基づく認証方法と割り当てられたロールに基づく認証方法を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | No       | AWS S3バケットに権限を持つIAMロールのARN。AWS S3へのアクセスにロールベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.region               | Yes      | AWS S3バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | No       | IAMユーザーのアクセスキー。IAMユーザーに基づく認証方法を使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | No       | IAMユーザーのシークレットキー。IAMユーザーに基づく認証方法を使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。 |

AWS S3へのアクセス認証方法の選択とAWS IAMコンソールでのアクセス制御ポリシーの設定方法については、[AWS S3へのアクセス認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

Delta Lake catalogsはv2.5以降、S3互換ストレージシステムをサポートしています。

MinIOなどのS3互換ストレージシステムをストレージとして選択する場合、正常な統合を確保するために、以下のように`StorageCredentialParams`を構成してください:

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

`StorageCredentialParams`で構成する必要があるパラメータに関する情報は、次の表に記載されています。

| パラメータ                        | 必須      | 説明                                                  |
| -------------------------------- | ----------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | Yes       | SSL接続を有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access  | Yes       | パス形式のアクセスを有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値: `false`。MinIOの場合は、値を`true`に設定する必要があります。<br />パス形式URLは以下の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、米国西部（オレゴン）リージョンで `DOC-EXAMPLE-BUCKET1` という名前のバケットを作成し、そのバケット内の `alice.jpg` オブジェクトにアクセスしたい場合は、次のようなパス形式URLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | Yes       | AWS S3の代わりにS3互換ストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key                | Yes       | IAMユーザーのアクセスキー。 |
| aws.s3.secret_key                | Yes       | IAMユーザーのシークレットキー。 |

##### Microsoft Azure Storage

Delta Lake catalogsはv3.0以降、Microsoft Azure Storageをサポートしています。

###### Azure Blob Storage

Delta LakeクラスタのストレージとしてBlob Storageを選択する場合、次のいずれかのアクションを実行してください:

- Shared Key認証メソッドを選択する場合は、`StorageCredentialParams`を以下のように構成してください:

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams`で構成する必要があるパラメータに関する情報は、次の表に記載されています。

  | **パラメータ**              | **必須** | **説明**                               |
  | -------------------------- | ----------- | ------------------------------------------ |
  | azure.blob.storage_account | Yes       | Blob Storageアカウントのユーザー名。     |
  | azure.blob.shared_key      | Yes       | Blob Storageアカウントの共有キー。     |

- SASトークン認証メソッドを選択する場合は、`StorageCredentialParams`を以下のように構成してください:

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams`で構成する必要があるパラメータに関する情報は、次の表に記載されています。

  | **パラメータ**             | **必須** | **説明**                                              |
  | -------------------------- | ----------- | ------------------------------------------ |
  | azure.blob.account_name   | Yes       | Blob Storageアカウントのユーザー名。             |
  | azure.blob.container_name | Yes       | データを格納するblobコンテナの名前。        |
  | azure.blob.sas_token      | Yes       | Blob Storageアカウントにアクセスするために使用されるSASトークン。 |

###### Azure Data Lake Storage Gen1

Delta LakeクラスタのストレージとしてData Lake Storage Gen1を選択する場合、次のいずれかのアクションを実行してください:

- Managed Service Identity認証メソッドを選択する場合は、`StorageCredentialParams`を以下のように構成してください:

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams`で構成する必要があるパラメータに関する情報は、次の表に記載されています。

  | **パラメータ**                            | **必須** | **説明**                                              |
  | ---------------------------------------- | ----------- | ------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes       | Managed Service Identity認証メソッドを有効にするかどうかを指定します。値を `true` に設定します。 |

- Service Principal認証メソッドを選択する場合は、`StorageCredentialParams`を以下のように構成してください:

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams`で構成する必要があるパラメータに関する情報は、次の表に記載されています。

  | **パラメータ**                 | **必須** | **説明**                                              |
  | ----------------------------- | ----------- | ------------------------------------------ |
  | azure.adls1.oauth2_client_id  | Yes       | Service Principalのクライアント（アプリケーション）ID。       |
  | azure.adls1.oauth2_credential | Yes       | 作成された新しいクライアント（アプリケーション）シークレットの値。 |
  | azure.adls1.oauth2_endpoint   | Yes       | サービス プリンシパルまたはアプリケーションのOAuth 2.0トークン エンドポイント(v1)。 |

###### Azure Data Lake Storage Gen2
データ レイク ストレージ Gen2 を Delta Lake クラスターのストレージとして選択した場合は、次のいずれかのアクションを実行します。

- マネージド ID 認証メソッドを選択する場合は、`StorageCredentialParams` を次のように設定します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  次の表は、`StorageCredentialParams` で構成する必要のあるパラメータを説明します。

  | **パラメーター**                       | **必須** | **説明**                                           |
  | ----------------------------------- | -------- | -------------------------------------------------- |
  | azure.adls2.oauth2_use_managed_identity | Yes      | マネージド ID 認証メソッドを有効にするかどうかを指定します。値を `true` に設定します。 |
  | azure.adls2.oauth2_tenant_id            | Yes      | アクセスするデータのテナントの ID。               |
  | azure.adls2.oauth2_client_id            | Yes      | マネージド ID のクライアント (アプリケーション) ID。 |

- 共有キー認証メソッドを選択する場合は、`StorageCredentialParams` を次のように設定します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次の表は、`StorageCredentialParams` で構成する必要のあるパラメータを説明します。

  | **パラメーター**               | **必須** | **説明**                                           |
  | ----------------------------- | -------- | -------------------------------------------------- |
  | azure.adls2.storage_account    | Yes      | Data Lake Storage Gen2 ストレージ アカウントのユーザー名。 |
  | azure.adls2.shared_key         | Yes      | Data Lake Storage Gen2 ストレージ アカウントの共有キー。 |

- サービス プリンシパル認証メソッドを選択する場合は、`StorageCredentialParams` を次のように設定します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  次の表は、`StorageCredentialParams` で構成する必要のあるパラメータを説明します。

  | **パラメーター**                 | **必須** | **説明**                                           |
  | ----------------------------- | -------- | -------------------------------------------------- |
  | azure.adls2.oauth2_client_id  | Yes      | サービス プリンシパルのクライアント (アプリケーション) ID。 |
  | azure.adls2.oauth2_client_secret | Yes      | 作成した新しいクライアント (アプリケーション) シークレットの値。 |
  | azure.adls2.oauth2_client_endpoint | Yes      | サービス プリンシパルまたはアプリケーションの OAuth 2.0 トークン エンドポイント (v1)。 |

##### Google GCS

Delta Lake カタログは、v3.0 以降から Google GCS をサポートしています。

Delta Lake クラスターのストレージとして Google GCS を選択した場合は、次のいずれかのアクションを実行します。

- VM ベースの認証メソッドを選択する場合は、`StorageCredentialParams` を次のように設定します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次の表は、`StorageCredentialParams` で構成する必要のあるパラメータを説明します。

  | **パラメーター**                              | **デフォルト値** | **値 の例**           | **説明**                                           |
  | ------------------------------------------ | ----------------- | --------------------- | -------------------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | 指定された Compute Engine にバインドされたサービス アカウントを直接使用するかどうかを指定します。 |

- サービス アカウント ベースの認証メソッドを選択する場合は、`StorageCredentialParams` を次のように設定します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>",
  ```

  次の表は、`StorageCredentialParams` で構成する必要のあるパラメータを説明します。

  | **パラメーター**                         | **デフォルト値** | **値 の例**                                        | **説明**                                           |
  | ------------------------------------- | ----------------- | -------------------------------------------------- | -------------------------------------------------- |
  | gcp.gcs.service_account_email         | ""                | "user@hello.iam.gserviceaccount.com"               | サービス アカウント作成時に生成される JSON ファイル内のメールアドレス。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"         | サービス アカウント作成時に生成される JSON ファイル内のプライベート キー ID。 |
  | gcp.gcs.service_account_private_key   | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービス アカウント作成時に生成される JSON ファイル内のプライベート キー。 |

- インパーソネーション ベースの認証メソッドを選択する場合は、`StorageCredentialParams` を次のように設定します。

  - VM インスタンスをサービス アカウントになりすませる場合:

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次の表は、`StorageCredentialParams` で構成する必要のあるパラメータを説明します。

    | **パラメーター**                              | **デフォルト値** | **値 の例**           | **説明**                                           |
    | ------------------------------------------ | ----------------- | --------------------- | -------------------------------------------------- |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | 指定された Compute Engine にバインドされたサービス アカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | なりすますことを望むサービス アカウント。            |

  - サービス アカウント (一時的に meta サービス アカウントと呼ばれます) が別のサービス アカウント (一時的にデータ サービス アカウントと呼ばれます) になりすます場合:

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表は、`StorageCredentialParams` で構成する必要のあるパラメータを説明します。

    | **パラメーター**                          | **デフォルト値** | **値 の例**                                        | **説明**                                           |
    | -------------------------------------- | ----------------- | -------------------------------------------------- | -------------------------------------------------- |
    | gcp.gcs.service_account_email          | ""                | "user@hello.iam.gserviceaccount.com"               | meta サービス アカウント作成時に生成される JSON ファイル内の電子メール アドレス。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"         | meta サービス アカウント作成時に生成される JSON ファイル内のプライベート キー ID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | meta サービス アカウント作成時に生成される JSON ファイル内のプライベート キー。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                              | なりすますことを望むデータ サービス アカウント。     |

#### MetadataUpdateParams

StarRocks が Delta Lake のキャッシュ メタデータを更新する方法に関するパラメーターのセットです。このパラメーター セットはオプションです。

StarRocks はデフォルトで [自動非同期更新ポリシー](#appendix-understand-metadata-automatic-asynchronous-update) を実装します。

ほとんどの場合、`MetadataUpdateParams` を無視し、それに含まれるポリシー パラメーターを調整する必要はありません。なぜなら、これらのパラメーターのデフォルト値は、すでに使えるパフォーマンスを提供しているからです。

ただし、Delta Lake のデータ更新頻度が高い場合は、これらのパラメーターを調整して自動非同期更新のパフォーマンスをさらに最適化できます。

> **注記**
>
> Delta Lake のデータ更新頻度が 1 時間以下である場合、ほとんどの場合、データ更新頻度が高いと見なされます。

| パラメーター                                  | 必須 | 説明                                                     |
|---------------------------------------------| ---- | --------------------------------------------------------- |
| enable_metastore_cache                       | No   | StarRocks が Delta Lake テーブルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。 `true` ではキャッシュが有効になり、`false` ではキャッシュが無効になります。 |
| enable_remote_file_cache                     | No   | StarRocks が Delta Lake テーブルまたはパーティションの基礎データ ファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。 `true` ではキャッシュが有効になり、`false` ではキャッシュが無効になります。 |
| metastore_cache_refresh_interval_sec         | No   | StarRocks が自身でキャッシュされた Delta Lake テーブルまたはパーティションのメタデータを非同期に更新する時間間隔。単位: 秒。デフォルト値: `7200` (2 時間)。 |
| remote_file_cache_refresh_interval_sec | いいえ | StarRocksがデルタレイクテーブルまたはパーティションのメタデータを非同期で更新するタイムインターバルです。単位：秒。デフォルト値： `60`。 |
| metastore_cache_ttl_sec | いいえ | StarRocksが自動的に削除するデルタレイクテーブルまたはパーティションのメタデータのタイムインターバルです。単位：秒。デフォルト値： `86400`、つまり24時間です。 |
| remote_file_cache_ttl_sec | いいえ | StarRocksが自動的に削除するデルタレイクテーブルまたはパーティションの基礎データファイルのメタデータのタイムインターバルです。単位：秒。デフォルト値： `129600`、つまり36時間です。 |

### 例

以下の例では、Delta Lakeクラスタからデータをクエリするために、使用するメタストアの種類に応じて、`deltalake_catalog_hms`または`deltalake_catalog_glue`というDelta Lakeカタログを作成します。

#### HDFS

ストレージとしてHDFSを使用する場合、次のようにコマンドを実行します。

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

##### インスタンスプロファイルベースの認証を選択した場合

- Delta LakeクラスタでHiveメタストアを使用する場合、次のようにコマンドを実行します。

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

- Amazon EMR Delta LakeクラスタでAWS Glueを使用する場合、次のようにコマンドを実行します。

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

##### 仮定されたロールベースの認証を選択した場合

- Delta LakeクラスタでHiveメタストアを使用する場合、次のようにコマンドを実行します。

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

- Amazon EMR Delta LakeクラスタでAWS Glueを使用する場合、次のようにコマンドを実行します。

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

- Delta LakeクラスタでHiveメタストアを使用する場合、次のようにコマンドを実行します。

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

- Amazon EMR Delta LakeクラスタでAWS Glueを使用する場合、次のようにコマンドを実行します。

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

MinIOを例にして使用する場合、次のようにコマンドを実行します。

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

- 共有キー認証メソッドを選択した場合、次のようにコマンドを実行します。

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

- SASトークン認証メソッドを選択した場合、次のようにコマンドを実行します。

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

- Managed Service Identity認証メソッドを選択した場合、次のようにコマンドを実行します。

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

- Service Principal認証メソッドを選択した場合、次のようにコマンドを実行します。

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

- Managed Identity認証メソッドを選択した場合、次のようにコマンドを実行します。

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

- もしShared Key認証方式を選択する場合、以下のようなコマンドを実行してください：

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

- もしService Principal認証方式を選択する場合、以下のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- もしVMベースの認証方式を選択する場合、以下のようなコマンドを実行してください：

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

- もしサービスアカウントベースの認証方式を選択する場合、以下のようなコマンドを実行してください：

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

- もし偽装ベースの認証方式を選択する場合：

  - もしVMインスタンスがサービスアカウントを偽装する場合、以下のようなコマンドを実行してください：

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

  - もしサービスアカウントが別のサービスアカウントを偽装する場合、以下のようなコマンドを実行してください：

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

現在のStarRocksクラスター内のすべてのカタログをクエリするには、[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) を使用してください：

```SQL
SHOW CATALOGS;
```

また、指定されたDelta Lakeカタログの作成ステートメントをクエリするには、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を使用してください。次の例は、`deltalake_catalog_glue` という名前のDelta Lakeカタログの作成ステートメントをクエリしています：

```SQL
SHOW CREATE CATALOG deltalake_catalog_glue;
```

## Delta Lakeカタログとその中のデータベースに切り替える

Delta Lakeカタログとその中のデータベースに切り替えるために、次のいずれかの方法を使用できます：

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) を使用して、現在のセッションでDelta Lakeカタログを指定し、その後[USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用してアクティブなデータベースを指定します：

  ```SQL
  -- 現在のセッションで指定したカタログに切り替えます：
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定します：
  USE <db_name>
  ```

- 直接に[USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、Delta Lakeカタログとその中のデータベースに切り替えます：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Delta Lakeカタログを削除する

外部カタログを削除するには、[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) を使用できます。

次の例では、`deltalake_catalog_glue` という名前のDelta Lakeカタログを削除しています：

```SQL
DROP Catalog deltalake_catalog_glue;
```

## Delta Lakeテーブルのスキーマを表示する

Delta Lakeテーブルのスキーマを表示するために、次の構文のいずれかを使用できます：

- スキーマを表示する

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Delta Lakeテーブルをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を使用して、Delta Lakeクラスター内のデータベースを表示してください：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Delta Lakeカタログとその中のデータベースに切り替える](#delta-lakeカタログとその中のデータベースに切り替える)。

3. 指定されたデータベース内の宛先テーブルをクエリするために[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用してください：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Delta Lakeからデータをロードする

`olap_tbl` という名前のOLAPテーブルがある場合、以下のようにデータを変換してロードできます：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM deltalake_table
```

## メタデータキャッシュを手動または自動的に更新する

### 手動更新

デフォルトでは、StarRocksはDelta Lakeのメタデータをキャッシュし、パフォーマンスを向上させるために非同期モードでメタデータを自動的に更新します。また、Delta Lakeテーブルでスキーマの変更やテーブルの更新が行われた場合は、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) を使用して、メタデータを手動で更新することもできます。これにより、StarRocksが最新のメタデータを取得し、適切な実行プランを生成できるようになります：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自動増分更新

自動的な非同期更新ポリシーとは異なり、自動的な増分更新ポリシーは、StarRocksクラスター内のFEがHiveメタストアから列を追加したりパーティションを削除したり、データを更新するなどのイベントを自動的に読み取ることができるようにします。これにより、StarRocksはこれらのイベントに基づいてFEにキャッシュされたメタデータを自動的に更新することができます。つまり、Delta Lakeテーブルのメタデータを手動で更新する必要はありません。

自動的な増分更新を有効にするために、次の手順に従ってください：

#### ステップ1：Hiveメタストアのイベントリスナーを構成する

Hiveメタストアv2.xおよびv3.xの両方でイベントリスナーを構成することができます。この手順では、Hiveメタストアv3.1.2で使用されるイベントリスナーの構成を例として示します。次の構成項目を **$HiveMetastore/conf/hive-site.xml** ファイルに追加し、その後Hiveメタストアを再起動してください：

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
```xml
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

FEログファイルで`event id`を検索して、イベントリスナーが正常に構成されたかどうかを確認できます。構成に失敗した場合、`event id`の値は`0`です。

#### ステップ2：StarRocksでの自動増分更新の有効化

StarRocksでは、単一のDelta LakeカタログまたはStarRocksクラスタ内のすべてのDelta Lakeカタログに対して自動増分更新を有効にすることができます。

- 単一のDelta Lakeカタログで自動増分更新を有効にするには、Delta Lakeカタログを作成する際に、以下のように`enable_hms_events_incremental_sync`パラメータを`true`に設定します。

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

- すべてのDelta Lakeカタログで自動増分更新を有効にするには、各FEの`$FE_HOME/conf/fe.conf`ファイルに`"enable_hms_events_incremental_sync" = "true"`を追加し、その後、各FEを再起動してパラメータ設定を有効にします。

また、以下のパラメータをビジネス要件に基づいて各FEの`$FE_HOME/conf/fe.conf`ファイルで調整し、その後、各FEを再起動してパラメータ設定を有効にします。

| Parameter                         | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocksがHiveメタストアからイベントを読み取る時間間隔。デフォルト値：`5000`。単位：ミリ秒。 |
| hms_events_batch_size_per_rpc     | StarRocksが一度に読み取ることができる最大イベント数。デフォルト値：`500`。 |
| enable_hms_parallel_process_evens | StarRocksがイベントを読み取りながら並列で処理するかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`true`。`true`は並列処理を有効にし、`false`は無効にします。 |
| hms_process_events_parallel_num   | StarRocksが並列で処理できる最大イベント数。デフォルト値：`4`。 |

## 付録：メタデータの自動非同期更新の理解

自動非同期更新は、StarRocksがDelta Lakeカタログのメタデータを更新するために使用するデフォルトポリシーです。

デフォルトでは（つまり、`enable_metastore_cache`および`enable_remote_file_cache`パラメータがともに`true`に設定されている場合）、クエリがDelta Lakeテーブルのパーティションにヒットすると、StarRocksは自動的にパーティションのメタデータとパーティションの基礎データファイルのメタデータをキャッシュします。キャッシュされたメタデータは遅延更新ポリシーを使用して更新されます。

例えば、`table2`という名前のDelta Lakeテーブルがあり、`p1`、`p2`、`p3`、`p4`の4つのパーティションがあるとします。クエリが`p1`にヒットし、StarRocksは`p1`のメタデータと`p1`の基礎データファイルのメタデータをキャッシュします。デフォルトのメタデータの更新および破棄の時間間隔は次のとおりと仮定します。

- `p1`のキャッシュされたメタデータを非同期に更新する時間間隔（`metastore_cache_refresh_interval_sec`パラメータによって指定）は2時間です。
- `p1`の基礎データファイルのキャッシュされたメタデータを非同期に更新する時間間隔（`remote_file_cache_refresh_interval_sec`パラメータによって指定）は60秒です。
- `p1`のキャッシュされたメタデータを自動的に破棄する時間間隔（`metastore_cache_ttl_sec`パラメータによって指定）は24時間です。
- `p1`の基礎データファイルのキャッシュされたメタデータを自動的に破棄する時間間隔（`remote_file_cache_ttl_sec`パラメータによって指定）は36時間です。

次の図は、理解を容易にするためにタイムライン上の時間間隔を示しています。

![更新およびキャッシュされたメタデータの破棄のためのタイムライン](../../assets/catalog_timeline.png)

その後、StarRocksは以下のルールに従ってメタデータを更新または破棄します。

- もう一度`p1`にクエリがヒットし、前回の更新からの現在の時間が60秒未満の場合、StarRocksは`p1`のキャッシュされたメタデータまたは`p1`の基礎データファイルのキャッシュされたメタデータを更新しません。
- もう一度`p1`にクエリがヒットし、前回の更新からの現在の時間が60秒を超える場合、StarRocksは`p1`の基礎データファイルのキャシュされたメタデータを更新します。
- もう一度`p1`にクエリがヒットし、前回の更新からの現在の時間が2時間を超える場合、StarRocksは`p1`のキャシュされたメタデータを更新します。
- 最終更新から24時間以内に`p1`がアクセスされない場合、StarRocksは`p1`のキャッシュされたメタデータを破棄します。メタデータは次のクエリでキャッシュされます。
- 最終更新から36時間以内に`p1`にアクセスされない場合、StarRocksは`p1`の基礎データファイルのキャッシュされたメタデータを破棄します。メタデータは次のクエリでキャッシュされます。