---
displayed_sidebar: English
---

# Paimonカタログ

StarRocksは、v3.1以降のPaimonカタログをサポートしています。

Paimonカタログは、Apache Paimonからデータを取り込むことなくクエリできる外部カタログの一種です。

また、Paimonカタログを使用して[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)をベースにPaimonからデータを直接変換し、ロードすることもできます。

PaimonクラスターでSQLワークロードを成功させるためには、StarRocksクラスターが以下の2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCS、またはMinIOなどのS3互換ストレージシステム
- ファイルシステムやHiveメタストアなどのメタストア

## 使用上の注意

Paimonカタログはデータをクエリするためにのみ使用できます。Paimonクラスターにデータをドロップ、削除、または挿入するためにPaimonカタログを使用することはできません。

## 統合の準備

Paimonカタログを作成する前に、StarRocksクラスターがPaimonクラスターのストレージシステムおよびメタストアと統合できることを確認してください。

### AWS IAM

PaimonクラスターがAWS S3をストレージとして使用している場合、適切な認証方法を選択し、StarRocksクラスターが関連するAWSクラウドリソースにアクセスできるように必要な準備を行ってください。

推奨される認証方法は以下の通りです：

- インスタンスプロファイル（推奨）
- アサムドロール
- IAMユーザー

上記の認証方法の中で、インスタンスプロファイルが最も広く使用されています。

詳細については、[AWS IAMでの認証の準備](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)を参照してください。

### HDFS

ストレージとしてHDFSを選択する場合、StarRocksクラスターを以下のように構成します：

- （オプション）HDFSクラスターとHiveメタストアにアクセスするために使用するユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスターとHiveメタストアにアクセスします。`export HADOOP_USER_NAME="<user_name>"`を各FEの**fe/conf/hadoop_env.sh**ファイルと各BEの**be/conf/hadoop_env.sh**ファイルの先頭に追加することでユーザー名を設定することもできます。これらのファイルにユーザー名を設定した後、各FEと各BEを再起動してパラメータ設定を有効にします。StarRocksクラスターごとに1つのユーザー名のみを設定できます。
- Paimonデータをクエリする際、StarRocksクラスターのFEとBEはHDFSクライアントを使用してHDFSクラスターにアクセスします。ほとんどの場合、StarRocksはデフォルトの設定を使用してHDFSクライアントを起動するため、StarRocksクラスターを特別に設定する必要はありません。StarRocksクラスターを設定する必要があるのは、以下の状況のみです：
  - HDFSクラスターで高可用性（HA）が有効になっている場合：HDFSクラスターの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。
  - View File System（ViewFs）がHDFSクラスターで有効になっている場合：HDFSクラスターの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。

> **注記**
>
> クエリを送信したときに未知のホストを示すエラーが返される場合は、HDFSクラスターノードのホスト名とIPアドレスのマッピングを**/etc/hosts**に追加する必要があります。

### Kerberos認証

HDFSクラスターやHiveメタストアでKerberos認証が有効になっている場合、StarRocksクラスターを以下のように構成します：

- 各FEおよび各BEで`kinit -kt keytab_path principal`コマンドを実行し、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFSクラスターとHiveメタストアへのアクセス権が必要です。KDCへのアクセスは時間に敏感なため、このコマンドを定期的に実行するためにcronを使用する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルと各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。この例では、`/etc/krb5.conf`は**krb5.conf**ファイルの保存パスです。必要に応じてパスを変更してください。

## Paimonカタログを作成する

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

### パラメータ

#### catalog_name

Paimonカタログの名前。命名規則は以下の通りです：

- 名前には文字、数字（0-9）、アンダースコア（_）を含めることができます。文字で始まる必要があります。
- 名前は大文字と小文字が区別され、長さは1023文字を超えることはできません。

#### comment

Paimonカタログの説明。このパラメータはオプションです。

#### type

データソースのタイプ。値を`paimon`に設定します。

#### CatalogParams

StarRocksがPaimonクラスタのメタデータにアクセスする方法に関するパラメータセット。

`CatalogParams`で設定する必要があるパラメータについては、以下の表を参照してください。

| パラメータ                  | 必須 | 説明                                                  |
| ------------------------ | ---- | ---------------------------------------------------- |
| paimon.catalog.type      | はい  | Paimonクラスターで使用するメタストアのタイプ。このパラメータを`filesystem`または`hive`に設定します。 |
| paimon.catalog.warehouse | はい  | Paimonデータのウェアハウスストレージパス。 |
| hive.metastore.uris      | いいえ | HiveメタストアのURI。形式：`thrift://<metastore_IP_address>:<metastore_port>`。Hiveメタストアで高可用性（HA）が有効な場合、複数のメタストアURIを指定し、コンマ（`,`）で区切ることができます。例：`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

> **注記**
>
> Hiveメタストアを使用する場合、Paimonデータをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを`/etc/hosts`に追加する必要があります。そうしないと、クエリを開始する際にStarRocksがHiveメタストアにアクセスできない可能性があります。

#### StorageCredentialParams

StarRocksがストレージシステムと統合する方法に関するパラメータセット。このパラメータセットはオプションです。

HDFSをストレージとして使用する場合、`StorageCredentialParams`を設定する必要はありません。

AWS S3、他のS3互換ストレージシステム、Microsoft Azure Storage、またはGoogle GCSをストレージとして使用する場合は、`StorageCredentialParams`を設定する必要があります。

##### AWS S3

PaimonクラスタのストレージとしてAWS S3を選択する場合、以下のいずれかのアクションを行ってください。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams`は以下のように設定します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 想定ロールベースの認証方法を選択する場合、`StorageCredentialParams`は以下のように設定します。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーベースの認証方法を選択する場合、`StorageCredentialParams`は以下のように設定します。

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

`StorageCredentialParams`で設定する必要があるパラメータについては、以下の表を参照してください。

| パラメーター                   | 必須 | 説明                                                  |
| --------------------------- | :--: | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい      | インスタンスプロファイルベースまたは想定ロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | いいえ       | AWS S3バケットに権限を持つIAMロールのARN。想定ロールベースの認証方法でAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい      | AWS S3バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | いいえ       | IAMユーザーのアクセスキー。IAMユーザーベースの認証方法でAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAMユーザーのシークレットキー。IAMユーザーベースの認証方法でAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3へのアクセスに使用する認証方法の選択とAWS IAMコンソールでのアクセス制御ポリシーの設定については、[AWS S3へのアクセスに使用する認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

MinIOなどのS3互換ストレージシステムをPaimonクラスタのストレージとして選択する場合は、成功した統合を確保するために`StorageCredentialParams`を以下のように設定します。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

`StorageCredentialParams`で設定する必要があるパラメータについては、以下の表を参照してください。

| パラメーター                       | 必須 | 説明                                                  |
| ------------------------------- | :--: | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | はい      | SSL接続を有効にするかどうかを指定します。<br />有効な値: `true`、`false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access | はい      | パススタイルアクセスを有効にするかどうかを指定します。<br />有効な値: `true`、`false`。デフォルト値: `false`。MinIOでは、値を`true`に設定する必要があります。<br />パススタイルURLは次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、US West (Oregon)リージョンに`DOC-EXAMPLE-BUCKET1`という名前のバケットを作成し、そのバケット内の`alice.jpg`オブジェクトにアクセスする場合、次のパススタイルURLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | はい      | AWS S3ではなく、S3互換ストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key               | はい      | IAMユーザーのアクセスキー。                             |
| aws.s3.secret_key               | はい      | IAMユーザーのシークレットキー。                             |

##### Microsoft Azureストレージ

###### Azure Blob Storage

PaimonクラスタのストレージとしてBlob Storageを選択する場合、以下のいずれかのアクションを行ってください。

- 共有キー認証方法を選択する場合、`StorageCredentialParams`は以下のように設定します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams`で設定する必要があるパラメータについては、以下の表を参照してください。

  | パラメーター                  | 必須 | 説明                                  |
  | -------------------------- | :--: | -------------------------------------------- |
  | azure.blob.storage_account | はい      | Blob Storageアカウントのユーザー名。   |
  | azure.blob.shared_key      | はい      | Blob Storageアカウントの共有キー。 |

- SASトークン認証方法を選択する場合、`StorageCredentialParams`は以下のように設定します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams`で設定する必要があるパラメータについては、以下の表を参照してください。

  | パラメーター                 | 必須 | 説明                                                  |
  | ------------------------- | :--: | ------------------------------------------------------------ |
  | azure.blob.storage_account| はい      | Blob Storageアカウントのユーザー名。                   |
  | azure.blob.container      | はい      | データを格納するblobコンテナの名前。        |
  | azure.blob.sas_token      | はい      | Blob Storageアカウントへのアクセスに使用されるSASトークン。 |

###### Azure Data Lake Storage Gen1

PaimonクラスタのストレージとしてData Lake Storage Gen1を選択する場合、以下のいずれかのアクションを行ってください。

- 管理サービスID認証方法を選択する場合、`StorageCredentialParams`は以下のように設定します。

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams`で設定する必要があるパラメータについては、以下の表を参照してください。

  | パラメーター                                | 必須 | 説明                                                  |
  | ---------------------------------------- | :--: | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい      | 管理サービスID認証方法を有効にするかどうかを指定します。値は`true`に設定します。 |

- サービスプリンシパル認証方法を選択する場合、`StorageCredentialParams`は以下のように設定します。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams`で設定する必要があるパラメータについては、以下の表を参照してください。

  | パラメーター                     | 必須 | 説明                                                  |
  | ----------------------------- | :--: | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい      | サービスプリンシパルのクライアント（アプリケーション）ID。        |
  | azure.adls1.oauth2_credential | はい      | 新しく作成されたクライアント（アプリケーション）シークレットの値。    |
  | azure.adls1.oauth2_endpoint   | はい      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2

PaimonクラスタのストレージとしてData Lake Storage Gen2を選択する場合、以下のいずれかのアクションを行ってください。


- マネージド ID の認証方法を選択するには、 `StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメータの説明です。

  | パラメーター                               | 必須 | 説明                                                  |
  | --------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | はい      | マネージド ID 認証方法を有効にするかどうかを指定します。値を `true` に設定します。 |
  | azure.adls2.oauth2_tenant_id            | はい      | データにアクセスするテナントの ID。          |
  | azure.adls2.oauth2_client_id            | はい      | マネージド ID のクライアント (アプリケーション) ID。         |

- 共有キーの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメータの説明です。

  | パラメーター                   | 必須 | 説明                                                  |
  | --------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.storage_account | はい      | Data Lake Storage Gen2 ストレージ アカウントのユーザー名。 |
  | azure.adls2.shared_key      | はい      | Data Lake Storage Gen2 ストレージ アカウントの共有キー。 |

- サービス プリンシパルの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  次の表に、`StorageCredentialParams` で設定する必要があるパラメータを示します。

  | パラメーター                          | 必須 | 説明                                                  |
  | ---------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい      | サービス プリンシパルのクライアント (アプリケーション) ID。        |
  | azure.adls2.oauth2_client_secret   | はい      | 作成された新しいクライアント (アプリケーション) シークレットの値。    |
  | azure.adls2.oauth2_client_endpoint | はい      | サービス プリンシパルまたはアプリケーションの OAuth 2.0 トークン エンドポイント (v1)。 |

##### Google GCS

Paimon クラスターのストレージとして Google GCS を選択した場合は、次のいずれかの操作を行います。

- VM ベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメータの説明です。

  | パラメーター                                  | 既定値 | 値の例 | 説明                                                  |
  | ------------------------------------------ | ------------- | ------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | FALSE         | TRUE          | Compute Engine にバインドされているサービス アカウントを直接使用するかどうかを指定します。 |

- サービス アカウント ベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  以下の表は、`StorageCredentialParams` で設定する必要があるパラメータの説明です。

  | パラメーター                              | 既定値 | 値の例                                                | 説明                                                  |
  | -------------------------------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""            | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントの作成時に生成されたJSONファイル内のメールアドレス。 |
  | gcp.gcs.service_account_private_key_id | ""            | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービス アカウントの作成時に生成された JSON ファイル内の秘密キー ID。 |
  | gcp.gcs.service_account_private_key    | ""            | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウントの作成時に生成されたJSONファイル内の秘密鍵。 |

- 偽装ベースの認証方法を選択するには、`StorageCredentialParams` を次のように構成します。

  - VMインスタンスをサービスアカウントに偽装させる場合：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    以下の表は、`StorageCredentialParams` で設定する必要があるパラメータの説明です。

    | パラメーター                                  | 既定値 | 値の例 | 説明                                                  |
    | ------------------------------------------ | ------------- | ------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | FALSE         | TRUE          | Compute Engine にバインドされているサービス アカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""            | "hello"       | 偽装するサービス アカウント。            |

  - サービス アカウント (一時的にメタ サービス アカウントという名前) を別のサービス アカウント (一時的にデータ サービス アカウントという名前) に偽装させる場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    以下の表は、`StorageCredentialParams` で設定する必要があるパラメータの説明です。

    | パラメーター                              | 既定値 | 値の例                                                | 説明                                                  |
    | -------------------------------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""            | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウントの作成時に生成されたJSONファイル内のメールアドレス。 |
    | gcp.gcs.service_account_private_key_id | ""            | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタ サービス アカウントの作成時に生成された JSON ファイル内の秘密キー ID。 |
    | gcp.gcs.service_account_private_key    | ""            | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウントの作成時に生成されたJSONファイル内の秘密キー。 |
    | gcp.gcs.impersonation_service_account  | ""            | "hello"                                                      | 偽装するデータ サービス アカウント。       |

### 例

次の例では、Paimon クラスターからデータをクエリするために `paimon_catalog_fs` という名前の Paimon カタログを作成します。メタストアタイプ `paimon.catalog.type` は `filesystem` に設定されています。

#### AWS S3

- インスタンスプロファイルベースの認証方法を選択した場合は、以下のコマンドを実行します。

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

- 想定されるロールベースの認証方法を選択した場合は、以下のコマンドを実行します。

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


- IAMユーザーベースの認証方法を選択した場合は、以下のようなコマンドを実行します。

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

#### S3互換ストレージシステム

例としてMinIOを使用します。以下のようなコマンドを実行します。

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

#### Microsoft Azureストレージ

##### Azure Blob Storage

- 共有キー認証方法を選択した場合は、以下のようなコマンドを実行します。

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

- SASトークン認証方法を選択した場合は、以下のようなコマンドを実行します。

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

- マネージドサービスアイデンティティ認証方法を選択した場合は、以下のようなコマンドを実行します。

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

- サービスプリンシパル認証方法を選択した場合は、以下のようなコマンドを実行します。

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

- マネージドアイデンティティ認証方法を選択した場合は、以下のようなコマンドを実行します。

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

- 共有キー認証方法を選択した場合は、以下のようなコマンドを実行します。

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

- サービスプリンシパル認証方法を選択した場合は、以下のようなコマンドを実行します。

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

- VMベースの認証方法を選択した場合は、以下のようなコマンドを実行します。

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

- サービスアカウントベースの認証方法を選択した場合は、以下のようなコマンドを実行します。

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

- 代理ベースの認証方法を選択した場合:

  - VMインスタンスがサービスアカウントを代理する場合は、以下のようなコマンドを実行します。

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

  - サービスアカウントが別のサービスアカウントを代理する場合は、以下のようなコマンドを実行します。

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

## Paimonカタログを表示する

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスター内のすべてのカタログを照会できます。

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントを照会することもできます。次の例では、`paimon_catalog_fs`という名前のPaimonカタログの作成ステートメントをクエリします。

```SQL
SHOW CREATE CATALOG paimon_catalog_fs;
```

## Paimonカタログを削除する

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、外部カタログを削除できます。

次の例では、`paimon_catalog_fs`という名前のPaimonカタログを削除します。

```SQL
DROP CATALOG paimon_catalog_fs;
```

## Paimonテーブルのスキーマを表示する

Paimonテーブルのスキーマを表示するには、以下の構文のいずれかを使用します。

- スキーマを表示する

  ```SQL
  DESCRIBE <catalog_name>.<database_name>.<table_name>;
  ```

- CREATEステートメントからスキーマと場所を表示する

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>;
  ```

## Paimonテーブルをクエリする

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Paimonクラスタ内のデータベースを表示します。

   ```SQL
   SHOW DATABASES FROM <catalog_name>;
   ```

2. [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションで宛先カタログに切り替えます。

   ```SQL
   SET CATALOG <catalog_name>;
   ```

   次に、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、現在のセッションでアクティブなデータベースを指定します。

   ```SQL
   USE <db_name>;
   ```


   または、[USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用して、宛先カタログのアクティブなデータベースを直接指定することもできます：

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して、指定されたデータベース内の目的のテーブルをクエリします：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10;
   ```

## Paimon からデータをロードする

`olap_tbl` という名前の OLAP テーブルがある場合、以下のようにデータを変換してロードすることができます：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM paimon_table;
```

