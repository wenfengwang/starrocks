```yaml
---
displayed_sidebar: "Japanese"
---
# Hudiカタログ

Hudiカタログは、Apache Hudiからデータを問い合わせるための外部カタログであり、データの取り込みなしで使用できます。

また、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して、StarRocksはHudiカタログに基づいてHudiから直接変換およびデータのロードをサポートします。 StarRocksは、v2.4以降でHudiカタログをサポートしています。

HudiクラスターでのSQLワークロードの実行を確実にするために、StarRocksクラスターは次の2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCSなどのオブジェクトストレージ、または他のS3互換ストレージシステム（たとえば、MinIO）のようなオブジェクトストレージシステム

- HiveメタストアまたはAWS Glueのようなメタストア

  > **注意**
  >
  > ストレージとしてAWS S3を選択する場合、HMSまたはAWS Glueをメタストアとして使用できます。他のストレージシステムを選択する場合は、メタストアとしてHMSのみを使用できます。

## 使用上の注意

- StarRocksがサポートするHudiのファイルフォーマットはParquetです。 Parquetファイルは、SNAPPY、LZ4、ZSTD、GZIP、およびNO_COMPRESSIONの次の圧縮形式をサポートしています。
- StarRocksは、HudiからのCopy On Write（COW）テーブルおよびMerge On Read（MOR）テーブルを完全にサポートしています。

## 統合の準備

Hudiカタログを作成する前に、StarRocksクラスターがHudiクラスターのストレージシステムおよびメタストアと統合できることを確認してください。

### AWS IAM

HudiクラスターがストレージとしてAWS S3またはメタストアとしてAWS Glueを使用する場合は、適切な認証メソッドを選択し、StarRocksクラスターが関連するAWSクラウドリソースにアクセスできるようにするために必要な準備を行ってください。

以下の認証メソッドを推奨します。

- インスタンスプロファイル
- 仮定されたロール
- IAMユーザー

上記の3つの認証メソッドのうち、インスタンスプロファイルが最も広く使用されています。

詳細については、[AWS IAMでの認証の準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとしてHDFSを選択する場合は、StarRocksクラスターを次のように構成します。

- （オプション）HDFSクラスターおよびHiveメタストアにアクセスするために使用されるユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスターおよびHiveメタストアにアクセスします。各FEの**fe/conf/hadoop_env.sh**ファイルおよび各BEの**be/conf/hadoop_env.sh**ファイルの先頭に`export HADOOP_USER_NAME="<user_name>"`を追加することで、ユーザー名を設定できます。これらのファイルにユーザー名を設定した後は、各FEと各BEを再起動してパラメータ設定を有効にします。StarRocksクラスターには1つのユーザー名のみを設定できます。
- Hudiデータを問い合わせる場合、StarRocksクラスターのFEおよびBEはHDFSクライアントを使用してHDFSクラスターにアクセスします。ほとんどの場合、この目的を達成するためにStarRocksクラスターを構成する必要はありません。StarRocksはデフォルトの設定でHDFSクライアントを起動します。StarRocksクラスターを構成する必要があるのは、以下の状況のみです。

  - HDFSクラスターの高可用性（HA）が有効になっている場合：HDFSクラスターの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスおよび各BEの**$BE_HOME/conf**パスに追加します。
  - HDFSクラスターのView File System（ViewFs）が有効になっている場合：HDFSクラスターの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスおよび各BEの**$BE_HOME/conf**パスに追加します。

> **注意**
>
> クエリを送信する際に不明なホストを示すエラーが返される場合は、「/etc/hosts」パスにHDFSクラスターノードのホスト名とIPアドレスのマッピングを追加する必要があります。

### Kerberos認証

HDFSクラスターまたはHiveメタストアでKerberos認証が有効になっている場合は、StarRocksクラスターを次のように構成します。

- 各FEおよび各BEで`kinit -kt keytab_path principal`コマンドを実行して、Key Distribution Center（KDC）からチケット発行チケット（TGT）を取得します。このコマンドを実行するには、HDFSクラスターやHiveメタストアにアクセスするアクセス許可が必要です。このコマンドでKDCにアクセスするため、定期的にこのコマンドを実行するためにcronを使用する必要があります。
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

Hudiカタログの名前。命名規則は次のとおりです。

- 名前には、文字、数字（0-9）、アンダースコア（_）を含めることができます。文字で始まる必要があります。
- 名前は大文字と小文字を区別し、1023文字を超えることはできません。

#### comment

Hudiカタログの説明。このパラメータはオプションです。

#### type

データソースのタイプ。値を`hudi`に設定します。

#### MetastoreParams

データソースのメタストアとの統合方法に関する一連のパラメータ。

##### Hiveメタストア

データソースのメタストアとしてHiveメタストアを選択した場合は、`MetastoreParams`を次のように構成します。

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注**
>
> Hudiデータをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを「/etc/hosts」パスに追加する必要があります。そうしないと、クエリを開始するとStarRocksがHiveメタストアにアクセスできない場合があります。

以下の表は、`MetastoreParams`で構成する必要のあるパラメータについて説明します。

| パラメータ              | 必須     | 説明                   |
| ------------------- | -------- | ---------------------- |
| hive.metastore.type | はい      | Hudiクラスターで使用するメタストアのタイプ。値を`hive`に設定します。 |
| hive.metastore.uris | はい      | HiveメタストアのURI。形式：`thrift://<metastore_IP_address>:<metastore_port>`。<br />Hiveメタストアの高可用性（HA）が有効になっている場合は、複数のメタストアURIを指定し、カンマ（`,`）で区切って指定します。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

データソースのメタストアとしてAWS Glueを選択した場合（これはAWS S3をストレージとして選択した場合のみサポートされます）、次のいずれかのアクションを実行します。

- インスタンスプロファイルベースの認証メソッドを選択する場合は、`MetastoreParams`を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 仮定されたロールベースの認証メソッドを選択する場合は、`MetastoreParams`を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAMユーザーベースの認証メソッドを選択する場合は、`MetastoreParams`を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

以下の表は、`MetastoreParams`で構成する必要のあるパラメータについて説明します。

| パラメータ                     | 必須     | 説明                   |
| ----------------------------- | -------- | ---------------------- |
| hive.metastore.type           | はい      | Hudiクラスターで使用するメタストアのタイプ。値を`glue`に設定します。 |
| aws.glue.use_instance_profile | はい      | インスタンスプロファイルベースの認証メソッドおよび仮定されたロールベースの認証メソッドを有効にするかどうかを指定します。有効な値は`true`および`false`です。デフォルト値は`false`です。 |
| aws.glue.iam_role_arn         | No       | AWS Glueデータカタログに権限を持つIAMロールのARN。AWS Glueへのアクセスにアサームドロールベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.region               | Yes      | AWS Glueデータカタログが存在するリージョン。例: `us-west-1`。 |
| aws.glue.access_key           | No       | AWS IAMユーザーのアクセスキー。AWS GlueへのアクセスにIAMユーザーベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | No       | AWS IAMユーザーのシークレットキー。AWS GlueへのアクセスにIAMユーザーベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |

AWS Glueへのアクセスの認証メソッドの選択およびAWS IAMコンソールでアクセス制御ポリシーを設定する方法についての情報については、[AWS Glueへのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocksがストレージシステムと統合する方法に関する一連のパラメータ。このパラメータセットはオプションです。

ストレージとしてHDFSを使用する場合、`StorageCredentialParams`を構成する必要はありません。

AWS S3、その他のS3互換のストレージシステム、Microsoft Azure Storage、またはGoogle GCSを使用する場合は、`StorageCredentialParams`を構成する必要があります。

##### AWS S3

HudiクラスタのストレージとしてAWS S3を選択する場合、次のいずれかのアクションを実行してください。

- インスタンスプロファイルベースの認証メソッドを選択する場合は、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- アサームドロールベースの認証メソッドを選択する場合は、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーベースの認証メソッドを選択する場合は、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

| パラメータ                  | 必須     | 説明                                                       |
| --------------------------- | -------- | ---------------------------------------------------------- |
| aws.s3.use_instance_profile | Yes      | インスタンスプロファイルベースの認証メソッドとアサームドロールベースの認証メソッドを有効にするかどうかを指定します。有効な値: `true`および `false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | No       | AWS S3バケットに権限を持つIAMロールのARN。AWS S3へのアクセスにアサームドロールベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.region               | Yes      | AWS S3バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | No       | IAMユーザーのアクセスキー。AWS S3へのアクセスにIAMユーザーベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | No       | IAMユーザーのシークレットキー。AWS S3へのアクセスにIAMユーザーベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |

AWS S3へのアクセスの認証メソッドの選択およびAWS IAMコンソールでアクセス制御ポリシーを設定する方法についての情報については、[AWS S3へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換のストレージシステム

Hudiカタログはv2.5以降、S3互換のストレージシステムをサポートしています。

MinIOなどのS3互換のストレージシステムをストレージとして選択する場合は、次のように`StorageCredentialParams`を構成して、統合を正常に行う必要があります：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

| パラメータ                     | 必須     | 説明                                                       |
| ----------------------------- | -------- | ---------------------------------------------------------- |
| aws.s3.enable_ssl             | Yes      | SSL接続を有効にするかどうかを指定します。<br />有効な値: `true`および `false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access | Yes      | パス形式のアクセスを有効にするかどうかを指定します。<br />有効な値: `true`および `false`。デフォルト値: `false`。MinIOの場合、値を`true`に設定する必要があります。<br />パス形式のURLは、次の形式を使用します：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、米国西部（オレゴン）リージョンに`DOC-EXAMPLE-BUCKET1`という名前のバケットを作成し、そのバケット内の`alice.jpg`オブジェクトにアクセスしたい場合は、次のパス形式のURLを使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint               | Yes      | AWS S3の代わりにS3互換のストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key             | Yes      | IAMユーザーのアクセスキー。 |
| aws.s3.secret_key             | Yes      | IAMユーザーのシークレットキー。 |

##### Microsoft Azure Storage

Hudiカタログはv3.0以降、Microsoft Azure Storageをサポートしています。

###### Azure Blob Storage

HudiクラスタのストレージとしてBlob Storageを選択する場合は、次のいずれかのアクションを実行してください：

- 共有キー認証メソッドを選択する場合は、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

| パラメータ              | 必須     | 説明                                     |
| ---------------------- | -------- | ---------------------------------------- |
| azure.blob.storage_account | Yes      | Blob Storageアカウントのユーザー名。    |
| azure.blob.shared_key | Yes      | Blob Storageアカウントの共有キー。      |

- SASトークン認証メソッドを選択する場合は、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

| パラメータ             | 必須     | 説明                                              |
| ---------------------- | -------- | ------------------------------------------------- |
| azure.blob.account_name | Yes      | Blob Storageアカウントのユーザー名。             |
| azure.blob.container_name | Yes      | データを格納するBlobコンテナの名前。         |
| azure.blob.sas_token   | Yes      | Blob Storageアカウントにアクセスするために使用されるSASトークン。 |

###### Azure Data Lake Storage Gen1

HudiクラスタのストレージとしてData Lake Storage Gen1を選択する場合は、次のいずれかのアクションを実行してください：

- マネージドサービスアイデンティティ認証メソッドを選択する場合は、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

| パラメータ                            | 必須     | 説明                                              |
| ------------------------------------ | -------- | ------------------------------------------------- |
| azure.adls1.use_managed_service_identity | Yes      | マネージドサービスアイデンティティ認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |

- サービスプリンシパル認証メソッドを選択する場合は、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

以下の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

| パラメータ                | 必須     | 説明                                                     |
| ------------------------ | -------- | -------------------------------------------------------- |
| azure.adls1.oauth2_client_id | Yes      | サービスプリンシパルのクライアント（アプリケーション）ID。 |
| azure.adls1.oauth2_credential | Yes      | 新しく作成されたクライアント（アプリケーション）シークレットの値。 |
| azure.adls1.oauth2_endpoint | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2
If you choose Data Lake Storage Gen2 as storage for your Hudi cluster, take one of the following actions:

- To choose the Managed Identity authentication method, configure `StorageCredentialParams` as follows:

```SQL
"azure.adls2.oauth2_use_managed_identity" = "true",
"azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
"azure.adls2.oauth2_client_id" = "<service_client_id>"
```

The following table describes the parameters you need to configure in `StorageCredentialParams`.

| **Parameter**                           | **Required** | **Description**                                              |
| --------------------------------------- | ------------ | ------------------------------------------------------------ |
| azure.adls2.oauth2_use_managed_identity | Yes          | マネージドID認証方法を有効にするかどうかを指定します。値を`true`に設定してください。 |
| azure.adls2.oauth2_tenant_id            | Yes          | アクセスするデータが属するテナントのIDです。                  |
| azure.adls2.oauth2_client_id            | Yes          | マネージドIDのクライアント(アプリケーション)IDです。          |

- To choose the Shared Key authentication method, configure `StorageCredentialParams` as follows:

```SQL
"azure.adls2.storage_account" = "<storage_account_name>",
"azure.adls2.shared_key" = "<shared_key>"
```

The following table describes the parameters you need to configure in `StorageCredentialParams`.

| **Parameter**               | **Required** | **Description**                                              |
| --------------------------- | ------------ | ------------------------------------------------------------ |
| azure.adls2.storage_account | Yes          | Data Lake Storage Gen2ストレージアカウントのユーザー名です。 |
| azure.adls2.shared_key      | Yes          | Data Lake Storage Gen2ストレージアカウントの共有キーです。  |

- To choose the Service Principal authentication method, configure `StorageCredentialParams` as follows:

```SQL
"azure.adls2.oauth2_client_id" = "<service_client_id>",
"azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
"azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
```

The following table describes the parameters you need to configure in `StorageCredentialParams`.

| **Parameter**                      | **Required** | **Description**                                              |
| ---------------------------------- | ------------ | ------------------------------------------------------------ |
| azure.adls2.oauth2_client_id       | Yes          | サービスプリンシパルのクライアント(アプリケーション)IDです。   |
| azure.adls2.oauth2_client_secret   | Yes          | 新しいクライアント(アプリケーション)シークレットの値です。     |
| azure.adls2.oauth2_client_endpoint | Yes          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント(v1)です。 | 

##### Google GCS

Hudiカタログは、v3.0以降でGoogle GCSをサポートしています。

If you choose Google GCS as storage for your Hudi cluster, take one of the following actions:

- To choose the VM-based authentication method, configure `StorageCredentialParams` as follows:

```SQL
"gcp.gcs.use_compute_engine_service_account" = "true"
```

The following table describes the parameters you need to configure in `StorageCredentialParams`.

| **Parameter**                              | **Default value** | **Value** **example** | **Description**                                              |
| ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接 Compute Engine にバインドされたサービスアカウントを使用するかどうかを指定します。 |

- To choose the service account-based authentication method, configure `StorageCredentialParams` as follows:

```SQL
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

The following table describes the parameters you need to configure in `StorageCredentialParams`.

| **Parameter**                          | **Default value** | **Value** **example**                                        | **Description**                                              |
| -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントの作成時に生成されるJSONファイル内の電子メールアドレスです。       |
| gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウント作成時に生成されるJSONファイル内の秘密キーIDです。                    |
| gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウント作成時に生成されるJSONファイル内の秘密キーです。                        |

- To choose the impersonation-based authentication method, configure `StorageCredentialParams` as follows:

  - VMインスタンスをサービスアカウントをなりすますようにします:

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    The following table describes the parameters you need to configure in `StorageCredentialParams`.

    | **Parameter**                              | **Default value** | **Value** **example** | **Description**                                              |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接 Compute Engine にバインドされたサービスアカウントを使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | なりすますサービスアカウントです。                           |

  - サービスアカウント(一時的な名前はメタサービスアカウントとします)を他のサービスアカウント(一時的な名前はデータサービスアカウントとします)になりすますようにします:

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    The following table describes the parameters you need to configure in `StorageCredentialParams`.

    | **Parameter**                          | **Default value** | **Value** **example**                                        | **Description**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウントの作成時に生成されるJSONファイル内の電子メールアドレスです。         |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタサービスアカウント作成時に生成されるJSONファイル内の秘密キーIDです。                   |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウント作成時に生成されるJSONファイル内の秘密キーです。                       |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | なりすますデータサービスアカウントです。                |

#### MetadataUpdateParams

Hudiのキャッシュメタデータを更新するStarRocksのパラメータセットです。このパラメータセットはオプションです。

StarRocksはデフォルトで[自動非同期更新ポリシー](#appendix-understand-metadata-automatic-asynchronous-update)を実装しています。

ほとんどの場合、`MetadataUpdateParams`を無視し、それに含まれるポリシーパラメータをチューニングする必要はありません。なぜなら、これらのパラメータのデフォルト値がそのままOut-of-the-boxのパフォーマンスを提供しているからです。

> **注意**
>
> ほとんどの場合、Hudiデータが1時間以下の粒度で更新される場合、データの更新頻度は高いと見なされます。

| Parameter                              | Required | Description                                                  |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | No       | StarRocksがHudiテーブルのメタデータをキャッシュするかどうかを指定します。有効な値: `true`および`false`。デフォルト値: `true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| enable_remote_file_cache               | No       | StarRocksがHudiテーブルやパーティションの基礎データファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true`および`false`。デフォルト値: `true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | No       | StarRocksが自身でキャッシュしているHudiテーブルやパーティションのメタデータを非同期で更新する間隔です。単位: 秒。デフォルト値: `7200`、つまり2時間です。 |
| remote_file_cache_refresh_interval_sec | いいえ | StarRocksが、Hudiテーブルまたはそれにキャッシュされたパーティションの基礎データファイルのメタデータを非同期で更新する間隔。単位: 秒。デフォルト値:`60`。 |
| metastore_cache_ttl_sec | いいえ | StarRocksが、Hudiテーブルまたはそれにキャッシュされたパーティションのメタデータを自動的に破棄する間隔。単位: 秒。デフォルト値:`86400`（24時間）。 |
| remote_file_cache_ttl_sec | いいえ | StarRocksが、Hudiテーブルまたはそれにキャッシュされたパーティションの基礎データファイルのメタデータを自動的に破棄する間隔。単位: 秒。デフォルト値:`129600`（36時間）。 |

### 例

以下の例では、Hudiクラスタからデータをクエリするために、`hudi_catalog_hms`または`hudi_catalog_glue`というHudiカタログを作成します。これは、使用するメタストアのタイプに応じて異なります。

#### HDFS

ストレージとしてHDFSを使用する場合、以下のようなコマンドを実行してください:

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

- HudiクラスタでHiveメタストアを使用する場合、以下のようなコマンドを実行してください:

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

- Amazon EMR HudiクラスタでAWS Glueを使用する場合、以下のようなコマンドを実行してください:

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

##### 仮定されたロールベースの資格情報を選択した場合

- HudiクラスタでHiveメタストアを使用する場合、以下のようなコマンドを実行してください:

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

- Amazon EMR HudiクラスタでAWS Glueを使用する場合、以下のようなコマンドを実行してください:

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

- HudiクラスタでHiveメタストアを使用する場合、以下のようなコマンドを実行してください:

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

- Amazon EMR HudiクラスタでAWS Glueを使用する場合、以下のようなコマンドを実行してください:

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

MinIOを例に使用します。以下のようなコマンドを実行してください:

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

- 共有キー認証メソッドを選択した場合、以下のようなコマンドを実行してください:

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

- SASトークン認証メソッドを選択した場合、以下のようなコマンドを実行してください:

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

- マネージドサービスアイデンティティ認証メソッドを選択した場合、以下のようなコマンドを実行してください:

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

- サービスプリンシパル認証メソッドを選択した場合、以下のようなコマンドを実行してください:

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

- マネージドアイデンティティ認証メソッドを選択した場合、以下のようなコマンドを実行してください:

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
```
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- もしShared Key認証方法を選択した場合は以下のようなコマンドを実行してください:

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

- もしService Principal認証方法を選択した場合は以下のようなコマンドを実行してください:

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

- もしVMベースの認証方法を選択した場合は以下のようなコマンドを実行してください:

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

- もしサービスアカウントベースの認証方法を選択した場合は以下のようなコマンドを実行してください:

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

- もし偽装ベースの認証方法を選択した場合:

  - もしVMインスタンスがサービスアカウントを偽装する場合は以下のようなコマンドを実行してください:

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

  - もしサービスアカウントが別のサービスアカウントを偽装する場合は以下のようなコマンドを実行してください:

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

## Hudiカタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスタ内のすべてのカタログをクエリすることができます:

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリすることもできます。以下の例は`hudi_catalog_glue`というHudiカタログの作成ステートメントをクエリしています:

```SQL
SHOW CREATE CATALOG hudi_catalog_glue;
```

## Hudiカタログとその中のデータベースを切り替える

Hudiカタログとその中のデータベースを切り替えるために、以下の方法のいずれかを使用することができます:

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションでHudiカタログを指定し、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します:

  ```SQL
  -- 現在のセッションで指定したカタログに切り替える:
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定します:
  USE <db_name>
  ```

- 直接[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、Hudiカタログとその中のデータベースに切り替えます:

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Hudiカタログを削除する

外部カタログを削除するために、[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用することができます。

以下の例は`hudi_catalog_glue`というHudiカタログを削除しています:

```SQL
DROP Catalog hudi_catalog_glue;
```

## Hudiテーブルのスキーマを表示する

Hudiテーブルのスキーマを表示するために、以下の構文のいずれかを使用することができます:

- スキーマを表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Hudiテーブルをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Hudiクラスタ内のデータベースを表示します:

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Hudiカタログとその中のデータベースを切り替える](#hudiカタログとその中のデータベースを切り替える)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、指定したデータベース内の宛先テーブルをクエリします:

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hudiからデータを読み込む

OLAPテーブル`olap_tbl`があると仮定すると、以下のようにデータを変換して読み込むことができます:

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hudi_table
```

## メタデータキャッシュを手動または自動的に更新する

### 手動更新

デフォルトでは、StarRocksはHudiのメタデータをキャッシュし、パフォーマンスを向上させるために非同期モードでメタデータを自動的に更新します。また、Hudiテーブルでスキーマの変更やテーブルの更新が行われた後、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)を使用して、メタデータを手動で更新することもできます。これにより、StarRocksができるだけ早く最新のメタデータを取得し、適切な実行プランを生成できます:

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自動的な増分更新

自動的な非同期更新ポリシーとは異なり、自動的な増分更新ポリシーでは、StarRocksクラスタ内のFEがHiveメタストアから列の追加、パーティションの削除、データの更新などのイベントを読み取ることができます。StarRocksはこれらのイベントに基づいてFEにキャッシュされたメタデータを自動的に更新することができます。つまり、Hudiテーブルのメタデータを手動で更新する必要はありません。

自動的な増分更新を有効にするには、以下の手順に従ってください:

#### ステップ1: Hiveメタストアにイベントリスナーを構成する

Hiveメタストアv2.xおよびv3.xの両方がイベントリスナーを構成することをサポートしています。この手順では、Hiveメタストアv3.1.2用のイベントリスナー構成を例としています。以下の構成アイテムを**$HiveMetastore/conf/hive-site.xml**ファイルに追加し、Hiveメタストアを再起動してください:

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
{
  "hive.metastore.server.max.message.size": "858993459"
}
```

`event id`をFEログファイルで検索してイベントリスナーが正常に構成されているかどうかを確認できます。構成が失敗した場合、`event id`の値は`0`です。

#### ステップ2: StarRocksでの自動増分更新を有効にする

StarRocksでは、単一のHudiカタログまたはStarRocksクラスター内のすべてのHudiカタログに対して自動増分更新を有効にすることができます。

- 単一のHudiカタログに対して自動増分更新を有効にするには、次のように`PROPERTIES`内で`enable_hms_events_incremental_sync`パラメーターを`true`に設定してHudiカタログを作成します:

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://102.168.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```

- すべてのHudiカタログに対して自動増分更新を有効にするには、各FEの`$FE_HOME/conf/fe.conf`ファイルに`"enable_hms_events_incremental_sync" = "true"`を追加し、その後各FEを再起動してパラメータ設定を有効にします。

また、ビジネス要件に基づいて各FEの`$FE_HOME/conf/fe.conf`ファイルで以下のパラメータを調整し、その後各FEを再起動してパラメータ設定を有効にすることができます。

| パラメータ                     | 説明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms | StarRocksがHiveメタストアからイベントを読み込む時間間隔。デフォルト値: `5000`。単位: ミリ秒。 |
| hms_events_batch_size_per_rpc | StarRocksが一度に読み取ることができる最大イベント数。デフォルト値: `500`。 |
| enable_hms_parallel_process_evens | StarRocksがイベントを読み取りながら並列で処理するかどうかを指定します。有効な値: `true`および`false`。デフォルト値: `true`。`true`は並列処理を有効にし、`false`は並列処理を無効にします。 |
| hms_process_events_parallel_num | StarRocksが並列で処理できる最大イベント数。デフォルト値: `4`。 |

## 付録: メタデータ自動非同期更新の理解

自動非同期更新は、StarRocksがHudiカタログ内のメタデータを更新するために使用するデフォルトのポリシーです。

デフォルトでは（つまり`enable_metastore_cache`および`enable_remote_file_cache`パラメータがいずれも`true`に設定されている場合）、クエリがHudiテーブルのパーティションにアクセスすると、StarRocksは自動的にそのパーティションのメタデータおよびパーティションの基礎データファイルのメタデータをキャッシュします。キャッシュされたメタデータは、遅延更新ポリシーを使用して更新されます。

例えば、`table2`という名前のHudiテーブルがあるとします。このテーブルには`p1`、`p2`、`p3`、`p4`という4つのパーティションがあります。クエリが`p1`にアクセスすると、StarRocksは`p1`のメタデータと`p1`の基礎データファイルのメタデータを自動的にキャッシュします。キャッシュされたメタデータが更新および破棄されるデフォルトの時間間隔は次のとおりです:

- `metastore_cache_refresh_interval_sec`パラメータで指定された時間間隔で`p1`のキャッシュされたメタデータを非同期に更新する時間間隔は2時間です。
- `remote_file_cache_refresh_interval_sec`パラメータで指定された時間間隔で`p1`の基礎データファイルのキャッシュされたメタデータを非同期に更新する時間間隔は60秒です。
- `metastore_cache_ttl_sec`パラメータで指定された時間間隔で`p1`のキャッシュされたメタデータを自動的に破棄する時間間隔は24時間です。
- `remote_file_cache_ttl_sec`パラメータで指定された時間間隔で`p1`の基礎データファイルのキャッシュされたメタデータを自動的に破棄する時間間隔は36時間です。

以下の図は、理解を容易にするためにタイムライン上に更新および破棄されたキャッシュされたメタデータの時間間隔を示しています。

![更新と破棄したキャッシュされたメタデータのタイムライン](../../assets/catalog_timeline.png)

その後、StarRocksは次の規則に従ってメタデータを更新または破棄します:

- もう一度`p1`にクエリがヒットし、前回の更新からの現在の時間が60秒未満の場合、StarRocksは`p1`のキャッシュされたメタデータまたは`p1`の基礎データファイルのキャッシュされたメタデータを更新しません。
- もう一度`p1`にクエリがヒットし、前回の更新からの現在の時間が60秒を超える場合、StarRocksは`p1`の基礎データファイルのキャッシュされたメタデータを更新します。
- もう一度`p1`にクエリがヒットし、前回の更新からの現在の時間が2時間を超える場合、StarRocksは`p1`のキャッシュされたメタデータを更新します。
- `p1`が最後の更新から24時間以内にアクセスされていない場合、StarRocksは`p1`のキャッシュされたメタデータを破棄します。次のクエリでメタデータがキャッシュされます。
- `p1`が最後の更新から36時間以内にアクセスされていない場合、StarRocksは`p1`の基礎データファイルのキャッシュされたメタデータを破棄します。次のクエリでメタデータがキャッシュされます。