---
displayed_sidebar: "Japanese"
---

# Hive カタログ

Hive カタログは、StarRocks が v2.4 以降でサポートされる外部カタログの一種です。Hive カタログでは、次のことができます。

- 手動でテーブルを作成する必要なしに、Hive に格納されたデータを直接クエリできます。
- [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用したり、非同期のマテリアライズド・ビュー (これは v2.5 以降でサポート) を使用して、Hive に格納されたデータを処理し、データを StarRocks にロードできます。
- StarRocks 上で、Hive データベースおよびテーブルを作成または削除したり、StarRocks テーブルから Parquet 形式の Hive テーブルにデータをシンクするために、[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用できます (この機能は v3.2 以降でサポート)。

Hive クラスタ上での SQL ワークロードの成功を確保するためには、StarRocks クラスタが次の 2 つの重要なコンポーネントと統合する必要があります。

- 分散ファイルシステム (HDFS) または AWS S3、Microsoft Azure Storage、Google GCS などのオブジェクトストレージ、または他の S3 互換ストレージシステム (たとえば MinIO) のようなオブジェクトストレージ

- Hive メタストアまたは AWS Glue のようなメタストア

  > **注意**
  >
  > ストレージとして AWS S3 を選択した場合、HMS または AWS Glue をメタストアとして使用できます。その他のストレージシステムを選択した場合は、HMS のみをメタストアとして使用できます。

## 使用上の注意

- StarRocks がサポートする Hive のファイル形式は、Parquet、ORC、CSV、Avro、RCFile、SequenceFile です。

  - Parquet ファイルは、次の圧縮形式をサポートしています: SNAPPY、LZ4、ZSTD、GZIP、および NO_COMPRESSION。v3.1.5 以降、Parquet ファイルは LZO 圧縮形式もサポートしています。
  - ORC ファイルは、次の圧縮形式をサポートしています: ZLIB、SNAPPY、LZO、LZ4、ZSTD、および NO_COMPRESSION。
  - CSV ファイルは、v3.1.5 以降、LZO 圧縮形式をサポートしています。

- StarRocks がサポートしない Hive のデータ型は INTERVAL、BINARY、UNION です。さらに、CSV 形式の Hive テーブルでは、MAP および STRUCT データ型もサポートされていません。
- Hive カタログを使用してデータをクエリすることはできますが、Hive カタログを使用して Hive クラスタにデータを削除、削除、または挿入することはできません。

## 統合の準備

Hive カタログを作成する前に、StarRocks クラスタが Hive クラスタのストレージシステムおよびメタストアと統合できることを確認してください。

### AWS IAM

Hive クラスタが AWS S3 をストレージとして使用する場合や AWS Glue をメタストアとして使用する場合は、適切な認証方法を選択し、StarRocks クラスタが関連する AWS クラウドリソースにアクセスできるように必要な準備を行ってください。

次の認証方法を推奨します。

- インスタンスプロファイル
- 仮定された役割
- IAM ユーザー

上記の 3 つの認証方法のうち、インスタンスプロファイルが最も広く使用されています。

詳細については、[AWS IAM での認証の準備](../../integrations/authenticate_to_aws_resources.md#preparations)を参照してください。

### HDFS

ストレージとして HDFS を選択した場合は、StarRocks クラスタを次のように構成してください。

- (オプション) HDFS クラスタにアクセスするために使用するユーザー名を設定します。デフォルトでは、StarRocks は FE および BE プロセスのユーザー名を使用して HDFS クラスタと Hive メタストアにアクセスします。各 FE の **fe/conf/hadoop_env.sh** ファイルと各 BE の **be/conf/hadoop_env.sh** ファイルの先頭に `export HADOOP_USER_NAME="<user_name>"` を追加してユーザー名を設定できます。これらのファイルにユーザー名を設定した後は、各 FE と各 BE を再起動してパラメータ設定を有効にします。StarRocks クラスタごとに 1 つのユーザー名のみを設定できます。
- Hive データをクエリする際に、StarRocks クラスタの FE と BE は HDFS クライアントを使用して HDFS クラスタにアクセスします。ほとんどの場合、この目的を達成するために StarRocks クラスタを構成する必要はありません。StarRocks はデフォルトの構成で HDFS クライアントを起動します。次の状況にのみ StarRocks クラスタを構成する必要があります。

  - HDFS クラスタで高可用性 (HA) が有効になっている場合: HDFS クラスタの **hdfs-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスと各 BE の **$BE_HOME/conf** パスに追加します。
  - HDFS クラスタで View File System (ViewFs) が有効になっている場合: HDFS クラスタの **core-site.xml** ファイルを各 FE の **$FE_HOME/conf** パスと各 BE の **$BE_HOME/conf** パスに追加します。

> **注意**
>
> クエリを送信した際に未知のホストを示すエラーが返された場合は、HDFS クラスタのホスト名と IP アドレスのマッピングを **/etc/hosts** パスに追加する必要があります。

### Kerberos 認証

HDFS クラスタまたは Hive メタストアで Kerberos 認証が有効になっている場合は、StarRocks クラスタを次のように構成してください。

- Key Distribution Center (KDC) から Ticket Granting Ticket (TGT) を取得するために、各 FE および各 BE で `kinit -kt keytab_path principal` コマンドを実行します。このコマンドを実行するには、HDFS クラスタと Hive メタストアにアクセスする権限が必要です。このコマンドで KDC にアクセスするのは時間的に敏感です。そのため、このコマンドを定期的に実行するために cron を使用する必要があります。
- 各 FE の **$FE_HOME/conf/fe.conf** ファイルと各 BE の **$BE_HOME/conf/be.conf** ファイルに `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` を追加します。ここでの例では、`/etc/krb5.conf` は **krb5.conf** ファイルの保存パスです。必要に応じてパスを変更できます。

## Hive カタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "hive",
    GeneralParams,
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### パラメータ

#### catalog_name

Hive カタログの名前。次の規則に従います。

- 名前には文字、数字 (0-9)、およびアンダーバー (_) を含めることができます。ただし、文字で始める必要があります。
- 名前は大文字と小文字を区別し、長さが 1023 文字を超えることはできません。

#### comment

Hive カタログの説明。このパラメータはオプションです。

#### type

データソースのタイプ。値を `hive` に設定します。

#### GeneralParams

一連の一般的なパラメータ。

`GeneralParams` で構成できるパラメータについては、次の表に記載されています。

| パラメータ                | 必須     | 説明                                                  |
| ------------------------ | -------- | ------------------------------------------------------------ |
| enable_recursive_listing | いいえ       | StarRocks がテーブルおよびそのパーティションからデータを読み取り、テーブルとそのパーティションの物理的な場所内のサブディレクトリからデータを再帰的にリストするかどうかを指定します。有効な値: `true` および `false`。デフォルト値: `false`。値 `true` はサブディレクトリを再帰的にリストし、値 `false` はサブディレクトリを無視することを意味します。 |

#### MetastoreParams

データソースのメタストアとの統合に関する一連のパラメータ。

##### Hive メタストア

データソースのメタストアとして Hive メタストアを選択した場合、`MetastoreParams` を次のように構成してください。

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注意**
>
> Hive データをクエリする前に、Hive メタストアノードのホスト名と IP アドレスのマッピングを **/etc/hosts** パスに追加する必要があります。そうしないと、クエリを開始したときに StarRocks が Hive メタストアにアクセスできなくなる場合があります。

`MetastoreParams` で構成する必要があるパラメータについては、次の表に記載されています。

| パラメータ           | 必須     | 説明                                                  |
| ------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type | はい       | Hive クラスタで使用するメタストアのタイプ。値を `hive` に設定します。 |
| hive.metastore.uris | はい       | Hive メタストアの URI。フォーマット: `thrift://<metastore_IP_address>:<metastore_port>`。<br />Hive メタストアで高可用性 (HA) が有効になっている場合は、複数のメタストア URI を指定し、カンマ (`,`) で区切ってで使用できます。たとえば、`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

データソースのメタストアとして AWS S3 を選択し、かつ AWS Glue を使用する場合は、次のいずれかのアクションを実行してください。

- インスタンスプロファイルベースの認証方法を選択する場合は、`MetastoreParams` を次のように構成してください。

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 想要选择假定基于角色的认证方法，请按以下步骤配置`MetastoreParams`：

```SQL
"hive.metastore.type" = "glue",
"aws.glue.use_instance_profile" = "true",
"aws.glue.iam_role_arn" = "<iam_role_arn>",
"aws.glue.region" = "<aws_glue_region>"
```

- 想要选择基于IAM用户的认证方法，请按以下步骤配置`MetastoreParams`：

```SQL
"hive.metastore.type" = "glue",
"aws.glue.use_instance_profile" = "false",
"aws.glue.access_key" = "<iam_user_access_key>",
"aws.glue.secret_key" = "<iam_user_secret_key>",
"aws.glue.region" = "<aws_s3_region>"
```

以下表格描述了您需要在`MetastoreParams`中配置的参数。

| 参数名                        | 必需    | 描述                                        |
| ----------------------------- | ------- | ------------------------------------------ |
| hive.metastore.type           | 是      | 您在Hive集群中使用的元数据存储类型。将值设置为`glue`。 |
| aws.glue.use_instance_profile | 是      | 指定是否启用基于实例配置文件的认证方法和基于假定角色的认证。有效值：`true`和`false`。默认值：`false`。 |
| aws.glue.iam_role_arn         | 否      | 在您的AWS Glue数据目录上具有特权的IAM角色的ARN。如果您使用假定基于角色的认证方法来访问AWS Glue，则必须指定此参数。 |
| aws.glue.region               | 是      | 您的AWS Glue数据目录所属的区域。示例：`us-west-1`。 |
| aws.glue.access_key           | 否      | 您的AWS IAM用户的访问密钥。如果您使用基于IAM用户的认证方法来访问AWS Glue，则必须指定此参数。 |
| aws.glue.secret_key           | 否      | 您的AWS IAM用户的密钥。如果您使用基于IAM用户的认证方法来访问AWS Glue，则必须指定此参数。 |

关于如何选择用于访问AWS Glue的认证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参阅[访问AWS Glue的认证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)。

#### StorageCredentialParams

关于StarRocks如何与您的存储系统集成的一组参数。此参数集是可选的。

如果您使用HDFS作为存储系统，则无需配置`StorageCredentialParams`。

如果您使用AWS S3、其他与S3兼容的存储系统、Microsoft Azure Storage或Google GCS作为存储系统，您必须配置`StorageCredentialParams`。

##### AWS S3

如果您将AWS S3作为Hive集群的存储，请执行以下操作之一：

- 如果选择基于实例配置文件的认证方法，请按以下步骤配置`StorageCredentialParams`：

```SQL
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- 如果选择假定基于角色的认证方法，请按以下步骤配置`StorageCredentialParams`：

```SQL
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<iam_role_arn>",
"aws.s3.region" = "<aws_s3_region>"
```

- 如果选择基于IAM用户的认证方法，请按以下步骤配置`StorageCredentialParams`：

```SQL
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数名                   | 必需    | 描述                                        |
| ----------------------- | ------- | ------------------------------------------ |
| aws.s3.use_instance_profile | 是      | 指定是否启用基于实例配置文件的认证方法和基于假定角色的认证方法。有效值：`true`和`false`。默认值：`false`。 |
| aws.s3.iam_role_arn         | 否      | 在您的AWS S3存储桶上具有特权的IAM角色的ARN。如果您使用假定基于角色的认证方法来访问AWS S3，则必须指定此参数。  |
| aws.s3.region               | 是      | 您的AWS S3存储桶所属的区域。示例：`us-west-1`。 |
| aws.s3.access_key           | 否      | 您的IAM用户的访问密钥。如果您使用基于IAM用户的认证方法来访问AWS S3，则必须指定此参数。 |
| aws.s3.secret_key           | 否      | 您的IAM用户的密钥。如果您使用基于IAM用户的认证方法来访问AWS S3，则必须指定此参数。 |

关于如何选择用于访问AWS S3的认证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参阅[访问AWS S3的认证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 与S3兼容的存储系统

从v2.5开始，Hive目录支持与S3兼容的存储系统。

如果您选择S3兼容存储系统（如MinIO）作为Hive集群的存储，请按以下步骤配置`StorageCredentialParams`，以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数名                   | 必需    | 描述                                        |
| ----------------------- | ------- | ------------------------------------------ |
| aws.s3.enable_ssl       | 是      | 指定是否启用SSL连接。<br />有效值：`true`和`false`。默认值：`true`。 |
| aws.s3.enable_path_style_access | 是      | 指定是否启用路径样式访问。<br />有效值：`true`和`false`。默认值：`false`。对于MinIO，您必须将值设置为`true`。<br />路径样式URL使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈州）区域创建了名为`DOC-EXAMPLE-BUCKET1`的存储桶，并且要访问该存储桶中的`alice.jpg`对象，您可以使用以下路径样式URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint         | 是      | 用于连接到您的S3兼容存储系统的端点。 |
| aws.s3.access_key       | 是      | 您的IAM用户的访问密钥。 |
| aws.s3.secret_key       | 是      | 您的IAM用户的密钥。 |

##### Microsoft Azure Storage

从v3.0开始，Hive目录支持Microsoft Azure Storage。

###### Azure Blob Storage

如果您选择Blob Storage为Hive集群的存储，请执行以下操作之一：

- 若要选择Shared Key认证方法，请按以下步骤配置`StorageCredentialParams`：

```SQL
"azure.blob.storage_account" = "<blob_storage_account_name>",
"azure.blob.shared_key" = "<blob_storage_account_shared_key>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数名**                   | **必需** | **描述**                                      |
| -------------------------- | -------- | -------------------------------------------- |
| azure.blob.storage_account | 是       | 您的Blob Storage账户的用户名。                    |
| azure.blob.shared_key      | 是       | 您的Blob Storage账户的共享密钥。                   |

- 若要选择SAS Token认证方法，请按以下步骤配置`StorageCredentialParams`：

```SQL
"azure.blob.account_name" = "<blob_storage_account_name>",
"azure.blob.container_name" = "<blob_container_name>",
"azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数名**              | **必需** | **描述**                                        |
| -------------------- | -------- | ---------------------------------------------- |
| azure.blob.account_name | 是       | 您的Blob Storage账户的用户名。                    |
| azure.blob.container_name | 是       | 存储您的数据的blob容器的名称。                      |
| azure.blob.sas_token   | 是       | 用于访问您的Blob Storage账户的SAS令牌。               |

###### Azure Data Lake Storage Gen1

如果您选择Data Lake Storage Gen1作为Hive集群的存储，请执行以下操作之一：

- 若要选择受管服务标识认证方法，请按以下步骤配置`StorageCredentialParams`：

```SQL
"azure.adls1.use_managed_service_identity" = "true"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。
  | **Parameter**                            | **Required** | **Description**                                              |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | Yes          | マネージド サービス ID 認証方法を有効にするかどうかを指定します。値を `true` に設定してください。 |

- サービス プリンシパル認証方法を選択する場合、次のように `StorageCredentialParams` を構成します。

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  次のテーブルには、`StorageCredentialParams` で構成する必要のあるパラメーターについて説明されています。

  | **Parameter**                 | **Required** | **Description**                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | Yes          | サービス プリンシパルのクライアント (アプリケーション) ID です。        |
  | azure.adls1.oauth2_credential | Yes          | 新しく作成されたクライアント (アプリケーション) シークレットの値です。    |
  | azure.adls1.oauth2_endpoint   | Yes          | サービス プリンシパルまたはアプリケーションの OAuth 2.0 トークン エンドポイント (v1) です。 |

###### Azure Data Lake Storage Gen2

Hive クラスターのストレージとして Data Lake Storage Gen2 を選択する場合は、次のいずれかのアクションを実行してください。

- マネージド ID 認証方法を選択する場合、次のように `StorageCredentialParams` を構成します。

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  次のテーブルには、`StorageCredentialParams` で構成する必要のあるパラメーターについて説明されています。

  | **Parameter**                           | **Required** | **Description**                                              |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | Yes          | マネージド ID 認証方法を有効にするかどうかを指定します。値を `true` に設定してください。 |
  | azure.adls2.oauth2_tenant_id            | Yes          | アクセスするデータが含まれるテナントの ID です。          |
  | azure.adls2.oauth2_client_id            | Yes          | マネージド ID のクライアント (アプリケーション) ID です。    |

- 共有キー認証方法を選択する場合、次のように `StorageCredentialParams` を構成します。

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  次のテーブルには、`StorageCredentialParams` で構成する必要のあるパラメーターについて説明されています。

  | **Parameter**               | **Required** | **Description**                                              |
  | --------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account | Yes          | Data Lake Storage Gen2 ストレージ アカウントのユーザー名です。 |
  | azure.adls2.shared_key      | Yes          | Data Lake Storage Gen2 ストレージ アカウントの共有キーです。 |

- サービス プリンシパル認証方法を選択する場合、次のように `StorageCredentialParams` を構成します。

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  次のテーブルには、`StorageCredentialParams` で構成する必要のあるパラメーターについて説明されています。

  | **Parameter**                      | **Required** | **Description**                                              |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | Yes          | サービス プリンシパルのクライアント (アプリケーション) ID です。        |
  | azure.adls2.oauth2_client_secret   | Yes          | 新しく作成されたクライアント (アプリケーション) シークレットの値です。    |
  | azure.adls2.oauth2_client_endpoint | Yes          | サービス プリンシパルまたはアプリケーションの OAuth 2.0 トークン エンドポイント (v1) です。 |

##### Google GCS

Hive カタログは、バージョン 3.0 以降で Google GCS をサポートしています。

Hive クラスターのストレージとして Google GCS を選択する場合は、次のいずれかのアクションを実行してください。

- VM ベースの認証方法を選択する場合、次のように `StorageCredentialParams` を構成します。

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  次のテーブルには、`StorageCredentialParams` で構成する必要のあるパラメーターについて説明されています。

  | **Parameter**                              | **Default value** | **Value** **example** | **Description**                                              |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接 Compute Engine にバインドされているサービス アカウントを使用するかどうかを指定します。 |

- サービス アカウント ベースの認証方法を選択する場合、次のように `StorageCredentialParams` を構成します。

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  次のテーブルには、`StorageCredentialParams` で構成する必要のあるパラメーターについて説明されています。

  | **Parameter**                          | **Default value** | **Value** **example**                                        | **Description**                                              |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービス アカウント作成時に生成された JSON ファイル内のメール アドレスです。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービス アカウント作成時に生成された JSON ファイル内のプライベート キー ID です。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービス アカウント作成時に生成された JSON ファイル内のプライベート キーです。 |

- 偽装ベースの認証方法を選択する場合、次のように `StorageCredentialParams` を構成します。

  - VM インスタンスをサービス アカウントになりすます:

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    次のテーブルには、`StorageCredentialParams` で構成する必要のあるパラメーターについて説明されています。

    | **Parameter**                              | **Default value** | **Value** **example** | **Description**                                              |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | 直接 Compute Engine にバインドされているサービス アカウントを使用するかどうかを指定します。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | 擬似化するサービス アカウントです。            |

  - 一時的な meta サービス アカウントが他のサービス アカウントをなりすます:

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    次のテーブルには、`StorageCredentialParams` で構成する必要のあるパラメーターについて説明されています。

    | **Parameter**                          | **Default value** | **Value** **example**                                        | **Description**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | サービス アカウント作成時に生成された meta サービス アカウントの JSON ファイル内のメール アドレスです。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | サービス アカウント作成時に生成された meta サービス アカウントの JSON ファイル内のプライベート キー ID です。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | サービス アカウント作成時に生成された meta サービス アカウントの JSON ファイル内のプライベート キーです。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | なりすますデータ サービス アカウントです。       |

#### MetadataUpdateParams

StarRocks が Hive のキャッシュされたメタデータを更新する方法に関する一連のパラメーターです。このパラメーター セットはオプションです。

StarRocks はデフォルトで [自動非同期更新ポリシー](#appendix-understand-metadata-automatic-asynchronous-update) を実装しています。

ほとんどの場合、`MetadataUpdateParams` を無視し、その中のポリシー パラメーターを調整する必要はありません。なぜなら、これらのパラメーターのデフォルト値は、すでに即席のパフォーマンスを提供しているからです。
しかし、Hiveでのデータ更新頻度が高い場合は、これらのパラメータを調整して、自動非同期更新のパフォーマンスをさらに最適化できます。

> **注意**
>
> ほとんどの場合、Hiveのデータが1時間以下の細かい粒度で更新される場合、データ更新頻度は高いと見なされます。

| パラメーター                              | 必須 | 説明                                                  |
|----------------------------------------| ------ | ------------------------------------------------------------ |
| enable_metastore_cache                 | いいえ       | StarRocksがHiveテーブルのメタデータをキャッシュするかどうかを指定します。有効な値：`true` および `false`。デフォルト値：`true`。 `true` はキャッシュを有効にし、 `false` はキャッシュを無効にします。 |
| enable_remote_file_cache               | いいえ       | StarRocksがHiveテーブルやパーティションの基本データファイルのメタデータをキャッシュするかどうかを指定します。有効な値：`true` および `false`。デフォルト値：`true`。 `true` はキャッシュを有効にし、 `false` はキャッシュを無効にします。 |
| metastore_cache_refresh_interval_sec   | いいえ       | StarRocksが自身でキャッシュされたHiveテーブルやパーティションのメタデータを非同期に更新する間隔。単位：秒。デフォルト値：`7200` 、つまり2時間。 |
| remote_file_cache_refresh_interval_sec | いいえ       | StarRocksが自身でキャッシュされたHiveテーブルやパーティションの基本データファイルのメタデータを非同期に更新する間隔。単位：秒。デフォルト値：`60`。 |
| metastore_cache_ttl_sec                | いいえ       | StarRocksが自身でキャッシュされたHiveテーブルやパーティションのメタデータを自動的に破棄する間隔。単位：秒。デフォルト値：`86400`、つまり24時間。 |
| remote_file_cache_ttl_sec              | いいえ       | StarRocksが自身でキャッシュされたHiveテーブルやパーティションの基本データファイルのメタデータを自動的に破棄する間隔。単位：秒。デフォルト値：`129600`、つまり36時間。 |
| enable_cache_list_names                | いいえ       | StarRocksがHiveのパーティション名をキャッシュするかどうかを指定します。有効な値：`true` および `false`。デフォルト値：`true`。 `true` はキャッシュを有効にし、 `false` はキャッシュを無効にします。 |

### 例

以下の例では、Hiveクラスターからデータをクエリするために、`hive_catalog_hms`または`hive_catalog_glue`というHiveカタログを作成します。作成するHiveメタストアのタイプに応じて異なります。

#### HDFS

ストレージとしてHDFSを使用する場合、次のようにコマンドを実行します:

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
);
```

#### AWS S3

##### インスタンスプロファイルベースの認証

- HiveクラスターでHiveメタストアを使用する場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 仮定されたロールベースの認証

- HiveクラスターでHiveメタストアを使用する場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### IAMユーザーベースの認証

- HiveクラスターでHiveメタストアを使用する場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- Amazon EMR HiveクラスターでAWS Glueを使用する場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
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

例としてMinIOを使用します。次のようにコマンドを実行します:

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
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

- 共有キー認証メソッドを選択した場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- SASトークン認証メソッドを選択した場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- 管理されたサービスアイデンティティ認証メソッドを選択した場合、次のようにコマンドを実行します:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- Service Principalの認証方法を選択した場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- Managed Identity認証方法を選択した場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- Shared Key認証方法を選択した場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- Service Principal認証方法を選択した場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- VMベースの認証方法を選択した場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- サービスアカウントベースの認証方法を選択した場合は、以下のようにコマンドを実行してください：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 偽装ベースの認証方法を選択した場合：

  - VMインスタンスがサービスアカウントを偽装する場合は、以下のようにコマンドを実行してください：

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - サービスアカウントが別のサービスアカウントを偽装する場合は、以下のようにコマンドを実行してください：

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## Hiveカタログを表示

[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) を使用して、現在のStarRocksクラスタ内のすべてのカタログをクエリできます：

```SQL
SHOW CATALOGS;
```

[Hiveカタログの作成ステートメント](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を使用して、外部カタログの作成ステートメントをクエリできます。以下の例では、`hive_catalog_glue` という名前のHiveカタログの作成ステートメントをクエリしています：

```SQL
SHOW CREATE CATALOG hive_catalog_glue;
```

## Hiveカタログとそのデータベースに切り替える

次の手法のいずれかを使用して、Hiveカタログとそのデータベースに切り替えることができます：

- [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) を使用して、現在のセッションでHiveカタログを指定し、次に [USE](../../sql-reference/sql-statements/data-definition/USE.md) を使用してアクティブなデータベースを指定します：

  ```SQL
  -- 現在のセッションで指定されたカタログに切り替える：
  SET CATALOG <catalog_name>
  -- 現在のセッションでアクティブなデータベースを指定する：
  USE <db_name>
  ```

- [USE](../../sql-reference/sql-statements/data-definition/USE.md) を直接使用して、Hiveカタログとそのデータベースに直接切り替えることができます：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## Hiveカタログを削除

[Hiveカタログ](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) を使用して、外部カタログを削除できます。

次の例では、`hive_catalog_glue` という名前のHiveカタログを削除しています：

```SQL
DROP Catalog hive_catalog_glue;
```

## Hiveテーブルのスキーマを表示

次の構文のいずれかを使用して、Hiveテーブルのスキーマを表示できます：

- スキーマを表示

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- CREATEステートメントからスキーマと場所を表示

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## Hiveテーブルをクエリ

1. [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を使用して、Hiveクラスタ内のデータベースを表示します：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [Hiveカタログとそのデータベースに切り替える](#hiveカタログとそのデータベースに切り替える)。

3. 指定したデータベース内のデスティネーションテーブルをクエリするために [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用します：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## Hiveからデータを読み込む

`olap_tbl` という名前のOLAPテーブルがある場合は、以下のようにデータを変換して読み込むことができます：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hive_table
```

## Hiveテーブルとビューの権限を付与

[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) ステートメントを使用して、Hiveカタログ内のすべてのテーブルまたはビューに特定のロールに権限を付与できます。

- ロールにHiveカタログ内のすべてのテーブルへのクエリ権限を付与する：

  ```SQL
  GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE <role_name>
  ```

- ロールにHiveカタログ内のすべてのビューへのクエリ権限を付与する：

  ```SQL
  GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE <role_name>
  ```

例えば、次のコマンドを使用して、`hive_catalog` という名前のHiveカタログの中で、ロール `hive_role_table` に対してHiveカタログ内のすべてのテーブルとビューへのクエリ権限を付与します：

```SQL
-- ロール `hive_role_table` を作成します。
CREATE ROLE hive_role_table;

-- Hiveカタログ `hive_catalog` に切り替えます。
SET CATALOG hive_catalog;

-- ロール `hive_role_table` に対してHiveカタログ `hive_catalog` 内のすべてのテーブルへのクエリ権限を付与します。
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE hive_role_table;
```
-- ユーザーロールhive_role_tableにHiveカタログhive_catalog内のすべてのビューをクエリする権限を付与します。
-- ロールhive_role_tableにビュー内のSELECT権限をすべてのデータベースに付与します。
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE hive_role_table;
```

## Hiveデータベースの作成

StarRocksの内部カタログと同様に、Hiveカタログで[CREATE DATABASE](../../administration/privilege_item.md#catalog)権限がある場合、[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)ステートメントを使用してそのHiveカタログ内にデータベースを作成できます。この機能はv3.2以上でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Hiveカタログに切り替える](#switch-to-a-hive-catalog-and-a-database-in-it)  、次のステートメントを使用して、そのカタログ内にHiveデータベースを作成します。

```SQL
CREATE DATABASE <データベース名>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location`パラメーターは、HDFSまたはクラウドストレージのいずれかにデータベースを作成するファイルパスを指定します。

- HiveクラスタのメタストアとしてHiveメタストアを使用する場合、`location`パラメーターのデフォルトは`<warehouse_location>/<database_name.db>`となります。これは、データベース作成時にそのパラメータを指定しない場合、Hiveメタストアでサポートされます。
- HiveクラスタのメタストアとしてAWS Glueを使用する場合、`location`パラメーターにはデフォルト値がないため、データベース作成時にそのパラメータを指定する必要があります。

`prefix`は、使用するストレージシステムによって異なります:

| **ストレージシステム**                                     | **`Prefix`の値**                                             |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                      | `hdfs`                                                       |
| Google GCS                                                | `gs`                                                         |
| Azure Blob Storage                                        | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可する場合、`prefix`は`wasb`です。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可する場合、`prefix`は`wasbs`です。</li></ul> |
| Azure Data Lake Storage Gen1                              | `adl`                                                        |
| Azure Data Lake Storage Gen2                              | <ul><li>ストレージアカウントがHTTP経由でアクセスを許可する場合、`prefix`は`abfs`です。</li><li>ストレージアカウントがHTTPS経由でアクセスを許可する場合、`prefix`は`abfss`です。</li></ul> |
| AWS S3またはその他のS3互換ストレージ（例：MinIO）       | `s3`                                                         |

## Hiveデータベースの削除

StarRocksの内部データベースと同様に、Hiveデータベースの[DROP](../../administration/privilege_item.md#database)権限がある場合、[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md)ステートメントを使用してそのHiveデータベースを削除できます。この機能はv3.2以上でサポートされており、空のデータベースのみを削除できます。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

Hiveデータベースを削除すると、そのデータベースのHDFSクラスタまたはクラウドストレージ上のファイルパスは削除されません。

[Hiveカタログに切り替える](#switch-to-a-hive-catalog-and-a-database-in-it)  、次のステートメントを使用して、そのカタログ内のHiveデータベースを削除します。

```SQL
DROP DATABASE <データベース名>
```

## Hiveテーブルの作成

StarRocksの内部データベースと同様に、Hiveデータベースで[CREATE TABLE](../../administration/privilege_item.md#database)権限がある場合、[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)ステートメントまたは[CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)ステートメントを使用してそのHiveデータベース内に管理テーブルを作成できます。この機能はv3.2以上でサポートされています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Hiveカタログとその中のデータベースに切り替える](#switch-to-a-hive-catalog-and-a-database-in-it)  、そのデータベース内にHive管理テーブルを作成するために次の構文を使用します。

### 構文

```SQL
CREATE TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...
partition_column_definition1,partition_column_definition2...])
[partition_desc]
[PROPERTIES ("key" = "value", ...)]
[AS SELECT query]
```

### パラメーター

#### column_definition

`column_definition`の構文は以下の通りです:

```SQL
col_name col_type [COMMENT 'comment']
```

以下の表はパラメーターを説明しています。

| パラメーター | 説明                         |
| ------------- | ---------------------------- |
| col_name      | 列の名前                     |
| col_type      | 列のデータ型                 |

#### partition_desc

`partition_desc`の構文は以下の通りです:

```SQL
PARTITION BY (par_col1[, par_col2...])
```

現在、StarRocksは一意なパーティション値ごとにパーティションを作成するアイデンティティ変換のみをサポートしています。

#### PROPERTIES

`プロパティ`内で`"key" = "value"`の形式でテーブルの属性を指定できます。

以下の表はいくつかの主要なプロパティを説明しています。

| **プロパティ**  | **説明**                                  |
| ---------------- | ----------------------------------------- |
| location         | 管理テーブルを作成するファイルパス       |
| file_format      | 管理テーブルのファイル形式                |
| compression_codec| 管理テーブルで使用される圧縮アルゴリズム |

### 例

1. `unpartition_tbl`という名前のパーティションされていないテーブルを作成します。次のように、2つの列`id`と`score`からなります。

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. `partition_tbl_1`という名前のパーティションされたテーブルを作成します。次のように、3つの列`action`、`id`、`dt`からなります。そのうち`id`と`dt`はパーティション列として定義され、次のようになります:

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. 既存の`partition_tbl_1`という名前のテーブルをクエリし、そのクエリ結果に基づいてパーティションされた`partition_tbl_2`という名前のテーブルを作成します。`partition_tbl_2`では、次のように`id`と`dt`がパーティション列として定義されています:

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (k1, k2)
   AS SELECT * from partition_tbl_1;
   ```

## Hiveテーブルへのデータのシンクロード
スターロックスの内部テーブルに似たように、Hiveテーブルに[INSERT](../../administration/privilege_item.md#table)権限がある場合（管理されたテーブルまたは外部テーブルであっても）[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ステートメントを使用して、スターロックステーブルのデータをHiveテーブルに流し込むことができます（現在はParquet形式のHiveテーブルのみサポートされています）。この機能はv3.2以降でサポートされています。外部テーブルにデータを流し込むことはデフォルトで無効になっています。外部テーブルにデータを流し込むためには、[システム変数`ENABLE_WRITE_HIVE_EXTERNAL_TABLE`](../../reference/System_variable.md)を`true`に設定する必要があります。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

[Hiveカタログおよびその中のデータベースに移動する](#switch-to-a-hive-catalog-and-a-database-in-it)にしてから、以下の構文を使用して、StarRocksテーブルのデータをそのデータベース内のParquet形式のHiveテーブルに流し込みます。

### 構文

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 特定のパーティションにデータを流し込む場合は、次の構文を使用してください:
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

> **注意**
>
> パーティションの列には`NULL`の値を指定できません。そのため、Hiveテーブルのパーティション列に空の値がロードされないようにしてください。

### パラメーター

| パラメーター | 説明                                                    |
| ----------- | ------------------------------------------------------- |
| INTO        | StarRocksテーブルのデータをHiveテーブルに追加します。    |
| OVERWRITE   | StarRocksテーブルのデータでHiveテーブルの既存データを上書きします。 |
| column_name | データをロードする宛先の列の名前。1つ以上の列を指定できます。複数の列を指定する場合は、カンマ(,)で区切ります。宛先の列には実際にHiveテーブルに存在する列のみを指定でき、指定した宛先の列はHiveテーブルのパーティション列を含める必要があります。指定した宛先の列は、宛先の列名に関係なく、StarRocksテーブルの列と1対1で結びつけられます。宛先の列が指定されていない場合、データはHiveテーブルのすべての列にロードされます。StarRocksテーブルのパーティション列をHiveテーブルの任意の列にマッピングできない場合、StarRocksはデフォルト値`NULL`をHiveテーブルの列に書き込みます。INSERTステートメントに、戻り値の列のタイプが宛先の列のデータタイプと異なるクエリステートメントが含まれている場合、StarRocksは不一致の列に対して暗黙の変換を行います。変換に失敗した場合は構文解析エラーが返されます。 |
| expression  | 宛先の列に値を割り当てる式。                               |
| DEFAULT     | 宛先の列にデフォルト値を割り当てます。                      |
| query       | StarRocksでサポートされる任意のSQLステートメントを含むクエリステートメントの結果をHiveテーブルにロードします。 |
| PARTITION   | データをロードするパーティション。このプロパティでHiveテーブルのすべてのパーティション列を指定する必要があります。このプロパティで指定するパーティション列は、テーブル作成ステートメントで定義したパーティション列の順序と異なっていても構いません。このプロパティを指定する場合、`column_name`プロパティを指定できません。 |

### 例

1. `partition_tbl_1`テーブルに3つのデータ行を挿入します：

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. 単純な計算を含むSELECTクエリの結果を`partition_tbl_1`テーブルに挿入します：

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. `partition_tbl_1`テーブルからデータを読み取るSELECTクエリの結果を、同じテーブルに挿入します：

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. 2つの条件`dt='2023-09-01'`および`id=1`を満たすパーティションにSELECTクエリの結果を挿入します。

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   または

   ```SQL
   INSERT INTO partition_tbl_2 partition(dt='2023-09-01',id=1) SELECT 'order';
   ```

5. 2つの条件`dt='2023-09-01'`および`id=1`を満たすパーティション内のすべての`action`列の値を`close`に上書きします：

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```
   
   または

   ```SQL
   INSERT OVERWRITE partition_tbl_1 partition(dt='2023-09-01',id=1) SELECT 'close';
   ```

## Hiveテーブルの削除

スターロックスの内部テーブルと同様に、Hiveテーブルに[DROP](../../administration/privilege_item.md#table)権限がある場合、[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)ステートメントを使用してそのHiveテーブルを削除できます。この機能はv3.1以降でサポートされています。現時点では、StarRocksはHiveの管理対象テーブルのみの削除をサポートしています。

> **注意**
>
> [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)を使用して権限を付与および取り消すことができます。

Hiveテーブルを削除する際には、`FORCE`キーワードを`DROP TABLE`ステートメントで指定する必要があります。操作が完了すると、テーブルのファイルパスは保持されますが、テーブルのデータはHDFSクラスターまたはクラウドストレージ上のテーブルとともにすべて削除されます。Hiveテーブルを削除する操作は慎重に行ってください。

[Hiveカタログおよびその中のデータベースに移動する](#switch-to-a-hive-catalog-and-a-database-in-it)にしてから、そのデータベース内のHiveテーブルを削除するために以下のステートメントを使用します。

```SQL
DROP TABLE <table_name> FORCE
```

## メタデータキャッシュの手動または自動的な更新

### 手動更新

デフォルトでは、StarRocksはHiveのメタデータをキャッシュし、さらなるパフォーマンス向上のためにメタデータを非同期モードで自動的に更新します。また、Hiveテーブルのいくつかのスキーマ変更やテーブルの更新が行われた後に、[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)を使用してメタデータを手動で更新し、StarRocksが最新のメタデータを取得し、適切な実行計画を生成できるようにしておくことができます。

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

以下の状況でメタデータを手動で更新する必要があります。

- 既存パーティション内のデータファイルが変更された場合、例えば`INSERT OVERWRITE ... PARTITION ...`コマンドを実行した場合
- Hiveテーブルでスキーマの変更が行われた場合
- `DROP`ステートメントを使用して既存のHiveテーブルが削除され、削除されたHiveテーブルと同じ名前の新しいHiveテーブルが作成された場合
- Hiveカタログの作成時に`PROPERTIES`で`"enable_cache_list_names" = "true"`が指定されており、Hiveクラスターで新しく作成したパーティションをクエリしたい場合

  > **注意**
  >
  > v2.5.5以降では、StarRocksでは定期的なHiveメタデータキャッシュの更新機能が提供されています。詳細については、以下のトピックの"[定期的なメタデータキャッシュの更新](#periodically-refresh-metadata-cache)"セクションを参照してください。この機能を有効にした後、StarRocksはデフォルトで10分ごとにHiveメタデータキャッシュを更新します。そのため、ほとんどの場合、手動での更新は不要です。新しいパーティションをHiveクラスターで作成した後すぐに新しいパーティションをクエリしたい場合にのみ手動で更新する必要があります。

なお、REFRESH EXTERNAL TABLEは、FEでキャッシュされているテーブルとパーティションのみを更新します。

### 自動的な増分更新

自動的な非同期更新方針とは異なり、自動的な増分更新方針により、StarRocksクラスターのFEはHiveメタストアから列の追加やパーティションの削除、データの更新などのイベントを読み取ります。これにより、StarRocksはこれらのイベントに基づいて自動的にFE内でキャッシュされたメタデータを更新することができます。つまり、Hiveテーブルのメタデータを手動で更新する必要はありません。

この機能はHMSに大きな負荷をかける可能性があります。この機能を使用する際は慎重にお願いします。[定期的なメタデータキャッシュの更新](#periodically-refresh-metadata-cache)を使用することをお勧めします。

自動的な増分更新を有効にするには、以下の手順に従います。
#### ステップ1: Hiveメタストアのイベントリスナーを構成する

Hiveメタストア v2.x および v3.x の両方がイベントリスナーを構成することをサポートしています。このステップでは、Hiveメタストアv3.1.2で使用されるイベントリスナー構成を例として使用します。以下の構成項目を **$HiveMetastore/conf/hive-site.xml** ファイルに追加し、その後Hiveメタストアを再起動します。

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

イベントリスナーが正常に構成されたかどうかを確認するために、FEログファイルで `event id` を検索できます。構成に失敗した場合、 `event id` の値は `0` です。

#### ステップ2: StarRocksで自動増分更新を有効にする

StarRocksクラスターの単一のHiveカタログまたはすべてのHiveカタログに対して、自動増分更新を有効にできます。

- 単一のHiveカタログに対して自動増分更新を有効にするには、Hiveカタログを作成する際に `PROPERTIES` で以下のように `enable_hms_events_incremental_sync` パラメータを `true` に設定します：

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.uris" = "thrift://102.168.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```

- すべてのHiveカタログに対して自動増分更新を有効にするには、各FEの `$FE_HOME/conf/fe.conf` ファイルに `"enable_hms_events_incremental_sync" = "true"` を追加し、その後各FEを再起動してパラメータ設定を有効にします。

また、ビジネス要件に応じて、各FEの `$FE_HOME/conf/fe.conf` ファイルで以下のパラメータを調整し、その後各FEを再起動してパラメータ設定を有効にします。

| パラメータ                         | 説明                                               |
| --------------------------------- | --------------------------------------------------- |
| hms_events_polling_interval_ms    | StarRocksがHiveメタストアからイベントを読み取る間隔。デフォルト値: `5000`。 単位: ミリ秒。          |
| hms_events_batch_size_per_rpc     | StarRocksが一度に読み取れるイベントの最大数。デフォルト値: `500`。              |
| enable_hms_parallel_process_evens | StarRocksがイベントを読み取りながら並列で処理するかを指定。有効な値: `true` および `false`。デフォルト値: `true`。 `true` は並列処理を有効にし、`false` は並列処理を無効にします。 |
| hms_process_events_parallel_num   | StarRocksが並列で処理できる最大数のイベント。デフォルト値: `4`。               |

## 定期的なメタデータキャッシュの更新

v2.5.5以降、StarRocksは頻繁にアクセスされるHiveカタログのキャッシュされたメタデータを定期的に更新できます。Hiveメタデータキャッシュの更新は、以下の[FEパラメータ](../../administration/Configuration.md#fe-configuration-items)を介して構成できます：

| 構成項目                                           | デフォルト値                          | 説明                                               |
| ------------------------------------------------- | ------------------------------------- | --------------------------------------------------- |
| enable_background_refresh_connector_metadata       | v3.0では `true`<br />v2.5では `false`  | 定期的なHiveメタデータキャッシュの更新を有効にするかどうか。 有効にすると、StarRocksはHiveクラスターのメタストア（HiveメタストアまたはAWS Glue）をポーリングし、頻繁にアクセスされるHiveカタログのキャッシュされたメタデータを更新してデータ変更を感知します。 `true` はHiveメタデータキャッシュの更新を有効にし、 `false` は無効にします。「FE動的パラメータ」です。[ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) コマンドを使用して変更できます。 |
| background_refresh_metadata_interval_millis        | `600000` (10分)                       | 2回の連続したHiveメタデータキャッシュの更新間隔。 単位: ミリ秒。        「FE動的パラメータ」です。[ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) コマンドを使用して変更できます。 |
| background_refresh_metadata_time_secs_since_last_access_secs | `86400` (24時間)                   | Hiveメタデータキャッシュの更新タスクの有効期限。アクセス済みのHiveカタログに対して、指定された時間より長いアクセスがない場合、StarRocksはそのキャッシュされたメタデータの更新を停止します。アクセスされていないHiveカタログに対しては、StarRocksはそのキャッシュされたメタデータを更新しません。単位: 秒。 「FE動的パラメータ」です。[ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) コマンドを使用して変更できます。 |

定期的なHiveメタデータキャッシュの更新機能とメタデータの自動非同期更新ポリシーを併用することで、データアクセスを大幅に高速化し、外部データソースからの読み込み負荷を軽減し、クエリのパフォーマンスを向上させることができます。

## 付録: メタデータの自動非同期更新の理解

自動非同期更新は、StarRocksがHiveカタログのメタデータを更新するためにデフォルトで使用するポリシーです。

デフォルトで（つまり、`enable_metastore_cache` と `enable_remote_file_cache` パラメータが両方とも `true` に設定されている場合）、クエリがHiveテーブルのパーティションにヒットすると、StarRocksはそのパーティションのメタデータとパーティションの基になるデータファイルのメタデータを自動的にキャッシュします。キャッシュされたメタデータは遅延更新ポリシーを使用して更新されます。

例えば、`table2` という名前のHiveテーブルがあり、そのテーブルには `p1`、 `p2`、 `p3`、および `p4` の4つのパーティションがあります。クエリが `p1` にヒットし、StarRocksは `p1` のメタデータと `p1` の基になるデータファイルのメタデータをキャッシュします。デフォルトのメタデータを更新・破棄する時間間隔は次のとおりとします：

- `p1` のキャッシュされたメタデータを非同期に更新する時間間隔（`metastore_cache_refresh_interval_sec` パラメータで指定）は2時間です。
- `p1` の基になるデータファイルのメタデータを非同期に更新する時間間隔（`remote_file_cache_refresh_interval_sec` パラメータで指定）は60秒です。
- `p1` のキャッシュされたメタデータを自動的に破棄する時間間隔（`metastore_cache_ttl_sec` パラメータで指定）は24時間です。
- `p1` の基になるデータファイルのメタデータを自動的に破棄する時間間隔（`remote_file_cache_ttl_sec` パラメータで指定）は36時間です。

以下の図は、理解を助けるためにタイムライン上の時間間隔を示しています。

![キャッシュされたメタデータの更新と破棄のタイムライン](../../assets/catalog_timeline.png)

それからStarRocksは以下のルールに基づいてメタデータを更新または破棄します：

- 他のクエリが再び `p1` にヒットし、最終更新からの現在の時間が60秒未満の場合、StarRocksは `p1` のキャッシュされたメタデータまたは `p1` の基になるデータファイルのメタデータを更新しません。
- 他のクエリが再び `p1` にヒットし、最終更新からの現在の時間が60秒を超える場合、StarRocksは `p1` の基になるデータファイルのメタデータを更新します。
- 他のクエリが再び `p1` にヒットし、最終更新からの現在の時間が2時間を超える場合、StarRocksは `p1` のキャッシュされたメタデータを更新します。
- 最終更新から24時間経過し、`p1` がアクセスされていない場合、StarRocksは `p1` のキャッシュされたメタデータを破棄します。そのメタデータは次回のクエリでキャッシュされます。
- 最終更新から36時間経過し、`p1` がアクセスされていない場合、StarRocksは `p1` の基になるデータファイルのメタデータを破棄します。そのメタデータは次回のクエリでキャッシュされます。