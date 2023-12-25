---
displayed_sidebar: English
toc_max_heading_level: 3
---
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Icebergカタログ

Icebergカタログは、v2.4以降でStarRocksがサポートする外部カタログの一種です。Icebergカタログを使用すると、次のことができます。

- 手動でテーブルを作成することなく、Icebergに保存されているデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)や非同期マテリアライズドビュー（v2.5以降でサポート）を使用して、Icebergに保存されているデータを処理し、StarRocksにロードできます。
- StarRocksで操作を実行してIcebergデータベースやテーブルを作成・削除したり、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用してStarRocksテーブルからParquet形式のIcebergテーブルにデータをシンクすることができます（この機能はv3.1以降でサポートされています）。

IcebergクラスターでSQLワークロードを成功させるには、StarRocksクラスターが以下の2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはオブジェクトストレージ（AWS S3、Microsoft Azure Storage、Google GCS、またはその他のS3互換ストレージシステム、例えばMinIO）

- Hiveメタストア、AWS Glue、またはTabularのようなメタストア

:::note

- ストレージとしてAWS S3を選択した場合、メタストアとしてHMSまたはAWS Glueを使用できます。他のストレージシステムを選択した場合は、HMSをメタストアとしてのみ使用できます。
- メタストアとしてTabularを選択した場合、Iceberg RESTカタログを使用する必要があります。

:::

## 使用上の注意

- StarRocksがサポートするIcebergのファイル形式はParquetとORCです。

  - Parquetファイルは、SNAPPY、LZ4、ZSTD、GZIP、およびNO_COMPRESSIONの圧縮形式をサポートしています。
  - ORCファイルは、ZLIB、SNAPPY、LZO、LZ4、ZSTD、およびNO_COMPRESSIONの圧縮形式をサポートしています。

- Icebergカタログはv1テーブルをサポートし、StarRocks v3.0以降ではORC形式のv2テーブルをサポートします。
- Icebergカタログはv1テーブルをサポートします。さらに、StarRocks v3.0以降ではORC形式のv2テーブルを、StarRocks v3.1以降ではParquet形式のv2テーブルをサポートします。

## 統合準備

Icebergカタログを作成する前に、StarRocksクラスターがIcebergクラスターのストレージシステムおよびメタストアと統合できることを確認してください。

---

### ストレージ

ストレージタイプに一致するタブを選択します。

<Tabs groupId="storage">
<TabItem value="AWS" label="AWS S3" default>

IcebergクラスターがストレージとしてAWS S3を使用するか、メタストアとしてAWS Glueを使用する場合、適切な認証方法を選択し、必要な準備をして、StarRocksクラスターが関連するAWSクラウドリソースにアクセスできるようにします。

推奨される認証方法は以下の通りです。

- インスタンスプロファイル
- アサムドロール
- IAMユーザー

上記の認証方法の中で、インスタンスプロファイルが最も広く使用されています。

詳細については、[AWS IAMでの認証準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

</TabItem>

<TabItem value="HDFS" label="HDFS">

ストレージとしてHDFSを選択する場合、StarRocksクラスターを以下のように構成します。

- （オプション）HDFSクラスターとHiveメタストアへのアクセスに使用するユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスターとHiveメタストアにアクセスします。`export HADOOP_USER_NAME="<user_name>"`を各FEの**fe/conf/hadoop_env.sh**ファイルの先頭に、各BEの**be/conf/hadoop_env.sh**ファイルの先頭に追加することでユーザー名を設定することもできます。これらのファイルにユーザー名を設定した後、各FEとBEを再起動して、パラメータ設定を有効にします。StarRocksクラスターごとに1つのユーザー名のみを設定できます。
- Icebergデータをクエリする際、StarRocksクラスターのFEとBEはHDFSクライアントを使用してHDFSクラスターにアクセスします。ほとんどの場合、その目的を達成するためにStarRocksクラスターを設定する必要はありません。StarRocksはデフォルトの設定を使用してHDFSクライアントを起動します。StarRocksクラスターを設定する必要があるのは、以下の状況のみです。

  - HDFSクラスターで高可用性（HA）が有効になっている場合：HDFSクラスターの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。
  - View File System（ViewFs）がHDFSクラスターで有効になっている場合：HDFSクラスターの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。

:::tip

クエリを送信した際に不明なホストを示すエラーが返される場合、HDFSクラスターノードのホスト名とIPアドレスのマッピングを**/etc/hosts**に追加する必要があります。

:::

---

#### Kerberos認証

HDFSクラスターやHiveメタストアでKerberos認証が有効になっている場合、StarRocksクラスターを以下のように構成します。

- 各FEおよびBEで`kinit -kt keytab_path principal`コマンドを実行して、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFSクラスターとHiveメタストアにアクセスする権限が必要です。KDCへのアクセスは時間に敏感なため、このコマンドを定期的に実行するためにcronを使用する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルと各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。この例では、`/etc/krb5.conf`は**krb5.conf**ファイルの保存パスです。必要に応じてパスを変更できます。

</TabItem>

</Tabs>

---

## Icebergカタログの作成

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

---

### パラメータ

#### catalog_name

Icebergカタログの名前です。命名規則は以下の通りです。

- 名前には文字、数字（0-9）、およびアンダースコア（_）を含めることができます。最初の文字は文字でなければなりません。
- 名前は大文字と小文字が区別され、長さは1023文字を超えることはできません。

#### comment

Icebergカタログの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。値を`iceberg`に設定します。

#### MetastoreParams

StarRocksがデータソースのメタストアと統合する方法に関するパラメータセットです。メタストアのタイプに一致するタブを選択します。

<Tabs groupId="metastore">
<TabItem value="HIVE" label="Hiveメタストア" default>

##### Hiveメタストア

データソースのメタストアとしてHiveメタストアを選択する場合、`MetastoreParams`を以下のように構成します。

```SQL
"iceberg.catalog.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

:::note

Icebergデータをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを`/etc/hosts`に追加する必要があります。そうしないと、クエリを開始する際にStarRocksがHiveメタストアにアクセスできない可能性があります。

:::

以下の表は、`MetastoreParams`で設定する必要があるパラメータを説明しています。

##### iceberg.catalog.type

必須: はい
説明: Icebergクラスターで使用するメタストアのタイプです。値を`hive`に設定します。

##### hive.metastore.uris

必須: はい
説明: Hive メタストアの URI。フォーマット: `thrift://<metastore_IP_address>:<metastore_port>`。<br />Hive メタストアで高可用性 (HA) が有効になっている場合、複数のメタストア URI を指定し、コンマ (`,`) で区切ることができます。例: `"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。

</TabItem>
<TabItem value="GLUE" label="AWS Glue">

##### AWS Glue

データソースのメタストアとして AWS Glue を選択する場合（ストレージとして AWS S3 を選択した場合にのみサポートされます）、以下のアクションのいずれかを取ります：

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams` を次のように設定します：

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 想定ロールベースの認証方法を選択する場合、`MetastoreParams` を次のように設定します：

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAM ユーザーベースの認証方法を選択する場合、`MetastoreParams` を次のように設定します：

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

AWS Glue の `MetastoreParams`:

###### iceberg.catalog.type

必須: はい
説明: Iceberg クラスターで使用するメタストアのタイプ。値を `glue` に設定します。

###### aws.glue.use_instance_profile

必須: はい
説明: インスタンスプロファイルベースの認証方法および想定ロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。

###### aws.glue.iam_role_arn

必須: いいえ
説明: AWS Glue Data Catalog に対する権限を持つ IAM ロールの ARN。想定ロールベースの認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。

###### aws.glue.region

必須: はい
説明: AWS Glue Data Catalog が存在するリージョン。例: `us-west-1`。

###### aws.glue.access_key

必須: いいえ
説明: IAM ユーザーのアクセスキー。IAM ユーザーベースの認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。

###### aws.glue.secret_key

必須: いいえ
説明: IAM ユーザーのシークレットキー。IAM ユーザーベースの認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。

AWS Glue へのアクセスに使用する認証方法の選択と、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS Glue へのアクセスに関する認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

</TabItem>
<TabItem value="TABULAR" label="Tabular">

##### Tabular

メタストアとして Tabular を使用する場合、メタストアのタイプを REST (`"iceberg.catalog.type" = "rest"`) として指定する必要があります。`MetastoreParams` を次のように設定します：

```SQL
"iceberg.catalog.type" = "rest",
"iceberg.catalog.uri" = "<rest_server_api_endpoint>",
"iceberg.catalog.credential" = "<credential>",
"iceberg.catalog.warehouse" = "<identifier_or_path_to_warehouse>"
```

Tabular の `MetastoreParams`:

###### iceberg.catalog.type

必須: はい
説明: Iceberg クラスターで使用するメタストアのタイプ。値を `rest` に設定します。

###### iceberg.catalog.uri

必須: はい
説明: Tabular サービスエンドポイントの URI。例: `https://api.tabular.io/ws`。

###### iceberg.catalog.credential

必須: はい
説明: Tabular サービスの認証情報。

###### iceberg.catalog.warehouse

必須: いいえ
説明: Iceberg カタログの倉庫の場所または識別子。例: `s3://my_bucket/warehouse_location` または `sandbox`。

以下の例では、メタストアとして Tabular を使用する `tabular` という名前の Iceberg カタログを作成します：

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
</TabItem>

</Tabs>

---

#### `StorageCredentialParams`

StarRocks がストレージシステムと統合する方法に関するパラメータセットです。このパラメータセットはオプションです。

以下の点に注意してください：

- HDFS をストレージとして使用する場合、`StorageCredentialParams` を設定する必要はありません。このセクションはスキップできます。AWS S3、その他の S3 互換ストレージシステム、Microsoft Azure Storage、または Google GCS をストレージとして使用する場合、`StorageCredentialParams` を設定する必要があります。

- メタストアとして Tabular を使用する場合、`StorageCredentialParams` を設定する必要はありません。このセクションはスキップできます。HMS または AWS Glue をメタストアとして使用する場合、`StorageCredentialParams` を設定する必要があります。

ストレージのタイプに合わせたタブを選択してください：

<Tabs groupId="storage">
<TabItem value="AWS" label="AWS S3" default>

##### AWS S3

Iceberg クラスターのストレージとして AWS S3 を選択する場合、以下のアクションのいずれかを取ります：

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams` を次のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 想定ロールベースの認証方法を選択する場合、`StorageCredentialParams` を次のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM ユーザーベースの認証方法を選択する場合、`StorageCredentialParams` を次のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

AWS S3 の `StorageCredentialParams`:

###### aws.s3.use_instance_profile

必須: はい
説明: インスタンスプロファイルベースの認証方法および想定ロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。

###### aws.s3.iam_role_arn

必須: いいえ
説明: AWS S3 バケットに対する権限を持つ IAM ロールの ARN。想定ロールベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。

###### aws.s3.region

必須: はい
説明: AWS S3 バケットが存在するリージョン。例: `us-west-1`。

###### aws.s3.access_key

必須: いいえ
説明: IAM ユーザーのアクセスキー。IAM ユーザーベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。

###### aws.s3.secret_key

必須: いいえ
説明: IAM ユーザーのシークレットキー。IAM ユーザーベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。

AWS S3 へのアクセスに使用する認証方法の選択と、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS S3 へのアクセスに関する認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

</TabItem>

<TabItem value="HDFS" label="HDFS" >

HDFS ストレージを使用する場合、ストレージ資格情報は不要です。

</TabItem>

<TabItem value="MINIO" label="MinIO" >

##### S3 互換ストレージシステム

Iceberg カタログは、v2.5 以降の S3 互換ストレージシステムをサポートしています。


MinIOなどのS3互換ストレージシステムをIcebergクラスターのストレージとして選択する場合は、`StorageCredentialParams`を以下のように設定して統合が成功するようにします：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

MinIOおよびその他のS3互換システム用の`StorageCredentialParams`：

###### aws.s3.enable_ssl

必須：はい
説明：SSL接続を有効にするかどうかを指定します。<br />有効な値：`true`、`false`。デフォルト値：`true`。

###### aws.s3.enable_path_style_access

必須：はい
説明：パススタイルアクセスを有効にするかどうかを指定します。<br />有効な値：`true`、`false`。デフォルト値：`false`。MinIOでは、この値を`true`に設定する必要があります。<br />パススタイルのURLは以下の形式を使用します：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部（オレゴン）リージョンに`DOC-EXAMPLE-BUCKET1`という名前のバケットを作成し、そのバケット内の`alice.jpg`オブジェクトにアクセスしたい場合、以下のパススタイルURLを使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。

###### aws.s3.endpoint

必須：はい
説明：AWS S3ではなく、S3互換ストレージシステムに接続するために使用されるエンドポイント。

###### aws.s3.access_key

必須：はい
説明：IAMユーザーのアクセスキー。

###### aws.s3.secret_key

必須：はい
説明：IAMユーザーのシークレットキー。

</TabItem>

<TabItem value="AZURE" label="Microsoft Azure Blob Storage">

##### Microsoft Azure Storage

Icebergカタログは、v3.0以降、Microsoft Azure Storageをサポートしています。

###### Azure Blob Storage

IcebergクラスターのストレージとしてBlob Storageを選択する場合、以下のいずれかのアクションを取ります：

- 共有キー認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

- SASトークン認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

Microsoft Azure用の`StorageCredentialParams`：

###### azure.blob.storage_account

必須：はい
説明：Blob Storageアカウントのユーザー名。

###### azure.blob.shared_key

必須：はい
説明：Blob Storageアカウントの共有キー。

###### azure.blob.account_name

必須：はい
説明：Blob Storageアカウントのユーザー名。

###### azure.blob.container

必須：はい
説明：データを格納するblobコンテナの名前。

###### azure.blob.sas_token

必須：はい
説明：Blob Storageアカウントへのアクセスに使用されるSASトークン。

###### Azure Data Lake Storage Gen1

IcebergクラスターのストレージとしてData Lake Storage Gen1を選択する場合、以下のいずれかのアクションを取ります：

- 管理されたサービスID認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

または：

- サービスプリンシパル認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

###### Azure Data Lake Storage Gen2

IcebergクラスターのストレージとしてData Lake Storage Gen2を選択する場合、以下のいずれかのアクションを取ります：

- マネージドID認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  または：

- 共有キー認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  または：

- サービスプリンシパル認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

</TabItem>

<TabItem value="GCS" label="Google GCS">

##### Google GCS

Icebergカタログは、v3.0以降、Google GCSをサポートしています。

IcebergクラスターのストレージとしてGoogle GCSを選択する場合、以下のいずれかのアクションを取ります：

- VMベースの認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

- サービスアカウントベースの認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

- なりすましベースの認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

  - VMインスタンスがサービスアカウントをなりすます場合：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

  - サービスアカウント（一時的にメタサービスアカウントと呼ばれる）が別のサービスアカウント（一時的にデータサービスアカウントと呼ばれる）をなりすます場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

Google GCS用の`StorageCredentialParams`：

###### gcp.gcs.service_account_email

デフォルト値：""
例：`user@hello.iam.gserviceaccount.com`
説明：サービスアカウント作成時に生成されたJSONファイル内のメールアドレス。

###### gcp.gcs.service_account_private_key_id

デフォルト値：""
例："61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"
説明：サービスアカウント作成時に生成されたJSONファイル内のプライベートキーID。

###### gcp.gcs.service_account_private_key

デフォルト値：""
例："-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"
説明：サービスアカウント作成時に生成されたJSONファイル内のプライベートキー。

###### gcp.gcs.impersonation_service_account

デフォルト値：""
例："hello"
説明：なりすますサービスアカウント。

</TabItem>

</Tabs>

---

### 例

以下の例では、`iceberg_catalog_hms`または`iceberg_catalog_glue`という名前のIcebergカタログを作成し、Icebergクラスターからデータをクエリします。使用するメタストアのタイプに応じて、ストレージの種類に一致するタブを選択します。

<Tabs groupId="storage">
<TabItem value="AWS" label="AWS S3" default>

#### AWS S3

##### インスタンスプロファイルベースのクレデンシャルを選択した場合

- IcebergクラスターでHiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR IcebergクラスターでAWS Glueを使用する場合、以下のコマンドを実行します：

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

##### 想定されるロールベースのクレデンシャルを選択した場合

- IcebergクラスターでHiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Iceberg クラスターで AWS Glue を使用する場合、以下のコマンドを実行します。

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

##### IAM ユーザーベースの認証情報を選択する場合

- Iceberg クラスターで Hive メタストアを使用する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Iceberg クラスターで AWS Glue を使用する場合、以下のコマンドを実行します。

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
</TabItem>

<TabItem value="HDFS" label="HDFS" >

#### HDFS

HDFSをストレージとして使用する場合、以下のコマンドを実行します。

```SQL
CREATE EXTERNAL CATALOG iceberg_catalog_hms
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

</TabItem>

<TabItem value="MINIO" label="MinIO" >

#### S3互換ストレージシステム

MinIOを例として、以下のコマンドを実行します。

```SQL
CREATE EXTERNAL CATALOG iceberg_catalog_hms
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "aws.s3.enable_ssl" = "true",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```
</TabItem>

<TabItem value="AZURE" label="Microsoft Azure Blob Storage" >

#### Microsoft Azure ストレージ

##### Azure Blob Storage

- 共有キー認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SAS トークン認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- マネージドサービスID認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- サービスプリンシパル認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- マネージドアイデンティティ認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 共有キー認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- サービスプリンシパル認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

</TabItem>

<TabItem value="GCS" label="Google GCS" >

#### Google GCS

- VMベースの認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- サービスアカウントベースの認証方法を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 偽装ベースの認証方法を選択する場合：

  - VMインスタンスがサービスアカウントを偽装する場合、以下のコマンドを実行します。

    ```SQL
    CREATE EXTERNAL CATALOG iceberg_catalog_hms
    PROPERTIES
    (
        "type" = "iceberg",
        "iceberg.catalog.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - サービスアカウントが別のサービスアカウントを偽装する場合、以下のコマンドを実行します。

    ```SQL
    CREATE EXTERNAL CATALOG iceberg_catalog_hms
    PROPERTIES
    (
        "type" = "iceberg",
        "iceberg.catalog.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_private_key>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    );
    ```
</TabItem>

</Tabs>
 
 ---

## カタログの使用

### Iceberg カタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) を使用して、現在の StarRocks クラスター内のすべてのカタログを照会できます。

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を使用して、外部カタログの作成ステートメントを照会することもできます。次の例では、`iceberg_catalog_glue` という名前の Iceberg カタログの作成ステートメントをクエリします。

```SQL
SHOW CREATE CATALOG iceberg_catalog_glue;
```

---

### Iceberg カタログとその中のデータベースに切り替える

以下のいずれかの方法を使用して、Iceberg カタログとその中のデータベースに切り替えることができます：

- 現在のセッションで Iceberg カタログを指定するために [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) を使用し、その後アクティブなデータベースを指定するために [USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用します：

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替える：
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定する：
  USE <db_name>
  ```

- Iceberg カタログとその中のデータベースに直接切り替えるために [USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用します：

  ```SQL
  USE <catalog_name>.<db_name>
  ```
 
---

### Iceberg カタログを削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) を使用して外部カタログを削除できます。

以下の例では、`iceberg_catalog_glue` という名前の Iceberg カタログを削除します：

```SQL
DROP CATALOG iceberg_catalog_glue;
```

---

### Iceberg テーブルのスキーマを表示する

Iceberg テーブルのスキーマを表示するために以下の構文のいずれかを使用できます：

- スキーマを表示する

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATE ステートメントからスキーマと場所を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

---

### Iceberg テーブルをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を使用して、Iceberg クラスタ内のデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Iceberg カタログとその中のデータベースに切り替える](#switch-to-an-iceberg-catalog-and-a-database-in-it)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して、指定されたデータベース内の目的のテーブルをクエリします：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

---

### Iceberg データベースを作成する

StarRocks の内部カタログと同様に、Iceberg カタログに [CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) 権限がある場合、[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) ステートメントを使用してその Iceberg カタログ内にデータベースを作成できます。この機能は v3.1 以降でサポートされています。

:::tip

権限の付与と取り消しは [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して行うことができます。

:::

[Iceberg カタログに切り替え](#switch-to-an-iceberg-catalog-and-a-database-in-it)、その後以下のステートメントを使用してそのカタログ内に Iceberg データベースを作成します：

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>/")]
```

`location` パラメータを使用して、データベースを作成するファイルパスを指定できます。HDFS とクラウドストレージがサポートされています。`location` パラメータを指定しない場合、StarRocks は Iceberg カタログのデフォルトファイルパスにデータベースを作成します。

`prefix` は使用するストレージシステムに基づいて異なります：

#### HDFS

`prefix` の値：`hdfs`

#### Google GCS

`prefix` の値：`gs`

#### Azure Blob Storage

`prefix` の値：

- ストレージアカウントが HTTP 経由でのアクセスを許可している場合、`prefix` は `wasb` です。
- ストレージアカウントが HTTPS 経由でのアクセスを許可している場合、`prefix` は `wasbs` です。

#### Azure Data Lake Storage Gen1

`prefix` の値：`adl`

#### Azure Data Lake Storage Gen2

`prefix` の値：

- ストレージアカウントが HTTP 経由でのアクセスを許可している場合、`prefix` は `abfs` です。
- ストレージアカウントが HTTPS 経由でのアクセスを許可している場合、`prefix` は `abfss` です。

#### AWS S3 またはその他の S3 互換ストレージ（例：MinIO）

`prefix` の値：`s3`

---

### Iceberg データベースを削除する

StarRocks の内部データベースと同様に、Iceberg データベースに [DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md) 権限がある場合、[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md) ステートメントを使用してその Iceberg データベースを削除できます。この機能は v3.1 以降でサポートされています。空のデータベースのみを削除できます。

Iceberg データベースを削除しても、HDFS クラスターやクラウドストレージ上のデータベースのファイルパスは削除されません。

[Iceberg カタログに切り替え](#switch-to-an-iceberg-catalog-and-a-database-in-it)、その後以下のステートメントを使用してそのカタログ内の Iceberg データベースを削除します：

```SQL
DROP DATABASE <database_name>;
```

---

### Iceberg テーブルを作成する

StarRocks の内部データベースと同様に、Iceberg データベースに [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 権限がある場合、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) または [CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) ステートメントを使用してその Iceberg データベース内にテーブルを作成できます。この機能は v3.1 以降でサポートされています。

[Iceberg カタログとその中のデータベースに切り替え](#switch-to-an-iceberg-catalog-and-a-database-in-it)、その後以下の構文を使用してそのデータベース内に Iceberg テーブルを作成します。

#### 構文

```SQL
CREATE TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...
partition_column_definition1, partition_column_definition2...])
[partition_desc]
[PROPERTIES ("key" = "value", ...)]
[AS SELECT query]
```

#### パラメータ

##### column_definition

`column_definition` の構文は以下の通りです：

```SQL
col_name col_type [COMMENT 'comment']
```

:::note

パーティション以外のすべての列は `NULL` をデフォルト値として使用する必要があります。これは、テーブル作成ステートメントで非パーティション列ごとに `DEFAULT "NULL"` を指定する必要があることを意味します。さらに、パーティション列は非パーティション列の後に定義され、`NULL` をデフォルト値として使用することはできません。

:::

##### partition_desc

`partition_desc` の構文は以下の通りです：

```SQL
PARTITION BY (par_col1[, par_col2...])
```

現在 StarRocks は [identity transforms](https://iceberg.apache.org/spec/#partitioning) のみをサポートしており、StarRocks は一意のパーティション値ごとにパーティションを作成します。

:::note

パーティション列は非パーティション列の後に定義される必要があります。パーティション列は FLOAT、DOUBLE、DECIMAL、DATETIME を除くすべてのデータ型をサポートし、`NULL` をデフォルト値として使用することはできません。

:::

##### PROPERTIES

`PROPERTIES` で `"key" = "value"` 形式でテーブル属性を指定できます。[Iceberg テーブル属性](https://iceberg.apache.org/docs/latest/configuration/) を参照してください。

以下の表はいくつかの主要なプロパティを説明しています。

###### location

説明：Iceberg テーブルを作成するファイルパス。HMS をメタストアとして使用する場合、`location` パラメータを指定する必要はありません。StarRocks は現在の Iceberg カタログのデフォルトファイルパスにテーブルを作成します。AWS Glue をメタストアとして使用する場合：

- テーブルを作成するデータベースに `location` パラメータを指定している場合、テーブルに `location` パラメータを指定する必要はありません。そのため、テーブルはデフォルトでそれが属するデータベースのファイルパスになります。
- テーブルを作成するデータベースに `location` パラメータを指定していない場合、テーブルに `location` パラメータを指定する必要があります。

###### file_format

説明：Iceberg テーブルのファイル形式。現在サポートされているのは Parquet 形式のみです。デフォルト値：`parquet`。

###### compression_codec

説明：Iceberg テーブルで使用される圧縮アルゴリズム。サポートされている圧縮アルゴリズムは SNAPPY、GZIP、ZSTD、LZ4 です。デフォルト値：`gzip`。

---

### 例

1. `unpartition_tbl` という名前の非パーティションテーブルを作成します。このテーブルは、以下に示すように、`id` と `score` の 2 つの列で構成されています：

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id INT,
       score DOUBLE
   );
   ```
2. `partition_tbl_1` という名前のパーティションテーブルを作成します。このテーブルは、`action`、`id`、`dt` の3つのカラムで構成され、`id` と `dt` はパーティションカラムとして定義されます。以下のようになります：

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id, dt);
   ```

3. `partition_tbl_1` という名前の既存のテーブルをクエリし、`partition_tbl_1` のクエリ結果に基づいて `partition_tbl_2` という名前のパーティションテーブルを作成します。`partition_tbl_2` では、`id` と `dt` がパーティションカラムとして定義されます。以下のようになります：

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (id, dt)
   AS SELECT * FROM employee;
   ```

---

### Icebergテーブルへデータをシンクする

StarRocksの内部テーブルと同様に、Icebergテーブルに対する[INSERT](../../administration/privilege_item.md#table)権限がある場合、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ステートメントを使用してStarRocksテーブルのデータをそのIcebergテーブルにシンクできます（現在はParquet形式のIcebergテーブルのみがサポートされています）。この機能はv3.1以降でサポートされています。

[Icebergカタログとその中のデータベースに切り替え](#switch-to-an-iceberg-catalog-and-a-database-in-it)、次に示す構文を使用して、StarRocksテーブルのデータをそのデータベース内のParquet形式のIcebergテーブルにシンクします。

#### 構文

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータをシンクしたい場合は、以下の構文を使用します：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

:::note

パーティションカラムは `NULL` 値を許可しません。したがって、Icebergテーブルのパーティションカラムに空の値がロードされないように注意する必要があります。

:::

#### パラメータ

##### INTO

StarRocksテーブルのデータをIcebergテーブルに追加します。

##### OVERWRITE

StarRocksテーブルのデータでIcebergテーブルの既存のデータを上書きします。

##### column_name

データをロードする宛先カラムの名前です。一つまたは複数のカラムを指定できます。複数のカラムを指定する場合は、カンマ(`,`)で区切ります。指定できるのはIcebergテーブルに実際に存在するカラムのみであり、指定する宛先カラムにはIcebergテーブルのパーティションカラムが含まれている必要があります。指定した宛先カラムは、宛先カラム名に関係なく、StarRocksテーブルのカラムに一対一で順番にマッピングされます。宛先カラムが指定されていない場合、データはIcebergテーブルのすべてのカラムにロードされます。StarRocksテーブルの非パーティションカラムがIcebergテーブルのどのカラムにもマッピングできない場合、StarRocksはデフォルト値`NULL`をIcebergテーブルのカラムに書き込みます。INSERTステートメントに返されるカラムの型が宛先カラムのデータ型と異なるクエリステートメントが含まれている場合、StarRocksは一致しないカラムに対して暗黙の型変換を行います。型変換に失敗すると、構文解析エラーが返されます。

##### expression

宛先カラムに値を割り当てる式です。

##### DEFAULT

宛先カラムにデフォルト値を割り当てます。

##### query

その結果がIcebergテーブルにロードされるクエリステートメントです。StarRocksがサポートする任意のSQLステートメントが可能です。

##### PARTITION

データをロードするパーティションです。このプロパティには、Icebergテーブルのすべてのパーティションカラムを指定する必要があります。このプロパティで指定するパーティションカラムは、テーブル作成ステートメントで定義したパーティションカラムとは異なる順序であっても構いません。このプロパティを指定した場合、`column_name`プロパティは指定できません。

#### 例

1. `partition_tbl_1` テーブルに3つのデータ行を挿入します：

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. 単純な計算を含むSELECTクエリの結果を `partition_tbl_1` テーブルに挿入します：

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. `partition_tbl_1` テーブルからデータを読み取るSELECTクエリの結果を同じテーブルに挿入します：

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. `dt='2023-09-01'` と `id=1` の2つの条件を満たすパーティションにSELECTクエリの結果を `partition_tbl_2` テーブルに挿入します：

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT INTO partition_tbl_2 PARTITION (dt='2023-09-01', id=1) SELECT 'order';
   ```

5. `partition_tbl_1` テーブルの `dt='2023-09-01'` と `id=1` の2つの条件を満たすパーティション内のすべての `action` カラムの値を `close` で上書きします：

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT OVERWRITE partition_tbl_1 PARTITION (dt='2023-09-01', id=1) SELECT 'close';
   ```

---

### Icebergテーブルを削除する

StarRocksの内部テーブルと同様に、Icebergテーブルに対する [DROP](../../administration/privilege_item.md#table) 権限がある場合、[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) ステートメントを使用してそのIcebergテーブルを削除できます。この機能はv3.1以降でサポートされています。

Icebergテーブルを削除しても、HDFSクラスターやクラウドストレージ上のテーブルのファイルパスとデータは削除されません。

Icebergテーブルを強制的に削除する場合（つまり、DROP TABLEステートメントで `FORCE` キーワードが指定されている場合）、HDFSクラスターやクラウドストレージ上のテーブルのデータはテーブルと共に削除されますが、テーブルのファイルパスは保持されます。

[Icebergカタログとその中のデータベースに切り替え](#switch-to-an-iceberg-catalog-and-a-database-in-it)、次に示すステートメントを使用して、そのデータベース内のIcebergテーブルを削除します。

```SQL
DROP TABLE <table_name> [FORCE];
```

---

### メタデータキャッシングを設定する

Icebergクラスターのメタデータファイルは、AWS S3やHDFSなどのリモートストレージに保存されることがあります。デフォルトでは、StarRocksはIcebergメタデータをメモリにキャッシュします。クエリを高速化するため、StarRocksは二段階のメタデータキャッシングメカニズムを採用しており、メタデータをメモリとディスクの両方にキャッシュすることができます。各初回クエリに対して、StarRocksはその計算結果をキャッシュします。以前のクエリと意味的に同等のクエリが後続で発行された場合、StarRocksはまずキャッシュから要求されたメタデータを取得しようとします。キャッシュ内でメタデータがヒットしない場合のみ、リモートストレージからメタデータを取得します。

StarRocksは、最近最も使用されていない(LRU)アルゴリズムを使用してデータをキャッシュし、排出します。基本的なルールは以下の通りです：

- StarRocksはまず、要求されたメタデータをメモリから取得しようとします。メモリ内でメタデータがヒットしない場合、StarRocksはディスクからメタデータを取得しようとします。ディスクから取得したメタデータはメモリにロードされます。ディスク内でもメタデータがヒットしない場合、StarRocksはリモートストレージからメタデータを取得し、取得したメタデータをメモリにキャッシュします。
- メモリから排出されたメタデータはディスクに書き込まれますが、ディスクから排出されたメタデータは直接破棄されます。

#### Icebergメタデータキャッシングパラメータ

##### enable_iceberg_metadata_disk_cache

単位: なし
デフォルト値： `false`
説明: ディスクキャッシュを有効にするかどうかを指定します。

##### iceberg_metadata_cache_disk_path

単位: なし
デフォルト値： `StarRocksFE.STARROCKS_HOME_DIR + "/caches/iceberg"`
説明: ディスク上にキャッシュされたメタデータファイルの保存パス。

##### iceberg_metadata_disk_cache_capacity

単位: バイト
デフォルト値: `2147483648`、これは2 GBに相当します
説明: ディスク上にキャッシュできるメタデータの最大サイズ。

##### iceberg_metadata_memory_cache_capacity

単位: バイト
デフォルト値: `536870912`、これは512 MBに相当します
説明: メモリ内にキャッシュできるメタデータの最大サイズ。

##### iceberg_metadata_memory_cache_expiration_seconds

単位: 秒
デフォルト値: `86500`
説明: メモリ内のキャッシュエントリが最後のアクセスから期限切れになるまでの時間。

##### iceberg_metadata_disk_cache_expiration_seconds

単位: 秒
デフォルト値: `604800`、これは1週間に相当します
説明: ディスク上のキャッシュエントリが最後のアクセスから期限切れになるまでの時間。

##### iceberg_metadata_cache_max_entry_size

単位: バイト
デフォルト値: `8388608`、これは8 MBに相当します
説明: キャッシュ可能なファイルの最大サイズ。このパラメータの値を超えるサイズのファイルはキャッシュされません。これらのファイルを要求するクエリがある場合、StarRocksはリモートストレージからそれらを取得します。
