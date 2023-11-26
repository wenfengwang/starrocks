---
displayed_sidebar: "Japanese"
---

# Paimonカタログ

StarRocksは、v3.1以降でPaimonカタログをサポートしています。

Paimonカタログは、Apache Paimonからデータをクエリするための外部カタログであり、インジェストなしでデータをクエリできます。

また、Paimonカタログをベースに[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用して、Paimonから直接データを変換およびロードすることもできます。

PaimonクラスタでSQLワークロードを正常に実行するためには、StarRocksクラスタが2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはAWS S3、Microsoft Azure Storage、Google GCSなどのオブジェクトストレージ、またはその他のS3互換ストレージシステム（たとえば、MinIO）。
- ファイルシステムまたはHiveメタストアのようなメタストア。

## 使用上の注意

Paimonカタログはデータのクエリにのみ使用できます。Paimonカタログを使用してPaimonクラスタにデータを削除、削除、または挿入することはできません。

## 統合の準備

Paimonカタログを作成する前に、StarRocksクラスタがPaimonクラスタのストレージシステムとメタストアと統合できることを確認してください。

### AWS IAM

PaimonクラスタがストレージとしてAWS S3を使用する場合は、適切な認証方法を選択し、StarRocksクラスタが関連するAWSクラウドリソースにアクセスできるように必要な準備を行ってください。

以下の認証方法が推奨されています。

- インスタンスプロファイル（推奨）
- 仮定される役割
- IAMユーザー

上記の3つの認証方法のうち、インスタンスプロファイルが最も一般的に使用されています。

詳細については、[AWS IAMでの認証の準備](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)を参照してください。

### HDFS

ストレージとしてHDFSを選択する場合、StarRocksクラスタを次のように構成してください。

- （オプション）HDFSクラスタとHiveメタストアにアクセスするために使用されるユーザー名を設定します。デフォルトでは、StarRocksはFEおよびBEプロセスのユーザー名を使用してHDFSクラスタとHiveメタストアにアクセスします。各FEの**fe/conf/hadoop_env.sh**ファイルと各BEの**be/conf/hadoop_env.sh**ファイルの先頭に`export HADOOP_USER_NAME="<user_name>"`を追加することで、ユーザー名を設定することもできます。これらのファイルでユーザー名を設定した後、各FEと各BEを再起動してパラメータ設定を有効にします。StarRocksクラスタごとに1つのユーザー名のみを設定できます。
- Paimonデータをクエリする際、StarRocksクラスタのFEおよびBEはHDFSクライアントを使用してHDFSクラスタにアクセスします。ほとんどの場合、この目的を達成するためにStarRocksクラスタを構成する必要はありません。StarRocksはデフォルトの設定でHDFSクライアントを起動します。次の状況では、StarRocksクラスタを構成する必要があります。
  - HDFSクラスタに対して高可用性（HA）が有効になっている場合：HDFSクラスタの**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。
  - HDFSクラスタに対してView File System（ViewFs）が有効になっている場合：HDFSクラスタの**core-site.xml**ファイルを各FEの**$FE_HOME/conf**パスと各BEの**$BE_HOME/conf**パスに追加します。

> **注意**
>
> クエリを送信する際にホスト名が不明と示されるエラーが返される場合は、HDFSクラスタノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに追加する必要があります。

### Kerberos認証

HDFSクラスタまたはHiveメタストアにKerberos認証が有効になっている場合は、StarRocksクラスタを次のように構成してください。

- 各FEおよび各BEで`kinit -kt keytab_path principal`コマンドを実行して、Key Distribution Center（KDC）からTicket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFSクラスタとHiveメタストアにアクセスする権限が必要です。このコマンドを使用してKDCにアクセスする場合は、時間的に制約があることに注意してください。したがって、このコマンドを定期的に実行するためにcronを使用する必要があります。
- 各FEの**$FE_HOME/conf/fe.conf**ファイルと各BEの**$BE_HOME/conf/be.conf**ファイルに`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`を追加します。この例では、`/etc/krb5.conf`は**krb5.conf**ファイルの保存パスです。必要に応じてパスを変更できます。

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

### パラメータ

#### catalog_name

Paimonカタログの名前です。命名規則は次のとおりです。

- 名前には、文字、数字（0-9）、アンダースコア（_）を含めることができます。先頭は文字で始める必要があります。
- 名前は大文字と小文字を区別し、長さが1023文字を超えることはできません。

#### comment

Paimonカタログの説明です。このパラメータはオプションです。

#### type

データソースのタイプです。値を`paimon`に設定します。

#### CatalogParams

StarRocksがPaimonクラスタのメタデータにアクセスする方法に関するパラメータのセットです。

`CatalogParams`で構成する必要があるパラメータについては、以下の表に説明があります。

| パラメータ                | 必須 | 説明                                                         |
| ------------------------ | ---- | ------------------------------------------------------------ |
| paimon.catalog.type      | Yes  | Paimonクラスタで使用するメタストアのタイプ。このパラメータを`filesystem`または`hive`に設定します。 |
| paimon.catalog.warehouse | Yes  | Paimonデータのウェアハウスストレージパス。                    |
| hive.metastore.uris      | No   | HiveメタストアのURI。形式：`thrift://<metastore_IP_address>:<metastore_port>`。Hiveメタストアに対して高可用性（HA）が有効になっている場合は、複数のメタストアURIを指定し、カンマ（`,`）で区切って指定できます。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

> **注意**
>
> Hiveメタストアを使用する場合は、Paimonデータをクエリする前に、Hiveメタストアノードのホスト名とIPアドレスのマッピングを**/etc/hosts**パスに追加する必要があります。そうしないと、クエリを開始するとStarRocksがHiveメタストアにアクセスできなくなる場合があります。

#### StorageCredentialParams

StarRocksがストレージシステムと統合する方法に関するパラメータのセットです。このパラメータセットはオプションです。

ストレージとしてHDFSを使用する場合、`StorageCredentialParams`を構成する必要はありません。

AWS S3、その他のS3互換ストレージシステム、Microsoft Azure Storage、またはGoogle GCSを使用する場合は、`StorageCredentialParams`を構成する必要があります。

##### AWS S3

PaimonクラスタのストレージとしてAWS S3を選択する場合は、次のいずれかの操作を実行してください。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 仮定される役割ベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAMユーザーベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

| パラメータ                   | 必須 | 説明                                                         |
| --------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes  | インスタンスプロファイルベースの認証方法と仮定される役割ベースの認証方法を有効にするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。 |
| aws.s3.iam_role_arn         | No   | AWS S3バケットに特権を持つIAMロールのARN。AWS S3へのアクセスに仮定される役割ベースの認証方法を使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.region               | Yes  | AWS S3バケットが存在するリージョン。例：`us-west-1`。         |
| aws.s3.access_key           | No   | IAMユーザーのアクセスキー。IAMユーザーベースの認証方法を使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | No   | IAMユーザーのシークレットキー。IAMユーザーベースの認証方法を使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。 |

AWS S3へのアクセス方法の選択とAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS S3へのアクセスの認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

MinIOを例に挙げます。次のように`StorageCredentialParams`を構成して、正常な統合を確保してください。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

| パラメータ                       | 必須 | 説明                                                         |
| ------------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | Yes  | SSL接続を有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`true`。 |
| aws.s3.enable_path_style_access | Yes  | パス形式のアクセスを有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`false`。MinIOの場合、値を`true`に設定する必要があります。<br />パス形式のURLは、次の形式を使用します：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、US West（Oregon）リージョンで`DOC-EXAMPLE-BUCKET1`というバケットを作成し、そのバケットの`alice.jpg`オブジェクトにアクセスしたい場合、次のパス形式のURLを使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | Yes  | AWS S3の代わりにS3互換ストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key               | Yes  | IAMユーザーのアクセスキー。                                 |
| aws.s3.secret_key               | Yes  | IAMユーザーのシークレットキー。                             |

##### Microsoft Azure Storage

###### Azure Blob Storage

PaimonクラスタのストレージとしてBlob Storageを選択する場合、次のいずれかの操作を実行してください。

- 共有キー認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

  | パラメータ                  | 必須 | 説明                                                         |
  | -------------------------- | ---- | ------------------------------------------------------------ |
  | azure.blob.storage_account | Yes  | Blob Storageアカウントのユーザー名。                          |
  | azure.blob.shared_key      | Yes  | Blob Storageアカウントの共有キー。                            |

- SASトークン認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

  | パラメータ                | 必須 | 説明                                                         |
  | ------------------------ | ---- | ------------------------------------------------------------ |
  | azure.blob.account_name   | Yes  | Blob Storageアカウントのユーザー名。                          |
  | azure.blob.container_name | Yes  | データを格納するBlobコンテナの名前。                          |
  | azure.blob.sas_token      | Yes  | Blob Storageアカウントにアクセスするために使用されるSASトークン。 |

###### Azure Data Lake Storage Gen1

PaimonクラスタのストレージとしてData Lake Storage Gen1を選択する場合、次のいずれかの操作を実行してください。

- Managed Service Identity認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

  | パラメータ                                | 必須 | 説明                                                         |
  | ---------------------------------------- | ---- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes  | Managed Service Identity認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |

- Service Principal認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

  | パラメータ                     | 必須 | 説明                                                         |
  | ----------------------------- | ---- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | Yes  | サービスプリンシパルのクライアント（アプリケーション）ID。    |
  | azure.adls1.oauth2_credential | Yes  | 作成時に生成された新しいクライアント（アプリケーション）シークレットの値。 |
  | azure.adls1.oauth2_endpoint   | Yes  | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

###### Azure Data Lake Storage Gen2

PaimonクラスタのストレージとしてData Lake Storage Gen2を選択する場合、次のいずれかの操作を実行してください。

- Managed Identity認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

  | パラメータ                               | 必須 | 説明                                                         |
  | --------------------------------------- | ---- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes  | Managed Identity認証方法を有効にするかどうかを指定します。値を`true`に設定します。 |
  | azure.adls2.oauth2_tenant_id            | Yes  | アクセスするテナントのID。                                   |
  | azure.adls2.oauth2_client_id            | Yes  | 管理されるIDのクライアント（アプリケーション）ID。            |

- Shared Key認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

  | パラメータ                   | 必須 | 説明                                                         |
  | --------------------------- | ---- | ------------------------------------------------------------ |
  | azure.adls2.storage_account | Yes  | Data Lake Storage Gen2ストレージアカウントのユーザー名。      |
  | azure.adls2.shared_key      | Yes  | Data Lake Storage Gen2ストレージアカウントの共有キー。        |

- Service Principal認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

  | パラメータ                          | 必須 | 説明                                                         |
  | ---------------------------------- | ---- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | Yes  | サービスプリンシパルのクライアント（アプリケーション）ID。    |
  | azure.adls2.oauth2_client_secret   | Yes  | 作成時に生成された新しいクライアント（アプリケーション）シークレットの値。 |
  | azure.adls2.oauth2_client_endpoint | Yes  | サービスプリンシパルまたはアプリケーションのOAuth 2.0トークンエンドポイント（v1）。 |

##### Google GCS

PaimonクラスタのストレージとしてGoogle GCSを選択する場合、次のいずれかの操作を実行してください。

- VMベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

  | パラメータ                                  | デフォルト値 | 値の例 | 説明                                                         |
  | ------------------------------------------ | ------------ | ------ | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | FALSE        | TRUE   | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

  | パラメータ                              | デフォルト値 | 値の例                                                         | 説明                                                         |
  | -------------------------------------- | ------------ | -------------------------------------------------------------- | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""           | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントのJSONファイル作成時に生成されたメールアドレス。 |
  | gcp.gcs.service_account_private_key_id | ""           | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                     | サービスアカウントのJSONファイル作成時に生成されたプライベートキーID。 |
  | gcp.gcs.service_account_private_key    | ""           | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"    | サービスアカウントのJSONファイル作成時に生成されたプライベートキー。 |

- 偽装ベースの認証方法を選択する場合、`StorageCredentialParams`を次のように構成してください。

  - VMインスタンスがサービスアカウントを偽装する場合：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

    | パラメータ                                  | デフォルト値 | 値の例 | 説明                                                         |
    | ------------------------------------------ | ------------ | ------ | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | FALSE        | TRUE   | Compute Engineにバインドされたサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""           | "hello" | 偽装したいサービスアカウント。                                |

  - サービスアカウント（一時的にメタサービスアカウントと呼ばれる）が別のサービスアカウント（一時的にデータサービスアカウントと呼ばれる）を偽装する場合：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    以下の表には、`StorageCredentialParams`で構成する必要のあるパラメータについて説明があります。

    | パラメータ                              | デフォルト値 | 値の例                                                         | 説明                                                         |
    | -------------------------------------- | ------------ | -------------------------------------------------------------- | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""           | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウントのJSONファイル作成時に生成されたメールアドレス。 |
    | gcp.gcs.service_account_private_key_id | ""           | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウントのJSONファイル作成時に生成されたプライベートキーID。 |
    | gcp.gcs.service_account_private_key    | ""           | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウントのJSONファイル作成時に生成されたプライベートキー。 |
    | gcp.gcs.impersonation_service_account  | ""           | "hello"                                                      | 偽装したいデータサービスアカウント。                        |

### 例

次の例では、`paimon_catalog_fs`という名前のPaimonカタログを作成し、メタストアタイプ`paimon.catalog.type`を`filesystem`に設定してPaimonクラスタからデータをクエリします。

#### AWS S3

- インスタンスプロファイルベースの認証方法を選択する場合、次のようなコマンドを実行します。

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

- 仮定される役割ベースの認証方法を選択する場合、次のようなコマンドを実行します。

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

- IAMユーザーベースの認証方法を選択する場合、次のようなコマンドを実行します。

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

MinIOを例に挙げます。次のようなコマンドを実行します。

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

- 共有キー認証方法を選択する場合、次のようなコマンドを実行します。

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

- SASトークン認証方法を選択する場合、次のようなコマンドを実行します。

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

- Managed Service Identity認証方法を選択する場合、次のようなコマンドを実行します。

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

- Service Principal認証方法を選択する場合、次のようなコマンドを実行します。

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

- Managed Identity認証方法を選択する場合、次のようなコマンドを実行します。

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

- Shared Key認証方法を選択する場合、次のようなコマンドを実行します。

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

- Service Principal認証方法を選択する場合、次のようなコマンドを実行します。

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

- VMベースの認証方法を選択する場合、次のようなコマンドを実行します。

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

- サービスアカウントベースの認証方法を選択する場合、次のようなコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  プロパティ
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

  - VMインスタンスをサービスアカウントになりすます場合、以下のようなコマンドを実行します：

    ```SQL
    CREATE EXTERNAL CATALOG paimon_catalog_fs
    プロパティ
    (
        "type" = "paimon",
        "paimon.catalog.type" = "filesystem",
        "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    );
    ```

  - サービスアカウントが別のサービスアカウントになりすます場合、以下のようなコマンドを実行します：

    ```SQL
    CREATE EXTERNAL CATALOG paimon_catalog_fs
    プロパティ
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

## Paimonカタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスタ内のすべてのカタログをクエリできます：

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリできます。次の例では、`paimon_catalog_fs`という名前のPaimonカタログの作成ステートメントをクエリしています：

```SQL
SHOW CREATE CATALOG paimon_catalog_fs;
```

## Paimonカタログの削除

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、外部カタログを削除できます。

次の例では、`paimon_catalog_fs`という名前のPaimonカタログを削除しています：

```SQL
DROP Catalog paimon_catalog_fs;
```

## Paimonテーブルのスキーマの表示

次の構文のいずれかを使用して、Paimonテーブルのスキーマを表示できます：

- スキーマの表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>;
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>;
  ```

## Paimonテーブルのクエリ

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、Paimonクラスタ内のデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>;
   ```

2. [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションでの宛先カタログに切り替えます：

   ```SQL
   SET CATALOG <catalog_name>;
   ```

   次に、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、現在のセッションでのアクティブなデータベースを指定します：

   ```SQL
   USE <db_name>;
   ```

   または、[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用して、宛先カタログ内のアクティブなデータベースを直接指定することもできます：

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、指定したデータベースの宛先テーブルをクエリします：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10;
   ```

## Paimonからデータをロードする

`olap_tbl`という名前のOLAPテーブルがあると仮定し、以下のようにデータを変換してロードできます：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM paimon_table;
```
