---
displayed_sidebar: "Japanese"
---

# 統一カタログ

統一カタログとは、StarRocksがv3.2以降で提供する外部カタログの一種であり、Apache Hive™、Apache Iceberg、Apache Hudi、およびDelta Lakeなどのさまざまなデータソースからテーブルを取り扱うためのものです。これにより、統合データソースとして処理することなく、Hive、Iceberg、Hudi、およびDelta Lakeに格納されているデータを直接クエリできます。統一カタログを使用すると、以下の操作が可能です。

- 手動でテーブルを作成する必要なく、Hive、Iceberg、Hudi、Delta Lakeに格納されたデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) や非同期マテリアライズド ビュー (v2.5以降でサポート) を使用して、Hive、Iceberg、Hudi、およびDelta Lakeに格納されたデータをStarRocksにロードできます。
- StarRocksでHiveおよびIcebergのデータベースおよびテーブルを作成または削除できます。

統一データソース上でのSQLワークロードの成功を確実にするためには、StarRocksクラスタは2つの重要なコンポーネントと統合する必要があります。 

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCS、またはその他のS3互換ストレージシステム（MinIOなど）などのオブジェクトストレージ
- HiveメタストアまたはAWS Glueなどのメタストア

  > **注意**
  >
  > ストレージとしてAWS S3を選択した場合、メタストアとしてHMSまたはAWS Glueを使用できます。その他のストレージシステムを選択した場合、メタストアとしてHMSのみを使用できます。

## 制限事項

1つの統一カタログは、単一のストレージシステムおよび単一のメタストアサービスとの統合をサポートします。したがって、StarRocksと統合して統一データソースとして統合したいすべてのデータソースが同じストレージシステムとメタストアサービスを使用することを確認してください。

## 使用上の注意

- サポートされているファイル形式やデータ型を理解するために、[Hiveカタログ](../../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../../data_source/catalog/hudi_catalog.md)、および[Delta Lakeカタログ](../../data_source/catalog/deltalake_catalog.md)の「使用上の注意」セクションを参照してください。

- フォーマット固有の操作は特定のテーブル形式のみをサポートします。例えば、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) および [DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) はHiveとIcebergのみをサポートしており、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) はHiveとHudiのみをサポートしています。

  [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントを使用して統一カタログ内でテーブルを作成する場合、`ENGINE`パラメータを使用してテーブル形式（HiveまたはIceberg）を指定してください。

## 統合準備

統一カタログを作成する前に、StarRocksクラスタが統一データソースのストレージシステムとメタストアと統合できることを確認してください。

### AWS IAM

ストレージとしてAWS S3を使用するか、メタストアとしてAWS Glueを使用する場合、適切な認証方法を選択し、StarRocksクラスタが関連するAWSクラウドリソースにアクセスできるように必要な準備を行ってください。詳細については、[AWSリソースへの認証 - 準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとしてHDFSを選択する場合は、StarRocksクラスタを以下のように構成してください。

- （オプション）HDFSクラスタおよびHiveメタストアにアクセスするために使用されるユーザー名を設定してください。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスタおよびHiveメタストアにアクセスします。`export HADOOP_USER_NAME="<ユーザー名>"`を各FEの **fe/conf/hadoop_env.sh** ファイルおよび各BEの **be/conf/hadoop_env.sh** ファイルの先頭に追加してユーザー名を設定できます。これらのファイルにユーザー名を設定した後は、各FEと各BEを再起動してパラメータ設定を有効にしてください。StarRocksクラスタごとに1つのユーザー名のみを設定できます。
- クエリを実行する際、StarRocksクラスタのFEおよびBEはHDFSクライアントを使用してHDFSクラスタにアクセスします。ほとんどの場合、この目的を達成するためにStarRocksクラスタを構成する必要はありません。StarRocksはデフォルトの構成を使用してHDFSクライアントを起動します。StarRocksクラスタを構成する必要があるのは以下の場合のみです。
  - HDFSクラスタで高可用性（HA）が有効になっている場合：HDFSクラスタの **hdfs-site.xml** ファイルを各FEの **$FE_HOME/conf** パスおよび各BEの **$BE_HOME/conf** パスに追加してください。
  - HDFSクラスタでView File System（ViewFs）が有効になっている場合：HDFSクラスタの **core-site.xml** ファイルを各FEの **$FE_HOME/conf** パスおよび各BEの **$BE_HOME/conf** パスに追加してください。

> **注意**
>
> クエリを送信する際に不明なホストを示すエラーが返された場合、HDFSクラスタノードのホスト名とIPアドレスのマッピングを **/etc/hosts** パスに追加する必要があります。

### Kerberos認証

HDFSクラスタまたはHiveメタストアでKerberos認証が有効になっている場合は、StarRocksクラスタを以下のように構成してください。

- 各FEおよび各BEで `kinit -kt keytab_path principal` コマンドを実行して、キータブ配布センター（KDC）からのチケット発行チケット（TGT）を取得してください。このコマンドを実行するには、HDFSクラスタおよびHiveメタストアにアクセスする権限が必要です。このコマンドでKDCにアクセスするのは時間的に感じるため、このコマンドを定期的に実行するためにcronを使用する必要があります。
- 各FEの **$FE_HOME/conf/fe.conf** ファイルおよび各BEの **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加してください。この例では、`/etc/krb5.conf` が krb5.conf ファイルの保存パスです。必要に応じてパスを変更することができます。

## 統一カタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <カタログ名>
[COMMENT <コメント>]
PROPERTIES
(
    "type" = "unified",
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### パラメータ

#### カタログ名

統一カタログの名前。命名規則は以下の通りです。

- 名前には文字、数字（0-9）、アンダースコア (_) を含めることができます。文字で始まる必要があります。
- 名前は大文字と小文字を区別し、長さが1023文字を超えることはできません。

#### コメント

統一カタログの説明。このパラメータは任意です。

#### タイプ

データソースのタイプ。値を `unified` に設定してください。

#### MetastoreParams

StarRocksがメタストアと統合する方法についてのパラメータのセット。

##### Hiveメタストア

統一データソースのメタストアとしてHiveメタストアを選択した場合、`MetastoreParams` を以下のように構成してください。

```SQL
"unified.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_URI>"
```

> **注意**
>
> データをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを **/etc/hosts** パスに追加する必要があります。さもないと、クエリを開始した際にStarRocksのHiveメタストアへのアクセスが失敗する可能性があります。

以下の表は、`MetastoreParams` で構成する必要のあるパラメータについて説明しています。

| パラメータ              | 必須      | 説明                                                        |
| ------------------------ | ---------- | ------------------------------------------------------- |
| unified.metastore.type | はい      | 統一データソースで使用するメタストアのタイプ。値を `hive` に設定してください。 |
| hive.metastore.uris    | はい      | HiveメタストアのURI。フォーマット：`thrift://<メタストア_IP_アドレス>:<メタストア_ポート>`。Hiveメタストアで高可用性（HA）が有効になっている場合は、複数のメタストアURIを指定し、カンマ (`,`) で区切ってください。例: `"thrift://<メタストア_IP_アドレス_1>:<メタストア_ポート_1>,thrift://<メタストア_IP_アドレス_2>:<メタストア_ポート_2>,thrift://<メタストア_IP_アドレス_3>:<メタストア_ポート_3>"`。 |

##### AWS Glue

データソースのメタストアとしてAWS S3を選択した場合、AWS Glueをメタストアとして使用する場合、以下のアクションのうちいずれかを実行してください。

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams` を以下のように構成してください:

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 仮定されるロールベースの認証方法を選択する場合、`MetastoreParams` を以下のように構成してください:

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAMユーザーベースの認証方法を選択する場合、`MetastoreParams` を以下のように構成してください:
```SQL
"unified.metastore.type" = "glue",
"aws.glue.use_instance_profile" = "false",
"aws.glue.access_key" = "<iam_user_access_key>",
"aws.glue.secret_key" = "<iam_user_secret_key>",
"aws.glue.region" = "<aws_s3_region>"
```

以下の表では、`MetastoreParams`で構成する必要のあるパラメータを説明します。

| パラメータ                       | 必須    | 説明                                                         |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| unified.metastore.type        | はい      | 統合データソースで使用するメタストアのタイプ。値を `glue` に設定します。 |
| aws.glue.use_instance_profile | はい      | インスタンスプロファイルベースの認証方法と、仮定される役割ベースの認証方法を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.glue.iam_role_arn         | いいえ    | AWS Glue Data Catalogの特権を持つIAMロールのARN。AWS Glueへのアクセスに仮定される役割ベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.region               | はい     | AWS Glue Data Catalogが存在する地域。例: `us-west-1`。 |
| aws.glue.access_key           | いいえ     | AWS IAMユーザーのアクセスキー。AWS GlueへのアクセスにIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | いいえ    | AWS IAMユーザーのシークレットキー。AWS GlueへのアクセスにIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |

AWS Glueへのアクセスの認証方法の選択およびAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS Glueへのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocksがストレージシステムと統合する方法に関するパラメータのセット。このパラメータセットはオプションです。

ストレージとしてHDFSを使用する場合、`StorageCredentialParams`を構成する必要はありません。

ストレージとしてAWS S3、その他のS3互換のストレージシステム、Microsoft Azure Storage、またはGoogle GCSを使用する場合は、`StorageCredentialParams`を構成する必要があります。

##### AWS S3

ストレージとしてAWS S3を選択した場合、次のアクションのいずれかを実行してください。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 仮定される役割ベースの認証方法を選択する場合、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーベースの認証方法を選択する場合、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表では、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

| パラメータ                   | 必須       | 説明                                                                 |
| --------------------------- | ----------- | -------------------------------------------------------------------- |
| aws.s3.use_instance_profile | はい       | インスタンスプロファイルベースの認証方法と仮定される役割ベースの認証方法を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | いいえ      | AWS S3バケットに特権を持つIAMロールのARN。仮定される役割ベースの認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。  |
| aws.s3.region               | はい       | AWS S3バケットが存在する地域。例: `us-west-1`。 |
| aws.s3.access_key           | いいえ      | IAMユーザーのアクセスキー。IAMユーザーベースの認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ      | IAMユーザーのシークレットキー。IAMユーザーベースの認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3へのアクセスの認証方法の選択およびAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS S3へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換のストレージシステム

MinIOなどのS3互換のストレージシステムを選択した場合、次のように`StorageCredentialParams`を構成して統合を成功させるようにしてください：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表では、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

| パラメータ                       | 必須    | 説明                                                              |
| -------------------------------- | -------- | ----------------------------------------------------------------- |
| aws.s3.enable_ssl                | はい      | SSL接続を有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access  | はい      | パス形式のアクセスを有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値: `false`。MinIOの場合、値を`true`に設定する必要があります。<br />パス形式のURLは次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、US West（Oregon）リージョンで`DOC-EXAMPLE-BUCKET1`というバケットを作成し、そのバケット内の`alice.jpg`オブジェクトにアクセスする場合、次のパス形式のURLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | はい      | AWS S3の代わりにS3互換のストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key                | はい      | IAMユーザーのアクセスキー。 |
| aws.s3.secret_key                | はい      | IAMユーザーのシークレットキー。 |

##### Microsoft Azure Storage

###### Azure Blob Storage

Blob Storageをストレージとして選択した場合、次のアクションのいずれかを実行してください：

- Shared Key認証方法を選択する場合、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下の表では、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**              | **必須** | **説明**                           |
  | -------------------------- | -------- | --------------------------------- |
  | azure.blob.storage_account | はい      | Blob Storageアカウントのユーザー名。  |
  | azure.blob.shared_key      | はい      | Blob Storageアカウントの共有キー。 |

- SAS Token認証方法を選択する場合、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  以下の表では、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**             | **必須** | **説明**                           |
  | ------------------------- | -------- | --------------------------------- |
  | azure.blob.account_name   | はい      | Blob Storageアカウントのユーザー名。  |
  | azure.blob.container_name | はい      | データを保存しているBlobコンテナの名前。 |
  | azure.blob.sas_token      | はい      | Blob Storageアカウントにアクセスするために使用されるSASトークン。|

###### Azure Data Lake Storage Gen1

Data Lake Storage Gen1を選択した場合、次のアクションのいずれかを実行してください：

- Managed Service Identity認証メソッドを選択する場合、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下の表では、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

  | **パラメータ**                            | **必須** | **説明**                                                              |
  | ---------------------------------------- | -------- | --------------------------------------------------------------------- |
  | azure.adls1.use_managed_service_identity | はい      | Managed Service Identity認証メソッドを有効にするかどうかを指定します。値を `true` に設定してください。|

- Service Principal認証メソッドを選択する場合、`StorageCredentialParams`を以下のように構成します：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```


```
`StorageCredentialParams`で構成する必要のあるパラメータを次の表に示します。

| **パラメータ**                   | **必須** | **説明**                                              |
| ------------------------------ | -------- | --------------------------------------------------- |
| azure.adls1.oauth2_client_id   | Yes      | サービスプリンシパルのクライアント（アプリケーション）ID。      |
| azure.adls1.oauth2_credential  | Yes      | 作成した新しいクライアント（アプリケーション）シークレットの値。     |
| azure.adls1.oauth2_endpoint    | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0 トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2

ストレージとしてData Lake Storage Gen2を選択する場合、次のいずれかのアクションを実行します。

- マネージドアイデンティティ認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します。

```SQL
"azure.adls2.oauth2_use_managed_identity" = "true",
"azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
"azure.adls2.oauth2_client_id" = "<service_client_id>"
```

`StorageCredentialParams`で構成する必要のあるパラメータを次の表に示します。

| **パラメータ**                          | **必須** | **説明**                                              |
| ------------------------------------- | -------- | --------------------------------------------------- |
| azure.adls2.oauth2_use_managed_identity | Yes      | マネージドアイデンティティ認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |
| azure.adls2.oauth2_tenant_id           | Yes      | アクセスしたいテナントのID。                    |
| azure.adls2.oauth2_client_id           | Yes      | マネージドアイデンティティのクライアント（アプリケーション）ID。    |

- 共有キー認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します。

```SQL
"azure.adls2.storage_account" = "<storage_account_name>",
"azure.adls2.shared_key" = "<shared_key>"
```

`StorageCredentialParams`で構成する必要のあるパラメータを次の表に示します。

| **パラメータ**              | **必須** | **説明**                                              |
| ------------------------ | -------- | --------------------------------------------------- |
| azure.adls2.storage_account | Yes      | Data Lake Storage Gen2ストレージアカウントのユーザー名。   |
| azure.adls2.shared_key    | Yes      | Data Lake Storage Gen2ストレージアカウントの共有キー。    |

- サービスプリンシパル認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します。

```SQL
"azure.adls2.oauth2_client_id" = "<service_client_id>",
"azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
"azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
```

`StorageCredentialParams`で構成する必要のあるパラメータを次の表に示します。

| **パラメータ**                   | **必須** | **説明**                                              |
| ------------------------------- | -------- | --------------------------------------------------- |
| azure.adls2.oauth2_client_id     | Yes      | サービスプリンシパルのクライアント（アプリケーション）ID。      |
| azure.adls2.oauth2_client_secret | Yes      | 作成した新しいクライアント（アプリケーション）シークレットの値。     |
| azure.adls2.oauth2_client_endpoint | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0 トークンエンドポイント（v1）。 |

##### Google GCS

ストレージとしてGoogle GCSを選択する場合、次のいずれかのアクションを実行します。

- VMベースの認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します。

```SQL
"gcp.gcs.use_compute_engine_service_account" = "true"
```

`StorageCredentialParams`で構成する必要のあるパラメータを次の表に示します。

| **パラメータ**                                | **デフォルト値** | **値の例**               | **説明**                                              |
| -------------------------------------------- | -------------- | --------------------- | --------------------------------------------------- |
| gcp.gcs.use_compute_engine_service_account | false          | true                  | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します。

```SQL
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

`StorageCredentialParams`で構成する必要のあるパラメータを次の表に示します。

| **パラメータ**                    | **デフォルト値** | **値の例**                                        | **説明**                                              |
| -------------------------------- | -------------- | ------------------------------------------------ | --------------------------------------------------- |
| gcp.gcs.service_account_email    | ""             | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウント作成時に生成されたJSONファイルの電子メールアドレス。   |
| gcp.gcs.service_account_private_key_id | ""             | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"        | サービスアカウント作成時に生成されたJSONファイルのプライベートキーID。     |
| gcp.gcs.service_account_private_key | ""             | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | サービスアカウント作成時に生成されたJSONファイルのプライベートキー。      |

- 偽装ベースの認証メソッドを選択するには、`StorageCredentialParams`を次のように構成します。

  - VMインスタンスがサービスアカウントを偽装する場合：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams`で構成する必要のあるパラメータを次の表に示します。

    | **パラメータ**                              | **デフォルト値** | **値の例**               | **説明**                                              |
    | ------------------------------------------ | -------------- | --------------------- | --------------------------------------------------- |
    | gcp.gcs.use_compute_engine_service_account | false          | true                  | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""             | "hello"               | 偽装したいサービスアカウント。                               |

  - サービスアカウント（一時的にメタサービスアカウントとして命名）が別のサービスアカウント（一時的にデータサービスアカウントとして命名）を偽装する場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    `StorageCredentialParams`で構成する必要のあるパラメータを次の表に示します。

    | **パラメータ**                    | **デフォルト値** | **値の例**                                        | **説明**                                              |
    | -------------------------------- | -------------- | ------------------------------------------------ | --------------------------------------------------- |
    | gcp.gcs.service_account_email    | ""             | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウント作成時に生成されたJSONファイルの電子メールアドレス。  |
    | gcp.gcs.service_account_private_key_id | ""             | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"        | メタサービスアカウント作成時に生成されたJSONファイルのプライベートキーID。    |
    | gcp.gcs.service_account_private_key | ""             | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | メタサービスアカウント作成時に生成されたJSONファイルのプライベートキー。     |
    | gcp.gcs.impersonation_service_account | ""             | "hello"                | 偽装したいデータサービスアカウント。                              |

#### MetadataUpdateParams

StarRocksがHive、Hudi、Delta Lakeのキャッシュされたメタデータを更新する方法に関するパラメータのセット。このパラメータセットはオプションです。Hive、Hudi、Delta Lakeのキャッシュされたメタデータを更新するポリシーについての詳細については、[Hiveカタログ](../../data_source/catalog/hive_catalog.md)、[Hudiカタログ](../../data_source/catalog/hudi_catalog.md)、[Delta Lakeカタログ](../../data_source/catalog/deltalake_catalog.md)を参照してください。

ほとんどの場合、`MetadataUpdateParams`を無視し、それに含まれるポリシーパラメータを調整する必要はありません。なぜなら、これらのパラメータのデフォルト値はすでに即時に使用可能なパフォーマンスを提供しているからです。

ただし、Hive、Hudi、Delta Lakeでデータが頻繁に更新される場合は、これらのパラメータを調整して自動非同期更新のパフォーマンスをさらに最適化することができます。

| パラメータ                   | 必須 | 説明                                        |
| --------------------------- | ---- | ----------------------------------------- |
| enable_metastore_cache      | No   | StarRocksがHive、Hudi、またはDelta Lakeテーブルのメタデータをキャッシュするかどうかを指定します。有効な値は`true`および`false`です。デフォルト値は`true`です。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
```
| enable_remote_file_cache               | いいえ     | StarRocksがHive、Hudi、またはDelta Lakeのテーブルやパーティションの基礎データファイルのメタデータをキャッシュするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`true`。trueの値はキャッシュを有効にし、falseの値はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | いいえ       | StarRocksが自身でキャッシュしたHive、Hudi、またはDelta Lakeのテーブルやパーティションのメタデータを非同期に更新する時間間隔。単位：秒。デフォルト値：`7200`、つまり2時間。 |
| remote_file_cache_refresh_interval_sec | いいえ       | StarRocksが自身でキャッシュしたHive、Hudi、またはDelta Lakeのテーブルやパーティションの基礎データファイルのメタデータを非同期に更新する時間間隔。単位：秒。デフォルト値：`60`。 |
| metastore_cache_ttl_sec                | いいえ       | StarRocksが自身でキャッシュしたHive、Hudi、またはDelta Lakeのテーブルやパーティションのメタデータを自動的に破棄する時間間隔。単位：秒。デフォルト値：`86400`、つまり24時間。 |
| remote_file_cache_ttl_sec              | いいえ       | StarRocksが自身でキャッシュしたHive、Hudi、またはDelta Lakeのテーブルやパーティションの基礎データファイルのメタデータを自動的に破棄する時間間隔。単位：秒。デフォルト値：`129600`、つまり36時間。 |

### 例

以下の例は、統合されたデータソースからデータをクエリするために、「unified_catalog_hms」または「unified_catalog_glue」という統一されたカタログを作成します。使用するメタストアの種類によって異なります。

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

##### 仮定されたロールベースの認証

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

例としてMinIOを使用する場合、次のようなコマンドを実行します:

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

#### Microsoft Azureストレージ

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

- 管理されたサービスアイデンティティ認証方法を選択する場合、次のようなコマンドを実行します:

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
- マネージドID認証方式を選択する場合、以下のようにコマンドを実行してください：

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

- 共有キー認証方式を選択する場合、以下のようにコマンドを実行してください：

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

- サービスプリンシパル認証方式を選択する場合、以下のようにコマンドを実行してください：

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

- VMベースの認証方法を選択する場合、以下のようにコマンドを実行してください：

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

- サービスアカウントベースの認証方法を選択する場合、以下のようにコマンドを実行してください：

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

- 偽装ベースの認証方法を選択する場合：

  - VMインスタンスがサービスアカウントを偽装する場合、以下のようにコマンドを実行してください：

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

  - サービスアカウントが別のサービスアカウントを偽装する場合、以下のようにコマンドを実行してください：

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

## 統合カタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) を使用して、現在のStarRocksクラスタに存在するすべてのカタログを照会できます：

```SQL
SHOW CATALOGS;
```

統合カタログ「unified_catalog_glue」の作成ステートメントをクエリするには、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を使用できます。次の例は、名前が「unified_catalog_glue」という統合カタログの作成ステートメントを照会しています：

```SQL
SHOW CREATE CATALOG unified_catalog_glue;
```

## 統合カタログとその中のデータベースに切り替える

次の方法のいずれかを使用して、統合カタログとその中のデータベースに切り替えることができます：

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) を使用して、現在のセッションで統合カタログを指定し、[USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用してアクティブなデータベースを指定します：

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替える：
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定する：
  USE <db_name>
  ```

- 直接的に [USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、統合カタログとその中のデータベースに切り替えます：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 統合カタログの削除

外部カタログを削除するには、[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) を使用できます。

次の例では、「unified_catalog_glue」という名前の統合カタログを削除しています：

```SQL
DROP CATALOG unified_catalog_glue;
```

## 統合カタログからテーブルのスキーマを表示する

統合カタログからテーブルのスキーマを表示するには、次の構文のいずれかを使用できます：

- スキーマの表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## 統合カタログからデータをクエリする

統合カタログからデータをクエリするには、次の手順に従います：

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を使用して、統合データソースに関連付けられた統合カタログのデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [統合カタログとその中のデータベースに切り替える](#統合カタログとその中のデータベースに切り替える)。

3. 指定されたデータベースの宛先テーブルをクエリするには、[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用します：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hive、Iceberg、Hudi、またはDelta Lakeからデータを読み込む

[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用して、Hive、Iceberg、Hudi、またはDelta Lakeのデータを、統合カタログ内で作成されたStarRocksテーブルに読み込むことができます。

次の例は、統合カタログ「unified_catalog」に属するデータベース「test_database」内で作成されたStarRocksテーブル「test_tbl」に、Hiveテーブル「hive_table」のデータを読み込むものです：

```SQL
INSERT INTO unified_catalog.test_database.test_table SELECT * FROM hive_table
```

## 統合カタログ内にデータベースを作成する

StarRocksの内部カタログと同様に、統合カタログ上で [CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) ステートメントを使用してデータベースを作成できます。これには、統合カタログ上での[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) 権限が必要です。

> **注**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) および [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して権限を付与および取り消すことができます。

StarRocksは、統合カタログにHiveおよびIcebergデータベースのみを作成することができます。

[統合カタログとその中のデータベースに切り替える](#統合カタログとその中のデータベースに切り替える) ことを忘れずに、次のステートメントを使用して統合カタログ上にデータベースを作成してください：

```SQL
CREATE DATABASE <database_name>
[properties ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location` パラメータは、作成するデータベースのファイルパスを指定します。これはHDFSまたはクラウドストレージに存在することができます。

- データソースのメタストアとしてHiveメタストアを使用している場合、データベース作成時にパラメータを指定しない場合は、`location` パラメータのデフォルト値は `<warehouse_location>/<database_name.db>` となります。
- データソースのメタストアとしてAWS Glueを使用する場合、`location` パラメータにはデフォルト値がないため、データベース作成時にそのパラメータを指定する必要があります。

使用するストレージシステムによって、`prefix` が異なります:

| **ストレージシステム**                                 | **`Prefix`の値**                                               |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                 | `hdfs`                                                       |
| Google GCS                                           | `gs`                                                         |
| Azure Blob Storage                                   | <ul><li>ストレージアカウントがHTTP経由でのアクセスを許可する場合、`prefix` は `wasb` です。</li><li>ストレージアカウントがHTTPS経由でのアクセスを許可する場合、`prefix` は `wasbs` です。</li></ul> |
| Azure Data Lake Storage Gen1                         | `adl`                                                        |
| Azure Data Lake Storage Gen2                         | <ul><li>ストレージアカウントがHTTP経由でのアクセスを許可する場合、`prefix` は `abfs` です。</li><li>ストレージアカウントがHTTPS経由でのアクセスを許可する場合、`prefix` は `abfss` です。</li></ul> |
| AWS S3またはその他のS3互換ストレージ（例: MinIO） | `s3`                                                         |

## 統合カタログからデータベースを削除する

StarRocksの内部データベースと同様に、統合カタログ内で作成されたデータベースに[DROP](../../administration/privilege_item.md#database)権限がある場合、[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md) ステートメントを使用してそのデータベースを削除できます。空のデータベースのみを削除できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksは、統合カタログからHiveおよびIcebergデータベースのみを削除することができます。

統合カタログからデータベースを削除する際は、そのデータベースのHDFSクラスタまたはクラウドストレージ上のファイルパスはデータベースと一緒に削除されません。

[統合カタログに切り替え](#switch-to-a-unified-catalog-and-a-database-in-it)し、そのカタログ内で以下のステートメントを使用してデータベースを削除します：

```SQL
DROP DATABASE <database_name>
```

## 統合カタログ内でテーブルを作成する

StarRocksの内部データベースと同様に、統合カタログ内で作成されたデータベースに[CREATE TABLE](../../administration/privilege_item.md#database)権限がある場合、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)または[CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)ステートメントを使用してそのデータベース内にテーブルを作成できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksは、統合カタログ内でHiveおよびIcebergテーブルのみを作成することができます。

[Hiveカタログに切り替え](#switch-to-a-unified-catalog-and-a-database-in-it)し、そのデータベース内にHiveまたはIcebergテーブルを作成するために[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を使用します：

```SQL
CREATE TABLE <table_name>
(column_definition1[, column_definition2, ...]
ENGINE = {|hive|iceberg}
[partition_desc]
```

詳細については、[Hiveテーブルの作成](../catalog/hive_catalog.md#create-a-hive-table)および[Icebergテーブルの作成](../catalog/iceberg_catalog.md#create-an-iceberg-table)を参照してください。

以下の例では、`hive_table`という名前のHiveテーブルが作成されます。テーブルは`action`、`id`、`dt`の3つのカラムから構成され、`id`と`dt`はパーティションカラムです。

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

## 統合カタログ内のテーブルにデータをシンクする

StarRocksの内部テーブルと同様に、統合カタログ内で作成されたテーブルに[INSERT](../../administration/privilege_item.md#table)権限がある場合、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ステートメントを使用してStarRocksテーブルのデータを統合カタログテーブルにシンクすることができます（現在はParquet形式の統合カタログテーブルのみがサポートされます）。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksは、統合カタログ内のHiveおよびIcebergテーブルのみにデータをシンクすることができます。

[Hiveカタログに切り替え](#switch-to-a-unified-catalog-and-a-database-in-it)し、そのデータベース内のHiveまたはIcebergテーブルにデータを挿入するために[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用します：

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータをシンクしたい場合、以下の構文を使用してください：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

詳細については、[Hiveテーブルにデータをシンク](../catalog/hive_catalog.md#sink-data-to-a-hive-table)および[Icebergテーブルにデータをシンク](../catalog/iceberg_catalog.md#sink-data-to-an-iceberg-table)を参照してください。

以下の例では、`hive_table`という名前のHiveテーブルに3つのデータ行が挿入されます：

```SQL
INSERT INTO hive_table
VALUES
    ("buy", 1, "2023-09-01"),
    ("sell", 2, "2023-09-02"),
    ("buy", 3, "2023-09-03");
```

## 統合カタログからテーブルを削除する

StarRocksの内部テーブルと同様に、統合カタログ内で作成されたテーブルに[DROP](../../administration/privilege_item.md#table)権限がある場合、[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)ステートメントを使用してそのテーブルを削除できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksは、統合カタログからHiveおよびIcebergテーブルのみを削除することができます。

[Hiveカタログに切り替え](#switch-to-a-unified-catalog-and-a-database-in-it)し、そのデータベース内のHiveまたはIcebergテーブルを削除するために[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)を使用します：

```SQL
DROP TABLE <table_name>
```

詳細については、[Hiveテーブルの削除](../catalog/hive_catalog.md#drop-a-hive-table)および[Icebergテーブルの削除](../catalog/iceberg_catalog.md#drop-an-iceberg-table)を参照してください。

以下の例では、`hive_table`という名前のHiveテーブルが削除されます：

```SQL
DROP TABLE hive_table FORCE
```