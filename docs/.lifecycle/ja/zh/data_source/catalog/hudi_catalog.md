---
displayed_sidebar: Chinese
---

# Hudiカタログ

Hudi CatalogはExternal Catalogの一種です。Hudi Catalogを使用することで、データのインポートを行わずにApache Hudi内のデータを直接クエリすることができます。

さらに、Hudi Catalogを基にして[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)機能を組み合わせることで、データ変換やインポートを実現できます。StarRocksはバージョン2.4からHudi Catalogをサポートしています。

Hudi内のデータに正常にアクセスするために、StarRocksクラスターは以下の2つの重要なコンポーネントを統合する必要があります：

- 分散ファイルシステム（HDFS）またはオブジェクトストレージ。現在サポートされているオブジェクトストレージには、AWS S3、Microsoft Azure Storage、Google GCS、その他のS3プロトコル互換のオブジェクトストレージ（例：アリババクラウドOSS、MinIO）が含まれます。

- メタデータサービス。現在サポートされているメタデータサービスには、Hive Metastore（以下、HMSと略）とAWS Glueがあります。

  > **説明**
  >
  > AWS S3をストレージシステムとして選択した場合、HMSまたはAWS Glueをメタデータサービスとして選択できます。他のストレージシステムを選択した場合は、HMSのみをメタデータサービスとして選択できます。

## 使用説明

- StarRocksがHudiデータをクエリする際には、Parquetファイル形式をサポートしています。ParquetファイルはSNAPPY、LZ4、ZSTD、GZIP、NO_COMPRESSIONの圧縮形式をサポートしています。
- StarRocksはHudiのCopy On Write（COW）テーブルとMerge On Read（MOR）テーブルを完全にサポートしています。

## 準備作業

Hudi Catalogを作成する前に、StarRocksクラスターがHudiのファイルストレージおよびメタデータサービスに正常にアクセスできることを確認してください。

### AWS IAM

HudiがAWS S3をファイルストレージとして使用する場合、またはAWS Glueをメタデータサービスとして使用する場合、StarRocksクラスターが関連するAWSクラウドリソースにアクセスできるように、適切な認証および認可スキームを選択する必要があります。

以下の認証および認可スキームを選択できます：

- Instance Profile（推奨）
- Assumed Role
- IAM User

StarRocksがAWS認証を行う詳細については、[AWS認証方法の設定 - 準備作業](../../integrations/authenticate_to_aws_resources.md#準備作業)を参照してください。

### HDFS

HDFSをファイルストレージとして使用する場合、StarRocksクラスターで以下の設定を行う必要があります：

- （オプション）HDFSクラスターおよびHMSにアクセスするためのユーザー名を設定します。各FEの**fe/conf/hadoop_env.sh**ファイルと各BEの**be/conf/hadoop_env.sh**ファイルの先頭に`export HADOOP_USER_NAME="<user_name>"`を追加してユーザー名を設定できます。設定後、各FEおよびBEを再起動して設定を有効にする必要があります。このユーザー名を設定しない場合、デフォルトではFEおよびBEプロセスのユーザー名が使用されます。各StarRocksクラスターは1つのユーザー名のみを設定できます。
- Hudiデータをクエリする際、StarRocksクラスターのFEおよびBEはHDFSクライアントを通じてHDFSクラスターにアクセスします。通常、StarRocksはデフォルト設定に従ってHDFSクライアントを起動し、手動での設定は必要ありません。しかし、以下のシナリオでは手動での設定が必要です：
  - HDFSクラスターがHigh Availability（HA）モードを有効にしている場合、HDFSクラスターの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**ディレクトリおよび各BEの**$BE_HOME/conf**ディレクトリに配置する必要があります。
  - HDFSクラスターがViewFsを設定している場合、HDFSクラスターの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**ディレクトリおよび各BEの**$BE_HOME/conf**ディレクトリに配置する必要があります。

> **注意**
>
> クエリ時にドメイン名が認識できない（Unknown Host）ためにアクセスに失敗する場合は、HDFSクラスターの各ノードのホスト名とIPアドレスのマッピングを**/etc/hosts**に設定する必要があります。

### Kerberos認証

HDFSクラスターまたはHMSがKerberos認証を有効にしている場合、StarRocksクラスターで以下の設定を行う必要があります：

- 各FEおよびBEで`kinit -kt keytab_path principal`コマンドを実行し、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するユーザーはHMSおよびHDFSにアクセスする権限を持っている必要があります。このコマンドを使用してKDCにアクセスすることには有効期限があるため、cronを使用して定期的にこのコマンドを実行する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルおよび各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。ここで、`/etc/krb5.conf`は**krb5.conf**ファイルのパスであり、実際のファイルパスに応じて変更できます。

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

### パラメータ説明

#### catalog_name

Hudi Catalogの名前です。命名規則は以下の通りです：

- 英字（a-zまたはA-Z）、数字（0-9）、アンダースコア（_）のみを使用し、英字で始める必要があります。
- 合計長さは1023文字を超えることはできません。
- Catalog名は大文字と小文字を区別します。

#### コメント

Hudi Catalogの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。`hudi`に設定します。

#### MetastoreParams

StarRocksがHudiクラスターのメタデータサービスにアクセスするためのパラメータ設定です。

##### HMS

HMSをHudiクラスターのメタデータサービスとして選択する場合、以下のように`MetastoreParams`を設定してください：

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **説明**
>
> Hudiデータをクエリする前に、すべてのHMSノードのホスト名とIPアドレスのマッピングを**/etc/hosts**に追加する必要があります。そうしないと、クエリを開始したときにStarRocksがHMSにアクセスできない可能性があります。

`MetastoreParams`には以下のパラメータが含まれます。

| パラメータ                | 必須 | 説明                                                         |
| ------------------- | ---- | ------------------------------------------------------------ |
| hive.metastore.type | はい  | Hudiクラスターで使用されるメタデータサービスのタイプです。`hive`に設定します。 |
| hive.metastore.uris | はい  | HMSのURIです。形式：`thrift://<HMS IPアドレス>:<HMSポート番号>`。<br />HMSが高可用性モードを有効にしている場合、ここに複数のHMSアドレスをカンマで区切って記入できます。例：`"thrift://<HMS IPアドレス1>:<HMSポート番号1>,thrift://<HMS IPアドレス2>:<HMSポート番号2>,thrift://<HMS IPアドレス3>:<HMSポート番号3>"`。 |

##### AWS Glue

AWS GlueをHudiクラスターのメタデータサービスとして選択する場合（AWS S3をストレージシステムとして使用している場合のみサポートされます）、以下のように`MetastoreParams`を設定してください：

- Instance Profileを使用した認証と認可

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- Assumed Roleを使用した認証と認可

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
  "aws.glue.region" = "<aws_glue_region>"
  ```

`MetastoreParams`には以下のパラメータが含まれます。

| パラメータ                      | 必須 | 説明                                                         |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| hive.metastore.type           | はい  | Hudiクラスターで使用されるメタデータサービスのタイプです。`glue`に設定します。 |
| aws.glue.use_instance_profile | はい  | Instance ProfileとAssumed Roleの2つの認証方式を使用するかどうかを指定します。値の範囲は`true`または`false`です。デフォルト値は`false`です。 |
| aws.glue.iam_role_arn         | いいえ | AWS Glue Data Catalogにアクセスする権限を持つIAM RoleのARNです。Assumed Role認証方式を使用してAWS Glueにアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.region               | はい  | AWS Glue Data Catalogが存在するリージョンです。例：`us-west-1`。 |
| aws.glue.access_key           | いいえ | IAM UserのAccess Keyです。IAM User認証方式を使用してAWS Glueにアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | いいえ | IAM UserのSecret Keyです。IAM User認証方式を使用してAWS Glueにアクセスする場合、このパラメータを指定する必要があります。 |

AWS Glueへのアクセスに使用する認証方式の選択方法、およびAWS IAMコンソールでアクセス制御ポリシーを設定する方法については、[AWS Glueへの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#aws-glueへの認証パラメータ)を参照してください。

#### StorageCredentialParams

StarRocksがHudiクラスターのファイルストレージにアクセスするためのパラメータ設定です。

HDFSをストレージシステムとして使用する場合は、`StorageCredentialParams`の設定は必要ありません。

AWS S3、S3プロトコル互換のオブジェクトストレージ、Microsoft Azure Storage、またはGCSを使用する場合は、`StorageCredentialParams`を設定する必要があります。

##### AWS S3

AWS S3をHudiクラスターのファイルストレージとして選択する場合、以下のように`StorageCredentialParams`を設定してください：

- Instance Profileを使用した認証と認可

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- Assumed Roleを使用した認証と認可

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

`StorageCredentialParams`には以下のパラメータが含まれます。

| パラメータ                    | 必須 | 説明                                                         |
| --------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい  | Instance ProfileとAssumed Roleの2つの認証方式を使用するかどうかを指定します。値の範囲は`true`または`false`です。デフォルト値は`false`です。 |
| aws.s3.iam_role_arn         | いいえ | AWS S3バケットにアクセスする権限を持つIAM RoleのARNです。Assumed Role認証方式を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい  | AWS S3バケットが存在するリージョンです。例：`us-west-1`。    |

| aws.s3.access_key           | 否       | IAM User の Access Key。IAM User 認証方式で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | 否       | IAM User の Secret Key。IAM User 認証方式で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3 へのアクセスに使用する認証方式の選択方法と、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS S3 認証パラメータ](../../integrations/authenticate_to_aws_resources.md#访问-aws-s3-の认证参数)を参照してください。

##### 阿里云 OSS

阿里云 OSS を Hudi クラスタのファイルストレージとして選択する場合は、`StorageCredentialParams` に以下の認証パラメータを設定する必要があります：

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

##### S3 互換オブジェクトストレージ

Hudi Catalog はバージョン 2.5 から S3 互換オブジェクトストレージをサポートしています。

S3 互換オブジェクトストレージ（例：MinIO）を Hudi クラスタのファイルストレージとして選択する場合、`StorageCredentialParams` を以下のように設定してください：

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
| aws.s3.enable_path_style_access  | はい | パススタイルアクセス（Path-Style Access）を有効にするかどうか。<br />選択肢：`true` または `false`。デフォルト値：`false`。MinIO では `true` に設定する必要があります。<br />パススタイル URL の形式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部（オレゴン）リージョンに `DOC-EXAMPLE-BUCKET1` というバケットを作成し、その中の `alice.jpg` オブジェクトにアクセスしたい場合、次のパススタイル URL を使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | はい | S3 互換オブジェクトストレージへのアクセスに使用するエンドポイント。 |
| aws.s3.access_key                | はい | IAM User の Access Key。 |
| aws.s3.secret_key                | はい | IAM User の Secret Key。 |

##### Microsoft Azure ストレージ

Hudi Catalog はバージョン 3.0 から Microsoft Azure Storage をサポートしています。

###### Azure Blob Storage

Azure Blob Storage を Hudi クラスタのファイルストレージとして選択する場合、`StorageCredentialParams` を以下のように設定してください：

- Shared Key による認証と承認

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                   | **必須** | **説明**                         |
  | -------------------------- | ---- | -------------------------------- |
  | azure.blob.storage_account | はい | Blob Storage アカウントのユーザー名。 |
  | azure.blob.shared_key      | はい | Blob Storage アカウントの Shared Key。 |

- SAS Token による認証と承認

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

Azure Data Lake Storage Gen1 を Hudi クラスタのファイルストレージとして選択する場合、`StorageCredentialParams` を以下のように設定してください：

- Managed Service Identity による認証と承認

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                                 | **必須** | **説明**                                                     |
  | ---------------------------------------- | ---- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい | Managed Service Identity 認証方式を有効にするかどうか。`true` に設定します。 |

- Service Principal による認証と承認

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                 | **必須** | **説明**                                              |
  | ----------------------------- | ---- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい | Service Principal のクライアント（アプリケーション）ID。    |
  | azure.adls1.oauth2_credential | はい | 新規作成されたクライアント（アプリケーション）シークレット。 |
  | azure.adls1.oauth2_endpoint   | はい | Service Principal またはアプリケーションの OAuth 2.0 トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2

Azure Data Lake Storage Gen2 を Hudi クラスタのファイルストレージとして選択する場合、`StorageCredentialParams` を以下のように設定してください：

- Managed Identity による認証と承認

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

- Shared Key による認証と承認

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                    | **必須** | **説明**                                   |
  | --------------------------- | ---- | ------------------------------------------ |
  | azure.adls2.storage_account | はい | Data Lake Storage Gen2 アカウントのユーザー名。 |
  | azure.adls2.shared_key      | はい | Data Lake Storage Gen2 アカウントの Shared Key。 |

- Service Principal による認証と承認

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                           | **必須** | **説明**                                                     |
  | ---------------------------------- | ---- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい | Service Principal のクライアント（アプリケーション）ID。    |
  | azure.adls2.oauth2_client_secret   | はい | 新規作成されたクライアント（アプリケーション）シークレット。 |
  | azure.adls2.oauth2_client_endpoint | はい | Service Principal またはアプリケーションの OAuth 2.0 トークンエンドポイント（v1）。 |

##### Google GCS

Hudi Catalog はバージョン 3.0 から Google GCS をサポートしています。

Google GCS を Hudi クラスタのファイルストレージとして選択する場合、`StorageCredentialParams` を以下のように設定してください：

- VM による認証と承認

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                                   | **デフォルト値** | **取りうる値** | **説明**                                                 |
  | ------------------------------------------ | ------------ | ------------ | -------------------------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | false        | true         | Compute Engine に紐づけられた Service Account を直接使用するかどうか。 |

- Service Account による認証と承認

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                               | **デフォルト値** | **取りうる値**                                             | **説明**                                                     |
  | -------------------------------------- | ------------ | -------------------------------------------------------- | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""           | "user@hello.iam.gserviceaccount.com"                     | Service Account を作成した際に生成された JSON ファイル内のメールアドレス。 |
  | gcp.gcs.service_account_private_key_id | ""           | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | Service Account を作成した際に生成された JSON ファイル内のプライベートキー ID。 |
  | gcp.gcs.service_account_private_key    | ""           | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n" | Service Account を作成した際に生成された JSON ファイル内のプライベートキー。 |

- Impersonation による認証と承認

  - VM インスタンスで Service Account を模倣

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | **パラメータ**                                   | **デフォルト値** | **取りうる値** | **説明**                                                     |
    | ------------------------------------------ | ------------ | ------------ | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false        | true         | Compute Engine に紐づけられた Service Account を直接使用するかどうか。 |
    | gcp.gcs.impersonation_service_account      | ""           | "hello"      | 模倣する対象の Service Account。 |

  - 一つの Service Account（「Meta Service Account」と仮称）で別の Service Account（「Data Service Account」と仮称）を模倣

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | **パラメータ**                               | **デフォルト値** | **例**                                                 | **説明**                                                     |
    | -------------------------------------- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""         | 「[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)」 | Meta Service Account を作成する際に生成される JSON ファイル内の Email。     |
    | gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | Meta Service Account を作成する際に生成される JSON ファイル内の Private Key ID。 |
    | gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | Meta Service Account を作成する際に生成される JSON ファイル内の Private Key。 |
    | gcp.gcs.impersonation_service_account  | ""         | 「hello」                                                      | 模擬する必要がある対象の Data Service Account。 |

#### MetadataUpdateParams (メタデータ更新パラメーター)

キャッシュされたメタデータの更新戦略を指定する一連のパラメータ。StarRocks はこの戦略に基づいてキャッシュされた Hudi メタデータを更新します。このパラメータ群はオプションです。

StarRocks はデフォルトで[自動非同期更新戦略](#附录理解元数据自动异步更新策略)を採用し、すぐに使用できます。したがって、通常は `MetadataUpdateParams` を無視して、戦略パラメータを調整する必要はありません。

Hudi データの更新頻度が高い場合は、これらのパラメータを調整して、自動非同期更新戦略のパフォーマンスを最適化できます。

| パラメータ                                   | 必須 | 説明                                                         |
| -------------------------------------- | -------- | ------------------------------------------------------------ |
| enable_metastore_cache            | いいえ       | StarRocks が Hudi テーブルのメタデータをキャッシュするかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`true`。`true` はキャッシュを有効にし、`false` はキャッシュを無効にします。|
| enable_remote_file_cache               | いいえ       | StarRocks が Hudi テーブルまたはパーティションのデータファイルのメタデータをキャッシュするかどうかを指定します。値の範囲：`true` または `false`。デフォルト値：`true`。`true` はキャッシュを有効にし、`false` はキャッシュを無効にします。|
| metastore_cache_refresh_interval_sec   | いいえ       | StarRocks がキャッシュされた Hudi テーブルまたはパーティションのメタデータを非同期で更新する間隔。単位：秒。デフォルト値：`7200`、つまり 2 時間。 |
| remote_file_cache_refresh_interval_sec | いいえ       | StarRocks がキャッシュされた Hudi テーブルまたはパーティションのデータファイルのメタデータを非同期で更新する間隔。単位：秒。デフォルト値：`60`。 |
| metastore_cache_ttl_sec                | いいえ       | StarRocks がキャッシュされた Hudi テーブルまたはパーティションのメタデータを自動的に削除するまでの時間。単位：秒。デフォルト値：`86400`、つまり 24 時間。 |
| remote_file_cache_ttl_sec              | いいえ       | StarRocks がキャッシュされた Hudi テーブルまたはパーティションのデータファイルのメタデータを自動的に削除するまでの時間。単位：秒。デフォルト値：`129600`、つまり 36 時間。 |

### 例

以下の例では、`hudi_catalog_hms` または `hudi_catalog_glue` という名前の Hudi カタログを作成し、Hudi クラスタ内のデータをクエリします。

#### HDFS

HDFS をストレージとして使用する場合、以下のように Hudi カタログを作成できます：

```SQL
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

#### AWS S3

##### Instance Profile を使用して認証と認可を行う場合

- Hudi クラスタが HMS をメタデータサービスとして使用している場合、以下のように Hudi カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Hudi クラスタが AWS Glue をメタデータサービスとして使用している場合、以下のように Hudi カタログを作成できます：

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

##### Assumed Role を使用して認証と認可を行う場合

- Hudi クラスタが HMS をメタデータサービスとして使用している場合、以下のように Hudi カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Hudi クラスタが AWS Glue をメタデータサービスとして使用している場合、以下のように Hudi カタログを作成できます：

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

##### IAM User を使用して認証と認可を行う場合

- Hudi クラスタが HMS をメタデータサービスとして使用している場合、以下のように Hudi カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR Hudi クラスタが AWS Glue をメタデータサービスとして使用している場合、以下のように Hudi カタログを作成できます：

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

#### S3 プロトコルに対応したオブジェクトストレージ

MinIO を例に、以下のように Hudi カタログを作成できます：

```SQL
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
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

- Shared Key を使用して認証と認可を行う場合、以下のように Hudi カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SAS Token を使用して認証と認可を行う場合、以下のように Hudi カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- Managed Service Identity を使用して認証と認可を行う場合、以下のように Hudi カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- Service Principal を使用して認証と認可を行う場合、以下のように Hudi カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- Managed Identity を使用して認証と認可を行う場合、以下のように Hudi カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- Shared Key を使用して認証と認可を行う場合、以下のように Hudi カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- Service Principal を使用して認証と認可を行う場合、以下のように Hudi カタログを作成できます：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>" 
  );
  ```

#### Google GCS

- VMをベースに認証と権限付与を行う場合は、以下のようにHudiカタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- Service Accountをベースに認証と権限付与を行う場合は、以下のようにHudiカタログを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- Impersonationをベースに認証と権限付与を行う場合

  - VMインスタンスでService Accountを模倣する場合は、以下のようにHudiカタログを作成できます:

    ```SQL
    CREATE EXTERNAL CATALOG hudi_catalog_hms
    PROPERTIES
    (
        "type" = "hudi",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - あるService Accountで別のService Accountを模倣する場合は、以下のようにHudiカタログを作成できます:

    ```SQL
    CREATE EXTERNAL CATALOG hudi_catalog_hms
    PROPERTIES
    (
        "type" = "hudi",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Hudiカタログを確認する

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスター内のすべてのカタログを確認できます:

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、特定のExternal Catalogの作成ステートメントを確認できます。例えば、以下のコマンドでHudiカタログ`hudi_catalog_glue`の作成ステートメントを確認できます:

```SQL
SHOW CREATE CATALOG hudi_catalog_glue;
```

## Hudiカタログとデータベースを切り替える

以下の方法で目的のHudiカタログとデータベースに切り替えることができます:

- まず[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して現在のセッションに有効なHudiカタログを指定し、次に[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してデータベースを指定します:

  ```SQL
  -- 現在のセッションに有効なカタログを切り替える：
  SET CATALOG <catalog_name>
  -- 現在のセッションに有効なデータベースを指定する：
  USE <db_name>
  ```

- [USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、セッションを直接目的のHudiカタログの特定のデータベースに切り替えます:

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Hudiカタログを削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、特定のExternal Catalogを削除できます。

例えば、以下のコマンドでHudiカタログ`hudi_catalog_glue`を削除できます:

```SQL
DROP CATALOG hudi_catalog_glue;
```

## Hudiテーブルの構造を確認する

以下の方法でHudiテーブルの構造を確認できます:

- テーブルの構造を確認する

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEコマンドからテーブルの構造とファイルの保存場所を確認する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Hudiテーブルのデータをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、特定のカタログに属するHudiクラスター内のデータベースを確認します:

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [目的のHudiカタログとデータベースに切り替える](#hudiカタログとデータベースを切り替える)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、目的のデータベース内の目的のテーブルをクエリします:

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hudiデータをインポートする

OLAPテーブルがあり、その名前が`olap_tbl`とすると、以下のようにしてそのテーブルのデータを変換し、StarRocksにインポートできます:

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hudi_table
```

## メタデータキャッシュを手動または自動で更新する

### 手動で更新する

デフォルトでは、StarRocksはHudiのメタデータをキャッシュし、非同期モードでキャッシュされたメタデータを自動更新してクエリのパフォーマンスを向上させます。さらに、Hudiテーブルの構造変更やその他の更新を行った後、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)を使用してそのテーブルのメタデータを手動で更新し、StarRocksが迅速に適切なクエリプランを生成することを保証できます:

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自動インクリメンタル更新

自動非同期更新戦略とは異なり、自動インクリメンタル更新戦略では、FEは定期的にHMSから各種イベントを読み取り、Hudiテーブルのメタデータの変更を感知できます。これには、列の追加や削除、パーティションの追加や削除、パーティションデータの更新などが含まれます。Hudiテーブルのメタデータを手動で更新する必要はありません。

自動インクリメンタル更新戦略を有効にする手順は以下の通りです：

#### ステップ1: HMSでイベントリスナーを設定する

HMS 2.xおよび3.xバージョンは、イベントリスナーの設定をサポートしています。ここでは、HMS 3.1.2バージョンのイベントリスナー設定を例にします。以下の設定項目を**$HiveMetastore/conf/hive-site.xml**ファイルに追加し、HMSを再起動します:

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

設定完了後、FEのログファイルで`event id`を検索し、イベントIDを確認することでイベントリスナーが正常に設定されたかどうかをチェックできます。設定が失敗した場合、すべての`event id`は`0`になります。

#### ステップ2: StarRocksで自動インクリメンタル更新戦略を有効にする

StarRocksクラスタ内の特定のHudiカタログに自動インクリメンタル更新戦略を有効にすることも、すべてのHudiカタログに対して有効にすることもできます。

- 特定のHudiカタログに自動インクリメンタル更新戦略を有効にする場合は、そのHudiカタログを作成する際に`PROPERTIES`内の`enable_hms_events_incremental_sync`パラメータを`true`に設定する必要があります。以下のようにします:

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```
  
- すべてのHudiカタログに自動インクリメンタル更新戦略を有効にする場合は、`enable_hms_events_incremental_sync`パラメータを各FEの**$FE_HOME/conf/fe.conf**ファイルに追加し、`true`に設定した後、FEを再起動してパラメータ設定を有効にします。

ビジネスニーズに応じて、各FEの**$FE_HOME/conf/fe.conf**ファイルで以下のパラメータを調整し、FEを再起動してパラメータ設定を有効にすることもできます。

| パラメータ                         | 説明                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocksがHMSからイベントを読み取る間隔。デフォルト値は`5000`ミリ秒です。 |
| hms_events_batch_size_per_rpc     | StarRocksが一度に読み取るイベントの最大数。デフォルト値は`500`です。            |
| enable_hms_parallel_process_events | StarRocksがイベントを読み取る際に、読み取ったイベントを並行して処理するかどうかを指定します。値は`true`または`false`です。デフォルト値は`true`で、`true`の場合は並行処理が有効になり、`false`の場合は無効になります。 |
| hms_process_events_parallel_num   | StarRocksがイベントを処理する際の最大並行数。デフォルト値は`4`です。            |

## 付録：メタデータ自動非同期更新戦略を理解する

自動非同期更新戦略は、StarRocksがHudiカタログ内のメタデータを更新するためのデフォルト戦略です。

デフォルトでは（つまり、`enable_metastore_cache`パラメータと`enable_remote_file_cache`パラメータが両方とも`true`に設定されている場合）、クエリがHudiテーブルの特定のパーティションにヒットした場合、StarRocksはそのパーティションのメタデータとそのパーティションにあるデータファイルのメタデータを自動的にキャッシュします。キャッシュされたメタデータは、Lazy Update（遅延更新）戦略を採用しています。

例えば、`table2`という名前のHudiテーブルがあり、そのデータは`p1`、`p2`、`p3`、`p4`の4つのパーティションに分散しているとします。クエリが`p1`にヒットした場合、StarRocksは`p1`のメタデータと`p1`にあるデータファイルのメタデータを自動的にキャッシュします。現在のキャッシュされたメタデータの更新と削除戦略は以下のように設定されているとします：

- `p1`のキャッシュされたメタデータを非同期で更新する間隔（`metastore_cache_refresh_interval_sec`パラメータで指定）は2時間です。
- `p1`にあるデータファイルのキャッシュされたメタデータを非同期で更新する間隔（`remote_file_cache_refresh_interval_sec`パラメータで指定）は60秒です。
- `p1`のキャッシュされたメタデータを自動的に削除する間隔（`metastore_cache_ttl_sec`パラメータで指定）は24時間です。
- `p1`にあるデータファイルのキャッシュされたメタデータを自動的に削除する間隔（`remote_file_cache_ttl_sec`パラメータで指定）は36時間です。

以下の図に示されています。

![タイムライン上の更新ポリシー](../../assets/catalog_timeline_zh.png)

StarRocksは、キャッシュされたメタデータを更新および削除するために以下の戦略を採用しています:


- もし他のクエリが `p1` を再びヒットし、かつ現在の時間が最後の更新から60秒以内であれば、StarRocks は `p1` のキャッシュメタデータも `p1` のデータファイルのキャッシュメタデータも更新しません。
- もし他のクエリが `p1` を再びヒットし、かつ現在の時間が最後の更新から60秒を超える場合、StarRocks は `p1` のデータファイルのキャッシュメタデータを更新します。
- もし他のクエリが `p1` を再びヒットし、かつ現在の時間が最後の更新から2時間を超える場合、StarRocks は `p1` のキャッシュメタデータを更新します。
- 最後の更新が終了してから24時間以内に `p1` がアクセスされなかった場合、StarRocks は `p1` のキャッシュメタデータを削除します。その後、クエリが `p1` を再びヒットした場合、`p1` のメタデータは再キャッシュされます。
- 最後の更新が終了してから36時間以内に `p1` がアクセスされなかった場合、StarRocks は `p1` のデータファイルのキャッシュメタデータを削除します。その後、クエリが `p1` を再びヒットした場合、`p1` のデータファイルのメタデータは再キャッシュされます。
