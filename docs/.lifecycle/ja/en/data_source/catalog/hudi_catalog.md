---
displayed_sidebar: English
---

# Hudiカタログ

Hudiカタログは、Apache Hudiから取り込みなしでデータをクエリできる外部カタログの一種です。

また、Hudiカタログを基に[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用して、Hudiからデータを直接変換およびロードすることもできます。StarRocksはv2.4以降、Hudiカタログをサポートしています。

HudiクラスターでSQLワークロードを成功させるためには、StarRocksクラスターが以下の2つの重要なコンポーネントと統合する必要があります：

- 分散ファイルシステム（HDFS）またはオブジェクトストレージ（AWS S3、Microsoft Azure Storage、Google GCS、またはその他のS3互換ストレージシステム、例えばMinIO）

- メタストア（HiveメタストアやAWS Glueなど）

  > **注記**
  >
  > ストレージとしてAWS S3を選択する場合、メタストアとしてHMSまたはAWS Glueを使用できます。他のストレージシステムを選択する場合、HMSをメタストアとしてのみ使用できます。

## 使用上の注意

- StarRocksがサポートするHudiのファイル形式はParquetです。Parquetファイルは、SNAPPY、LZ4、ZSTD、GZIP、およびNO_COMPRESSIONの圧縮形式をサポートしています。
- StarRocksは、HudiのCopy On Write（COW）テーブルとMerge On Read（MOR）テーブルを完全にサポートしています。

## 統合の準備

Hudiカタログを作成する前に、StarRocksクラスターがHudiクラスターのストレージシステムおよびメタストアと統合できることを確認してください。

### AWS IAM

HudiクラスターがストレージとしてAWS S3を使用するか、メタストアとしてAWS Glueを使用する場合、適切な認証方法を選択し、StarRocksクラスターが関連するAWSクラウドリソースにアクセスできるように必要な準備を行ってください。

以下の認証方法が推奨されます：

- インスタンスプロファイル
- アサムドロール
- IAMユーザー

上記の認証方法の中で、インスタンスプロファイルが最も広く使用されています。

詳細については、[AWS IAMでの認証の準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとしてHDFSを選択する場合、StarRocksクラスターを以下のように設定します：

- （オプション）HDFSクラスターおよびHiveメタストアにアクセスするために使用するユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスターおよびHiveメタストアにアクセスします。または、各FEの**fe/conf/hadoop_env.sh**ファイルと各BEの**be/conf/hadoop_env.sh**ファイルの先頭に`export HADOOP_USER_NAME="<user_name>"`を追加することでユーザー名を設定することもできます。これらのファイルにユーザー名を設定した後、各FEおよびBEを再起動してパラメータ設定を有効にします。StarRocksクラスターごとに1つのユーザー名のみを設定できます。
- Hudiデータをクエリする際、StarRocksクラスターのFEとBEはHDFSクライアントを使用してHDFSクラスターにアクセスします。ほとんどの場合、この目的を達成するためにStarRocksクラスターを設定する必要はありません。StarRocksはデフォルトの設定を使用してHDFSクライアントを起動します。StarRocksクラスターを設定する必要があるのは、以下の状況のみです：

  - HDFSクラスターで高可用性（HA）が有効になっている場合：HDFSクラスターの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。
  - View File System（ViewFs）がHDFSクラスターで有効になっている場合：HDFSクラスターの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。

> **注記**
>
> クエリを送信した際に不明なホストを示すエラーが返される場合は、HDFSクラスターノードのホスト名とIPアドレスのマッピングを**/etc/hosts**に追加する必要があります。

### Kerberos認証

HDFSクラスターやHiveメタストアでKerberos認証が有効になっている場合、StarRocksクラスターを以下のように設定します：

- 各FEおよびBEで`kinit -kt keytab_path principal`コマンドを実行して、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFSクラスターとHiveメタストアにアクセスする権限が必要です。KDCへのアクセスは時間に敏感なため、このコマンドを定期的に実行するためにcronを使用する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルと各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。この例では、`/etc/krb5.conf`は**krb5.conf**ファイルの保存パスです。必要に応じてパスを変更してください。

## Hudiカタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "hudi",
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### パラメータ

#### catalog_name

Hudiカタログの名前です。命名規則は以下の通りです：

- 名前には文字、数字（0-9）、およびアンダースコア（_）を含めることができます。文字で始まる必要があります。
- 名前は大文字と小文字を区別し、長さは1023文字を超えることはできません。

#### comment

Hudiカタログの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。値を`hudi`に設定します。

#### MetastoreParams

StarRocksがデータソースのメタストアと統合する方法に関するパラメータのセットです。

##### Hiveメタストア

データソースのメタストアとしてHiveメタストアを選択する場合、`MetastoreParams`を次のように設定します：

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> Hudiデータをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを**/etc/hosts**に追加する必要があります。そうしないと、クエリを開始する際にStarRocksがHiveメタストアへのアクセスに失敗する可能性があります。

以下の表は、`MetastoreParams`で設定する必要があるパラメータを説明しています。

| パラメータ           | 必須 | 説明                                                  |
| ------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type | はい      | Hudiクラスターに使用するメタストアのタイプです。値を`hive`に設定します。 |
| hive.metastore.uris | はい      | HiveメタストアのURIです。形式：`thrift://<metastore_IP_address>:<metastore_port>`。<br />Hiveメタストアで高可用性（HA）が有効な場合、複数のメタストアURIを指定し、それらをコンマ（,）で区切ることができます。例：`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。|

##### AWS Glue

データソースのメタストアとしてAWS Glueを選択する場合（ストレージとしてAWS S3を選択した場合にのみサポートされます）、`MetastoreParams`を次のように設定します：

  ```SQL
  # ここには具体的な設定が記載されていません。AWS Glueを使用する場合の設定例または説明が必要です。
  ```

  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 想定されるロールベースの認証方法を選択するには、`MetastoreParams`を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAMユーザーベースの認証方法を選択するには、`MetastoreParams`を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

以下の表は、`MetastoreParams`で設定する必要があるパラメータの説明です。

| パラメーター                     | 必須 | 説明                                                  |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type           | はい      | Hudiクラスターで使用するメタストアのタイプ。値を`glue`に設定します。 |
| aws.glue.use_instance_profile | はい      | インスタンスプロファイルベースの認証方法または想定されるロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。 |
| aws.glue.iam_role_arn         | いいえ       | AWS Glue Data Catalogに対する権限を持つIAMロールのARN。想定されるロールベースの認証方法を使用してAWS Glueにアクセスする場合は、このパラメータを指定する必要があります。 |
| aws.glue.region               | はい      | AWS Glue Data Catalogが存在するリージョン。例: `us-west-1`。 |
| aws.glue.access_key           | いいえ       | AWS IAMユーザーのアクセスキー。IAMユーザーベースの認証方法を使用してAWS Glueにアクセスする場合は、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | いいえ       | AWS IAMユーザーのシークレットキー。IAMユーザーベースの認証方法を使用してAWS Glueにアクセスする場合は、このパラメータを指定する必要があります。 |

AWS Glueにアクセスするための認証方法を選択する方法と、AWS IAMコンソールでアクセスコントロールポリシーを設定する方法については、[AWS Glueへのアクセスのための認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocksがストレージシステムと統合するための一連のパラメータです。このパラメータセットはオプションです。

HDFSをストレージとして使用する場合、`StorageCredentialParams`を設定する必要はありません。

AWS S3、S3互換ストレージシステム、Microsoft Azure Storage、またはGoogle GCSをストレージとして使用する場合は、`StorageCredentialParams`を設定する必要があります。

##### AWS S3

HudiクラスターのストレージとしてAWS S3を選択する場合、以下のいずれかのアクションを実行します。

- インスタンスプロファイルベースの認証方法を選択するには、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 想定されるロールベースの認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーベースの認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表は、`StorageCredentialParams`で設定する必要があるパラメータの説明です。

| パラメーター                   | 必須 | 説明                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい      | インスタンスプロファイルベースの認証方法または想定されるロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | いいえ       | AWS S3バケットに対する権限を持つIAMロールのARN。想定されるロールベースの認証方法を使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい      | AWS S3バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | いいえ       | IAMユーザーのアクセスキー。IAMユーザーベースの認証方法を使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAMユーザーのシークレットキー。IAMユーザーベースの認証方法を使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。 |

AWS S3にアクセスするための認証方法を選択する方法と、AWS IAMコンソールでアクセスコントロールポリシーを設定する方法については、[AWS S3へのアクセスのための認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

Hudiカタログは、v2.5以降のS3互換ストレージシステムをサポートしています。

S3互換ストレージシステム、例えばMinIOをHudiクラスターのストレージとして選択する場合、`StorageCredentialParams`を次のように構成して統合を成功させます。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表は、`StorageCredentialParams`で設定する必要があるパラメータの説明です。

| パラメーター                        | 必須 | 説明                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | はい      | SSL接続を有効にするかどうかを指定します。<br />有効な値: `true`、`false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access  | はい      | パス形式のアクセスを有効にするかどうかを指定します。<br />有効な値: `true`、`false`。デフォルト値: `false`。MinIOを使用する場合は、値を`true`に設定する必要があります。<br />パス形式のURLは次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部（オレゴン）リージョンに`DOC-EXAMPLE-BUCKET1`という名前のバケットを作成し、そのバケット内の`alice.jpg`オブジェクトにアクセスする場合、次のパス形式のURLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | はい      | AWS S3ではなく、S3互換ストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key                | はい      | IAMユーザーのアクセスキー。 |
| aws.s3.secret_key                | はい      | IAMユーザーのシークレットキー。 |

##### Microsoft Azure Storage

Hudiカタログは、v3.0以降のMicrosoft Azure Storageをサポートしています。

###### Azure Blob Storage

HudiクラスターのストレージとしてBlob Storageを選択する場合、以下のいずれかのアクションを実行します。

- 共有キー認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```


  以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

  | **パラメーター**              | **必須** | **説明**                                          |
  | -------------------------- | ------------ | ------------------------------------------------ |
  | azure.blob.storage_account | はい          | Blob Storage アカウントのユーザー名です。         |
  | azure.blob.shared_key      | はい          | Blob Storage アカウントの共有キーです。           |

- SAS トークン認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

  | **パラメーター**             | **必須** | **説明**                                                      |
  | ------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.storage_account| はい          | Blob Storage アカウントのユーザー名です。                     |
  | azure.blob.container      | はい          | データを格納するblobコンテナの名前です。                      |
  | azure.blob.sas_token      | はい          | Blob Storage アカウントへのアクセスに使用されるSASトークンです。 |

###### Azure Data Lake Storage Gen1

HudiクラスターのストレージとしてData Lake Storage Gen1を選択する場合、以下のいずれかのアクションを実行します。

- 管理対象サービスIDの認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

  | **パラメーター**                            | **必須** | **説明**                                                      |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい          | 管理対象サービスIDの認証方法を有効にするかどうかを指定します。値は`true`に設定します。 |

- サービスプリンシパルの認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

  | **パラメーター**                 | **必須** | **説明**                                                      |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい          | サービスプリンシパルのクライアント（アプリケーション）IDです。        |
  | azure.adls1.oauth2_credential | はい          | 新しく作成されたクライアント（アプリケーション）シークレットの値です。    |
  | azure.adls1.oauth2_endpoint   | はい          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

###### Azure Data Lake Storage Gen2

HudiクラスターのストレージとしてData Lake Storage Gen2を選択する場合、以下のいずれかのアクションを実行します。

- マネージドIDの認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

  | **パラメーター**                           | **必須** | **説明**                                                      |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | はい          | マネージドID認証方法を有効にするかどうかを指定します。値は`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | はい          | アクセスしたいテナントのIDです。                              |
  | azure.adls2.oauth2_client_id            | はい          | マネージドIDのクライアント（アプリケーション）IDです。         |

- 共有キーの認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

  | **パラメーター**               | **必須** | **説明**                                                      |
  | --------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account | はい          | Data Lake Storage Gen2ストレージアカウントのユーザー名です。 |
  | azure.adls2.shared_key      | はい          | Data Lake Storage Gen2ストレージアカウントの共有キーです。   |

- サービスプリンシパルの認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

  | **パラメーター**                      | **必須** | **説明**                                                      |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい          | サービスプリンシパルのクライアント（アプリケーション）IDです。        |
  | azure.adls2.oauth2_client_secret   | はい          | 新しく作成されたクライアント（アプリケーション）シークレットの値です。    |
  | azure.adls2.oauth2_client_endpoint | はい          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

##### Google GCS

Hudiカタログは、v3.0以降Google GCSをサポートしています。

HudiクラスターのストレージとしてGoogle GCSを選択する場合、以下のいずれかのアクションを実行します。

- VMベースの認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

  | **パラメーター**                              | **デフォルト値** | **値の例** | **説明**                                                      |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineに紐付けられたサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

  | **パラメーター**                          | **デフォルト値** | **値の例**                                        | **説明**                                                      |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウント作成時に生成されたJSONファイル内のメールアドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウント作成時に生成されたJSONファイル内のプライベートキーIDです。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウント作成時に生成されたJSONファイル内のプライベートキーです。 |

- 偽装ベースの認証方法を選択するには、`StorageCredentialParams`を次のように構成します。

  - VMインスタンスがサービスアカウントを偽装するようにします。
  
    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。


    | **パラメーター**                              | **デフォルト値** | **値の例** | **説明**                                              |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineにバインドされているサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | なりすますサービスアカウントを指定します。            |

  - サービスアカウント（一時的にメタサービスアカウントと呼ばれる）が別のサービスアカウント（一時的にデータサービスアカウントと呼ばれる）をなりすます場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

    | **パラメーター**                          | **デフォルト値** | **値の例**                                        | **説明**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウント作成時に生成されたJSONファイル内のメールアドレス。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキーID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキー。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | なりすますデータサービスアカウントを指定します。       |

#### MetadataUpdateParams

StarRocksがHudiのキャッシュされたメタデータを更新する方法に関するパラメーターセットです。このパラメーターセットはオプションです。

StarRocksはデフォルトで[自動非同期更新ポリシー](#appendix-understand-metadata-automatic-asynchronous-update)を実装しています。

ほとんどの場合、`MetadataUpdateParams`を無視して、ポリシーパラメーターを調整する必要はありません。なぜなら、これらのパラメーターのデフォルト値は既にプラグアンドプレイのパフォーマンスを提供しているからです。

しかし、Hudiでのデータ更新頻度が高い場合は、これらのパラメーターを調整して、自動非同期更新のパフォーマンスをさらに最適化することができます。

> **注記**
>
> ほとんどの場合、Hudiデータが1時間以下の粒度で更新される場合、データ更新頻度は高いと見なされます。

| パラメーター                              | 必須 | 説明                                                  |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | いいえ       | StarRocksがHudiテーブルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| enable_remote_file_cache               | いいえ       | StarRocksがHudiテーブルまたはパーティションの基盤となるデータファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | いいえ       | StarRocksが自身にキャッシュされたHudiテーブルまたはパーティションのメタデータを非同期に更新する時間間隔。単位: 秒。デフォルト値: `7200`（2時間）。 |
| remote_file_cache_refresh_interval_sec | いいえ       | StarRocksが自身にキャッシュされたHudiテーブルまたはパーティションの基盤となるデータファイルのメタデータを非同期に更新する時間間隔。単位: 秒。デフォルト値: `60`。 |
| metastore_cache_ttl_sec                | いいえ       | StarRocksが自身にキャッシュされたHudiテーブルまたはパーティションのメタデータを自動的に破棄する時間間隔。単位: 秒。デフォルト値: `86400`（24時間）。 |
| remote_file_cache_ttl_sec              | いいえ       | StarRocksが自身にキャッシュされたHudiテーブルまたはパーティションの基盤となるデータファイルのメタデータを自動的に破棄する時間間隔。単位: 秒。デフォルト値: `129600`（36時間）。 |

### 例

次の例では、`hudi_catalog_hms`または`hudi_catalog_glue`（使用するメタストアのタイプに応じて）という名前のHudiカタログを作成し、Hudiクラスターからデータをクエリします。

#### HDFS

HDFSをストレージとして使用する場合、以下のコマンドを実行します：

```SQL
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

#### AWS S3

##### インスタンスプロファイルベースの認証情報を選択する場合

- HudiクラスターでHiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HudiクラスターでAWS Glueを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_glue
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 想定されるロールベースの認証情報を選択する場合

- HudiクラスターでHiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HudiクラスターでAWS Glueを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_glue
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### IAMユーザーベースの認証情報を選択する場合

- HudiクラスターでHiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HudiクラスターでAWS Glueを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_glue
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```
  CREATE EXTERNAL CATALOG hudi_catalog_glue
  PROPERTIES
  (
      "type" = "hudi",
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
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
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

- 共有キー認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SASトークン認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- マネージドサービスID認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- サービスプリンシパル認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- マネージドID認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 共有キー認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- サービスプリンシパル認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMベースの認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- サービスアカウントベースの認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 代理認証方式を選択する場合:

  - VMインスタンスがサービスアカウントを代理する場合、以下のコマンドを実行します。

    ```SQL
    CREATE EXTERNAL CATALOG hudi_catalog_hms
    PROPERTIES
    (
        "type" = "hudi",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - サービスアカウントが他のサービスアカウントを代理する場合、以下のコマンドを実行します。

    ```SQL
    CREATE EXTERNAL CATALOG hudi_catalog_hms
    PROPERTIES
    (
        "type" = "hudi",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Hudiカタログを表示する

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスタ内のすべてのカタログを照会できます。

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントを照会することもできます。以下の例では、`hudi_catalog_glue`という名前のHudiカタログの作成ステートメントを照会します。

```SQL
SHOW CREATE CATALOG hudi_catalog_glue;
```

## Hudiカタログとその中のデータベースに切り替える

次のいずれかの方法を使用して、Hudiカタログとその中のデータベースに切り替えることができます。

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して現在のセッションでHudiカタログを指定し、その後[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します。

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替えます:
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定します:
  USE <db_name>
  ```

- [USE](../../sql-reference/sql-statements/data-definition/USE.md)を直接使用して、Hudiカタログとその中のデータベースに切り替えます。

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Hudiカタログを削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、外部カタログを削除します。

以下の例では、`hudi_catalog_glue`という名前のHudiカタログを削除します。

```SQL
DROP CATALOG hudi_catalog_glue;
```

## Hudiテーブルのスキーマを表示する

Hudiテーブルのスキーマを表示するには、以下の構文のいずれかを使用します。

- スキーマを表示する

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Hudiテーブルをクエリする


1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を使用して、Hudi クラスター内のデータベースを表示します。

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Hudi カタログとその中のデータベースに切り替える](#switch-to-a-hudi-catalog-and-a-database-in-it)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して、指定されたデータベース内の目的のテーブルをクエリします。

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hudi からデータをロードする

`olap_tbl` という名前の OLAP テーブルがある場合、以下のようにデータを変換してロードできます。

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hudi_table
```

## メタデータキャッシュの手動更新または自動更新

### 手動更新

デフォルトでは、StarRocks は Hudi のメタデータをキャッシュし、非同期モードで自動的にメタデータを更新してパフォーマンスを向上させます。さらに、Hudi テーブルにスキーマ変更やテーブル更新が行われた後、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) を使用してメタデータを手動で更新することもできます。これにより、StarRocks は最新のメタデータを迅速に取得し、適切な実行計画を生成することができます。

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自動増分更新

自動非同期更新ポリシーとは異なり、自動増分更新ポリシーでは、StarRocks クラスター内の FE が Hive メタストアからイベント（列の追加、パーティションの削除、データの更新など）を読み取ります。StarRocks はこれらのイベントに基づいて FE にキャッシュされたメタデータを自動的に更新できます。つまり、Hudi テーブルのメタデータを手動で更新する必要はありません。

自動増分更新を有効にするには、以下の手順に従ってください。

#### ステップ 1: Hive メタストアのイベントリスナーを設定する

Hive メタストア v2.x および v3.x はイベントリスナーの設定をサポートしています。ここでは、Hive メタストア v3.1.2 用のイベントリスナー設定を例に挙げます。以下の設定項目を **$HiveMetastore/conf/hive-site.xml** ファイルに追加し、Hive メタストアを再起動します。

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

自動増分更新は、StarRocks クラスター内の単一の Hudi カタログまたはすべての Hudi カタログに対して有効にできます。

- 単一の Hudi カタログに対して自動増分更新を有効にするには、Hudi カタログを作成する際に `PROPERTIES` で `enable_hms_events_incremental_sync` パラメータを `true` に設定します。

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```

- すべての Hudi カタログに対して自動増分更新を有効にするには、各 FE の `$FE_HOME/conf/fe.conf` ファイルに `"enable_hms_events_incremental_sync" = "true"` を追加し、各 FE を再起動して設定を有効にします。

ビジネス要件に基づいて、各 FE の `$FE_HOME/conf/fe.conf` ファイルで以下のパラメータを調整し、各 FE を再起動して設定を有効にすることもできます。

| パラメーター                         | 説明                                                  |
| --------------------------------- | ---------------------------------------------------- |
| hms_events_polling_interval_ms    | StarRocks が Hive メタストアからイベントを読み取る時間間隔。デフォルト値は `5000` ミリ秒です。 |
| hms_events_batch_size_per_rpc     | StarRocks が一度に読み取ることができるイベントの最大数。デフォルト値は `500` です。 |
| enable_hms_parallel_process_evens | StarRocks がイベントを読み取る際に、イベントを並行して処理するかどうかを指定します。有効な値は `true` と `false` です。デフォルト値は `true` で、`true` は並列処理を有効にし、`false` は並列処理を無効にします。 |
| hms_process_events_parallel_num   | StarRocks が並行して処理できるイベントの最大数。デフォルト値は `4` です。 |

## 付録: メタデータの自動非同期更新について理解する

自動非同期更新は、StarRocks が Hudi カタログのメタデータを更新するために使用するデフォルトのポリシーです。

デフォルト（つまり、`enable_metastore_cache` と `enable_remote_file_cache` の両方のパラメータが `true` に設定されている場合）、クエリが Hudi テーブルのパーティションを検索すると、StarRocks はそのパーティションのメタデータとそのパーティションの下層にあるデータファイルのメタデータを自動的にキャッシュします。キャッシュされたメタデータは、遅延更新ポリシーを使用して更新されます。

例えば、`table2` という名前の Hudi テーブルがあり、`p1`、`p2`、`p3`、`p4` の 4 つのパーティションがあります。クエリが `p1` を検索し、StarRocks は `p1` のメタデータとその下層にあるデータファイルのメタデータをキャッシュします。キャッシュされたメタデータを更新および破棄するデフォルトの時間間隔は以下の通りです。

- `p1` のキャッシュされたメタデータを非同期で更新する時間間隔（`metastore_cache_refresh_interval_sec` パラメータによって指定）は 2 時間です。
- `p1` の下層にあるデータファイルのキャッシュされたメタデータを非同期で更新する時間間隔（`remote_file_cache_refresh_interval_sec` パラメータによって指定）は 60 秒です。
- `p1` のキャッシュされたメタデータを自動的に破棄する時間間隔（`metastore_cache_ttl_sec` パラメータによって指定）は 24 時間です。
- `p1` の下層にあるデータファイルのキャッシュされたメタデータを自動的に破棄する時間間隔（`remote_file_cache_ttl_sec` パラメータによって指定）は 36 時間です。

以下の図は、タイムライン上の時間間隔を分かりやすく示しています。

![キャッシュされたメタデータの更新と破棄のタイムライン](../../assets/catalog_timeline.png)

その後、StarRocks は以下のルールに従ってメタデータを更新または破棄します。

- 別のクエリが `p1` を再び検索し、最後の更新からの現在の時間が 60 秒未満である場合、StarRocks は `p1` のキャッシュされたメタデータやその下層にあるデータファイルのキャッシュされたメタデータを更新しません。
- 別のクエリが`p1`を再び参照し、最後の更新から現在までの時間が60秒を超えた場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを更新します。
- 別のクエリが`p1`を再び参照し、最後の更新から現在までの時間が2時間を超えた場合、StarRocksは`p1`のキャッシュされたメタデータを更新します。
- `p1`が最後の更新から24時間以内にアクセスされていない場合、StarRocksは`p1`のキャッシュされたメタデータを破棄します。メタデータは次のクエリ時に再度キャッシュされます。
- `p1`が最後の更新から36時間以内にアクセスされていない場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを破棄します。メタデータは次のクエリ時に再度キャッシュされます。
