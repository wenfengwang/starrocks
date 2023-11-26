---
displayed_sidebar: "Japanese"
---

# Hiveカタログ

Hiveカタログは、StarRocks v2.4以降でサポートされる外部カタログの一種です。Hiveカタログでは、次の操作ができます。

- 手動でテーブルを作成する必要なく、Hiveに格納されたデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)または非同期マテリアライズドビュー（v2.5以降でサポート）を使用して、Hiveに格納されたデータを処理し、データをStarRocksにロードできます。
- StarRocksで操作を実行して、Hiveデータベースやテーブルを作成または削除したり、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用してStarRocksテーブルからParquet形式のHiveテーブルにデータをシンクすることができます（この機能はv3.2以降でサポート）。

HiveクラスタでのSQLワークロードの成功を保証するためには、StarRocksクラスタを次の2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCSなどのオブジェクトストレージ、またはその他のS3互換ストレージシステム（たとえば、MinIO）。
- HiveメタストアまたはAWS Glueのようなメタストア。

  > **注記**
  >
  > ストレージとしてAWS S3を選択した場合、メタストアとしてHMSまたはAWS Glueを使用できます。他のストレージシステムを選択した場合、メタストアとしてHMSのみを使用できます。

## 使用上の注意

- StarRocksがサポートするHiveのファイル形式は、Parquet、ORC、CSV、Avro、RCFile、およびSequenceFileです。

  - Parquetファイルは、次の圧縮形式をサポートしています：SNAPPY、LZ4、ZSTD、GZIP、およびNO_COMPRESSION。
  - ORCファイルは、次の圧縮形式をサポートしています：ZLIB、SNAPPY、LZO、LZ4、ZSTD、およびNO_COMPRESSION。

- StarRocksがサポートしないHiveのデータ型は、INTERVAL、BINARY、およびUNIONです。さらに、StarRocksはCSV形式のHiveテーブルに対してMAPおよびSTRUCTデータ型をサポートしていません。
- Hiveカタログはデータのクエリにのみ使用できます。Hiveカタログを使用してHiveクラスタにデータをドロップ、削除、または挿入することはできません。

## 統合の準備

Hiveカタログを作成する前に、StarRocksクラスタがHiveクラスタのストレージシステムとメタストアと統合できることを確認してください。

### AWS IAM

HiveクラスタがストレージとしてAWS S3またはメタストアとしてAWS Glueを使用する場合は、適切な認証方法を選択し、関連するAWSクラウドリソースにStarRocksクラスタがアクセスできるように必要な準備を行ってください。

以下の認証方法が推奨されています。

- インスタンスプロファイル
- 仮定された役割
- IAMユーザー

上記の3つの認証方法のうち、インスタンスプロファイルが最も一般的に使用されています。

詳細については、[AWS IAMでの認証の準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとしてHDFSを選択した場合は、StarRocksクラスタを次のように構成してください。

- （オプション）HDFSクラスタとHiveメタストアにアクセスするために使用されるユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスタとHiveメタストアにアクセスします。各FEの**fe/conf/hadoop_env.sh**ファイルと各BEの**be/conf/hadoop_env.sh**ファイルの先頭に`export HADOOP_USER_NAME="<user_name>"`を追加してユーザー名を設定することもできます。これらのファイルでユーザー名を設定した後、各FEと各BEを再起動してパラメータ設定を有効にします。1つのStarRocksクラスタについては、1つのユーザー名のみを設定できます。
- Hiveデータをクエリする際、StarRocksクラスタのFEおよびBEはHDFSクライアントを使用してHDFSクラスタにアクセスします。ほとんどの場合、この目的を達成するためにStarRocksクラスタを構成する必要はありません。StarRocksはデフォルトの設定でHDFSクライアントを起動します。StarRocksクラスタを構成する必要があるのは、次の場合だけです。

  - HDFSクラスタで高可用性（HA）が有効になっている場合：HDFSクラスタの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。
  - HDFSクラスタでView File System（ViewFs）が有効になっている場合：HDFSクラスタの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。

> **注記**
>
> クエリを送信する際にホスト名が不明のホストを示すエラーが返される場合は、HDFSクラスタノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに追加する必要があります。

### Kerberos認証

HDFSクラスタまたはHiveメタストアでKerberos認証が有効になっている場合は、StarRocksクラスタを次のように構成してください。

- 各FEおよび各BEで`kinit -kt keytab_path principal`コマンドを実行し、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFSクラスタとHiveメタストアにアクセスする権限が必要です。このコマンドを使用してKDCにアクセスする際は、時間の制約があるため、cronを使用して定期的にこのコマンドを実行する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルと各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。この例では、`/etc/krb5.conf`は**krb5.conf**ファイルの保存パスです。必要に応じてパスを変更してください。

## Hiveカタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
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

#### catalog_name

Hiveカタログの名前です。命名規則は次のとおりです。

- 名前には、文字、数字（0-9）、およびアンダースコア（_）を含めることができます。ただし、文字で始める必要があります。
- 名前は大文字と小文字を区別し、長さが1023文字を超えることはできません。

#### comment

Hiveカタログの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。値を`hive`に設定します。

#### GeneralParams

一連の一般的なパラメータです。

次の表には、`GeneralParams`で設定できるパラメータの説明があります。

| パラメータ                  | 必須     | 説明                                                         |
| -------------------------- | -------- | ------------------------------------------------------------ |
| enable_recursive_listing   | いいえ   | StarRocksがテーブルとそのパーティション、およびテーブルとそのパーティションの物理的な場所内のサブディレクトリからデータを読み取るかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。値が`true`の場合、サブディレクトリを再帰的にリストし、値が`false`の場合、サブディレクトリを無視します。 |

#### MetastoreParams

データソースのメタストアとの統合方法に関するパラメータのセットです。

##### Hiveメタストア

データソースのメタストアとしてHiveメタストアを選択した場合、`MetastoreParams`を次のように構成します。

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> Hiveデータをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに追加する必要があります。そうしないと、クエリを開始するときにStarRocksがHiveメタストアにアクセスできなくなる場合があります。

次の表に、`MetastoreParams`で構成する必要があるパラメータの説明があります。

| パラメータ             | 必須 | 説明                                                         |
| --------------------- | ---- | ------------------------------------------------------------ |
| hive.metastore.type   | はい | Hiveクラスタで使用するメタストアのタイプです。値を`hive`に設定します。 |
| hive.metastore.uris   | はい | HiveメタストアのURIです。形式：`thrift://<metastore_IP_address>:<metastore_port>`。<br />Hiveメタストアで高可用性（HA）が有効になっている場合、複数のメタストアURIを指定し、カンマ（`,`）で区切ってください。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`のように指定します。 |

##### AWS Glue

データソースのメタストアとしてAWS Glueを選択した場合（ストレージとしてAWS S3を選択した場合のみサポート）、次のいずれかの操作を実行します。

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams`を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 仮定された役割ベースの認証方法を選択する場合、`MetastoreParams`を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAMユーザーベースの認証方法を選択する場合、`MetastoreParams`を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

次の表に、`MetastoreParams`で構成する必要があるパラメータの説明があります。

| パラメータ                     | 必須 | 説明                                                         |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| hive.metastore.type           | はい | Hiveクラスタで使用するメタストアのタイプです。値を`glue`に設定します。 |
| aws.glue.use_instance_profile | はい | インスタンスプロファイルベースの認証方法と仮定された役割ベースの認証を有効にするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。 |
| aws.glue.iam_role_arn         | いいえ | AWS Glue Data Catalogに特権を持つIAMロールのARNです。AWS Glueへのアクセスに仮定された役割ベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.region               | はい | AWS Glue Data Catalogが存在するリージョンです。例：`us-west-1`。 |
| aws.glue.access_key           | いいえ | AWS IAMユーザーのアクセスキーです。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | いいえ | AWS IAMユーザーのシークレットキーです。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |

AWS Glueへのアクセス方法の選択およびAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS Glueへのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

ストレージシステムとの統合方法に関するパラメータのセットです。このパラメータセットはオプションです。

ストレージとしてHDFSを使用する場合、`StorageCredentialParams`を構成する必要はありません。

ストレージとしてAWS S3、その他のS3互換ストレージシステム、Microsoft Azure Storage、またはGoogle GCSを使用する場合は、`StorageCredentialParams`を構成する必要があります。

##### AWS S3

HiveクラスタのストレージとしてAWS S3を選択する場合、次のいずれかの操作を実行します。

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

次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

| パラメータ                   | 必須 | 説明                                                         |
| --------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい | インスタンスプロファイルベースの認証方法と仮定された役割ベースの認証方法を有効にするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。 |
| aws.s3.iam_role_arn         | いいえ | AWS S3バケットに特権を持つIAMロールのARNです。AWS S3へのアクセスに仮定された役割ベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい | AWS S3バケットが存在するリージョンです。例：`us-west-1`。 |
| aws.s3.access_key           | いいえ | IAMユーザーのアクセスキーです。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ | IAMユーザーのシークレットキーです。AWS S3へのアクセスにIAMユーザーベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |

AWS S3へのアクセス方法の選択およびAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS S3へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

Hiveカタログは、v2.5以降でS3互換ストレージシステム（たとえば、MinIO）をサポートしています。

S3互換ストレージシステム（たとえば、MinIO）をHiveクラスタのストレージとして選択する場合は、次のように`StorageCredentialParams`を構成して、正常な統合を確保してください。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

| パラメータ                        | 必須 | 説明                                                         |
| -------------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | はい | SSL接続を有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`true`。 |
| aws.s3.enable_path_style_access  | はい | パススタイルアクセスを有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`false`。MinIOの場合、値を`true`に設定する必要があります。<br />パススタイルURLは、次の形式を使用します：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、US West（Oregon）リージョンに`DOC-EXAMPLE-BUCKET1`という名前のバケットを作成し、そのバケットの`alice.jpg`オブジェクトにアクセスする場合、次のパススタイルURLを使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | はい | AWS S3の代わりにS3互換ストレージシステムに接続するために使用されるエンドポイントです。 |
| aws.s3.access_key                | はい | IAMユーザーのアクセスキーです。 |
| aws.s3.secret_key                | はい | IAMユーザーのシークレットキーです。 |

##### Microsoft Azure Storage

Hiveカタログは、v3.0以降でMicrosoft Azure Storageをサポートしています。

###### Azure Blob Storage

HiveクラスタのストレージとしてBlob Storageを選択する場合、次のいずれかの操作を実行します。

- 共有キー認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

  | **パラメータ**              | **必須** | **説明**                                                     |
  | -------------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.storage_account | はい     | Blob Storageアカウントのユーザー名です。                       |
  | azure.blob.shared_key      | はい     | Blob Storageアカウントの共有キーです。                         |

- SASトークン認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

  | **パラメータ**             | **必須** | **説明**                                                     |
  | ------------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.account_name   | はい     | Blob Storageアカウントのユーザー名です。                       |
  | azure.blob.container_name | はい     | データを格納するBlobコンテナの名前です。                       |
  | azure.blob.sas_token      | はい     | Blob Storageアカウントにアクセスするために使用されるSASトークンです。 |

###### Azure Data Lake Storage Gen1

HiveクラスタのストレージとしてData Lake Storage Gen1を選択する場合、次のいずれかの操作を実行します。

- Managed Service Identity認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

  | **パラメータ**                            | **必須** | **説明**                                                     |
  | ---------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい     | Managed Service Identity認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |

- Service Principal認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

  | **パラメータ**                 | **必須** | **説明**                                                     |
  | ----------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい     | サービスプリンシパルのクライアント（アプリケーション）IDです。 |
  | azure.adls1.oauth2_credential | はい     | 作成時に生成された新しいクライアント（アプリケーション）シークレットの値です。 |
  | azure.adls1.oauth2_endpoint   | はい     | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

###### Azure Data Lake Storage Gen2

HiveクラスタのストレージとしてData Lake Storage Gen2を選択する場合、次のいずれかの操作を実行します。

- Managed Identity認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

  | **パラメータ**                           | **必須** | **説明**                                                     |
  | --------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | はい     | Managed Identity認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | はい     | アクセスするデータのテナントのIDです。                       |
  | azure.adls2.oauth2_client_id            | はい     | マネージドアイデンティティのクライアント（アプリケーション）IDです。 |

- Shared Key認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

  | **パラメータ**               | **必須** | **説明**                                                     |
  | --------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.storage_account | はい     | Data Lake Storage Gen2ストレージアカウントのユーザー名です。 |
  | azure.adls2.shared_key      | はい     | Data Lake Storage Gen2ストレージアカウントの共有キーです。 |

- Service Principal認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

  | **パラメータ**                      | **必須** | **説明**                                                     |
  | ---------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい     | サービスプリンシパルのクライアント（アプリケーション）IDです。 |
  | azure.adls2.oauth2_client_secret   | はい     | 作成時に生成された新しいクライアント（アプリケーション）シークレットの値です。 |
  | azure.adls2.oauth2_client_endpoint | はい     | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

##### Google GCS

Hiveカタログは、v3.0以降でGoogle GCSをサポートしています。

HiveクラスタのストレージとしてGoogle GCSを選択する場合、次のいずれかの操作を実行します。

- VMベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

  | **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                                     |
  | ------------------------------------------ | ---------------- | ----------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false            | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

  | **パラメータ**                          | **デフォルト値** | **値の例**                                                | **説明**                                                     |
  | -------------------------------------- | ---------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""               | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントのJSONファイル作成時に生成されたメールアドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | サービスアカウントのJSONファイル作成時に生成されたプライベートキーIDです。 |
  | gcp.gcs.service_account_private_key    | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウントのJSONファイル作成時に生成されたプライベートキーです。 |

- 偽装ベースの認証方法を選択する場合、次のいずれかの操作を実行します。

  - VMインスタンスをサービスアカウントに偽装する場合：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

    | **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                                     |
    | ------------------------------------------ | ---------------- | ----------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false            | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""               | "hello"     | 偽装したいサービスアカウントです。                           |

  - サービスアカウント（一時的にメタサービスアカウントと呼ばれる）が別のサービスアカウント（一時的にデータサービスアカウントと呼ばれる）を偽装する場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表に、`StorageCredentialParams`で構成する必要があるパラメータの説明があります。

    | **パラメータ**                          | **デフォルト値** | **値の例**                                                | **説明**                                                     |
    | -------------------------------------- | ---------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""               | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントのJSONファイル作成時に生成されたメールアドレスです。 |
    | gcp.gcs.service_account_private_key_id | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | サービスアカウントのJSONファイル作成時に生成されたプライベートキーIDです。 |
    | gcp.gcs.service_account_private_key    | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウントのJSONファイル作成時に生成されたプライベートキーです。 |


    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | JSONファイルのメタサービスアカウントの作成時に生成されたプライベートキーIDです。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | JSONファイルのメタサービスアカウントの作成時に生成されたプライベートキーです。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 擬似化したいデータサービスアカウントです。       |

#### MetadataUpdateParams

Hiveのキャッシュメタデータを更新する方法に関するパラメータのセットです。このパラメータセットはオプションです。

StarRocksはデフォルトで[自動非同期更新ポリシー](#appendix-understand-metadata-automatic-asynchronous-update)を実装しています。

ほとんどの場合、`MetadataUpdateParams`を無視し、それに含まれるポリシーパラメータを調整する必要はありません。なぜなら、これらのパラメータのデフォルト値はすでに使いやすいパフォーマンスを提供しているからです。

ただし、Hiveのデータ更新頻度が高い場合は、これらのパラメータを調整して自動非同期更新のパフォーマンスをさらに最適化することができます。

> **注意**
>
> ほとんどの場合、Hiveデータが1時間以下の粒度で更新される場合、データ更新頻度は高いと見なされます。

| パラメータ                              | 必須 | 説明                                                  |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | No       | StarRocksがHiveテーブルのメタデータをキャッシュするかどうかを指定します。有効な値は`true`と`false`です。デフォルト値は`true`です。値が`true`の場合、キャッシュが有効になり、値が`false`の場合、キャッシュが無効になります。 |
| enable_remote_file_cache               | No       | StarRocksがHiveテーブルまたはパーティションの基礎データファイルのメタデータをキャッシュするかどうかを指定します。有効な値は`true`と`false`です。デフォルト値は`true`です。値が`true`の場合、キャッシュが有効になり、値が`false`の場合、キャッシュが無効になります。 |
| metastore_cache_refresh_interval_sec   | No       | StarRocksが自身でキャッシュされたHiveテーブルまたはパーティションのメタデータを非同期に更新する時間間隔です。単位：秒。デフォルト値は`7200`で、2時間です。 |
| remote_file_cache_refresh_interval_sec | No       | StarRocksが自身でキャッシュされたHiveテーブルまたはパーティションの基礎データファイルのメタデータを非同期に更新する時間間隔です。単位：秒。デフォルト値は`60`です。 |
| metastore_cache_ttl_sec                | No       | StarRocksが自動的に破棄するHiveテーブルまたはパーティションのメタデータの時間間隔です。単位：秒。デフォルト値は`86400`で、24時間です。 |
| remote_file_cache_ttl_sec              | No       | StarRocksが自動的に破棄するHiveテーブルまたはパーティションの基礎データファイルのメタデータの時間間隔です。単位：秒。デフォルト値は`129600`で、36時間です。 |
| enable_cache_list_names                | No       | StarRocksがHiveパーティション名をキャッシュするかどうかを指定します。有効な値は`true`と`false`です。デフォルト値は`true`です。値が`true`の場合、キャッシュが有効になり、値が`false`の場合、キャッシュが無効になります。 |

### 例

以下の例では、Hiveクラスタからデータをクエリするために、`hive_catalog_hms`または`hive_catalog_glue`という名前のHiveカタログを作成します。作成するカタログは、使用するメタストアのタイプによって異なります。

#### HDFS

ストレージとしてHDFSを使用する場合、次のようなコマンドを実行します。

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

- HiveクラスタでHiveメタストアを使用する場合、次のようなコマンドを実行します。

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

- Amazon EMR HiveクラスタでAWS Glueを使用する場合、次のようなコマンドを実行します。

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

##### 偽装ベースの認証

- HiveクラスタでHiveメタストアを使用する場合、次のようなコマンドを実行します。

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

- Amazon EMR HiveクラスタでAWS Glueを使用する場合、次のようなコマンドを実行します。

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

- HiveクラスタでHiveメタストアを使用する場合、次のようなコマンドを実行します。

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

- Amazon EMR HiveクラスタでAWS Glueを使用する場合、次のようなコマンドを実行します。

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

MinIOを例として使用します。次のようなコマンドを実行します。

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

- 共有キー認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- SASトークン認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- マネージドサービスアイデンティティ認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- サービスプリンシパル認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- マネージドアイデンティティ認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- 共有キー認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- サービスプリンシパル認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- VMベースの認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- サービスアカウントベースの認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- 偽装ベースの認証メソッドを選択した場合:

  - VMインスタンスがサービスアカウントを偽装する場合、次のようなコマンドを実行します。

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

  - サービスアカウントが別のサービスアカウントを偽装する場合、次のようなコマンドを実行します。

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

## Hiveカタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスタ内のすべてのカタログをクエリできます。

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリできます。次の例では、Hiveカタログ`hive_catalog_glue`の作成ステートメントをクエリしています。

```SQL
SHOW CREATE CATALOG hive_catalog_glue;
```

## Hiveカタログとその内のデータベースに切り替える

Hiveカタログとその内のデータベースに切り替えるには、次のいずれかの方法を使用できます。

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションでHiveカタログを指定し、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します。

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替える:
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定する:
  USE <db_name>
  ```

- [USE](../../sql-reference/sql-statements/data-definition/USE.md)を直接使用して、Hiveカタログとその内のデータベースに切り替えます。

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Hiveカタログを削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、外部カタログを削除できます。

次の例では、`hive_catalog_glue`という名前のHiveカタログを削除しています。

```SQL
DROP Catalog hive_catalog_glue;
```

## Hiveテーブルのスキーマを表示する

Hiveテーブルのスキーマを表示するには、次の構文のいずれかを使用できます。

- スキーマを表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Hiveテーブルをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Hiveクラスタ内のデータベースを表示します。

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Hiveカタログとその内のデータベースに切り替えます](#hiveカタログとその内のデータベースに切り替える)。

3. 指定したデータベースの宛先テーブルをクエリするために[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用します。

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hiveからデータをロードする

OLAPテーブル`olap_tbl`があると仮定し、次のようにデータを変換してロードできます。

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hive_table
```

## Hiveテーブルとビューに対する権限を付与する

[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)ステートメントを使用して、特定のロールにHiveカタログ内のすべてのテーブルまたはビューに対する権限を付与できます。

- ロールにHiveカタログ内のすべてのテーブルに対するクエリ権限を付与する場合:

  ```SQL
  GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE <role_name>
  ```

- ロールにHiveカタログ内のすべてのビューに対するクエリ権限を付与する場合:

  ```SQL
  GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE <role_name>
  ```

たとえば、次のコマンドを使用して`hive_role_table`という名前のロールを作成し、Hiveカタログ`hive_catalog`に対して`hive_role_table`ロールに対するすべてのテーブルとビューのクエリ権限を付与します。

```SQL
-- hive_role_tableという名前のロールを作成します。
CREATE ROLE hive_role_table;

-- Hiveカタログhive_catalogに切り替えます。
SET CATALOG hive_catalog;

-- hive_role_tableロールにHiveカタログhive_catalog内のすべてのテーブルのクエリ権限を付与します。
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE hive_role_table;

-- hive_role_tableロールにHiveカタログhive_catalog内のすべてのビューのクエリ権限を付与します。
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE hive_role_table;
```

## Hiveデータベースを作成する

StarRocksの内部カタログと同様に、Hiveデータベースに[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)権限がある場合、そのHiveデータベースにデータベースを作成するために[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)ステートメントを使用できます。この機能はv3.2以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Hiveカタログに切り替えます](#hiveカタログとその内のデータベースに切り替える)。次に、次のステートメントを使用してそのデータベース内にHiveデータベースを作成します。

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location`パラメータは、データベースを作成するファイルパスを指定します。これはHDFSまたはクラウドストレージのいずれかになります。

- HiveクラスタのメタストアとしてHiveメタストアを使用する場合、`location`パラメータのデフォルト値は`<warehouse_location>/<database_name.db>`で、データベースの作成時にそのパラメータを指定しない場合はHiveメタストアでサポートされます。
- HiveクラスタのメタストアとしてAWS Glueを使用する場合、`location`パラメータにはデフォルト値がなく、データベースの作成時にそのパラメータを指定する必要があります。

`prefix`は使用するストレージシステムによって異なります。

| **ストレージシステム**                                         | **`prefix`** **の値**                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                       | `hdfs`                                                       |
| Google GCS                                                 | `gs`                                                         |
| Azure Blob Storage                                         | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可している場合、`prefix`は`wasb`です。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可している場合、`prefix`は`wasbs`です。</li></ul> |
| Azure Data Lake Storage Gen1                               | `adl`                                                        |
| Azure Data Lake Storage Gen2                               | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可している場合、`prefix`は`abfs`です。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可している場合、`prefix`は`abfss`です。</li></ul> |
| AWS S3またはその他のS3互換ストレージ（たとえば、MinIO） | `s3`                                                         |

## Hiveデータベースを削除する

StarRocksの内部データベースと同様に、Hiveデータベースを削除するには、[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md)ステートメントを使用できます。この機能はv3.2以降でサポートされています。空のデータベースのみを削除できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

Hiveデータベースを削除すると、データベースのファイルパスはデータベースと一緒に削除されません。

[Hiveカタログに切り替えます](#hiveカタログとその内のデータベースに切り替える)。次に、次のステートメントを使用してそのカタログ内のHiveデータベースを削除します。

```SQL
DROP DATABASE <database_name>
```

## Hiveテーブルを作成する

StarRocksの内部データベースと同様に、Hiveデータベースに[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)または[CREATE TABLE AS SELECT（CTAS）](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)ステートメントを使用して管理されたテーブルを作成することができます。この機能はv3.2以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Hiveカタログとその内のデータベースに切り替えます](#hiveカタログとその内のデータベースに切り替える)。次に、次の構文を使用してそのデータベース内にHive管理テーブルを作成します。

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

次の表に、パラメータの説明を示します。

| パラメータ | 説明                                                  |
| --------- | ------------------------------------------------------------ |
| col_name  | 列の名前。                                      |
| col_type  | 列のデータ型。次のデータ型がサポートされています：TINYINT、SMALLINT、INT、BIGINT、FLOAT、DOUBLE、DECIMAL、DATE、DATETIME、CHAR、VARCHAR（長さ）、ARRAY、MAP、およびSTRUCT。LARGEINT、HLL、およびBITMAPデータ型はサポートされていません。 |

> **注意**
>
> すべての非パーティション列はデフォルト値として`NULL`を使用する必要があります。つまり、テーブル作成ステートメントの各非パーティション列に対して`DEFAULT "NULL"`を指定する必要があります。また、パーティション列は非パーティション列の後に定義する必要があり、`NULL`をデフォルト値として使用することはできません。

#### partition_desc

`partition_desc`の構文は次のとおりです。

```SQL
PARTITION BY (par_col1[, par_col2...])
```

現在、StarRocksはID変換のみをサポートしており、つまり、StarRocksは一意のパーティション値ごとにパーティションを作成します。

> **注意**
>
> パーティション列は非パーティション列の後に定義する必要があります。パーティション列はFLOAT、DOUBLE、DECIMAL、DATETIMEを除くすべてのデータ型をサポートし、デフォルト値として`NULL`を使用することはできません。また、`partition_desc`で宣言されたパーティション列のシーケンスは、`column_definition`で定義された列のシーケンスと一致する必要があります。

#### PROPERTIES

`properties`で`"key" = "value"`形式でテーブル属性を指定できます。

次の表に、いくつかの主要なプロパティを説明します。

| **プロパティ**      | **説明**                                              |
| ----------------- | ------------------------------------------------------------ |
| location          | 管理テーブルを作成するファイルパスを指定します。HMSをメタストアとして使用する場合、`location`パラメータを指定する必要はありません。なぜなら、StarRocksはテーブルを現在のHiveカタログのデフォルトファイルパスに作成するからです。AWS Glueをメタデータサービスとして使用する場合：<ul><li>テーブルを作成したいデータベースの`location`パラメータを指定している場合、テーブルの`location`パラメータを指定する必要はありません。その場合、テーブルは所属するデータベースのファイルパスにデフォルトで設定されます。</li><li>テーブルを作成したいデータベースの`location`を指定していない場合、テーブルの`location`パラメータを指定する必要があります。</li></ul> |
| file_format       | 管理テーブルのファイルフォーマットです。Parquetフォーマットのみがサポートされています。デフォルト値は`parquet`です。 |
| compression_codec | 管理テーブルで使用する圧縮アルゴリズムです。サポートされている圧縮アルゴリズムはSNAPPY、GZIP、ZSTD、LZ4です。デフォルト値は`gzip`です。 |

### 例

1. `unpartition_tbl`という名前のパーティションされていないテーブルを作成します。テーブルは次のような2つの列、`id`と`score`から構成されています：

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. `partition_tbl_1`という名前のパーティションされたテーブルを作成します。テーブルは次のような3つの列、`action`、`id`、`dt`から構成されており、`id`と`dt`がパーティション列として定義されています：

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. `partition_tbl_1`という名前の既存のテーブルをクエリし、そのクエリ結果を基に`partition_tbl_2`という名前のパーティションされたテーブルを作成します。`partition_tbl_2`では、`id`と`dt`がパーティション列として定義されています：

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (k1, k2)
   AS SELECT * from partition_tbl_1;
   ```

## Hiveテーブルにデータをシンクする

StarRocksの内部テーブルと同様に、Hiveテーブル（管理テーブルまたは外部テーブル）に[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ステートメントを使用してStarRocksテーブルのデータをシンクすることができます（現在はParquet形式のHiveテーブルのみサポートされています）。この機能はv3.2以降でサポートされています。外部テーブルへのデータのシンクはデフォルトで無効になっています。外部テーブルへのデータのシンクを有効にするには、[システム変数`ENABLE_WRITE_HIVE_EXTERNAL_TABLE`](../../reference/System_variable.md)を`true`に設定する必要があります。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Hiveカタログとそのデータベースに切り替える](#switch-to-a-hive-catalog-and-a-database-in-it)し、次の構文を使用してStarRocksテーブルのデータをParquet形式のHiveテーブルにシンクします。

### 構文

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- データを指定したパーティションにシンクする場合は、次の構文を使用します：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

> **注意**
>
> パーティション列には`NULL`値を指定できません。したがって、Hiveテーブルのパーティション列に空の値がロードされないようにする必要があります。

### パラメータ

| パラメータ   | 説明                                                        |
| ----------- | ------------------------------------------------------------ |
| INTO        | StarRocksテーブルのデータをHiveテーブルに追加します。               |
| OVERWRITE   | StarRocksテーブルのデータでHiveテーブルの既存データを上書きします。 |
| column_name | データをロードする宛先の列の名前を指定します。1つ以上の列を指定できます。複数の列を指定する場合は、カンマ（`,`）で区切ります。指定する列は実際にHiveテーブルに存在する列のみ指定でき、指定する宛先列にはHiveテーブルのパーティション列を含める必要があります。指定した宛先列は、StarRocksテーブルの列とは関係なく、順番に1対1でマッピングされます。宛先列が指定されていない場合、データはHiveテーブルのすべての列にロードされます。StarRocksテーブルの非パーティション列がHiveテーブルのどの列にもマッピングできない場合、StarRocksはHiveテーブルの列にデフォルト値`NULL`を書き込みます。INSERTステートメントには、返される列の型が宛先列のデータ型と異なるクエリステートメントが含まれている場合、StarRocksは不一致の列に対して暗黙の型変換を実行します。変換に失敗した場合、構文解析エラーが返されます。 |
| expression  | 宛先列に値を割り当てる式です。                                  |
| DEFAULT     | 宛先列にデフォルト値を割り当てます。                             |
| query       | Hiveテーブルにロードされるクエリステートメントの結果です。StarRocksでサポートされている任意のSQLステートメントを使用できます。 |
| PARTITION   | データをロードするパーティションです。このプロパティにはHiveテーブルのすべてのパーティション列を指定する必要があります。このプロパティで指定するパーティション列は、テーブル作成ステートメントで定義したパーティション列とは異なる順序で指定することができます。このプロパティを指定する場合、`column_name`プロパティを指定することはできません。 |

### 例

1. `partition_tbl_1`テーブルに3つのデータ行を挿入します：

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. 単純な計算を含むSELECTクエリの結果を`partition_tbl_1`テーブルに挿入します：

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. `partition_tbl_1`テーブルからデータを読み取り、同じテーブルに挿入します：

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. `partition_tbl_2`テーブルの`dt='2023-09-01'`および`id=1`の条件に一致するパーティションにSELECTクエリの結果を挿入します：

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT INTO partition_tbl_2 partition(dt='2023-09-01',id=1) SELECT 'order';
   ```

5. `partition_tbl_1`テーブルの`dt='2023-09-01'`および`id=1`の条件に一致するパーティションの`action`列の値をすべて`close`に上書きします：

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT OVERWRITE partition_tbl_1 partition(dt='2023-09-01',id=1) SELECT 'close';
   ```

## Hiveテーブルを削除する

StarRocksでは、Hiveの管理テーブルを削除するために[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)ステートメントを使用することができます。この機能はv3.1以降でサポートされています。現在、StarRocksはHiveの管理テーブルのみを削除することができます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

Hiveテーブルを削除する場合は、`DROP TABLE`ステートメントで`FORCE`キーワードを指定する必要があります。操作が完了すると、テーブルのファイルパスは保持されますが、テーブルのデータはすべて削除されます。この操作を使用してHiveテーブルを削除する際は注意してください。

[Hiveカタログとそのデータベースに切り替える](#switch-to-a-hive-catalog-and-a-database-in-it)し、次のステートメントを使用してHiveテーブルを削除します。

```SQL
DROP TABLE <table_name> FORCE
```

## メタデータキャッシュの手動または自動更新

### 手動更新

デフォルトでは、StarRocksはHiveのメタデータをキャッシュし、非同期モードでメタデータを更新してパフォーマンスを向上させます。また、スキーマの変更やテーブルの更新がHiveテーブルで行われた場合、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)を使用してメタデータを手動で更新し、StarRocksが最新のメタデータをできるだけ早く取得し、適切な実行計画を生成できるようにすることもできます。

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

以下の状況でメタデータを手動で更新する必要があります：

- 既存のパーティション内のデータファイルが変更された場合、たとえば`INSERT OVERWRITE ... PARTITION ...`コマンドを実行することで。
- Hiveテーブルでスキーマの変更が行われた場合。
- DROPステートメントを使用して既存のHiveテーブルを削除し、削除したHiveテーブルと同じ名前の新しいHiveテーブルを作成した場合。
- Hiveカタログの作成時に`PROPERTIES`で`"enable_cache_list_names" = "true"`を指定し、Hiveクラスタで作成したばかりの新しいパーティションをクエリしたい場合。

  > **注意**
  >
  > v2.5.5以降、StarRocksは定期的なHiveメタデータキャッシュの更新機能を提供しています。詳細については、以下のトピックの"[定期的なメタデータキャッシュの更新](#periodically-refresh-metadata-cache)"セクションを参照してください。この機能を有効にすると、StarRocksはデフォルトで10分ごとにHiveメタデータキャッシュを更新します。したがって、ほとんどの場合は手動で更新する必要はありません。Hiveクラスタで新しいパーティションを作成した直後に新しいパーティションをクエリしたい場合にのみ手動で更新する必要があります。

HiveテーブルのテーブルとパーティションのメタデータのみがFEにキャッシュされます。

### 自動増分更新

自動増分更新は、StarRocksクラスタのFEがHiveメタストアからイベント（列の追加、パーティションの削除、データの更新など）を読み取り、これらのイベントに基づいてキャッシュされたメタデータを自動的に更新する機能です。これにより、メタデータの手動更新は不要になります。

この機能はHiveメタストアのイベントリスナーを設定することで有効にすることができます。

#### ステップ1：Hiveメタストアにイベントリスナーを設定する

Hiveメタストアv2.xおよびv3.xの両方でイベントリスナーを設定することができます。ここでは、Hiveメタストアv3.1.2で使用されるイベントリスナーの設定例を示します。次の設定項目を**$HiveMetastore/conf/hive-site.xml**ファイルに追加し、Hiveメタストアを再起動します。

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

#### ステップ2：StarRocksで自動増分更新を有効にする

StarRocksの単一のHiveカタログまたはすべてのHiveカタログに対して自動増分更新を有効にすることができます。

- 単一のHiveカタログに対して自動増分更新を有効にするには、Hiveカタログを作成する際に`enable_hms_events_incremental_sync`パラメータを`true`に設定します。以下のように指定します：

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

- すべてのHiveカタログに対して自動増分更新を有効にするには、各FEの`$FE_HOME/conf/fe.conf`ファイルに`"enable_hms_events_incremental_sync" = "true"`を追加し、各FEを再起動してパラメータの設定を有効にします。

また、各FEの`$FE_HOME/conf/fe.conf`ファイルで次のパラメータを調整し、ビジネス要件に合わせてチューニングすることもできます。各FEを再起動してパラメータの設定を有効にします。

| パラメータ                         | 説明                                                        |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocksがHiveメタストアからイベントを読み取る間隔です。デフォルト値は`5000`です。単位はミリ秒です。 |
| hms_events_batch_size_per_rpc     | StarRocksが一度に読み取ることができるイベントの最大数です。デフォルト値は`500`です。 |
| enable_hms_parallel_process_evens | StarRocksがイベントを読み取りながら並列で処理するかどうかを指定します。有効な値は`true`と`false`です。デフォルト値は`true`です。値が`true`の場合、並列処理が有効になり、値が`false`の場合、並列処理が無効になります。 |
| hms_process_events_parallel_num   | StarRocksが並列で処理できるイベントの最大数です。デフォルト値は`4`です。 |

## 付録：メタデータの自動非同期更新の理解

自動非同期更新は、StarRocksがHiveカタログのメタデータを更新するために使用するデフォルトのポリシーです。

デフォルトでは（つまり、`enable_metastore_cache`パラメータと`enable_remote_file_cache`パラメータの両方が`true`に設定されている場合）、クエリがHiveテーブルのパーティションにヒットすると、StarRocksは自動的にそのパーティションのメタデータとパーティションの基になるデータファイルのメタデータをキャッシュします。キャッシュされたメタデータは遅延更新ポリシーを使用して更新されます。

たとえば、`table2`という名前のHiveテーブルがあり、`p1`、`p2`、`p3`、`p4`の4つのパーティションがあります。クエリが`p1`にヒットすると、StarRocksは`p1`のメタデータと`p1`の基になるデータファイルのメタデータをキャッシュします。次のようなデフォルトの更新および破棄のタイムインターバルがあるとします：

- `p1`のキャッシュされたメタデータを非同期に更新するための時間間隔（`metastore_cache_refresh_interval_sec`パラメータで指定）は2時間です。
- `p1`の基になるデータファイルのキャッシュされたメタデータを非同期に更新するための時間間隔（`remote_file_cache_refresh_interval_sec`パラメータで指定）は60秒です。
- `p1`のキャッシュされたメタデータを自動的に破棄するための時間間隔（`metastore_cache_ttl_sec`パラメータで指定）は24時間です。
- `p1`の基になるデータファイルのキャッシュされたメタデータを自動的に破棄するための時間間隔（`remote_file_cache_ttl_sec`パラメータで指定）は36時間です。

以下の図は、タイムライン上の時間間隔をわかりやすくするためのものです。

![キャッシュされたメタデータの更新と破棄のタイムライン](../../assets/catalog_timeline.png)

その後、StarRocksは次のルールに従ってメタデータを更新または破棄します。

- もう一度`p1`にヒットするクエリがあり、最後の更新からの経過時間が60秒未満の場合、StarRocksは`p1`のキャッシュされたメタデータまたは`p1`の基になるデータファイルのキャッシュされたメタデータを更新しません。
- もう一度`p1`にヒットするクエリがあり、最後の更新からの経過時間が60秒を超える場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを更新します。
- もう一度`p1`にヒットするクエリがあり、最後の更新からの経過時間が2時間を超える場合、StarRocksは`p1`のキャッシュされたメタデータを更新します。
- 最後の更新から24時間以上経過し、`p1`がアクセスされていない場合、StarRocksは`p1`のキャッシュされたメタデータを破棄します。メタデータは次のクエリでキャッシュされます。
- 最後の更新から36時間以上経過し、`p1`がアクセスされていない場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを破棄します。メタデータは次のクエリでキャッシュされます。
