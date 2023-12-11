---
displayed_sidebar: "Japanese"
---

# Hiveカタログ

Hiveカタログは、StarRocksがv2.4以降でサポートしている外部カタログの一種です。Hiveカタログでは、以下の操作が可能です。

- 手動でテーブルを作成する必要なく、Hiveに保存されているデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) や非同期マテリアライズドビュー（v2.5以降でサポート）を使用して、Hiveに保存されているデータを処理し、そのデータをStarRocksに読み込むことができます。
- StarRocksで、Hiveデータベースやテーブルを作成したり削除したりする操作を実行したり、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用してStarRocksテーブルからParquet形式のHiveテーブルにデータをシンクすることができます（この機能はv3.2以降でサポート）。

HiveクラスタでのSQLワークロードの正常な実行を確保するには、StarRocksクラスタが2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCSなどのオブジェクトストレージ、またはその他のS3互換のストレージシステム（例: MinIO）
- HiveメタストアまたはAWS Glueなどのメタストア

  > **注意**
  >
  > ストレージとしてAWS S3を選択した場合、HMSまたはAWS Glueをメタストアとして使用できます。その他のストレージシステムを選択した場合、HMSのみをメタストアとして使用できます。

## 使用上の注意

StarRocksがサポートするHiveのファイルフォーマットは、Parquet、ORC、CSV、Avro、RCFile、SequenceFileです。

- Parquetファイルは、SNAPPY、LZ4、ZSTD、GZIPおよびNO_COMPRESSIONの圧縮形式をサポートしています。v3.1.5以降、ParquetファイルはまたLZO圧縮形式をサポートしています。
- ORCファイルは、ZLIB、SNAPPY、LZO、LZ4、ZSTD、およびNO_COMPRESSIONの圧縮形式をサポートしています。
- CSVファイルは、v3.1.5以降、LZO圧縮形式をサポートしています。

StarRocksがサポートしないHiveのデータ型は、INTERVAL、BINARY、UNIONです。さらに、CSV形式のHiveテーブルにおいて、StarRocksはMAPおよびSTRUCTデータ型をサポートしていません。

Hiveカタログを使用してデータをクエリすることができますが、Hiveカタログを使用してHiveクラスタに対してデータの削除、削除、または挿入を行うことはできません。

## 統合の準備

Hiveカタログを作成する前に、StarRocksクラスタがHiveクラスタのストレージシステムおよびメタストアと統合できることを確認してください。

### AWS IAM

HiveクラスタがAWS S3をストレージとして使用する場合、またはAWS Glueをメタストアとして使用する場合は、適切な認証方法を選択し、StarRocksクラスタが関連するAWSクラウドリソースにアクセスできるように必要な準備を行ってください。

以下の認証方法が推奨されています。

- インスタンスプロファイル
- 仮定された役割（Assumed role）
- IAMユーザー

上記の3つの認証方法のうち、インスタンスプロファイルが最も一般的に使用されています。

詳細は、[AWS IAMでの認証の準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとしてHDFSを選択した場合、StarRocksクラスタを次のように構成してください。

- （オプション）HDFSクラスタにアクセスするためのユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスタおよびHiveメタストアにアクセスします。各FEの **fe/conf/hadoop_env.sh** ファイルおよび各BEの **be/conf/hadoop_env.sh** ファイルの先頭に `export HADOOP_USER_NAME="<ユーザー名>"` を追加してユーザー名を設定できます。これらのファイルでユーザー名を設定した後は、各FEおよび各BEを再起動してパラメータ設定が有効になります。StarRocksクラスタにつき1つのユーザー名のみを設定できます。
- Hiveデータをクエリする際、StarRocksクラスタのFEおよびBEはHDFSクライアントを使用してHDFSクラスタにアクセスします。ほとんどのケースでは、この目的を達成するためにStarRocksクラスタを構成する必要はありません。StarRocksはデフォルトの構成を使用してHDFSクライアントを起動します。StarRocksクラスタを構成する必要があるのは、次の状況のみです。
  - HDFSクラスタの高可用性（HA）が有効になっている場合: HDFSクラスタの **hdfs-site.xml** ファイルを各FEの **$FE_HOME/conf** パスおよび各BEの **$BE_HOME/conf** パスに追加します。
  - HDFSクラスタのView File System（ViewFs）が有効になっている場合: HDFSクラスタの **core-site.xml** ファイルを各FEの **$FE_HOME/conf** パスおよび各BEの **$BE_HOME/conf** パスに追加します。

> **注意**
>
> クエリを送信する際に不明なホストを示すエラーが返された場合は、HDFSクラスタノードのホスト名とIPアドレスのマッピングを **/etc/hosts** パスに追加する必要があります。

### Kerberos認証

HDFSクラスタまたはHiveメタストアでKerberos認証が有効になっている場合は、StarRocksクラスタを次のように構成してください。

- 各FEおよび各BEで`kinit -kt keytab_path principal`コマンドを実行して、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するためには、HDFSクラスタとHiveメタストアにアクセスする権限が必要です。このコマンドでKDCにアクセスする際にはタイムリー性が重要です。そのため、このコマンドを定期的に実行するためにcronを使用する必要があります。
- 各FEの **$FE_HOME/conf/fe.conf** ファイルおよび各BEの **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。ここでの `/etc/krb5.conf` は **krb5.conf** ファイルの保存先パスです。必要に応じてパスを変更できます。

## Hiveカタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <カタログ名>
[COMMENT <説明>]
PROPERTIES
(
    "type" = "hive",
    GeneralParams,
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### パラメータ

#### カタログ名

Hiveカタログの名前。以下の命名規則が適用されます。

- 名前には英字、数字（0-9）、アンダースコア(_) を含めることができます。英字で始まる必要があります。
- 名前は大文字と小文字を区別し、1023文字を超えてはいけません。

#### 説明

Hiveカタログの説明。このパラメータはオプションです。

#### type

データソースの種類。値を `hive` に設定します。

#### GeneralParams

一連の一般パラメータ。

`GeneralParams` で設定できるパラメータについて、以下のテーブルに記載します。

| パラメータ                | 必須     | 説明                                                     |
| ------------------------ | -------- | ------------------------------------------------------------ |
| enable_recursive_listing | いいえ    | StarRocksがテーブルおよびそのパーティションと、テーブルおよびそのパーティションの物理位置内のサブディレクトリからデータを読み取るかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。`true` はサブディレクトリを再帰的にリストし、`false` はサブディレクトリを無視することを指定します。 |

#### MetastoreParams

データソースのメタストアとの統合に関する一連のパラメータ。

##### Hiveメタストア

データソースのメタストアとしてHiveメタストアを選択した場合、次のように`MetastoreParams` を構成します。

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hiveメタストアURI>"
```

> **注意**
>
> Hiveデータをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを `/etc/hosts` パスに追加する必要があります。そうしないと、クエリを開始する際にStarRocksはHiveメタストアにアクセスできなくなることがあります。

以下のテーブルに、`MetastoreParams` で構成する必要があるパラメータについて記載します。

| パラメータ           | 必須     | 説明                                                     |
| ------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type | はい    | Hiveクラスタで使用するメタストアのタイプ。値を `hive` に設定します。 |
| hive.metastore.uris | はい    | HiveメタストアのURI。フォーマット: `thrift://<metaストアIPアドレス>:<メタストアポート>`。<br />Hiveメタストアの高可用性（HA）が有効になっている場合、複数のメタストアURIを指定し、カンマ（`,`）で区切ってください。例: `"thrift://<メタストアIPアドレス1>:<メタストアポート1>,thrift://<メタストアIPアドレス2>:<メタストアポート2>,thrift://<メタストアIPアドレス3>:<メタストアポート3>"`。 |

##### AWS Glue

データソースのメタストアとしてAWS Glueを選択し、かつストレージとしてAWS S3を選択した場合、次のいずれかのアクションを実行します。

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams` を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glueリージョン>"
  ```

- 次の設定で、推定されるロールベースの認証メソッドを選択するには、`MetastoreParams` を以下のように設定します：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAMユーザーベースの認証メソッドを選択するには、`MetastoreParams` を以下のように設定します：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

以下の表に、`MetastoreParams` で設定する必要があるパラメータについて説明しています。

| パラメータ                     | 必須     | 説明                                                         |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type           | はい    | Hiveクラスタで使用するメタストアのタイプ。値を `glue` に設定します。  |
| aws.glue.use_instance_profile | はい    | インスタンスプロファイルベースの認証メソッドおよび推定されるロールベースの認証を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。  |
| aws.glue.iam_role_arn         | いいえ  | AWS Glueデータカタログに権限を持つIAMロールのARN。AWS Glueへのアクセスに推定されるロールベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。  |
| aws.glue.region               | はい    | AWS Glue Data Catalogが存在するリージョン。例: `us-west-1`。  |
| aws.glue.access_key           | いいえ  | AWS IAMユーザーのアクセスキー。IAMユーザーベースの認証メソッドを使用してAWS Glueにアクセスする場合は、このパラメータを指定する必要があります。  |
| aws.glue.secret_key           | いいえ  | AWS IAMユーザーのシークレットキー。IAMユーザーベースの認証メソッドを使用してAWS Glueにアクセスする場合は、このパラメータを指定する必要があります。  |

AWS Glueにアクセスする認証メソッドを選択し、AWS IAMコンソールでアクセス制御ポリシーを構成する方法についての情報については、[AWS Glueへのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocksがストレージシステムと統合する方法に関するパラメータセットです。このパラメータセットはオプションです。

ストレージとしてHDFSを使用する場合、`StorageCredentialParams`を設定する必要はありません。

ストレージとしてAWS S3、その他のS3互換ストレージシステム、Microsoft Azure Storage、またはGoogle GCSを使用する場合は、`StorageCredentialParams`を設定する必要があります。

##### AWS S3

HiveクラスタのストレージとしてAWS S3を選択する場合は、次のアクションのいずれかを実行してください：

- インスタンスプロファイルベースの認証メソッドを選択するには、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 推定されるロールベースの認証メソッドを選択するには、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーベースの認証メソッドを選択するには、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表に、`StorageCredentialParams` で設定する必要があるパラメータについて説明しています。

| パラメータ                   | 必須     | 説明                                                   |
| --------------------------- | -------- | ------------------------------------------------------ |
| aws.s3.use_instance_profile | はい    | インスタンスプロファイルベースの認証メソッドおよび推定されるロールベースの認証メソッドを有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。  |
| aws.s3.iam_role_arn         | いいえ  | AWS S3バケットに権限を持つIAMロールのARN。AWS S3へのアクセスに推定されるロールベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。  |
| aws.s3.region               | はい    | AWS S3バケットが存在するリージョン。例: `us-west-1`。  |
| aws.s3.access_key           | いいえ  | IAMユーザーのアクセスキー。IAMユーザーベースの認証メソッドを使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。  |
| aws.s3.secret_key           | いいえ  | IAMユーザーのシークレットキー。IAMユーザーベースの認証メソッドを使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。  |

AWS S3へのアクセスの認証メソッドを選択し、AWS IAMコンソールでアクセス制御ポリシーを構成する方法については、[AWS S3へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

Hiveカタログは、v2.5以降でS3互換のストレージシステムをサポートしています。

MinIOなどのS3互換のストレージシステムを選択する場合は、次のように`StorageCredentialParams` を構成して、成功した統合を保証してください：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表に、`StorageCredentialParams` で設定する必要があるパラメータについて説明しています。

| パラメータ                        | 必須     | 説明                                                         |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | はい    | SSL接続を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。  |
| aws.s3.enable_path_style_access  | はい    | パススタイルのアクセスを有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。MinIOの場合、値を `true` に設定する必要があります。パススタイルURLは、次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部（オレゴン州）リージョンで`DOC-EXAMPLE-BUCKET1`というバケットを作成し、そのバケット内の`alice.jpg`オブジェクトにアクセスしたい場合、以下のパススタイルのURLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | はい    | AWS S3の代わりにS3互換のストレージシステムに接続するために使用されるエンドポイント。  |
| aws.s3.access_key                | はい    | IAMユーザーのアクセスキー。                                 |
| aws.s3.secret_key                | はい    | IAMユーザーのシークレットキー。                             |

##### Microsoft Azure Storage

Hiveカタログは、v3.0以降でMicrosoft Azure Storageをサポートしています。

###### Azure Blob Storage

Blob StorageをHiveクラスタのストレージとして選択する場合は、次のアクションのいずれかを実行してください：

- 共有キー認証メソッドを選択するには、`StorageCredentialParams` を以下のように構成します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下の表に、`StorageCredentialParams` で設定する必要があるパラメータについて説明しています。

  | **パラメータ**              | **必須** | **説明**                                         |
  | -------------------------- | -------- | ------------------------------------------------- |
  | azure.blob.storage_account | はい    | Blob Storageアカウントのユーザー名。                   |
  | azure.blob.shared_key      | はい    | Blob Storageアカウントの共有キー。             |

- SASトークン認証メソッドを選択するには、`StorageCredentialParams` を以下のように構成します：

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  以下の表に、`StorageCredentialParams` で設定する必要があるパラメータについて説明しています。

  | **パラメータ**             | **必須** | **説明**                                                     |
  | ------------------------- | -------- | ------------------------------------------------------------- |
  | azure.blob.account_name   | はい    | Blob Storageアカウントのユーザー名。                             |
  | azure.blob.container_name | はい    | データを格納するBlobコンテナの名前。                           |
  | azure.blob.sas_token      | はい    | Blob Storageアカウントへのアクセスに使用されるSASトークン。     |

###### Azure Data Lake Storage Gen1

Data Lake Storage Gen1をHiveクラスタのストレージとして選択する場合は、次のアクションのいずれかを実行してください：

- マネージドサービスアイデンティティ認証メソッドを選択するには、`StorageCredentialParams` を以下のように構成します：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下の表に、`StorageCredentialParams` で設定する必要があるパラメータについて説明しています。
| **パラメータ**                            | **必須** | **説明**                                           |
| ---------------------------------------- | ------------ | ------------------------------------------------------------ |
| azure.adls1.use_managed_service_identity | Yes          | Managed Service Identity認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |

- Service Principal認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                 | **必須** | **説明**                                           |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | Yes          | Service Principalのクライアント(アプリケーション)ID。       |
  | azure.adls1.oauth2_credential | Yes          | 作成された新しいクライアント(アプリケーション)シークレットの値。|
  | azure.adls1.oauth2_endpoint   | Yes          | Service PrincipalまたはアプリケーションのOAuth 2.0トークンエンドポイント(v1)。|

###### Azure Data Lake Storage Gen2

HiveクラスタのストレージとしてData Lake Storage Gen2を選択する場合、次のいずれかのアクションを実行します。

- Managed Identity認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                           | **必須** | **説明**                                           |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes          | Managed Identity認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | Yes          | アクセスしたいテナントのID。                          |
  | azure.adls2.oauth2_client_id            | Yes          | 管理されるIDのクライアント(アプリケーション)ID。         |

- Shared Key認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**               | **必須** | **説明**                                           |
  | --------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account | Yes          | Data Lake Storage Gen2ストレージアカウントのユーザー名。 |
  | azure.adls2.shared_key      | Yes          | Data Lake Storage Gen2ストレージアカウントの共有キー。|

- Service Principal認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                      | **必須** | **説明**                                           |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | Yes          | Service Principalのクライアント(アプリケーション)ID。       |
  | azure.adls2.oauth2_client_secret   | Yes          | 作成された新しいクライアント(アプリケーション)シークレットの値。|
  | azure.adls2.oauth2_client_endpoint | Yes          | Service PrincipalまたはアプリケーションのOAuth 2.0トークンエンドポイント(v1)。|

##### Google GCS

Hiveカタログはv3.0以降からGoogle GCSをサポートしています。

HiveクラスタのストレージとしてGoogle GCSを選択する場合、次のいずれかのアクションを実行します:

- VMベースの認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                           |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接Compute Engineにバインドされたサービスアカウントを使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                          | **デフォルト値** | **値の例**                                        | **説明**                                           |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウント作成時に生成されたJSONファイル内のメールアドレス。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウント作成時に生成されたJSONファイル内のプライベートキーID。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウント作成時に生成されたJSONファイル内のプライベートキー。 |

- 偽装ベースの認証方法を選択する場合、`StorageCredentialParams`を次のように設定します。

  - VMインスタンスをサービスアカウントに偽装させる:

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

    | **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                           |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接Compute Engineにバインドされたサービスアカウントを使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | 偽装したいサービスアカウント。                          |

  - サービスアカウント（一時的にmetaサービスアカウントとして名前が付けられている）を別のサービスアカウント（一時的にデータサービスアカウントとして名前が付けられている）に偽装させる:

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表は`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

    | **パラメータ**                          | **デフォルト値** | **値の例**                                        | **説明**                                           |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウント作成時に生成されたメタサービスアカウントのJSONファイル内のメールアドレス。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウント作成時に生成されたメタサービスアカウントのJSONファイル内のプライベートキーID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウント作成時に生成されたメタサービスアカウントのJSONファイル内のプライベートキー。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 偽装したいデータサービスアカウント。                   |

#### MetadataUpdateParams

StarRocksがHiveのキャッシュメタデータを更新する方法に関する一連のパラメータです。このパラメータセットはオプションです。

StarRocksはデフォルトで[自動非同期更新ポリシー](#appendix-understand-metadata-automatic-asynchronous-update)を実装しています。

ほとんどの場合、`MetadataUpdateParams`を無視して、これらのパラメータのデフォルト値を調整する必要はありません。デフォルト値で十分なパフォーマンスが提供されます。
しかしながら、Hiveのデータ更新頻度が高い場合、これらのパラメータを調整して、自動の非同期更新のパフォーマンスをさらに最適化することができます。

> **注意**
>
> ほとんどの場合、Hiveのデータが1時間以下の粒度で更新される場合、データ更新頻度が高いと見なされます。

| パラメータ                              | 必須     | 説明                                                  |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | いいえ    | StarRocksがHiveテーブルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および`false`。デフォルト値:`true`。`true` はキャッシュを有効にし、 `false` はキャッシュを無効にします。 |
| enable_remote_file_cache               | いいえ    | StarRocksがHiveテーブルまたはパーティションの基礎データファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および`false`。デフォルト値: `true`。`true` はキャッシュを有効にし、 `false` はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | いいえ    | StarRocksが自身でキャッシュされたHiveテーブルまたはパーティションのメタデータを非同期で更新する時間間隔。単位: 秒。デフォルト値: `7200`、つまり2時間。 |
| remote_file_cache_refresh_interval_sec | いいえ    | StarRocksが自身でキャッシュされたHiveテーブルまたはパーティションの基礎データファイルのメタデータを非同期で更新する時間間隔。単位: 秒。デフォルト値: `60`。 |
| metastore_cache_ttl_sec                | いいえ    | StarRocksが自身でキャッシュされたHiveテーブルまたはパーティションのメタデータを自動的に破棄する時間間隔。単位: 秒。デフォルト値: `86400`、つまり24時間。 |
| remote_file_cache_ttl_sec              | いいえ    | StarRocksが自身でキャッシュされたHiveテーブルまたはパーティションの基礎データファイルのメタデータを自動的に破棄する時間間隔。単位: 秒。デフォルト値: `129600`、つまり36時間。 |
| enable_cache_list_names                | いいえ    | StarRocksがHiveのパーティション名をキャッシュするかどうかを指定します。有効な値: `true` および`false`。デフォルト値: `true`。`true` はキャッシュを有効にし、 `false` はキャッシュを無効にします。 |

### 例

以下の例では、Hiveクラスタからデータをクエリする場合に、`hive_catalog_hms`または`hive_catalog_glue`というHiveカタログを作成します。

#### HDFS

ストレージとしてHDFSを使用する場合、次のようにコマンドを実行します:

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
);
```

#### AWS S3

##### インスタンスプロファイルベースの認証

- HiveクラスタでHiveメタストアを使用する場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HiveクラスタでAWS Glueを使用する場合、次のようにコマンドを実行します:

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

##### 仮定されたロールベースの認証

- HiveクラスタでHiveメタストアを使用する場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HiveクラスタでAWS Glueを使用する場合、次のようにコマンドを実行します:

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

##### IAMユーザーベースの認証

- HiveクラスタでHiveメタストアを使用する場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HiveクラスタでAWS Glueを使用する場合、次のようにコマンドを実行します:

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

#### S3互換ストレージシステム

例として、MinIOを使用する場合、次のようにコマンドを実行します:

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
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

- 共有キー認証方法を選択した場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SASトークン認証方法を選択した場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- サービス マネージド ID 認証メソッドを選択した場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```
- Service Principal authentication methodを選択する場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- Managed Identity認証メソッドを選択する場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- Shared Key認証メソッドを選択する場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- Service Principal authentication methodを選択する場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMベースの認証メソッドを選択する場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- サービスアカウントベースの認証メソッドを選択する場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- インパーソネーションベースの認証メソッドを選択する場合：

  - VMインスタンスがサービスアカウントを偽装する場合は、以下のようにコマンドを実行してください：

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - サービスアカウントが別のサービスアカウントを偽装する場合は、以下のようにコマンドを実行してください：

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Hiveカタログを表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスタ内のすべてのカタログをクエリすることができます：

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリすることもできます。次の例は、Hiveカタログが`hive_catalog_glue`という名前の作成ステートメントをクエリしています：

```SQL
SHOW CREATE CATALOG hive_catalog_glue;
```

## Hiveカタログとその中のデータベースに切り替える

次のいずれかの方法を使用して、Hiveカタログとその中のデータベースに切り替えることができます：

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションでHiveカタログを指定し、その後で[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します：

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替えます:
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定します:
  USE <db_name>
  ```

- [USE](../../sql-reference/sql-statements/data-definition/USE.md)を直接使用して、Hiveカタログおよびその中のデータベースに切り替えます：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Hiveカタログを削除

外部カタログを削除するには、[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用できます。

次の例は、`hive_catalog_glue`という名前のHiveカタログを削除しています：

```SQL
DROP Catalog hive_catalog_glue;
```

## Hiveテーブルのスキーマを表示

次の構文のいずれかを使用して、Hiveテーブルのスキーマを表示できます：

- スキーマを表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Hiveテーブルをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Hiveクラスタ内のデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Hiveカタログとその中のデータベースに切り替える](#hiveカタログとその中のデータベースに切り替える)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、指定されたデータベース内の宛先テーブルをクエリします：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hiveからデータをロードする

`olap_tbl`という名前のOLAPテーブルがあると仮定し、以下のようにデータを変換してロードできます：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hive_table
```

## Hiveテーブルおよびビューに権限を付与する

[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)ステートメントを使用して、Hiveカタログ内のすべてのテーブルまたはビューに対する特定のロールの権限を付与できます。

- ロールにHiveカタログ内のすべてのテーブルへのクエリ権限を付与する：

  ```SQL
  GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE <role_name>
  ```

- ロールにHiveカタログ内のすべてのビューへのクエリ権限を付与する：

  ```SQL
  GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE <role_name>
  ```

たとえば、以下のコマンドを使用して、`hive_role_table`という名前のロールを作成し、Hiveカタログ`hive_catalog`に切り替え、その後、`hive_role_table`にHiveカタログ`hive_catalog`内のすべてのテーブルおよびビューのクエリ権限を付与します：

```SQL
-- hive_role_tableというロールを作成します。
CREATE ROLE hive_role_table;

-- Hiveカタログhive_catalogに切り替えます。
SET CATALOG hive_catalog;

-- hive_role_tableにHiveカタログhive_catalog内のすべてのテーブルのクエリ権限を付与します。
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE hive_role_table;
```
-- ユーザークエリのテクニカルドキュメントを日本語に翻訳します。

```
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE hive_role_table;
```

## Hiveデータベースの作成

StarRocksの内部カタログと同様に、Hiveカタログに対する[CREATE DATABASE](../../administration/privilege_item.md#catalog)権限がある場合は、そのHiveカタログ内にデータベースを作成するために[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)ステートメントを使用できます。この機能はv3.2以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Hiveカタログに切り替える](#switch-to-a-hive-catalog-and-a-database-in-it) と、次のステートメントを使用してそのカタログ内にHiveデータベースを作成します：

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location`パラメータは、HDFSまたはクラウドストレージいずれかにデータベースを作成するファイルパスを指定します。

- HiveクラスタのメタストアとしてHiveメタストアを使用する場合、`location`パラメータのデフォルト値は`<warehouse_location>/<database_name.db>`であり、データベース作成時にそのパラメータを指定しない限り、Hiveメタストアでサポートされています。
- HiveクラスタのメタストアとしてAWS Glueを使用する場合、`location`パラメータにデフォルト値がないため、データベース作成時にそのパラメータを指定する必要があります。

`prefix`は使用するストレージシステムによって異なります：

| **Storage system**                                         | **`Prefix`** **value**                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                       | `hdfs`                                                       |
| Google GCS                                                 | `gs`                                                         |
| Azure Blob Storage                                         | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可する場合、`prefix`は`wasb`です。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可する場合、`prefix`は`wasbs`です。</li></ul> |
| Azure Data Lake Storage Gen1                               | `adl`                                                        |
| Azure Data Lake Storage Gen2                               | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可する場合、`prefix`は`abfs`です。</li><li>ストレージアカウントがHTTP経由でアクセスを許可する場合、`prefix`は`abfss`です。</li></ul> |
| AWS S3またはその他のS3互換ストレージ（例：MinIO）         | `s3`                                                         |

## Hiveデータベースの削除

StarRocksの内部データベースと同様に、Hiveデータベースに対する[DROP](../../administration/privilege_item.md#database)権限がある場合は、そのHiveデータベースを削除するために[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md)ステートメントを使用できます。この機能はv3.2以降でサポートされています。空のデータベースのみを削除できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

Hiveデータベースを削除すると、HDFSクラスタまたはクラウドストレージ上のデータベースのファイルパスは、データベースと一緒に削除されません。

[Hiveカタログに切り替える](#switch-to-a-hive-catalog-and-a-database-in-it) と、次のステートメントを使用してそのカタログ内のHiveデータベースを削除します：

```SQL
DROP DATABASE <database_name>
```

## Hiveテーブルの作成

StarRocksの内部データベースと同様に、Hiveデータベースに対する[CREATE TABLE](../../administration/privilege_item.md#database)権限がある場合は、そのHiveデータベース内にメインテーブルを作成するために[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)または[CREATE TABLE AS SELECT(CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)ステートメントを使用できます。この機能はv3.2以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Hiveカタログおよびそのデータベースに切り替える](#switch-to-a-hive-catalog-and-a-database-in-it) と、次のシンタックスを使用してそのデータベース内にHiveメインテーブルを作成します。

### シンタックス

```SQL
CREATE TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...
partition_column_definition1,partition_column_definition2...])
[partition_desc]
[PROPERTIES ("key" = "value", ...)]
[AS SELECT query]
```

### パラメータ

#### column_definition

`column_definition`の構文は以下の通りです：

```SQL
col_name col_type [COMMENT 'comment']
```

次の表はパラメータを説明しています。

| パラメータ | 説明                                                      |
| --------- | ----------------------------------------------------------- |
| col_name  | 列の名前                                                  |
| col_type  | 列のデータ型。次のデータ型がサポートされています：TINYINT、SMALLINT、INT、BIGINT、FLOAT、DOUBLE、DECIMAL、DATE、DATETIME、CHAR、VARCHAR[(length)]、ARRAY、MAP、STRUCT。LARGEINT、HLL、BITMAPのデータ型はサポートされていません。 |

> **注意**
>
> すべての非パーティション列は、デフォルト値として`NULL`を使用する必要があります。つまり、テーブル作成ステートメントの各非パーティション列について`DEFAULT "NULL"`を指定する必要があります。さらに、パーティション列は非パーティション列に続いて定義する必要があり、デフォルト値として`NULL`を使用することはできません。

#### partition_desc

`partition_desc`の構文は以下の通りです：

```SQL
PARTITION BY (par_col1[, par_col2...])
```

現在、StarRocksはID変換のみをサポートしているため、StarRocksは各ユニークなパーティション値ごとにパーティションを作成します。

> **注意**
>
> パーティション列は非パーティション列に続いて定義する必要があります。パーティション列はFLOAT、DOUBLE、DECIMAL、DATETIMEを含むすべてのデータ型をサポートし、デフォルト値として`NULL`を使用することはできません。さらに、`partition_desc`で宣言されたパーティション列の順序は、`column_definition`で定義された列の順序と一致している必要があります。

#### PROPERTIES

`properties`内で`"key" = "value"`形式でテーブル属性を指定できます。

次の表はいくつかの主要なプロパティを説明しています。

| **Property**      | **Description**                                              |
| ----------------- | ------------------------------------------------------------ |
| location          | 管理テーブルを作成したいファイルパス。メタストアとしてHMSを使用する場合、Hiveカタログの現在のデフォルトファイルパスにテーブルが作成されるため、`location`パラメータを指定する必要はありません。メタデータサービスとしてAWS Glueを使用する場合：<ul><li>テーブルを作成したいデータベースの`location`パラメータを指定した場合、テーブルの`location`パラメータは指定する必要はありません。そのような場合、テーブルは所属するデータベースのファイルパスにデフォルトでなります。</li><li>データベースの`location`を指定していない場合、テーブルの`location`パラメータを指定する必要があります。</li></ul> |
| file_format       | 管理テーブルのファイルフォーマット。Parquet形式のみがサポートされています。デフォルト値：`parquet`。 |
| compression_codec | 管理テーブルで使用される圧縮アルゴリズム。サポートされている圧縮アルゴリズムはSNAPPY、GZIP、ZSTD、LZ4です。デフォルト値：`gzip`。 |

### 例

1. `id`と`score`の2つの列からなる非パーティションテーブル`unpartition_tbl`を作成する。

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. `action`、`id`、`dt`の3つの列からなるパーティションされたテーブル`partition_tbl_1`を作成する。ここで`id`および`dt`はパーティション列として定義されています。

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. 既存のテーブル`partition_tbl_1`をクエリし、クエリ結果に基づいてパーティションされたテーブル`partition_tbl_2`を作成する。`partition_tbl_2`では`id`と`dt`がパーティション列として定義されています。

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (k1, k2)
   AS SELECT * from partition_tbl_1;
   ```

## Hiveテーブルへのデータのシンク
StarRocksの内部テーブルと同様に、Hiveテーブル（管理されたテーブルまたは外部テーブルのいずれか）に[INSERT](../../administration/privilege_item.md#table)権限がある場合、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ステートメントを使用してStarRocksテーブルのデータをそのHiveテーブルにシンクすることができます（現在はParquet形式のHiveテーブルのみサポートされています）。 この機能はv3.2以降でサポートされています。外部テーブルにデータをシンクすることはデフォルトで無効になっています。外部テーブルにデータをシンクするには、[system variable `ENABLE_WRITE_HIVE_EXTERNAL_TABLE`](../../reference/System_variable.md)を`true`に設定する必要があります。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、権限を付与および取り消すことができます。

[Hiveカタログおよびその中のデータベースに切り替える](#switch-to-a-hive-catalog-and-a-database-in-it) 、そして次の構文を使用して、StarRocksテーブルのデータをそのデータベース内のParquet形式のHiveテーブルにシンクします。

### 構文

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- データを指定したパーティションにシンクする場合、次の構文を使用します:
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

> **通知**
>
> パーティション列は`NULL`値を許容しません。したがって、Hiveテーブルのパーティション列に空の値が読み込まれないようにしてください。

### パラメータ

| パラメータ   | 説明                                     |
| ----------- | --------------------------------------- |
| INTO        | StarRocksテーブルのデータをHiveテーブルに追加します。     |
| OVERWRITE   | StarRocksテーブルのデータでHiveテーブルの既存データを上書きします。 |
| column_name | データをロードする宛先の列の名前。1つまたは複数の列を指定できます。複数の列を指定する場合は、それらをカンマ（`,`）で区切ります。指定できるのは実際にHiveテーブルに存在する列のみであり、指定した宛先の列にはHiveテーブルのパーティション列を含める必要があります。指定した宛先の列は、順番にStarRocksテーブルの列に1対1でマッピングされます。宛先の列名は何であろうと関係ありません。宛先の列を指定しない場合、データはHiveテーブルのすべての列に読み込まれます。StarRocksテーブルのパーティション列以外の列がHiveテーブルのどの列にもマッピングされない場合、StarRocksはデフォルト値`NULL`をHiveテーブルの列に書き込みます。INSERTステートメントには、返された列の型が宛先の列のデータ型と異なるクエリが含まれている場合、不一致の列に対して暗黙の変換が行われます。変換に失敗すると、構文の解析エラーが返されます。 |
| expression  | 宛先の列に値を割り当てる式。                     |
| DEFAULT     | 宛先の列にデフォルト値を割り当てます。               |
| query       | Hiveテーブルにロードされる、SELECTクエリステートメント。StarRocksでサポートされている任意のSQLステートメントを使用できます。 |
| PARTITION   | データをロードするパーティション。このプロパティには、Hiveテーブルのすべてのパーティション列を指定する必要があります。このプロパティで指定したパーティション列は、テーブルの作成ステートメントで定義したパーティション列とは異なる順序であっても構いません。このプロパティを指定すると、`column_name`プロパティを指定できません。 |

### 例

1. `partition_tbl_1`テーブルに3つのデータ行を挿入:

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. 単純な計算を含むSELECTクエリの結果を`partition_tbl_1`テーブルに挿入:

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. `partition_tbl_1`テーブルからデータを読み取るSELECTクエリの結果を同じテーブルに挿入:

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. 2つの条件`dt='2023-09-01'`および`id=1`に一致するパーティションにSELECTクエリの結果を挿入:

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT INTO partition_tbl_2 PARTITION(dt='2023-09-01',id=1) SELECT 'order';
   ```

5. 2つの条件`dt='2023-09-01'`および`id=1`に一致するパーティションのすべての`action`列値を`close`に上書き:

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT OVERWRITE partition_tbl_1 PARTITION(dt='2023-09-01',id=1) SELECT 'close';
   ```

## Hiveテーブルの削除

StarRocksの内部テーブルと同様に、Hiveテーブルに[DROP](../../administration/privilege_item.md#table)権限がある場合、そのHiveテーブルを削除するために[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)ステートメントを使用できます。この機能はv3.1以降でサポートされています。ただし、現在、StarRocksはHiveの管理されたテーブルのみを削除することができます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、権限を付与および取り消すことができます。

Hiveテーブルを削除する場合は、`FORCE`キーワードを`DROP TABLE`ステートメントで指定する必要があります。この操作が完了すると、テーブルのファイルパスは保持されますが、テーブルのデータはHDFSクラスタまたはクラウドストレージ上で完全に削除されます。Hiveテーブルを削除する操作を行う際は注意が必要です。

[Hiveカタログおよびその中のデータベースに切り替える](#switch-to-a-hive-catalog-and-a-database-in-it) 、そしてそのデータベース内のHiveテーブルを削除するために以下のステートメントを使用します。

```SQL
DROP TABLE <table_name> FORCE
```

## 手動または自動的なメタデータキャッシュの更新

### 手動更新

デフォルトでは、StarRocksはHiveのメタデータをキャッシュし、パフォーマンスを向上させるためにメタデータを非同期モードで自動更新します。また、Hiveテーブルでスキーマ変更やテーブル更新を行った後、手動でメタデータを更新するために [REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) を使用することもできます。これにより、StarRocksはできるだけ早く最新のメタデータを取得し、適切な実行プランを生成できます。

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

以下の状況で、メタデータを手動で更新する必要があります。

- 既存のパーティション内のデータファイルが変更される（例：`INSERT OVERWRITE ... PARTITION ...`コマンドを実行するなど）。
- Hiveテーブルのスキーマに変更が加えられる。
- `DROP`ステートメントを使用して既存のHiveテーブルを削除し、同じ名前の新しいHiveテーブルを作成した場合。
- Hiveカタログの作成時に `PROPERTIES` で "`enable_cache_list_names` = `true`" を指定し、Hiveクラスタ上で新しく作成したパーティションをクエリしたい場合。

  > **注意**
  >
  > StarRocksはv2.5.5以降で定期的なHiveメタデータキャッシュの更新機能を提供しています。詳細については、このトピックの下の"[定期的なメタデータキャッシュの更新](#periodically-refresh-metadata-cache)"セクションを参照してください。この機能を有効にした後、StarRocksはデフォルトで10分ごとにHiveメタデータキャッシュを更新します。したがって、ほとんどの場合、手動更新は必要ありません。Hiveクラスタ上で新しいパーティションを作成した直後に新しいパーティションをすぐにクエリしたい場合にのみ手動更新が必要です。

なお、REFRESH EXTERNAL TABLEはFEにキャッシュされたテーブルおよびパーティションのメタデータのみを更新します。

### 自動的な増分更新

自動的な非同期更新ポリシーとは異なり、自動的な増分更新ポリシーでは、StarRocksクラスタ内のFEはHiveメタストアから列の追加、パーティションの削除、およびデータの更新などのイベントを読み取ります。これに基づいてFEにキャッシュされたメタデータを自動的に更新することができます。つまり、Hiveテーブルのメタデータを手動で更新する必要はありません。

この機能はHMSに大きな圧力を与える可能性がありますので、使用時には注意が必要です。[定期的なメタデータキャッシュの更新](#periodically-refresh-metadata-cache) を使用することをお勧めします。

自動的な増分更新を有効にするには、以下の手順に従います。
#### ステップ1：Hiveメタストアのイベントリスナーの構成

Hiveメタストアv2.xおよびv3.xの両方でイベントリスナーを構成することができます。このステップでは、Hiveメタストアv3.1.2で使用されるイベントリスナー構成を例として使用しています。以下の設定項目を **$HiveMetastore/conf/hive-site.xml** ファイルに追加し、その後Hiveメタストアを再起動してください:

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

イベントリスナーが正常に構成されたかどうかを確認するには、FEログファイルで`event id`を検索してください。構成に失敗すると、`event id`の値は`0`になります。

#### ステップ2：StarRocksの自動増分更新を有効にする

StarRocksクラスターにおいて、単一のHiveカタログまたはすべてのHiveカタログに対して自動増分更新を有効にすることができます。

- 単一のHiveカタログに対して自動増分更新を有効にするには、Hiveカタログを作成する際に `enable_hms_events_incremental_sync` パラメータを `true` に設定する必要があります。以下のように`PROPERTIES`に `"enable_hms_events_incremental_sync" = "true"` を追加してください：

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.uris" = "thrift://102.168.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```

- すべてのHiveカタログに対して自動増分更新を有効にするには、各FEの`$FE_HOME/conf/fe.conf`ファイルに `"enable_hms_events_incremental_sync" = "true"` を追加し、その後各FEを再起動してパラメータ設定を有効にしてください。

また、ビジネス要件に基づいて、各FEの`$FE_HOME/conf/fe.conf`ファイルで以下のパラメータを調整し、その後各FEを再起動してパラメータ設定を有効にすることもできます。

| パラメータ                         | 説明                                                  |
| --------------------------------- | ------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocksがHiveメタストアからイベントを読み取る間隔。デフォルト値: `5000`。単位: ミリ秒。 |
| hms_events_batch_size_per_rpc     | StarRocksが一度に読み取ることができるイベントの最大数。デフォルト値: `500`。 |
| enable_hms_parallel_process_evens | StarRocksがイベントを読み取りながら並列処理を行うかどうかを指定します。有効な値: `true`および`false`。デフォルト値: `true`。`true`は並列処理を有効にし、`false`は並列処理を無効にします。 |
| hms_process_events_parallel_num   | StarRocksが並列処理で処理できるイベントの最大数。デフォルト値: `4`。 |

## 定期的なメタデータキャッシュの更新

v2.5.5以降、StarRocksは頻繁にアクセスされるHiveカタログのキャッシュされたメタデータを定期的に更新してデータ変更を感知することができます。Hiveメタデータキャッシュの更新は、次の[FEパラメータ](../../administration/Configuration.md#fe-configuration-items)を介して設定することができます：

| 設定項目                                           | デフォルト値                           | 説明                          |
| ------------------------------------------------------------ | ------------------------------------ | ------------------------------- |
| enable_background_refresh_connector_metadata                 | v3.0では`true`<br />v2.5では`false`  | 定期的なHiveメタデータキャッシュの更新を有効にするかどうか。有効にすると、StarRocksはHiveクラスターのメタストア（HiveメタストアまたはAWS Glue）をポーリングし、頻繁にアクセスされるHiveカタログのキャッシュされたメタデータを更新してデータの変更を感知します。`true`はHiveメタデータキャッシュの更新を有効にし、`false`は無効にします。この項目は[FE動的パラメータ](../../administration/Configuration.md#configure-fe-dynamic-parameters)です。[ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) コマンドを使用して変更することができます。 |
| background_refresh_metadata_interval_millis                  | `600000` (10 分)                | 2回の連続するHiveメタデータキャッシュの更新間隔。単位: ミリ秒。この項目は[FE動的パラメータ](../../administration/Configuration.md#configure-fe-dynamic-parameters)です。[ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) コマンドを使用して変更することができます。 |
| background_refresh_metadata_time_secs_since_last_access_secs | `86400` (24 時間)                   | Hiveメタデータキャッシュの更新タスクの有効期限時間。アクセス済みのHiveカタログに対して、指定された時間より長くアクセスされていない場合、StarRocksはそのキャッシュされたメタデータの更新を停止します。アクセスされていないHiveカタログに対しては、キャッシュされたメタデータを更新しません。単位: 秒。この項目は[FE動的パラメータ](../../administration/Configuration.md#configure-fe-dynamic-parameters)です。[ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) コマンドを使用して変更することができます。 |

定期的なHiveメタデータキャッシュの更新機能とメタデータの自動非同期更新ポリシーを併用することで、データアクセスを大幅に加速し、外部データソースからの読み込み負荷を軽減し、クエリパフォーマンスを向上させることができます。

## 付録：メタデータの自動非同期更新

自動非同期更新は、StarRocksがHiveカタログのメタデータを更新するためのデフォルトポリシーです。

デフォルトでは（つまり、`enable_metastore_cache`および`enable_remote_file_cache`パラメータがともに`true`に設定されている場合）、クエリがHiveテーブルのパーティションにヒットすると、StarRocksはそのパーティションのメタデータとそのパーティションの基になるデータファイルのメタデータを自動的にキャッシュします。キャッシュされたメタデータは遅延更新ポリシーを使用して更新されます。

例えば、`table2`という名前のHiveテーブルがあり、そのテーブルには`p1`、`p2`、`p3`、`p4`の4つのパーティションがあります。クエリが`p1`にヒットすると、StarRocksは`p1`のメタデータと`p1`の基になるデータファイルのメタデータをキャッシュします。キャッシュされたメタデータを更新および破棄するデフォルトの時間間隔は次のとおりです：

- `metastore_cache_refresh_interval_sec`パラメータで指定された時間間隔で`p1`のキャッシュされたメタデータを非同期更新する時間間隔は2時間です。
- `remote_file_cache_refresh_interval_sec`パラメータで指定された時間間隔で`p1`の基になるデータファイルのキャッシュされたメタデータを非同期更新する時間間隔は60秒です。
- `metastore_cache_ttl_sec`パラメータで指定された時間間隔で`p1`のキャッシュされたメタデータを自動的に破棄する時間間隔は24時間です。
- `remote_file_cache_ttl_sec`パラメータで指定された時間間隔で`p1`の基になるデータファイルのキャッシュされたメタデータを自動的に破棄する時間間隔は36時間です。

以下の図は、理解を助けるためにタイムライン上での時間間隔を示しています。

![キャッシュされたメタデータの更新と破棄のためのタイムライン](../../assets/catalog_timeline.png)

その後、StarRocksは以下のルールに従ってメタデータを更新または破棄します：

- もう一度`p1`にヒットするクエリがあり、直近の更新からの経過時間が60秒未満である場合、StarRocksは`p1`のキャッシュされたメタデータまたは`p1`の基になるデータファイルのキャッシュされたメタデータを更新しません。
- もう一度`p1`にヒットするクエリがあり、直近の更新からの経過時間が60秒を超える場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを更新します。
- もう一度`p1`にヒットするクエリがあり、直近の更新からの経過時間が2時間を超える場合、StarRocksは`p1`のキャッシュされたメタデータを更新します。
- 直近の更新から24時間以内に`p1`がアクセスされていない場合、StarRocksは`p1`のキャッシュされたメタデータを破棄します。次回のクエリでメタデータがキャッシュされます。
- 直近の更新から36時間以内に`p1`がアクセスされていない場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを破棄します。次回のクエリでメタデータがキャッシュされます。