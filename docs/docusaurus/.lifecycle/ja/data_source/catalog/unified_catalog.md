---
displayed_sidebar: "Japanese"
---

# 統合カタログ

統合カタログは、StarRocksがv3.2以降で提供する外部カタログの一種で、Apache Hive™、Apache Iceberg、Apache Hudi、およびDelta Lakeなどのさまざまなデータソースからのテーブルを取り扱うためのもので、統合カタログを使用して、データを取り込むことなく統合データソースとして扱うことができます。統合カタログを使用すると、次のことができます。

- Hive、Iceberg、Hudi、およびDelta Lakeに格納されているデータを直接クエリできます。手動でテーブルを作成する必要はありません。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) や非同期マテリアライズド ビュー（v2.5以降でサポートされています）を使用して、Hive、Iceberg、Hudi、およびDelta Lakeに格納されているデータを処理し、StarRocksにデータをロードできます。
- StarRocksで、HiveおよびIcebergのデータベースおよびテーブルを作成または削除する操作を実行できます。

統合データソースでの成功したSQLワークロードを確実にするには、StarRocksクラスタに次の2つの重要なコンポーネントを統合する必要があります。

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCS、またはその他のS3互換のストレージシステム（例えば、MinIO）などのオブジェクトストレージ
- HiveメタストアまたはAWS Glueなどのメタストア

  > **注記**
  >
  > ストレージとしてAWS S3を選択する場合は、メタストアとしてHMSまたはAWS Glueを使用できます。他のストレージシステムを選択する場合は、メタストアとしてHMSのみを使用できます。

## 制限

統合カタログ1つは、1つのストレージシステムと1つのメタストアサービスとの統合をサポートします。したがって、StarRocksと統合して統合データソースとして統合したいすべてのデータソースが、同じストレージシステムとメタストアサービスを使用していることを確認してください。

## 使用の注意点

- サポートされているファイル形式とデータ型については、[Hiveカタログ](../../data_source/catalog/hive_catalog.md)、[Icebergカタログ](../../data_source/catalog/iceberg_catalog.md)、[Hudiカタログ](../../data_source/catalog/hudi_catalog.md)、[Delta Lakeカタログ](../../data_source/catalog/deltalake_catalog.md)の「使用の注意点」セクションを参照してください。

- フォーマット固有の操作は、特定のテーブルフォーマットのみサポートされています。たとえば、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) および [DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) はHiveおよびIcebergのみをサポートしており、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) はHiveおよびHudiのみをサポートしています。

  [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントを使用して統合カタログ内にテーブルを作成する場合は、`ENGINE`パラメータを使用してテーブル形式（HiveまたはIceberg）を指定してください。

## 統合の準備

統合カタログを作成する前に、StarRocksクラスタが統合データソースのストレージシステムおよびメタストアと統合できることを確認してください。

### AWS IAM

ストレージとしてAWS S3を使用する場合やメタストアとしてAWS Glueを使用する場合は、適切な認証方法を選択し、StarRocksクラスタが関連するAWSクラウドリソースにアクセスできるように必要な準備を行ってください。詳細は、[AWSリソースへの認証 - 準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとしてHDFSを選択する場合は、StarRocksクラスタを次のように構成してください。

- （オプション）HDFSクラスタおよびHiveメタストアにアクセスするために使用されるユーザ名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザ名を使用してHDFSクラスタおよびHiveメタストアにアクセスします。`export HADOOP_USER_NAME="<user_name>"` を各FEの **fe/conf/hadoop_env.sh** ファイルの先頭に追加し、各BEの **be/conf/hadoop_env.sh** ファイルの先頭に追加してユーザ名を設定します。これらのファイルにユーザ名を設定した後は、各FEおよび各BEを再起動してパラメータ設定が有効になるようにしてください。1つのStarRocksクラスタにつき1つのユーザ名のみを設定できます。
- データをクエリする際、StarRocksクラスタのFEおよびBEはHDFSクライアントを使用してHDFSクラスタにアクセスします。ほとんどの場合、この目的を達成するためにStarRocksクラスタを設定する必要はありません。StarRocksはデフォルトの構成でHDFSクライアントを起動します。次の状況でのみStarRocksクラスタを構成する必要があります。
  - HDFSクラスタで高可用性（HA）が有効になっている場合: HDFSクラスタの **hdfs-site.xml** ファイルを各FEの **$FE_HOME/conf** パスおよび各BEの **$BE_HOME/conf** パスに追加します。
  - HDFSクラスタでView File System（ViewFs）が有効になっている場合: HDFSクラスタの **core-site.xml** ファイルを各FEの **$FE_HOME/conf** パスおよび各BEの **$BE_HOME/conf** パスに追加します。

> **注記**
>
> クエリを送信すると不明なホストというエラーが返される場合は、HDFSクラスタノードのホスト名とIPアドレスのマッピングを **/etc/hosts** パスに追加する必要があります。

### Kerberos認証

HDFSクラスタまたはHiveメタストアでKerberos認証が有効になっている場合は、StarRocksクラスタを次のように構成してください。

- 各FEおよび各BEで `kinit -kt keytab_path principal` コマンドを実行して、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFSクラスタやHiveメタストアにアクセスする権限が必要です。このコマンドでKDCにアクセスするのは時間に制約があるため、このコマンドを定期的に実行するためにcronを使用する必要があります。
- 各FEの **$FE_HOME/conf/fe.conf** ファイルおよび各BEの **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。この例では `/etc/krb5.conf` が krb5.conf ファイルの保存パスです。必要に応じてパスを変更できます。

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

統合カタログの名前。以下の命名規則に従います。

- 名前には、文字、数字 (0-9)、およびアンダースコア (_) を含めることができます。文字で始まる必要があります。
- 名前は大文字と小文字を区別し、長さは1023文字を超えることはできません。

#### comment

統合カタログの説明。このパラメータはオプションです。

#### type

データソースのタイプ。値を `unified` に設定します。

#### MetastoreParams

お使いのメタストアとStarRocksの統合に関するパラメータのセット。

##### Hiveメタストア

統合データソースのメタストアとしてHiveメタストアを選択する場合、`MetastoreParams` を次のように構成してください。

```SQL
"unified.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> データをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを **/etc/hosts** パスに追加する必要があります。そうしないと、クエリを開始したときにStarRocksがHiveメタストアにアクセスできなくなる場合があります。

次の表に、`MetastoreParams` で構成する必要があるパラメータについて説明します。

| パラメータ              | 必須     | 説明                                                   |
| ---------------------- | -------- | ------------------------------------------------------ |
| unified.metastore.type | はい     | 統合データソースで使用するメタストアの種類。値を `hive` に設定します。 |
| hive.metastore.uris    | はい     | HiveメタストアのURI。形式: `thrift://<metastore_IP_address>:<metastore_port>`。Hiveメタストアで高可用性（HA）が有効になっている場合は、複数のメタストアURIを指定し、それらをコンマ（`,`）で区切って指定します。例: `"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

統合データソースのストレージとしてAWS S3を選択し、メタストアとしてAWS Glueを選択する場合、次のいずれかの操作を行ってください。

- インスタンスプロファイルベースの認証方法を選択する場合、「MetastoreParams」を次のように構成してください。

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 仮定される役割ベースの認証方法を選択する場合、「MetastoreParams」を次のように構成してください。

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAMユーザーベースの認証方法を選択する場合、「MetastoreParams」を次のように構成してください。

```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

次の表には、`MetastoreParams` で構成する必要があるパラメータが記載されています。

| Parameter                     | Required | Description                                                  |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| unified.metastore.type        | Yes      | 統合データソースに使用するメタストアのタイプ。値を `glue` に設定します。 |
| aws.glue.use_instance_profile | Yes      | インスタンスプロファイルベースの認証方法と、前提となるロールベースの認証を有効にするかどうかを指定します。有効な値は `true` および `false` です。デフォルト値は `false` です。 |
| aws.glue.iam_role_arn         | No       | AWS Glue Data Catalog に権限を持つ IAM ロールの ARN。AWS Glue へのアクセスに前提となるロールベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.region               | Yes      | AWS Glue Data Catalog が存在するリージョン。例: `us-west-1`。 |
| aws.glue.access_key           | No       | AWS IAM ユーザーのアクセスキー。IAM ユーザーに基づく認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | No       | AWS IAM ユーザーのシークレットキー。IAM ユーザーに基づく認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |

AWS Glue へのアクセスにおける認証方法の選択と AWS IAM コンソールでのアクセス制御ポリシーの構成方法については、[AWS Glue へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue) を参照してください。

#### StorageCredentialParams

StarRocks がストレージシステムと統合する方法に関する一連のパラメータ。このパラメータセットはオプションです。

ストレージとして HDFS を使用する場合は、`StorageCredentialParams` を構成する必要はありません。

AWS S3、その他の S3 互換ストレージシステム、Microsoft Azure Storage、または Google GCS をストレージとして使用する場合は、`StorageCredentialParams` を構成する必要があります。

##### AWS S3

ストレージとして AWS S3 を選択する場合、次のいずれかの操作を実行します。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams` を以下のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 前提となるロールベースの認証方法を選択する場合、`StorageCredentialParams` を以下のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM ユーザーに基づく認証方法を選択する場合、`StorageCredentialParams` を以下のように構成します：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

次の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

| Parameter                   | Required | Description                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes      | インスタンスプロファイルベースの認証方法と前提となるロールベースの認証方法を有効にするかどうかを指定します。有効な値は `true` および `false` です。デフォルト値は `false` です。 |
| aws.s3.iam_role_arn         | No       | AWS S3 バケットへのアクセス権限を持つ IAM ロールの ARN。前提となるロールベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。  |
| aws.s3.region               | Yes      | AWS S3 バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | No       | IAM ユーザーのアクセスキー。IAM ユーザーに基づく認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | No       | IAM ユーザーのシークレットキー。IAM ユーザーに基づく認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3 へのアクセスにおける認証方法の選択と AWS IAM コンソールでのアクセス制御ポリシーの構成方法については、[AWS S3 へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3) を参照してください。

##### S3 互換ストレージシステム

MinIO などの S3 互換ストレージシステムをストレージとして選択する場合、成功した統合を確実にするために、`StorageCredentialParams` を以下のように構成します：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

次の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

| Parameter                        | Required | Description                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | Yes      | SSL 接続を有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access  | Yes      | パススタイルアクセスを有効にするかどうかを指定します。<br />有効な値: `true` および `false`。デフォルト値: `false`。MinIO の場合、この値を `true` に設定する必要があります。<br />パススタイルの URL は次のような形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、US West (Oregon) リージョンで `DOC-EXAMPLE-BUCKET1` というバケットを作成し、そのバケット内の `alice.jpg` オブジェクトにアクセスしたい場合、次のパススタイル URL を使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | Yes      | AWS S3 の代わりに S3 互換ストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key                | Yes      | IAM ユーザーのアクセスキー。 |
| aws.s3.secret_key                | Yes      | IAM ユーザーのシークレットキー。 |

##### Microsoft Azure Storage

###### Azure Blob Storage

ストレージとして Blob Storage を選択する場合、次のいずれかの操作を実行します：

- 共有キー認証方法を選択する場合、`StorageCredentialParams` を以下のように構成します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  次の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

  | **Parameter**              | **Required** | **Description**                              |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | Yes          | Blob Storage アカウントのユーザー名。   |
  | azure.blob.shared_key      | Yes          | Blob Storage アカウントの共有キー。 |

- SAS トークン認証方法を選択する場合、`StorageCredentialParams` を以下のように構成します：

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  次の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

  | **Parameter**             | **Required** | **Description**                                              |
  | ------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.account_name   | Yes          | Blob Storage アカウントのユーザー名。                   |
  | azure.blob.container_name | Yes          | データを格納するブロブコンテナの名前。        |
  | azure.blob.sas_token      | Yes          | Blob Storage アカウントにアクセスするために使用される SAS トークン。 |

###### Azure Data Lake Storage Gen1

ストレージとして Data Lake Storage Gen1 を選択する場合、次のいずれかの操作を実行します：

- Managed Service Identity 認証方法を選択する場合、`StorageCredentialParams` を以下のように構成します：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  次の表には、`StorageCredentialParams` で構成する必要があるパラメータが記載されています。

  | **Parameter**                            | **Required** | **Description**                                              |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes          | Managed Service Identity 認証方法を有効にするかどうかを指定します。値を `true` に設定します。 |

- Service Principal 認証方法を選択する場合、`StorageCredentialParams` を以下のように構成します：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```
```
`StorageCredentialParams`で構成する必要があるパラメータを以下の表に示します。

| **パラメータ**                                 | **必須** | **説明**                                                      |
| ---------------------------------------------- | -------- | ------------------------------------------------------------ |
| azure.adls1.oauth2_client_id                   | Yes      | サービスプリンシパルのクライアント（アプリケーション）ID。  |
| azure.adls1.oauth2_credential                  | Yes      | 作成された新しいクライアント（アプリケーション）シークレットの値。  |
| azure.adls1.oauth2_endpoint                    | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0 トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2

ストレージとしてData Lake Storage Gen2を選択すると、次のいずれかのアクションを実行します。

- マネージドアイデンティティ認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成します。

```SQL
"azure.adls2.oauth2_use_managed_identity" = "true",
"azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
"azure.adls2.oauth2_client_id" = "<service_client_id>"
```

`StorageCredentialParams`で構成する必要があるパラメータを以下の表に示します。

| **パラメータ**                              | **必須** | **説明**                                                      |
| ----------------------------------------- | ---------- | ------------------------------------------------------------ |
| azure.adls2.oauth2_use_managed_identity  | Yes        | マネージドアイデンティティ認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |
| azure.adls2.oauth2_tenant_id             | Yes        | アクセスするデータのテナントのID。                             |
| azure.adls2.oauth2_client_id             | Yes        | マネージドアイデンティティのクライアント（アプリケーション）ID。 |

- 共有キー認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成します。

```SQL
"azure.adls2.storage_account" = "<storage_account_name>",
"azure.adls2.shared_key" = "<shared_key>"
```

`StorageCredentialParams`で構成する必要があるパラメータを以下の表に示します。

| **パラメータ**               | **必須** | **説明**                                                      |
| --------------------------- | ---------- | ------------------------------------------------------------ |
| azure.adls2.storage_account | Yes        | あなたのData Lake Storage Gen2ストレージアカウントのユーザー名。   |
| azure.adls2.shared_key      | Yes        | あなたのData Lake Storage Gen2ストレージアカウントの共有キー。    |

- サービスプリンシパル認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成します。

```SQL
"azure.adls2.oauth2_client_id" = "<service_client_id>",
"azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
"azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
```

`StorageCredentialParams`で構成する必要があるパラメータを以下の表に示します。

| **パラメータ**                 | **必須** | **説明**                                                      |
| ----------------------------- | ---------- | ------------------------------------------------------------ |
| azure.adls2.oauth2_client_id  | Yes        | サービスプリンシパルのクライアント（アプリケーション）ID。  |
| azure.adls2.oauth2_client_secret | Yes      | 作成された新しいクライアント（アプリケーション）シークレットの値。  |
| azure.adls2.oauth2_client_endpoint | Yes    | サービスプリンシパルまたはアプリケーションのOAuth 2.0 トークンエンドポイント（v1）。 |

##### Google GCS

ストレージとしてGoogle GCSを選択すると、次のいずれかのアクションを実行します。

- VMベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成します。

```SQL
"gcp.gcs.use_compute_engine_service_account" = "true"
```

`StorageCredentialParams`で構成する必要があるパラメータを以下の表に示します。

| **パラメータ**                              | **デフォルト値** | **値の例**               | **説明**                                                      |
| ----------------------------------------- | ----------------- | --------------------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成します。

```SQL
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

`StorageCredentialParams`で構成する必要があるパラメータを以下の表に示します。

| **パラメータ**                          | **デフォルト値** | **値の例**                                        | **説明**                                                      |
| -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| gcp.gcs.service_account_email         | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウント作成時に生成されたJSONファイル内のメールアドレス。 |
| gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウント作成時に生成されたJSONファイル内のプライベートキーID。 |
| gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウント作成時に生成されたJSONファイル内のプライベートキー。 |

- 偽装ベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成します。

  - VMインスタンスがサービスアカウントになりすます場合：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true",
  "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
  ```

  `StorageCredentialParams`で構成する必要があるパラメータを以下の表に示します。

  | **パラメータ**                                | **デフォルト値** | **値の例**     | **説明**                                                      |
  | ------------------------------------------ | ----------------- | ------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
  | gcp.gcs.impersonation_service_account      | ""                | "hello"               | なりすますことを望むサービスアカウント。                     |

  - サービスアカウント（一時的にメタサービスアカウントとして命名）が別のサービスアカウント（一時的にデータサービスアカウントとして命名）を偽装する場合：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
  "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
  "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
  ```

  `StorageCredentialParams`で構成する必要があるパラメータを以下の表に示します。

  | **パラメータ**                          | **デフォルト値** | **値の例**                                        | **説明**                                                      |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウント作成時に生成されたJSONファイル内のメールアドレス。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキーID。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキー。 |
  | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | なりすますことを望むデータサービスアカウント。          |

#### MetadataUpdateParams

StarRocksがHive、Hudi、およびDelta Lakeのキャッシュメタデータを更新するポリシーについての一連のパラメータです。このパラメータセットはオプションです。Hive、Hudi、およびDelta Lakeからのキャッシュメタデータの更新のポリシーの詳細については、[Hiveカタログ](../../data_source/catalog/hive_catalog.md)、[Hudiカタログ](../../data_source/catalog/hudi_catalog.md)、および[Delta Lakeカタログ](../../data_source/catalog/deltalake_catalog.md)を参照してください。

ほとんどの場合、`MetadataUpdateParams`を無視し、それに含まれるポリシーパラメータを調整する必要はありません。なぜなら、これらのパラメータのデフォルト値はすでに貴方に対して使いやすいパフォーマンスを提供しているからです。

しかし、Hive、Hudi、またはDelta Lakeでのデータ更新の頻度が高い場合は、これらのパラメータを調整して自動非同期更新のパフォーマンスをさらに最適化することができます。

| パラメータ                         | 必須    | 説明                        |
| --------------------------------- | ------ | -------------------------- |
| enable_metastore_cache            | No     | StarRocksがHive、Hudi、またはDelta Lakeのメタデータをキャッシュするかどうかを指定します。有効な値：`true`と`false`。デフォルト値：`true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。|
```
| enable_remote_file_cache               | No       | StarRocksはHive、Hudi、またはDelta Lakeのテーブルまたはパーティションの基礎データファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。`true` の場合、キャッシュが有効になり、`false` の場合、キャッシュが無効になります。 |
| metastore_cache_refresh_interval_sec   | No       | StarRocksが自身にキャッシュされたHive、Hudi、またはDelta Lakeのテーブルまたはパーティションのメタデータを非同期に更新する間隔です。単位: 秒。デフォルト値: `7200`（2時間）。 |
| remote_file_cache_refresh_interval_sec | No       | StarRocksが自身にキャッシュされたHive、Hudi、またはDelta Lakeのテーブルまたはパーティションの基礎データファイルのメタデータを非同期に更新する間隔です。単位: 秒。デフォルト値: `60`。 |
| metastore_cache_ttl_sec                | No       | StarRocksが自身にキャッシュされたHive、Hudi、またはDelta Lakeのテーブルまたはパーティションのメタデータを自動的に破棄する間隔です。単位: 秒。デフォルト値: `86400`（24時間）。 |
| remote_file_cache_ttl_sec              | No       | StarRocksが自身にキャッシュされたHive、Hudi、またはDelta Lakeのテーブルまたはパーティションの基礎データファイルのメタデータを自動的に破棄する間隔です。単位: 秒。デフォルト値: `129600`（36時間）。 |

### 例

次の例は、統一データソースからデータをクエリするために、`unified_catalog_hms`または`unified_catalog_glue`という統一カタログを作成します。使用するメタストアのタイプに応じて、名前は異なります。

#### HDFS

ストレージとしてHDFSを使用する場合、次のようにコマンドを実行します。

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

- Hiveメタストアを使用する場合、次のようにコマンドを実行します。

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

- Amazon EMRでAWS Glueを使用する場合、次のようにコマンドを実行します。

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

##### 仮定ベースの認証

- Hiveメタストアを使用する場合、次のようにコマンドを実行します。

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

- Amazon EMRでAWS Glueを使用する場合、次のようにコマンドを実行します。

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

- Hiveメタストアを使用する場合、次のようにコマンドを実行します。

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

- Amazon EMRでAWS Glueを使用する場合、次のようにコマンドを実行します。

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

#### S3互換のストレージシステム

例としてMinIOを使用する場合、次のようにコマンドを実行します。

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

- 共有キー認証方法を選択する場合、次のようにコマンドを実行します。

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

- SASトークン認証方法を選択する場合、次のようにコマンドを実行します。

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

- 管理サービス ID認証方法を選択する場合、次のようにコマンドを実行します。

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

- サービス プリンシパル認証方法を選択する場合、次のようにコマンドを実行します。

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
- マネージドアイデンティティ認証メソッドを選択した場合、以下のようにコマンドを実行してください:

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

- 共有キー認証メソッドを選択した場合、以下のようにコマンドを実行してください:

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

- サービスプリンシパル認証メソッドを選択した場合、以下のようにコマンドを実行してください:

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

- VMベースの認証メソッドを選択した場合、以下のようにコマンドを実行してください:

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

- サービス アカウントベースの認証メソッドを選択した場合、以下のようにコマンドを実行してください:

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

- インパーソネーションベースの認証メソッドを選択した場合:

  - VMインスタンスをサービス アカウントになりすます場合は、以下のようにコマンドを実行してください:

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

  - サービス アカウントが別のサービス アカウントになりすます場合は、以下のようにコマンドを実行してください:

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

## 統一されたカタログを表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスタ内のすべてのカタログをクエリできます:

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、統一カタログ`unified_catalog_glue`の作成ステートメントをクエリできます:

```SQL
SHOW CREATE CATALOG unified_catalog_glue;
```

## 統一カタログとその中のデータベースに切り替える

以下の方法を使用して、統一カタログとそのデータベースに切り替えることができます:

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションで統一カタログを指定し、その後、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します:

  ```SQL
  -- 現在のセッションで指定したカタログに切り替える:
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定します:
  USE <db_name>
  ```

- 直接[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、統一カタログとそのデータベースに切り替えることができます:

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 統一カタログを削除

外部カタログを削除するために[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用できます。

次の例では、`unified_catalog_glue`という名前の統一カタログを削除しています:

```SQL
DROP CATALOG unified_catalog_glue;
```

## 統一カタログからテーブルのスキーマを表示

次の構文を使用して、統一カタログからテーブルのスキーマを表示できます:

- スキーマ表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## 統一カタログからデータをクエリ

統一カタログからデータをクエリするために、以下の手順を実行してください:

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、統一データソースで使用されるデータベースを表示します:

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Hiveカタログとそのデータベースに切り替える](#統一カタログとその中のデータベースに切り替える).

3. 指定したデータベース内の宛先テーブルをクエリするために[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用してください:

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hive、Iceberg、Hudi、またはDelta Lakeからデータをロード

[Hive、Iceberg、Hudi、またはDelta Lakeのデータ](../../sql-reference/sql-statements/data-manipulation/INSERT.md)をStarRocksテーブルにロードするには、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用できます。

次の例では、`unified_catalog`に属する`test_database`内に作成されたStarRocksテーブル`test_tbl`にHiveテーブル`hive_table`のデータをロードしています:

```SQL
INSERT INTO unified_catalog.test_database.test_table SELECT * FROM hive_table
```

## 統一カタログ内にデータベースを作成

StarRocksの内部カタログと同様に、統一カタログ上で[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)ステートメントを使用してデータベースを作成できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksは統一カタログ内にHiveおよびIcebergのデータベースのみを作成できます。

[統一カタログに切り替える](#統一カタログとその中のデータベースに切り替える)と、それから以下のステートメントを使用して、そのカタログ内にデータベースを作成してください:

```SQL
CREATE DATABASE <database_name>
[properties ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location`パラメーターは、HDFSまたはクラウドストレージ内にデータベースを作成するファイルパスを指定します。

- データソースのメタストアとしてHiveメタストアを使用する場合、`location`パラメーターは、データベース作成時にパラメーターを指定しないとHiveメタストアでサポートされる`<warehouse_location>/<database_name.db>`にデフォルトで設定されます。
- データソースのメタストアとしてAWS Glueを使用する場合、`location`パラメータにはデフォルト値がありません。したがって、データベースを作成する際にはこのパラメータを指定する必要があります。

`prefix`は使用するストレージシステムによって異なります：

| **ストレージシステム**                                      | **`Prefix`の値**                                             |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                       | `hdfs`                                                       |
| Google GCS                                                 | `gs`                                                         |
| Azure Blob Storage                                         | <ul><li>ストレージアカウントがHTTPアクセスを許可する場合、`prefix`は`wasb`です。</li><li>ストレージアカウントがHTTPSアクセスを許可する場合、`prefix`は`wasbs`です。</li></ul> |
| Azure Data Lake Storage Gen1                               | `adl`                                                        |
| Azure Data Lake Storage Gen2                               | <ul><li>ストレージアカウントがHTTPアクセスを許可する場合、`prefix`は`abfs`です。</li><li>ストレージアカウントがHTTPSアクセスを許可する場合、`prefix`は`abfss`です。</li></ul> |
| AWS S3または他のS3互換のストレージ（例：MinIO）           | `s3`                                                         |

## 統合カタログからデータベースを削除

StarRocksの内部データベースと同様に、統合カタログ内で作成されたデータベースに[DROP](../../administration/privilege_item.md#database)権限がある場合は、[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md)ステートメントを使用してそのデータベースを削除できます。空のデータベースのみを削除できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksでは、統合カタログからHiveおよびIcebergデータベースのみを削除できます。

統合カタログからデータベースを削除すると、そのデータベースのファイルパス（HDFSクラスターまたはクラウドストレージ上）がデータベースと同時に削除されません。

[統合カタログとそのデータベースに切り替える](#switch-to-a-unified-catalog-and-a-database-in-it)と、次のステートメントを使用してそのカタログ内のデータベースを削除します：

```SQL
DROP DATABASE <database_name>
```

## 統合カタログ内にテーブルを作成

StarRocksの内部データベースと同様に、統合カタログ内で作成されたデータベースに[CREATE TABLE](../../administration/privilege_item.md#database)権限がある場合は、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)または[CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)ステートメントを使用してそのデータベース内にテーブルを作成できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksでは、統合カタログ内でHiveおよびIcebergテーブルのみを作成できます。

[Hiveカタログとそのデータベースに切り替える](#switch-to-a-unified-catalog-and-a-database-in-it)。次に、そのデータベース内にHiveまたはIcebergテーブルを作成するために[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を使用します：

```SQL
CREATE TABLE <table_name>
(column_definition1[, column_definition2, ...]
ENGINE = {|hive|iceberg}
[partition_desc]
```

詳細については、[Hiveテーブルの作成](../catalog/hive_catalog.md#create-a-hive-table)および[Icebergテーブルの作成](../catalog/iceberg_catalog.md#create-an-iceberg-table)を参照してください。

次の例は、`hive_table`という名前のHiveテーブルを作成します。このテーブルには`action`、`id`、`dt`の3つの列があり、そのうち`id`と`dt`はパーティション列です。

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

StarRocksの内部テーブルと同様に、統合カタログ内で作成されたテーブルに[INSERT](../../administration/privilege_item.md#table)権限がある場合は、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ステートメントを使用してStarRocksテーブルのデータをその統合カタログテーブルにシンクできます（現在はParquet形式の統合カタログテーブルのみがサポートされています）。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksでは、統合カタログ内のHiveおよびIcebergテーブルにのみデータをシンクできます。

[Hiveカタログとそのデータベースに切り替える](#switch-to-a-unified-catalog-and-a-database-in-it)。次に、そのデータベース内のHiveまたはIcebergテーブルにデータを挿入するために[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用します：

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータをシンクする場合、次の構文を使用します：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

詳細については、[Hiveテーブルへのデータシンク](../catalog/hive_catalog.md#sink-data-to-a-hive-table)および[Icebergテーブルへのデータシンク](../catalog/iceberg_catalog.md#sink-data-to-an-iceberg-table)を参照してください。

次の例は、`hive_table`という名前のHiveテーブルに3つのデータ行を挿入します：

```SQL
INSERT INTO hive_table
VALUES
    ("buy", 1, "2023-09-01"),
    ("sell", 2, "2023-09-02"),
    ("buy", 3, "2023-09-03");
```

## 統合カタログからテーブルを削除

StarRocksの内部テーブルと同様に、統合カタログ内で作成されたテーブルに[DROP](../../administration/privilege_item.md#table)権限がある場合は、[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)ステートメントを使用してそのテーブルを削除できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

StarRocksでは、統合カタログからHiveおよびIcebergテーブルのみを削除できます。

[Hiveカタログとそのデータベースに切り替える](#switch-to-a-unified-catalog-and-a-database-in-it)。次に、そのデータベース内のHiveまたはIcebergテーブルを削除するために[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)を使用します：

```SQL
DROP TABLE <table_name>
```

詳細については、[Hiveテーブルの削除](../catalog/hive_catalog.md#drop-a-hive-table)および[Icebergテーブルの削除](../catalog/iceberg_catalog.md#drop-an-iceberg-table)を参照してください。

次の例は、`hive_table`という名前のHiveテーブルを削除します：

```SQL
DROP TABLE hive_table FORCE
```