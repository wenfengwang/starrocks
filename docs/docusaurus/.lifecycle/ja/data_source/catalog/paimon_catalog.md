---
displayed_sidebar: "Japanese"
---

# Paimon カタログ

StarRocksは v3.1以降で Paimon カタログをサポートします。

Paimon カタログは、データをApache Paimon から取得するための外部カタログで、インジェストなしでクエリーを実行できるようにします。

また、Paimon カタログを使用することで、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)をベースにして、Paimon からデータを直接変換およびロードすることができます。

Paimon クラスター上での SQL ワークロードの成功を確実にするためには、StarRocks クラスターが次の2つの重要なコンポーネントと統合する必要があります。

- 配布ファイルシステム (HDFS) または AWS S3、Microsoft Azure Storage、Google GCS、またはその他の S3 互換のストレージシステム (例: MinIO)
- ファイルシステムまたは Hive メタストアのようなメタストア

## 使用上の注意

Paimon カタログではデータのクエリーのみが可能です。Paimon カタログを使用して Paimon クラスターにデータをドロップ、削除、または挿入することはできません。

## 統合の準備

Paimon カタログを作成する前に、StarRocks クラスターが Paimon クラスターのストレージシステムおよびメタストアと統合できることを確認してください。

### AWS IAM

Paimon クラスターが AWS S3 をストレージとして使用する場合、適切な認証方法を選択し、関連する AWS クラウドリソースに StarRocks クラスターがアクセスできるように必要な準備を行ってください。

以下の認証方法が推奨されています。

- インスタンスプロファイル (推奨)
- 仮定されたロール
- IAM ユーザー

上記の3つの認証方法のうち、インスタンスプロファイルが最も広く使用されています。

詳細は、[AWS IAM での認証の準備](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)を参照してください。

### HDFS

ストレージとして HDFS を選択する場合、StarRocks クラスターを次のように構成してください。

- (オプション) HDFS クラスターおよび Hive メタストアにアクセスするために使用されるユーザー名を設定します。デフォルトでは、StarRocks は FE および BE プロセスのユーザー名を使用して HDFS クラスターや Hive メタストアにアクセスします。各 FE の **fe/conf/hadoop_env.sh** ファイルおよび各 BE の **be/conf/hadoop_env.sh** ファイルの先頭に `export HADOOP_USER_NAME="<user_name>"` を追加することでユーザー名を設定できます。これらのファイルにユーザー名を設定した後、各 FE および各 BE を再起動してパラメーター設定を有効にします。StarRocks クラスターごとに1つのユーザー名のみを設定できます。
- Paimon データをクエリーする際、StarRocks クラスターの FE および BE は HDFS クラスターにアクセスするために HDFS クライアントを使用します。ほとんどの場合、この目的を達成するために StarRocks クラスターを構成する必要はありません。StarRocks はデフォルトの構成を使用して HDFS クライアントを起動します。次の状況でのみ StarRocks クラスターを構成する必要があります。
  - HDFS クラスターに対して高可用性 (HA) が有効になっている場合: HDFS クラスターの **hdfs-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスと各 BE の **$BE_HOME/conf** パスに追加します。
  - HDFS クラスターに対して View File System (ViewFs) が有効になっている場合: HDFS クラスターの **core-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスと各 BE の **$BE_HOME/conf** パスに追加します。

> **注意**
>
> クエリーを送信する際に不明なホストを示すエラーが返される場合は、HDFS クラスターノードのホスト名と IP アドレスのマッピングを **/etc/hosts** パスに追加する必要があります。

### Kerberos 認証

HDFS クラスターや Hive メタストアに対して Kerberos 認証が有効になっている場合は、StarRocks クラスターを次のように構成してください。

- 各 FE および各 BE で `kinit -kt keytab_path principal` コマンドを実行して、Key Distribution Center (KDC) から Ticket Granting Ticket (TGT) を取得します。このコマンドを実行するには、HDFS クラスターや Hive メタストアにアクセスする権限が必要です。このコマンドで KDC にアクセスするには時間制限があります。そのため、このコマンドを定期的に実行するために cron を使用する必要があります。
- 各 FE の **$FE_HOME/conf/fe.conf** ファイルおよび各 BE の **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。この例では、 `/etc/krb5.conf` が **krb5.conf** ファイルの保存パスです。必要に応じてパスを変更することができます。

## Paimon カタログの作成

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

### パラメーター

#### catalog_name

Paimon カタログの名前。命名規則は以下の通りです。

- 名前には英字、数字 (0-9)、およびアンダースコア (_) を含めることができます。英字で始める必要があります。
- 名前は大文字と小文字を区別し、長さが 1023 文字を超えてはいけません。

#### comment

Paimon カタログの説明。このパラメーターはオプションです。

#### type

データソースの種類。値を `paimon` に設定します。

#### CatalogParams

StarRocks が Paimon クラスターのメタデータにアクセスする方法に関するパラメーターのセット。

`CatalogParams` で設定する必要のあるパラメーターについては、次の表に説明があります。

| パラメーター               | 必須     | 説明                                                     |
| ------------------------ | -------- | ------------------------------------------------------------ |
| paimon.catalog.type      | Yes      | Paimon クラスターで使用するメタストアのタイプ。このパラメーターには `filesystem` または `hive` を設定します。 |
| paimon.catalog.warehouse | Yes      | Paimon データの保管パス。 |
| hive.metastore.uris      | No       | Hive メタストアの URI。形式: `thrift://<metastore_IP_address>:<metastore_port>`。Hive メタストアで高可用性 (HA) が有効になっている場合、複数のメタストア URI を指定し、カンマ (`,`) で区切ります。例: `"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

> **注意**
>
> Hive メタストアを使用する場合は、Paimon データをクエリーする前に、Hive メタストアノードのホスト名と IP アドレスのマッピングを **/etc/hosts** パスに追加する必要があります。そうしないと、クエリーを開始したときに StarRocks が Hive メタストアにアクセスできなくなる可能性があります。

#### StorageCredentialParams

ストレージシステムとの統合に関するパラメーターのセット。このパラメーターセットはオプションです。

ストレージとして HDFS を使用する場合は、 `StorageCredentialParams` を構成する必要はありません。

ストレージとして AWS S3、他の S3 互換のストレージシステム、Microsoft Azure Storage、または Google GCS を使用する場合は、 `StorageCredentialParams` を構成する必要があります。

##### AWS S3

Paimon クラスターのストレージとして AWS S3 を選択する場合、次のアクションのいずれかを実行してください。

- インスタンスプロファイルベースの認証メソッドを選択する場合は、`StorageCredentialParams` を次のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 仮定されたロールベースの認証メソッドを選択する場合は、`StorageCredentialParams` を次のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM ユーザーベースの認証メソッドを選択する場合は、`StorageCredentialParams` を次のように構成してください:

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

`StorageCredentialParams` で設定する必要のあるパラメーターについては、次の表に説明があります。

| パラメーター                    | 必須     | 説明                                                     |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes      | インスタンスプロファイルベースの認証メソッドおよび仮定されたロールベースの認証メソッドを有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | No       | AWS S3 バケットに特権がある IAM ロールの ARN。AWS S3 へのアクセスに仮定されたロールベースの認証メソッドを使用する場合、このパラメーターを指定する必要があります。 |
| aws.s3.region               | Yes      | AWS S3 バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | No       | IAM ユーザーのアクセスキー。IAM ユーザーベースの認証メソッドを使用して AWS S3 にアクセスする場合、このパラメーターを指定する必要があります。 |
| aws.s3.secret_key           | No       | IAM ユーザーのシークレットキー。IAM ユーザーベースの認証メソッドを使用して AWS S3 にアクセスする場合、このパラメーターを指定する必要があります。 |

AWS S3へのアクセス認証方法の選択とAWS IAMコンソールでのアクセス制御ポリシーの設定方法については、[AWS S3へのアクセス認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

MinIOなどのS3互換ストレージシステムを選択した場合、Paimonクラスターのストレージとして成功する統合を確保するために、次のように`StorageCredentialParams`を構成してください。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

次の表は`StorageCredentialParams`で構成する必要があるパラメータを説明します。

| パラメータ                       | 必須    | 説明                                                         |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | はい    | SSL接続を有効にするかどうかを指定します。<br />有効な値: `true`および`false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access | はい    | パス形式アクセスを有効にするかどうかを指定します。<br />有効な値: `true`および`false`。デフォルト値: `false`。MinIOの場合、値を`true`に設定する必要があります。<br />パス形式のURLは次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、US West (Oregon) リージョンに`DOC-EXAMPLE-BUCKET1`というバケットを作成し、そのバケット内の`alice.jpg`オブジェクトにアクセスしたい場合、次のパス形式のURLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | はい    | AWS S3の代わりにS3互換ストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key               | はい    | IAMユーザーのアクセスキー。                                    |
| aws.s3.secret_key               | はい    | IAMユーザーのシークレットキー。                            |

##### Microsoft Azure Storage

###### Azure Blob Storage

PaimonクラスターのストレージとしてBlob Storageを選択した場合、次のいずれかの操作を行ってください。

- 共有キー認証メソッドを選択する場合は、次のように`StorageCredentialParams`を構成してください。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  次の表は`StorageCredentialParams`で構成する必要があるパラメータを説明します。

  | パラメータ                  | 必須    | 説明                                        |
  | -------------------------- | -------- | -------------------------------------------- |
  | azure.blob.storage_account | はい    | Blob Storageアカウントのユーザー名。         |
  | azure.blob.shared_key      | はい    | Blob Storageアカウントの共有キー。 |

- SASトークン認証メソッドを選択する場合は、次のように`StorageCredentialParams`を構成してください。

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  次の表は`StorageCredentialParams`で構成する必要があるパラメータを説明します。

  | パラメータ                | 必須    | 説明                                                         |
  | ------------------------ | -------- | ------------------------------------------------------------ |
  | azure.blob.account_name  | はい    | Blob Storageアカウントのユーザー名。                          |
  | azure.blob.container_name| はい    | データを格納するBlobコンテナの名前。                        |
  | azure.blob.sas_token      | はい    | Blob Storageアカウントにアクセスするために使用されるSASトークン。 |

###### Azure Data Lake Storage Gen1

PaimonクラスターのストレージとしてData Lake Storage Gen1を選択した場合、次のいずれかの操作を行ってください。

- 管理されたサービスアイデンティティ認証メソッドを選択する場合は、次のように`StorageCredentialParams`を構成してください。

```SQL
  "azure.adls1.use_managed_service_identity" = "true"
```

  次の表は`StorageCredentialParams`で構成する必要があるパラメータを説明します。

  | パラメータ                                | 必須    | 説明                                                         |
  | ---------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい    | 管理されたサービスアイデンティティ認証メソッドを有効にするかどうかを指定します。値を`true`に設定してください。 |

- Service Principal認証メソッドを選択する場合は、次のように`StorageCredentialParams`を構成してください。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  次の表は`StorageCredentialParams`で構成する必要があるパラメータを説明します。

  | パラメータ                     | 必須    | 説明                                                         |
  | ----------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい    | Service Principalのクライアント（アプリケーション）ID。            |
  | azure.adls1.oauth2_credential | はい    | 作成した新しいクライアント（アプリケーション）シークレットの値。    |
  | azure.adls1.oauth2_endpoint   | はい    | Service PrincipalまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2

PaimonクラスターのストレージとしてData Lake Storage Gen2を選択した場合、次のいずれかの操作を行ってください。

- Managed Identity認証メソッドを選択する場合は、次のように`StorageCredentialParams`を構成してください。

```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
```

  次の表は`StorageCredentialParams`で構成する必要があるパラメータを説明します。

  | パラメータ                               | 必須    | 説明                                                         |
  | --------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | はい    | 管理されたIdentity認証メソッドを有効にするかどうかを指定します。値を`true`に設定してください。 |
  | azure.adls2.oauth2_tenant_id            | はい    | アクセスしたいテナントのID。                                 |
  | azure.adls2.oauth2_client_id            | はい    | マネージドアイデンティティのクライアント（アプリケーション）ID。 |

- 共有キー認証メソッドを選択する場合は、次のように`StorageCredentialParams`を構成してください。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次の表は`StorageCredentialParams`で構成する必要があるパラメータを説明します。

  | パラメータ                   | 必須    | 説明                                                            |
  | --------------------------- | -------- | --------------------------------------------------------------- |
  | azure.adls2.storage_account | はい    | Data Lake Storage Gen2のストレージアカウントのユーザー名。           |
  | azure.adls2.shared_key      | はい    | Data Lake Storage Gen2のストレージアカウントの共有キー。          |

- Service Principal認証メソッドを選択する場合は、次のように`StorageCredentialParams`を構成してください。

```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
```

  次の表は`StorageCredentialParams`で構成する必要があるパラメータを説明します。

  | パラメータ                     | 必須    | 説明                                                         |
  | ----------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id  | はい    | Service Principalのクライアント（アプリケーション）ID。            |
  | azure.adls2.oauth2_client_secret | はい   | 作成した新しいクライアント（アプリケーション）シークレットの値。    |
  | azure.adls2.oauth2_client_endpoint | はい | Service PrincipalまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

##### Google GCS

PaimonクラスターのストレージとしてGoogle GCSを選択した場合、次のいずれかの操作を行ってください。

- VMベースの認証メソッドを選択する場合は、次のように`StorageCredentialParams`を構成してください。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次の表は`StorageCredentialParams`で構成する必要があるパラメータを説明します。

  | パラメータ                                  | デフォルト値 | 値の例       | 説明                                                         |
| -------------------------------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""            | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントの作成時に生成されたJSONファイル内のメールアドレスです。 |
| gcp.gcs.service_account_private_key_id | ""            | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウントの作成時に生成されたJSONファイル内のプライベートキーIDです。 |
| gcp.gcs.service_account_private_key    | ""            | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウントの作成時に生成されたJSONファイル内のプライベートキーです。 |

- 擬似化ベースの認証方法を選択するには、`StorageCredentialParams`を以下のように設定します:

  - VMインスタンスによるサービスアカウントの擬似化:

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次の表は、`StorageCredentialParams`で構成する必要のあるパラメータを説明しています。

    | Parameter                                  | Default value | Value example | Description                                                  |
    | ------------------------------------------ | ------------- | ------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | FALSE         | TRUE          | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""            | "hello"       | 擬似化したいサービスアカウントです。                      |

  - サービスアカウント（一時的にmetaサービスアカウントとして呼ばれます）を他のサービスアカウント（一時的にdataサービスアカウントとして呼ばれます）に擬似化する:

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表は、`StorageCredentialParams`で構成する必要のあるパラメータを説明しています。

    | Parameter                              | Default value | Value example                                                | Description                                                  |
    | -------------------------------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""            | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | metaサービスアカウントの作成時に生成されたJSONファイル内のメールアドレスです。 |
    | gcp.gcs.service_account_private_key_id | ""            | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | metaサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーIDです。 |
    | gcp.gcs.service_account_private_key    | ""            | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | metaサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーです。 |
    | gcp.gcs.impersonation_service_account  | ""            | "hello"                                                      | 擬似化したいdataサービスアカウントです。                 |

### Examples

以下の例では、Paimonクラスタからデータをクエリするために、`paimon.catalog.type`を`filesystem`に設定し、`paimon_catalog_fs`というPaimonカタログを作成します。

#### AWS S3

- インスタンスプロファイルベースの認証方法を選択した場合、次のようなコマンドを実行します:

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

- 想定される役割ベースの認証方法を選択した場合、次のようなコマンドを実行します:

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

- IAMユーザーベースの認証方法を選択した場合、次のようなコマンドを実行します:

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

MinIOを例に挙げます。次のようなコマンドを実行します:

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

#### Microsoft Azure Storage

##### Azure Blob Storage

- 共有キー認証方法を選択した場合、次のようなコマンドを実行します:

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

- SASトークン認証方法を選択した場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<blob_paimon_warehouse_path>",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- Managed Service Identity認証方法を選択した場合、次のようなコマンドを実行します:

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

- サービスプリンシパル認証方法を選択した場合、次のようなコマンドを実行します:

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

- Managed Identity認証方法を選択した場合、次のようなコマンドを実行します:

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

- 共有キー認証方法を選択した場合、次のようなコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
```
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls2_paimon_warehouse_path>",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"
  );

- サービスプリンシパル認証方法を選択した場合、以下のようにコマンドを実行してください：

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

- VMベースの認証方法を選択した場合、以下のようにコマンドを実行してください：

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

- サービスアカウントベースの認証方法を選択した場合、以下のようにコマンドを実行してください：

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

- インパーソネーションベースの認証方法を選択した場合：

  - VMインスタンスがサービスアカウントを偽装する場合、以下のようにコマンドを実行してください：

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

  - サービスアカウントが別のサービスアカウントを偽装する場合、以下のようにコマンドを実行してください：

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

## Paimonカタログを表示

現在のStarRocksクラスタ内のすべてのカタログをクエリするには、[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用できます：

```SQL
SHOW CATALOGS;
```

また、指定したPaimonカタログの作成ステートメントをクエリするには、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用できます。次の例では、Paimonカタログの名前が`paimon_catalog_fs`の作成ステートメントをクエリしています：

```SQL
SHOW CREATE CATALOG paimon_catalog_fs;
```

## Paimonカタログを削除

外部カタログを削除するには、[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用できます。

次の例では、名前が`paimon_catalog_fs`のPaimonカタログを削除しています：

```SQL
DROP Catalog paimon_catalog_fs;
```

## Paimonテーブルのスキーマを表示

Paimonテーブルのスキーマを表示するには、次のいずれかの構文を使用できます：

- スキーマを表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>;
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>;
  ```

## Paimonテーブルをクエリ

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Paimonクラスタ内のデータベースを表示できます：

   ```SQL
   SHOW DATABASES FROM <catalog_name>;
   ```

2. [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションで目的のカタログに切り替えることができます：

   ```SQL
   SET CATALOG <catalog_name>;
   ```

   次に、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、現在のセッションでアクティブなデータベースを指定できます：

   ```SQL
   USE <db_name>;
   ```

   または、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、直接目的のカタログ内のアクティブなデータベースを指定できます：

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

3. 指定したデータベース内のテーブルをクエリするには、[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用できます：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10;
   ```

## Paimonからデータをロード

Paimonテーブルの名前が`olap_tbl`である場合、以下のようにデータを変換してロードできます：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM paimon_table;
```