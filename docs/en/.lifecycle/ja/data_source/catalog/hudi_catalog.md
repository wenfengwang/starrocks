---
displayed_sidebar: "Japanese"
---

# Hudiカタログ

Hudiカタログは、Apache Hudiからデータをクエリするための外部カタログであり、インジェストなしでデータをクエリできます。

また、Hudiカタログに基づいて、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用して直接Hudiからデータを変換およびロードすることもできます。StarRocksは、v2.4以降のHudiカタログをサポートしています。

HudiクラスタでのSQLワークロードの成功を確保するためには、StarRocksクラスタが2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCSなどのオブジェクトストレージ、またはその他のS3互換ストレージシステム（たとえば、MinIO）などのオブジェクトストレージ

- HiveメタストアまたはAWS Glueなどのメタストア

  > **注**
  >
  > ストレージとしてAWS S3を選択した場合、メタストアとしてHMSまたはAWS Glueを使用できます。他のストレージシステムを選択した場合、メタストアとしてHMSのみを使用できます。

## 使用上の注意

- StarRocksがサポートするHudiのファイル形式はParquetです。Parquetファイルは、以下の圧縮形式をサポートしています：SNAPPY、LZ4、ZSTD、GZIP、およびNO_COMPRESSION。
- StarRocksは、HudiからのCopy On Write（COW）テーブルとMerge On Read（MOR）テーブルを完全にサポートしています。

## 統合の準備

Hudiカタログを作成する前に、StarRocksクラスタがHudiクラスタのストレージシステムとメタストアと統合できることを確認してください。

### AWS IAM

HudiクラスタがストレージとしてAWS S3またはメタストアとしてAWS Glueを使用する場合は、適切な認証方法を選択し、関連するAWSクラウドリソースにStarRocksクラスタがアクセスできるように必要な準備を行ってください。

以下の認証方法が推奨されています。

- インスタンスプロファイル
- 仮定されたロール
- IAMユーザー

上記の3つの認証方法のうち、インスタンスプロファイルが最も一般的に使用されています。

詳細については、[AWS IAMでの認証の準備](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)を参照してください。

### HDFS

ストレージとしてHDFSを選択した場合、StarRocksクラスタを次のように構成してください。

- （オプション）HDFSクラスタとHiveメタストアにアクセスするために使用されるユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスタとHiveメタストアにアクセスします。各FEの**fe/conf/hadoop_env.sh**ファイルと各BEの**be/conf/hadoop_env.sh**ファイルの先頭に`export HADOOP_USER_NAME="<user_name>"`を追加してユーザー名を設定することもできます。これらのファイルでユーザー名を設定した後、各FEと各BEを再起動してパラメータ設定が有効になるようにします。StarRocksクラスタごとに1つのユーザー名のみを設定できます。
- Hudiデータをクエリする際、StarRocksクラスタのFEおよびBEはHDFSクラスタにアクセスするためにHDFSクライアントを使用します。ほとんどの場合、この目的を達成するためにStarRocksクラスタを構成する必要はありません。StarRocksはデフォルトの設定でHDFSクライアントを起動します。次の状況では、StarRocksクラスタを構成する必要があります。

  - HDFSクラスタに対して高可用性（HA）が有効になっている場合：HDFSクラスタの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。
  - HDFSクラスタに対してView File System（ViewFs）が有効になっている場合：HDFSクラスタの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。

> **注**
>
> クエリを送信する際に不明なホストを示すエラーが返される場合は、HDFSクラスタノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに追加する必要があります。

### Kerberos認証

HDFSクラスタまたはHiveメタストアにKerberos認証が有効になっている場合、StarRocksクラスタを次のように構成してください。

- 各FEおよび各BEで`kinit -kt keytab_path principal`コマンドを実行して、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFSクラスタとHiveメタストアにアクセスする権限が必要です。このコマンドを使用してKDCにアクセスする際は、時間的な制約があるため、このコマンドを定期的に実行するためにcronを使用する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルと各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。この例では、`/etc/krb5.conf`は**krb5.conf**ファイルの保存パスです。必要に応じてパスを変更できます。

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

Hudiカタログの名前です。命名規則は次のとおりです。

- 名前には、文字、数字（0-9）、アンダースコア（_）を含めることができます。ただし、文字で始める必要があります。
- 名前は大文字と小文字を区別し、長さは1023文字を超えることはできません。

#### comment

Hudiカタログの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。値を`hudi`に設定します。

#### MetastoreParams

データソースのメタストアとの統合方法に関する一連のパラメータです。

##### Hiveメタストア

データソースのメタストアとしてHiveメタストアを選択した場合、`MetastoreParams`を次のように構成します。

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注**
>
> Hudiデータをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを`/etc/hosts`パスに追加する必要があります。そうしないと、クエリを開始する際にStarRocksがHiveメタストアにアクセスできなくなる場合があります。

以下の表に、`MetastoreParams`で構成する必要のあるパラメータについて説明します。

| パラメータ名             | 必須 | 説明                                                         |
| ----------------------- | ---- | ------------------------------------------------------------ |
| hive.metastore.type     | Yes  | Hudiクラスタで使用するメタストアのタイプ。値を`hive`に設定します。 |
| hive.metastore.uris     | Yes  | HiveメタストアのURI。形式：`thrift://<metastore_IP_address>:<metastore_port>`。<br />Hiveメタストアに対して高可用性（HA）が有効になっている場合、複数のメタストアURIを指定し、カンマ（`,`）で区切って指定できます。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`のように指定します。 |

##### AWS Glue

データソースのメタストアとしてAWS Glueを選択した場合（ストレージとしてAWS S3を選択した場合のみサポートされます）、次のいずれかの操作を実行します。

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams`を次のように構成します。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 仮定されたロールベースの認証方法を選択する場合、`MetastoreParams`を次のように構成します。

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

以下の表に、`MetastoreParams`で構成する必要のあるパラメータについて説明します。

| パラメータ名                     | 必須 | 説明                                                         |
| ------------------------------- | ---- | ------------------------------------------------------------ |
| hive.metastore.type             | Yes  | Hudiクラスタで使用するメタストアのタイプ。値を`glue`に設定します。 |
| aws.glue.use_instance_profile   | Yes  | インスタンスプロファイルベースの認証方法および仮定されたロールベースの認証方法を有効にするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。 |
| aws.glue.iam_role_arn           | No   | AWS Glue Data Catalogに特権を持つIAMロールのARN。AWS Glueへのアクセスに仮定されたロールベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.region                 | Yes  | AWS Glue Data Catalogが存在するリージョン。例：`us-west-1`。 |
| aws.glue.access_key             | No   | AWS IAMユーザーのアクセスキー。AWS S3へのIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key             | No   | AWS IAMユーザーのシークレットキー。AWS S3へのIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |

AWS Glueへのアクセス方法の選択およびAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS Glueへのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocksがストレージシステムと統合する方法に関する一連のパラメータです。このパラメータセットはオプションです。

ストレージとしてHDFSを使用する場合、`StorageCredentialParams`を構成する必要はありません。

AWS S3、その他のS3互換ストレージシステム、Microsoft Azure Storage、またはGoogle GCSをストレージとして使用する場合、`StorageCredentialParams`を構成する必要があります。

##### AWS S3

HudiクラスタのストレージとしてAWS S3を選択する場合、次のいずれかの操作を実行します。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 仮定されたロールベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

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

以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

| パラメータ名                   | 必須 | 説明                                                         |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile   | Yes  | インスタンスプロファイルベースの認証方法および仮定されたロールベースの認証方法を有効にするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。 |
| aws.s3.iam_role_arn           | No   | AWS S3バケットに特権を持つIAMロールのARN。AWS S3への仮定されたロールベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.s3.region                 | Yes  | AWS S3バケットが存在するリージョン。例：`us-west-1`。         |
| aws.s3.access_key             | No   | IAMユーザーのアクセスキー。AWS S3へのIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key             | No   | IAMユーザーのシークレットキー。AWS S3へのIAMユーザーベースの認証方法を使用する場合、このパラメータを指定する必要があります。 |

AWS S3へのアクセス方法の選択およびAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS S3へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

Hudiカタログは、v2.5以降でS3互換ストレージシステム（たとえば、MinIO）をサポートしています。

S3互換ストレージシステムをストレージとして選択した場合（たとえば、MinIO）、次のように`StorageCredentialParams`を構成して、正常な統合を確保してください。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

| パラメータ名                    | 必須 | 説明                                                         |
| ------------------------------ | ---- | ------------------------------------------------------------ |
| aws.s3.enable_ssl              | Yes  | SSL接続を有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`true`。 |
| aws.s3.enable_path_style_access | Yes  | パス形式のアクセスを有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`false`。MinIOの場合、値を`true`に設定する必要があります。<br />パス形式のURLは、次の形式を使用します：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、US West（Oregon）リージョンで`DOC-EXAMPLE-BUCKET1`というバケットを作成し、そのバケットの`alice.jpg`オブジェクトにアクセスする場合、次のパス形式のURLを使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                | Yes  | AWS S3の代わりにS3互換ストレージシステムに接続するために使用されるエンドポイントです。 |
| aws.s3.access_key              | Yes  | IAMユーザーのアクセスキーです。                              |
| aws.s3.secret_key              | Yes  | IAMユーザーのシークレットキーです。                          |

##### Microsoft Azure Storage

Hudiカタログは、v3.0以降でMicrosoft Azure Storageをサポートしています。

###### Azure Blob Storage

HudiクラスタのストレージとしてBlob Storageを選択する場合、次のいずれかの操作を実行します。

- 共有キー認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**             | **必須** | **説明**                                                     |
  | --------------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.storage_account  | Yes      | Blob Storageアカウントのユーザー名です。                       |
  | azure.blob.shared_key       | Yes      | Blob Storageアカウントの共有キーです。                         |

- SASトークン認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**        | **必須** | **説明**                                                     |
  | ---------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.account_name | Yes      | Blob Storageアカウントのユーザー名です。                       |
  | azure.blob.container_name | Yes      | データを格納するBlobコンテナの名前です。                       |
  | azure.blob.sas_token    | Yes      | Blob Storageアカウントにアクセスするために使用されるSASトークンです。 |

###### Azure Data Lake Storage Gen1

HudiクラスタのストレージとしてData Lake Storage Gen1を選択する場合、次のいずれかの操作を実行します。

- Managed Service Identity認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                            | **必須** | **説明**                                                     |
  | ---------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes      | Managed Service Identity認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |

- Service Principal認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                  | **必須** | **説明**                                                     |
  | ---------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id       | Yes      | サービスプリンシパルのクライアント（アプリケーション）IDです。 |
  | azure.adls1.oauth2_credential      | Yes      | 作成時に生成された新しいクライアント（アプリケーション）シークレットの値です。 |
  | azure.adls1.oauth2_endpoint        | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

###### Azure Data Lake Storage Gen2

HudiクラスタのストレージとしてData Lake Storage Gen2を選択する場合、次のいずれかの操作を実行します。

- Managed Identity認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                           | **必須** | **説明**                                                     |
  | --------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes      | Managed Identity認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | Yes      | アクセスするデータの所属するテナントのIDです。               |
  | azure.adls2.oauth2_client_id            | Yes      | 管理されたIDのクライアント（アプリケーション）IDです。         |

- Shared Key認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                | **必須** | **説明**                                                     |
  | ------------------------------ | -------- | ------------------------------------------------------------ |
  | azure.adls2.storage_account    | Yes      | Data Lake Storage Gen2ストレージアカウントのユーザー名です。   |
  | azure.adls2.shared_key         | Yes      | Data Lake Storage Gen2ストレージアカウントの共有キーです。     |

- Service Principal認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                     | **必須** | **説明**                                                     |
  | --------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id      | Yes      | サービスプリンシパルのクライアント（アプリケーション）IDです。 |
  | azure.adls2.oauth2_client_secret  | Yes      | 作成時に生成された新しいクライアント（アプリケーション）シークレットの値です。 |
  | azure.adls2.oauth2_client_endpoint | Yes      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）です。 |

##### Google GCS

Hudiカタログは、v3.0以降でGoogle GCSをサポートしています。

HudiクラスタのストレージとしてGoogle GCSを選択する場合、次のいずれかの操作を実行します。

- VMベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                              | **デフォルト値** | **値の例** | **説明**                                                     |
  | ------------------------------------------ | ----------------- | ----------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

  | **パラメータ名**                          | **デフォルト値** | **値の例**                                                   | **説明**                                                     |
  | -------------------------------------- | ----------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントのJSONファイル作成時に生成されたメールアドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウントのJSONファイル作成時に生成されたプライベートキーIDです。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウントのJSONファイル作成時に生成されたプライベートキーです。 |

- 偽装ベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成します。

  - VMインスタンスがサービスアカウントを偽装する場合：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

    | **パラメータ名**                              | **デフォルト値** | **値の例** | **説明**                                                     |
    | ------------------------------------------ | ----------------- | ----------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true        | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"     | 偽装したいサービスアカウントです。                            |

  - サービスアカウント（一時的にメタサービスアカウントと呼ばれる）が別のサービスアカウント（一時的にデータサービスアカウントと呼ばれる）を偽装する場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    以下の表に、`StorageCredentialParams`で構成する必要のあるパラメータについて説明します。

    | **パラメータ名**                          | **デフォルト値** | **値の例**                                                   | **説明**                                                     |
    | -------------------------------------- | ----------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントのJSONファイル作成時に生成されたメールアドレスです。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウントのJSONファイル作成時に生成されたプライベートキーIDです。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウントのJSONファイル作成時に生成されたプライベートキーです。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 偽装したいデータサービスアカウントです。                     |

#### MetadataUpdateParams

StarRocksがHudiのキャッシュメタデータを更新する方法に関する一連のパラメータです。このパラメータセットはオプションです。

StarRocksは、デフォルトで[自動非同期更新ポリシー](#appendix-understand-metadata-automatic-asynchronous-update)を実装しています。

ほとんどの場合、`MetadataUpdateParams`を無視し、このパラメータのポリシーパラメータを調整する必要はありません。デフォルト値は、すでに使いやすいパフォーマンスを提供しています。

ただし、Hudiのデータ更新頻度が高い場合は、これらのパラメータを調整して自動非同期更新のパフォーマンスをさらに最適化することができます。

> **注**
>
> ほとんどの場合、Hudiデータが1時間以下の粒度で更新される場合、データの更新頻度は高いと見なされます。

| パラメーター                            | 必須      | 説明                                                         |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | いいえ    | StarRocksがHudiテーブルのメタデータをキャッシュするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| enable_remote_file_cache               | いいえ    | StarRocksがHudiテーブルまたはパーティションの基になるデータファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `true`。`true`はキャッシュを有効にし、`false`はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | いいえ    | StarRocksが自身にキャッシュされたHudiテーブルまたはパーティションのメタデータを非同期で更新する時間間隔。単位: 秒。デフォルト値: `7200`（2時間）。 |
| remote_file_cache_refresh_interval_sec | いいえ    | StarRocksが自身にキャッシュされたHudiテーブルまたはパーティションの基になるデータファイルのメタデータを非同期で更新する時間間隔。単位: 秒。デフォルト値: `60`。 |
| metastore_cache_ttl_sec                | いいえ    | StarRocksが自動的に破棄する自身にキャッシュされたHudiテーブルまたはパーティションのメタデータの時間間隔。単位: 秒。デフォルト値: `86400`（24時間）。 |
| remote_file_cache_ttl_sec              | いいえ    | StarRocksが自動的に破棄する自身にキャッシュされたHudiテーブルまたはパーティションの基になるデータファイルのメタデータの時間間隔。単位: 秒。デフォルト値: `129600`（36時間）。 |

### 例

以下の例では、Hudiクラスターからデータをクエリするために、`hudi_catalog_hms`または`hudi_catalog_glue`という名前のHudiカタログを作成します。

#### HDFS

ストレージとしてHDFSを使用する場合、次のようなコマンドを実行します：

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

##### インスタンスプロファイルベースの認証情報を使用する場合

- HudiクラスターでHiveメタストアを使用する場合、次のようなコマンドを実行します：

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

- Amazon EMR HudiクラスターでAWS Glueを使用する場合、次のようなコマンドを実行します：

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

##### アサムドロールベースの認証情報を使用する場合

- HudiクラスターでHiveメタストアを使用する場合、次のようなコマンドを実行します：

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

- Amazon EMR HudiクラスターでAWS Glueを使用する場合、次のようなコマンドを実行します：

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

##### IAMユーザーベースの認証情報を使用する場合

- HudiクラスターでHiveメタストアを使用する場合、次のようなコマンドを実行します：

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

- Amazon EMR HudiクラスターでAWS Glueを使用する場合、次のようなコマンドを実行します：

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

MinIOを例に使用します。次のようなコマンドを実行します：

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

- 共有キー認証メソッドを選択する場合、次のようなコマンドを実行します：

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

- SASトークン認証メソッドを選択する場合、次のようなコマンドを実行します：

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

- マネージドサービスアイデンティティ認証メソッドを選択する場合、次のようなコマンドを実行します：

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

- サービスプリンシパル認証メソッドを選択する場合、次のようなコマンドを実行します：

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

- マネージドアイデンティティ認証メソッドを選択する場合、次のようなコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 共有キー認証メソッドを選択する場合、次のようなコマンドを実行します：

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

- サービスプリンシパル認証メソッドを選択する場合、次のようなコマンドを実行します：

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

- VMベースの認証メソッドを選択する場合、次のようなコマンドを実行します：

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

- サービスアカウントベースの認証メソッドを選択する場合、次のようなコマンドを実行します：

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

- インパーソネーションベースの認証メソッドを選択する場合：

  - VMインスタンスがサービスアカウントを偽装する場合、次のようなコマンドを実行します：

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

  - サービスアカウントが別のサービスアカウントを偽装する場合、次のようなコマンドを実行します：

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

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスターのすべてのカタログをクエリできます：

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリできます。次の例では、Hudiカタログ`hudi_catalog_glue`の作成ステートメントをクエリしています：

```SQL
SHOW CREATE CATALOG hudi_catalog_glue;
```

## Hudiカタログとその中のデータベースに切り替える

Hudiカタログとその中のデータベースに切り替えるには、次のいずれかの方法を使用できます：

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションでHudiカタログを指定し、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します：

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替える：
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定する：
  USE <db_name>
  ```

- 直接[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、Hudiカタログとその中のデータベースに切り替えます：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Hudiカタログの削除

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、外部カタログを削除できます。

次の例では、Hudiカタログ`hudi_catalog_glue`を削除しています：

```SQL
DROP Catalog hudi_catalog_glue;
```

## Hudiテーブルのスキーマの表示

次の構文のいずれかを使用して、Hudiテーブルのスキーマを表示できます：

- スキーマの表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Hudiテーブルのクエリ

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Hudiクラスター内のデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Hudiカタログとその中のデータベースに切り替えます](#hudiカタログとその中のデータベースに切り替える)。

3. 指定したデータベース内の宛先テーブルをクエリするために[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用します：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hudiからデータをロードする

OLAPテーブル`olap_tbl`があると仮定し、次のようにデータを変換してロードできます：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hudi_table
```

## メタデータキャッシュの手動または自動更新

### 手動更新

デフォルトでは、StarRocksはHudiカタログのメタデータをキャッシュし、非同期モードでメタデータを更新してパフォーマンスを向上させます。また、スキーマの変更やテーブルの更新がHudiテーブルで行われた場合、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)を使用してメタデータを手動で更新し、StarRocksが最新のメタデータをできるだけ早く取得し、適切な実行計画を生成できるようにすることもできます：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自動増分更新

自動増分更新は、StarRocksクラスターのFEがHiveメタストアからイベント（列の追加、パーティションの削除、データの更新など）を読み取ることで、自動的にメタデータを更新するポリシーです。StarRocksはこれらのイベントに基づいてFEにキャッシュされたメタデータを自動的に更新できます。つまり、メタデータの手動更新は不要です。

自動増分更新を有効にするには、次の手順に従います。

#### ステップ1：Hiveメタストアにイベントリスナーを設定する

Hiveメタストアのv2.xおよびv3.xの両方は、イベントリスナーを設定することができます。この手順では、Hiveメタストアv3.1.2で使用されるイベントリスナーの設定を例に説明します。次の構成項目を**$HiveMetastore/conf/hive-site.xml**ファイルに追加し、Hiveメタストアを再起動します。

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

FEのログファイルで`event id`を検索して、イベントリスナーが正常に設定されているかどうかを確認できます。設定が失敗した場合、`event id`の値は`0`です。

#### ステップ2：StarRocksで自動増分更新を有効にする

StarRocksで単一のHudiカタログまたはすべてのHudiカタログに対して自動増分更新を有効にすることができます。

- 単一のHudiカタログに自動増分更新を有効にするには、Hudiカタログを作成する際に`enable_hms_events_incremental_sync`パラメータを`true`に設定します：

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

- すべてのHudiカタログに自動増分更新を有効にするには、各FEの`$FE_HOME/conf/fe.conf`ファイルに`"enable_hms_events_incremental_sync" = "true"`を追加し、各FEを再起動してパラメータ設定を有効にします。

ビジネス要件に基づいて、各FEの`$FE_HOME/conf/fe.conf`ファイルで次のパラメータを調整し、各FEを再起動してパラメータ設定を有効にすることもできます。

| パラメーター                         | 説明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocksがHiveメタストアからイベントを読み取る時間間隔。デフォルト値: `5000`。単位: ミリ秒。 |
| hms_events_batch_size_per_rpc     | StarRocksが一度に読み取ることができるイベントの最大数。デフォルト値: `500`。 |
| enable_hms_parallel_process_evens | StarRocksがイベントを読み取りながらイベントを並列で処理するかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `true`。`true`は並列処理を有効にし、`false`は並列処理を無効にします。 |
| hms_process_events_parallel_num   | StarRocksが並列で処理できるイベントの最大数。デフォルト値: `4`。 |

## 付録：メタデータの自動非同期更新の理解

自動非同期更新は、StarRocksがHudiカタログのメタデータを更新するために使用するデフォルトのポリシーです。

デフォルトでは（つまり、`enable_metastore_cache`パラメータと`enable_remote_file_cache`パラメータの両方が`true`に設定されている場合）、クエリがHudiテーブルのパーティションにヒットすると、StarRocksはそのパーティションのメタデータとパーティションの基になるデータファイルのメタデータを自動的にキャッシュします。キャッシュされたメタデータは遅延更新ポリシーを使用して更新されます。

例えば、`table2`という名前のHudiテーブルがあり、`p1`、`p2`、`p3`、`p4`の4つのパーティションがあります。クエリが`p1`にヒットし、StarRocksは`p1`のメタデータと`p1`の基になるデータファイルのメタデータをキャッシュします。次のようなデフォルトの時間間隔を更新および破棄する時間間隔として仮定します。

- `p1`のキャッシュされたメタデータの非同期更新時間間隔（`metastore_cache_refresh_interval_sec`パラメータで指定）は2時間です。
- `p1`の基になるデータファイルのキャッシュされたメタデータの非同期更新時間間隔（`remote_file_cache_refresh_interval_sec`パラメータで指定）は60秒です。
- `p1`のキャッシュされたメタデータを自動的に破棄する時間間隔（`metastore_cache_ttl_sec`パラメータで指定）は24時間です。
- `p1`の基になるデータファイルのキャッシュされたメタデータを自動的に破棄する時間間隔（`remote_file_cache_ttl_sec`パラメータで指定）は36時間です。

以下の図は、理解しやすくするためのタイムライン上の時間間隔を示しています。

![キャッシュされたメタデータの更新と破棄のタイムライン](../../assets/catalog_timeline.png)

その後、StarRocksは次のルールに従ってメタデータを更新または破棄します。

- もう一度`p1`にヒットし、最後の更新からの現在の時間が60秒未満の場合、StarRocksは`p1`のキャッシュされたメタデータまたは`p1`の基になるデータファイルのキャッシュされたメタデータを更新しません。
- もう一度`p1`にヒットし、最後の更新からの現在の時間が60秒以上の場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを更新します。
- もう一度`p1`にヒットし、最後の更新からの現在の時間が2時間以上の場合、StarRocksは`p1`のキャッシュされたメタデータを更新します。
- 最後の更新から24時間以内に`p1`にアクセスがない場合、StarRocksは`p1`のキャッシュされたメタデータを破棄します。次のクエリでメタデータがキャッシュされます。
- 最後の更新から36時間以内に`p1`にアクセスがない場合、StarRocksは`p1`の基になるデータファイルのキャッシュされたメタデータを破棄します。次のクエリでメタデータがキャッシュされます。
