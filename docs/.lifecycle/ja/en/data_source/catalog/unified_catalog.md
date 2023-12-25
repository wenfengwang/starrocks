---
displayed_sidebar: English
---

# 統合カタログ

統合カタログは、Apache Hive™、Apache Iceberg、Apache Hudi、Delta Lake などのさまざまなデータソースのテーブルを、取り込みなしで統合データソースとして処理するために、StarRocks v3.2 以降で提供される外部カタログの一種です。統合カタログを使用すると、次のことができます。

- Hive、Iceberg、Hudi、Delta Lake に格納されているデータに対して、手動でテーブルを作成することなく、データを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) または非同期マテリアライズドビュー（v2.5 以降でサポート）を使用して、Hive、Iceberg、Hudi、Delta Lake に保存されているデータを処理し、そのデータを StarRocks にロードします。
- StarRocks で操作を実行して、Hive と Iceberg のデータベースとテーブルを作成または削除します。

統合データソースで SQL ワークロードを成功させるには、StarRocks クラスタを以下の2つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム（HDFS）またはオブジェクトストレージ（AWS S3、Microsoft Azure Storage、Google GCS、その他の S3 互換ストレージシステムなど）（例：MinIO）

- メタストア（例：Hive メタストアや AWS Glue）

  > **注記**
  >
  > ストレージとして AWS S3 を選択した場合は、メタストアとして HMS または AWS Glue を使用できます。他のストレージシステムを選択した場合は、HMS をメタストアとしてのみ使用できます。

## 制限事項

一つの統合カタログは、単一のストレージシステムおよび単一のメタストアサービスとの統合のみをサポートします。したがって、StarRocks と統合データソースとして統合するすべてのデータソースが、同じストレージシステムとメタストアサービスを使用していることを確認してください。

## 使用上の注意

- サポートされているファイル形式とデータ型については、[Hive カタログ](../../data_source/catalog/hive_catalog.md)、[Iceberg カタログ](../../data_source/catalog/iceberg_catalog.md)、[Hudi カタログ](../../data_source/catalog/hudi_catalog.md)、[Delta Lake カタログ](../../data_source/catalog/deltalake_catalog.md)の「使用上の注意」セクションを参照してください。

- 形式固有の操作は、特定のテーブル形式に対してのみサポートされます。たとえば、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) と [DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) は Hive と Iceberg でのみサポートされ、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) は Hive と Hudi でのみサポートされます。

  [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) ステートメントを使用して統合カタログ内にテーブルを作成する場合は、`ENGINE` パラメーターを使用してテーブル形式（Hive または Iceberg）を指定します。

## 統合の準備

統合カタログを作成する前に、StarRocks クラスタが統合データソースのストレージシステムおよびメタストアと統合できることを確認してください。

### AWS IAM

AWS S3 をストレージとして使用する場合、または AWS Glue をメタストアとして使用する場合は、適切な認証方法を選択し、必要な準備をして、StarRocks クラスタが関連する AWS クラウドリソースにアクセスできるようにします。詳細については、「[AWS リソースへの認証 - 準備](../../integrations/authenticate_to_aws_resources.md#preparations)」を参照してください。

### HDFS

ストレージとして HDFS を選択した場合は、StarRocks クラスタを次のように構成します。

- （オプション）HDFS クラスターと Hive メタストアへのアクセスに使用するユーザー名を設定します。デフォルトでは、StarRocks は FE および BE プロセスのユーザー名を使用して、HDFS クラスターと Hive メタストアにアクセスします。また、各 FE の **fe/conf/hadoop_env.sh** ファイルと各 BE の **be/conf/hadoop_env.sh** ファイルの先頭に `export HADOOP_USER_NAME="<user_name>"` を追加して、ユーザー名を設定することもできます。これらのファイルにユーザー名を設定した後、各 FE と各 BE を再起動して、パラメータ設定を有効にします。StarRocks クラスタごとに設定できるユーザー名は1つだけです。
- データをクエリする際、StarRocks クラスタの FE と BE は HDFS クライアントを使用して HDFS クラスタにアクセスします。ほとんどの場合、その目的を達成するために StarRocks クラスタを設定する必要はありません。StarRocks はデフォルトの設定を使用して HDFS クライアントを起動します。StarRocks クラスタを設定する必要があるのは、以下の状況のみです。
  - HDFS クラスタで高可用性（HA）が有効になっている場合：HDFS クラスタの **hdfs-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスと各 BE の **$BE_HOME/conf** パスに追加します。
  - View File System（ViewFs）が HDFS クラスタで有効になっている場合：HDFS クラスタの **core-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスと各 BE の **$BE_HOME/conf** パスに追加します。

> **注記**
>
> クエリを送信した際に不明なホストを示すエラーが返される場合は、HDFS クラスタノードのホスト名と IP アドレスのマッピングを **/etc/hosts** に追加する必要があります。

### Kerberos 認証

HDFS クラスタまたは Hive メタストアで Kerberos 認証が有効になっている場合は、StarRocks クラスタを次のように構成します。

- 各 FE および各 BE で `kinit -kt keytab_path principal` コマンドを実行して、Key Distribution Center（KDC）から Ticket Granting Ticket（TGT）を取得します。このコマンドを実行するには、HDFS クラスターと Hive メタストアへのアクセス権が必要です。KDC へのアクセスは時間に敏感なため、このコマンドを定期的に実行するために cron を使用する必要があります。
- 各 FE の **$FE_HOME/conf/fe.conf** ファイルと各 BE の **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。この例では、`/etc/krb5.conf` は krb5.conf ファイルの保存パスです。必要に応じてパスを変更できます。

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

統合カタログの名前。命名規則は次のとおりです。

- 名前には、文字、数字（0-9）、およびアンダースコア（_）を含めることができます。文字で始まる必要があります。
- 名前は大文字と小文字が区別され、長さは 1023 文字を超えることはできません。

#### comment

統合カタログの説明。このパラメータはオプションです。

#### type

データソースのタイプ。値を `unified` に設定します。

#### MetastoreParams

StarRocks がメタストアと統合する方法に関するパラメータのセット。

##### Hive メタストア

統合データソースのメタストアとして Hive メタストアを選択する場合、`MetastoreParams` を次のように構成します。

```SQL
"unified.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注記**
>
> データをクエリする前に、Hive メタストアノードのホスト名と IP アドレスのマッピングを **/etc/hosts** に追加する必要があります。そうしないと、クエリを開始する際に StarRocks が Hive メタストアへのアクセスに失敗する可能性があります。

以下の表は、`MetastoreParams` で設定する必要があるパラメータを説明しています。

| パラメータ              | 必須 | 説明                                                  |
| ---------------------- | -------- | ------------------------------------------------------------ |
| unified.metastore.type | はい      | 統合データソースで使用するメタストアのタイプです。値を `hive` に設定してください。 |
| hive.metastore.uris    | はい      | Hive メタストアの URI。形式: `thrift://<metastore_IP_address>:<metastore_port>`。Hive メタストアで高可用性 (HA) が有効な場合、複数のメタストア URI を指定し、コンマ (`,`) で区切ることができます。例: `"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

データソースのメタストアとして AWS Glue を選択する場合（ストレージとして AWS S3 を選択した場合のみサポートされます）、以下のアクションのいずれかを実行してください。

- インスタンスプロファイルベースの認証方法を選択する場合、`MetastoreParams` を以下のように設定します：

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 想定ロールベースの認証方法を選択する場合、`MetastoreParams` を以下のように設定します：

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- IAM ユーザーベースの認証方法を選択する場合、`MetastoreParams` を以下のように設定します：

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

以下の表は、`MetastoreParams` で設定する必要があるパラメータを説明しています。

| パラメーター                     | 必須 | 説明                                                  |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| unified.metastore.type        | はい      | 統合データソースで使用するメタストアのタイプです。値を `glue` に設定してください。 |
| aws.glue.use_instance_profile | はい      | インスタンスプロファイルベースの認証方法または想定ロールベースの認証を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.glue.iam_role_arn         | いいえ       | AWS Glue Data Catalog に権限を持つ IAM ロールの ARN。想定ロールベースの認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.region               | はい      | AWS Glue Data Catalog が存在するリージョン。例: `us-west-1`。 |
| aws.glue.access_key           | いいえ       | AWS IAM ユーザーのアクセスキー。IAM ユーザーベースの認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.glue.secret_key           | いいえ       | AWS IAM ユーザーのシークレットキー。IAM ユーザーベースの認証方法を使用して AWS Glue にアクセスする場合、このパラメータを指定する必要があります。 |

AWS Glue へのアクセスに使用する認証方法の選択と、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS Glue へのアクセスに関する認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)を参照してください。

#### StorageCredentialParams

StarRocks がストレージシステムと統合する方法に関するパラメータセットです。このパラメータセットはオプションです。

HDFS をストレージとして使用する場合、`StorageCredentialParams` を設定する必要はありません。

AWS S3、その他の S3 互換ストレージシステム、Microsoft Azure Storage、または Google GCS をストレージとして使用する場合は、`StorageCredentialParams` を設定する必要があります。

##### AWS S3

ストレージとして AWS S3 を選択する場合、以下のアクションのいずれかを実行してください。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 想定ロールベースの認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- IAM ユーザーベースの認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下の表は、`StorageCredentialParams` で設定する必要があるパラメータを説明しています。

| パラメーター                   | 必須 | 説明                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい      | インスタンスプロファイルベースの認証方法または想定ロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | いいえ       | AWS S3 バケットに権限を持つ IAM ロールの ARN。想定ロールベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.region               | はい      | AWS S3 バケットが存在するリージョン。例: `us-west-1`。 |
| aws.s3.access_key           | いいえ       | IAM ユーザーのアクセスキー。IAM ユーザーベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAM ユーザーのシークレットキー。IAM ユーザーベースの認証方法を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3 へのアクセスに使用する認証方法の選択と、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS S3 へのアクセスに関する認証パラメータ](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージシステム

MinIO などの S3 互換ストレージシステムをストレージとして選択する場合、統合を成功させるために `StorageCredentialParams` を以下のように設定してください。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表は、`StorageCredentialParams` で設定する必要があるパラメータを説明しています。

| パラメーター                        | 必須 | 説明                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | はい      | SSL 接続を有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access  | はい      | パス形式のアクセスを有効にするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。MinIO を使用する場合は、値を `true` に設定する必要があります。<br />パス形式の URL は次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部 (オレゴン) リージョンに `DOC-EXAMPLE-BUCKET1` という名前のバケットを作成し、そのバケット内の `alice.jpg` オブジェクトにアクセスする場合、次のパス形式の URL を使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | はい      | AWS S3 ではなく、S3 互換ストレージシステムに接続するために使用されるエンドポイントです。 |
| aws.s3.access_key                | はい      | IAM ユーザーのアクセスキーです。 |

| aws.s3.secret_key                | Yes      | IAM ユーザーのシークレットキーです。 |

##### Microsoft Azure ストレージ

###### Azure Blob Storage

ストレージとして Blob Storage を選択する場合、以下のいずれかの操作を行ってください。

- 共有キー認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` で設定が必要なパラメーターは以下の表に記載されています。

  | **パラメーター**              | **必須** | **説明**                                        |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | Yes          | Blob Storage アカウントのユーザー名です。   |
  | azure.blob.shared_key      | Yes          | Blob Storage アカウントの共有キーです。     |

- SAS トークン認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams` で設定が必要なパラメーターは以下の表に記載されています。

  | **パラメーター**             | **必須** | **説明**                                                |
  | ------------------------- | ------------ | ------------------------------------------------------ |
  | azure.blob.storage_account| Yes          | Blob Storage アカウントのユーザー名です。             |
  | azure.blob.container      | Yes          | データを格納する blob コンテナの名前です。            |
  | azure.blob.sas_token      | Yes          | Blob Storage アカウントへのアクセスに使用する SAS トークンです。 |

###### Azure Data Lake Storage Gen1

ストレージとして Data Lake Storage Gen1 を選択する場合、以下のいずれかの操作を行ってください。

- 管理対象サービス ID 認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` で設定が必要なパラメーターは以下の表に記載されています。

  | **パラメーター**                            | **必須** | **説明**                                                |
  | ---------------------------------------- | ------------ | ------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes          | 管理対象サービス ID 認証方法を有効にするかどうかを指定します。`true` に設定してください。 |

- サービスプリンシパル認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams` で設定が必要なパラメーターは以下の表に記載されています。

  | **パラメーター**                 | **必須** | **説明**                                                |
  | ----------------------------- | ------------ | ------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | Yes          | サービスプリンシパルのクライアント（アプリケーション）IDです。        |
  | azure.adls1.oauth2_credential | Yes          | 新しく作成されたクライアント（アプリケーション）シークレットの値です。    |
  | azure.adls1.oauth2_endpoint   | Yes          | サービスプリンシパルまたはアプリケーションの OAuth 2.0 トークンエンドポイント（v1）です。 |

###### Azure Data Lake Storage Gen2

ストレージとして Data Lake Storage Gen2 を選択する場合、以下のいずれかの操作を行ってください。

- マネージド ID 認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  `StorageCredentialParams` で設定が必要なパラメーターは以下の表に記載されています。

  | **パラメーター**                           | **必須** | **説明**                                                |
  | --------------------------------------- | ------------ | ------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes          | マネージド ID 認証方法を有効にするかどうかを指定します。`true` に設定してください。 |
  | azure.adls2.oauth2_tenant_id            | Yes          | アクセスしたいテナントの ID です。                      |
  | azure.adls2.oauth2_client_id            | Yes          | マネージド ID のクライアント（アプリケーション）IDです。         |

- 共有キー認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams` で設定が必要なパラメーターは以下の表に記載されています。

  | **パラメーター**               | **必須** | **説明**                                                |
  | --------------------------- | ------------ | ------------------------------------------------------ |
  | azure.adls2.storage_account | Yes          | Data Lake Storage Gen2 ストレージアカウントのユーザー名です。 |
  | azure.adls2.shared_key      | Yes          | Data Lake Storage Gen2 ストレージアカウントの共有キーです。 |

- サービスプリンシパル認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams` で設定が必要なパラメーターは以下の表に記載されています。

  | **パラメーター**                      | **必須** | **説明**                                                |
  | ---------------------------------- | ------------ | ------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | Yes          | サービスプリンシパルのクライアント（アプリケーション）IDです。        |
  | azure.adls2.oauth2_client_secret   | Yes          | 新しく作成されたクライアント（アプリケーション）シークレットの値です。    |
  | azure.adls2.oauth2_client_endpoint | Yes          | サービスプリンシパルまたはアプリケーションの OAuth 2.0 トークンエンドポイント（v1）です。 |

##### Google GCS

ストレージとして Google GCS を選択する場合、以下のいずれかの操作を行ってください。

- VM ベースの認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams` で設定が必要なパラメーターは以下の表に記載されています。

  | **パラメーター**                              | **デフォルト値** | **値の例** | **説明**                                                |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engine に紐付けられたサービスアカウントを直接使用するかどうかを指定します。 |

- サービスアカウントベースの認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  `StorageCredentialParams` で設定が必要なパラメーターは以下の表に記載されています。

  | **パラメーター**                          | **デフォルト値** | **値の例**                                        | **説明**                                                |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービスアカウント作成時に生成された JSON ファイル内のメールアドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービスアカウント作成時に生成された JSON ファイル内のプライベートキー ID です。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | サービスアカウント作成時に生成された JSON ファイル内のプライベートキーです。 |

- 偽装ベースの認証方法を選択する場合、`StorageCredentialParams` を以下のように設定します：

  - VM インスタンスがサービスアカウントを偽装するように設定します：
  
    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams` で設定が必要なパラメーターは以下の表に記載されています。

    | **パラメータ**                              | **デフォルト値** | **値の例** | **説明**                                              |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | Compute Engineに紐づけられたサービスアカウントを直接使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | なりすますサービスアカウントを指定します。            |

  - サービスアカウント（一時的にメタサービスアカウントと呼ぶ）が別のサービスアカウント（一時的にデータサービスアカウントと呼ぶ）をなりすますように設定します：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

    | **パラメータ**                          | **デフォルト値** | **値の例**                                        | **説明**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | メタサービスアカウント作成時に生成されたJSONファイル内のメールアドレスです。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキーIDです。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY-----xxxx-----END PRIVATE KEY-----\n"  | メタサービスアカウント作成時に生成されたJSONファイル内のプライベートキーです。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | なりすますデータサービスアカウントを指定します。       |

#### MetadataUpdateParams

StarRocksがHive、Hudi、Delta Lakeのキャッシュされたメタデータを更新する方法に関するパラメータセットです。このパラメータセットはオプションです。Hive、Hudi、Delta Lakeからキャッシュされたメタデータを更新するポリシーの詳細については、[Hiveカタログ](../../data_source/catalog/hive_catalog.md)、[Hudiカタログ](../../data_source/catalog/hudi_catalog.md)、および[Delta Lakeカタログ](../../data_source/catalog/deltalake_catalog.md)を参照してください。

ほとんどの場合、`MetadataUpdateParams`を無視しても問題ありませんし、その中のポリシーパラメータを調整する必要もありません。なぜなら、これらのパラメータのデフォルト値は既に最適なパフォーマンスを提供しているからです。

しかし、Hive、Hudi、またはDelta Lakeでのデータ更新頻度が高い場合、これらのパラメータを調整することで、自動非同期更新のパフォーマンスをさらに最適化することができます。

| パラメータ                              | 必須 | 説明                                                  |
| -------------------------------------- | -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | No       | StarRocksがHive、Hudi、またはDelta Lakeテーブルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。値 `true` はキャッシュを有効にし、値 `false` はキャッシュを無効にします。 |
| enable_remote_file_cache               | No       | StarRocksがHive、Hudi、またはDelta Lakeテーブルまたはパーティションの基底データファイルのメタデータをキャッシュするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `true`。値 `true` はキャッシュを有効にし、値 `false` はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | No       | StarRocksが自身にキャッシュされたHive、Hudi、またはDelta Lakeテーブルまたはパーティションのメタデータを非同期で更新する時間間隔。単位: 秒。デフォルト値: `7200`（2時間）。 |
| remote_file_cache_refresh_interval_sec | No       | StarRocksが自身にキャッシュされたHive、Hudi、またはDelta Lakeテーブルまたはパーティションの基底データファイルのメタデータを非同期で更新する時間間隔。単位: 秒。デフォルト値: `60`。 |
| metastore_cache_ttl_sec                | No       | StarRocksが自身にキャッシュされたHive、Hudi、またはDelta Lakeテーブルまたはパーティションのメタデータを自動的に破棄する時間間隔。単位: 秒。デフォルト値: `86400`（24時間）。 |
| remote_file_cache_ttl_sec              | No       | StarRocksが自身にキャッシュされたHive、Hudi、またはDelta Lakeテーブルまたはパーティションの基底データファイルのメタデータを自動的に破棄する時間間隔。単位: 秒。デフォルト値: `129600`（36時間）。 |

### 例

以下の例では、使用するメタストアのタイプに応じて`unified_catalog_hms`または`unified_catalog_glue`という名前の統合カタログを作成し、統合データソースからデータをクエリします。

#### HDFS

HDFSをストレージとして使用する場合、以下のコマンドを実行します：

```SQL
CREATE EXTERNAL CATALOG unified_catalog_hms
PROPERTIES
(
    "type" = "unified",
    "unified.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

#### AWS S3

##### インスタンスプロファイルに基づく認証

- Hiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMRでAWS Glueを使用する場合、以下のコマンドを実行します：

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

##### 役割ベースの認証を想定

- Hiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMRでAWS Glueを使用する場合、以下のコマンドを実行します：

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

- Hiveメタストアを使用する場合、以下のコマンドを実行します：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMRでAWS Glueを使用する場合、以下のコマンドを実行します：

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

#### S3互換ストレージシステム

例としてMinIOを使用します。以下のコマンドを実行します。

```SQL
CREATE EXTERNAL CATALOG unified_catalog_hms
PROPERTIES
(
    "type" = "unified",
    "unified.metastore.type" = "hive",
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

- 共有キー認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SASトークン認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- マネージドサービスアイデンティティ認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- サービスプリンシパル認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- マネージドアイデンティティ認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 共有キー認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- サービスプリンシパル認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMベースの認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- サービスアカウントベースの認証方式を選択する場合、以下のコマンドを実行します。

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 代理ベースの認証方式を選択する場合:

  - VMインスタンスがサービスアカウントを代理する場合、以下のコマンドを実行します。

    ```SQL
    CREATE EXTERNAL CATALOG unified_catalog_hms
    PROPERTIES
    (
        "type" = "unified",
        "unified.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - サービスアカウントが別のサービスアカウントを代理する場合、以下のコマンドを実行します。

    ```SQL
    CREATE EXTERNAL CATALOG unified_catalog_hms
    PROPERTIES
    (
        "type" = "unified",
        "unified.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## 統合カタログの表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を使用して、現在のStarRocksクラスター内のすべてのカタログを照会できます。

```SQL
SHOW CATALOGS;
```

また、[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントを照会することもできます。次の例では、`unified_catalog_glue`という名前の統合カタログの作成ステートメントを照会します。

```SQL
SHOW CREATE CATALOG unified_catalog_glue;
```

## 統合カタログとその中のデータベースへの切り替え

次のいずれかの方法を使用して、統合カタログとその中のデータネースに切り替えることができます。

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して現在のセッションで統合カタログを指定し、次に[USE](../../sql-reference/sql-statements/data-definition/USE.md)を使用してアクティブなデータベースを指定します。

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替えます：
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定します：
  USE <db_name>
  ```

- [USE](../../sql-reference/sql-statements/data-definition/USE.md)を直接使用して、統合カタログとその中のデータベースに切り替えます。

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 統合カタログの削除

[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)を使用して、外部カタログを削除できます。

次の例では、`unified_catalog_glue`という名前の統合カタログを削除します。

```SQL
DROP CATALOG unified_catalog_glue;
```

## 統合カタログからのテーブルのスキーマの表示

以下のいずれかの構文を使用して、統合カタログからテーブルのスキーマを表示できます。

- スキーマの表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからのスキーマと場所の表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## 統合カタログからのデータのクエリ

統合カタログからデータをクエリするには、次の手順に従います。

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を使用して、統合カタログが関連付けられている統合データソース内のデータベースを表示します。

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [統合カタログとその中のデータベースに切り替えます](#switch-to-a-unified-catalog-and-a-database-in-it)。

3. [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、指定したデータベース内の目的のテーブルをクエリします。


   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hive、Iceberg、Hudi、または Delta Lake からデータを読み込む

[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用して、Hive、Iceberg、Hudi、または Delta Lake テーブルのデータを統合カタログ内に作成された StarRocks テーブルにロードできます。

次の例では、統合カタログ `unified_catalog` に属するデータベース `test_database` に作成された StarRocks テーブル `test_table` に Hive テーブル `hive_table` のデータをロードします：

```SQL
INSERT INTO unified_catalog.test_database.test_table SELECT * FROM hive_table
```

## 統合カタログにデータベースを作成する

StarRocks の内部カタログと同様に、統合カタログに対する [CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) 権限を持っている場合、[CREATE DATABASE](../../administration/privilege_item.md#catalog) ステートメントを使用してそのカタログにデータベースを作成できます。

> **注記**
>
> 権限の付与と取り消しは、[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して行うことができます。

StarRocks は統合カタログでの Hive および Iceberg データベースの作成のみをサポートしています。

[統合カタログとその中のデータベースに切り替え](#switch-to-a-unified-catalog-and-a-database-in-it)、次のステートメントを使用してそのカタログにデータベースを作成します：

```SQL
CREATE DATABASE <database_name>
[properties ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location` パラメータは、データベースを作成するファイルパスを指定します。これは HDFS またはクラウドストレージのいずれかになります。

- Hive メタストアをデータソースのメタストアとして使用する場合、`location` パラメータのデフォルト値は `<warehouse_location>/<database_name.db>` であり、データベース作成時にそのパラメータを指定しない場合、Hive メタストアでサポートされます。
- AWS Glue をデータソースのメタストアとして使用する場合、`location` パラメータにはデフォルト値がないため、データベース作成時にそのパラメータを指定する必要があります。

`prefix` は使用するストレージシステムに基づいて異なります：

| **ストレージシステム**                                         | **`Prefix` の値**                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                       | `hdfs`                                                       |
| Google GCS                                                 | `gs`                                                         |
| Azure Blob Storage                                         | <ul><li>ストレージアカウントが HTTP 経由のアクセスを許可している場合、`prefix` は `wasb` です。</li><li>ストレージアカウントが HTTPS 経由のアクセスを許可している場合、`prefix` は `wasbs` です。</li></ul> |
| Azure Data Lake Storage Gen1                               | `adl`                                                        |
| Azure Data Lake Storage Gen2                               | <ul><li>ストレージアカウントが HTTP 経由のアクセスを許可している場合、`prefix` は `abfs` です。</li><li>ストレージアカウントが HTTPS 経由のアクセスを許可している場合、`prefix` は `abfss` です。</li></ul> |
| AWS S3 またはその他の S3 互換ストレージ（例：MinIO） | `s3`                                                         |

## 統合カタログからデータベースを削除する

StarRocks の内部データベースと同様に、統合カタログ内で作成されたデータベースに対する [DROP](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md) 権限を持っている場合、[DROP DATABASE](../../administration/privilege_item.md#database) ステートメントを使用してそのデータベースを削除できます。空のデータベースのみ削除可能です。

> **注記**
>
> 権限の付与と取り消しは、[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して行うことができます。

StarRocks は統合カタログから Hive および Iceberg データベースのみを削除することをサポートしています。

統合カタログからデータベースを削除しても、HDFS クラスターやクラウドストレージ上のデータベースのファイルパスは削除されません。

[統合カタログに切り替え](#switch-to-a-unified-catalog-and-a-database-in-it)、次のステートメントを使用してそのカタログ内のデータベースを削除します：

```SQL
DROP DATABASE <database_name>
```

## 統合カタログにテーブルを作成する

StarRocks の内部データベースと同様に、統合カタログ内で作成されたデータベースに対する [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 権限を持っている場合、[CREATE TABLE](../../administration/privilege_item.md#database) または [CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) ステートメントを使用してそのデータベースにテーブルを作成できます。

> **注記**
>
> 権限の付与と取り消しは、[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して行うことができます。

StarRocks は統合カタログでの Hive および Iceberg テーブルの作成のみをサポートしています。

[Hive カタログとその中のデータベースに切り替え](#switch-to-a-unified-catalog-and-a-database-in-it)、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) を使用してそのデータベースに Hive または Iceberg テーブルを作成します：

```SQL
CREATE TABLE <table_name>
(column_definition1[, column_definition2, ...]
ENGINE = {hive|iceberg}
[partition_desc]
```

詳細については、「[Hive テーブルを作成する](../catalog/hive_catalog.md#create-a-hive-table)」および「[Iceberg テーブルを作成する](../catalog/iceberg_catalog.md#create-an-iceberg-table)」を参照してください。

次の例では、`hive_table` という名前の Hive テーブルを作成します。このテーブルは `action`、`id`、`dt` の 3 つの列で構成され、`id` と `dt` はパーティション列です：

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

## 統合カタログのテーブルにデータをシンクする

StarRocks の内部テーブルと同様に、統合カタログ内で作成されたテーブルに対する [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 権限を持っている場合、[INSERT](../../administration/privilege_item.md#table) ステートメントを使用して StarRocks テーブルのデータをその統合カタログテーブルにシンクできます（現在、Parquet 形式の統合カタログテーブルのみがサポートされています）。

> **注記**
>
> 権限の付与と取り消しは、[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して行うことができます。

StarRocks は統合カタログの Hive および Iceberg テーブルへのデータシンクのみをサポートしています。

[Hive カタログとその中のデータベースに切り替え](#switch-to-a-unified-catalog-and-a-database-in-it)、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用してそのデータベース内の Hive または Iceberg テーブルにデータを挿入します：

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータをシンクしたい場合は、以下の構文を使用します：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

詳細については、「[Hive テーブルにデータをシンクする](../catalog/hive_catalog.md#sink-data-to-a-hive-table)」および「[Iceberg テーブルにデータをシンクする](../catalog/iceberg_catalog.md#sink-data-to-an-iceberg-table)」を参照してください。

次の例では、`hive_table` という名前の Hive テーブルに 3 つのデータ行を挿入します：

```SQL
INSERT INTO hive_table
VALUES
    ("buy", 1, "2023-09-01"),
    ("sell", 2, "2023-09-02"),
    ("buy", 3, "2023-09-03");
```

## 統合カタログからテーブルを削除する

StarRocks の内部テーブルと同様に、統合カタログ内で作成されたテーブルに対する [DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) 権限を持っている場合、[DROP TABLE](../../administration/privilege_item.md#table) ステートメントを使用してそのテーブルを削除できます。

> **注記**
>

> 特権の付与と取り消しは、[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) と [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) を使用して行うことができます。

StarRocks は、統合カタログから Hive および Iceberg テーブルのみを削除することをサポートしています。

[Hive カタログとその中のデータベースに切り替える](#switch-to-a-unified-catalog-and-a-database-in-it)。その後、[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) を使用して、そのデータベース内の Hive または Iceberg テーブルを削除します：

```SQL
DROP TABLE <table_name>
```

詳細については、[Hive テーブルの削除](../catalog/hive_catalog.md#drop-a-hive-table) と [Iceberg テーブルの削除](../catalog/iceberg_catalog.md#drop-an-iceberg-table) を参照してください。

以下の例では、`hive_table` という名前の Hive テーブルを削除します：

```SQL
DROP TABLE hive_table FORCE
```

