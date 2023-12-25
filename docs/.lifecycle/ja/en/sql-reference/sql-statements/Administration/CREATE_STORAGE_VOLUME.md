---
displayed_sidebar: English
---

# ストレージボリュームの作成

## 説明

リモートストレージシステム用のストレージボリュームを作成します。この機能はv3.1からサポートされています。

ストレージボリュームは、リモートデータストレージのプロパティと資格情報から構成されます。[共有データStarRocksクラスタ](../../../deployment/shared_data/s3.md)でデータベースやクラウドネイティブテーブルを作成する際に、ストレージボリュームを参照できます。

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

## パラメーター

| **パラメーター**       | **説明**                                              |
| ------------------- | ------------------------------------------------------------ |
| storage_volume_name | ストレージボリュームの名前。`builtin_storage_volume`という名前のストレージボリュームは作成できません。これは組み込みストレージボリュームの作成に予約されています。|
| TYPE                | リモートストレージシステムのタイプ。有効な値: `S3` と `AZBLOB`。`S3`はAWS S3またはS3互換ストレージシステムを指します。`AZBLOB`はAzure Blob Storageを指します（v3.1.1以降でサポート）。 |
| LOCATIONS           | ストレージの場所。形式は以下の通りです：<ul><li>AWS S3またはS3プロトコル互換ストレージシステムの場合: `s3://<s3_path>`。`<s3_path>`は絶対パスである必要があります。例: `s3://testbucket/subpath`。</li><li>Azure Blob Storageの場合: `azblob://<azblob_path>`。`<azblob_path>`は絶対パスである必要があります。例: `azblob://testcontainer/subpath`。</li></ul> |
| COMMENT             | ストレージボリュームに関するコメント。                           |
| PROPERTIES          | `"key" = "value"`のペアでリモートストレージシステムへのアクセスに必要なプロパティと資格情報を指定するために使用されるパラメータ。詳細は[PROPERTIES](#properties)を参照してください。 |

### PROPERTIES

- AWS S3を使用する場合:

  - AWS SDKのデフォルト認証情報を使用してS3にアクセスする場合、以下のプロパティを設定します。

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "true"
    ```

  - IAMユーザーベースの認証情報（アクセスキーとシークレットキー）を使用してS3にアクセスする場合、以下のプロパティを設定します。

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<access_key>",
    "aws.s3.secret_key" = "<secret_key>"
    ```

  - インスタンスプロファイルを使用してS3にアクセスする場合、以下のプロパティを設定します。

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true"
    ```

  - Assume Roleを使用してS3にアクセスする場合、以下のプロパティを設定します。

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "<role_arn>"
    ```

  - 外部AWSアカウントからAssume Roleを使用してS3にアクセスする場合、以下のプロパティを設定します。

    ```SQL
    "enabled" = "{ true | false }",
    "aws.s3.region" = "<region>",
    "aws.s3.endpoint" = "<endpoint_url>",
    "aws.s3.use_aws_sdk_default_behavior" = "false",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "<role_arn>",
    "aws.s3.external_id" = "<external_id>"
    ```

- GCP Cloud Storageを使用する場合、以下のプロパティを設定します。

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例: us-east-1
  "aws.s3.region" = "<region>",
  
  -- 例: https://storage.googleapis.com
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secret_key>"
  ```

- MinIOを使用する場合、以下のプロパティを設定します。

  ```SQL
  "enabled" = "{ true | false }",
  
  -- 例: us-east-1
  "aws.s3.region" = "<region>",
  
  -- 例: http://172.26.xx.xxx:39000
  "aws.s3.endpoint" = "<endpoint_url>",
  
  "aws.s3.access_key" = "<access_key>",
  "aws.s3.secret_key" = "<secret_key>"
  ```

  | **プロパティ**                        | **説明**                                              |
  | ----------------------------------- | ------------------------------------------------------------ |
  | enabled                             | このストレージボリュームを有効にするかどうか。デフォルトは`false`です。無効なストレージボリュームは参照できません。 |
  | aws.s3.region                       | S3バケットが存在するリージョン。例: `us-west-2`。 |
  | aws.s3.endpoint                     | S3バケットへのアクセスに使用されるエンドポイントURL。例: `https://s3.us-west-2.amazonaws.com`。 |
  | aws.s3.use_aws_sdk_default_behavior | AWS SDKのデフォルト認証情報を使用するかどうか。有効な値は`true`または`false`（デフォルト）。 |
  | aws.s3.use_instance_profile         | S3へのアクセスにインスタンスプロファイルやAssume Roleを使用するかどうか。有効な値は`true`または`false`（デフォルト）。<ul><li>IAMユーザーベースの認証情報（アクセスキーとシークレットキー）を使用してS3にアクセスする場合は、この項目を`false`に設定し、`aws.s3.access_key`と`aws.s3.secret_key`を指定する必要があります。</li><li>インスタンスプロファイルを使用してS3にアクセスする場合は、この項目を`true`に設定する必要があります。</li><li>Assume Roleを使用してS3にアクセスする場合は、この項目を`true`に設定し、`aws.s3.iam_role_arn`を指定する必要があります。</li><li>外部AWSアカウントを使用する場合は、この項目を`true`に設定し、`aws.s3.iam_role_arn`と`aws.s3.external_id`を指定する必要があります。</li></ul> |
  | aws.s3.access_key                   | S3バケットへのアクセスに使用するアクセスキーID。             |
  | aws.s3.secret_key                   | S3バケットへのアクセスに使用するシークレットアクセスキー。         |
  | aws.s3.iam_role_arn                 | データファイルが保存されているS3バケットに権限を持つIAMロールのARN。 |
  | aws.s3.external_id                  | S3バケットへのクロスアカウントアクセスに使用されるAWSアカウントの外部ID。 |

- Azure Blob Storageを使用する場合（v3.1.1以降でサポート）:

  - 共有キーを使用してAzure Blob Storageにアクセスする場合、以下のプロパティを設定します。

    ```SQL
    "enabled" = "{ true | false }",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.shared_key" = "<shared_key>"
    ```

  - Shared Access Signature（SAS）を使用してAzure Blob Storageにアクセスする場合、以下のプロパティを設定します。

    ```SQL
    "enabled" = "{ true | false }",
    "azure.blob.endpoint" = "<endpoint_url>",
    "azure.blob.sas_token" = "<sas_token>"
    ```

  > **注意**
  >
  > Azure Blob Storageアカウントを作成する際には、階層型名前空間を無効にする必要があります。

  | **プロパティ**          | **説明**                                              |
  | --------------------- | ------------------------------------------------------------ |
  | enabled               | このストレージボリュームを有効にするかどうか。デフォルトは`false`です。無効なストレージボリュームは参照できません。 |
  | azure.blob.endpoint   | Azure Blob Storageアカウントのエンドポイント。例: `https://test.blob.core.windows.net`。 |
  | azure.blob.shared_key | Azure Blob Storageのリクエストを認証するために使用される共有キー。 |
  | azure.blob.sas_token  | Azure Blob Storageのリクエストを認証するために使用されるShared Access Signature（SAS）。 |

## 例

例1: AWS S3バケット`defaultbucket`用のストレージボリューム`my_s3_volume`を作成し、IAMユーザーベースの認証情報（アクセスキーとシークレットキー）を使用してS3にアクセスし、有効にします。

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

- [ストレージボリュームの変更](./ALTER_STORAGE_VOLUME.md)
- [ストレージボリュームの削除](./DROP_STORAGE_VOLUME.md)
- [デフォルトストレージボリュームの設定](./SET_DEFAULT_STORAGE_VOLUME.md)
- [ストレージボリュームの説明](./DESC_STORAGE_VOLUME.md)
- [ストレージボリュームの表示](./SHOW_STORAGE_VOLUMES.md)
