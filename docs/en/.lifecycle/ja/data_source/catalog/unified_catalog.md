---
displayed_sidebar: "Japanese"
---

# 統合カタログ

統合カタログは、StarRocks v3.2以降で提供される外部カタログの一種であり、Apache Hive™、Apache Iceberg、Apache Hudi、Delta Lakeなどのさまざまなデータソースのテーブルを、統合データソースとして取り扱うためのものです。統合カタログを使用すると、次のことができます。

- 手動でテーブルを作成する必要なく、Hive、Iceberg、Hudi、Delta Lakeに格納されたデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)や非同期マテリアライズドビュー（v2.5以降でサポート）を使用して、Hive、Iceberg、Hudi、Delta Lakeに格納されたデータを処理し、StarRocksにデータをロードできます。
- StarRocksでHiveおよびIcebergのデータベースとテーブルを作成または削除する操作を実行できます。

統合データソース上でのSQLワークロードの成功を保証するためには、StarRocksクラスタは2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCSなどのオブジェクトストレージなどのS3互換ストレージシステム（たとえば、MinIOなど）

- HiveメタストアまたはAWS Glueなどのメタストア

  > **注記**
  >
  > ストレージとしてAWS S3を選択した場合、メタストアとしてHMSまたはAWS Glueを使用できます。他のストレージシステムを選択した場合、メタストアとしてHMSのみを使用できます。

## 制限事項

1つの統合カタログは、1つのストレージシステムと1つのメタストアサービスとの統合のみをサポートしています。したがって、StarRocksと統合する統合データソースとして統合したいすべてのデータソースが、同じストレージシステムとメタストアサービスを使用していることを確認してください。

## 使用上の注意

- [Hiveカタログ](../../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../../data_source/catalog/hudi_catalog.md)、[Delta Lakeカタログ](../../data_source/catalog/deltalake_catalog.md)の「使用上の注意」セクションを参照して、サポートされているファイル形式とデータ型を理解してください。

- フォーマット固有の操作は、特定のテーブル形式のみでサポートされています。たとえば、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)および[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)は、HiveとIcebergのみでサポートされており、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)は、HiveとHudiのみでサポートされています。

  [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントを使用して統合カタログ内にテーブルを作成する場合は、`ENGINE`パラメータを使用してテーブル形式（HiveまたはIceberg）を指定してください。

## 統合の準備

統合カタログを作成する前に、StarRocksクラスタが統合データソースのストレージシステムとメタストアと統合できることを確認してください。

### AWS IAM

ストレージとしてAWS S3またはメタストアとしてAWS Glueを使用する場合は、適切な認証方法を選択し、関連するAWSクラウドリソースにStarRocksクラスタがアクセスできるように必要な準備を行ってください。詳細については、[AWSリソースへの認証 - 準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとしてHDFSを選択した場合は、StarRocksクラスタを次のように構成してください。

- （オプション）HDFSクラスタとHiveメタストアにアクセスするために使用されるユーザ名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザ名を使用してHDFSクラスタとHiveメタストアにアクセスします。各FEの**fe/conf/hadoop_env.sh**ファイルと各BEの**be/conf/hadoop_env.sh**ファイルの先頭に`export HADOOP_USER_NAME="<user_name>"`を追加してユーザ名を設定できます。これらのファイルでユーザ名を設定した後、各FEと各BEを再起動してパラメータ設定が有効になるようにしてください。1つのStarRocksクラスタには1つのユーザ名のみを設定できます。
- データをクエリするとき、StarRocksクラスタのFEとBEはHDFSクラスタにアクセスするためにHDFSクライアントを使用します。ほとんどの場合、この目的を達成するためにStarRocksクラスタを構成する必要はありません。StarRocksはデフォルトの設定でHDFSクライアントを起動します。StarRocksクラスタを構成する必要があるのは、次の場合だけです。
  - HDFSクラスタで高可用性（HA）が有効になっている場合：HDFSクラスタの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加してください。
  - HDFSクラスタでView File System（ViewFs）が有効になっている場合：HDFSクラスタの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加してください。

> **注記**
>
> クエリを送信する際に不明なホストを示すエラーが返される場合は、HDFSクラスタノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに追加する必要があります。

### Kerberos認証

HDFSクラスタまたはHiveメタストアでKerberos認証が有効になっている場合は、StarRocksクラスタを次のように構成してください。

- 各FEと各BEで`kinit -kt keytab_path principal`コマンドを実行して、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFSクラスタとHiveメタストアにアクセスする権限が必要です。このコマンドを使用してKDCにアクセスするのは時間的に制約があるため、cronを使用して定期的にこのコマンドを実行する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルと各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加してください。この例では、`/etc/krb5.conf`はkrb5.confファイルの保存パスです。必要に応じてパスを変更してください。

## 統合カタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "unified",
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### パラメータ

#### catalog_name

統合カタログの名前です。命名規則は次のとおりです。

- 名前には、文字、数字（0-9）、アンダースコア（_）を含めることができます。ただし、文字で始める必要があります。
- 名前は大文字と小文字を区別し、長さが1023文字を超えることはできません。

#### comment

統合カタログの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。値を`unified`に設定します。

#### MetastoreParams

StarRocksがメタストアと統合する方法に関するパラメータのセットです。

##### Hiveメタストア

統合データソースのメタストアとしてHiveメタストアを選択した場合、`MetastoreParams`を次のように構成してください。

```SQL
"unified.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> データをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに追加する必要があります。そうしないと、クエリを開始するときにStarRocksがHiveメタストアにアクセスできなくなる場合があります。

次の表に、`MetastoreParams`で構成する必要のあるパラメータについて説明します。

| パラメータ                   | 必須 | 説明                                                         |
| ---------------------------- | ---- | ------------------------------------------------------------ |
| unified.metastore.type       | Yes  | 統合データソースで使用するメタストアのタイプです。値を`hive`に設定します。 |
| hive.metastore.uris          | Yes  | HiveメタストアのURIです。形式: `thrift://<metastore_IP_address>:<metastore_port>`。Hiveメタストアで高可用性（HA）が有効になっている場合は、複数のメタストアURIを指定し、カンマ（`,`）で区切って指定します。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`と指定します。 |

##### AWS Glue

データソースのメタストアとしてAWS Glueを選択した場合（ストレージとしてAWS S3を選択した場合のみサポートされています）、次のいずれかの操作を実行してください。

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams`を次のように構成してください。

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- アサームドロールベースの認証方法を選択する場合、`MetastoreParams`を次のように構成してください。

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAMユーザベースの認証方法を選択する場合、`MetastoreParams`を次のように構成してください。

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

次の表に、`MetastoreParams`で構成する必要のあるパラメータについて説明します。

| パラメータ                     | 必須 | 説明                                                         |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| unified.metastore.type        | Yes  | 統合データソースで使用するメタストアのタイプです。値を`glue`に設定します。 |
| aws.glue.use_instance_profile | Yes  | インスタンスプロファイルベースの認証方法とアサームドロールベースの認証を有効にするかどうかを指定します。有効な値は`true`および`false`です。デフォルト値は`false`です。 |
| aws.glue.iam_role_arn         | No   | AWS Glueデータカタログに特権を持つIAMロールのARNです。AWS Glueへのアクセスにアサームドロールベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.region               | Yes  | AWS Glueデータカタログが存在するリージョンです。例: `us-west-1`。 |
| aws.glue.access_key           | No   | AWS IAMユーザのアクセスキーです。AWS S3へのアクセスにIAMユーザベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | No   | AWS IAMユーザのシークレットキーです。AWS S3へのアクセスにIAMユーザベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |

AWS Glueへのアクセス方法の選択およびAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS Glueへのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocksがストレージシステムと統合する方法に関するパラメータのセットです。このパラメータセットはオプションです。

ストレージとしてHDFSを使用する場合、`StorageCredentialParams`を構成する必要はありません。

ストレージとしてAWS S3、その他のS3互換ストレージシステム、Microsoft Azure Storage、またはGoogle GCSを使用する場合、`StorageCredentialParams`を構成する必要があります。

##### AWS S3

ストレージとしてAWS S3を選択した場合、次のいずれかの操作を実行してください。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- アサームドロールベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

| パラメータ                   | 必須 | 説明                                                         |
| --------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes  | インスタンスプロファイルベースの認証方法とアサームドロールベースの認証方法を有効にするかどうかを指定します。有効な値は`true`および`false`です。デフォルト値は`false`です。 |
| aws.s3.iam_role_arn         | No   | AWS S3バケットに特権を持つIAMロールのARNです。AWS S3へのアクセスにアサームドロールベースの認証方法を使用する場合は、このパラメータを指定する必要があります。  |
| aws.s3.region               | Yes  | AWS S3バケットが存在するリージョンです。例: `us-west-1`。 |
| aws.s3.access_key           | No   | IAMユーザのアクセスキーです。AWS S3へのアクセスにIAMユーザベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | No   | IAMユーザのシークレットキーです。AWS S3へのアクセスにIAMユーザベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |

AWS S3へのアクセス方法の選択およびAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS S3へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

S3互換ストレージシステム（たとえば、MinIOなど）をストレージとして選択した場合、次のように`StorageCredentialParams`を構成して、統合が成功するようにしてください。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

| パラメータ                        | 必須 | 説明                                                         |
| -------------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | Yes  | SSL接続を有効にするかどうかを指定します。<br />有効な値は`true`および`false`です。デフォルト値は`true`です。 |
| aws.s3.enable_path_style_access  | Yes  | パススタイルアクセスを有効にするかどうかを指定します。<br />有効な値は`true`および`false`です。デフォルト値は`false`です。MinIOの場合、この値を`true`に設定する必要があります。<br />パススタイルURLは次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、US West（Oregon）リージョンで`DOC-EXAMPLE-BUCKET1`というバケットを作成し、そのバケットの`alice.jpg`オブジェクトにアクセスする場合、次のパススタイルURLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | Yes  | AWS S3の代わりにS3互換ストレージシステムに接続するために使用されるエンドポイントです。 |
| aws.s3.access_key                | Yes  | IAMユーザのアクセスキーです。                               |
| aws.s3.secret_key                | Yes  | IAMユーザのシークレットキーです。                             |

##### Microsoft Azure Storage

###### Azure Blob Storage

ストレージとしてBlob Storageを選択した場合、次のいずれかの操作を実行してください。

- 共有キー認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ**              | **必須** | **説明**                                                     |
  | -------------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.storage_account | Yes      | Blob Storageアカウントのユーザ名です。                        |
  | azure.blob.shared_key      | Yes      | Blob Storageアカウントの共有キーです。                        |

- SASトークン認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ**             | **必須** | **説明**                                                     |
  | ------------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.account_name   | Yes      | Blob Storageアカウントのユーザ名です。                        |
  | azure.blob.container_name | Yes      | データを格納するBlobコンテナの名前です。                      |
  | azure.blob.sas_token      | Yes      | Blob Storageアカウントにアクセスするために使用されるSASトークンです。 |

###### Azure Data Lake Storage Gen1

ストレージとしてData Lake Storage Gen1を選択した場合、次のいずれかの操作を実行してください。

- マネージドサービスアイデンティティ認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ**                            | **必須** | **説明**                                                     |
  | ---------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes      | マネージドサービスアイデンティティ認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |

- サービスプリンシパル認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ**                 | **必須** | **説明**                                                     |
  | ----------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | Yes      | サービスプリンシパルのクライアント（アプリケーション）IDです。 |
  | azure.adls1.oauth2_credential | Yes      | 作成時に生成された新しいクライアント（アプリケーション）シークレットの値です。 |
  | azure.adls1.oauth2_endpoint   | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

###### Azure Data Lake Storage Gen2

ストレージとしてData Lake Storage Gen2を選択した場合、次のいずれかの操作を実行してください。

- マネージドアイデンティティ認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ**                           | **必須** | **説明**                                                     |
  | --------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes      | マネージドアイデンティティ認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | Yes      | アクセスするデータのテナントのIDです。                        |
  | azure.adls2.oauth2_client_id            | Yes      | マネージドアイデンティティのクライアント（アプリケーション）IDです。 |

- 共有キー認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ**               | **必須** | **説明**                                                     |
  | --------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.storage_account | Yes      | Data Lake Storage Gen2ストレージアカウントのユーザ名です。    |
  | azure.adls2.shared_key      | Yes      | Data Lake Storage Gen2ストレージアカウントの共有キーです。    |

- サービスプリンシパル認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ**                      | **必須** | **説明**                                                     |
  | ---------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | Yes      | サービスプリンシパルのクライアント（アプリケーション）IDです。 |
  | azure.adls2.oauth2_client_secret   | Yes      | 作成時に生成された新しいクライアント（アプリケーション）シークレットの値です。 |
  | azure.adls2.oauth2_client_endpoint | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

##### Google GCS

ストレージとしてGoogle GCSを選択した場合、次のいずれかの操作を実行してください。

- VMベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                                     |
  | ------------------------------------------ | ---------------- | ----------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false            | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ**                          | **デフォルト値** | **値の例**                                                  | **説明**                                                     |
  | -------------------------------------- | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""               | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントのJSONファイル作成時に生成されたメールアドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウントのJSONファイル作成時に生成されたプライベートキーIDです。 |
  | gcp.gcs.service_account_private_key    | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウントのJSONファイル作成時に生成されたプライベートキーです。 |

- 偽装ベースの認証方法を選択する場合、次のいずれかの操作を実行してください。

  - VMインスタンスがサービスアカウントを偽装するようにする場合：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

    | **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                                     |
    | ------------------------------------------ | ---------------- | ----------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false            | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""               | "hello"     | 偽装したいサービスアカウントです。                            |

  - サービスアカウント（一時的にメタサービスアカウントと呼ばれる）が別のサービスアカウント（一時的にデータサービスアカウントと呼ばれる）を偽装するようにする場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

    | **パラメータ**                          | **デフォルト値** | **値の例**                                                  | **説明**                                                     |
    | -------------------------------------- | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""               | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントのJSONファイル作成時に生成されたメールアドレスです。 |
    | gcp.gcs.service_account_private_key_id | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウントのJSONファイル作成時に生成されたプライベートキーIDです。 |
    | gcp.gcs.service_account_private_key    | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウントのJSONファイル作成時に生成されたプライベートキーです。 |
#### MetadataUpdateParams

Hive、Hudi、Delta Lakeのキャッシュメタデータを更新するためのパラメータのセットです。このパラメータセットはオプションです。Hive、Hudi、Delta Lakeのキャッシュメタデータの更新ポリシーの詳細については、[Hiveカタログ](../../data_source/catalog/hive_catalog.md)、[Hudiカタログ](../../data_source/catalog/hudi_catalog.md)、[Delta Lakeカタログ](../../data_source/catalog/deltalake_catalog.md)を参照してください。

ほとんどの場合、`MetadataUpdateParams`は無視して、パラメータを調整する必要はありません。なぜなら、これらのパラメータのデフォルト値は、すでに使いやすいパフォーマンスを提供しているからです。

ただし、Hive、Hudi、またはDelta Lakeのデータ更新の頻度が高い場合は、これらのパラメータを調整して、自動非同期更新のパフォーマンスをさらに最適化することができます。

| パラメータ名                            | 必須     | 説明                                                         |
| -------------------------------------- | -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | いいえ   | StarRocksがHive、Hudi、またはDelta Lakeのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。値 `true` はキャッシュを有効にし、値 `false` はキャッシュを無効にします。 |
| enable_remote_file_cache               | いいえ   | StarRocksがHive、Hudi、またはDelta Lakeのテーブルまたはパーティションの基礎データファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。値 `true` はキャッシュを有効にし、値 `false` はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | いいえ   | StarRocksが自身でキャッシュしているHive、Hudi、またはDelta Lakeのテーブルまたはパーティションのメタデータを非同期に更新する間隔です。単位: 秒。デフォルト値: `7200` (2時間)。 |
| remote_file_cache_refresh_interval_sec | いいえ   | StarRocksが自身でキャッシュしているHive、Hudi、またはDelta Lakeのテーブルまたはパーティションの基礎データファイルのメタデータを非同期に更新する間隔です。単位: 秒。デフォルト値: `60`。 |
| metastore_cache_ttl_sec                | いいえ   | StarRocksが自動的に破棄するHive、Hudi、またはDelta Lakeのテーブルまたはパーティションのメタデータの間隔です。単位: 秒。デフォルト値: `86400` (24時間)。 |
| remote_file_cache_ttl_sec              | いいえ   | StarRocksが自動的に破棄するHive、Hudi、またはDelta Lakeのテーブルまたはパーティションの基礎データファイルのメタデータの間隔です。単位: 秒。デフォルト値: `129600` (36時間)。 |

### 例

以下の例では、統一されたデータソースからデータをクエリするために、`unified_catalog_hms`または`unified_catalog_glue`という名前の統一カタログを作成します。統一カタログのタイプによって、`unified_catalog_hms`または`unified_catalog_glue`が作成されます。

#### HDFS

ストレージとしてHDFSを使用する場合、次のようなコマンドを実行します:

```SQL
CREATE EXTERNAL CATALOG unified_catalog_hms
PROPERTIES
(
    "type" = "unified",
    "unified.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
);
```

#### AWS S3

##### インスタンスプロファイルベースの認証

- Hiveメタストアを使用する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMRでAWS Glueを使用する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_glue
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 偽装ベースの認証

- Hiveメタストアを使用する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMRでAWS Glueを使用する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_glue
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### IAMユーザーベースの認証

- Hiveメタストアを使用する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMRでAWS Glueを使用する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_glue
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "glue",
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

MinIOを例に使用します。次のようなコマンドを実行します:

```SQL
CREATE EXTERNAL CATALOG unified_catalog_hms
PROPERTIES
(
    "type" = "unified",
    "unified.metastore.type" = "hive",
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

- 共有キー認証メソッドを選択する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SASトークン認証メソッドを選択する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- マネージドサービスアイデンティティ認証メソッドを選択する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- サービスプリンシパル認証メソッドを選択する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- マネージドアイデンティティ認証メソッドを選択する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 共有キー認証メソッドを選択する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- サービスプリンシパル認証メソッドを選択する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMベースの認証メソッドを選択する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- サービスアカウントベースの認証メソッドを選択する場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 偽装ベースの認証メソッドを選択する場合:

  - VMインスタンスがサービスアカウントを偽装する場合、次のようなコマンドを実行します:

    ```SQL
    CREATE EXTERNAL CATALOG unified_catalog_hms
    PROPERTIES
    (
        "type" = "unified",
        "unified.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - サービスアカウントが別のサービスアカウントを偽装する場合、次のようなコマンドを実行します:

    ```SQL
    CREATE EXTERNAL CATALOG unified_catalog_hms
    PROPERTIES
    (
        "type" = "unified",
        "unified.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## 統一カタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスタ内のすべてのカタログをクエリできます:

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリできます。次の例では、`unified_catalog_glue`という名前の統一カタログの作成ステートメントをクエリしています:

```SQL
SHOW CREATE CATALOG unified_catalog_glue;
```

## 統一カタログとその中のデータベースに切り替える

統一カタログとその中のデータベースに切り替えるには、次のいずれかの方法を使用できます:

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションで統一カタログを指定し、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します:

  ```SQL
  -- 現在のセッションで指定したカタログに切り替える:
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定する:
  USE <db_name>
  ```

- 直接に[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、統一カタログとその中のデータベースに切り替えます:

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 統一カタログを削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、外部カタログを削除できます。

次の例では、`unified_catalog_glue`という名前の統一カタログを削除しています:

```SQL
DROP CATALOG unified_catalog_glue;
```

## 統一カタログのテーブルのスキーマを表示する

統一カタログのテーブルのスキーマを表示するには、次の構文のいずれかを使用できます:

- スキーマを表示する

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## 統一カタログからデータをクエリする

統一カタログからデータをクエリするには、次の手順に従います:

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、統一データソースに関連付けられた統一カタログのデータベースを表示します:

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [統一カタログとその中のデータベースに切り替えます](#統一カタログとその中のデータベースに切り替える)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、指定したデータベースの宛先テーブルをクエリします:

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hive、Iceberg、Hudi、またはDelta Lakeからデータをロードする

[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用して、Hive、Iceberg、Hudi、またはDelta Lakeテーブルのデータを統一カタログ内のStarRocksテーブルにロードできます。

次の例では、Hiveテーブル`hive_table`のデータを、統一カタログ`unified_catalog`に所属するデータベース`test_database`内に作成されたStarRocksテーブル`test_tbl`にロードしています。

```SQL
INSERT INTO unified_catalog.test_database.test_table SELECT * FROM hive_table
```

## 統一カタログのデータベースを作成する

StarRocksでは、内部カタログと同様に、統一カタログ内のデータベースに[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)ステートメントを使用してデータベースを作成できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksでは、統一カタログにはHiveおよびIcebergデータベースの作成のみがサポートされています。

[統一カタログに切り替えます](#統一カタログとその中のデータベースに切り替える)。次に、次のステートメントを使用してそのデータベース内にデータベースを作成します。

```SQL
CREATE DATABASE <database_name>
[properties ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location`パラメータは、HDFSまたはクラウドストレージ内のファイルパスを指定します。

- データソースのメタストアとしてHiveメタストアを使用する場合、`location`パラメータのデフォルト値は`<warehouse_location>/<database_name.db>`です。データベース作成時にそのパラメータを指定しない場合、Hiveメタストアでサポートされているデフォルト値です。
- データソースのメタストアとしてAWS Glueを使用する場合、`location`パラメータにはデフォルト値がありません。したがって、データベース作成時にそのパラメータを指定する必要があります。

`prefix`は、使用するストレージシステムによって異なります:

| **ストレージシステム**                                     | **`prefix`の値**                                             |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                       | `hdfs`                                                       |
| Google GCS                                                 | `gs`                                                         |
| Azure Blob Storage                                         | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可している場合、`prefix`は`wasb`です。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可している場合、`prefix`は`wasbs`です。</li></ul> |
| Azure Data Lake Storage Gen1                               | `adl`                                                        |
| Azure Data Lake Storage Gen2                               | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可している場合、`prefix`は`abfs`です。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可している場合、`prefix`は`abfss`です。</li></ul> |
| AWS S3またはその他のS3互換ストレージ（たとえばMinIO）     | `s3`                                                         |

## 統一カタログからデータベースを削除する

StarRocksでは、内部データベースと同様に、統一カタログ内のデータベースを[DROP](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md)ステートメントを使用して削除できます。空のデータベースのみを削除できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksでは、統一カタログからHiveおよびIcebergデータベースのみを削除できます。

統一カタログからデータベースを削除すると、データベースのファイルパスはHDFSクラスタまたはクラウドストレージ上で削除されません。

[統一カタログに切り替えます](#統一カタログとその中のデータベースに切り替える)。次に、次のステートメントを使用してそのデータベースを削除します。

```SQL
DROP DATABASE <database_name>
```

## 統一カタログにテーブルを作成する

StarRocksでは、内部データベースと同様に、統一カタログ内のデータベースに[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)または[CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)ステートメントを使用してテーブルを作成できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksでは、統一カタログにはHiveおよびIcebergテーブルの作成のみがサポートされています。

[統一カタログとその中のデータベースに切り替えます](#統一カタログとその中のデータベースに切り替える)。次に、次のステートメントを使用してそのデータベース内にHiveまたはIcebergテーブルを作成します。

```SQL
CREATE TABLE <table_name>
(column_definition1[, column_definition2, ...]
ENGINE = {|hive|iceberg}
[partition_desc]
```

詳細については、[Hiveテーブルの作成](../catalog/hive_catalog.md#hiveテーブルの作成)および[Icebergテーブルの作成](../catalog/iceberg_catalog.md#icebergテーブルの作成)を参照してください。

次の例では、`hive_table`という名前のHiveテーブルを作成しています。テーブルは3つの列`action`、`id`、`dt`で構成されており、`id`と`dt`はパーティション列です。

```SQL
CREATE TABLE hive_table
(
    action varchar(65533),
    id int,
    dt date
)
ENGINE = hive
PARTITION BY (id,dt);
```

## 統一カタログのテーブルにデータをシンクする

StarRocksでは、内部テーブルと同様に、統一カタログ内のテーブルに[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ステートメントを使用してデータをシンクすることができます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksでは、統一カタログにはHiveおよびIcebergテーブルへのデータのシンクのみがサポートされています。

[統一カタログとその中のデータベースに切り替えます](#統一カタログとその中のデータベースに切り替える)。次に、次のステートメントを使用してそのデータベース内のHiveまたはIcebergテーブルにデータを挿入します。

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータをシンクする場合、次の構文を使用します:
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

詳細については、[Hiveテーブルへのデータのシンク](../catalog/hive_catalog.md#hiveテーブルへのデータのシンク)および[Icebergテーブルへのデータのシンク](../catalog/iceberg_catalog.md#icebergテーブルへのデータのシンク)を参照してください。

次の例では、Hiveテーブル`hive_table`に3つのデータ行を挿入しています。

```SQL
INSERT INTO hive_table
VALUES
    ("buy", 1, "2023-09-01"),
    ("sell", 2, "2023-09-02"),
    ("buy", 3, "2023-09-03");
```

## 統一カタログからテーブルを削除する

StarRocksでは、内部テーブルと同様に、統一カタログ内のテーブルを[DROP](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)ステートメントを使用して削除できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、権限を付与および取り消すことができます。

StarRocksは、統合カタログからのみHiveおよびIcebergテーブルの削除をサポートしています。

[Hiveカタログとその中のデータベースに切り替える](#switch-to-a-unified-catalog-and-a-database-in-it)。その後、そのデータベース内のHiveまたはIcebergテーブルを削除するには、[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)を使用します：

```SQL
DROP TABLE <table_name>
```

詳細については、[Hiveテーブルの削除](../catalog/hive_catalog.md#drop-a-hive-table)および[Icebergテーブルの削除](../catalog/iceberg_catalog.md#drop-an-iceberg-table)を参照してください。

以下の例では、`hive_table`という名前のHiveテーブルを削除します：

```SQL
DROP TABLE hive_table FORCE
```
