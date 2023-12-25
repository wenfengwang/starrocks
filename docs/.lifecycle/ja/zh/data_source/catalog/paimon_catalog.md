---
displayed_sidebar: Chinese
---

# Paimonカタログ

StarRocksはバージョン3.1からPaimon Catalogに対応しています。

Paimon Catalogは、外部カタログの一種です。Paimon Catalogを使用することで、データのインポートを行わずにApache Paimonのデータを直接クエリすることができます。

さらに、Paimon Catalogを基に、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)機能を組み合わせることで、データの変換やインポートを実現することができます。

Paimon内のデータに正常にアクセスするためには、StarRocksクラスターは以下の2つの重要なコンポーネントを統合する必要があります：

- 分散ファイルシステム（HDFS）またはオブジェクトストレージ。現在サポートされているオブジェクトストレージには、AWS S3、Microsoft Azure Storage、Google GCS、その他のS3プロトコル互換オブジェクトストレージ（例：アリババクラウドOSS、MinIO）が含まれます。

- メタデータサービス。現在サポートされているメタデータサービスには、ファイルシステム（File System）、Hive Metastore（以下、HMSと略）があります。

## 使用説明

Paimon CatalogはPaimonデータのクエリのみをサポートし、Paimonに対する書き込み/削除操作はサポートしていません。

## 準備作業

Paimon Catalogを作成する前に、StarRocksクラスターがPaimonのファイルストレージおよびメタデータサービスに正常にアクセスできることを確認してください。

### AWS IAM

PaimonがAWS S3をファイルストレージとして使用している場合、StarRocksクラスターが関連するAWSクラウドリソースにアクセスできるように、適切な認証および認可スキームを選択する必要があります。

以下の認証および認可スキームを選択できます：

- Instance Profile（推奨）
- Assumed Role
- IAM User

StarRocksがAWS認証にアクセスする詳細については、[AWS認証方式の設定 - 準備作業](../../integrations/authenticate_to_aws_resources.md#準備作業)を参照してください。

### HDFS

HDFSをファイルストレージとして使用する場合、StarRocksクラスターで以下の設定を行う必要があります：

- （オプション）HDFSクラスターおよびHMSへのアクセスに使用するユーザー名を設定します。各FEの**fe/conf/hadoop_env.sh**ファイルおよび各BEの**be/conf/hadoop_env.sh**ファイルの最初に`export HADOOP_USER_NAME="<user_name>"`を追加してユーザー名を設定できます。設定後、各FEおよびBEを再起動して設定を有効にします。このユーザー名を設定しない場合、デフォルトではFEおよびBEプロセスのユーザー名が使用されます。各StarRocksクラスターは1つのユーザー名のみを設定できます。
- Paimonデータをクエリする際、StarRocksクラスターのFEおよびBEはHDFSクライアントを介してHDFSクラスターにアクセスします。通常、StarRocksはデフォルトの設定に従ってHDFSクライアントを起動し、手動での設定は必要ありません。ただし、以下のシナリオでは手動での設定が必要です：
  - HDFSクラスターが高可用性（High Availability、HA）モードを有効にしている場合、HDFSクラスターの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**ディレクトリおよび各BEの**$BE_HOME/conf**ディレクトリに配置する必要があります。
  - HDFSクラスターがViewFsを設定している場合、HDFSクラスターの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**ディレクトリおよび各BEの**$BE_HOME/conf**ディレクトリに配置する必要があります。

> **注意**
>
> クエリ時にドメイン名が認識できない（Unknown Host）ためにアクセスに失敗する場合は、HDFSクラスターの各ノードのホスト名とIPアドレスのマッピングを**/etc/hosts**に設定する必要があります。

### Kerberos認証

HDFSクラスターまたはHMSがKerberos認証を有効にしている場合、StarRocksクラスターで以下の設定を行う必要があります：

- 各FEおよびBEで`kinit -kt keytab_path principal`コマンドを実行し、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するユーザーはHMSおよびHDFSにアクセスする権限を持っている必要があります。このコマンドを使用してKDCにアクセスすることには有効期限があるため、cronを使用して定期的に実行する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルおよび各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。ここで、`/etc/krb5.conf`は**krb5.conf**ファイルのパスであり、実際のファイルのパスに応じて変更することができます。

## Paimonカタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "paimon",
    CatalogParams,
    StorageCredentialParams,
)
```

### パラメータ説明

#### catalog_name

Paimon Catalogの名前。命名要件は以下の通りです：

- 英字（a-zまたはA-Z）、数字（0-9）、アンダースコア（_）で構成され、英字で始まる必要があります。
- 全体の長さは1023文字を超えてはいけません。
- カタログ名は大文字小文字を区別します。

#### コメント

Paimon Catalogの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。`paimon`に設定します。

#### CatalogParams

StarRocksがPaimonクラスターのメタデータにアクセスするためのパラメータ設定です。

`CatalogParams`には以下のパラメータが含まれます。

| パラメータ                      | 必須   | 説明                                                         |
| ------------------------ | -------- | ------------------------------------------------------------ |
| paimon.catalog.type      | はい       | Paimonが使用するメタデータのタイプです。`filesystem`または`hive`に設定します。           |
| paimon.catalog.warehouse | はい       | PaimonデータがあるWarehouseのストレージパスです。 |
| hive.metastore.uris      | いいえ       | HMSのURIです。形式：`thrift://<HMSのIPアドレス>:<HMSのポート番号>`。`paimon.catalog.type`が`hive`の場合のみ設定します。<br />HMSが高可用性モードを有効にしている場合、複数のHMSアドレスをカンマで区切って記述できます。例：`"thrift://<HMSのIPアドレス1>:<HMSのポート番号1>,thrift://<HMSのIPアドレス2>:<HMSのポート番号2>,thrift://<HMSのIPアドレス3>:<HMSのポート番号3>"`。 |

> **説明**
>
> HMSをメタデータサービスとして使用する場合、Paimonデータをクエリする前に、すべてのHMSノードのホスト名とIPアドレスのマッピングを**/etc/hosts**に追加する必要があります。そうしないと、クエリを開始したときにStarRocksがHMSにアクセスできない可能性があります。

#### StorageCredentialParams

StarRocksがPaimonクラスターのファイルストレージにアクセスするためのパラメータ設定です。

HDFSをストレージシステムとして使用する場合は、`StorageCredentialParams`を設定する必要はありません。

AWS S3、アリババクラウドOSS、その他のS3プロトコル互換オブジェクトストレージ、Microsoft Azure Storage、またはGCSを使用する場合は、`StorageCredentialParams`を設定する必要があります。

##### AWS S3

AWS S3をPaimonクラスターのファイルストレージとして選択する場合は、以下のように`StorageCredentialParams`を設定してください：

- Instance Profileを使用して認証および認可を行う

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- Assumed Roleを使用して認証および認可を行う

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM Userを使用して認証および認可を行う

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

`StorageCredentialParams`には以下のパラメータが含まれます。

| パラメータ                        | 必須   | 説明                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい       | Instance ProfileとAssumed Roleの2つの認証方法を有効にするかどうかを指定します。値の範囲は`true`または`false`です。デフォルト値は`false`です。 |
| aws.s3.iam_role_arn         | いいえ       | AWS S3バケットにアクセスする権限を持つIAM RoleのARNです。Assumed Role認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい       | AWS S3バケットがあるリージョンです。例：`us-west-1`。                |
| aws.s3.access_key           | いいえ       | IAMユーザーのアクセスキーです。IAMユーザー認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAMユーザーのシークレットキーです。IAMユーザー認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3へのアクセスに使用する認証方法の選択方法およびAWS IAMコンソールでのアクセス制御ポリシーの設定方法については、[AWS S3への認証パラメータ](../../integrations/authenticate_to_aws_resources.md#AWS-S3への認証パラメータ)を参照してください。

##### アリババクラウドOSS

アリババクラウドOSSをPaimonクラスターのファイルストレージとして選択する場合は、`StorageCredentialParams`で以下の認証パラメータを設定する必要があります：

```SQL
"aliyun.oss.access_key" = "<user_access_key>",
"aliyun.oss.secret_key" = "<user_secret_key>",
"aliyun.oss.endpoint" = "<oss_endpoint>" 
```

| パラメータ                            | 必須 | 説明                                                         |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aliyun.oss.endpoint             | はい      | アリババクラウドOSSのエンドポイントです。例：`oss-cn-beijing.aliyuncs.com`。エンドポイントとリージョンの関係については、[アクセスドメインとデータセンター](https://help.aliyun.com/document_detail/31837.html)を参照してください。    |
| aliyun.oss.access_key           | はい      | アリババクラウドアカウントまたはRAMユーザーのAccessKey IDです。取得方法については、[AccessKeyの取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。                                     |
| aliyun.oss.secret_key           | はい      | アリババクラウドアカウントまたはRAMユーザーのAccessKey Secretです。取得方法については、[AccessKeyの取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。      |

##### S3プロトコル互換オブジェクトストレージ


S3 プロトコル互換のオブジェクトストレージ（例：MinIO）を Paimon クラスタのファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

`StorageCredentialParams` には以下のパラメータが含まれます。

| パラメータ                           | 必須       | 説明                                                  |
| ------------------------------------ | ---------- | ----------------------------------------------------- |
| aws.s3.enable_ssl                    | はい       | SSL 接続を有効にするかどうか。<br />値の範囲：`true` または `false`。デフォルト値：`true`。 |
| aws.s3.enable_path_style_access      | はい       | パススタイルアクセス（Path-Style Access）を有効にするかどうか。<br />値の範囲：`true` または `false`。デフォルト値：`false`。MinIO では `true` に設定する必要があります。<br />パススタイルの URL は次の形式です：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部（オレゴン）リージョンに `DOC-EXAMPLE-BUCKET1` というバケットを作成し、その中の `alice.jpg` オブジェクトにアクセスしたい場合は、次のパススタイルの URL を使用します：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                      | はい       | S3 プロトコル互換のオブジェクトストレージにアクセスするためのエンドポイント。 |
| aws.s3.access_key                    | はい       | IAM ユーザーのアクセスキー。 |
| aws.s3.secret_key                    | はい       | IAM ユーザーのシークレットキー。 |

##### Microsoft Azure ストレージ

###### Azure Blob Storage

Azure Blob Storage を Paimon クラスタのファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- Shared Key 認証と認可

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                  | **必須** | **説明**                               |
  | ------------------------------- | -------- | -------------------------------------- |
  | azure.blob.storage_account      | はい     | Blob Storage アカウントのユーザー名。  |
  | azure.blob.shared_key           | はい     | Blob Storage アカウントの Shared Key。 |

- SAS Token 認証と認可

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                  | **必須** | **説明**                                     |
  | ------------------------------- | -------- | -------------------------------------------- |
  | azure.blob.storage_account      | はい     | Blob Storage アカウントのユーザー名。        |
  | azure.blob.container            | はい     | Blob コンテナの名前。                        |
  | azure.blob.sas_token            | はい     | Blob Storage アカウントへのアクセス用 SAS Token。 |

###### Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 を Paimon クラスタのファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- Managed Service Identity 認証と認可

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                            | **必須** | **説明**                                           |
  | ----------------------------------------- | -------- | -------------------------------------------------- |
  | azure.adls1.use_managed_service_identity  | はい     | Managed Service Identity 認証方式を有効にするかどうか。`true` に設定します。 |

- Service Principal 認証と認可

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                       | **必須** | **説明**                                             |
  | ------------------------------------ | -------- | ---------------------------------------------------- |
  | azure.adls1.oauth2_client_id         | はい     | Service Principal のクライアント（アプリケーション）ID。 |
  | azure.adls1.oauth2_credential        | はい     | 新規作成されたクライアント（アプリケーション）シークレット。 |
  | azure.adls1.oauth2_endpoint          | はい     | Service Principal またはアプリケーションの OAuth 2.0 トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2

Azure Data Lake Storage Gen2 を Paimon クラスタのファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- Managed Identity 認証と認可

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                           | **必須** | **説明**                                      |
  | ---------------------------------------- | -------- | --------------------------------------------- |
  | azure.adls2.oauth2_use_managed_identity  | はい     | Managed Identity 認証方式を有効にするかどうか。`true` に設定します。 |
  | azure.adls2.oauth2_tenant_id             | はい     | テナントの ID。                               |
  | azure.adls2.oauth2_client_id             | はい     | Managed Identity のクライアント（アプリケーション）ID。 |

- Shared Key 認証と認可

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                       | **必須** | **説明**                                   |
  | ------------------------------------ | -------- | ------------------------------------------ |
  | azure.adls2.storage_account          | はい     | Data Lake Storage Gen2 アカウントのユーザー名。 |
  | azure.adls2.shared_key               | はい     | Data Lake Storage Gen2 アカウントの Shared Key。 |

- Service Principal 認証と認可

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                         | **必須** | **説明**                                             |
  | -------------------------------------- | -------- | ---------------------------------------------------- |
  | azure.adls2.oauth2_client_id           | はい     | Service Principal のクライアント（アプリケーション）ID。 |
  | azure.adls2.oauth2_client_secret       | はい     | 新規作成されたクライアント（アプリケーション）シークレット。 |
  | azure.adls2.oauth2_client_endpoint     | はい     | Service Principal またはアプリケーションの OAuth 2.0 トークンエンドポイント（v1）。 |

##### Google GCS

Google GCS を Paimon クラスタのファイルストレージとして選択する場合は、以下のように `StorageCredentialParams` を設定してください：

- VM 認証と認可

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                                 | **デフォルト値** | **取りうる値** | **説明**                                                 |
  | ---------------------------------------------- | ---------------- | -------------- | -------------------------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account     | false            | true           | Compute Engine に紐づけられた Service Account を直接使用するかどうか。 |

- Service Account 認証と認可

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  `StorageCredentialParams` には以下のパラメータが含まれます。

  | **パラメータ**                             | **デフォルト値** | **取りうる値**                                           | **説明**                                                     |
  | ------------------------------------------ | ---------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email              | ""               | "user@hello.iam.gserviceaccount.com"                     | Service Account を作成した際に生成された JSON ファイル内の Email。 |
  | gcp.gcs.service_account_private_key_id     | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | Service Account を作成した際に生成された JSON ファイル内の Private Key ID。 |
  | gcp.gcs.service_account_private_key        | ""               | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n" | Service Account を作成した際に生成された JSON ファイル内の Private Key。 |

- Impersonation 認証と認可

  - VM インスタンスで Service Account を模倣

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | **パラメータ**                                 | **デフォルト値** | **取りうる値** | **説明**                                                     |
    | ---------------------------------------------- | ---------------- | -------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account     | false            | true           | Compute Engine に紐づけられた Service Account を直接使用するかどうか。 |
    | gcp.gcs.impersonation_service_account          | ""               | "hello"        | 模倣する対象の Service Account。                             |

  - ある Service Account（仮に「Meta Service Account」とします）で別の Service Account（仮に「Data Service Account」とします）を模倣

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    `StorageCredentialParams` には以下のパラメータが含まれます。

    | **パラメータ**                             | **デフォルト値** | **取りうる値**                                           | **説明**                                                     |
    | ------------------------------------------ | ---------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email              | ""               | "user@hello.iam.gserviceaccount.com"                     | Meta Service Account を作成した際に生成された JSON ファイル内の Email。 |
    | gcp.gcs.service_account_private_key_id     | ""               | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | Meta Service Account を作成した際に生成された JSON ファイル内の Private Key ID。 |
    | gcp.gcs.service_account_private_key        | ""               | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n" | Meta Service Account を作成した際に生成された JSON ファイル内の Private Key。 |

    | gcp.gcs.impersonation_service_account  | ""         | "hello"                                                      | 必要な模擬対象のData Service Account。 |

### 例

以下の例では、メタデータタイプ `paimon.catalog.type` が `filesystem` である `paimon_catalog_fs` という名前のPaimon Catalogを作成し、Paimonクラスタ内のデータを照会するために使用します。

#### AWS S3

- Instance Profileを使用して認証と認可を行う場合

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<s3_paimon_warehouse_path>",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Assumed Roleを使用して認証と認可を行う場合

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<s3_paimon_warehouse_path>",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- IAM Userを使用して認証と認可を行う場合

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<s3_paimon_warehouse_path>",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

#### 阿里云 OSS

```SQL
CREATE EXTERNAL CATALOG paimon_catalog_fs
PROPERTIES
(
    "type" = "paimon",
    "paimon.catalog.type" = "filesystem",
    "paimon.catalog.warehouse" = "<oss_paimon_warehouse_path>",
    "aliyun.oss.access_key" = "<user_access_key>",
    "aliyun.oss.secret_key" = "<user_secret_key>",
    "aliyun.oss.endpoint" = "<oss_endpoint>"
);
```

#### S3プロトコル互換のオブジェクトストレージ

MinIOを例に:

```SQL
CREATE EXTERNAL CATALOG paimon_catalog_fs
PROPERTIES
(
    "type" = "paimon",
    "paimon.catalog.type" = "filesystem",
    "paimon.catalog.warehouse" = "<paimon_warehouse_path>",
    "aws.s3.enable_ssl" = "true",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```

#### Microsoft Azure ストレージ

##### Azure Blob Storage

- Shared Keyを使用して認証と認可を行う場合、以下のようにPaimon Catalogを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<blob_paimon_warehouse_path>",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SAS Tokenを使用して認証と認可を行う場合、以下のようにPaimon Catalogを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<blob_paimon_warehouse_path>",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- Managed Service Identityを使用して認証と認可を行う場合、以下のようにPaimon Catalogを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls1_paimon_warehouse_path>",
      "azure.adls1.use_managed_service_identity" = "true"
  );
  ```

- Service Principalを使用して認証と認可を行う場合、以下のようにPaimon Catalogを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls1_paimon_warehouse_path>",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- Managed Identityを使用して認証と認可を行う場合、以下のようにPaimon Catalogを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls2_paimon_warehouse_path>",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- Shared Keyを使用して認証と認可を行う場合、以下のようにPaimon Catalogを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls2_paimon_warehouse_path>",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"
  );
  ```

- Service Principalを使用して認証と認可を行う場合、以下のようにPaimon Catalogを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls2_paimon_warehouse_path>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMを使用して認証と認可を行う場合、以下のようにPaimon Catalogを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
      "gcp.gcs.use_compute_engine_service_account" = "true"
  );
  ```

- Service Accountを使用して認証と認可を行う場合、以下のようにPaimon Catalogを作成できます:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  );
  ```

- Impersonationを使用して認証と認可を行う場合

  - VMインスタンスでService Accountを模擬する場合、以下のようにPaimon Catalogを作成できます:

    ```SQL
    CREATE EXTERNAL CATALOG paimon_catalog_fs
    PROPERTIES
    (
        "type" = "paimon",
        "paimon.catalog.type" = "filesystem",
        "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    );
    ```

  - あるService Accountで別のService Accountを模擬する場合、以下のようにPaimon Catalogを作成できます:

    ```SQL
    CREATE EXTERNAL CATALOG paimon_catalog_fs
    PROPERTIES
    (
        "type" = "paimon",
        "paimon.catalog.type" = "filesystem",
        "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    );
    ```

## パイモンカタログを表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) を使用して、現在のStarRocksクラスター内のすべてのカタログを照会できます:

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を使用して、特定のExternal Catalogの作成ステートメントを照会できます。例えば、以下のコマンドでPaimon Catalog `paimon_catalog_fs` の作成ステートメントを照会できます:

```SQL
SHOW CREATE CATALOG paimon_catalog_fs;
```

## パイモンカタログを削除

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) を使用して、特定のExternal Catalogを削除できます。

例えば、以下のコマンドでPaimon Catalog `paimon_catalog_fs` を削除できます:

```SQL
DROP CATALOG paimon_catalog_fs;
```

## パイモンテーブルの構造を表示

以下の方法でPaimonテーブルの構造を表示できます:

- テーブルの構造を表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>;
  ```

- CREATEコマンドからテーブルの構造とファイルの保存場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>;
  ```

## パイモンテーブルのデータを照会

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を使用して、指定されたCatalogに属するPaimon Catalog内のデータベースを表示します:

   ```SQL
   SHOW DATABASES FROM <catalog_name>;
   ```

2. [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) を使用して、現在のセッションで有効なCatalogに切り替えます:

   ```SQL
   SET CATALOG <catalog_name>;
   ```

   その後、[USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、現在のセッションで有効なデータベースを指定します:

   ```SQL
   USE <db_name>;
   ```


   または、[USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、セッションを目的のカタログの指定されたデータベースに直接切り替えることもできます:

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して、目的のデータベース内の目的のテーブルを照会します:

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10;
   ```

## Paimon データのインポート

OLAP テーブルがあると仮定し、そのテーブル名を `olap_tbl` とします。このテーブルのデータを変換し、StarRocks にインポートするには、次のようにします:

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM paimon_table;
```

