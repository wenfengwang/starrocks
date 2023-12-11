---
displayed_sidebar: "Japanese"
---

# ストレージボリュームの作成

## 説明

リモートストレージシステムのストレージボリュームを作成します。この機能はv3.1からサポートされています。

ストレージボリュームには、リモートデータストレージのプロパティや資格情報が含まれます。[共有データのStarRocksクラスタ](../../../deployment/shared_data/s3.md)でデータベースやクラウドネイティブテーブルを作成する際にストレージボリュームを参照できます。

> **注意**
>
> SYSTEMレベルでCREATE STORAGE VOLUME権限を持つユーザーのみがこの操作を実行できます。

## 構文

```SQL
CREATE STORAGE VOLUME [IF NOT EXISTS] <storage_volume_name>
TYPE = { S3 | AZBLOB }
LOCATIONS = ('<remote_storage_path>')
[ COMMENT '<comment_string>' ]
PROPERTIES
("key" = "value",...)
```

## パラメータ

| **パラメータ**       | **説明**                                              |
| ------------------- | ------------------------------------------------------------ |
| storage_volume_name | ストレージボリュームの名前。`builtin_storage_volume`という名前のストレージボリュームを作成できないことに注意してください。なぜなら、これはビルトインストレージボリュームを作成するために使用されているからです。 |
| TYPE                | リモートストレージシステムのタイプ。有効な値は`S3`および`AZBLOB`です。`S3`はAWS S3またはS3互換のストレージシステムを意味します。`AZBLOB`はAzure Blob Storageを意味します（v3.1.1以降でサポートされています）。 |
| LOCATIONS           | ストレージの場所。フォーマットは以下の通りです：<ul><li>AWS S3またはS3プロトコル互換のストレージシステムの場合：`s3://<s3_path>`。`<s3_path>`は絶対パスである必要があります。例えば、`s3://testbucket/subpath`。</li><li>Azure Blob Storageの場合：`azblob://<azblob_path>`。`<azblob_path>`は絶対パスである必要があります。例えば、`azblob://testcontainer/subpath`。</li></ul> |
| COMMENT             | ストレージボリュームについてのコメント。                           |
| PROPERTIES          | リモートストレージシステムにアクセスするためのプロパティや資格情報を指定するために使用される`"key" = "value"`ペアのパラメータ。詳細については[PROPERTIES](#properties)を参照してください。 |

### PROPERTIES

- AWS S3を使用する場合：

  - AWS SDKのデフォルト認証資格情報を使用してS3にアクセスする場合、以下のプロパティを設定します：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "true"
    ```

  - IAMユーザーベースの認証資格情報（アクセスキーおよびシークレットキー）を使用してS3にアクセスする場合、以下のプロパティを設定します：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<access_key>",
    "aws.s3.secret_key" = "<secrete_key>"
    ```

  - インスタンスプロファイルを使用してS3にアクセスする場合、以下のプロパティを設定します：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true"
    ```

  - アサムドロールを使用してS3にアクセスする場合、以下のプロパティを設定します：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "<role_arn>"
    ```

  - 外部のAWSアカウントからS3にアクセスする場合、以下のプロパティを設定します：

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "<role_arn>",
    "aws.s3.external_id" = "<external_id>"
    ```

- GCP Cloud Storageを使用する場合、以下のプロパティを設定します：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例：us-east-1
  "aws.s3.region" = "<region>",
  
  -- 例：https://storage.googleapis.com
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secrete_key>"
  ```

- MinIOを使用する場合、以下のプロパティを設定します：

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例：us-east-1
  "aws.s3.region" = "<region>",
  
  -- 例：http://172.26.xx.xxx:39000
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secrete_key>"
  ```

  | **プロパティ**                        | **説明**                                              |
  | ----------------------------------- | ------------------------------------------------------------ |
  | enabled                             | このストレージボリュームを有効にするかどうか。デフォルト：`false`。無効なストレージボリュームは参照できません。 |
  | aws.s3.region                       | S3バケットが存在する地域、例えば`us-west-2`。 |
  | aws.s3.endpoint                     | S3バケットにアクセスするためのエンドポイントURL、例えば`https://s3.us-west-2.amazonaws.com`。 |
  | aws.s3.use_aws_sdk_default_behavior | AWS SDKのデフォルト認証資格情報を使用するかどうか。有効な値は`true`および`false`（デフォルト）です。 |
  | aws.s3.use_instance_profile         | S3にアクセスするための認証方法としてインスタンスプロファイルおよびアサムドロールを使用するかどうか。有効な値は`true`および`false`（デフォルト）です。<ul><li>IAMユーザーベースの認証資格情報（アクセスキーおよびシークレットキー）を使用する場合、この項目を`false`として指定し、`aws.s3.access_key`および`aws.s3.secret_key`を指定する必要があります。</li><li>インスタンスプロファイルを使用してS3にアクセスする場合、この項目を`true`として指定する必要があります。</li><li>アサムドロールを使用してS3にアクセスする場合、この項目を`true`として指定し、`aws.s3.iam_role_arn`を指定する必要があります。</li><li>外部のAWSアカウントを使用する場合、この項目を`true`として指定し、`aws.s3.iam_role_arn`および`aws.s3.external_id`を指定する必要があります。</li></ul> |
  | aws.s3.access_key                   | S3バケットにアクセスするためのアクセスキーID。 |
  | aws.s3.secret_key                   | S3バケットにアクセスするためのシークレットアクセスキー。 |
  | aws.s3.iam_role_arn                 | データファイルが保存されているS3バケットで特権を持つIAMロールのARN。 |
  | aws.s3.external_id                  | S3バケットへのクロスアカウントアクセスに使用されるAWSアカウントの外部ID。 |

- Azure Blob Storage（v3.1.1以降でサポートされています）を使用する場合：

  - Shared Keyを使用してAzure Blob Storageにアクセスする場合、以下のプロパティを設定します：

    ```SQL
    "enabled" = "{ true | false }",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.shared_key" = "<shared_key>"
    ```

  - 共有アクセス署名（SAS）を使用してAzure Blob Storageにアクセスする場合、以下のプロパティを設定します：

    ```SQL
    "enabled" = "{ true | false }",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.sas_token" = "<sas_token>"
    ```

  > **注意**
  >
  > Azure Blob Storageアカウントを作成する際に階層ネームスペースを無効にする必要があります。

  | **プロパティ**          | **説明**                                              |
  | --------------------- | ------------------------------------------------------------ |
  | enabled               | このストレージボリュームを有効にするかどうか。デフォルト：`false`。無効なストレージボリュームは参照できません。 |
  | azure.blob.endpoint   | Azure Blob Storageアカウントのエンドポイント、例：`https://test.blob.core.windows.net`。 |
  | azure.blob.shared_key | Azure Blob Storageのリクエストを承認するために使用される共有キー。 |
  | azure.blob.sas_token  | Azure Blob Storegeのリクエストを承認するために使用される共有アクセス署名（SAS）。 |

## 例

例1：AWS S3バケット`defaultbucket`のためのストレージボリューム`my_s3_volume`を作成し、IAMユーザーベースの認証資格情報（アクセスキーおよびシークレットキー）を使用してS3にアクセスし、有効にします。

```SQL
CREATE STORAGE VOLUME my_s3_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "aws.s3.region" = "us-west-2",
    "aws.s3.endpoint" = "https://s3.us-west-2.amazonaws.com",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "xxxxxxxxxx",
    "aws.s3.secret_key" = "yyyyyyyyyy"
);
```

## 関連するSQLステートメント

- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)