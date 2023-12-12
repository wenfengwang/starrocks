```yaml
---
displayed_sidebar: "Japanese"
---

# Hudiカタログ

Hudiカタログは、Apache Hudiからのデータクエリをインジェストせずに可能にする外部カタログの一種です。

また、Hudiカタログに基づいて[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用して、Hudiからデータを直接変換およびロードすることができます。StarRocksはv2.4以降でHudiカタログをサポートしています。

HudiクラスターでのSQLワークロードの成功を確実にするには、StarRocksクラスターは次の2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCS、またはその他のS3互換ストレージシステム（たとえば、MinIO）などのオブジェクトストレージ
- HiveメタストアまたはAWS Glueなどのフィメタストア

  > **注意**
  >
  > ストレージとしてAWS S3を選択した場合、メタストアとしてHMSまたはAWS Glueを使用できます。他のストレージシステムを選択した場合は、メタストアとしてHMSのみを使用できます。

## 使用上の注意

StarRocksがサポートするHudiのファイルフォーマットはParquetです。Parquetファイルは、以下の圧縮フォーマットをサポートしています: SNAPPY、LZ4、ZSTD、GZIP、およびNO_COMPRESSION。
StarRocksはHudiからのCopy On Write（COW）テーブルとMerge On Read（MOR）テーブルを完全にサポートしています。

## 統合の準備

Hudiカタログを作成する前に、StarRocksクラスターがHudiクラスターのストレージシステムおよびメタストアと統合できることを確認してください。

### AWS IAM

HudiクラスターがストレージとしてAWS S3またはメタストアとしてAWS Glueを使用する場合は、適切な認証方法を選択し、StarRocksクラスターが関連するAWSクラウドリソースにアクセスできるように必要な準備を行ってください。

次の認証方法が推奨されています:

- インスタンスプロファイル
- 仮定ロール
- IAMユーザー

上記の3つの認証方法のうち、インスタンスプロファイルが最も広く使用されています。

詳細については、[AWS IAMでの認証の準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとしてHDFSを選択した場合は、StarRocksクラスターを次のように構成してください:

- （オプション）HDFSクラスターとHiveメタストアへのアクセスに使用されるユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスターおよびHiveメタストアにアクセスします。また、各FEの**fe/conf/hadoop_env.sh**ファイルおよび各BEの**be/conf/hadoop_env.sh**ファイルの先頭に`export HADOOP_USER_NAME="<user_name>"`を追加してユーザー名を設定できます。これらのファイルにユーザー名を設定した後は、各FEおよび各BEを再起動してパラメータ設定を有効にします。1つのStarRocksクラスターにつき、1つのユーザー名のみを設定できます。
- Hudiデータのクエリを実行する際、StarRocksクラスターのFEおよびBEはHDFSクライアントを使用してHDFSクラスターにアクセスします。ほとんどの場合、この目的を達成するためにStarRocksクラスターを構成する必要はありません。StarRocksはデフォルトの構成でHDFSクライアントを起動します。次の状況でのみStarRocksクラスターを構成する必要があります:

  - HDFSクラスターの高可用性（HA）が有効になっている場合: HDFSクラスターの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスおよび各BEの**$BE_HOME/conf**パスに追加します。
  - HDFSクラスターのView File System（ViewFs）が有効になっている場合: HDFSクラスターの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスおよび各BEの**$BE_HOME/conf**パスに追加します。

> **注意**
>
> クエリを送信する際に不明なホストを示すエラーが返される場合は、HDFSクラスターノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに追加する必要があります。

### Kerberos認証

HDFSクラスターまたはHiveメタストアにKerberos認証が有効になっている場合は、StarRocksクラスターを次のように構成してください:

- 各FEおよび各BEで`kinit -kt keytab_path principal`コマンドを実行して、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFSクラスターおよびHiveメタストアにアクセスする権限が必要です。このコマンドでKDCにアクセスするためには、cronを使用してこのコマンドを定期的に実行する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルおよび各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。この例では、`/etc/krb5.conf`が**krb5.conf**ファイルの保存パスです。必要に応じてパスを変更できます。

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

Hudiカタログの名前。次の命名規則が適用されます:

- 名前には、英字、数字（0-9）、アンダースコア（_）を含めることができます。英字で始まる必要があります。
- 名前は大文字と小文字を区別し、1023文字を超えることはできません。

#### comment

Hudiカタログの説明。このパラメータはオプションです。

#### type

データソースの種類。値を`hudi`に設定します。

#### MetastoreParams

データソースのメタストアにStarRocksがどのように統合するかに関するパラメータのセット。

##### Hiveメタストア

データソースのメタストアとしてHiveメタストアを選択した場合、`MetastoreParams`を次のように構成します:

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注意**
>
> Hudiデータをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに追加する必要があります。それ以外の場合、クエリを開始するとStarRocksがHiveメタストアにアクセスできなくなる場合があります。

次の表は、`MetastoreParams`で構成する必要があるパラメータについて説明しています。

| パラメータ               | 必須     | 説明                               |
| ------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type | Yes      | Hudiクラスターに使用するメタストアのタイプ。値を`hive`に設定します。 |
| hive.metastore.uris | Yes      | HiveメタストアのURI。形式: `thrift://<metastore_IP_address>:<metastore_port>`。<br />Hiveメタストアの高可用性（HA）が有効になっている場合は、複数のメタストアURIを指定し、カンマ（`,`）で区切って指定できます。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

データソースのメタストアとしてAWS Glueを選択し、ストレージとしてAWS S3を選択した場合、次のいずれかの操作を実行します:

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams`を次のように構成します:

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 仮定ロールベースの認証方法を選択する場合、`MetastoreParams`を次のように構成します:

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAMユーザーベースの認証方法を選択する場合、`MetastoreParams`を次のように構成します:

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

次の表は、`MetastoreParams`で構成する必要があるパラメータについて説明しています。

| パラメータ                     | 必須     | 説明                               |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type           | Yes      | Hudiクラスターに使用するメタストアのタイプ。値を`glue`に設定します。 |
| aws.glue.use_instance_profile | Yes      | インスタンスプロファイルベースの認証方法および仮定ロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true`および`false`。デフォルト値: `false`。 |
```
| aws.glue.iam_role_arn         | いいえ     | AWS Glueデータカタログに特権を持つIAMロールのARNです。AWS Glueにアクセスするために前提のロールベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.region               | はい      | AWS Glueデータカタログが存在するリージョンです。例: `us-west-1`。 |
| aws.glue.access_key           | いいえ     | AWS IAMユーザーのアクセスキーです。AWS GlueにアクセスするためにIAMユーザーをベースとした認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | いいえ     | AWS IAMユーザーのシークレットキーです。AWS GlueにアクセスするためにIAMユーザーをベースとした認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |

AWS Glueにアクセスするための認証メソッドの選択方法とAWS IAM Consoleでのアクセス制御ポリシーの構成方法についての詳細については、[AWS Glueへのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocksがストレージシステムとどのように統合されるかに関する一連のパラメータです。このパラメータセットはオプションです。

ストレージとしてHDFSを使用する場合、`StorageCredentialParams`を設定する必要はありません。

AWS S3、その他のS3互換のストレージシステム、Microsoft Azure Storage、またはGoogle GCSをストレージとして使用する場合は、`StorageCredentialParams`を構成する必要があります。

##### AWS S3

HudiクラスタのストレージとしてAWS S3を選択した場合、次のいずれかのアクションを実行してください。

- インスタンスプロファイルベースの認証メソッドを選択する場合、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 前提のロールベースの認証メソッドを選択する場合、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーをベースとした認証メソッドを選択する場合、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表では、`StorageCredentialParams`で構成する必要があるパラメータについて説明しています。

| パラメータ                      | 必須     | 説明                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい      | インスタンスプロファイルベースの認証メソッドと前提のロールベースの認証メソッドを有効にするかどうかを指定します。有効な値: `true`と`false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | いいえ     | AWS S3バケットに特権を持つIAMロールのARNです。AWS S3にアクセスするために前提のロールベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい      | AWS S3バケットが存在するリージョンです。例: `us-west-1`。 |
| aws.s3.access_key           | いいえ     | IAMユーザーのアクセスキーです。AWS S3にアクセスするためにIAMユーザーをベースとした認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ     | IAMユーザーのシークレットキーです。AWS S3にアクセスするためにIAMユーザーをベースとした認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |

AWS S3へのアクセス方法を選択する方法やAWS IAM Consoleでアクセス制御ポリシーを構成する方法についての詳細については、[AWS S3へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換のストレージシステム

Hudiカタログはv2.5以降、S3互換のストレージシステムをサポートしています。

MinIOなどのS3互換のストレージシステムを選択した場合、Hudiクラスタのストレージとして使用するために`StorageCredentialParams`を以下のように構成して、統合が成功するようにしてください:

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表では、`StorageCredentialParams`で構成する必要があるパラメータについて説明しています。

| パラメータ                        | 必須     | 説明                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | はい      | SSL接続を有効にするかどうかを指定します。<br />有効な値: `true`と`false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access  | はい      | パス形式のアクセスを有効にするかどうかを指定します。<br />有効な値: `true`と`false`。デフォルト値: `false`。MinIOの場合、値を`true`に設定する必要があります。<br />パス形式URLは以下の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、US West (Oregon)リージョンで`DOC-EXAMPLE-BUCKET1`というバケットを作成し、そのバケット内の`alice.jpg`オブジェクトにアクセスしたい場合、次のパス形式URLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | はい      | AWS S3の代わりにS3互換のストレージシステムに接続するために使用するエンドポイントです。 |
| aws.s3.access_key                | はい      | IAMユーザーのアクセスキーです。 |
| aws.s3.secret_key                | はい      | IAMユーザーのシークレットキーです。 |

##### Microsoft Azure Storage

Hudiカタログはv3.0以降、Microsoft Azure Storageをサポートしています。

###### Azure Blob Storage

HudiクラスタのストレージとしてBlob Storageを選択した場合、次のいずれかのアクションを実行してください:

- 共有キー認証メソッドを選択する場合、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下の表では、`StorageCredentialParams`で構成する必要があるパラメータについて説明しています。

  | **Parameter**              | **Required** | **Description**                              |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | はい          | Blob Storageアカウントのユーザー名です。   |
  | azure.blob.shared_key      | はい          | Blob Storageアカウントの共有キーです。 |

- SASトークン認証メソッドを選択する場合、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  以下の表では、`StorageCredentialParams`で構成する必要があるパラメータについて説明しています。

  | **Parameter**             | **Required** | **Description**                                              |
  | ------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.account_name   | はい          | Blob Storageアカウントのユーザー名です。                   |
  | azure.blob.container_name | はい          | データを格納するBlobコンテナの名前です。                   |
  | azure.blob.sas_token      | はい          | Blob Storageアカウントにアクセスするために使用されるSASトークンです。 |

###### Azure Data Lake Storage Gen1

HudiクラスタのストレージとしてData Lake Storage Gen1を選択した場合、次のいずれかのアクションを実行してください:

- マネージドサービスアイデンティティ認証メソッドを選択する場合、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下の表では、`StorageCredentialParams`で構成する必要があるパラメータについて説明しています。

  | **Parameter**                            | **Required** | **Description**                                              |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい          | マネージドサービスアイデンティティ認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |

- サービスプリンシパル認証メソッドを選択する場合、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下の表では、`StorageCredentialParams`で構成する必要があるパラメータについて説明しています。

  | **Parameter**                 | **Required** | **Description**                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい          | サービスプリンシパルのクライアント（アプリケーション）IDです。        |
  | azure.adls1.oauth2_credential | はい          | 新しいクライアント（アプリケーション）シークレットの値です。    |
  | azure.adls1.oauth2_endpoint   | はい          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

###### Azure Data Lake Storage Gen2
Data Lake Storage Gen2 を Hudi クラスターのストレージとして選択する場合、次のいずれかのアクションを実行します。

- マネージド ID 認証メソッドを選択する場合、`StorageCredentialParams` を次のように設定します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  次の表に、`StorageCredentialParams` で構成する必要があるパラメータについて説明します。

  | **パラメータ**                          | **必須** | **説明**                                                      |
  | ------------------------------------- | ---------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes      | マネージド ID 認証メソッドを有効にするかどうかを指定します。値を `true` に設定します。              |
  | azure.adls2.oauth2_tenant_id            | Yes      | アクセスするデータのテナントの ID です。                                      |
  | azure.adls2.oauth2_client_id            | Yes      | マネージド ID のクライアント（アプリケーション）ID です。                                    |

- 共有キー認証メソッドを選択する場合、`StorageCredentialParams` を次のように設定します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次の表に、`StorageCredentialParams` で構成する必要があるパラメータについて説明します。

  | **パラメータ**               | **必須** | **説明**                                                      |
  | --------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.storage_account | Yes      | Data Lake Storage Gen2 ストレージアカウントのユーザー名です。                      |
  | azure.adls2.shared_key      | Yes      | Data Lake Storage Gen2 ストレージアカウントの共有キーです。                        |

- サービス プリンシパル認証メソッドを選択する場合、`StorageCredentialParams` を次のように設定します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  次の表に、`StorageCredentialParams` で構成する必要があるパラメータについて説明します。

  | **パラメータ**                    | **必須** | **説明**                                                      |
  | -------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id     | Yes      | サービス プリンシパルのクライアント（アプリケーション）ID です。                           |
  | azure.adls2.oauth2_client_secret | Yes      | 新しいクライアント（アプリケーション）シークレットの値です。                         |
  | azure.adls2.oauth2_client_endpoint | Yes      | サービス プリンシパルまたはアプリケーションの OAuth 2.0 トークン エンドポイント（v1）です。     |

##### Google GCS

Hudi カタログは、バージョン 3.0 以降で Google GCS をサポートします。

Hudi クラスターのストレージとして Google GCS を選択する場合、次のいずれかのアクションを実行します。

- VM ベースの認証メソッドを選択する場合、`StorageCredentialParams` を次のように設定します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次の表に、`StorageCredentialParams` で構成する必要があるパラメータについて説明します。

  | **パラメータ**                                       | **デフォルト値** | **例**                        | **説明**                                                      |
  | -------------------------------------------------- | ----------------- | ----------------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account           | false             | true                          | 直接 Compute Engine にバインドされたサービス アカウントを使用するかどうかを指定します。 |

- サービス アカウントベースの認証メソッドを選択する場合、`StorageCredentialParams` を次のように設定します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  次の表に、`StorageCredentialParams` で構成する必要があるパラメータについて説明します。

  | **パラメータ**                          | **デフォルト値** | **例**                                                                      | **説明**                                                      |
  | ------------------------------------- | ----------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービス アカウントの作成時に生成された JSON ファイル内の電子メール アドレスです。  |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                             | サービス アカウントの作成時に生成された JSON ファイル内のプライベート キー ID です。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"            | サービス アカウントの作成時に生成された JSON ファイル内のプライベート キーです。    |

- 擬装ベースの認証メソッドを選択する場合、`StorageCredentialParams` を次のように設定します。

  - VM インスタンスをサービス アカウントに擬装させる場合:

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次の表に、`StorageCredentialParams` で構成する必要があるパラメータについて説明します。

    | **パラメータ**                                      | **デフォルト値** | **例**                        | **説明**                                                      |
    | ------------------------------------------------- | ----------------- | ----------------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account         | false             | true                          | 直接 Compute Engine にバインドされたサービス アカウントを使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account              | ""                | "hello"                       | 擬装したいサービス アカウントです。                            |

  - サービス アカウント（一時的にメタ サービス アカウントと呼ばれます）を別のサービス アカウント（一時的にデータ サービス アカウントと呼ばれます）に擬装させる場合:

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表に、`StorageCredentialParams` で構成する必要があるパラメータについて説明します。

    | **パラメータ**                          | **デフォルト値** | **例**                                                                      | **説明**                                                      |
    | ------------------------------------- | ----------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタ サービス アカウントの作成時に生成された JSON ファイル内の電子メール アドレスです。  |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                             | メタ サービス アカウントの作成時に生成された JSON ファイル内のプライベート キー ID です。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"            | メタ サービス アカウントの作成時に生成された JSON ファイル内のプライベート キーです。    |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                                    | 擬装したいデータ サービス アカウントです。 |

#### MetadataUpdateParams

StarRocks が Hudi のキャッシュされたメタデータを更新する方法に関するパラメータのセットです。このパラメータ セットはオプションです。

StarRocks はデフォルトで [自動非同期更新ポリシー](#付録-メタデータの自動非同期更新の理解) を実装します。

ほとんどの場合、`MetadataUpdateParams` を無視し、内部のこれらのパラメータのデフォルト値を調整する必要はありません。これらのパラメータのデフォルト値は、すでに使いやすいパフォーマンスを提供しています。

ただし、Hudi のデータ更新の頻度が高い場合は、これらのパラメータを調整して自動非同期更新のパフォーマンスをさらに最適化できます。

> **注記**
>
> ほとんどの場合、Hudi のデータが 1 時間以下の粒度で更新される場合、データ更新の頻度は高いと見なされます。

| パラメータ                      | 必須 | 説明                                                    |
|--------------------------------| ---- | ------------------------------------------------------- |
| enable_metastore_cache          | No   | StarRocks が Hudi テーブルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。値 `true` はキャッシュを有効にし、値 `false` はキャッシュを無効にします。 |
| enable_remote_file_cache        | No   | StarRocks が Hudi テーブルまたはパーティションの基礎データ ファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。値 `true` はキャッシュを有効にし、値 `false` はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec | No | StarRocks が自身で Hudi テーブルまたはパーティションのメタデータを非同期に更新する時間間隔です。単位: 秒。デフォルト値: `7200`、つまり 2 時間。 |
| remote_file_cache_refresh_interval_sec | いいえ | StarRocksが自身でキャッシュされているHudiテーブルまたはパーティションの基になるデータファイルのメタデータを非同期に更新する間隔。単位: 秒。デフォルト値: `60`。 |
| metastore_cache_ttl_sec | いいえ | StarRocksが自身でキャッシュされているHudiテーブルまたはパーティションのメタデータを自動的に破棄する間隔。単位: 秒。デフォルト値: `86400` (24時間)。 |
| remote_file_cache_ttl_sec | いいえ | StarRocksが自身でキャッシュされているHudiテーブルまたはパーティションの基になるデータファイルのメタデータを自動的に破棄する間隔。単位: 秒。デフォルト値: `129600` (36時間)。 |

### 例

以下の例は、Hudiクラスタからデータをクエリするために、Hiveメタストアを使用する場合は`hudi_catalog_hms`、AWS Glueを使用する場合は`hudi_catalog_glue`という名前のHudiカタログを作成します。

#### HDFS

ストレージとしてHDFSを使用する場合は、次のようにコマンドを実行してください:

```SQL
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
);
```

#### AWS S3

##### インスタンスプロファイルベースの資格情報を選択した場合

- HudiクラスタでHiveメタストアを使用する場合は、次のようにコマンドを実行してください:

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HudiクラスタでAWS Glueを使用する場合は、次のようにコマンドを実行してください:

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

##### 仮定ロールベースの資格情報を選択した場合

- HudiクラスタでHiveメタストアを使用する場合は、次のようにコマンドを実行してください:

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HudiクラスタでAWS Glueを使用する場合は、次のようにコマンドを実行してください:

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

##### IAMユーザーベースの資格情報を選択した場合

- HudiクラスタでHiveメタストアを使用する場合は、次のようにコマンドを実行してください:

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HudiクラスタでAWS Glueを使用する場合は、次のようにコマンドを実行してください:

  ```SQL
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

#### S3互換のストレージシステム

例として、MinIOを使用する場合は、次のようにコマンドを実行してください:

```SQL
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
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

- 共有キー認証方法を選択した場合は、次のようにコマンドを実行してください:

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SASトークン認証方法を選択した場合は、次のようにコマンドを実行してください:

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- サービスプリンシパル認証方法を選択した場合は、次のようにコマンドを実行してください:

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- サービスプリンシパル認証方法を選択した場合は、次のようにコマンドを実行してください:

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- 管理されたアイデンティティ認証方法を選択した場合は、次のようにコマンドを実行してください:

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- もしShared Key認証手法を選択した場合、以下のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- もしService Principal認証手法を選択した場合、以下のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- もしVMベースの認証手法を選択した場合、以下のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- もしサービスアカウントベースの認証手法を選択した場合、以下のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- もしインパーソネーションベースの認証手法を選択した場合：

  - VMインスタンスがサービスアカウントを偽装する場合、以下のようなコマンドを実行してください：

    ```SQL
    CREATE EXTERNAL CATALOG hudi_catalog_hms
    PROPERTIES
    (
        "type" = "hudi",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - もしサービスアカウントが他のサービスアカウントを偽装する場合、以下のようなコマンドを実行してください：

    ```SQL
    CREATE EXTERNAL CATALOG hudi_catalog_hms
    PROPERTIES
    (
        "type" = "hudi",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Hudiカタログのビュー

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) を使用して、現在のStarRocksクラスター内のすべてのカタログをクエリできます：

```SQL
SHOW CATALOGS;
```

[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を使用して、外部カタログの作成ステートメントをクエリできます。以下の例は、Hudiカタログ名`hudi_catalog_glue`の作成ステートメントをクエリしています：

```SQL
SHOW CREATE CATALOG hudi_catalog_glue;
```

## Hudiカタログおよびそれに含まれるデータベースに切り替える

以下の方法のいずれかを使用して、Hudiカタログおよびそれに含まれるデータベースに切り替えることができます：

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) を使用して、セッション内のHudiカタログを指定し、それから[USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用してアクティブなデータベースを指定します：

  ```SQL
  -- セッション内で指定されたカタログに切り替える：
  SET CATALOG <catalog_name>
  -- セッション内のアクティブなデータベースを指定する：
  USE <db_name>
  ```

- 直接に[USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、Hudiカタログとそれに含まれるデータベースに切り替える：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Hudiカタログを削除する

[Hudiカタログ](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) を使用して、外部カタログを削除することができます。

以下の例は、`hudi_catalog_glue` という名前のHudiカタログを削除しています：

```SQL
DROP Catalog hudi_catalog_glue;
```

## Hudiテーブルのスキーマを表示する

Hudiテーブルのスキーマを表示するためには、以下の構文のいずれかを使用できます：

- スキーマを表示する

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Hudiテーブルをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を使用して、Hudiクラスター内のデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Hudiカタログとそれに含まれるデータベースに切り替える](#hudiカタログおよびそれに含まれるデータベースに切り替える)。

3. 指定したデータベース内の宛先テーブルをクエリするために[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用します：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hudiからデータをロードする

OLAPテーブルである`olap_tbl` を持っている場合、以下のようにデータを変換してロードすることができます：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hudi_table
```

## メタデータキャッシュの手動または自動更新

### 手動更新

デフォルトでは、StarRocksはHudiのメタデータをキャッシュし、メタデータを非同期モードで自動的に更新してパフォーマンスを向上させています。さらに、Hudiテーブルでスキーマ変更やテーブルの更新が行われた後、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) を使用してそのメタデータを手動で更新することもできます。これにより、StarRocksが最新のメタデータをすみやかに取得し、適切な実行計画を生成できるようになります：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自動増分更新

自動非同期更新ポリシーとは異なり、自動増分更新ポリシーではStarRocksクラスター内のFEは、Hiveメタストアから列の追加、パーティションの削除、およびデータの更新などのイベントを読み取ることができます。StarRocksは、これらのイベントに基づいてFEにキャッシュされたメタデータを自動的に更新します。つまり、Hudiテーブルのメタデータを手動で更新する必要はありません。

自動増分更新を有効にするには、次の手順に従ってください：

#### 手順1：Hiveメタストアのイベントリスナーを構成する

Hiveメタストアv2.xおよびv3.xの両方はイベントリスナーを構成することができます。この手順では、Hiveメタストアv3.1.2用のイベントリスナー構成を例として示します。以下の構成項目を **$HiveMetastore/conf/hive-site.xml** ファイルに追加し、その後Hiveメタストアを再起動してください：

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
```json
    + {R}
```