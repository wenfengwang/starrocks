---
displayed_sidebar: Chinese
---

# Delta Lake カタログ

Delta Lake カタログは、External Catalogの一種です。Delta Lake カタログを使用することで、データのインポートなしでDelta Lakeのデータを直接クエリできます。

また、Delta Lake カタログを使用して、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)の機能を組み合わせてデータの変換とインポートを行うこともできます。StarRocksは2.5バージョンからDelta Lake カタログをサポートしています。

Delta Lakeのデータに正常にアクセスするためには、StarRocksクラスタに以下の2つの重要なコンポーネントを統合する必要があります：

- 分散ファイルシステム（HDFS）またはオブジェクトストレージ。サポートされているオブジェクトストレージには、AWS S3、Microsoft Azure Storage、Google GCS、その他のS3プロトコル互換のオブジェクトストレージ（例：Alibaba Cloud OSS、MinIO）があります。

- メタデータサービス。サポートされているメタデータサービスには、Hive Metastore（以下HMS）、AWS Glueがあります。

  > **注意**
  >
  > ストレージシステムとしてAWS S3を選択する場合、メタデータサービスとしてHMSまたはAWS Glueを選択できます。他のストレージシステムを選択する場合は、メタデータサービスとしてHMSのみを選択できます。

## 使用方法

- StarRocksはDelta Lakeのデータをクエリする際、Parquetファイル形式をサポートしています。ParquetファイルはSNAPPY、LZ4、ZSTD、GZIP、NO_COMPRESSIONの圧縮形式に対応しています。
- StarRocksはDelta Lakeのデータをクエリする際、MAPおよびSTRUCTデータ型はサポートしていません。

## 準備

Delta Lakeカタログを作成する前に、StarRocksクラスタがDelta Lakeのファイルストレージとメタデータサービスに正常にアクセスできることを確認してください。

### AWS IAM

Delta LakeがファイルストレージとしてAWS S3を使用する場合、またはメタデータサービスとしてAWS Glueを使用する場合、StarRocksクラスタが関連するAWSクラウドリソースにアクセスできるように、適切な認証および認可方法を選択する必要があります。

次の認証および認可方法を選択できます：

- インスタンスプロファイル（推奨）
- 引き受けた役割
- IAMユーザー

AWSリソースへのアクセスに関する詳細な情報については、[AWS認証方法の設定 - 準備](../../integrations/authenticate_to_aws_resources.md#準備工作)を参照してください。

### HDFS

ファイルストレージとしてHDFSを使用する場合、StarRocksクラスタで次の設定を行う必要があります：

- （オプション）HDFSクラスタとHMSへのアクセスに使用するユーザー名を設定します。各FEの**fe/conf/hadoop_env.sh**ファイルと各BEの**be/conf/hadoop_env.sh**ファイルの先頭に`export HADOOP_USER_NAME="<user_name>"`を追加して、このユーザー名を設定できます。設定後、各FEとBEを再起動して設定を有効にする必要があります。このユーザー名を設定しない場合、デフォルトでFEとBEプロセスのユーザー名が使用されます。StarRocksクラスタでは、1つのユーザー名の設定のみがサポートされます。
- Delta Lakeのデータをクエリする際、StarRocksクラスタのFEとBEはHDFSクライアントを介してHDFSクラスタにアクセスします。通常、StarRocksはデフォルトの設定でHDFSクライアントを起動するため、手動での設定は不要です。ただし、次のシナリオでは手動での設定が必要です：
  - HDFSクラスタが高可用性（High Availability、HA）モードで動作している場合、HDFSクラスタの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに配置する必要があります。
  - HDFSクラスタでViewFsが構成されている場合、HDFSクラスタの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに配置する必要があります。

> **注意**
>
> クエリの実行中にホスト名が認識できない（Unknown Host）エラーが発生する場合は、HDFSクラスタの各ノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに設定する必要があります。

### Kerberos認証

HDFSクラスタまたはHMSがKerberos認証を有効にしている場合、StarRocksクラスタで次の設定を行う必要があります：

- 各FEおよび各BEで`kinit -kt keytab_path principal`コマンドを実行し、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。コマンドを実行するユーザーは、HMSおよびHDFSにアクセスする権限を持っている必要があります。なお、このコマンドを使用してKDCにアクセスする場合は、有効期限があるため、このコマンドを定期的にcronで実行する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルと各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。ここで、`/etc/krb5.conf`は**krb5.conf**ファイルのパスであり、ファイルの実際のパスに応じて変更することができます。

## Delta Lake カタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "deltalake",
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### パラメータの説明

#### catalog_name

Delta Lake カタログの名前。以下の要件を満たす必要があります：

- 英字（a-zまたはA-Z）、数字（0-9）、またはアンダースコア（_）で構成する必要があります。また、英字で始まる必要があります。
- 総文字数は1023文字以下である必要があります。
- カタログ名は大文字と小文字を区別します。

#### comment

Delta Lake カタログの説明。このパラメータはオプションです。

#### type

データソースのタイプ。`deltalake`に設定します。

#### MetastoreParams

StarRocksがDelta Lakeクラスタのメタデータサービスにアクセスするための関連するパラメータの設定。

##### HMS

Delta LakeクラスタのメタデータサービスとしてHMSを選択する場合、次のように`MetastoreParams`を設定します：

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注意**
>
> Delta Lakeのデータをクエリする前に、すべてのHMSノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに追加する必要があります。そうしないと、クエリの実行時にStarRocksがHMSにアクセスできない場合があります。

`MetastoreParams`には次のパラメータが含まれます。

| パラメータ                | 必須   | 説明                                                         |
| ------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type (英語) | 必須       | Delta Lakeクラスタで使用するメタデータサービスのタイプ。`hive`に設定します。           |
| hive.metastore.uris です | 必須       | HMSのURI。形式:`thrift://<HMS IP アドレス>:<HMS ポート番号>`。<br />HMSが高可用性モードで動作している場合、複数のHMSアドレスを指定し、カンマで区切ることができます。例：`"thrift://<HMS IP アドレス 1>:<HMS ポート番号 1>,thrift://<HMS IP アドレス 2>:<HMS ポート番号 2>,thrift://<HMS IP アドレス 3>:<HMS ポート番号 3>"`。 |

##### AWS Glue

Delta LakeクラスタのメタデータサービスとしてAWS Glueを選択する場合（ストレージシステムとしてAWS S3を使用する場合のみサポート）、次のように`MetastoreParams`を設定します：

- インスタンスプロファイルを使用した認証と認可

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- Assume Roleを使用した認証と認可

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAM Userを使用した認証と認可

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

`MetastoreParams`には次のパラメータが含まれます。

| パラメータ                          | 必須   | 説明                                                         |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type (英語)           | 必須       | Delta Lakeクラスタで使用するメタデータサービスのタイプ。`glue`に設定します。           |
| aws.glue.use_instance_profile | 必須       | インスタンスプロファイルとAssume Roleの2つの認証方法を有効にするかどうかを指定します。`true`または`false`の値を取ります。デフォルト値は`false`です。 |
| aws.glue.iam_role_arn         | オプション       | AWS Glue Data CatalogにアクセスできるIAMロールのARN。Assume Roleの認証方法でAWS Glueにアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.region (英語)               | 必須       | AWS Glue Data Catalogのリージョン。例：`us-west-1`。        |
| aws.glue.access_key           | オプション       | IAM UserのAccess Key。IAM Userの認証方法でAWS Glueにアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | オプション       | IAM UserのSecret Key。IAM Userの認証方法でAWS Glueにアクセスする場合、このパラメータを指定する必要があります。 |

AWS Glueへのアクセスに使用する認証方法の選択方法、およびAWS IAMコンソールでアクセス制御ポリシーを設定する方法については、[AWS Glueへのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#aws-glue-へのアクセスの認証パラメータ)を参照してください。

#### StorageCredentialParams (英語)

StarRocksがDelta Lakeクラスタのファイルストレージにアクセスするための関連するパラメータの設定。

ファイルストレージとしてHDFSを使用する場合、`StorageCredentialParams`の設定は必要ありません。

AWS S3、その他のS3プロトコル互換のオブジェクトストレージ、Microsoft Azure Storage、またはGCSを使用する場合、`StorageCredentialParams`の設定が必要です。

##### AWS S3

Delta LakeクラスタのファイルストレージとしてAWS S3を選択する場合、次のように`StorageCredentialParams`を設定します：

- インスタンスプロファイルを使用した認証と認可

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- Assume Roleを使用した認証と認可

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM Userを使用した認証と認可

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

`StorageCredentialParams`には次のパラメータが含まれます。

| パラメータ                        | 必須   | 説明                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | 必須       | インスタンスプロファイルとAssume Roleの2つの認証方法を有効にするかどうかを指定します。`true`または`false`の値を取ります。デフォルト値は`false`です。 |
| aws.s3.iam_role_arn         | オプション       | AWS S3バケットにアクセスできるIAMロールのARN。Assume Roleの認証方法でAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.region (英語)               | 必須       | AWS S3バケットのリージョン。例：`us-west-1`。                |
| aws.s3.access_key           | オプション       | IAM UserのAccess Key。IAM Userの認証方法でAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | 否       | IAM User の Secret Key。IAM User 認証方式で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3 へのアクセスに使用する認証方式の選択方法や、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS S3 の認証パラメータ](../../integrations/authenticate_to_aws_resources.md#访问-aws-s3-の认证参数)を参照してください。

##### 阿里云 OSS

Delta Lake クラスタのファイルストレージとして阿里云 OSS を選択する場合、`StorageCredentialParams` で以下の認証パラメータを設定する必要があります：

```SQL
"aliyun.oss.access_key" = "<user_access_key>",
"aliyun.oss.secret_key" = "<user_secret_key>",
"aliyun.oss.endpoint" = "<oss_endpoint>" 
```

| パラメータ                        | 必須 | 説明                                                         |
| ------------------------------- | ---- | ------------------------------------------------------------ |
| aliyun.oss.endpoint             | はい | 阿里云 OSS のエンドポイント。例：`oss-cn-beijing.aliyuncs.com`。エンドポイントとリージョンの関係を調べるには、[アクセスドメインとデータセンター](https://help.aliyun.com/document_detail/31837.html)を参照してください。 |
| aliyun.oss.access_key           | はい | 阿里云アカウントまたは RAM ユーザーの AccessKey ID。取得方法については、[AccessKey の取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。 |
| aliyun.oss.secret_key           | はい | 阿里云アカウントまたは RAM ユーザーの AccessKey Secret。取得方法については、[AccessKey の取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。 |

##### S3 プロトコル互換オブジェクトストレージ

Delta Lake Catalog はバージョン 2.5 から S3 プロトコル互換のオブジェクトストレージをサポートしています。

S3 プロトコル互換のオブジェクトストレージ（例：MinIO）を Delta Lake クラスタのファイルストレージとして選択する場合、`StorageCredentialParams` を以下のように設定してください：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

`StorageCredentialParams` には以下のパラメータが含まれます。

| パラメータ                             | 必須   | 説明                                                  |
| -------------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | はい | SSL 接続を有効にするかどうか。<br />選択肢：`true` または `false`。デフォルト値：`true`。 |
| aws.s3.enable_path_style_access  | はい | パススタイルアクセス（Path-Style Access）を有効にするかどうか。<br />選択肢：`true` または `false`。デフォルト値：`false`。MinIO の場合は `true` に設定する必要があります。<br />パススタイルの URL は以下の形式です：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部（オレゴン）リージョンに `DOC-EXAMPLE-BUCKET1` というバケットを作成し、その中の `alice.jpg` オブジェクトにアクセスしたい場合、以下のパススタイルの URL を使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | はい | S3 プロトコル互換のオブジェクトストレージにアクセスするためのエンドポイント。 |
| aws.s3.access_key                | はい | IAM User の Access Key。 |
| aws.s3.secret_key                | はい | IAM User の Secret Key。 |

##### Microsoft Azure ストレージ

Delta Lake Catalog はバージョン 3.0 から Microsoft Azure Storage をサポートしています。

###### Azure Blob Storage

Azure Blob Storage を Delta Lake クラスタのファイルストレージとして選択する場合、`StorageCredentialParams` を以下のように設定してください：

- Shared Key を使用した認証と承認

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                   | **必須** | **説明**                         |
  | -------------------------- | ---- | -------------------------------- |
  | azure.blob.storage_account | はい | Blob Storage アカウントのユーザー名。 |
  | azure.blob.shared_key      | はい | Blob Storage アカウントの Shared Key。 |

- SAS Token を使用した認証と承認

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                  | **必須** | **説明**                                 |
  | ------------------------- | ---- | ---------------------------------------- |
  | azure.blob.storage_account| はい | Blob Storage アカウントのユーザー名。     |
  | azure.blob.container      | はい | Blob コンテナの名前。                    |
  | azure.blob.sas_token      | はい | Blob Storage アカウントへのアクセスに使用する SAS Token。 |

###### Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 を Delta Lake クラスタのファイルストレージとして選択する場合、`StorageCredentialParams` を以下のように設定してください：

- Managed Service Identity を使用した認証と承認

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                                 | **必須** | **説明**                                                     |
  | ---------------------------------------- | ---- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい | Managed Service Identity 認証方式を有効にするかどうか。`true` に設定します。 |

- Service Principal を使用した認証と承認

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                 | **必須** | **説明**                                              |
  | ----------------------------- | ---- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい | Service Principal のクライアント（アプリケーション）ID。     |
  | azure.adls1.oauth2_credential | はい | 作成したクライアント（アプリケーション）のシークレット。    |
  | azure.adls1.oauth2_endpoint   | はい | Service Principal またはアプリケーションの OAuth 2.0 トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2

Azure Data Lake Storage Gen2 を Delta Lake クラスタのファイルストレージとして選択する場合、`StorageCredentialParams` を以下のように設定してください：

- Managed Identity を使用した認証と承認

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                                | **必須** | **説明**                                                |
  | --------------------------------------- | ---- | ------------------------------------------------------- |
  | azure.adls2.oauth2_use_managed_identity | はい | Managed Identity 認証方式を有効にするかどうか。`true` に設定します。 |
  | azure.adls2.oauth2_tenant_id            | はい | テナントの ID。                                          |
  | azure.adls2.oauth2_client_id            | はい | Managed Identity のクライアント（アプリケーション）ID。  |

- Shared Key を使用した認証と承認

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                    | **必須** | **説明**                                   |
  | --------------------------- | ---- | ------------------------------------------ |
  | azure.adls2.storage_account | はい | Data Lake Storage Gen2 アカウントのユーザー名。 |
  | azure.adls2.shared_key      | はい | Data Lake Storage Gen2 アカウントの Shared Key。 |

- Service Principal を使用した認証と承認

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                           | **必須** | **説明**                                                     |
  | ---------------------------------- | ---- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい | Service Principal のクライアント（アプリケーション）ID。     |
  | azure.adls2.oauth2_client_secret   | はい | 作成したクライアント（アプリケーション）のシークレット。    |
  | azure.adls2.oauth2_client_endpoint | はい | Service Principal またはアプリケーションの OAuth 2.0 トークンエンドポイント（v1）。 |

##### Google GCS

Delta Lake Catalog はバージョン 3.0 から Google GCS をサポートしています。

Google GCS を Delta Lake クラスタのファイルストレージとして選択する場合、`StorageCredentialParams` を以下のように設定してください：

- VM を使用した認証と承認

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                                   | **デフォルト値** | **値の例** | **説明**                                                 |
  | ------------------------------------------ | ------------ | --------- | -------------------------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | false        | true      | Compute Engine に紐づけられた Service Account を直接使用するかどうか。 |

- Service Account を使用した認証と承認

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                               | **デフォルト値** | **値の例**                                                 | **説明**                                                     |
  | -------------------------------------- | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""           | "user@hello.iam.gserviceaccount.com" | Service Account を作成した際に生成された JSON ファイル内の Email。 |
  | gcp.gcs.service_account_private_key_id | ""           | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | Service Account を作成した際に生成された JSON ファイル内の Private Key ID。 |
  | gcp.gcs.service_account_private_key    | ""           | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n" | Service Account を作成した際に生成された JSON ファイル内の Private Key。 |

- Impersonation を使用した認証と承認

  - VM インスタンスで Service Account を模倣

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | **パラメータ**                                   | **デフォルト値** | **値の例** | **説明**                                                     |
    | ------------------------------------------ | ------------ | --------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false        | true      | Compute Engine に紐づけられた Service Account を直接使用するかどうか。 |
    | gcp.gcs.impersonation_service_account      | ""           | "hello"   | 模倣する対象の Service Account。                             |

  - ある Service Account（仮に "Meta Service Account" とします）を使用して別の Service Account（仮に "Data Service Account" とします）を模倣

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_private_key_id>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_private_key>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。


    | **パラメータ**                         | **デフォルト値** | **取得例**                                                   | **説明**                                                     |
    | -------------------------------------- | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""               | 「[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)」 | Meta Service Account を作成する際に生成される JSON ファイル内の Email。 |
    | gcp.gcs.service_account_private_key_id | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | Meta Service Account を作成する際に生成される JSON ファイル内の Private Key ID。 |
    | gcp.gcs.service_account_private_key    | ""               | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | Meta Service Account を作成する際に生成される JSON ファイル内の Private Key。 |
    | gcp.gcs.impersonation_service_account  | ""               | "hello"                                                      | 模擬する必要がある対象の Data Service Account。              |

#### MetadataUpdateParams

キャッシュされたメタデータの更新戦略を指定するパラメータのセット。StarRocks はこの戦略に基づいて Delta Lake のメタデータを更新します。このパラメータグループはオプションです。

StarRocks はデフォルトで[自動非同期更新戦略](#附录理解元数据自动异步更新策略)を採用しており、すぐに使用できます。したがって、通常は `MetadataUpdateParams` を無視して、戦略パラメータのチューニングを行う必要はありません。

Delta Lake のデータ更新頻度が高い場合は、これらのパラメータをチューニングして、自動非同期更新戦略のパフォーマンスを最適化することができます。

| パラメータ                                | 必須かどうか | 説明                                                         |
| -------------------------------------- | ------------ | ------------------------------------------------------------ |
| enable_metastore_cache            | いいえ       | StarRocks が Delta Lake のテーブルメタデータをキャッシュするかどうかを指定します。値の範囲: `true` または `false`。デフォルト値: `true`。`true` はキャッシュを有効にし、`false` はキャッシュを無効にします。 |
| enable_remote_file_cache               | いいえ       | StarRocks が Delta Lake のテーブルまたはパーティションのデータファイルのメタデータをキャッシュするかどうかを指定します。値の範囲: `true` または `false`。デフォルト値: `true`。`true` はキャッシュを有効にし、`false` はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | いいえ       | StarRocks がキャッシュされた Delta Lake のテーブルまたはパーティションのメタデータを非同期で更新する間隔。単位: 秒。デフォルト値: `7200`（2時間）。 |
| remote_file_cache_refresh_interval_sec | いいえ       | StarRocks がキャッシュされた Delta Lake のテーブルまたはパーティションのデータファイルのメタデータを非同期で更新する間隔。単位: 秒。デフォルト値: `60`。 |
| metastore_cache_ttl_sec                | いいえ       | StarRocks がキャッシュされた Delta Lake のテーブルまたはパーティションのメタデータを自動的に削除するまでの時間。単位: 秒。デフォルト値: `86400`（24時間）。 |
| remote_file_cache_ttl_sec              | いいえ       | StarRocks がキャッシュされた Delta Lake のテーブルまたはパーティションのデータファイルのメタデータを自動的に削除するまでの時間。単位: 秒。デフォルト値: `129600`（36時間）。 |

### 例

以下の例では、Delta Lake クラスタ内のデータをクエリするために `deltalake_catalog_hms` または `deltalake_catalog_glue` という名前の Delta Lake カタログを作成します。

#### HDFS

HDFS をストレージとして使用する場合、以下のように Delta Lake カタログを作成できます:

```SQL
CREATE EXTERNAL CATALOG deltalake_catalog_hms
PROPERTIES
(
    "type" = "deltalake",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

#### AWS S3

##### Instance Profile を使用した認証と認可の場合

- Delta Lake クラスタが HMS をメタデータサービスとして使用している場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Delta Lake クラスタが AWS Glue をメタデータサービスとして使用している場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_glue
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### Assumed Role を使用した認証と認可の場合

- Delta Lake クラスタが HMS をメタデータサービスとして使用している場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Delta Lake クラスタが AWS Glue をメタデータサービスとして使用している場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_glue
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### IAM User を使用した認証と認可の場合

- Delta Lake クラスタが HMS をメタデータサービスとして使用している場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Delta Lake クラスタが AWS Glue をメタデータサービスとして使用している場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_glue
  PROPERTIES
  (
      "type" = "deltalake",
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

#### S3 プロトコル互換のオブジェクトストレージ

MinIO を例に、以下のように Delta Lake カタログを作成できます:

```SQL
CREATE EXTERNAL CATALOG deltalake_catalog_hms
PROPERTIES
(
    "type" = "deltalake",
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

- Shared Key を使用した認証と認可の場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SAS Token を使用した認証と認可の場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- Managed Service Identity を使用した認証と認可の場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- Service Principal を使用した認証と認可の場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- Managed Identity を使用した認証と認可の場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- Shared Key を使用した認証と認可の場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- Service Principal を使用した認証と認可の場合、以下のように Delta Lake カタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMを使用して認証と認可を行う場合、以下のようにDelta Lake Catalogを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- Service Accountを使用して認証と認可を行う場合、以下のようにDelta Lake Catalogを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- Impersonationを使用して認証と認可を行う場合

  - VMインスタンスでService Accountを模倣する場合、以下のようにDelta Lake Catalogを作成できます：

    ```SQL
    CREATE EXTERNAL CATALOG deltalake_catalog_hms
    PROPERTIES
    (
        "type" = "deltalake",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    );
    ```

  - あるService Accountで別のService Accountを模倣する場合、以下のようにDelta Lake Catalogを作成できます：

    ```SQL
    CREATE EXTERNAL CATALOG deltalake_catalog_hms
    PROPERTIES
    (
        "type" = "deltalake",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Delta Lake Catalogの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスター内のすべてのCatalogを照会できます：

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、特定のExternal Catalogの作成ステートメントを照会できます。例えば、以下のコマンドでDelta Lake Catalog `deltalake_catalog_glue`の作成ステートメントを照会できます：

```SQL
SHOW CREATE CATALOG deltalake_catalog_glue;
```

## Delta Lake Catalogとデータベースの切り替え

以下の方法で目的のDelta Lake Catalogとデータベースに切り替えることができます：

- まず[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して現在のセッションに有効なDelta Lake Catalogを指定し、次に[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してデータベースを指定します：

  ```SQL
  -- 現在のセッションに有効なCatalogを切り替えます：
  SET CATALOG <catalog_name>
  -- 現在のセッションに有効なデータベースを指定します：
  USE <db_name>
  ```

- [USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、セッションを直接目的のDelta Lake Catalogの指定されたデータベースに切り替えます：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Delta Lake Catalogの削除

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、特定のExternal Catalogを削除できます。

例えば、以下のコマンドでDelta Lake Catalog `deltalake_catalog_glue`を削除できます：

```SQL
DROP CATALOG deltalake_catalog_glue;
```

## Delta Lakeテーブル構造の表示

以下の方法でDelta Lakeテーブルの構造を表示できます：

- テーブル構造を表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEコマンドからテーブル構造とテーブルファイルの保存場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Delta Lakeテーブルデータの照会

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、指定されたCatalogに属するDelta Lakeクラスター内のデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Delta Lake Catalogとデータベースへの切り替え](#delta-lake-catalogとデータベースの切り替え)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、目的のデータベース内の目的のテーブルを照会します：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Delta Lakeデータのインポート

OLAPテーブルがあり、そのテーブル名が`olap_tbl`であるとします。以下のようにしてそのテーブルのデータを変換し、StarRocksにインポートできます：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM deltalake_table
```

## メタデータキャッシュの手動または自動更新

### 手動更新

デフォルトでは、StarRocksはDelta Lakeのメタデータをキャッシュし、非同期モードでキャッシュされたメタデータを自動的に更新して、クエリのパフォーマンスを向上させます。さらに、Delta Lakeテーブルの構造を変更したり、他のテーブルの更新を行った後、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)を使用してそのテーブルのメタデータを手動で更新し、StarRocksが迅速に適切なクエリプランを生成することを確実にできます：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自動増分更新

自動非同期更新戦略とは異なり、自動増分更新戦略では、FEは定期的にHMSからさまざまなイベントを読み取り、Delta Lakeテーブルのメタデータの変更を感知できます。これには、列の追加や削除、パーティションの追加や削除、パーティションデータの更新などが含まれます。Delta Lakeテーブルのメタデータを手動で更新する必要はありません。

自動増分更新戦略を開始する手順は以下の通りです：

#### ステップ1：HMSでイベントリスナーを設定する

HMS 2.xおよび3.xバージョンは、イベントリスナーの設定をサポートしています。ここでは、HMS 3.1.2バージョンのイベントリスナー設定を例にしています。以下の設定項目を**$HiveMetastore/conf/hive-site.xml**ファイルに追加し、HMSを再起動します：

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

設定が完了したら、FEのログファイルで`event id`を検索し、イベントIDを確認してイベントリスナーが正常に設定されているかをチェックできます。設定に失敗した場合、すべての`event id`は`0`になります。

#### ステップ2：StarRocksで自動増分更新戦略を開始する

StarRocksクラスター内の特定のDelta Lake Catalogに自動増分更新戦略を開始することも、すべてのDelta Lake Catalogに自動増分更新戦略を開始することもできます。

- 特定のDelta Lake Catalogに自動増分更新戦略を開始する場合は、そのDelta Lake Catalogを作成する際に`PROPERTIES`内の`enable_hms_events_incremental_sync`パラメータを`true`に設定する必要があります。以下のようになります：

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```
  
- すべてのDelta Lake Catalogに自動増分更新戦略を開始する場合は、`enable_hms_events_incremental_sync`パラメータを各FEの**$FE_HOME/conf/fe.conf**ファイルに追加し、`true`に設定した後、FEを再起動してパラメータ設定を有効にする必要があります。

ビジネスの要件に応じて、各FEの**$FE_HOME/conf/fe.conf**ファイルで以下のパラメータを調整し、FEを再起動してパラメータ設定を有効にすることもできます。

| パラメータ                         | 説明                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocksがHMSからイベントを読み取る間隔。デフォルト値は`5000`。単位はミリ秒です。  |
| hms_events_batch_size_per_rpc     | StarRocksが一度に読み取るイベントの最大数。デフォルト値は`500`です。                  |
| enable_hms_parallel_process_evens | StarRocksがイベントを読み取る際に、読み取ったイベントを並行して処理するかどうかを指定します。値の範囲は`true`または`false`です。デフォルト値は`true`です。`true`を設定すると並行処理が有効になり、`false`を設定すると並行処理が無効になります。 |
| hms_process_events_parallel_num   | StarRocksが一度に処理するイベントの最大並行数。デフォルト値は`4`です。                  |

## 付録：メタデータ自動非同期更新戦略の理解

自動非同期更新戦略は、StarRocksがDelta Lake Catalog内のメタデータを更新するためのデフォルト戦略です。

デフォルトでは（つまり、`enable_metastore_cache`パラメータと`enable_remote_file_cache`パラメータが両方とも`true`に設定されている場合）、クエリがDelta Lakeテーブルの特定のパーティションをヒットすると、StarRocksはそのパーティションのメタデータとそのパーティション内のデータファイルのメタデータを自動的にキャッシュします。キャッシュされたメタデータは、遅延更新（Lazy Update）戦略を採用しています。

例えば、`table2`という名前のDelta Lakeテーブルがあり、そのデータは`p1`、`p2`、`p3`、`p4`の4つのパーティションに分散しています。クエリが`p1`をヒットすると、StarRocksは`p1`のメタデータと`p1`内のデータファイルのメタデータを自動的にキャッシュします。現在のキャッシュされたメタデータの更新と廃棄戦略が以下のように設定されているとします：

- `p1`のキャッシュされたメタデータを非同期で更新する間隔（`metastore_cache_refresh_interval_sec`パラメータで指定）は2時間です。
- `p1`内のデータファイルのキャッシュされたメタデータを非同期で更新する間隔（`remote_file_cache_refresh_interval_sec`パラメータで指定）は60秒です。
- `p1`のキャッシュされたメタデータを自動的に廃棄する間隔（`metastore_cache_ttl_sec`パラメータで指定）は24時間です。
- `p1`内のデータファイルのキャッシュされたメタデータを自動的に廃棄する間隔（`remote_file_cache_ttl_sec`パラメータで指定）は36時間です。

以下の図に示されています。

![タイムラインのポリシーの更新](../../assets/catalog_timeline_zh.png)

StarRocksは以下の戦略を採用してキャッシュされたメタデータを更新および廃棄します：

- 別のクエリが再び`p1`をヒットし、現在の時刻が前回の更新から60秒以内である場合、StarRocksは`p1`のキャッシュされたメタデータも`p1`内のデータファイルのキャッシュされたメタデータも更新しません。
- もし別のクエリが `p1` を再びヒットし、かつ現在の時刻が最後の更新から60秒以上経過している場合、StarRocks は `p1` のデータファイルのキャッシュメタデータを更新します。
- もし別のクエリが `p1` を再びヒットし、かつ現在の時刻が最後の更新から2時間以上経過している場合、StarRocks は `p1` のキャッシュメタデータを更新します。
- 前回の更新が終了してから24時間以内に `p1` がアクセスされなかった場合、StarRocks は `p1` のキャッシュメタデータを削除します。その後、クエリが再び `p1` をヒットした場合、`p1` のメタデータを再キャッシュします。
- 前回の更新が終了してから36時間以内に `p1` がアクセスされなかった場合、StarRocks は `p1` のデータファイルのキャッシュメタデータを削除します。その後、クエリが再び `p1` をヒットした場合、`p1` のデータファイルのメタデータを再キャッシュします。
