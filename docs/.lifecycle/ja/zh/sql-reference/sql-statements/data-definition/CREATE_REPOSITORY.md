---
displayed_sidebar: Chinese
---

# リポジトリの作成

## 機能

リモートストレージシステムに基づいてデータスナップショットを保存するためのリポジトリを作成します。リポジトリは [バックアップと復元](../../../administration/Backup_and_restore.md) に使用されます。

> **注意**
>
> System レベルの REPOSITORY 権限を持つユーザーのみがリポジトリを作成できます。

リポジトリの削除については [DROP REPOSITORY](../data-definition/DROP_REPOSITORY.md) を参照してください。

## 文法

```SQL
CREATE [READ ONLY] REPOSITORY <repository_name>
WITH BROKER
ON LOCATION "<repository_location>"
PROPERTIES ("key"="value", ...)
```

## パラメータ説明

| **パラメータ**      | **説明**                                                     |
| ------------------- | ------------------------------------------------------------ |
| READ ONLY           | 読み取り専用のリポジトリを作成します。読み取り専用リポジトリは復元操作のみ可能です。データを移行するために2つのクラスターに同じリポジトリを作成する場合、新しいクラスターに読み取り専用リポジトリを作成し、復元権限のみを付与することができます。|
| repository_name     | リポジトリ名。                                               |
| repository_location | リモートストレージシステムのパス。                           |
| PROPERTIES          | リモートストレージシステムにアクセスするためのキーと値、またはユーザー名とパスワード。具体的な使用方法は以下の説明を参照してください。 |

**PROPERTIES**：

StarRocks は、HDFS、AWS S3、Google GCS、アリババクラウド OSS、およびテンセントクラウド COS でリポジトリを作成することをサポートしています。

- HDFS：
  - "username"：HDFS クラスタの NameNode にアクセスするためのユーザー名。
  - "password"：HDFS クラスタの NameNode にアクセスするためのパスワード。

- S3：
  - "aws.s3.use_instance_profile"：Instance Profile または Assumed Role を使用して AWS S3 にアクセスするかどうか。デフォルト値：`false`。

    - IAM ユーザーの資格情報（Access Key と Secret Key）を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要はなく、"aws.s3.access_key"、"aws.s3.secret_key"、および "aws.s3.endpoint" を指定します。
    - Instance Profile を使用して AWS S3 にアクセスする場合、このパラメータを `true` に設定し、"aws.s3.region" を指定します。
    - Assumed Role を使用して AWS S3 にアクセスする場合、このパラメータを `true` に設定し、"aws.s3.iam_role_arn" と "aws.s3.region" を指定します。

  - "aws.s3.access_key"：AWS S3 ストレージスペースにアクセスするための Access Key。
  - "aws.s3.secret_key"：AWS S3 ストレージスペースにアクセスするための Secret Key。
  - "aws.s3.endpoint"：AWS S3 ストレージスペースにアクセスするためのエンドポイント。
  - "aws.s3.iam_role_arn"：AWS S3 ストレージスペースへのアクセス権を持つ IAM Role の ARN。Instance Profile または Assumed Role を使用して AWS S3 にアクセスする場合、このパラメータを指定する必要があります。
  - "aws.s3.region"：アクセスする AWS S3 ストレージスペースのリージョン、例えば `us-west-1`。

> **説明**
>
> StarRocks は AWS S3 でリポジトリを作成する際に S3A プロトコルのみをサポートしています。そのため、AWS S3 でリポジトリを作成する際には、`ON LOCATION` パラメータで S3 URI の `s3://` を `s3a://` に置き換える必要があります。

- GCS：
  - "fs.s3a.access.key"：Google GCS ストレージスペースにアクセスするための Access Key。
  - "fs.s3a.secret.key"：Google GCS ストレージスペースにアクセスするための Secret Key。
  - "fs.s3a.endpoint"：Google GCS ストレージスペースにアクセスするためのエンドポイント。

> **説明**
>
> StarRocks は Google GCS でリポジトリを作成する際に S3A プロトコルのみをサポートしています。そのため、Google GCS でリポジトリを作成する際には、`ON LOCATION` パラメータで GCS URI のプレフィックスを `s3a://` に置き換える必要があります。

- OSS：
  - "fs.oss.accessKeyId"：アリババクラウド OSS ストレージスペースにアクセスするための AccessKey ID。
  - "fs.oss.accessKeySecret"：アリババクラウド OSS ストレージスペースにアクセスするための AccessKey Secret。
  - "fs.oss.endpoint"：アリババクラウド OSS ストレージスペースにアクセスするためのエンドポイント。

- COS：
  - "fs.cosn.userinfo.secretId"：テンセントクラウド COS ストレージスペースにアクセスするための SecretId。
  - "fs.cosn.userinfo.secretKey"：テンセントクラウド COS ストレージスペースにアクセスするための SecretKey。
  - "fs.cosn.bucket.endpoint_suffix"：テンセントクラウド COS ストレージスペースにアクセスするためのエンドポイント。

## 例

例1：Apache™ Hadoop® クラスタに `hdfs_repo` という名前のリポジトリを作成します。

```SQL
CREATE REPOSITORY hdfs_repo
WITH BROKER
ON LOCATION "hdfs://x.x.x.x:yyyy/repo_dir/backup"
PROPERTIES(
    "username" = "xxxxxxxx",
    "password" = "yyyyyyyy"
);
```

例2：Amazon S3 のバケット `bucket_s3` に `s3_repo` という名前の読み取り専用リポジトリを作成します。

```SQL
CREATE READ ONLY REPOSITORY s3_repo
WITH BROKER
ON LOCATION "s3a://bucket_s3/backup"
PROPERTIES(
    "aws.s3.access_key" = "XXXXXXXXXXXXXXXXX",
    "aws.s3.secret_key" = "yyyyyyyyyyyyyyyyy",
    "aws.s3.endpoint" = "s3.us-east-1.amazonaws.com"
);
```

例3：Google GCS のバケット `bucket_gcs` に `gcs_repo` という名前のリポジトリを作成します。

```SQL
CREATE REPOSITORY gcs_repo
WITH BROKER
ON LOCATION "s3a://bucket_gcs/backup"
PROPERTIES(
    "fs.s3a.access.key" = "xxxxxxxxxxxxxxxxxxxx",
    "fs.s3a.secret.key" = "yyyyyyyyyyyyyyyyyyyy",
    "fs.s3a.endpoint" = "storage.googleapis.com"
);
```
