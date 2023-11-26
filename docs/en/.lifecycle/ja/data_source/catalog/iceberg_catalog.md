---
displayed_sidebar: "Japanese"
---

# アイスバーグカタログ

アイスバーグカタログは、StarRocks v2.4以降でサポートされる外部カタログの一種です。アイスバーグカタログを使用すると、次のことができます。

- 手動でテーブルを作成する必要なく、アイスバーグに格納されたデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)または非同期マテリアライズドビュー（v2.5以降でサポート）を使用して、アイスバーグに格納されたデータを処理し、データをStarRocksにロードできます。
- StarRocksで操作を実行して、アイスバーグデータベースとテーブルを作成または削除したり、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用してStarRocksテーブルからParquet形式のアイスバーグテーブルにデータをシンクすることができます（この機能はv3.1以降でサポート）。

アイスバーグクラスタでのSQLワークロードの成功を保証するには、StarRocksクラスタを次の2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCSなどのオブジェクトストレージ、またはその他のS3互換ストレージシステム（たとえば、MinIOなど）

- Hiveメタストア、AWS Glue、またはTabularなどのメタストア

  > **注記**
  >
  > - ストレージとしてAWS S3を選択した場合、メタストアとしてHMSまたはAWS Glueを使用できます。その他のストレージシステムを選択した場合、メタストアとしてHMSのみを使用できます。
  > - メタストアとしてTabularを選択した場合、Iceberg RESTカタログを使用する必要があります。

## 使用上の注意

- StarRocksがサポートするIcebergのファイル形式はParquetとORCです。

  - Parquetファイルは、次の圧縮形式をサポートしています：SNAPPY、LZ4、ZSTD、GZIP、NO_COMPRESSION。
  - ORCファイルは、次の圧縮形式をサポートしています：ZLIB、SNAPPY、LZO、LZ4、ZSTD、NO_COMPRESSION。

- アイスバーグカタログはv1テーブルをサポートし、StarRocks v3.0以降ではORC形式のv2テーブルもサポートしています。
- アイスバーグカタログはv1テーブルをサポートしています。さらに、StarRocks v3.0以降ではORC形式のv2テーブル、StarRocks v3.1以降ではParquet形式のv2テーブルもサポートしています。

## 統合の準備

アイスバーグカタログを作成する前に、StarRocksクラスタがアイスバーグクラスタのストレージシステムとメタストアと統合できることを確認してください。

### AWS IAM

アイスバーグクラスタがストレージとしてAWS S3またはメタストアとしてAWS Glueを使用する場合は、適切な認証方法を選択し、関連するAWSクラウドリソースにStarRocksクラスタがアクセスできるように必要な準備を行ってください。

以下の認証方法が推奨されます。

- インスタンスプロファイル
- 仮定された役割
- IAMユーザー

上記の3つの認証方法のうち、インスタンスプロファイルが最も一般的に使用されています。

詳細については、[AWS IAMでの認証の準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとしてHDFSを選択した場合、StarRocksクラスタを次のように構成してください。

- （オプション）HDFSクラスタとHiveメタストアにアクセスするために使用されるユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスタおよびHiveメタストアにアクセスします。各FEの**fe/conf/hadoop_env.sh**ファイルと各BEの**be/conf/hadoop_env.sh**ファイルの先頭に`export HADOOP_USER_NAME="<user_name>"`を追加してユーザー名を設定することもできます。これらのファイルでユーザー名を設定した後、各FEと各BEを再起動してパラメータ設定を有効にします。StarRocksクラスタごとに1つのユーザー名のみを設定できます。
- アイスバーグデータをクエリする場合、StarRocksクラスタのFEおよびBEはHDFSクラスタにアクセスするためにHDFSクライアントを使用します。ほとんどの場合、この目的を達成するためにStarRocksクラスタを構成する必要はありません。StarRocksはデフォルトの設定でHDFSクライアントを起動します。次の状況では、StarRocksクラスタを構成する必要があります。

  - HDFSクラスタで高可用性（HA）が有効になっている場合：HDFSクラスタの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。
  - HDFSクラスタでView File System（ViewFs）が有効になっている場合：HDFSクラスタの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。

> **注記**
>
> クエリを送信する際に不明なホストを示すエラーが返される場合は、HDFSクラスタノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに追加する必要があります。

### Kerberos認証

HDFSクラスタまたはHiveメタストアでKerberos認証が有効になっている場合は、StarRocksクラスタを次のように構成してください。

- 各FEおよび各BEで`kinit -kt keytab_path principal`コマンドを実行して、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFSクラスタとHiveメタストアにアクセスする権限が必要です。このコマンドを定期的に実行するためにcronを使用する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルと各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。この例では、`/etc/krb5.conf`は**krb5.conf**ファイルの保存パスです。必要に応じてパスを変更できます。

## アイスバーグカタログの作成

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

### パラメータ

#### catalog_name

アイスバーグカタログの名前です。命名規則は次のとおりです。

- 名前には、文字、数字（0-9）、アンダースコア（_）を含めることができます。ただし、文字で始める必要があります。
- 名前は大文字と小文字を区別し、長さは1023文字を超えることはできません。

#### comment

アイスバーグカタログの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。値を`iceberg`に設定します。

#### MetastoreParams

データソースのメタストアとの統合方法に関するパラメータのセットです。

##### Hiveメタストア

データソースのメタストアとしてHiveメタストアを選択した場合、`MetastoreParams`を次のように構成します。

```SQL
"iceberg.catalog.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> アイスバーグデータをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを`/etc/hosts`パスに追加する必要があります。そうしないと、クエリを開始するときにStarRocksがHiveメタストアにアクセスできなくなる場合があります。

次の表に、`MetastoreParams`で構成する必要のあるパラメータを説明します。

| パラメータ                      | 必須 | 説明                                                         |
| ------------------------------ | ---- | ------------------------------------------------------------ |
| iceberg.catalog.type           | Yes  | Icebergクラスタで使用するメタストアのタイプです。値を`hive`に設定します。 |
| hive.metastore.uris            | Yes  | HiveメタストアのURIです。形式：`thrift://<metastore_IP_address>:<metastore_port>`。<br />Hiveメタストアで高可用性（HA）が有効になっている場合、複数のメタストアURIを指定し、カンマ（`,`）で区切って指定できます。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`と指定します。 |

##### AWS Glue

データソースのメタストアとしてAWS Glueを選択した場合（ストレージとしてAWS S3を選択した場合のみサポート）、次のいずれかの操作を実行します。

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams`を次のように構成します。

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 仮定された役割ベースの認証方法を選択する場合、`MetastoreParams`を次のように構成します。

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAMユーザーベースの認証方法を選択する場合、`MetastoreParams`を次のように構成します。

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

次の表に、`MetastoreParams`で構成する必要のあるパラメータを説明します。

| パラメータ                     | 必須 | 説明                                                         |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| iceberg.catalog.type          | Yes  | Icebergクラスタで使用するメタストアのタイプです。値を`glue`に設定します。 |
| aws.glue.use_instance_profile | Yes  | インスタンスプロファイルベースの認証方法および仮定された役割ベースの認証方法を有効にするかどうかを指定します。有効な値は`true`および`false`です。デフォルト値は`false`です。 |
| aws.glue.iam_role_arn         | No   | AWS Glueデータカタログに特権を持つIAMロールのARNです。AWS Glueへのアクセスに仮定された役割ベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.region               | Yes  | AWS Glueデータカタログが存在するリージョンです。例：`us-west-1`。 |
| aws.glue.access_key           | No   | AWS IAMユーザーのアクセスキーです。AWS GlueへのアクセスにIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | No   | AWS IAMユーザーのシークレットキーです。AWS GlueへのアクセスにIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |

AWS Glueへのアクセス方法の選択およびAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS Glueへのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

##### Tabular

メタストアとしてTabularを使用する場合、メタストアのタイプをREST（`"iceberg.catalog.type" = "rest"`）と指定する必要があります。`MetastoreParams`を次のように構成します。

```SQL
"iceberg.catalog.type" = "rest",
"iceberg.catalog.uri" = "<rest_server_api_endpoint>",
"iceberg.catalog.credential" = "<credential>",
"iceberg.catalog.warehouse" = "<identifier_or_path_to_warehouse>"
```

次の表に、`MetastoreParams`で構成する必要のあるパラメータを説明します。

| パラメータ                  | 必須 | 説明                                                         |
| -------------------------- | ---- | ------------------------------------------------------------ |
| iceberg.catalog.type       | Yes  | Icebergクラスタで使用するメタストアのタイプです。値を`rest`に設定します。 |
| iceberg.catalog.uri        | Yes  | TabularサービスエンドポイントのURIです。例：`https://api.tabular.io/ws`。 |
| iceberg.catalog.credential | Yes  | Tabularサービスの認証情報です。                              |
| iceberg.catalog.warehouse  | No   | Icebergカタログのワークスペースの場所または識別子です。例：`s3://my_bucket/warehouse_location`または`sandbox`。 |

次の例では、Tabularをメタストアとして使用するIcebergカタログ`tabular`を作成します。

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

#### `StorageCredentialParams`

ストレージシステムとの統合方法に関するパラメータのセットです。このパラメータセットはオプションです。

次のポイントに注意してください。

- ストレージとしてHDFSを使用する場合、`StorageCredentialParams`を構成する必要はありません。AWS S3、その他のS3互換ストレージシステム、Microsoft Azure Storage、またはGoogle GCSを使用する場合は、`StorageCredentialParams`を構成する必要があります。

- メタストアとしてTabularを使用する場合、`StorageCredentialParams`を構成する必要はありません。HMSまたはAWS Glueを使用する場合は、`StorageCredentialParams`を構成する必要があります。

##### AWS S3

アイスバーグクラスタのストレージとしてAWS S3を選択した場合、次のいずれかの操作を実行します。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 仮定された役割ベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

| パラメータ                   | 必須 | 説明                                                         |
| --------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes  | インスタンスプロファイルベースの認証方法および仮定された役割ベースの認証方法を有効にするかどうかを指定します。有効な値は`true`および`false`です。デフォルト値は`false`です。 |
| aws.s3.iam_role_arn         | No   | AWS S3バケットに特権を持つIAMロールのARNです。AWS S3へのアクセスに仮定された役割ベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.s3.region               | Yes  | AWS S3バケットが存在するリージョンです。例：`us-west-1`。    |
| aws.s3.access_key           | No   | IAMユーザーのアクセスキーです。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | No   | IAMユーザーのシークレットキーです。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |

AWS S3へのアクセス方法の選択およびAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS S3へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

アイスバーグカタログはv2.5以降でS3互換ストレージシステム（MinIOなど）をサポートしています。

S3互換ストレージシステム（たとえば、MinIO）をストレージとして選択した場合、統合の成功を保証するために`StorageCredentialParams`を次のように構成します。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

| パラメータ                        | 必須 | 説明                                                         |
| -------------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | Yes  | SSL接続を有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`true`。 |
| aws.s3.enable_path_style_access  | Yes  | パススタイルアクセスを有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`false`。MinIOの場合、値を`true`に設定する必要があります。<br />パススタイルURLは次の形式を使用します：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、US West（Oregon）リージョンで`DOC-EXAMPLE-BUCKET1`というバケットを作成し、そのバケットの`alice.jpg`オブジェクトにアクセスする場合、次のパススタイルURLを使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | Yes  | AWS S3の代わりにS3互換ストレージシステムに接続するために使用するエンドポイントです。 |
| aws.s3.access_key                | Yes  | IAMユーザーのアクセスキーです。                             |
| aws.s3.secret_key                | Yes  | IAMユーザーのシークレットキーです。                           |

##### Microsoft Azure Storage

アイスバーグカタログはv3.0以降でMicrosoft Azure Storageをサポートしています。

###### Azure Blob Storage

Blob Storageをアイスバーグクラスタのストレージとして選択した場合、次のいずれかの操作を実行します。

- 共有キー認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**              | **必須** | **説明**                                                     |
  | -------------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.storage_account | Yes      | Blob Storageアカウントのユーザー名です。                       |
  | azure.blob.shared_key      | Yes      | Blob Storageアカウントの共有キーです。                         |

- SASトークン認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**             | **必須** | **説明**                                                     |
  | ------------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.account_name   | Yes      | Blob Storageアカウントのユーザー名です。                       |
  | azure.blob.container_name | Yes      | データを格納するBlobコンテナの名前です。                       |
  | azure.blob.sas_token      | Yes      | Blob Storageアカウントにアクセスするために使用されるSASトークンです。 |

###### Azure Data Lake Storage Gen1

Data Lake Storage Gen1をアイスバーグクラスタのストレージとして選択した場合、次のいずれかの操作を実行します。

- マネージドサービスアイデンティティ認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                            | **必須** | **説明**                                                     |
  | ---------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes      | マネージドサービスアイデンティティ認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |

- サービスプリンシパル認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                 | **必須** | **説明**                                                     |
  | ----------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | Yes      | サービスプリンシパルのクライアント（アプリケーション）IDです。 |
  | azure.adls1.oauth2_credential | Yes      | 作成時に生成された新しいクライアント（アプリケーション）シークレットの値です。 |
  | azure.adls1.oauth2_endpoint   | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

###### Azure Data Lake Storage Gen2

Data Lake Storage Gen2をアイスバーグクラスタのストレージとして選択した場合、次のいずれかの操作を実行します。

- マネージドアイデンティティ認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                           | **必須** | **説明**                                                     |
  | --------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes      | マネージドアイデンティティ認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | Yes      | アクセスするテナントのIDです。                               |
  | azure.adls2.oauth2_client_id            | Yes      | マネージドアイデンティティのクライアント（アプリケーション）IDです。 |

- 共有キー認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**               | **必須** | **説明**                                                     |
  | --------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.storage_account | Yes      | Data Lake Storage Gen2ストレージアカウントのユーザー名です。 |
  | azure.adls2.shared_key      | Yes      | Data Lake Storage Gen2ストレージアカウントの共有キーです。 |

- サービスプリンシパル認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                      | **必須** | **説明**                                                     |
  | ---------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | Yes      | サービスプリンシパルのクライアント（アプリケーション）IDです。 |
  | azure.adls2.oauth2_client_secret   | Yes      | 作成時に生成された新しいクライアント（アプリケーション）シークレットの値です。 |
  | azure.adls2.oauth2_client_endpoint | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

##### Google GCS

アイスバーグカタログはv3.0以降でGoogle GCSをサポートしています。

Google GCSをアイスバーグクラスタのストレージとして選択した場合、次のいずれかの操作を実行します。

- VMベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                                     |
  | ------------------------------------------ | ---------------- | ----------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false            | true        | Compute Engineにバインドされているサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                          | **デフォルト値** | **値の例**                                                 | **説明**                                                     |
  | -------------------------------------- | ---------------- | --------------------------------------------------------- | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""               | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントのJSONファイル作成時に生成されたメールアドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                | サービスアカウントのJSONファイル作成時に生成されたプライベートキーIDです。 |
  | gcp.gcs.service_account_private_key    | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウントのJSONファイル作成時に生成されたプライベートキーです。 |

- 偽装ベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  - VMインスタンスがサービスアカウントを偽装する場合：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

    | **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                                     |
    | ------------------------------------------ | ---------------- | ----------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false            | true        | Compute Engineにバインドされているサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""               | "hello"     | 偽装するサービスアカウントです。                              |

  - サービスアカウント（一時的にメタサービスアカウントと呼ばれる）が別のサービスアカウント（一時的にデータサービスアカウントと呼ばれる）を偽装する場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

    | **パラメータ**                          | **デフォルト値** | **値の例**                                                 | **説明**                                                     |
    | -------------------------------------- | ---------------- | --------------------------------------------------------- | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""               | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントのJSONファイル作成時に生成されたメールアドレスです。 |
    | gcp.gcs.service_account_private_key_id | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                | サービスアカウントのJSONファイル作成時に生成されたプライベートキーIDです。 |
    | gcp.gcs.service_account_private_key    | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウントのJSONファイル作成時に生成されたプライベートキーです。 |
    | gcp.gcs.impersonation_service_account  | ""               | "hello"                                                    | 偽装するデータサービスアカウントです。                      |

### 例

次の例では、アイスバーグクラスタからデータをクエリするために、使用するメタストアに応じて`iceberg_catalog_hms`または`iceberg_catalog_glue`という名前のアイスバーグカタログを作成します。

#### HDFS

ストレージとしてHDFSを使用する場合、次のようなコマンドを実行します。

```SQL
CREATE EXTERNAL CATALOG iceberg_catalog_hms
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
);
```

#### AWS S3

##### インスタンスプロファイルベースの認証方法を選択する場合
- IcebergクラスターでHiveメタストアを使用する場合は、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR IcebergクラスターでAWS Glueを使用する場合は、次のようなコマンドを実行してください：

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

##### アサムドロールベースの認証を選択した場合

- IcebergクラスターでHiveメタストアを使用する場合は、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR IcebergクラスターでAWS Glueを使用する場合は、次のようなコマンドを実行してください：

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

##### IAMユーザーベースの認証を選択した場合

- IcebergクラスターでHiveメタストアを使用する場合は、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR IcebergクラスターでAWS Glueを使用する場合は、次のようなコマンドを実行してください：

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

#### S3互換ストレージシステム

MinIOを例に使用します。次のようなコマンドを実行してください：

```SQL
CREATE EXTERNAL CATALOG iceberg_catalog_hms
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "hive",
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

- 共有キー認証メソッドを選択した場合、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SASトークン認証メソッドを選択した場合、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- マネージドサービスアイデンティティ認証メソッドを選択した場合、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- サービスプリンシパル認証メソッドを選択した場合、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- マネージドアイデンティティ認証メソッドを選択した場合、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 共有キー認証メソッドを選択した場合、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- サービスプリンシパル認証メソッドを選択した場合、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMベースの認証メソッドを選択した場合、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- サービスアカウントベースの認証メソッドを選択した場合、次のようなコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- インパーソネーションベースの認証メソッドを選択した場合：

  - VMインスタンスがサービスアカウントをインパーソネートする場合、次のようなコマンドを実行してください：

    ```SQL
    CREATE EXTERNAL CATALOG iceberg_catalog_hms
    PROPERTIES
    (
        "type" = "iceberg",
        "iceberg.catalog.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - サービスアカウントが別のサービスアカウントをインパーソネートする場合、次のようなコマンドを実行してください：

    ```SQL
    CREATE EXTERNAL CATALOG iceberg_catalog_hms
    PROPERTIES
    (
        "type" = "iceberg",
        "iceberg.catalog.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    );
    ```

## Icebergカタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスター内のすべてのカタログをクエリできます：

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリできます。次の例では、`iceberg_catalog_glue`という名前のIcebergカタログの作成ステートメントをクエリしています：

```SQL
SHOW CREATE CATALOG iceberg_catalog_glue;
```

## Icebergカタログとその中のデータベースに切り替える

次のいずれかの方法を使用して、Icebergカタログとその中のデータベースに切り替えることができます：

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションでIcebergカタログを指定し、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します：

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替える：
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定する：
  USE <db_name>
  ```

- 直接[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、Icebergカタログとその中のデータベースに切り替えます：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Icebergカタログを削除する

外部カタログを削除するには、[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用します。

次の例では、`iceberg_catalog_glue`という名前のIcebergカタログを削除しています：

```SQL
DROP Catalog iceberg_catalog_glue;
```

## Icebergテーブルのスキーマを表示する

Icebergテーブルのスキーマを表示するには、次の構文のいずれかを使用できます：

- スキーマを表示する

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- スキーマとCREATEステートメントからの場所を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Icebergテーブルをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Icebergクラスター内のデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Icebergカタログとその中のデータベースに切り替えます](#icebergカタログとその中のデータベースに切り替える)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、指定したデータベース内の宛先テーブルをクエリします：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Icebergデータベースを作成する

StarRocksの内部カタログと同様に、Icebergカタログに[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)ステートメントを使用してデータベースを作成できます。この機能はv3.1以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Icebergカタログに切り替えます](#icebergカタログとその中のデータベースに切り替える)し、次のステートメントを使用してそのカタログ内にIcebergデータベースを作成します。

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>/")]
```

データベースを作成するファイルパスを指定するには、`location`パラメータを使用します。HDFSとクラウドストレージの両方がサポートされています。`location`パラメータを指定しない場合、StarRocksはデフォルトのファイルパスにデータベースを作成します。

`prefix`は使用するストレージシステムによって異なります：

| **ストレージシステム**                                         | **`prefix`の値**                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                       | `hdfs`                                                       |
| Google GCS                                                 | `gs`                                                         |
| Azure Blob Storage                                         | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可している場合、`prefix`は`wasb`です。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可している場合、`prefix`は`wasbs`です。</li></ul> |
| Azure Data Lake Storage Gen1                               | `adl`                                                        |
| Azure Data Lake Storage Gen2                               | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可している場合、`prefix`は`abfs`です。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可している場合、`prefix`は`abfss`です。</li></ul> |
| AWS S3またはその他のS3互換ストレージ（たとえば、MinIO） | `s3`                                                         |

## Icebergデータベースを削除する

StarRocksの内部データベースと同様に、Icebergデータベースに[DROP](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md)ステートメントを使用してデータベースを削除できます。この機能はv3.1以降でサポートされています。空のデータベースのみ削除できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

Icebergデータベースを削除するには、[Icebergカタログに切り替えます](#icebergカタログとその中のデータベースに切り替える)し、次のステートメントを使用します。

```SQL
DROP DATABASE <database_name>;
```

## Icebergテーブルを作成する

StarRocksの内部データベースと同様に、Icebergデータベースに[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)または[CREATE TABLE AS SELECT（CTAS）](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)ステートメントを使用してテーブルを作成できます。この機能はv3.1以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Icebergカタログとその中のデータベースに切り替えます](#icebergカタログとその中のデータベースに切り替える)し、次の構文を使用してそのデータベース内にIcebergテーブルを作成します。

### 構文

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

`column_definition`の構文は次のとおりです。

```SQL
col_name col_type [COMMENT 'comment']
```

以下の表に、パラメータの説明を示します。

| パラメータ     | 説明                                              |
| ----------------- |------------------------------------------------------ |
| col_name          | カラムの名前                                                |
| col_type          | カラムのデータ型。以下のデータ型がサポートされています：TINYINT、SMALLINT、INT、BIGINT、FLOAT、DOUBLE、DECIMAL、DATE、DATETIME、CHAR、VARCHAR（長さ）、ARRAY、MAP、STRUCT。LARGEINT、HLL、BITMAPのデータ型はサポートされていません。 |

> **注意**
>
> パーティション以外のすべてのカラムはデフォルト値として`NULL`を使用する必要があります。つまり、テーブル作成ステートメントの非パーティションカラムごとに`DEFAULT "NULL"`を指定する必要があります。また、パーティションカラムは非パーティションカラムの後に定義する必要があり、デフォルト値として`NULL`を使用することはできません。

#### partition_desc

`partition_desc`の構文は次のとおりです。

```SQL
PARTITION BY (par_col1[, par_col2...])
```

現在、StarRocksは[identity transforms](https://iceberg.apache.org/spec/#partitioning)のみをサポートしており、つまり、StarRocksは一意のパーティション値ごとにパーティションを作成します。

> **注意**
>
> パーティションカラムは非パーティションカラムの後に定義する必要があります。パーティションカラムは、FLOAT、DOUBLE、DECIMAL、DATETIMEを除くすべてのデータ型をサポートし、デフォルト値として`NULL`を使用することはできません。

#### PROPERTIES

`PROPERTIES`では、`"key" = "value"`形式でテーブル属性を指定できます。[Icebergテーブル属性](https://iceberg.apache.org/docs/latest/configuration/)を参照してください。

以下の表にいくつかの主要なプロパティを説明します。

| プロパティ      | 説明                                              |
| ----------------- | ------------------------------------------------------------ |
| location          | Icebergテーブルを作成するファイルパス。HMSをメタストアとして使用する場合、`location`パラメータを指定する必要はありません。なぜなら、StarRocksはテーブルを現在のIcebergカタログのデフォルトのファイルパスに作成するからです。AWS Glueをメタストアとして使用する場合：<ul><li>テーブルを作成するデータベースの`location`パラメータを指定した場合、テーブルの`location`パラメータを指定する必要はありません。その場合、テーブルは所属するデータベースのファイルパスにデフォルトで作成されます。</li><li>テーブルを作成するデータベースの`location`を指定していない場合、テーブルの`location`パラメータを指定する必要があります。</li></ul> |
| file_format       | Icebergテーブルのファイルフォーマット。Parquetフォーマットのみがサポートされています。デフォルト値：`parquet` |
| compression_codec | Icebergテーブルに使用する圧縮アルゴリズム。サポートされている圧縮アルゴリズムはSNAPPY、GZIP、ZSTD、LZ4です。デフォルト値：`gzip` |

### 例

1. `unpartition_tbl`という名前の非パーティションテーブルを作成します。テーブルは2つのカラム、`id`と`score`で構成されています。

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. `partition_tbl_1`という名前のパーティションテーブルを作成します。テーブルは3つのカラム、`action`、`id`、`dt`で構成されており、`id`と`dt`がパーティションカラムとして定義されています。

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. `partition_tbl_1`という名前の既存のテーブルをクエリし、そのクエリ結果を使用して`partition_tbl_2`という名前のパーティションテーブルを作成します。`partition_tbl_2`では、`id`と`dt`がパーティションカラムとして定義されています。

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (id, dt)
   AS SELECT * from employee;
   ```

## Icebergテーブルにデータをシンクする

StarRocksの内部テーブルと同様に、Icebergテーブルに[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ステートメントを使用してStarRocksテーブルのデータをシンクすることができます（現在はParquet形式のIcebergテーブルのみサポートされています）。この機能はv3.1以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Icebergカタログとその中のデータベースに切り替えます](#icebergカタログとその中のデータベースに切り替える)し、次の構文を使用してStarRocksテーブルのデータをParquet形式のIcebergテーブルにシンクします。

### 構文

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータをシンクする場合は、次の構文を使用します：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

> **注意**
>
> パーティションカラムには`NULL`値を指定できません。したがって、Icebergテーブルのパーティションカラムに空の値がロードされないようにする必要があります。

### パラメータ

| パラメータ   | 説明                                                         |
| ----------- | ------------------------------------------------------------ |
| INTO        | StarRocksテーブルのデータをIcebergテーブルに追加します。                                       |
| OVERWRITE   | Icebergテーブルの既存のデータをStarRocksテーブルのデータで上書きします。                                       |
| column_name | データをロードする宛先カラムの名前を指定します。1つ以上のカラムを指定できます。複数のカラムを指定する場合は、カンマ（`,`）で区切ります。指定する宛先カラムは、実際にIcebergテーブルに存在するカラムのみ指定でき、指定する宛先カラムにはIcebergテーブルのパーティションカラムを含める必要があります。指定した宛先カラムは、宛先カラムの名前が何であるかに関係なく、StarRocksテーブルのカラムと1対1でマッピングされます。宛先カラムが指定されていない場合、データはIcebergテーブルのすべてのカラムにロードされます。StarRocksテーブルの非パーティションカラムがIcebergテーブルのどのカラムにもマッピングされない場合、StarRocksはデフォルト値`NULL`をIcebergテーブルのカラムに書き込みます。INSERTステートメントには、返されるカラムの型が宛先カラムのデータ型と異なるクエリステートメントが含まれている場合、StarRocksは不一致のカラムに対して暗黙の型変換を実行します。変換に失敗した場合、構文解析エラーが返されます。 |
| expression  | 宛先カラムに値を割り当てる式。                                   |
| DEFAULT     | 宛先カラムにデフォルト値を割り当てます。                                         |
| query       | Icebergテーブルにロードされるクエリステートメントの結果。StarRocksでサポートされている任意のSQLステートメントを使用できます。 |
| PARTITION   | データをロードするパーティションを指定します。このプロパティでは、Icebergテーブルのすべてのパーティションカラムを指定する必要があります。このプロパティで指定するパーティションカラムは、テーブル作成ステートメントで定義したパーティションカラムとは異なる順序で指定することができます。このプロパティを指定する場合、`column_name`プロパティを指定することはできません。 |

### 例

1. `partition_tbl_1`テーブルに3つのデータ行を挿入します：

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. SELECTクエリの結果を挿入します。このクエリには、単純な計算が含まれています。

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. SELECTクエリの結果を挿入します。このクエリは`partition_tbl_1`テーブルからデータを読み取ります。

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. SELECTクエリの結果を、`dt='2023-09-01'`および`id=1`の2つの条件を満たすパーティションに挿入します。

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT INTO partition_tbl_2 partition(dt='2023-09-01',id=1) SELECT 'order';
   ```

5. `partition_tbl_1`テーブルの`dt='2023-09-01'`および`id=1`の2つの条件を満たすパーティションのすべての`action`カラムの値を`close`に上書きします。

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT OVERWRITE partition_tbl_1 partition(dt='2023-09-01',id=1) SELECT 'close';
   ```

## Icebergテーブルを削除する

StarRocksの内部テーブルと同様に、Icebergテーブルを削除するには、[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)ステートメントを使用します。この機能はv3.1以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

Icebergテーブルを削除すると、テーブルのファイルパスとHDFSクラスターまたはクラウドストレージ上のデータは削除されません。
Icebergテーブルを強制的に削除する場合（つまり、DROP TABLEステートメントで`FORCE`キーワードが指定されている場合）、テーブルのデータはHDFSクラスターまたはクラウドストレージ上でテーブルと共に削除されますが、テーブルのファイルパスは保持されます。

[Icebergカタログとその中のデータベースに切り替える](#switch-to-an-iceberg-catalog-and-a-database-in-it) し、次のステートメントを使用してそのデータベース内のIcebergテーブルを削除します。

```SQL
DROP TABLE <table_name> [FORCE];
```

## メタデータキャッシュの設定

Icebergクラスターのメタデータファイルは、AWS S3やHDFSなどのリモートストレージに保存される場合があります。StarRocksはデフォルトでIcebergメタデータをメモリにキャッシュします。クエリの高速化のために、StarRocksはメモリとディスクの両方にメタデータキャッシュの仕組みを採用しています。初期クエリごとに、StarRocksは計算結果をキャッシュします。以前のクエリと意味的に等価な後続のクエリが発行された場合、StarRocksはまずキャッシュから要求されたメタデータを取得し、メタデータがキャッシュにヒットしない場合にのみリモートストレージからメタデータを取得します。

StarRocksは最も最近使用された（LRU）アルゴリズムを使用してデータをキャッシュし、削除します。基本的なルールは次のとおりです。

- StarRocksはまずメモリから要求されたメタデータを取得しようとします。メモリにメタデータがヒットしない場合、StarRocksはディスクからメタデータを取得しようとします。StarRocksがディスクから取得したメタデータはメモリにロードされます。ディスクにメタデータがヒットしない場合、StarRocksはリモートストレージからメタデータを取得し、取得したメタデータをメモリにキャッシュします。
- StarRocksはメモリから削除されたメタデータをディスクに書き込みますが、ディスクから削除されたメタデータは直接破棄されます。

以下の表は、Icebergメタデータキャッシュメカニズムを設定するために使用できるFE設定項目を説明しています。

| **設定項目**                                      | **単位** | **デフォルト値**                                      | **説明**                                                     |
| ------------------------------------------------ | -------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| enable_iceberg_metadata_disk_cache               | N/A      | `false`                                              | ディスクキャッシュを有効にするかどうかを指定します。                  |
| iceberg_metadata_cache_disk_path                 | N/A      | `StarRocksFE.STARROCKS_HOME_DIR + "/caches/iceberg"` | ディスク上のキャッシュされたメタデータファイルの保存パスです。              |
| iceberg_metadata_disk_cache_capacity             | Bytes    | `2147483648`（2 GBに相当）                          | ディスク上で許可されるキャッシュされたメタデータの最大サイズです。         |
| iceberg_metadata_memory_cache_capacity           | Bytes    | `536870912`（512 MBに相当）                         | メモリに許可されるキャッシュされたメタデータの最大サイズです。           |
| iceberg_metadata_memory_cache_expiration_seconds | Seconds  | `86500`                                              | メモリ内のキャッシュエントリが最後にアクセスされてからの経過時間です。 |
| iceberg_metadata_disk_cache_expiration_seconds   | Seconds  | `604800`（1週間に相当）                             | ディスク上のキャッシュエントリが最後にアクセスされてからの経過時間です。 |
| iceberg_metadata_cache_max_entry_size            | Bytes    | `8388608`（8 MBに相当）                              | キャッシュできるファイルの最大サイズです。このパラメータの値を超えるサイズのファイルはキャッシュできません。クエリがこれらのファイルを要求した場合、StarRocksはリモートストレージから取得します。 |
