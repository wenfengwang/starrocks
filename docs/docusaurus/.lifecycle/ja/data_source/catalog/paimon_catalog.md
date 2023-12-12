---
displayed_sidebar: "Japanese"
---

# Paimon カタログ

StarRocks は v3.1 以降から Paimon カタログをサポートしています。

Paimon カタログは、Apache Paimon からデータをクエリすることを可能にする外部カタログの一種であり、データを取り込まないで使用することができます。

また、Paimon カタログに基づいて Paimon から直接データを変換およびロードすることができます。[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用します。

Paimon クラスター上の SQL ワークロードの成功を確実にするためには、StarRocks クラスターが次の 2 つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム (HDFS) または AWS S3、Microsoft Azure Storage、Google GCS などのオブジェクトストレージ
- ファイルシステムまたは Hive メタストアなどのメタストア

## 使用上の注意

Paimon カタログを使用してデータをクエリすることができますが、Paimon クラスターにデータを削除、削除、または挿入することはできません。

## 統合の準備

Paimon カタログを作成する前に、StarRocks クラスターが Paimon クラスターのストレージシステムとメタストアと統合できることを確認してください。

### AWS IAM

Paimon クラスターがストレージとして AWS S3 を使用している場合、適切な認証方法を選択し、関連する AWS クラウドリソースに StarRocks クラスターがアクセスできるようにするための準備を行ってください。

次の認証方法が推奨されています。

- インスタンスプロファイル (推奨)
- 仮定されたロール
- IAM ユーザー

上記の 3 つの認証方法のうち、インスタンスプロファイルが最も広く使用されています。

詳細については、[AWS IAM での認証の準備](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)を参照してください。

### HDFS

ストレージとして HDFS を選択する場合は、StarRocks クラスターを次のように構成してください。

- (オプション) HDFS クラスターや Hive メタストアにアクセスするために使用されるユーザー名を設定してください。デフォルトでは、StarRocks は FE プロセスと BE プロセスのユーザー名を使用して HDFS クラスターや Hive メタストアにアクセスします。`export HADOOP_USER_NAME="<user_name>"` を各 FE の **fe/conf/hadoop_env.sh** ファイルの先頭に追加し、各 BE の **be/conf/hadoop_env.sh** ファイルの先頭に追加してユーザー名を設定できます。これらのファイルにユーザー名を設定した後は、各 FE および各 BE を再起動してパラメータ設定を有効にしてください。StarRocks クラスターごとに 1 つのユーザー名のみを設定できます。
- Paimon データをクエリするとき、StarRocks クラスターの FE と BE は HDFS クラスターにアクセスするために HDFS クライアントを使用します。ほとんどの場合、この目的を達成するために StarRocks クラスターを構成する必要はありません。StarRocks はデフォルトの構成で HDFS クライアントを起動します。次の状況でのみ StarRocks クラスターを構成する必要があります。
  - HDFS クラスターでハイアベイラビリティ (HA) が有効になっている場合: HDFS クラスターの **hdfs-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスと各 BE の **$BE_HOME/conf** パスに追加します。
  - HDFS クラスターで View File System (ViewFs) が有効になっている場合: HDFS クラスターの **core-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスおよび各 BE の **$BE_HOME/conf** パスに追加してください。

> **注記**
>
> クエリを送信すると不明なホストを示すエラーが返される場合は、ハードフリーマシンのホスト名と IP アドレスのマッピングを **/etc/hosts** パスに追加する必要があります。

### Kerberos 認証

HDFS クラスターや Hive メタストアで Kerberos 認証が有効になっている場合は、StarRocks クラスターを次のように構成してください。

- 各 FE および各 BE で `kinit -kt keytab_path principal` コマンドを実行して、Key Distribution Center (KDC) から Ticket Granting Ticket (TGT) を取得してください。このコマンドを実行するには、HDFS クラスターや Hive メタストアにアクセスする権限が必要です。このコマンドで KDC にアクセスするのは時間的に敏感なので、このコマンドを定期的に実行するために cron を使用する必要があります。
- 各 FE の **$FE_HOME/conf/fe.conf** ファイルと各 BE の **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加してください。この例では、`/etc/krb5.conf` は **krb5.conf** ファイルの保存パスです。必要に応じてパスを変更できます。

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

Paimon カタログの名前。命名規則は次のとおりです。

- 名前には、英字、数字 (0-9)、アンダースコア (_) を含めることができます。英字で始める必要があります。
- 名前は大文字と小文字を区別し、1023 文字を超えることはできません。

#### comment

Paimon カタログの説明。このパラメーターはオプションです。

#### type

データソースの種類。値を `paimon` に設定します。

#### CatalogParams

StarRocks が Paimon クラスターのメタデータにアクセスする方法に関するパラメーターのセットです。

`CatalogParams` で構成する必要のあるパラメーターに関する説明は、次の表に示されています。

| パラメーター                | 必須     | 説明                                                       |
| ------------------------ | -------- | ------------------------------------------------------------ |
| paimon.catalog.type      | はい      | Paimon クラスターで使用するメタストアのタイプ。このパラメーターを `filesystem` または `hive` に設定します。 |
| paimon.catalog.warehouse | はい      | Paimon データのウェアハウスストレージパス。 |
| hive.metastore.uris      | いいえ    | Hive メタストアの URI。形式: `thrift://<metastore_IP_address>:<metastore_port>`。Hive メタストアでハイアベイラビリティ (HA) が有効になっている場合、複数のメタストア URI を指定し、カンマ (`,`) で区切ることができます。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

> **注記**
>
> Hive メタストアを使用する場合、Paimon データをクエリする前に、Hive メタストアのホスト名と IP アドレスのマッピングを `/etc/hosts` パスに追加する必要があります。そうしないと、クエリを開始すると StarRocks は Hive メタストアにアクセスできなくなる場合があります。

#### StorageCredentialParams

ストレージシステムとの統合に関するパラメーターのセットです。このパラメーターセットはオプションです。

ストレージとして HDFS を使用する場合、`StorageCredentialParams` を構成する必要はありません。

ストレージとして AWS S3、他の S3 互換ストレージシステム、Microsoft Azure Storage、または Google GCS を使用する場合は、`StorageCredentialParams` を構成する必要があります。

##### AWS S3

Paimon クラスターのストレージとして AWS S3 を選択する場合、次のいずれかのアクションを実行してください。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams` を次のように構成してください。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 仮定されたロールベースの認証方法を選択する場合、`StorageCredentialParams` を次のように構成してください。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM ユーザーベースの認証方法を選択する場合、`StorageCredentialParams` を次のように構成してください。

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

`StorageCredentialParams` で構成する必要のあるパラメーターに関する説明は、次の表に示されています。

| パラメーター                   | 必須     | 説明                                                       |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい      | インスタンスプロファイルベースの認証方法および仮定されたロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | いいえ    | AWS S3 バケットに権限を持つ IAM ロールの ARN。AWS S3 へのアクセスに仮定されたロールベースの認証方法を使用する場合は、このパラメーターを指定する必要があります。 |
| aws.s3.region               | はい      | AWS S3 バケットのリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | いいえ    | IAM ユーザーのアクセスキー。AWS S3 へのアクセスに IAM ユーザーベースの認証方法を使用する場合は、このパラメーターを指定する必要があります。 |
| aws.s3.secret_key           | いいえ    | IAM ユーザーのシークレットキー。AWS S3 へのアクセスに IAM ユーザーベースの認証方法を使用する場合は、このパラメーターを指定する必要があります。 |

AWS S3へのアクセス認証方法の選択とAWS IAMコンソールでのアクセス制御ポリシーの構成については、[AWS S3へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

MinIOなどのS3互換ストレージシステムをストレージとして選択した場合、Paimonクラスターでの成功した統合を確実にするために、次のように`StorageCredentialParams`を構成します。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

次の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

| パラメータ                        | 必須     | 説明                                                    |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | はい      | SSL接続を有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`true`。 |
| aws.s3.enable_path_style_access | はい      | パス形式のアクセスを有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`false`。MinIOの場合、値を`true`に設定する必要があります。<br />パス形式のURLは、次の形式を使用します：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、米国西部（オレゴン）リージョンで`DOC-EXAMPLE-BUCKET1`というバケットを作成し、そのバケット内の`alice.jpg`オブジェクトにアクセスしたい場合、次のパス形式のURLを使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | はい      | AWS S3の代わりにS3互換ストレージシステムに接続するために使用するエンドポイント。 |
| aws.s3.access_key               | はい      | IAMユーザーのアクセスキー。                             |
| aws.s3.secret_key               | はい      | IAMユーザーのシークレットキー。                           |

##### Microsoft Azureストレージ

###### Azure Blob Storage

PaimonクラスターのストレージとしてBlob Storageを選択した場合、次のいずれかの操作を実行します。

- Shared Key認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  次の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

  | パラメータ                   | 必須     | 説明                                |
  | -------------------------- | -------- | ------------------------------------ |
  | azure.blob.storage_account | はい      | Blob Storageアカウントのユーザー名。     |
  | azure.blob.shared_key      | はい      | Blob Storageアカウントの共有キー。     |

- SASトークン認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  次の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

  | パラメータ                  | 必須     | 説明                                                    |
  | ------------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.account_name   | はい      | Blob Storageアカウントのユーザー名。                         |
  | azure.blob.container_name | はい      | データを格納するBlobコンテナの名前。                        |
  | azure.blob.sas_token      | はい      | Blob Storageアカウントへのアクセスに使用されるSASトークン。    |

###### Azure Data Lake Storage Gen1

PaimonクラスターのストレージとしてData Lake Storage Gen1を選択した場合、次のいずれかの操作を実行します。

- Managed Service Identity認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  次の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

  | パラメータ                                | 必須     | 説明                                                    |
  | ---------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | はい      | Managed Service Identity認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |

- Service Principal認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  次の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

  | パラメータ                     | 必須     | 説明                                                    |
  | ----------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | はい      | サービスプリンシパルのクライアント（アプリケーション）ID。    |
  | azure.adls1.oauth2_credential | はい      | 作成された新しいクライアント（アプリケーション）シークレットの値。 |
  | azure.adls1.oauth2_endpoint   | はい      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2

PaimonクラスターのストレージとしてData Lake Storage Gen2を選択した場合、次のいずれかの操作を実行します。

- Managed Identity認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  次の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

  | パラメータ                               | 必須     | 説明                                                    |
  | --------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | はい      | Managed Identity認証メソッドを有効にするかどうかを指定します。値を`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | はい      | アクセスするテナントのID。                                   |
  | azure.adls2.oauth2_client_id            | はい      | 管理されたアイデンティティのクライアント（アプリケーション）ID。 |

- Shared Key認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

  | パラメータ                  | 必須     | 説明                                                    |
  | ------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.storage_account | はい      | Data Lake Storage Gen2ストレージアカウントのユーザー名。     |
  | azure.adls2.shared_key      | はい      | Data Lake Storage Gen2ストレージアカウントの共有キー。     |

- Service Principal認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  次の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

  | パラメータ                          | 必須     | 説明                                                    |
  | ---------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | はい      | サービスプリンシパルのクライアント（アプリケーション）ID。    |
  | azure.adls2.oauth2_client_secret   | はい      | 作成された新しいクライアント（アプリケーション）シークレットの値。 |
  | azure.adls2.oauth2_client_endpoint | はい      | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

##### Google GCS

PaimonクラスターのストレージとしてGoogle GCSを選択した場合、次のいずれかの操作を実行します。

- VMベースの認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。

  | パラメータ                                  | デフォルト値 | 値の例     | 説明                                                    |
  | ------------------------------------------ | ------------- | ------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | FALSE         | TRUE          | 直接Compute Engineにバウンドされたサービスアカウントを使用するかどうかを指定します。 |

- サービスアカウントベースの認証メソッドを選択する場合、`StorageCredentialParams`を次のように構成します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  次の表は、`StorageCredentialParams`で構成する必要があるパラメータを説明しています。
```markdown
    + {R}
    | -------------------------------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""            | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントの作成時に生成されたJSONファイル内の電子メールアドレスです。 |
    | gcp.gcs.service_account_private_key_id | ""            | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウントの作成時に生成されたJSONファイル内のプライベートキーIDです。 |
    | gcp.gcs.service_account_private_key    | ""            | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウントの作成時に生成されたJSONファイル内のプライベートキーです。 |

- インパーソネーションベースの認証メソッドを選択する場合は、`StorageCredentialParams` を次のように構成します。

  - VMインスタンスをサービスアカウントになりすます：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次の表は、`StorageCredentialParams` で構成する必要があるパラメータを説明しています。

    | パラメータ                                  | デフォルト値 | 値の例       | 説明                                                        |
    | ------------------------------------------ | ------------- | ------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | FALSE         | TRUE          | Compute Engine にバインドされているサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""            | "hello"       | なりすますサービスアカウントです。                          |

  - サービスアカウント（一時的にメタサービスアカウントとして名前が付けられている）を、別のサービスアカウント（一時的にデータサービスアカウントとして名前が付けられている）になりすます：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表は、`StorageCredentialParams` で構成する必要があるパラメータを説明しています。

    | パラメータ                              | デフォルト値 | 値の例                                                       | 説明                                                       |
    | -------------------------------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""            | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウントの作成時に生成されたJSONファイル内の電子メールアドレスです。 |
    | gcp.gcs.service_account_private_key_id | ""            | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                 | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーIDです。 |
    | gcp.gcs.service_account_private_key    | ""            | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウントの作成時に生成されたJSONファイル内のプライベートキーです。 |
    | gcp.gcs.impersonation_service_account  | ""            | "hello"                                                      | なりすますデータサービスアカウントです。                   |

### 使用例

以下の使用例では、Paimonクラスタからデータを問い合わせるためのメタストアタイプ `paimon.catalog.type` が `filesystem` に設定された Paimonカタログ `paimon_catalog_fs` を作成します。

#### AWS S3

- インスタンスプロファイルベースの認証メソッドを選択する場合、次のようなコマンドを実行します：

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

- 仮定されるロールベースの認証メソッドを選択する場合、次のようなコマンドを実行します：

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

- IAMユーザーベースの認証メソッドを選択する場合、次のようなコマンドを実行します：

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

MinIO を例に使用します。以下のようなコマンドを実行します：

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

##### Azure Blobストレージ

- シェアドキー認証メソッドを選択する場合、以下のようなコマンドを実行します：

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

- SASトークン認証メソッドを選択する場合、以下のようなコマンドを実行します：

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

- マネージドサービスアイデンティティ認証メソッドを選択する場合、以下のようなコマンドを実行します：

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

- サービスプリンシパル認証メソッドを選択する場合、以下のようなコマンドを実行します：

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

- マネージドアイデンティティ認証メソッドを選択する場合、以下のようなコマンドを実行します：

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

- シェアドキー認証メソッドを選択する場合、以下のようなコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
```
```sql
Catalog paimon
PROPERTIES
(
    "paimon.catalog.type" = "filesystem",
    "paimon.catalog.warehouse" = "<adls2_paimon_warehouse_path>",
    "azure.adls2.storage_account" = "<storage_account_name>",
    "azure.adls2.shared_key" = "<shared_key>"
  );

- サービスプリンシパル認証メソッドを選択した場合は、次のようなコマンドを実行してください：

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

#### Google GCS

- VMベースの認証メソッドを選択した場合は、次のようなコマンドを実行してください：

```SQL
CREATE EXTERNAL CATALOG paimon_catalog_fs
PROPERTIES
(
    "type" = "paimon",
    "paimon.catalog.type" = "filesystem",
    "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
    "gcp.gcs.use_compute_engine_service_account" = "true"
);

- サービスアカウントベースの認証メソッドを選択した場合は、次のようなコマンドを実行してください：

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

- インパーソネーションベースの認証メソッドを選択した場合：

  - VMインスタンスがサービスアカウントを偽装する場合は、次のようなコマンドを実行してください：

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

  - サービスアカウントが別のサービスアカウントを偽装する場合は、次のようなコマンドを実行してください：

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

## Paimonカタログの表示

現在のStarRocksクラスタ内のすべてのカタログをクエリするには、[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用できます：

```SQL
SHOW CATALOGS;
```

外部カタログの作成文をクエリするには、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用できます。次の例は、Paimonカタログ名が`paimon_catalog_fs`の作成文をクエリしています：

```SQL
SHOW CREATE CATALOG paimon_catalog_fs;
```

## Paimonカタログの削除

外部カタログを削除するには、[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用できます。

次の例は、`paimon_catalog_fs`という名前のPaimonカタログを削除しています：

```SQL
DROP Catalog paimon_catalog_fs;
```

## Paimonテーブルのスキーマの表示

Paimonテーブルのスキーマを表示するには、次の構文のいずれかを使用できます：

- スキーマの表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>;
  ```

- CREATE文からスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>;
  ```

## Paimonテーブルのクエリ

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Paimonクラスタ内のデータベースを表示できます：

   ```SQL
   SHOW DATABASES FROM <catalog_name>;
   ```

2. [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションで宛先カタログに切り替えることができます：

   ```SQL
   SET CATALOG <catalog_name>;
   ```

   次に、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、現在のセッションでアクティブなデータベースを指定できます：

   ```SQL
   USE <db_name>;
   ```

   または、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、直接宛先カタログ内のアクティブなデータベースを指定できます：

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

3. 指定したデータベース内の宛先テーブルをクエリするには、[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用できます：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10;
   ```

## Paimonからデータをロード

OLAPテーブルが`olap_tbl`という名前の場合、次のようにデータを変換してロードできます：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM paimon_table;
```