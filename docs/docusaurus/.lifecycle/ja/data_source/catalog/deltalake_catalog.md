```yaml
---
displayed_sidebar: "Japanese"
---

# Delta Lake カタログ

Delta Lake カタログは、データの取り込みなしに Delta Lake のデータをクエリできる外部カタログの一種です。

また、Delta Lake カタログに基づいて [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して、直接 Delta Lake からデータを変換およびロードすることができます。StarRocks は、v2.5 以降で Delta Lake カタログをサポートしています。

Delta Lake クラスタでの SQL ワークロードの成功を保証するために、StarRocks クラスタは次の 2 つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム (HDFS) または AWS S3、Microsoft Azure Storage、Google GCS、またはその他の S3 互換ストレージシステム (例: MinIO) のようなオブジェクトストレージ
- Hive メタストアまたは AWS Glue のようなメタストア

  > **注記**
  >
  > ストレージとして AWS S3 を選択した場合、HMS または AWS Glue をメタストアとして使用できます。その他のストレージシステムを選択した場合、メタストアとしては HMS のみを使用できます。

## 使用上の注意事項

- StarRocks がサポートする Delta Lake のファイル形式は Parquet です。Parquet ファイルは以下の圧縮形式をサポートしています: SNAPPY、LZ4、ZSTD、GZIP、および NO_COMPRESSION。
- StarRocks がサポートしない Delta Lake のデータ型は MAP と STRUCT です。

## 統合の準備

Delta Lake カタログを作成する前に、Delta Lake クラスタのストレージシステムおよびメタストアと StarRocks クラスタが統合できることを確認してください。

### AWS IAM

Delta Lake クラスタで AWS S3 をストレージまたは AWS Glue をメタストアとして使用する場合、適切な認証方法を選択し、必要な準備を行って、StarRocks クラスタが関連する AWS クラウドリソースにアクセスできるようにします。

以下の認証方法を推奨しています:

- インスタンスプロファイル
- 仮定されたロール
- IAM ユーザー

上記の 3 つの認証方法のうち、インスタンスプロファイルが最も広く使用されています。

詳細については、[AWS IAM での認証の準備](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)を参照してください。

### HDFS

ストレージとして HDFS を選択した場合、StarRocks クラスタを以下のように構成します。

- (オプション) HDFS クラスタと Hive メタストアにアクセスするために使用されるユーザー名を設定します。デフォルトでは、StarRocks は FE プロセスと BE プロセスのユーザー名を使用して HDFS クラスタと Hive メタストアにアクセスします。各 FE の **fe/conf/hadoop_env.sh** ファイルと各 BE の **be/conf/hadoop_env.sh** ファイルの先頭に `export HADOOP_USER_NAME="<ユーザー名>"` を追加してユーザー名を設定できます。これらのファイルにユーザー名を設定した後、各 FE および各 BE を再起動してパラメータ設定が有効になります。StarRocks クラスタには 1 つのユーザー名のみを設定できます。
- Delta Lake データをクエリする際、StarRocks クラスタの FE および BE は HDFS クライアントを使用して HDFS クラスタにアクセスします。ほとんどの場合、この目的を達成するために StarRocks クラスタを構成する必要はありません。StarRocks はデフォルトの構成を使用して HDFS クライアントを起動します。ただし、次の状況では StarRocks クラスタを構成する必要があります:

  - HDFS クラスタの高可用性 (HA) が有効になっている場合: HDFS クラスタの **hdfs-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに追加します。
  - HDFS クラスタのビューファイルシステム (ViewFs) が有効になっている場合: HDFS クラスタの **core-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに追加します。

> **注記**
>
> クエリを送信する際に不明なホストのエラーが返される場合、HDFS クラスタノードのホスト名と IP アドレスのマッピングを **/etc/hosts** パスに追加する必要があります。

### Kerberos 認証

HDFS クラスタまたは Hive メタストアで Kerberos 認証が有効になっている場合、StarRocks クラスタを以下のように構成します。

- 各 FE および各 BE で `kinit -kt keytab_path principal` コマンドを実行し、Key Distribution Center (KDC) から Ticket Granting Ticket (TGT) を取得します。このコマンドを実行するには、HDFS クラスタと Hive メタストアにアクセスする権限が必要です。このコマンドで KDC にアクセスする際は時間が重要です。そのため、定期的にこのコマンドを実行するために cron を使用する必要があります。
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

Delta Lake カタログの名前。以下の命名規則が適用されます:

- 名前には、英字、数字 (0-9)、アンダースコア (_) を含めることができます。英字で始まる必要があります。
- 名前は大文字と小文字を区別し、1023 文字を超えることはできません。

#### comment

Delta Lake カタログの説明。このパラメータはオプションです。

#### type

データソースのタイプ。`deltalake` と設定します。

#### MetastoreParams

データソースのメタストアとの統合方法に関するパラメーターのセットです。

##### Hive メタストア

データソースのメタストアとして Hive メタストアを選択した場合、`MetastoreParams` を以下のように構成します:

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> Delta Lake データをクエリする前に、Hive メタストアノードのホスト名と IP アドレスのマッピングを **/etc/hosts** パスに追加する必要があります。そうしないと、クエリを開始する際に StarRocks が Hive メタストアにアクセスできなくなる可能性があります。

次の表は、`MetastoreParams` で構成する必要のあるパラメーターについて説明しています。

| パラメーター           | 必須 | 説明                                                  |
| ------------------- | ---- | ---------------------------------------------------- |
| hive.metastore.type | はい | Delta Lake クラスタで使用するメタストアのタイプを設定します。値を `hive` に設定します。 |
| hive.metastore.uris | はい | Hive メタストアの URI。形式: `thrift://<metastore_IP_address>:<metastore_port>`。<br />Hive メタストアの高可用性 (HA) が有効になっている場合、複数のメタストア URI を指定し、コンマ (`,`) で区切ります。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

ストレージとして AWS S3 を選択した場合にのみサポートされるデータソースのメタストアとして AWS Glue を選択した場合、次のいずれかのアクションを実行してください:

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams` を以下のように構成します:

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 仮定されたロールベースの認証方法を選択する場合、`MetastoreParams` を以下のように構成します:

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAM ユーザーベースの認証方法を選択する場合、`MetastoreParams` を以下のように構成します:

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

次の表は、`MetastoreParams` で構成する必要のあるパラメーターについて説明しています。

| パラメーター                     | 必須 | 説明                                                  |
| ----------------------------- | ---- | ---------------------------------------------------- |
| hive.metastore.type           | はい | Delta Lake クラスタで使用するメタストアのタイプを設定します。値を `glue` に設定します。 |
| aws.glue.use_instance_profile | はい | インスタンスプロファイルベースの認証方法および仮定されたロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。 |
```
| aws.glue.iam_role_arn         | No       | AWS Glue Data Catalog に対する特権を持つ IAM ロールの ARN。AWS Glue にアクセスするためにアサームドロールベースの認証方式を使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.region               | Yes      | AWS Glue Data Catalog が存在するリージョン。例: `us-west-1`。 |
| aws.glue.access_key           | No       | AWS IAM ユーザーのアクセスキー。AWS Glue にアクセスするために IAM ユーザーベースの認証方式を使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | No       | AWS IAM ユーザーのシークレットキー。AWS Glue にアクセスするために IAM ユーザーベースの認証方式を使用する場合は、このパラメータを指定する必要があります。 |

AWS Glue にアクセスする認証方式を選択する方法や AWS IAM コンソールでアクセス制御ポリシーを構成する方法についての詳細については、[AWS Glue へのアクセスのための認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

ストレージシステムとの StarRocks の統合に関するパラメータのセット。このパラメータセットはオプションです。

HDFS をストレージとして使用する場合は、`StorageCredentialParams` を構成する必要はありません。

AWS S3、その他の S3 互換のストレージシステム、Microsoft Azure Storage、または Google GCS をストレージとして使用する場合は、`StorageCredentialParams` を構成する必要があります。

##### AWS S3

Delta Lake クラスターのストレージとして AWS S3 を選択する場合は、次のいずれかのアクションを実行してください。

- インスタンスプロファイルベースの認証方式を選択する場合、`StorageCredentialParams` を以下のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- アサームドロールベースの認証方式を選択する場合、`StorageCredentialParams` を以下のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM ユーザーベースの認証方式を選択する場合、`StorageCredentialParams` を以下のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

`StorageCredentialParams` で構成する必要のあるパラメータについては、次の表に記載されています。

| パラメータ                     | 必須     | 説明                                                       |
| ---------------------------- | -------- | --------------------------------------------------------- |
| aws.s3.use_instance_profile  | Yes      | インスタンスプロファイルベースの認証方式とアサームドロールベースの認証方式を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn          | No       | AWS S3 バケットに特権を持つ IAM ロールの ARN。AWS S3 にアクセスするためにアサームドロールベースの認証方式を使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.region                | Yes      | AWS S3 バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key            | No       | IAM ユーザーのアクセスキー。AWS S3 にアクセスするために IAM ユーザーベースの認証方式を使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key            | No       | IAM ユーザーのシークレットキー。AWS S3 にアクセスするために IAM ユーザーベースの認証方式を使用する場合は、このパラメータを指定する必要があります。 |

AWS S3 にアクセスする認証方式を選択する方法や AWS IAM コンソールでアクセス制御ポリシーを構成する方法についての詳細については、[AWS S3 へのアクセスのための認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3 互換のストレージシステム

Delta Lake カタログは v2.5 以降で S3 互換のストレージシステムをサポートしています。

MinIO などの S3 互換のストレージシステムをストレージとして選択する場合は、次のように`StorageCredentialParams` を構成して、統合が成功するようにしてください:

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

`StorageCredentialParams` で構成する必要のあるパラメータについては、次の表に記載されています。

| パラメータ                        | 必須     | 説明                                                       |
| -------------------------------- | -------- | --------------------------------------------------------- |
| aws.s3.enable_ssl                | Yes      | SSL 接続を有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access  | Yes      | パススタイルのアクセスを有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値: `false`。MinIO の場合は、この値を `true` に設定する必要があります。<br />パススタイルURL は次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部 (オレゴン) リージョンに `DOC-EXAMPLE-BUCKET1` というバケットを作成し、そのバケット内の `alice.jpg` オブジェクトにアクセスしたい場合は、次のパススタイルURL を使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | Yes      | AWS S3 の代わりに S3 互換のストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key                | Yes      | IAM ユーザーのアクセスキー。 |
| aws.s3.secret_key                | Yes      | IAM ユーザーのシークレットキー。 |

Microsoft Azure Storage

Delta Lake カタログは v3.0 以降で Microsoft Azure Storage をサポートしています。

Azure Blob Storage

Delta Lake クラスターのストレージとして Blob Storage を選択する場合は、次のいずれかのアクションを実行してください:

- シェアードキー認証方式を選択する場合、`StorageCredentialParams` を以下のように構成してください:

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` で構成する必要のあるパラメータについては、次の表に記載されています。

  | **Parameter**              | **Required** | **Description**                              |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | Yes          | Blob Storage アカウントのユーザー名。        |
  | azure.blob.shared_key      | Yes          | Blob Storage アカウントの共有キー。         |

- SAS トークン認証方式を選択する場合、`StorageCredentialParams` を以下のように構成してください:

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams` で構成する必要のあるパラメータについては、次の表に記載されています。

  | **Parameter**             | **Required** | **Description**                                              |
  | ------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.account_name   | Yes          | Blob Storage アカウントのユーザー名。                        |
  | azure.blob.container_name | Yes          | データを格納する blob コンテナの名前。                       |
  | azure.blob.sas_token      | Yes          | Blob Storage アカウントにアクセスするために使用される SAS トークン。 |

Azure Data Lake Storage Gen1

Delta Lake クラスターのストレージとして Data Lake Storage Gen1 を選択する場合は、次のいずれかのアクションを実行してください:

- 管理されたサービスアイデンティティ認証方式を選択する場合、`StorageCredentialParams` を以下のように構成してください:

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` で構成する必要のあるパラメータについては、次の表に記載されています。

  | **Parameter**                            | **Required** | **Description**                                              |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes          | 管理されたサービスアイデンティティ認証方式を有効にするかどうかを指定します。値を `true` に設定します。 |

- サービスプリンシパル認証方式を選択する場合、`StorageCredentialParams` を以下のように構成してください:

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams` で構成する必要のあるパラメータについては、次の表に記載されています。

  | **Parameter**                 | **Required** | **Description**                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | Yes          | サービスプリンシパルのクライアント (アプリケーション) ID。   |
  | azure.adls1.oauth2_credential | Yes          | 新しく作成されたクライアント (アプリケーション) シークレットの値。 |
  | azure.adls1.oauth2_endpoint   | Yes          | サービスプリンシパルまたはアプリケーションの OAuth 2.0 トークンエンドポイント (v1)。 |

Azure Data Lake Storage Gen2
Delta Lake Storage Gen2をDelta Lakeクラスターのストレージとして選択した場合、次のアクションのいずれかを実行してください：

- Managed Identity認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成してください：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明しています。

  | **Parameter（パラメータ）**               | **Required（必須）** | **Description（説明）**                                      |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes          | マネージドID認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | Yes          | アクセスするデータを持つテナントのIDです。                  |
  | azure.adls2.oauth2_client_id            | Yes          | マネージドIDのクライアント（アプリケーション）IDです。      |

- 共有キー認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成してください：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明しています。

  | **Parameter（パラメータ）**           | **Required（必須）** | **Description（説明）**                                      |
  | ----------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account        | Yes          | Data Lake Storage Gen2のストレージアカウントのユーザー名です。|
  | azure.adls2.shared_key             | Yes          | Data Lake Storage Gen2のストレージアカウントの共有キーです。|

- Service Principal認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成してください：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明しています。

  | **Parameter（パラメータ）**              | **Required（必須）** | **Description（説明）**                                      |
  | -------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id           | Yes          | サービスプリンシパルのクライアアント（アプリケーション）IDです。|
  | azure.adls2.oauth2_client_secret       | Yes          | 新しく作成したクライアント（アプリケーション）のシークレット値です。|
  | azure.adls2.oauth2_client_endpoint     | Yes          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。|

##### Google GCS

Delta Lakeのカタログはv3.0以降でGoogle GCSをサポートしています。

Delta LakeクラスターのストレージとしてGoogle GCSを選択した場合、次のアクションのいずれかを実行してください：

- VMベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成してください：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明しています。

  | **Parameter（パラメータ）**                          | **Default value（デフォルト値）** | **Value example（値の例）** | **Description（説明）**                                      |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接Compute Engineにバインドされたサービスアカウントを使用するかどうかを指定します。 |

- サービスアカウントベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成してください：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>",
  ```

  次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明しています。

  | **Parameter（パラメータ）**                          | **Default value（デフォルト値）** | **Value example（値の例）** | **Description（説明）**                                      |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウント作成時に生成されるJSONファイル内のメールアドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウント作成時に生成されるJSONファイル内のプライベートキーIDです。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウント作成時に生成されるJSONファイル内のプライベートキーです。 |

- 偽装ベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成してください：

  - VMインスタンスがサービスアカウントを偽装する場合：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明しています。

    | **Parameter（パラメータ）**                          | **Default value（デフォルト値）** | **Value example（値の例）** | **Description（説明）**                                      |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接Compute Engineにバインドされたサービスアカウントを使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | 偽装したいサービスアカウントです。                           |

  - サービスアカウント（一時的にメタサービスアカウントという名前が付けられています）が別のサービスアカウント（一時的にデータサービスアカウントという名前が付けられています）を偽装する場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明しています。

    | **Parameter（パラメータ）**                          | **Default value（デフォルト値）** | **Value example（値の例）** | **Description（説明）**                                        |
    | -------------------------------------- | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウント作成時に生成されるJSONファイル内のメールアドレスです。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタサービスアカウント作成時に生成されるJSONファイル内のプライベートキーIDです。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウント作成時に生成されるJSONファイル内のプライベートキーです。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 偽装したいデータサービスアカウントです。                     |

#### MetadataUpdateParams

Delta LakeのキャッシュされたメタデータをStarRocksが更新する方法に関するパラメータセットです。このパラメータセットはオプションです。

StarRocksはデフォルトで[自動非同期更新ポリシー](#appendix-understand-metadata-automatic-asynchronous-update)を実装しています。

ほとんどの場合、`MetadataUpdateParams`を無視し、それに含まれるポリシーパラメータを調整する必要はありません。なぜなら、これらのパラメータのデフォルト値はすでにオートボックスのパフォーマンスを提供しているからです。

ただし、Delta Lakeでのデータ更新頻度が高い場合は、これらのパラメータを調整して自動非同期更新のパフォーマンスをさらに最適化することができます。

> **注記**
>
> ほとんどの場合、Delta Lakeデータが1時間以下の間隔で更新される場合は、データ更新頻度が高いと見なされます。

| Parameter（パラメータ）                   | Required（必須） | Description（説明）                                          |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache （メタストアキャッシュを有効にする） | No       | StarRocksがDelta Lakeテーブルのメタデータをキャッシュするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| enable_remote_file_cache （リモートファイルキャッシュを有効にする） | No       | StarRocksがDelta Lakeテーブルまたはパーティションの基礎データファイルのメタデータをキャッシュするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   （メタストアキャッシュの更新間隔秒数） | No       | StarRocksが自身でキャッシュしているDelta Lakeテーブルまたはパーティションのメタデータを非同期に更新する時間間隔です。単位：秒。デフォルト値：`7200`、つまり2時間です。 |
| remote_file_cache_refresh_interval_sec | No       | StarRocks がデルタレイクテーブルまたはパーティションの元のデータファイルのメタデータを非同期で更新する間隔。単位：秒。デフォルト値：`60`。 |
| metastore_cache_ttl_sec                | No       | StarRocks が自動的にキャッシュしたデルタレイクテーブルまたはパーティションのメタデータを破棄する間隔。単位：秒。デフォルト値：`86400`（24 時間）。 |
| remote_file_cache_ttl_sec              | No       | StarRocks が自動的にキャッシュしたデルタレイクテーブルまたはパーティションの元のデータファイルのメタデータを破棄する間隔。単位：秒。デフォルト値：`129600`（36 時間）。 |

### 例

以下の例では、Delta Lake カタログを作成し、Delta Lake クラスタからデータをクエリする方法が示されています。

#### HDFS

ストレージとして HDFS を使用する場合は、次のようにコマンドを実行します。

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

##### インスタンスプロファイルベースの資格情報を選択した場合

- Delta Lake クラスタで Hive メタストアを使用する場合は、次のようにコマンドを実行します:

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

- Amazon EMR Delta Lake クラスタで AWS Glue を使用する場合は、次のようにコマンドを実行します:

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

##### 仮定されたロールベースの資格情報を選択した場合

- Delta Lake クラスタで Hive メタストアを使用する場合は、次のようにコマンドを実行します:

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

- Amazon EMR Delta Lake クラスタで AWS Glue を使用する場合は、次のようにコマンドを実行します:

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

##### IAM ユーザーベースの資格情報を選択した場合

- Delta Lake クラスタで Hive メタストアを使用する場合は、次のようにコマンドを実行します:

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

- Amazon EMR Delta Lake クラスタで AWS Glue を使用する場合は、次のようにコマンドを実行します:

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

#### S3 互換のストレージシステム

MinIO の例を使用した場合は、次のようにコマンドを実行します:

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

- 共有キー認証方法を選択した場合は、次のようにコマンドを実行します:

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

- SAS トークン認証方法を選択した場合は、次のようにコマンドを実行します:

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

- マネージドサービスアイデンティティ認証方法を選択した場合は、次のようにコマンドを実行します:

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

- サービス プリンシパル認証方法を選択した場合は、次のようにコマンドを実行します:

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

- マネージドアイデンティティ認証方法を選択した場合は、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
```SQL
"thrift://34.132.15.127:9083"に"="を設定します。
"azure.adls2.oauth2_use_managed_identity"に"true"を設定します。
"azure.adls2.oauth2_tenant_id"に"<service_principal_tenant_id>"を設定します。
"azure.adls2.oauth2_client_id"に"<service_client_id>"を設定します。

- Shared Key認証メソッドを選択した場合、次のようにコマンドを実行します：

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

- Service Principal認証メソッドを選択した場合、次のようにコマンドを実行します：

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

- VMベースの認証メソッドを選択した場合、次のようにコマンドを実行します：

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

- サービスアカウントベースの認証メソッドを選択した場合、次のようにコマンドを実行します：

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

- インパーソネーションベースの認証メソッドを選択した場合：

  - VMインスタンスがサービスアカウントを模倣する場合、次のようにコマンドを実行します：

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

  - サービスアカウントが他のサービスアカウントを模倣する場合、次のようにコマンドを実行します：

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

## Delta Lakeカタログを表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスターのすべてのカタログをクエリできます：

```SQL
SHOW CATALOGS;
```

[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリできます。次の例は名前が`deltalake_catalog_glue`のDelta Lakeカタログの作成ステートメントをクエリします：

```SQL
SHOW CREATE CATALOG deltalake_catalog_glue;
```

## Delta Lakeカタログとその中のデータベースに切り替える

次のいずれかの方法を使用してDelta Lakeカタログおよびその中のデータベースに切り替えることができます：

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して現在のセッションでDelta Lakeカタログを指定し、その後[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します：

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替える：
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定する：
  USE <db_name>
  ```

- 直接[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してDelta Lakeカタログおよびその中のデータベースに切り替える：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Delta Lakeカタログを削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して外部カタログを削除できます。

次の例では名前が`deltalake_catalog_glue`のDelta Lakeカタログを削除します：

```SQL
DROP Catalog deltalake_catalog_glue;
```

## Delta Lakeテーブルのスキーマを表示

Delta Lakeテーブルのスキーマを表示するには、次の構文のいずれかを使用できます：

- スキーマを表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからのスキーマとロケーションの表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Delta Lakeテーブルのクエリ

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用してDelta Lakeクラスターのデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Delta Lakeカタログおよびその中のデータベースに切り替える](#delta-lake-カタログとその中のデータベースに切り替える)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して指定されたデータベースの宛先テーブルをクエリします：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Delta Lakeからデータをロードする

OLAPテーブルの名前が`olap_tbl`であると仮定し、次のようにデータを変換およびロードできます：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM deltalake_table
```

## メタデータキャッシュの手動または自動更新

### 手動更新

デフォルトでは、StarRocksはDelta Lakeのメタデータをキャッシュし、パフォーマンスを向上させるためにメタデータを非同期モードで自動更新します。また、Delta Lakeテーブルにスキーマ変更やテーブル更新があった後、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)を使用してメタデータを手動で更新し、StarRocksが最新のメタデータを取得して適切な実行計画を生成できるようにすることができます：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自動的な増分更新

自動的な非同期更新ポリシーと異なり、自動的な増分更新ポリシーでは、StarRocksクラスター内のFEがHiveメタストアから列の追加、パーティションの削除、およびデータの更新などのイベントを読み取り、これらのイベントに基づいてFEにキャッシュされたメタデータを自動的に更新します。つまり、Delta Lakeテーブルのメタデータを手動で更新する必要はありません。

自動的な増分更新を有効にするには、次の手順に従ってください：

#### ステップ1：Hiveメタストアのイベントリスナーを構成する

Hiveメタストアv2.xおよびv3.xの両方はイベントリスナーを構成することをサポートしています。このステップでは、Hiveメタストアv3.1.2で使用されるイベントリスナー構成を例として示します。以下の構成項目を**$HiveMetastore/conf/hive-site.xml**ファイルに追加し、次にHiveメタストアを再起動します：

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
```
```xml
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

FEログファイル内で`event id`を検索して、イベントリスナーが正常に構成されているかを確認できます。構成に失敗した場合、`event id`の値は`0`です。

#### ステップ2：StarRocksで自動増分更新を有効にする

StarRocksでは、単一のDelta LakeカタログまたはStarRocksクラスタ内のすべてのDelta Lakeカタログに対して、自動増分更新を有効にできます。

- 単一のDelta Lakeカタログに対して自動増分更新を有効にするには、Delta Lakeカタログを作成する際に、以下のように`enable_hms_events_incremental_sync`パラメータを`true`に設定します。

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

- すべてのDelta Lakeカタログに対して自動増分更新を有効にするには、各FEの`$FE_HOME/conf/fe.conf`ファイルに`"enable_hms_events_incremental_sync" = "true"`を追加し、その後各FEを再起動してパラメータ設定を有効にします。

また、ビジネス要件に基づいて各FEの`$FE_HOME/conf/fe.conf`ファイルで次のパラメータを調整し、その後各FEを再起動してパラメータ設定を有効にできます。

| パラメータ                       | 説明                                                    |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocksがHiveメタストアからイベントを読み取る時間間隔。デフォルト値：`5000`。単位：ミリ秒。 |
| hms_events_batch_size_per_rpc     | 1回にStarRocksが読み取ることができるイベントの最大数。デフォルト値：`500`。 |
| enable_hms_parallel_process_evens | StarRocksがイベントを並列で処理するかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`true`。値が`true`の場合、並列処理が有効になり、`false`の場合、並列処理が無効になります。 |
| hms_process_events_parallel_num   | StarRocksが並列で処理できるイベントの最大数。デフォルト値：`4`。 |

## 付録：メタデータの自動非同期更新の理解

自動非同期更新は、StarRocksがDelta Lakeカタログ内のメタデータを更新するためのデフォルトポリシーです。

デフォルトでは（つまり、`enable_metastore_cache`および`enable_remote_file_cache`パラメータが共に`true`に設定されている場合）、クエリがDelta Lakeテーブルのパーティションにヒットすると、StarRocksは自動的にそのパーティションのメタデータとそのパーティションの基になるデータファイルのメタデータをキャッシュします。キャッシュされたメタデータは遅延更新ポリシーを使用して更新されます。

例えば、`table2`という名前のDelta Lakeテーブルがあり、そのテーブルには`p1`、`p2`、`p3`、`p4`という4つのパーティションがあります。クエリが`p1`にヒットし、StarRocksは`p1`のメタデータと`p1`の基になるデータファイルのメタデータをキャッシュします。キャッシュされたメタデータを更新または破棄するデフォルトの時間間隔は次のとおりです。

- `p1`のキャッシュされたメタデータを非同期で更新するための時間間隔（`metastore_cache_refresh_interval_sec`パラメータで指定）は2時間です。
- `p1`の基になるデータファイルのメタデータを非同期で更新するための時間間隔（`remote_file_cache_refresh_interval_sec`パラメータで指定）は60秒です。
- `p1`のキャッシュされたメタデータを自動的に破棄するための時間間隔（`metastore_cache_ttl_sec`パラメータで指定）は24時間です。
- `p1`の基になるデータファイルのメタデータを自動的に破棄するための時間間隔（`remote_file_cache_ttl_sec`パラメータで指定）は36時間です。

以下の図は、理解を容易にするためにタイムライン上での時間間隔を示しています。

![更新および破棄されたメタデータのタイムライン](../../assets/catalog_timeline.png)

その後、StarRocksは次の規則に従ってメタデータを更新または破棄します。

- もう一度`p1`がヒットした場合で、前回の更新から現在の時間が60秒未満の場合、StarRocksは`p1`のキャッシュされたメタデータまたは`p1`の基になるデータファイルのキャッシュされたメタデータを更新しません。
- もう一度`p1`がヒットした場合で、前回の更新から現在の時間が60秒よりも長い場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを更新します。
- もう一度`p1`がヒットした場合で、前回の更新から現在の時間が2時間よりも長い場合、StarRocksは`p1`のキャッシュされたメタデータを更新します。
- `p1`が最後に更新されてから24時間経過している場合、StarRocksは`p1`のキャッシュされたメタデータを破棄します。メタデータは次のクエリ時にキャッシュされます。
- `p1`が最後に更新されてから36時間経過している場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを破棄します。メタデータは次のクエリ時にキャッシュされます。