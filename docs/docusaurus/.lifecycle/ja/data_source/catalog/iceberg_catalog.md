---
displayed_sidebar: "Japanese"
---

# アイスバーグ カタログ

アイスバーグ カタログは、StarRocks v2.4以降でサポートされている外部カタログの一種です。アイスバーグ カタログを使用すると、以下の操作が可能です。

- 手動でテーブルを作成する必要なく、アイスバーグに格納されているデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) または非同期マテリアライズド ビュー（v2.5以降でサポート）を使用して、アイスバーグに格納されているデータを処理し、StarRocksにデータをロードできます。
- StarRocksでの操作を行い、アイスバーグのデータベースやテーブルを作成または削除したり、Parquet形式のアイスバーグ テーブルにStarRocksテーブルからデータをシンクする際に [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用できます（この機能はv3.1以降でサポート）。

アイスバーグ クラスタでの成功するSQLワークロードを確保するには、StarRocks クラスタを次の2つの重要なコンポーネントに統合する必要があります。

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCS、またはその他のS3互換ストレージシステム（たとえば MinIO）などのオブジェクトストレージ
- Hiveメタストア、AWS Glue、またはTabularなどのメタストア

  > **注意**
  >
  > - ストレージとしてAWS S3を選択した場合、メタストアとしてHMSまたはAWS Glueを使用できます。その他のストレージシステムを選択した場合、HMSのみを使用できます。
  > - メタストアとしてTabularを選択した場合、Iceberg REST カタログを使用する必要があります。

## 使用上の注意

- StarRocksがサポートするアイスバーグのファイルフォーマットはParquetとORCです。

  - Parquetファイルは、以下の圧縮フォーマットをサポートしています：SNAPPY、LZ4、ZSTD、GZIP、およびNO_COMPRESSION。
  - ORCファイルは、以下の圧縮フォーマットをサポートしています：ZLIB、SNAPPY、LZO、LZ4、ZSTD、およびNO_COMPRESSION。

- アイスバーグ カタログはv1テーブルをサポートし、StarRocks v3.0以降でORC形式のv2テーブルをサポートしています。
- アイスバーグ カタログはv1テーブルをサポートしています。さらに、アイスバーグ カタログはStarRocks v3.0以降でORC形式のv2テーブルおよびStarRocks v3.1以降でParquet形式のv2テーブルをサポートしています。

## 統合の準備

アイスバーグ カタログを作成する前に、StarRocks クラスタがアイスバーグ クラスタのストレージシステムとメタストアに統合できることを確認してください。

### AWS IAM

アイスバーグ クラスタがストレージとしてAWS S3またはメタストアとしてAWS Glueを使用する場合は、適切な認証メソッドを選択し、StarRocks クラスタが関連するAWSクラウドリソースにアクセスできるように必要な準備を行ってください。

以下の認証メソッドが推奨されています：

- インスタンスプロファイル
- 仮定されたロール
- IAMユーザー

上記の3つの認証メソッドのうち、インスタンスプロファイルが最も一般的に使用されています。

詳細は[こちら](../../integrations/authenticate_to_aws_resources.md#preparations)をご覧ください。

### HDFS

ストレージとしてHDFSを選択した場合、StarRocks クラスタを次のように構成してください：

- (オプション) HDFSクラスタにアクセスするためのユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスタとHiveメタストアにアクセスします。`export HADOOP_USER_NAME="<user_name>"` を各FEの **fe/conf/hadoop_env.sh** ファイルと各BEの **be/conf/hadoop_env.sh** ファイルの先頭に追加することでユーザー名を設定できます。これらのファイルにユーザー名を設定した後は、各FEと各BEを再起動してパラメーター設定が有効になります。StarRocks クラスタには1つのユーザー名しか設定できません。
- アイスバーグデータをクエリする際、StarRocksクラスタのFEおよびBEはHDFSクライアントを使用してHDFSクラスタにアクセスします。ほとんどの場合、この目的を達成するためにStarRocksクラスタを構成する必要はありません。StarRocksはデフォルトの構成を使用してHDFSクライアントを開始します。次の状況下のみStarRocksクラスタを構成する必要があります：

  - HDFSクラスタの高可用性（HA）が有効になっている場合：HDFSクラスタの **hdfs-site.xml** ファイルを各FEの **$FE_HOME/conf** パスおよび各BEの **$BE_HOME/conf** パスに追加します。
  - HDFSクラスタのView File System (ViewFs)が有効になっている場合：HDFSクラスタの **core-site.xml** ファイルを各FEの **$FE_HOME/conf** パスおよび各BEの **$BE_HOME/conf** パスに追加します。

> **注意**
>
> クエリを送信した場合に不明なホストを示すエラーが返された場合、HDFSクラスタノードのホスト名とIPアドレスのマッピングを **/etc/hosts** パスに追加する必要があります。

### Kerberos 認証

HDFSクラスタまたはHiveメタストアに対してKerberos認証が有効になっている場合、StarRocks クラスタを次のように構成してください：

- 各FEおよび各BEで `kinit -kt keytab_path principal` コマンドを実行して、Key Distribution Center (KDC) からチケット発行チケット (TGT) を取得します。このコマンドを実行するには、HDFSクラスタとHiveメタストアにアクセスする権限が必要です。このコマンドを定期的に実行するためには、cronを使用する必要があります。このコマンドでKDCにアクセスすることは時間的に敏感です。したがって、このコマンドを定期的に実行するためにcronを使用する必要があります。
- 各FEの **$FE_HOME/conf/fe.conf** ファイルと各BEの **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。この例では、`/etc/krb5.conf` が **krb5.conf** ファイルの保存パスです。必要に応じてパスを変更することができます。

## アイスバーグ カタログの作成

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

アイスバーグ カタログの名前。命名規則は次のとおりです：

- 名前には、文字、数字（0-9）、アンダースコア(_)を含めることができます。ただし、文字で始める必要があります。
- 名前は大文字と小文字を区別し、長さが1023文字を超えることはできません。

#### comment

アイスバーグ カタログの説明。このパラメータはオプションです。

#### type

データソースの種類。値を `iceberg` に設定します。

#### MetastoreParams

データソースのメタストアとStarRocksの統合に関する一連のパラメータです。

##### Hiveメタストア

データソースのメタストアとしてHiveメタストアを選択した場合、`MetastoreParams` を次のように構成します：

```SQL
"iceberg.catalog.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

>**注記**
>
>- アイスバーグ データをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを **/etc/hosts** パスに追加する必要があります。そうしないと、クエリを開始する際にStarRocksがHiveメタストアにアクセスできなくなる可能性があります。
```json
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

以下の表は、 `MetastoreParams` で構成する必要のあるパラメータを説明しています。

| Parameter                     | Required | Description                                                  |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| iceberg.catalog.type          | Yes      | Iceberg クラスタで使用するメタストアのタイプ。`glue` に設定します。 |
| aws.glue.use_instance_profile | Yes      | インスタンス プロファイルに基づいた認証メソッドと、アサード ロールに基づいた認証メソッドを有効にするかどうかを指定します。`true` および `false` が有効な値です。デフォルト値は `false` です。 |
| aws.glue.iam_role_arn         | No       | AWS Glue データカタログに特権を持つ IAM ロールの ARN。AWS Glue へのアクセスにアサード ロールに基づいた認証メソッドを使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.region               | Yes      | AWS Glue データカタログが存在するリージョン。例: `us-west-1`。 |
| aws.glue.access_key           | No       | AWS IAM ユーザーのアクセスキー。AWS Glue へのアクセスに IAM ユーザーに基づいた認証メソッドを使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | No       | AWS IAM ユーザーのシークレットキー。AWS Glue へのアクセスに IAM ユーザーに基づいた認証メソッドを使用する場合、このパラメータを指定する必要があります。 |

AWS Glue にアクセスするための認証メソッドの選択と AWS IAM コンソールでのアクセス制御ポリシーの設定についての情報は、[AWS Glue アクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

##### 表形式

メタストアとして Tabular を使用する場合、 `MetastoreParams` を次のように構成する必要があります（`"iceberg.catalog.type" = "rest"`）。

```SQL
"iceberg.catalog.type" = "rest",
"iceberg.catalog.uri" = "<rest_server_api_endpoint>",
"iceberg.catalog.credential" = "<credential>",
"iceberg.catalog.warehouse" = "<identifier_or_path_to_warehouse>"
```

以下の表は、 `MetastoreParams` で構成する必要のあるパラメータを説明しています。

| Parameter                  | Required | Description                                                          |
| -------------------------- | -------- | -------------------------------------------------------------------- |
| iceberg.catalog.type       | Yes      | Iceberg クラスタで使用するメタストアのタイプ。`rest` に設定します。                 |
| iceberg.catalog.uri        | Yes      | Tabular サービスのエンドポイントの URI。例: `https://api.tabular.io/ws`。      |
| iceberg.catalog.credential | Yes      | Tabular サービスの認証情報。                                          |
| iceberg.catalog.warehouse  | No       | Iceberg カタログのウェアハウスの場所または識別子。例: `s3://my_bucket/warehouse_location` または `sandbox`。 |

以下の例は、Tabular をメタストアとして使用する `tabular` という名前の Iceberg カタログを作成します。

```SQL
CREATE EXTERNAL CATALOG tabular
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type"= "rest",
    "iceberg.catalog.uri" = "https://api.tabular.io/ws",
    "iceberg.catalog.credential" = "t-5Ii8e3FIbT9m0:aaaa-3bbbbbbbbbbbbbbbbbbb",
    "iceberg.catalog.warehouse" = "sandbox"
);
```

#### `StorageCredentialParams`

StarRocks がストレージ システムと統合する方法に関するパラメータのセット。このパラメータセットは任意です。

次の点に注意してください:

- ストレージとして HDFS を使用する場合、`StorageCredentialParams` を構成する必要はありません。このセクションをスキップすることができます。AWS S3、その他の S3 互換のストレージ システム、Microsoft Azure Storage、または Google GCS を使用する場合は、`StorageCredentialParams` を構成する必要があります。

- メタストアとして Tabular を使用する場合、`StorageCredentialParams` を構成する必要はありません。このセクションをスキップすることができます。HMS または AWS Glue をメタストアとして使用する場合は、`StorageCredentialParams` を構成する必要があります。

##### AWS S3

Iceberg クラスタのストレージとして AWS S3 を選択する場合、次のアクションのいずれかを実行してください:

- インスタンス プロファイルに基づいた認証メソッドを選択する場合は、`StorageCredentialParams` を次のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- アサード ロールに基づいた認証メソッドを選択する場合は、`StorageCredentialParams` を次のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM ユーザーに基づいた認証メソッドを選択する場合は、 `StorageCredentialParams` を次のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表は、`StorageCredentialParams` で構成する必要のあるパラメータを説明しています。

| Parameter                   | Required | Description                                                  |
| --------------------------- | -------- | ---------------------------------------------------------- |
| aws.s3.use_instance_profile | Yes      | インスタンス プロファイルに基づいた認証メソッドとアサード ロールに基づいた認証メソッドを有効にするかどうかを指定します。`true` および `false` が有効な値です。デフォルト値は `false` です。 |
| aws.s3.iam_role_arn         | No       | AWS S3 バケットに特権を持つ IAM ロールの ARN。AWS S3 へのアクセスにアサード ロールに基づいた認証メソッドを使用する場合、このパラメータを指定する必要があります。 |
| aws.s3.region               | Yes      | AWS S3 バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | No       | IAM ユーザーのアクセスキー。AWS S3 へのアクセスに IAM ユーザーに基づいた認証メソッドを使用する場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | No       | IAM ユーザーのシークレットキー。AWS S3 へのアクセスに IAM ユーザーに基づいた認証メソッドを使用する場合、このパラメータを指定する必要があります。 |

AWS S3 へのアクセスの認証メソッドの選択と AWS IAM コンソールでのアクセス制御ポリシーの設定についての情報は、[AWS S3 へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3 互換のストレージ システム

v2.5 以降、Iceberg カタログは S3 互換のストレージ システムをサポートしています。

S3 互換のストレージ システム（たとえば MinIO）をストレージとして選択する場合、以下のように `StorageCredentialParams` を構成してください。これにより、正常に統合できるようになります。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表は、`StorageCredentialParams` で構成する必要のあるパラメータを説明しています。

| Parameter                        | Required | Description                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | Yes      | SSL 接続を有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値は `true` です。 |
| aws.s3.enable_path_style_access  | Yes      | パス スタイルのアクセスを有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値は `false` です。MinIO の場合は、値を `true` に設定する必要があります。<br />パス スタイルの URL は次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、US West (Oregon) リージョンに `DOC-EXAMPLE-BUCKET1` という名前のバケットを作成し、そのバケット内の `alice.jpg` オブジェクトにアクセスしたい場合、次のパス スタイルの URL を使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | Yes      | Amazon S3 の代わりに S3 互換のストレージ システムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key                | Yes      | IAM ユーザーのアクセスキー。 |
| aws.s3.secret_key                | Yes      | IAM ユーザーのシークレットキー。 |

##### Microsoft Azure Storage

v3.0 以降、Iceberg カタログは Microsoft Azure Storage をサポートしています。

###### Azure Blob Storage

Blob Storage を Iceberg クラスタのストレージとして選択する場合、次のアクションのいずれかを実行してください:

- 共有キー認証メソッドを選択する場合は、`StorageCredentialParams` を次のように構成してください:

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下の表は、`StorageCredentialParams` で構成する必要のあるパラメータを説明しています。

  | **Parameter**              | **Required** | **Description**                              |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | Yes          | Blob Storage アカウントのユーザー名。    |
| azure.blob.shared_key      | Yes          | The shared key of your Blob Storage account. |

- SASトークン認証方法を選択する場合は、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  次のテーブルには、`StorageCredentialParams`で構成する必要があるパラメータが記載されています。

  | **パラメータ**             | **必須** | **説明**                                              |
  | ------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.account_name   | Yes          | Blob Storageアカウントのユーザー名。                   |
  | azure.blob.container_name | Yes          | データを格納しているblobコンテナの名前。        |
  | azure.blob.sas_token      | Yes          | Blob Storageアカウントにアクセスするために使用されるSASトークン。 |

###### Azure Data Lake Storage Gen1

IcebergクラスタのストレージとしてData Lake Storage Gen1を選択する場合は、次のいずれかの操作を行います。

- マネージドサービスアイデンティティ認証メソッドを選択する場合は、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  次のテーブルには、`StorageCredentialParams`で構成する必要があるパラメータが記載されています。

  | **パラメータ**                            | **必須** | **説明**                                              |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes          | マネージドサービスアイデンティティ認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |

- サービスプリンシパル認証メソッドを選択する場合は、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  次のテーブルには、`StorageCredentialParams`で構成する必要があるパラメータが記載されています。

  | **パラメータ**                 | **必須** | **説明**                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | Yes          | サービスプリンシパルのクライアント(アプリケーション)ID。        |
  | azure.adls1.oauth2_credential | Yes          | 作成された新しいクライアント(アプリケーション)シークレットの値。    |
  | azure.adls1.oauth2_endpoint   | Yes          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント(v1)。 |

###### Azure Data Lake Storage Gen2

IcebergクラスタのストレージとしてData Lake Storage Gen2を選択する場合は、次のいずれかの操作を行います。

- マネージドアイデンティティ認証メソッドを選択する場合は、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  次のテーブルには、`StorageCredentialParams`で構成する必要があるパラメータが記載されています。

  | **パラメータ**                           | **必須** | **説明**                                              |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes          | マネージドアイデンティティ認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | Yes          | アクセスしたいテナントのID。          |
  | azure.adls2.oauth2_client_id            | Yes          | マネージドアイデンティティのクライアント(アプリケーション)ID。      |

- 共有キー認証メソッドを選択する場合は、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次のテーブルには、`StorageCredentialParams`で構成する必要があるパラメータが記載されています。

  | **パラメータ**               | **必須** | **説明**                                              |
  | --------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account | Yes          | Data Lake Storage Gen2ストレージアカウントのユーザー名。 |
  | azure.adls2.shared_key      | Yes          | Data Lake Storage Gen2ストレージアカウントの共有キー。 |

- サービスプリンシパル認証メソッドを選択する場合は、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams`で構成する必要があるパラメータが記載されています。

  | **パラメータ**                      | **必須** | **説明**                        |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | Yes          | サービスプリンシパルのクライアント(アプリケーション)ID。     |
  | azure.adls2.oauth2_client_secret   | Yes          | 作成された新しいクライアント(アプリケーション)シークレットの値。     |
  | azure.adls2.oauth2_client_endpoint | Yes          | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント(v1)。 |

##### Google GCS

Icebergカタログは、v3.0以降でGoogle GCSをサポートしています。

IcebergクラスタのストレージとしてGoogle GCSを選択する場合は、次のいずれかの操作を行います。

- VMベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次のテーブルには、`StorageCredentialParams`で構成する必要があるパラメータが記載されています。

  | **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                              |
  | ------------------------------------------ | ----------------- | ----------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接Compute Engineにバインドされているサービスアカウントを使用するかどうかを指定します。 |

- サービスアカウントベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように設定します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  次のテーブルには、`StorageCredentialParams`で構成する必要があるパラメータが記載されています。

  | **パラメータ**                          | **デフォルト値** | **値の例**                                        | **説明**                                              |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウント作成時に生成されるJSONファイル内の電子メールアドレス。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウント作成時に生成されるJSONファイル内のプライベートキーID。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウント作成時に生成されるJSONファイル内のプライベートキー。 |

- インパーソネーションベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように設定します。

  - VMインスタンスがサービスアカウントを偽装するようにする場合:

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次のテーブルには、`StorageCredentialParams`で構成する必要があるパラメータが記載されています。

    | **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                              |
    | ------------------------------------------ | ----------------- | ----------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接Compute Engineにバインドされているサービスアカウントを使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | 偽装したいサービスアカウント。            |

  - サービスアカウント(一時的にmetaサービスアカウントと呼ばれます)が別のサービスアカウント(一時的にデータサービスアカウントと呼ばれます)を偽装する場合:

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次のテーブルには、`StorageCredentialParams`で構成する必要があるパラメータが記載されています。

    | **パラメータ**                          | **デフォルト値** | **値の例**                                        | **説明**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウントの作成時に生成されたJSONファイルでの電子メールアドレス。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキー。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 指定したデータサービスアカウントになりすまします。       |

### 例

次の例では、Icebergクラスタからデータをクエリするために、`iceberg_catalog_hms`または`iceberg_catalog_glue`というIcebergカタログを作成します。これは使用するメタストアのタイプに応じて異なります。

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

##### インスタンスプロファイルベースの資格情報を選択した場合

- IcebergクラスタでHiveメタストアを使用する場合、次のようなコマンドを実行します。

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

- Amazon EMR IcebergクラスタでAWS Glueを使用する場合、次のようなコマンドを実行します。

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

##### 仮定された役割ベースの資格情報を選択した場合

- IcebergクラスタでHiveメタストアを使用する場合、次のようなコマンドを実行します。

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

- Amazon EMR IcebergクラスタでAWS Glueを使用する場合、次のようなコマンドを実行します。

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

##### IAMユーザーベースの資格情報を選択した場合

- IcebergクラスタでHiveメタストアを使用する場合、次のようなコマンドを実行します。

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

- Amazon EMR IcebergクラスタでAWS Glueを使用する場合、次のようなコマンドを実行します。

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

#### S3互換のストレージシステム

例としてMinIOを使用する場合、次のようなコマンドを実行します。

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

- 共有キー認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- SASトークン認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- 管理されたサービスアイデンティティ認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- サービスプリンシパル認証メソッドを選択した場合、次のようなコマンドを実行します。

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

- 管理されたアイデンティティ認証メソッドを選択した場合、次のようなコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
```
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- もしShared Key認証方法を選択した場合は、以下のようなコマンドを実行してください:

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

- もしService Principal認証方法を選択した場合は、以下のようなコマンドを実行してください:

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

- もしVMベースの認証方法を選択した場合は、以下のようなコマンドを実行してください:

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

- もしサービスアカウントベースの認証方法を選択した場合は、以下のようなコマンドを実行してください:

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

- もし偽装ベースの認証方法を選択した場合:

  - もしVMインスタンスがサービスアカウントを偽装する場合は、以下のようなコマンドを実行してください:

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

  - もしサービスアカウントが他のサービスアカウントを偽装する場合は、以下のようなコマンドを実行してください:

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

## Icebergカタログを表示

現在のStarRocksクラスター内のすべてのカタログをクエリするには、[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用できます:

```SQL
SHOW CATALOGS;
```

また、次の例は、名前が`iceberg_catalog_glue`のIcebergカタログの作成ステートメントをクエリする[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用しています:

```SQL
SHOW CREATE CATALOG iceberg_catalog_glue;
```

## Icebergカタログおよびその中のデータベースに切り替える

以下のいずれかの方法を使用してIcebergカタログとその中のデータベースに切り替えることができます:

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションでIcebergカタログを指定し、次に[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します:

  ```SQL
  -- 現在のセッションで指定したカタログに切り替える:
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定する:
  USE <db_name>
  ```

- 直接[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、Icebergカタログとその中のデータベースに切り替える:

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Icebergカタログを削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して外部カタログを削除できます。

次の例では、`iceberg_catalog_glue`という名前のIcebergカタログを削除します:

```SQL
DROP Catalog iceberg_catalog_glue;
```

## Icebergテーブルのスキーマを表示する

次の構文のいずれかを使用してIcebergテーブルのスキーマを表示できます:

- スキーマを表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Icebergテーブルをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用してIcebergクラスター内のデータベースを表示します:

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Icebergカタログおよびその中のデータベースに切り替える](#Icebergカタログおよびその中のデータベースに切り替える)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して指定したデータベースの宛先テーブルをクエリします:

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Icebergデータベースを作成する

StarRocksの内部カタログと同様に、Icebergカタログに[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)権限がある場合、そのIcebergカタログ内でデータベースを作成するために[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)ステートメントを使用できます。この機能はv3.1以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Icebergカタログに切り替える](#Icebergカタログおよびその中のデータベースに切り替える)、そして次のステートメントを使用してそのカタログ内のIcebergデータベースを作成します:

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>/")]
```

`location`パラメータを使用してデータベースを作成するファイルパスを指定できます。HDFSとクラウドストレージの両方がサポートされています。`location`パラメータを指定しない場合、StarRocksはIcebergカタログのデフォルトのファイルパスにデータベースを作成します。

`prefix`は使用するストレージシステムに応じて異なります:

| **ストレージシステム**                                         | **`Prefix`値**                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                       | `hdfs`                                                       |
| Google GCS                                                 | `gs`                                                         |
| Azure Blob Storage                                         | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可している場合、`prefix`は`wasb`。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可している場合、`prefix`は`wasbs`。</li></ul> |
| Azure Data Lake Storage Gen1                               | `adl`                                                        |
| Azure Data Lake Storage Gen2                               | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可している場合、`prefix`は`abfs`。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可している場合、`prefix`は`abfss`。</li></ul> |
| AWS S3またはその他のS3互換ストレージ（MinIOなど） | `s3`                                                         |

## Icebergデータベースを削除する
```
同様に、StarRocksの内部データベースと同じように、アイスバーグデータベースに[DROP](../../administration/privilege_item.md#database)権限がある場合、[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md)ステートメントを使用してそのアイスバーグデータベースを削除できます。この機能はv3.1以降でサポートされています。空のデータベースのみ削除できます。

> **注意事項**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

アイスバーグデータベースを削除すると、データベースのファイルパスはHDFSクラスターまたはクラウドストレージから削除されません。

[アイスバーグカタログに切り替え](#switch-to-an-iceberg-catalog-and-a-database-in-it)し、次のステートメントを使用してそのカタログ内のアイスバーグデータベースを削除します。

```SQL
DROP DATABASE <database_name>;
```

## アイスバーグテーブルの作成

StarRocksの内部データベースと同様に、アイスバーグデータベースに[CREATE TABLE](../../administration/privilege_item.md#database)権限がある場合、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)または[CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) ステートメントを使用してそのアイスバーグデータベース内にテーブルを作成できます。この機能はv3.1以降でサポートされています。

> **注意事項**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[アイスバーグカタログおよびそのデータベースに切り替える](#switch-to-an-iceberg-catalog-and-a-database-in-it)と、以下の構文を使用してそのデータベース内にアイスバーグテーブルを作成できます。

### 構文

```SQL
CREATE TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1,column_definition2, ...
partition_column_definition1,partition_column_definition2...)
[partition_desc]
[PROPERTIES ("key" = "value", ...)]
[AS SELECT query]
```

### パラメータ

#### column_definition

`column_definition`の構文は以下の通りです。

```SQL
col_name col_type [COMMENT 'comment']
```

以下の表は、パラメータについて説明しています。

| パラメータ | 説明 |
| ----------------- |------------------------------------------------------ |
| col_name          | カラムの名前。                                                |
| col_type          | カラムのデータ型。以下のデータ型がサポートされています: TINYINT, SMALLINT, INT, BIGINT, FLOAT, DOUBLE, DECIMAL, DATE, DATETIME, CHAR, VARCHAR[(length)], ARRAY, MAP, and STRUCT。LARGEINT、HLL、およびBITMAPデータ型はサポートされていません。 |

> **注意**
>
> パーティション以外のすべてのカラムはデフォルト値として`NULL`を使用する必要があります。つまり、テーブル作成ステートメントの各非パーティション列には、`DEFAULT "NULL"`を指定する必要があります。さらに、パーティション列は非パーティション列に続いて定義する必要があり、デフォルト値として`NULL`を使用することはできません。

#### partition_desc

`partition_desc`の構文は以下の通りです。

```SQL
PARTITION BY (par_col1, par_col2...)
```

現在のStarRocksは[identity transforms](https://iceberg.apache.org/spec/#partitioning)のみをサポートしており、これによりStarRocksは各ユニークなパーティション値に対してパーティションを作成します。

> **注意**
>
> パーティション列は非パーティション列に続いて定義する必要があります。パーティション列はFLOAT、DOUBLE、DECIMAL、およびDATETIMEを除くすべてのデータ型をサポートしており、デフォルト値として`NULL`を使用することはできません。

#### PROPERTIES

`PROPERTIES`内でテーブル属性を`"key" = "value"`形式で指定できます。[Iceberg table attributes](https://iceberg.apache.org/docs/latest/configuration/)を参照してください。

次の表は、いくつかの主要なプロパティについて説明しています。

| プロパティ | 説明 |
| ----------------- | ------------------------------------------------------------ |
| location          | アイスバーグテーブルを作成するファイルパス。HMSをメタストアとして使用する場合、現在のアイスバーグカタログのデフォルトファイルパスにテーブルを作成するため、`location`パラメータを指定する必要はありません。AWS Glueをメタストアとして使用する場合:<ul><li>テーブルを作成するデータベースの`location`パラメータを指定した場合、テーブルの`location`パラメータを指定する必要はありません。そのような場合、テーブルはそれが属するデータベースのファイルパスにデフォルトで設定されます。</li><li>データベースの`location`を指定していない場合、テーブルを作成するデータベースの`location`パラメータを指定する必要があります。</li></ul> |
| file_format       | アイスバーグテーブルのファイル形式。Parquet形式のみがサポートされています。デフォルト値:`parquet`。 |
| compression_codec | アイスバーグテーブルで使用される圧縮アルゴリズム。サポートされている圧縮アルゴリズムはSNAPPY、GZIP、ZSTD、およびLZ4です。デフォルト値:`gzip`。 |

### 例

1. 次のように、非パーティションのテーブル`unpartition_tbl`を作成します。このテーブルには、`id`と`score`の2つのカラムが含まれています。

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. 次のように、パーティションのテーブル`partition_tbl_1`を作成します。このテーブルには、`action`、`id`、`dt`の3つのカラムが含まれており、`id`および`dt`がパーティション列として定義されています。

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. 既存の`partition_tbl_1`という名前のテーブルからクエリし、そのクエリ結果に基づいて`partition_tbl_2`という名前のパーティションテーブルを作成します。`partition_tbl_2`の場合、`id`および`dt`がパーティション列として定義されています。

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (id, dt)
   AS SELECT * from employee;
   ```

## アイスバーグテーブルへのデータシンク

StarRocksの内部テーブルと同様に、アイスバーグテーブルに[INSERT](../../administration/privilege_item.md#table)権限がある場合、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) ステートメントを使用してStarRocksのテーブルのデータをそのアイスバーグテーブルにシンクできます(現在はParquet形式のアイスバーグテーブルのみがサポートされています)。この機能はv3.1以降でサポートされています。

> **注意事項**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[アイスバーグカタログおよびそのデータベースに切り替え](#switch-to-an-iceberg-catalog-and-a-database-in-it)し、以下の構文を使用してそのデータベース内のParquet形式のアイスバーグテーブルにStarRocksテーブルのデータをシンクできます。

### 構文

```SQL
INSERT {INTO | OVERWRITE} <table_name>
(列名 [, ...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータをシンクしたい場合、次の構文を使用します:
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

> **注意事項**
>
> パーティション列には`NULL`値を指定できません。したがって、アイスバーグテーブルのパーティション列に空の値がロードされないようにする必要があります。

### パラメータ

| パラメータ   | 説明                                                         |
| ----------- | ------------------------------------------------------------ |
| INTO        | スターコックステーブルのデータをアイスバーグテーブルに追加します。                                       |
| OVERWRITE   | スターコックステーブルのデータでアイスバーグテーブルの既存データを上書きします。                                       |
| column_name | データをロードする宛先列の名前。1つ以上の列を指定できます。複数の列を指定する場合は、カンマ(,)で区切ります。指定できるのは実際にアイスバーグテーブルに存在する列のみで、指定する宛先列にはアイスバーグテーブルのパーティション列を含める必要があります。指定した宛先列は、スターコックステーブルの列のいかなる宛先列名でもあっても、順番に1対1でマップされます。宛先列名が何であろうとも、スターコックステーブルの非パーティション列がアイスバーグテーブルの任意の列にマップできない場合、スターコックスはアイスバーグテーブルの列にデフォルト値`NULL`を書き込みます。INSERT ステートメントには、返された列の型が宛先列のデータ型と異なるクエリステートメントが含まれている場合、型が一致しない列に対してスターコックスが暗黙的に変換を行います。変換に失敗した場合、構文解析エラーが返されます。|
| expression  | 宛先のカラムに値を割り当てる式。                                   |
| DEFAULT     | 宛先のカラムにデフォルト値を割り当てる。                                         |
| query       | Iceberg テーブルにロードされるクエリステートメント。StarRocks でサポートされている任意の SQL ステートメントを使用できます。 |
| PARTITION   | データをロードするパーティション。Iceberg テーブルのすべてのパーティションカラムを指定する必要があります。このプロパティで指定するパーティションカラムは、テーブル作成ステートメントで定義したパーティションカラムのシーケンスと異なる場合があります。このプロパティを指定すると、`column_name` プロパティを指定できません。 |

### 例

1. `partition_tbl_1` テーブルにデータ行を 3 つ挿入する場合：

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. 結果に対する SELECT クエリ（単純な計算を含む）を `partition_tbl_1` テーブルに挿入する場合：

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. `partition_tbl_1` テーブルからデータを読み取る SELECT クエリの結果を同じテーブルに挿入する場合：

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. 2 つの条件 `dt='2023-09-01'` および `id=1` を満たすパーティションに SELECT クエリの結果を挿入する場合（`partition_tbl_2` テーブル）：

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT INTO partition_tbl_2 partition(dt='2023-09-01',id=1) SELECT 'order';
   ```

5. 2 つの条件 `dt='2023-09-01'` および `id=1` を満たすパーティション内のすべての `action` カラムの値を `close` に上書きする場合（`partition_tbl_1` テーブル）：

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT OVERWRITE partition_tbl_1 partition(dt='2023-09-01',id=1) SELECT 'close';
   ```

## Iceberg テーブルの削除

StarRocks の内部テーブルと同様に、Iceberg テーブルの [DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) ステートメントを使用して Iceberg テーブルを削除できます。この機能は v3.1 以降でサポートされています。

> **注記**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) および [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して特権を付与および取り消すことができます。

Iceberg テーブルを削除すると、テーブルのファイルパスと HDFS クラスターまたはクラウドストレージのデータは削除されません。

Iceberg テーブルを強制的に削除する場合（つまり、DROP TABLE ステートメントで `FORCE` キーワードを指定した場合）、テーブルのデータは HDFS クラスターまたはクラウドストレージから削除されますが、テーブルのファイルパスは保持されます。

[Iceberg カタログとそのデータベースに切り替える](#switch-to-an-iceberg-catalog-and-a-database-in-it) と、それから次のステートメントを使用して、そのデータベース内の Iceberg テーブルを削除します。

```SQL
DROP TABLE <table_name> [FORCE];
```

## メタデータキャッシュの構成

Iceberg クラスターのメタデータファイルは、AWS S3 や HDFS などのリモートストレージに保存される場合があります。デフォルトでは、StarRocks は Iceberg メタデータをメモリにキャッシュします。クエリの高速化のために、StarRocks はメタデータの両方をメモリとディスクにキャッシュする二段階のメタデータキャッシングメカニズムを採用しています。初回のクエリごとに、StarRocks はその計算結果をキャッシュします。以前のクエリと意味的に等価な後続のクエリが発行された場合、StarRocks は最初にその要求されたメタデータをキャッシュから取得し、メタデータがキャッシュにヒットしない場合のみ、リモートストレージからメタデータを取得します。

StarRocks は、最近最も使用されなかった (LRU) アルゴリズムを使用してデータをキャッシュおよび削除します。基本的なルールは次のとおりです。

- StarRocks はまずメモリから要求されたメタデータを取得しようとします。メタデータがメモリにヒットしない場合、StarRocks はディスクからメタデータを取得しようとします。StarRocks がディスクから取得したメタデータはメモリにロードされます。メタデータがディスクにヒットしない場合、StarRocks はリモートストレージからメタデータを取得し、取得したメタデータをメモリにキャッシュします。
- StarRocks はメモリから追い出されたメタデータをディスクに書き込みますが、ディスクから追い出されたメタデータは直接破棄します。

以下の表には、Iceberg メタデータキャッシングメカニズムを構成するために使用できる FE 構成アイテムが示されています。

| **構成アイテム**                                   | **単位** | **デフォルト値**                                    | **説明**                                              |
| -------------------------------------------------- | -------- | ---------------------------------------------------- | ----------------------------------------------------- |
| enable_iceberg_metadata_disk_cache                  | N/A      | `false`                                              | ディスクキャッシュを有効にするかどうかを指定します。 |
| iceberg_metadata_cache_disk_path                    | N/A      | `StarRocksFE.STARROCKS_HOME_DIR + "/caches/iceberg"` | ディスク上のキャッシュされたメタデータファイルの保存パス。 |
| iceberg_metadata_disk_cache_capacity                | バイト   | `2147483648`、2 GB に相当                            | ディスク上で許可されるキャッシュされたメタデータの最大サイズ。 |
| iceberg_metadata_memory_cache_capacity              | バイト   | `536870912`、512 MB に相当                           | メモリ内で許可されるキャッシュされたメタデータの最大サイズ。 |
| iceberg_metadata_memory_cache_expiration_seconds    | 秒      | `86500`                                              | 最終アクセスからカウントされる、メモリ内のキャッシュエントリの有効期限までの時間。 |
| iceberg_metadata_disk_cache_expiration_seconds      | 秒      | `604800`、1 週間に相当                              | 最終アクセスからカウントされる、ディスク上のキャッシュエントリの有効期限までの時間。 |
| iceberg_metadata_cache_max_entry_size               | バイト   | `8388608`、8 MB に相当                               | キャッシュできるファイルの最大サイズ。このパラメータの値を超えるファイルはキャッシュできません。これらのファイルがクエリで要求される場合、StarRocks はリモートストレージからそれらを取得します。 |