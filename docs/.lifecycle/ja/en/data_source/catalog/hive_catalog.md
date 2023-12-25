---
displayed_sidebar: English
---

# Hive カタログ

Hiveカタログは、v2.4以降のStarRocksでサポートされている一種の外部カタログです。Hiveカタログ内では、次のことができます。

- Hiveに格納されているデータを、手動でテーブルを作成することなく直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)や非同期マテリアライズドビュー（v2.5以降でサポート）を使用して、Hiveに保存されているデータを処理し、StarRocksにロードします。
- StarRocksで操作を実行してHiveデータベースとテーブルを作成または削除する、または[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用してStarRocksテーブルからParquet形式のHiveテーブルにデータをシンクする（この機能はv3.2以降でサポートされています）。

HiveクラスターでSQLワークロードを成功させるには、StarRocksクラスターが以下の2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはオブジェクトストレージ（AWS S3、Microsoft Azure Storage、Google GCS、またはその他のS3互換ストレージシステム、例えばMinIO）

- メタストア（HiveメタストアやAWS Glueなど）

  > **注記**
  >
  > ストレージとしてAWS S3を選択する場合、メタストアとしてHMSまたはAWS Glueを使用できます。他のストレージシステムを選択する場合は、HMSをメタストアとしてのみ使用できます。

## 使用上の注意

- StarRocksがサポートするHiveのファイル形式は、Parquet、ORC、CSV、Avro、RCFile、およびSequenceFileです。

  - Parquetファイルは、SNAPPY、LZ4、ZSTD、GZIP、およびNO_COMPRESSIONの圧縮形式をサポートしています。v3.1.5以降、ParquetファイルはLZO圧縮形式もサポートしています。
  - ORCファイルは、ZLIB、SNAPPY、LZO、LZ4、ZSTD、およびNO_COMPRESSIONの圧縮形式をサポートしています。
  - CSVファイルは、v3.1.5以降LZO圧縮形式をサポートしています。

- StarRocksがサポートしていないHiveのデータ型は、INTERVAL、BINARY、およびUNIONです。さらに、StarRocksはCSV形式のHiveテーブルのMAPおよびSTRUCTデータ型をサポートしていません。
- Hiveカタログはデータのクエリにのみ使用できます。Hiveクラスターにデータをドロップ、削除、または挿入するためにHiveカタログを使用することはできません。

## 統合の準備

Hiveカタログを作成する前に、StarRocksクラスタがHiveクラスタのストレージシステムおよびメタストアと統合できることを確認してください。

### AWS IAM

HiveクラスターがストレージとしてAWS S3を使用するか、メタストアとしてAWS Glueを使用する場合、適切な認証方法を選択し、必要な準備を行って、StarRocksクラスターが関連するAWSクラウドリソースにアクセスできるようにしてください。

推奨される認証方法は以下の通りです：

- インスタンスプロファイル
- アサムドロール
- IAMユーザー

上記の認証方法の中で、インスタンスプロファイルが最も広く使用されています。

詳細については、[AWS IAMでの認証の準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとしてHDFSを選択する場合、StarRocksクラスタを次のように構成します：

- （オプション）HDFSクラスターとHiveメタストアへのアクセスに使用するユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスターとHiveメタストアにアクセスします。また、各FEの**fe/conf/hadoop_env.sh**ファイルと各BEの**be/conf/hadoop_env.sh**ファイルの先頭に`export HADOOP_USER_NAME="<user_name>"`を追加することでユーザー名を設定することもできます。これらのファイルにユーザー名を設定した後、各FEと各BEを再起動してパラメータ設定を有効にします。StarRocksクラスタごとに設定できるユーザー名は1つだけです。
- Hiveデータをクエリする際、StarRocksクラスタのFEとBEはHDFSクライアントを使用してHDFSクラスタにアクセスします。ほとんどの場合、その目的を達成するためにStarRocksクラスタを設定する必要はありません。StarRocksはデフォルト設定を使用してHDFSクライアントを起動します。StarRocksクラスタを設定する必要があるのは、以下の状況のみです：

  - HDFSクラスタで高可用性（HA）が有効になっている場合：HDFSクラスタの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。
  - View File System（ViewFs）がHDFSクラスタで有効になっている場合：HDFSクラスタの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。

> **注記**
>
> クエリを送信する際に不明なホストを示すエラーが返された場合、HDFSクラスタノードのホスト名とIPアドレスのマッピングを**/etc/hosts**に追加する必要があります。

### Kerberos認証

HDFSクラスタまたはHiveメタストアでKerberos認証が有効になっている場合、StarRocksクラスタを次のように構成します：

- 各FEおよび各BEで`kinit -kt keytab_path principal`コマンドを実行して、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFSクラスターとHiveメタストアへのアクセス権が必要です。このコマンドを使用してKDCにアクセスすることは時間に敏感なため、このコマンドを定期的に実行するためにcronを使用する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルと各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。この例では、`/etc/krb5.conf`は**krb5.conf**ファイルの保存パスです。必要に応じてパスを変更できます。

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

Hiveカタログの名前です。命名規則は以下の通りです：

- 名前には文字、数字（0-9）、およびアンダースコア（_）を含めることができます。文字で始まる必要があります。
- 名前は大文字と小文字が区別され、長さは1023文字を超えることはできません。

#### comment

Hiveカタログの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。値を`hive`に設定します。

#### GeneralParams（一般パラメータ）

一般的なパラメータのセットです。

以下の表は、`GeneralParams`で設定できるパラメータを説明しています。

| パラメータ                  | 必須 | 説明                                                  |
| ------------------------ | ---- | ---------------------------------------------------- |
| enable_recursive_listing | いいえ  | StarRocksがテーブルとそのパーティションから、およびテーブルとそのパーティションの物理的な場所にあるサブディレクトリからデータを読み取るかどうかを指定します。有効な値は`true`と`false`です。デフォルト値は`false`です。`true`はサブディレクトリを再帰的にリストすることを指定し、`false`はサブディレクトリを無視することを指定します。 |

#### MetastoreParams（メタストアパラメータ）

StarRocksがデータソースのメタストアと統合する方法に関する一連のパラメータです。

##### Hiveメタストア

データソースのメタストアとしてHiveメタストアを選択する場合、`MetastoreParams`を次のように構成します：

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> Hiveデータをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを`/etc/hosts`に追加する必要があります。そうしないと、クエリを開始する際にStarRocksがHiveメタストアへのアクセスに失敗する可能性があります。

以下の表は、`MetastoreParams`で設定する必要があるパラメータを説明しています。

| パラメータ              | 必須 | 説明                                                  |

| ------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type | はい      | Hive クラスターで使用するメタストアのタイプです。値を `hive` に設定してください。 |
| hive.metastore.uris | はい      | Hive メタストアの URI。形式: `thrift://<metastore_IP_address>:<metastore_port>`。<br />Hive メタストアで高可用性 (HA) が有効な場合、複数のメタストア URI を指定し、コンマ (`,`) で区切ることができます。例: `"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。|

##### AWS Glue

データソースのメタストアとして AWS Glue を選択する場合（ストレージとして AWS S3 を選択した場合のみサポートされます）、以下のアクションのいずれかを実行してください：

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams` を以下のように設定します：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 想定ロールベースの認証方法を選択する場合、`MetastoreParams` を以下のように設定します：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAM ユーザーベースの認証方法を選択する場合、`MetastoreParams` を以下のように設定します：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

以下の表は、`MetastoreParams` で設定する必要があるパラメータを説明しています。

| パラメーター                     | 必須 | 説明                                                  |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type           | はい      | Hive クラスターで使用するメタストアのタイプです。値を `glue` に設定してください。 |
| aws.glue.use_instance_profile | はい      | インスタンスプロファイルベースの認証方法または想定ロールベースの認証を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.glue.iam_role_arn         | いいえ       | AWS Glue Data Catalog に対する権限を持つ IAM ロールの ARN。想定ロールベースの認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.region               | はい      | AWS Glue Data Catalog が存在するリージョン。例: `us-west-1`。 |
| aws.glue.access_key           | いいえ       | AWS IAM ユーザーのアクセスキー。IAM ユーザーベースの認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | いいえ       | AWS IAM ユーザーのシークレットキー。IAM ユーザーベースの認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |

AWS Glue へのアクセスに使用する認証方法の選択と、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS Glue へのアクセスに関する認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocks がストレージシステムと統合する方法に関するパラメータセットです。このパラメータセットはオプションです。

ストレージとして HDFS を使用する場合、`StorageCredentialParams` を設定する必要はありません。

AWS S3、その他の S3 互換ストレージシステム、Microsoft Azure Storage、または Google GCS をストレージとして使用する場合は、`StorageCredentialParams` を設定する必要があります。

##### AWS S3

Hive クラスターのストレージとして AWS S3 を選択する場合、以下のアクションのいずれかを実行してください：

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 想定ロールベースの認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM ユーザーベースの認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表は、`StorageCredentialParams` で設定する必要があるパラメータを説明しています。

| パラメーター                   | 必須 | 説明                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい      | インスタンスプロファイルベースの認証方法または想定ロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | いいえ       | AWS S3 バケットに対する権限を持つ IAM ロールの ARN。想定ロールベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい      | AWS S3 バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | いいえ       | IAM ユーザーのアクセスキー。IAM ユーザーベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAM ユーザーのシークレットキー。IAM ユーザーベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3 へのアクセスに使用する認証方法の選択と、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS S3 へのアクセスに関する認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

v2.5 以降、Hive カタログは S3互換ストレージシステムをサポートしています。

Hive クラスターのストレージとして S3互換ストレージシステム（例: MinIO）を選択する場合、成功した統合を確保するために `StorageCredentialParams` を以下のように設定してください：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表は、`StorageCredentialParams` で設定する必要があるパラメータを説明しています。

| パラメーター                        | 必須 | 説明                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | はい      | SSL 接続を有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access  | はい      | パススタイルアクセスを有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値: `false`。MinIO を使用する場合、値を `true` に設定する必要があります。<br />パススタイルの URL は次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部 (オレゴン) リージョンに `DOC-EXAMPLE-BUCKET1` という名前のバケットを作成し、そのバケット内の `alice.jpg` オブジェクトにアクセスする場合、次のパススタイルの URL を使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | はい      | AWS S3 ではなく、S3互換ストレージシステムに接続するために使用されるエンドポイントです。 |
| aws.s3.access_key                | はい      | IAM ユーザーのアクセスキーです。 |
| aws.s3.secret_key                | はい      | IAM ユーザーのシークレットキーです。 |

##### Microsoft Azure ストレージ

v3.0 以降、Hive カタログは Microsoft Azure ストレージをサポートしています。

###### Azure Blob Storage

Hive クラスターのストレージとして Blob Storage を選択する場合、以下のアクションのいずれかを実行してください：


- 共有キー認証方式を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` で設定する必要があるパラメーターについて以下の表で説明します。

  | **パラメーター**              | **必須** | **説明**                                              |
  | -------------------------- | ------------ | ---------------------------------------------------- |
  | azure.blob.storage_account | はい          | Blob Storage アカウントのユーザー名です。             |
  | azure.blob.shared_key      | はい          | Blob Storage アカウントの共有キーです。               |

- SASトークン認証方式を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams` で設定する必要があるパラメーターについて以下の表で説明します。

  | **パラメーター**             | **必須** | **説明**                                                            |
  | ------------------------- | ------------ | ------------------------------------------------------------------ |
  | azure.blob.storage_account| はい          | Blob Storage アカウントのユーザー名です。                           |
  | azure.blob.container      | はい          | データを格納するblobコンテナの名前です。                            |
  | azure.blob.sas_token      | はい          | Blob Storage アカウントへのアクセスに使用するSASトークンです。     |

###### Azure Data Lake Storage Gen1

HiveクラスタのストレージとしてData Lake Storage Gen1を選択する場合、以下の操作を行ってください：

- 管理サービスID認証方式を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` で設定する必要があるパラメーターについて以下の表で説明します。

  | **パラメーター**                            | **必須** | **説明**                                                            |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい          | 管理サービスID認証方式を有効にするかどうかを指定します。`true`に設定してください。 |

- サービスプリンシパル認証方式を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams` で設定する必要があるパラメーターについて以下の表で説明します。

  | **パラメーター**                 | **必須** | **説明**                                                            |
  | ----------------------------- | ------------ | ------------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい          | サービスプリンシパルのクライアント（アプリケーション）IDです。       |
  | azure.adls1.oauth2_credential | はい          | 新しく作成されたクライアント（アプリケーション）シークレットの値です。 |
  | azure.adls1.oauth2_endpoint   | はい          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

###### Azure Data Lake Storage Gen2

HiveクラスタのストレージとしてData Lake Storage Gen2を選択する場合、以下の操作を行ってください：

- マネージドアイデンティティ認証方式を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  `StorageCredentialParams` で設定する必要があるパラメーターについて以下の表で説明します。

  | **パラメーター**                           | **必須** | **説明**                                                            |
  | --------------------------------------- | ------------ | ------------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | はい          | マネージドアイデンティティ認証方式を有効にするかどうかを指定します。`true`に設定してください。 |
  | azure.adls2.oauth2_tenant_id            | はい          | アクセスしたいテナントのIDです。                                    |
  | azure.adls2.oauth2_client_id            | はい          | マネージドアイデンティティのクライアント（アプリケーション）IDです。 |

- 共有キー認証方式を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams` で設定する必要があるパラメーターについて以下の表で説明します。

  | **パラメーター**               | **必須** | **説明**                                                            |
  | --------------------------- | ------------ | ------------------------------------------------------------------ |
  | azure.adls2.storage_account | はい          | Data Lake Storage Gen2のストレージアカウントのユーザー名です。       |
  | azure.adls2.shared_key      | はい          | Data Lake Storage Gen2のストレージアカウントの共有キーです。         |

- サービスプリンシパル認証方式を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams` で設定する必要があるパラメーターについて以下の表で説明します。

  | **パラメーター**                      | **必須** | **説明**                                                            |
  | ---------------------------------- | ------------ | ------------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい          | サービスプリンシパルのクライアント（アプリケーション）IDです。       |
  | azure.adls2.oauth2_client_secret   | はい          | 新しく作成されたクライアント（アプリケーション）シークレットの値です。 |
  | azure.adls2.oauth2_client_endpoint | はい          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

##### Google GCS

Hiveカタログはv3.0以降、Google GCSをサポートしています。

HiveクラスタのストレージとしてGoogle GCSを選択する場合、以下の操作を行ってください：

- VMベースの認証方式を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams` で設定する必要があるパラメーターについて以下の表で説明します。

  | **パラメーター**                              | **デフォルト値** | **値の例** | **説明**                                                            |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineに紐づけられたサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証方式を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  `StorageCredentialParams` で設定する必要があるパラメーターについて以下の表で説明します。

  | **パラメーター**                          | **デフォルト値** | **値の例**                                        | **説明**                                                            |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウント作成時に生成されたJSONファイル内のメールアドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウント作成時に生成されたJSONファイル内のプライベートキーIDです。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウント作成時に生成されたJSONファイル内のプライベートキーです。 |

- インパーソネーションベースの認証方式を選択する場合、`StorageCredentialParams` を以下のように設定します：

  - VMインスタンスがサービスアカウントをインパーソネートする場合：
  
    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams` で設定する必要があるパラメーターについて以下の表で説明します。

    | **パラメーター**                              | **デフォルト値** | **値の例** | **説明**                                                            |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineに紐づけられたサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | インパーソネートしたいサービスアカウントです。                       |

  - サービスアカウント（メタサービスアカウントと仮称）が別のサービスアカウント（データサービスアカウントと仮称）をインパーソネートする場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    以下の表は、`StorageCredentialParams`で設定する必要があるパラメーターを説明しています。

    | **パラメーター**                          | **デフォルト値** | **値の例**                                        | **説明**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | [user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com) | メタサービスアカウントの作成時に生成されたJSONファイル内のメールアドレス。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキー。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 偽装したいデータサービスアカウント。       |

#### MetadataUpdateParams

StarRocksがHiveのキャッシュされたメタデータをどのように更新するかについてのパラメーターセットです。このパラメーターセットはオプションです。

StarRocksはデフォルトで[自動非同期更新ポリシー](#appendix-understand-metadata-automatic-asynchronous-update)を実装しています。

ほとんどの場合、`MetadataUpdateParams`を無視して、ポリシーパラメーターを調整する必要はありません。なぜなら、これらのパラメーターのデフォルト値は既に即座に使用できるパフォーマンスを提供しているからです。

ただし、Hiveでのデータ更新の頻度が高い場合は、これらのパラメーターを調整して、自動非同期更新のパフォーマンスをさらに最適化することができます。

> **注記**
>
> ほとんどの場合、Hiveデータが1時間以下の粒度で更新される場合、データ更新の頻度は高いと考えられます。

| パラメーター                              | 必須 | 説明                                                  |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | いいえ       | StarRocksがHiveテーブルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| enable_remote_file_cache               | いいえ       | StarRocksがHiveテーブルまたはパーティションの基底データファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | いいえ       | StarRocksが自身にキャッシュされたHiveテーブルまたはパーティションのメタデータを非同期的に更新する時間間隔。単位: 秒。デフォルト値: `7200`（2時間）。 |
| remote_file_cache_refresh_interval_sec | いいえ       | StarRocksが自身にキャッシュされたHiveテーブルまたはパーティションの基底データファイルのメタデータを非同期的に更新する時間間隔。単位: 秒。デフォルト値: `60`。 |
| metastore_cache_ttl_sec                | いいえ       | StarRocksが自身にキャッシュされたHiveテーブルまたはパーティションのメタデータを自動的に破棄する時間間隔。単位: 秒。デフォルト値: `86400`（24時間）。 |
| remote_file_cache_ttl_sec              | いいえ       | StarRocksが自身にキャッシュされたHiveテーブルまたはパーティションの基底データファイルのメタデータを自動的に破棄する時間間隔。単位: 秒。デフォルト値: `129600`（36時間）。 |
| enable_cache_list_names                | いいえ       | StarRocksがHiveパーティション名をキャッシュするかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |

### 例

次の例は、使用するメタストアの種類に応じて`hive_catalog_hms`または`hive_catalog_glue`という名前のHiveカタログを作成し、Hiveクラスターからデータをクエリする方法を示しています。

#### HDFS

HDFSをストレージとして使用する場合、以下のコマンドを実行します：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

#### AWS S3

##### インスタンスプロファイルに基づく認証

- HiveクラスターでHiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、以下のコマンドを実行します：

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

##### 想定されるロールベースの認証

- HiveクラスターでHiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、以下のコマンドを実行します：

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

- HiveクラスターでHiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、以下のコマンドを実行します：

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

例としてMinIOを使用します。以下のコマンドを実行します：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
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

- 共有キー認証方法を選択した場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```


- SASトークン認証方法を選択した場合、以下のようなコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- マネージドサービスID認証方法を選択した場合、以下のようなコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- サービスプリンシパル認証方法を選択した場合、以下のようなコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- マネージドアイデンティティ認証方法を選択した場合、以下のようなコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 共有キー認証方法を選択した場合、以下のようなコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- サービスプリンシパル認証方法を選択した場合、以下のようなコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMベースの認証方法を選択した場合、以下のようなコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- サービスアカウントベースの認証方法を選択した場合、以下のようなコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- なりすましベースの認証方法を選択した場合：

  - VMインスタンスがサービスアカウントをなりすます場合、以下のようなコマンドを実行します。

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - サービスアカウントが他のサービスアカウントをなりすます場合、以下のようなコマンドを実行します。

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Hiveカタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスター内のすべてのカタログを照会できます。

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントを照会することもできます。次の例では、`hive_catalog_glue`という名前のHiveカタログの作成ステートメントをクエリします。

```SQL
SHOW CREATE CATALOG hive_catalog_glue;
```

## Hiveカタログとその中のデータベースに切り替える

次のいずれかの方法を使用して、Hiveカタログとその中のデータベースに切り替えることができます。

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して現在のセッションでHiveカタログを指定し、その後[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します。

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替えます：
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定します：
  USE <db_name>
  ```

- [USE](../../sql-reference/sql-statements/data-definition/USE.md)を直接使用して、Hiveカタログとその中のデータベースに切り替えます。

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Hiveカタログを削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、外部カタログを削除できます。

次の例では、`hive_catalog_glue`という名前のHiveカタログを削除します。

```SQL
DROP CATALOG hive_catalog_glue;
```

## Hiveテーブルのスキーマを表示する

次のいずれかの構文を使用して、Hiveテーブルのスキーマを表示できます。

- スキーマを表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Hiveテーブルをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Hiveクラスター内のデータベースを表示します。

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Hiveカタログとその中のデータネースに切り替える](#hiveカタログとその中のデータベースに切り替える)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、指定されたデータベース内の対象テーブルをクエリします。

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hiveからデータをロードする

`olap_tbl`という名前のOLAPテーブルがある場合、以下のようにデータを変換してロードできます。

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hive_table
```

## Hiveテーブルとビューに対する権限を付与する

[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)ステートメントを使用して、Hiveカタログ内のすべてのテーブルまたはビューに対する権限を特定のロールに付与できます。

- ロールにHiveカタログ内のすべてのテーブルをクエリする権限を付与します。

  ```SQL
  GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE <role_name>
  ```

- ロールにHiveカタログ内のすべてのビューをクエリする権限を付与します。

  ```SQL
  GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE <role_name>
  ```

たとえば、次のコマンドを使用して`hive_role_table`という名前のロールを作成し、Hiveカタログ`hive_catalog`に切り替えてから、ロール`hive_role_table`にHiveカタログ`hive_catalog`内のすべてのテーブルとビューをクエリする権限を付与します。

```SQL
-- hive_role_tableという名前のロールを作成します。
CREATE ROLE hive_role_table;

-- Hiveカタログhive_catalogに切り替えます。
SET CATALOG hive_catalog;

-- ロールhive_role_tableにHiveカタログhive_catalog内のすべてのテーブルをクエリする権限を付与します。
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE hive_role_table;

-- ロールhive_role_tableにHiveカタログhive_catalog内のすべてのビューをクエリする権限を付与します。
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE hive_role_table;
```

## Hiveデータベースを作成する

StarRocksの内部カタログと同様に、Hiveカタログに対する[CREATE DATABASE](../../administration/privilege_item.md#catalog)権限を持っている場合、[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)ステートメントを使用して、そのHiveカタログにデータベースを作成できます。この機能はv3.2以降でサポートされています。

> **注記**
>
> 特権の付与と取り消しは、[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して行うことができます。

[Hive カタログに切り替え](#switch-to-a-hive-catalog-and-a-database-in-it)てから、次のステートメントを使用して、そのカタログ内に Hive データベースを作成します:

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location` パラメータは、データベースを作成するファイルパスを指定します。これは HDFS またはクラウドストレージのいずれかになります。

- Hive メタストアを Hive クラスターのメタストアとして使用する場合、`location` パラメータはデータベース作成時に指定しない場合、`<warehouse_location>/<database_name.db>` がデフォルト値となります。これは Hive メタストアによってサポートされています。
- AWS Glue を Hive クラスターのメタストアとして使用する場合、`location` パラメータにはデフォルト値がないため、データベース作成時にそのパラメータを指定する必要があります。

`prefix` は使用するストレージシステムに基づいて異なります:

| **ストレージシステム**                                         | **`prefix` の値**                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                       | `hdfs`                                                       |
| Google GCS                                                 | `gs`                                                         |
| Azure Blob Storage                                         | <ul><li>ストレージアカウントが HTTP 経由でのアクセスを許可している場合、`prefix` は `wasb` です。</li><li>ストレージアカウントが HTTPS 経由でのアクセスを許可している場合、`prefix` は `wasbs` です。</li></ul> |
| Azure Data Lake Storage Gen1                               | `adl`                                                        |
| Azure Data Lake Storage Gen2                               | <ul><li>ストレージアカウントが HTTP 経由でのアクセスを許可している場合、`prefix` は `abfs` です。</li><li>ストレージアカウントが HTTPS 経由でのアクセスを許可している場合、`prefix` は `abfss` です。</li></ul> |
| AWS S3 またはその他の S3 互換ストレージ (例: MinIO) | `s3`                                                         |

## Hive データベースを削除する

StarRocks の内部データベースと同様に、Hive データベースに対する [DROP](../../administration/privilege_item.md#database) 権限を持っている場合、[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md) ステートメントを使用してその Hive データベースを削除できます。この機能は v3.2 以降でサポートされています。空のデータベースのみ削除可能です。

> **注記**
>
> 特権の付与と取り消しは、[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して行うことができます。

Hive データベースを削除しても、HDFS クラスターやクラウドストレージ上のデータベースのファイルパスは削除されません。

[Hive カタログに切り替え](#switch-to-a-hive-catalog-and-a-database-in-it)てから、次のステートメントを使用して、そのカタログ内の Hive データベースを削除します:

```SQL
DROP DATABASE <database_name>
```

## Hive テーブルを作成する

StarRocks の内部データベースと同様に、Hive データベースに対する [CREATE TABLE](../../administration/privilege_item.md#database) 権限を持っている場合、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) または [CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) ステートメントを使用して、その Hive データベース内にマネージドテーブルを作成できます。この機能は v3.2 以降でサポートされています。

> **注記**
>
> 特権の付与と取り消しは、[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して行うことができます。

[Hive カタログとその中のデータベースに切り替え](#switch-to-a-hive-catalog-and-a-database-in-it)てから、次の構文を使用して、そのデータベース内に Hive マネージドテーブルを作成します。

### 構文

```SQL
CREATE TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...
partition_column_definition1, partition_column_definition2...])
[partition_desc]
[PROPERTIES ("key" = "value", ...)]
[AS SELECT query]
```

### パラメーター

#### column_definition

`column_definition` の構文は次のとおりです:

```SQL
col_name col_type [COMMENT 'comment']
```

以下の表は、パラメーターを説明しています。

| パラメーター | 説明                                                  |
| --------- | ------------------------------------------------------------ |
| col_name  | 列の名前です。                                      |
| col_type  | 列のデータ型です。サポートされるデータ型には、TINYINT、SMALLINT、INT、BIGINT、FLOAT、DOUBLE、DECIMAL、DATE、DATETIME、CHAR、VARCHAR[(length)]、ARRAY、MAP、STRUCTがあります。LARGEINT、HLL、BITMAP データ型はサポートされていません。 |

> **注意**
>
> パーティション以外のすべての列は `NULL` をデフォルト値として使用する必要があります。これは、テーブル作成ステートメントで非パーティション列ごとに `DEFAULT "NULL"` を指定する必要があることを意味します。また、パーティション列は非パーティション列の後に定義され、`NULL` をデフォルト値として使用することはできません。

#### partition_desc

`partition_desc` の構文は次のとおりです:

```SQL
PARTITION BY (par_col1[, par_col2...])
```

現在 StarRocks は、一意のパーティション値ごとにパーティションを作成するアイデンティティ変換のみをサポートしています。

> **注意**
>
> パーティション列は非パーティション列の後に定義される必要があります。パーティション列は FLOAT、DOUBLE、DECIMAL、DATETIME を除くすべてのデータ型をサポートし、`NULL` をデフォルト値として使用することはできません。さらに、`partition_desc` で宣言されたパーティション列の順序は、`column_definition` で定義された列の順序と一致している必要があります。

#### PROPERTIES

`PROPERTIES` では、テーブル属性を `"key" = "value"` の形式で指定できます。

以下の表は、いくつかの重要なプロパティを説明しています。

| **プロパティ**      | **説明**                                              |
| ----------------- | ------------------------------------------------------------ |
| location          | マネージドテーブルを作成するファイルパスです。HMS をメタストアとして使用する場合、`location` パラメータを指定する必要はありません。StarRocks は現在の Hive カタログのデフォルトファイルパスにテーブルを作成します。AWS Glue をメタデータサービスとして使用する場合:<ul><li>テーブルを作成するデータベースの `location` パラメータを指定している場合、テーブルの `location` パラメータを指定する必要はありません。そのため、テーブルはデフォルトで、それが属するデータベースのファイルパスになります。</li><li>テーブルを作成するデータベースの `location` パラメータを指定していない場合、テーブルの `location` パラメータを指定する必要があります。</li></ul> |
| file_format       | マネージドテーブルのファイル形式です。Parquet 形式のみがサポートされています。デフォルト値は `parquet` です。 |
| compression_codec | マネージドテーブルに使用される圧縮アルゴリズムです。サポートされる圧縮アルゴリズムには、SNAPPY、GZIP、ZSTD、LZ4 があります。デフォルト値は `gzip` です。 |

### 例

1. `unpartition_tbl` という名前の非パーティションテーブルを作成します。このテーブルは、以下に示すように、`id` と `score` の 2 つの列で構成されています。

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. `partition_tbl_1` という名前のパーティションテーブルを作成します。このテーブルは、`action`、`id`、`dt` の 3 つの列で構成され、`id` と `dt` はパーティション列として定義されています。以下に示すように:

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id, dt);
   ```

3. 既存のテーブル `partition_tbl_1` をクエリし、そのクエリ結果に基づいて `partition_tbl_2` という名前のパーティションテーブルを作成します。`partition_tbl_2` では、`id` と `dt` がパーティション列として定義されています。以下に示すように:

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (k1, k2)
   AS SELECT * FROM partition_tbl_1;
   ```

## Hive テーブルへのデータシンク


StarRocksの内部テーブルと同様に、Hiveテーブル（マネージドテーブルまたは外部テーブル）に対する[INSERT](../../administration/privilege_item.md#table)権限がある場合、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ステートメントを使用してStarRocksテーブルのデータをそのHiveテーブルにシンクすることができます（現在はParquet形式のHiveテーブルのみがサポートされています）。この機能はv3.2以降でサポートされています。外部テーブルへのデータシンクはデフォルトでは無効です。外部テーブルへデータをシンクするには、[システム変数 `ENABLE_WRITE_HIVE_EXTERNAL_TABLE`](../../reference/System_variable.md) を `true` に設定する必要があります。

> **注記**
>
> 権限の付与と取り消しは、[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して行うことができます。

[Hiveカタログとその中のデータベースに切り替え](#switch-to-a-hive-catalog-and-a-database-in-it)、次に示す構文を使用して、StarRocksテーブルのデータをそのデータベース内のParquet形式のHiveテーブルにシンクします。

### 構文

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータをシンクしたい場合は、以下の構文を使用します:
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

> **注意**
>
> パーティション列は`NULL`値を許可しません。したがって、Hiveテーブルのパーティション列に空の値がロードされないように注意する必要があります。

### パラメーター

| パラメーター   | 説明                                                  |
| ----------- | ------------------------------------------------------------ |
| INTO        | StarRocksテーブルのデータをHiveテーブルに追加します。 |
| OVERWRITE   | StarRocksテーブルのデータでHiveテーブルの既存データを上書きします。 |
| column_name | データをロードする先の列名。一つ以上の列を指定できます。複数の列を指定する場合は、カンマ(`,`)で区切ります。指定できるのはHiveテーブルに実際に存在する列のみで、指定する宛先列にはHiveテーブルのパーティション列が含まれている必要があります。指定した宛先列は、宛先列名に関係なく、StarRocksテーブルの列に一対一で順番にマッピングされます。宛先列が指定されていない場合、データはHiveテーブルのすべての列にロードされます。StarRocksテーブルの非パーティション列がHiveテーブルのどの列にもマッピングできない場合、StarRocksはデフォルト値`NULL`をHiveテーブルの列に書き込みます。INSERTステートメントに返される列の型が宛先列のデータ型と異なるクエリステートメントが含まれている場合、StarRocksは不一致の列に対して暗黙の型変換を行います。変換が失敗すると、構文解析エラーが返されます。 |
| expression  | 宛先列に値を割り当てる式。    |
| DEFAULT     | 宛先列にデフォルト値を割り当てます。           |
| query       | 結果がHiveテーブルにロードされるクエリステートメント。StarRocksがサポートする任意のSQLステートメントが可能です。 |
| PARTITION   | データをロードするパーティション。このプロパティでHiveテーブルのすべてのパーティション列を指定する必要があります。このプロパティで指定するパーティション列は、テーブル作成ステートメントで定義したパーティション列とは異なる順序であっても構いません。このプロパティを指定する場合、`column_name`プロパティは指定できません。 |

### 例

1. `partition_tbl_1`テーブルに3つのデータ行を挿入します。

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. 単純な計算を含むSELECTクエリの結果を`partition_tbl_1`テーブルに挿入します。

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. `partition_tbl_1`テーブルからデータを読み取るSELECTクエリの結果を同じテーブルに挿入します。

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. `partition_tbl_2`テーブルの`dt='2023-09-01'`と`id=1`の2つの条件を満たすパーティションにSELECTクエリの結果を挿入します。

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT INTO partition_tbl_2 PARTITION(dt='2023-09-01',id=1) SELECT 'order';
   ```

5. `partition_tbl_1`テーブルの`dt='2023-09-01'`と`id=1`の2つの条件を満たすパーティション内のすべての`action`列値を`close`で上書きします。

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT OVERWRITE partition_tbl_1 PARTITION(dt='2023-09-01',id=1) SELECT 'close';
   ```

## Hiveテーブルを削除

StarRocksの内部テーブルと同様に、Hiveテーブルに対する [DROP](../../administration/privilege_item.md#table) 権限がある場合、[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) ステートメントを使用してそのHiveテーブルを削除できます。この機能はv3.1以降でサポートされています。現在、StarRocksはHiveのマネージドテーブルのみを削除することがサポートされています。

> **注記**
>
> 権限の付与と取り消しは、[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して行うことができます。

Hiveテーブルを削除する場合、DROP TABLEステートメントで`FORCE`キーワードを指定する必要があります。操作が完了すると、テーブルのファイルパスは保持されますが、HDFSクラスターやクラウドストレージ上のテーブルのデータはテーブルと共にすべて削除されます。この操作を行う際は注意してください。

[Hiveカタログとその中のデータベースに切り替え](#switch-to-a-hive-catalog-and-a-database-in-it)、次に示すステートメントを使用して、そのデータベース内のHiveテーブルを削除します。

```SQL
DROP TABLE <table_name> FORCE
```

## メタデータキャッシュの手動更新または自動更新

### 手動更新

デフォルトでは、StarRocksはHiveのメタデータをキャッシュし、非同期モードでメタデータを自動更新してパフォーマンスを向上させます。さらに、Hiveテーブルにスキーマ変更やテーブル更新が行われた後、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)を使用してメタデータを手動で更新し、StarRocksができるだけ早く最新のメタデータを取得し、適切な実行計画を生成することができます。

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

以下の状況では、メタデータを手動で更新する必要があります。

- 既存のパーティション内のデータファイルが変更された場合（例：`INSERT OVERWRITE ... PARTITION ...`コマンドを実行した場合）。
- Hiveテーブルにスキーマ変更が行われた場合。
- 既存のHiveテーブルがDROPステートメントを使用して削除され、削除されたHiveテーブルと同じ名前の新しいHiveテーブルが作成された場合。
- Hiveカタログの作成時に`"enable_cache_list_names" = "true"`を`PROPERTIES`に指定し、Hiveクラスターで作成したばかりの新しいパーティションをクエリしたい場合。

  > **注記**
  >
  > v2.5.5以降、StarRocksは定期的なHiveメタデータキャッシュのリフレッシュ機能を提供しています。詳細については、このトピックの「[定期的にメタデータキャッシュをリフレッシュする](#periodically-refresh-metadata-cache)」セクションをご覧ください。この機能を有効にすると、StarRocksはデフォルトで10分ごとにHiveメタデータキャッシュを更新します。そのため、ほとんどの場合、手動での更新は必要ありません。新しいパーティションがHiveクラスター上で作成された直後にすぐにクエリしたい場合のみ、手動での更新が必要です。

REFRESH EXTERNAL TABLEは、FEにキャッシュされているテーブルとパーティションのみをリフレッシュすることに注意してください。

### 自動増分アップデート

自動非同期アップデートポリシーとは異なり、自動増分アップデートポリシーでは、StarRocksクラスターのFEがHiveメタストアからイベント（列の追加、パーティションの削除、データの更新など）を読み取ることができます。StarRocksはこれらのイベントに基づいてFEにキャッシュされたメタデータを自動的にアップデートします。つまり、Hiveテーブルのメタデータを手動でアップデートする必要はありません。

この機能はHMSに大きな負荷をかける可能性があるため、使用する際は注意が必要です。[定期的にメタデータキャッシュをリフレッシュする](#periodically-refresh-metadata-cache)機能の使用を推奨します。

自動増分アップデートを有効にするには、以下の手順に従ってください：

#### ステップ1: Hiveメタストアのイベントリスナーを設定する

Hiveメタストアv2.xおよびv3.xはイベントリスナーの設定をサポートしています。ここでは、Hiveメタストアv3.1.2用のイベントリスナー設定を例にします。以下の設定項目を**$HiveMetastore/conf/hive-site.xml**ファイルに追加し、その後Hiveメタストアを再起動してください：

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

FEのログファイルで`event id`を検索して、イベントリスナーが正常に設定されているかどうかを確認できます。設定が失敗すると、`event id`の値は`0`になります。

#### ステップ2: StarRocksで自動増分アップデートを有効にする

自動増分アップデートは、StarRocksクラスタ内の単一のHiveカタログまたはすべてのHiveカタログに対して有効にできます。

- 単一のHiveカタログに対して自動増分アップデートを有効にするには、Hiveカタログを作成する際に`PROPERTIES`内で`enable_hms_events_incremental_sync`パラメータを`true`に設定します。

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```

- StarRocksクラスタ内のすべてのHiveカタログに対して自動増分アップデートを有効にするには、各FEの`$FE_HOME/conf/fe.conf`ファイルに`"enable_hms_events_incremental_sync" = "true"`を追加し、各FEを再起動してパラメータ設定を有効にします。

また、ビジネス要件に基づいて、各FEの`$FE_HOME/conf/fe.conf`ファイル内の以下のパラメータを調整し、各FEを再起動してパラメータ設定を有効にすることもできます。

| パラメータ                         | 説明                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocksがHiveメタストアからイベントを読み取る時間間隔。デフォルト値: `5000`。単位: ミリ秒。 |
| hms_events_batch_size_per_rpc     | StarRocksが一度に読み取ることができるイベントの最大数。デフォルト値: `500`。 |
| enable_hms_parallel_process_events | StarRocksがイベントを読み取る際に、イベントを並行処理するかどうかを指定します。有効な値: `true` と `false`。デフォルト値: `true`。`true`は並列処理を有効にし、`false`は並列処理を無効にします。 |
| hms_process_events_parallel_num   | StarRocksが並行処理できるイベントの最大数。デフォルト値: `4`。 |

## 定期的にメタデータキャッシュをリフレッシュする

v2.5.5以降、StarRocksは頻繁にアクセスされるHiveカタログのキャッシュされたメタデータを定期的にリフレッシュし、データ変更を感知することができます。Hiveメタデータキャッシュのリフレッシュは、以下の[FEパラメータ](../../administration/FE_configuration.md)を通じて設定できます：

| 設定項目                                           | デフォルト                              | 説明                                                  |
| ------------------------------------------------------------ | ------------------------------------ | ------------------------------------ |
| enable_background_refresh_connector_metadata                 | v3.0では`true`<br />v2.5では`false`  | 定期的なHiveメタデータキャッシュのリフレッシュを有効にするかどうか。有効にすると、StarRocksはHiveクラスターのメタストア（HiveメタストアまたはAWS Glue）をポーリングし、頻繁にアクセスされるHiveカタログのキャッシュされたメタデータをリフレッシュして、データ変更を感知します。`true`はリフレッシュを有効にし、`false`は無効にします。この項目は[FE動的パラメータ](../../administration/FE_configuration.md#configure-fe-dynamic-parameters)です。[ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)コマンドを使用して変更できます。 |
| background_refresh_metadata_interval_millis                  | `600000`（10分）                | 2回連続するHiveメタデータキャッシュのリフレッシュ間の間隔。単位: ミリ秒。この項目は[FE動的パラメータ](../../administration/FE_configuration.md#configure-fe-dynamic-parameters)です。[ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)コマンドを使用して変更できます。 |
| background_refresh_metadata_time_secs_since_last_access_secs | `86400`（24時間）                   | Hiveメタデータキャッシュリフレッシュタスクの有効期限。アクセスされたHiveカタログについて、指定された時間以上アクセスされていない場合、StarRocksはキャッシュされたメタデータのリフレッシュを停止します。アクセスされていないHiveカタログについては、StarRocksはキャッシュされたメタデータをリフレッシュしません。単位: 秒。この項目は[FE動的パラメータ](../../administration/FE_configuration.md#configure-fe-dynamic-parameters)です。[ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)コマンドを使用して変更できます。 |

定期的なHiveメタデータキャッシュのリフレッシュ機能とメタデータの自動非同期アップデートポリシーを併用することで、データアクセスが大幅に高速化され、外部データソースからの読み込み負荷が軽減され、クエリパフォーマンスが向上します。

## 付録：メタデータの自動非同期アップデートについて

自動非同期アップデートは、StarRocksがHiveカタログのメタデータをアップデートするために使用するデフォルトのポリシーです。

デフォルト（つまり、`enable_metastore_cache`と`enable_remote_file_cache`の両方が`true`に設定されている場合）、クエリがHiveテーブルのパーティションにヒットした場合、StarRocksはそのパーティションのメタデータとそのパーティションの下層データファイルのメタデータを自動的にキャッシュします。キャッシュされたメタデータは、遅延アップデートポリシーを使用してアップデートされます。

例えば、`table2`という名前のHiveテーブルがあり、`p1`、`p2`、`p3`、`p4`の4つのパーティションがあるとします。クエリが`p1`にヒットし、StarRocksは`p1`のメタデータと`p1`の下層データファイルのメタデータをキャッシュします。キャッシュされたメタデータをアップデートおよび破棄するデフォルトの時間間隔は以下の通りです：

- キャッシュされたメタデータを非同期にアップデートする時間間隔（`metastore_cache_refresh_interval_sec`パラメータによって指定される）は2時間です。
- `p1`の基になるデータファイルのキャッシュされたメタデータを非同期で更新する時間間隔は、`remote_file_cache_refresh_interval_sec`パラメータで指定された通り60秒です。
- `p1`のキャッシュされたメタデータを自動的に破棄する時間間隔は、`metastore_cache_ttl_sec`パラメータで指定された通り24時間です。
- `p1`の基になるデータファイルのキャッシュされたメタデータを自動的に破棄する時間間隔は、`remote_file_cache_ttl_sec`パラメータで指定された通り36時間です。

以下の図は、時間間隔をタイムライン上で理解しやすく示しています。

![キャッシュされたメタデータの更新と破棄のタイムライン](../../assets/catalog_timeline.png)

その後、StarRocksは以下のルールに従ってメタデータを更新または破棄します：

- 別のクエリが`p1`を再び参照し、最後の更新からの経過時間が60秒未満の場合、StarRocksは`p1`のキャッシュされたメタデータや基になるデータファイルのキャッシュされたメタデータを更新しません。
- 別のクエリが`p1`を再び参照し、最後の更新からの経過時間が60秒を超える場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを更新します。
- 別のクエリが`p1`を再び参照し、最後の更新からの経過時間が2時間を超える場合、StarRocksは`p1`のキャッシュされたメタデータを更新します。
- `p1`が最後の更新から24時間以内にアクセスされていない場合、StarRocksは`p1`のキャッシュされたメタデータを破棄します。メタデータは次のクエリ時に再度キャッシュされます。
- `p1`が最後の更新から36時間以内にアクセスされていない場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを破棄します。メタデータは次のクエリ時に再度キャッシュされます。
