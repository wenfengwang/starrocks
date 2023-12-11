---
displayed_sidebar: "Japanese"
---

# Iceberg カタログ

Iceberg カタログは、StarRocks v2.4 以降でサポートされる外部カタログの一種です。Iceberg カタログを使用すると、次のようなことができます。

- 手動でテーブルを作成する必要なく、Iceberg に格納されたデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) または非同期マテリアライズド ビュー (v2.5 以降でサポート) を使用して、Iceberg に格納されたデータを処理し、StarRocks にデータをロードできます。
- StarRocks での操作を実行して、Iceberg データベースおよびテーブルを作成または削除し、または[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して StarRocks テーブルから Parquet 形式の Iceberg テーブルにデータをシンクできます (この機能は v3.1 以降でサポート)。

Iceberg クラスターに対して成功した SQL ワークロードを実行するためには、StarRocks クラスターを次の 2 つの重要なコンポーネントに統合する必要があります。

- 分散ファイルシステム (HDFS) または AWS S3、Microsoft Azure Storage、Google GCS、またはその他の S3 互換ストレージ システム (たとえば MinIO) などのオブジェクト ストレージ
- Hive メタストア、AWS Glue、または Tabular などのメタストア

  > **注記**
  >
  > - ストレージとして AWS S3 を選択した場合、HMS または AWS Glue をメタストアとして使用できます。その他のストレージ システムを選択した場合、HMS のみをメタストアとして使用できます。
  > - メタストアとして Tabular を選択した場合、Iceberg REST カタログを使用する必要があります。

## 使用上の注意

- StarRocks がサポートする Iceberg のファイル形式は Parquet および ORC です。

  - Parquet ファイルは、次の圧縮形式をサポートしています: SNAPPY、LZ4、ZSTD、GZIP、および NO_COMPRESSION。
  - ORC ファイルは、次の圧縮形式をサポートしています: ZLIB、SNAPPY、LZO、LZ4、ZSTD、および NO_COMPRESSION。

- Iceberg カタログは v1 テーブルをサポートし、StarRocks v3.0 以降では ORC 形式の v2 テーブルもサポートします。
- Iceberg カタログは v1 テーブルをサポートします。さらに、Iceberg カタログは StarRocks v3.0 以降で ORC 形式の v2 テーブルをサポートし、StarRocks v3.1 以降で Parquet 形式の v2 テーブルもサポートします。

## 統合の準備

Iceberg カタログを作成する前に、StarRocks クラスターが Iceberg クラスターのストレージ システムおよびメタストアに統合できることを確認してください。

### AWS IAM

Iceberg クラスターが AWS S3 をストレージとして使用したり、AWS Glue をメタストアとして使用する場合は、適切な認証方法を選択し、StarRocks クラスターが関連する AWS クラウド リソースにアクセスできるように準備する必要があります。

次の認証方法が推奨されています。

- インスタンス プロファイル
- 仮定されたロール
- IAM ユーザー

上記の 3 つの認証方法のうち、インスタンス プロファイルが最も一般的に使用されます。

詳細については、[AWS IAM の認証の準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとして HDFS を選択した場合は、StarRocks クラスターを次のように構成します。

- (オプション) HDFS クラスターにアクセスするために使用されるユーザー名を設定します。デフォルトでは、StarRocks は FE および BE プロセスのユーザー名を使用して HDFS クラスターおよび Hive メタストアにアクセスします。`export HADOOP_USER_NAME="<user_name>"` を各 FE の **fe/conf/hadoop_env.sh** ファイルの先頭および各 BE の **be/conf/hadoop_env.sh** ファイルの先頭に追加してユーザー名を設定できます。これらのファイルにユーザー名を設定した後は、各 FE および各 BE を再起動してパラメーター設定を有効にします。StarRocks クラスターごとに 1 つのユーザー名のみを設定できます。
- Iceberg データをクエリする際に、StarRocks クラスターの FE と BE が HDFS クライアントを使用して HDFS クラスターにアクセスします。ほとんどの場合、この目的のために StarRocks クラスターを構成する必要はありません。StarRocks はデフォルトの構成で HDFS クライアントを起動します。StarRocks クラスターを構成する必要があるのは次の状況だけです。

  - HDFS クラスターで高可用性 (HA) が有効になっている場合: HDFS クラスターの **hdfs-site.xml** ファイルを、各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに追加します。
  - HDFS クラスターで View File System (ViewFs) が有効になっている場合: HDFS クラスターの **core-site.xml** ファイルを、各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに追加します。

> **注記**
>
> クエリを送信する際に不明なホストを示すエラーが返された場合は、HDFS クラスター ノードのホスト名と IP アドレスのマッピングを **/etc/hosts** パスに追加する必要があります。

### Kerberos 認証

HDFS クラスターまたは Hive メタストアで Kerberos 認証が有効になっている場合は、StarRocks クラスターを次のように構成します。

- 各 FE と各 BE で `kinit -kt keytab_path principal` コマンドを実行して、Key Distribution Center (KDC) から Ticket Granting Ticket (TGT) を取得します。このコマンドを実行するには、HDFS クラスターや Hive メタストアにアクセスする権限が必要です。このコマンドで KDC にアクセスするための時間制限があります。そのためこのコマンドを定期的に実行するには、cron を使用する必要があります。
- 各 FE の **$FE_HOME/conf/fe.conf** ファイルおよび各 BE の **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。この例では、`/etc/krb5.conf` が **krb5.conf** ファイルの保存パスです。必要に応じてパスを変更できます。

## Iceberg カタログの作成

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

### パラメーター

#### catalog_name

Iceberg カタログの名前です。名前の規則は次のとおりです。

- 名前には英字、数字 (0-9)、アンダースコア (_) を含めることができます。英字で始める必要があります。
- 名前は大文字と小文字を区別し、長さが 1023 文字を超えることはできません。

#### comment

Iceberg カタログの説明です。このパラメーターはオプションです。

#### type

データソースのタイプです。値を `iceberg` に設定します。

#### MetastoreParams

データソースのメタストアとの統合方法に関するパラメーターのセットです。

##### Hive メタストア

データソースのメタストアとして Hive メタストアを選択した場合は、`MetastoreParams` を次のように構成します。

```SQL
"iceberg.catalog.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> Iceberg データをクエリする前に、Hive メタストア ノードのホスト名と IP アドレスのマッピングを **/etc/hosts** パスに追加する必要があります。そうしないと、クエリを開始する際に StarRocks が Hive メタストアにアクセスできない場合があります。

次の表には、`MetastoreParams` で構成する必要のあるパラメーターについて説明します。

| パラメーター               | 必須   | 説明                                                         |
| -------------------------- | ------ | ------------------------------------------------------------ |
| iceberg.catalog.type       | はい   | Iceberg クラスターで使用するメタストアのタイプ。値を `hive` に設定します。 |
| hive.metastore.uris        | はい   | Hive メタストアの URI。フォーマット: `thrift://<metastore_IP_address>:<metastore_port>`。<br />Hive メタストアで高可用性 (HA) が有効になっている場合は、複数のメタストア URI を指定し、コンマ (`,`) で区切って指定します。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"` とします。 |

##### AWS Glue

ストレージとして AWS S3 を選択した場合にデータソースのメタストアとして AWS Glue を選択している場合は、次のいずれかの方法を選択します。

- インスタンス プロファイルベースの認証方法を選択する場合は、`MetastoreParams` を次のように構成します:

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 仮定されたロールベースの認証方法を選択する場合は、`MetastoreParams` を次のように構成します:

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAM ユーザー ベースの認証方法を選択する場合は、`MetastoreParams` を次のように構成します:

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
```
"aws.glue.secret_key" = "<iam_user_secret_key>",
"aws.glue.region" = "<aws_s3_region>"
```

`MetastoreParams` で構成する必要があるパラメーターに関する次の表を参照してください。

| パラメーター                    | 必須      | 説明                                                          |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| iceberg.catalog.type          | Yes      | Iceberg クラスターで使用するメタストアのタイプ。値を `glue` に設定します。 |
| aws.glue.use_instance_profile | Yes      | インスタンスプロファイルベースの認証方法と、前提となるロールベースの認証方法を有効にするかどうかを指定します。`true` および `false` が有効な値です。デフォルト値は `false` です。 |
| aws.glue.iam_role_arn         | No       | お使いの AWS Glue データカタログに特権を持つ IAM ロールの ARN。AWS Glue へのアクセスに前提となるロールベースの認証方法を使用する場合、このパラメーターを指定する必要があります。 |
| aws.glue.region               | Yes      | AWS Glue データカタログが存在するリージョン。例: `us-west-1`。|
| aws.glue.access_key           | No       | AWS IAM ユーザーのアクセスキー。AWS Glue へのアクセスに IAM ユーザーベースの認証方法を使用する場合、このパラメーターを指定する必要があります。 |
| aws.glue.secret_key           | No       | AWS IAM ユーザーのシークレットキー。AWS Glue へのアクセスに IAM ユーザーベースの認証方法を使用する場合、このパラメーターを指定する必要があります。|

AWS Glue へのアクセス認証方法の選択と AWS IAM コンソールでアクセスコントロールポリシーを構成する方法について詳細については、[AWS Glue へのアクセス認証パラメーター](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue) を参照してください。

##### 表形式

メタストアとして Tabular を使用する場合、REST(`"iceberg.catalog.type" = "rest"`) としてメタストアのタイプを指定する必要があります。`MetastoreParams` を次のように構成してください。

```SQL
"iceberg.catalog.type" = "rest",
"iceberg.catalog.uri" = "<rest_server_api_endpoint>",
"iceberg.catalog.credential" = "<credential>",
"iceberg.catalog.warehouse" = "<identifier_or_path_to_warehouse>"
```

`MetastoreParams` で構成する必要があるパラメーターに関する次の表を参照してください。

| パラメーター              | 必須      | 説明                                             |
| -------------------------- | -------- | ------------------------------------------------- |
| iceberg.catalog.type       | Yes      | Iceberg クラスターで使用するメタストアのタイプ。値を `rest` に設定します。          |
| iceberg.catalog.uri        | Yes      | Tabular サービスエンドポイントの URI。例: `https://api.tabular.io/ws`。     |
| iceberg.catalog.credential | Yes      | Tabular サービスの認証情報。                                             |
| iceberg.catalog.warehouse  | No       | Iceberg カタログのウェアハウスの場所または識別子。例: `s3://my_bucket/warehouse_location` または `sandbox`。|

次の例は、Tabular をメタストアとして使用する `tabular` という Iceberg カタログを作成します。

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

あなたのストレージシステムを StarRocks に統合する方法に関する一連のパラメーターです。このパラメーターセットは任意です。

以下の点に注意してください。

- ストレージとして HDFS を使用する場合、`StorageCredentialParams` を構成する必要はなく、この項目をスキップできます。AWS S3、その他の S3 互換のストレージシステム、Microsoft Azure Storage、または Google GCS を使用する場合、`StorageCredentialParams` を構成する必要があります。

- メタストアとして Tabular を使用する場合、`StorageCredentialParams` を構成する必要はなく、この項目をスキップできます。HMS または AWS Glue をメタストアとして使用する場合、`StorageCredentialParams` を構成する必要があります。

##### AWS S3

Iceberg クラスターのストレージとして AWS S3 を選択する場合は、次のいずれかのアクションを実行してください。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams` を次のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 前提となるロールベースの認証方法を選択する場合、`StorageCredentialParams` を次のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM ユーザーベースの認証方法を選択する場合、`StorageCredentialParams` を次のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

`StorageCredentialParams` で構成する必要があるパラメーターに関する次の表を参照してください。

| パラメーター                   | 必須      | 説明                                                          |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes      | インスタンスプロファイルベースの認証方法と、前提となるロールベースの認証方法を有効にするかどうかを指定します。`true` および `false` が有効な値です。デフォルト値は `false` です。 |
| aws.s3.iam_role_arn         | No       | AWS S3 バケットに特権を持つ IAM ロールの ARN。AWS S3 へのアクセスに前提となるロールベースの認証方法を使用する場合、このパラメーターを指定する必要があります。 |
| aws.s3.region               | Yes      | AWS S3 バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | No       | お使いの IAM ユーザーのアクセスキー。AWS S3 へのアクセスに IAM ユーザーベースの認証方法を使用する場合、このパラメーターを指定する必要があります。 |
| aws.s3.secret_key           | No       | お使いの IAM ユーザーのシークレットキー。AWS S3 へのアクセスに IAM ユーザーベースの認証方法を使用する場合、このパラメーターを指定する必要があります。 |

AWS S3 へのアクセス認証方法の選択と AWS IAM コンソールでアクセス制御ポリシーを構成する方法について詳細については、[AWS S3 へのアクセス認証パラメーター](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3) を参照してください。

##### S3 互換のストレージシステム

Iceberg カタログは v2.5 以降、S3 互換のストレージシステムをサポートしています。

ストレージとして MinIO などの S3 互換のストレージシステムを選択する場合、成功した統合を確実にするために `StorageCredentialParams` を次のように構成してください:

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

`StorageCredentialParams` で構成する必要があるパラメーターに関する次の表を参照してください。

| パラメーター                        | 必須      | 説明                                                          |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | Yes      | SSL 接続を有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値は `true` です。  |
| aws.s3.enable_path_style_access  | Yes      | パス形式のアクセスを有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値は `false` です。MinIO の場合、この値を `true` に設定する必要があります。<br />パス形式の URL は、次の形式を使用します:`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、米国西部（オレゴン）リージョンに `DOC-EXAMPLE-BUCKET1` という名前のバケットを作成し、そのバケット内の `alice.jpg` オブジェクトにアクセスする場合、次のパス形式の URL を使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | Yes      | AWS S3 の代わりに S3 互換のストレージシステムに接続するために使用されるエンドポイント。     |
| aws.s3.access_key                | Yes      | お使いの IAM ユーザーのアクセスキー。                 |
| aws.s3.secret_key                | Yes      | お使いの IAM ユーザーのシークレットキー。             |

##### Microsoft Azure Storage

Iceberg カタログは v3.0 以降、Microsoft Azure Storage をサポートしています。

###### Azure Blob Storage

Iceberg クラスターのストレージとして Blob Storage を選択する場合は、次のいずれかのアクションを実行してください:

- 共有キー認証方法を選択する場合、`StorageCredentialParams` を次のように構成してください:

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` で構成する必要があるパラメーターに関する次の表を参照してください。

  | **パラメーター**               | **必須**      | **説明**                                      |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | Yes          | Blob Storage アカウントのユーザー名。         |
```
- `azure.blob.shared_key` = "Yes"
  ブロブストレージアカウントの共有キーです。

- SASトークン認証方法を選択するには、`StorageCredentialParams`を以下のように構成します。

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams`で構成する必要のあるパラメータについては、以下の表を参照してください。

  | **パラメータ**             | **必須** | **説明**                                                             |
  | ------------------------- | ---------- | ------------------------------------------------------------ |
  | azure.blob.account_name   | Yes        | ブロブストレージアカウントのユーザー名です。                         |
  | azure.blob.container_name | Yes        | データを格納するブロブコンテナの名前です。                        |
  | azure.blob.sas_token      | Yes        | ブロブストレージアカウントにアクセスするために使用されるSASトークンです。 |

###### Azure Data Lake Storage Gen1

IcebergクラスタのストレージとしてData Lake Storage Gen1を選択する場合、次のいずれかのアクションを実行してください:

- 管理サービスID認証方法を選択するには、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams`で構成する必要のあるパラメータについては、以下の表を参照してください。

  | **パラメータ**                            | **必須** | **説明**                                                             |
  | ---------------------------------------- | ---------- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes        | 管理サービスID認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |

- サービスプリンシパル認証方法を選択するには、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams`で構成する必要のあるパラメータについては、以下の表を参照してください。

  | **パラメータ**                 | **必須** | **説明**                                                             |
  | ----------------------------- | ---------- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | Yes        | サービスプリンシパルのクライアント(アプリケーション)IDです。            |
  | azure.adls1.oauth2_credential | Yes        | 作成された新しいクライアント(アプリケーション)シークレットの値です。         |
  | azure.adls1.oauth2_endpoint   | Yes        | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント(v1)です。 |

###### Azure Data Lake Storage Gen2

IcebergクラスタのストレージとしてData Lake Storage Gen2を選択する場合、次のいずれかのアクションを実行してください:

- 管理ID認証方法を選択するには、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  `StorageCredentialParams`で構成する必要のあるパラメータについては、以下の表を参照してください。

  | **パラメータ**                           | **必須** | **説明**                                                             |
  | --------------------------------------- | ---------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes        | 管理ID認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | Yes        | アクセスするテナントのIDです。                                       |
  | azure.adls2.oauth2_client_id            | Yes        | 管理IDのクライアント(アプリケーション)IDです。                           |

- 共有キー認証方法を選択するには、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams`で構成する必要のあるパラメータについては、以下の表を参照してください。

  | **パラメータ**               | **必須** | **説明**                                                             |
  | --------------------------- | ---------- | ------------------------------------------------------------ |
  | azure.adls2.storage_account | Yes        | Data Lake Storage Gen2ストレージアカウントのユーザー名です。           |
  | azure.adls2.shared_key      | Yes        | Data Lake Storage Gen2ストレージアカウントの共有キーです。           |

- サービスプリンシパル認証方法を選択するには、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams`で構成する必要のあるパラメータについては、以下の表を参照してください。

  | **パラメータ**                      | **必須** | **説明**                                                             |
  | ---------------------------------- | ---------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | Yes        | サービスプリンシパルのクライアント(アプリケーション)IDです。            |
  | azure.adls2.oauth2_client_secret   | Yes        | 作成された新しいクライアント(アプリケーション)シークレットの値です。         |
  | azure.adls2.oauth2_client_endpoint | Yes        | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント(v1)です。 |

##### Google GCS

Icebergカタログはv3.0以降でGoogle GCSをサポートしています。

IcebergクラスタのストレージとしてGoogle GCSを選択する場合、次のいずれかのアクションを実行してください:

- VMベースの認証方法を選択するには、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams`で構成する必要のあるパラメータについては、以下の表を参照してください。

  | **パラメータ**                              | **既定値** | **値の例**                  | **説明**                                                             |
  | ------------------------------------------ | ---------- | -------------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false      | true                       | 直接、Compute Engineにバインドされたサービスアカウントを使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択するには、`StorageCredentialParams`を以下のように構成します:

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  `StorageCredentialParams`で構成する必要のあるパラメータについては、以下の表を参照してください。

  | **パラメータ**                          | **既定値** | **値の例**                                                  | **説明**                                                             |
  | -------------------------------------- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""         | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントのJSONファイル作成時に生成された電子メールアドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | サービスアカウントのJSONファイル作成時に生成されたプライベートキーIDです。 |
  | gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウントのJSONファイル作成時に生成されたプライベートキーです。 |

- 擬似化ベースの認証方法を選択するには、`StorageCredentialParams`を以下のように構成します:

  - VMインスタンスをサービスアカウントに擬似化させる場合:

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams`で構成する必要のあるパラメータについては、以下の表を参照してください。

    | **パラメータ**                              | **既定値** | **値の例**                  | **説明**                                                             |
    | ------------------------------------------ | ---------- | -------------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false      | true                       | 直接、Compute Engineにバインドされたサービスアカウントを使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""         | "hello"                    | 擬似化したいサービスアカウントです。                            |

  - サービスアカウント(一時的にメタサービスアカウントとして命名)を別のサービスアカウント(一時的にデータサービスアカウントとして命名)に擬似化する場合:

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    `StorageCredentialParams`で構成する必要のあるパラメータについては、以下の表を参照してください。
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウントの作成時に生成されたJSONファイル内の電子メールアドレスです。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーIDです。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウントの作成時に生成されたJSONファイル内の秘密鍵です。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 擬似化したいデータサービスアカウントです。       |

### 例

以下の例は、Icebergクラスタからデータをクエリするために、`iceberg_catalog_hms`または`iceberg_catalog_glue`というIcebergカタログを作成します。これは、使用しているメタストアのタイプに依存します。

#### HDFS

ストレージとしてHDFSを使用する場合、次のようなコマンドを実行してください。

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

##### インスタンスプロファイルベースの認証を選択した場合

- IcebergクラスタでHiveメタストアを使用する場合、次のようなコマンドを実行してください:

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

- Amazon EMR IcebergクラスタでAWS Glueを使用する場合、次のようなコマンドを実行してください:

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

##### 仮定されたロールベースの認証を選択した場合

- IcebergクラスタでHiveメタストアを使用する場合、次のようなコマンドを実行してください:

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

- Amazon EMR IcebergクラスタでAWS Glueを使用する場合、次のようなコマンドを実行してください:

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

- IcebergクラスタでHiveメタストアを使用する場合、次のようなコマンドを実行してください:

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

- Amazon EMR IcebergクラスタでAWS Glueを使用する場合、次のようなコマンドを実行してください:

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

MinIOを例にします。次のようなコマンドを実行してください。

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

- 共有キー認証方法を選択した場合、次のようなコマンドを実行してください:

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

- SASトークン認証方法を選択した場合、次のようなコマンドを実行してください:

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

- 管理されたサービスID認証方法を選択した場合、次のようなコマンドを実行してください:

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

- サービスプリンシパル認証方法を選択した場合、次のようなコマンドを実行してください:

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

- 管理対象ID認証方法を選択した場合、次のようなコマンドを実行してください:

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

- もしそのShared Key認証方式を選択する場合は、以下のようなコマンドを実行してください:

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

- もしService Principal認証方式を選択する場合は、以下のようなコマンドを実行してください:

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

- もしそのVMベースの認証方式を選択する場合は、以下のようなコマンドを実行してください:

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

- もしサービスアカウントベースの認証方式を選択する場合は、以下のようなコマンドを実行してください:

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

- もしインパーソネーションベースの認証方式を選択する場合:

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

  - もしサービスアカウントが別のサービスアカウントを偽装する場合は、以下のようなコマンドを実行してください:

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

## アイスバーグカタログを表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスター内のすべてのカタログをクエリすることができます：

```SQL
SHOW CATALOGS;
```

[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)も使用して、外部カタログの作成ステートメントをクエリすることができます。次の例では、`iceberg_catalog_glue`という名前のIcebergカタログの作成ステートメントをクエリしています：

```SQL
SHOW CREATE CATALOG iceberg_catalog_glue;
```

## Icebergカタログとその内のデータベースに切り替える

次のいずれかの方法を使用して、Icebergカタログとその内のデータベースに切り替えることができます:

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションでIcebergカタログを指定し、それから[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します:

  ```SQL
  -- 現在のセッションで指定したカタログに切り替えます:
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定します:
  USE <db_name>
  ```

- 直接[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、Icebergカタログとその内のデータベースに切り替えます:

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## アイスバーグカタログを削除する

外部カタログを削除するには、[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用します。

次の例では、`iceberg_catalog_glue`という名前のIcebergカタログを削除しています:

```SQL
DROP Catalog iceberg_catalog_glue;
```

## アイスバーグテーブルのスキーマを表示

アイスバーグテーブルのスキーマを表示するために、次のいずれかの構文を使用することができます:

- スキーマを表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## アイスバーグテーブルをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Icebergクラスター内のデータベースを表示します:

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Icebergカタログとその内のデータベースに切り替える](#switch-to-an-iceberg-catalog-and-a-database-in-it)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、指定されたデータベースの宛先テーブルをクエリします:

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## アイスバーグデータベースを作成する

StarRocksの内部カタログと同様に、Icebergカタログで[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)権限がある場合、そのIcebergカタログ内でデータベースを作成するために[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)ステートメントを使用できます。この機能はv3.1以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Icebergカタログに切り替える](#switch-to-an-iceberg-catalog-and-a-database-in-it)し、その後、次のステートメントを使用してそのカタログ内にIcebergデータベースを作成します:

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>/")]
```

`location`パラメータを使用して、データベースを作成するファイルパスを指定できます。HDFSおよびクラウドストレージの両方がサポートされています。`location`パラメータを指定しない場合、StarRocksはIcebergカタログのデフォルトファイルパスにデータベースを作成します。

`prefix`は、使用しているストレージシステムによって異なります:

| **ストレージシステム**                                         | **`Prefix`の値**                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                       | `hdfs`                                                       |
| Google GCS                                                 | `gs`                                                         |
| Azure Blob Storage                                         | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可する場合、`prefix`は`wasb`です。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可する場合、`prefix`は`wasbs`です。</li></ul> |
| Azure Data Lake Storage Gen1                               | `adl`                                                        |
| Azure Data Lake Storage Gen2                               | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可する場合、`prefix`は`abfs`です。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可する場合、`prefix`は`abfss`です。</li></ul> |
| AWS S3 やその他のS3互換ストレージ（例: MinIO）            | `s3`                                                         |

## Icebergデータベースを削除する
StarRocksの内部データベースと同様に、Icebergデータベースで[DROP](../../administration/privilege_item.md#database)権限がある場合、[DROP_DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md) ステートメントを使用してそのIcebergデータベースを削除できます。この機能はv3.1以降でサポートされています。空のデータベースのみを削除できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

Icebergデータベースを削除すると、そのデータベースのHDFSクラスタやクラウドストレージ上のファイルパスはデータベースと一緒に削除されません。

[Icebergカタログ](#switch-to-an-iceberg-catalog-and-a-database-in-it)に切り替えて、そのカタログ内のIcebergデータベースを削除するには、次のステートメントを使用します。

```SQL
DROP DATABASE <データベース名>;
```

## Icebergテーブルの作成

StarRocksの内部データベースと同様に、Icebergデータベースで[CREATE TABLE](../../administration/privilege_item.md#database)権限がある場合、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)または[CREATE TABLE AS SELECT（CTAS）](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)ステートメントを使用してそのIcebergデータベース内にテーブルを作成できます。この機能はv3.1以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Icebergカタログとその中のデータベースに切り替える](#switch-to-an-iceberg-catalog-and-a-database-in-it)と、次の構文を使用してそのデータベース内でIcebergテーブルを作成します。

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
col_name col_type [COMMENT 'コメント']
```

以下の表はパラメータを説明しています。

| **パラメータ**     | **説明**                                              |
| ----------------- |------------------------------------------------------ |
| col_name          | カラムの名前。                                                |
| col_type          | カラムのデータ型。次のデータ型がサポートされています：TINYINT、SMALLINT、INT、BIGINT、FLOAT、DOUBLE、DECIMAL、DATE、DATETIME、CHAR、VARCHAR[(length)]、ARRAY、MAP、およびSTRUCT。LARGEINT、HLL、およびBITMAPデータ型はサポートされていません。 |

> **注意**
>
> パーティションのないカラムはすべて`NULL`をデフォルト値として使用する必要があります。つまり、テーブル作成ステートメントにおいて、パーティション以外のすべてのカラムに対して`DEFAULT "NULL"`を指定する必要があります。さらに、パーティションカラムは、パーティションで使用することができず、`NULL`をデフォルト値として使用することはできません。

#### partition_desc

`partition_desc`の構文は次のとおりです。

```SQL
PARTITION BY (par_col1[, par_col2...])
```

現在、StarRocksでは[identity transforms](https://iceberg.apache.org/spec/#partitioning)のみがサポートされており、これによりStarRocksは一意のパーティション値ごとにパーティションを作成します。

> **注意**
>
> パーティションカラムはパーティションで使用することができず、FLOAT、DOUBLE、DECIMAL、およびDATETIMEを除くすべてのデータ型をサポートしており、デフォルト値として`NULL`を使用することはできません。

#### PROPERTIES

`PROPERTIES`で「"key" = "value"」形式でテーブル属性を指定できます。[Icebergテーブル属性](https://iceberg.apache.org/docs/latest/configuration/)を参照してください。

次の表はいくつかの主なプロパティについて説明しています。

| **プロパティ**      | **説明**                                              |
| ----------------- | ------------------------------------------------------------ |
| location          | Icebergテーブルを作成するファイルパス。HMSをメタストアとして使用する場合、現在のIcebergカタログのデフォルトファイルパスにテーブルを作成するため、`location`パラメータを指定する必要はありません。AWS Glueをメタストアとして使用する場合：<ul><li>テーブルを作成したいデータベースの`location`パラメータを指定した場合、テーブルの`location`パラメータを指定する必要はありません。そのため、テーブルはそれが属するデータベースのファイルパスにデフォルトで作成されます。 </li><li>データベースの`location`を指定しなかった場合、テーブルの`location`パラメータを指定する必要があります。</li></ul> |
| file_format       | Icebergテーブルのファイル形式。Parquet形式のみがサポートされています。デフォルト値：`parquet`。 |
| compression_codec | Icebergテーブルで使用される圧縮アルゴリズム。対応する圧縮アルゴリズムはSNAPPY、GZIP、ZSTD、およびLZ4です。デフォルト値：`gzip`。 |

### 例

1. `unpartition_tbl`という名前のパーティションされていないテーブルを作成します。テーブルは以下のように、`id`と`score`の2つのカラムで構成されています。

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. `partition_tbl_1`という名前のパーティションされたテーブルを作成します。テーブルは`action`、`id`、および`dt`の3つのカラムで構成されており、`id`と`dt`がパーティションカラムとして定義されています。

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. `partition_tbl_1`という既存のテーブルをクエリし、そのクエリ結果を基に`partition_tbl_2`という名前のパーティションされたテーブルを作成します。`partition_tbl_2`の場合、`id`と`dt`がパーティションカラムとして定義されています。

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (id, dt)
   AS SELECT * from employee;
   ```

## Icebergテーブルへのデータのシンク

StarRocksの内部テーブルと同様に、Icebergテーブルで[INSERT](../../administration/privilege_item.md#table)権限がある場合、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ステートメントを使用してStarRocksテーブルのデータをそのIcebergテーブルにシンクできます（現在はParquet形式のIcebergテーブルのみがサポートされています）。この機能はv3.1以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Icebergカタログおよびそのデータベースに切り替えて](#switch-to-an-iceberg-catalog-and-a-database-in-it)、次の構文を使用してそのデータベース内のParquet形式のIcebergテーブルにStarRocksテーブルのデータをシンクします。

### 構文

```SQL
INSERT {INTO | OVERWRITE} <table_name>
(column_name [, ...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータをシンクしたい場合は、次の構文を使用してください：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

> **注意**
>
> パーティション列では`NULL`値は許可されません。そのため、Icebergテーブルのパーティション列に空の値がロードされないようにする必要があります。
| expression  | 宛先カラムに値を割り当てる式。                                   |
| DEFAULT     | 宛先カラムにデフォルト値を割り当てます。                                         |
| query       | Iceberg テーブルに読み込むクエリ文。StarRocks でサポートされている任意の SQL ステートメントを使用できます。 |
| PARTITION   | データを読み込むパーティション。Iceberg テーブルのすべてのパーティション列をこのプロパティに指定する必要があります。このプロパティで指定するパーティション列は、テーブル作成ステートメントで定義したパーティション列とは異なる順序で指定できます。このプロパティを指定すると、`column_name` プロパティを指定できません。 |

### 例

1. `partition_tbl_1` テーブルにデータ行を 3 つ挿入します:

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. 単純な計算を含む SELECT クエリの結果を `partition_tbl_1` テーブルに挿入します:

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. `partition_tbl_1` テーブルからデータを読み取る SELECT クエリの結果を同じテーブルに挿入します:

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. 2 つの条件 `dt='2023-09-01'` および `id=1` に一致するパーティションに SELECT クエリの結果を挿入します:

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT INTO partition_tbl_2 partition(dt='2023-09-01',id=1) SELECT 'order';
   ```

5. 2 つの条件 `dt='2023-09-01'` および `id=1` に一致するパーティション内のすべての `action` 列の値を `close` に上書きします:

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT OVERWRITE partition_tbl_1 partition(dt='2023-09-01',id=1) SELECT 'close';
   ```

## Iceberg テーブルの削除

StarRocks の内部テーブルと同様に、Iceberg テーブルに [DROP](../../administration/privilege_item.md#table) 権限がある場合、[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) ステートメントを使用してその Iceberg テーブルを削除できます。この機能は v3.1 以降でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) および [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して権限を付与および取り消すことができます。

Iceberg テーブルを削除すると、テーブルのファイルパスと HDFS クラスターまたはクラウドストレージ上のデータは削除されません。

Iceberg テーブルを強制的に削除する場合（つまり、DROP TABLE ステートメントで `FORCE` キーワードが指定されている場合）、テーブルのデータは HDFS クラスターまたはクラウドストレージ上で削除されますが、テーブルのファイルパスは保持されます。

[Iceberg カタログとその中のデータベースに切り替える](#switch-to-an-iceberg-catalog-and-a-database-in-it) と、そのデータベースで次のステートメントを使用して Iceberg テーブルを削除します。

```SQL
DROP TABLE <table_name> [FORCE];
```

## メタデータキャッシュの構成

Iceberg クラスターのメタデータファイルは、AWS S3 や HDFS などのリモートストレージに保存される場合があります。デフォルトでは、StarRocks は Iceberg メタデータをメモリにキャッシュします。クエリを加速するために、StarRocks はメタデータのキャッシュをメモリとディスクの両方に採用しています。初期クエリごとに、StarRocks はその計算結果をキャッシュします。以前のクエリと同じ意味である後続のクエリが発行された場合、StarRocks はまずそのリクエストされたメタデータをキャッシュから取得し、メタデータをキャッシュでヒットできない場合にのみリモートストレージからメタデータを取得します。

StarRocks は、メタデータのキャッシュと削除に Least Recently Used（LRU）アルゴリズムを使用します。基本的なルールは次のとおりです:

- StarRocks はまずメモリから要求されたメタデータを取得しようとします。メモリでメタデータをヒットできない場合、StarRocks はディスクからメタデータを取得しようとします。ディスクから取得したメタデータはメモリにロードされます。ディスクでもメタデータをヒットできない場合、StarRocks はリモートストレージからメタデータを取得し、取得したメタデータをメモリにキャッシュします。
- StarRocks はメモリから追い出されたメタデータをディスクに書き込みますが、ディスクから追い出されたメタデータは直接破棄します。

以下の表は、Iceberg メタデータキャッシングメカニズムを構成するために使用できる FE 構成項目を説明しています。

| **構成項目**                                      | **単位** | **デフォルト値**                                  | **説明**                                                     |
| ------------------------------------------------ | -------- | ------------------------------------------------- | ------------------------------------------------------------ |
| enable_iceberg_metadata_disk_cache               | N/A      | `false`                                           | ディスクキャッシュを有効にするかどうかを指定します。                  |
| iceberg_metadata_cache_disk_path                 | N/A      | `StarRocksFE.STARROCKS_HOME_DIR + "/caches/iceberg"` | ディスク上のキャッシュされたメタデータファイルの保存パス。              |
| iceberg_metadata_disk_cache_capacity             | バイト   | `2147483648`（2 GB と同等）                        | ディスク上で許容されるキャッシュされたメタデータの最大サイズ。            |
| iceberg_metadata_memory_cache_capacity           | バイト   | `536870912`（512 MB と同等）                       | メモリに許容されるキャッシュされたメタデータの最大サイズ。               |
| iceberg_metadata_memory_cache_expiration_seconds | 秒      | `86500`                                           | メモリ内のキャッシュエントリが最後にアクセスされてからの経過時間。      |
| iceberg_metadata_disk_cache_expiration_seconds   | 秒      | `604800`（1 週間と同等）                          | ディスク上のキャッシュエントリが最後にアクセスされてからの経過時間。     |
| iceberg_metadata_cache_max_entry_size            | バイト   | `8388608`（8 MB と同等）                          | キャッシュできるファイルの最大サイズ。このパラメータの値を超えるサイズのファイルはキャッシュできません。クエリがこれらのファイルを要求した場合、StarRocks はそれらをリモートストレージから取得します。 |