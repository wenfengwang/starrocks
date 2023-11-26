---
displayed_sidebar: "Japanese"
---

# リポジトリの作成

## 説明

[データのバックアップとリストア](../../../administration/Backup_and_restore.md)のために使用されるリモートストレージシステムにリポジトリを作成します。

> **注意**
>
> 管理者権限を持つユーザーのみがリポジトリを作成できます。

リポジトリの削除の詳細な手順については、[DROP REPOSITORY](../data-definition/DROP_REPOSITORY.md)を参照してください。

## 構文

```SQL
CREATE [READ ONLY] REPOSITORY <repository_name>
WITH BROKER
ON LOCATION "<repository_location>"
PROPERTIES ("key"="value", ...)
```

## パラメータ

| **パラメータ**        | **説明**                                                     |
| ------------------- | ------------------------------------------------------------ |
| READ ONLY           | 読み取り専用のリポジトリを作成します。読み取り専用のリポジトリからのみデータをリストアできることに注意してください。データを移行するために同じリポジトリを2つのクラスタに作成する場合、新しいクラスタ用に読み取り専用のリポジトリを作成し、リストア権限のみを付与します。|
| repository_name     | リポジトリの名前。                                             |
| repository_location | リモートストレージシステム内のリポジトリの場所。                   |
| PROPERTIES          | リモートストレージシステムにアクセスするための認証方法。           |

**PROPERTIES**:

StarRocksは、HDFS、AWS S3、およびGoogle GCSでリポジトリを作成することをサポートしています。

- HDFSの場合:
  - "username": HDFSクラスタのNameNodeにアクセスするために使用するアカウントのユーザー名。
  - "password": HDFSクラスタのNameNodeにアクセスするために使用するアカウントのパスワード。

- AWS S3の場合:
  - "aws.s3.use_instance_profile": AWS S3にアクセスするための認証方法として、インスタンスプロファイルと仮定されるロールを許可するかどうか。デフォルト: `false`。

    - AWS S3へのアクセスにIAMユーザーベースの認証（アクセスキーとシークレットキー）を使用する場合、このパラメータを指定する必要はありません。代わりに、"aws.s3.access_key"、"aws.s3.secret_key"、および"aws.s3.endpoint"を指定する必要があります。
    - AWS S3へのアクセスにインスタンスプロファイルを使用する場合、このパラメータを`true`に設定し、"aws.s3.region"を指定する必要があります。
    - AWS S3へのアクセスに仮定されるロールを使用する場合、このパラメータを`true`に設定し、"aws.s3.iam_role_arn"と"aws.s3.region"を指定する必要があります。

  - "aws.s3.access_key": Amazon S3バケットにアクセスするために使用できるアクセスキーID。
  - "aws.s3.secret_key": Amazon S3バケットにアクセスするために使用できるシークレットアクセスキー。
  - "aws.s3.endpoint": Amazon S3バケットにアクセスするために使用できるエンドポイント。
  - "aws.s3.iam_role_arn": データファイルが格納されているAWS S3バケットに特権を持つIAMロールのARN。AWS S3へのアクセスに仮定されるロールを使用する場合、このパラメータを指定する必要があります。その後、StarRocksはHiveカタログを使用してHiveデータを解析する際に、このロールを仮定します。
  - "aws.s3.region": AWS S3バケットが存在するリージョン。例: `us-west-1`。

> **注意**
>
> StarRocksは、S3Aプロトコルに基づいてAWS S3のみでリポジトリを作成することをサポートしています。したがって、AWS S3でリポジトリを作成する場合は、`ON LOCATION`でリポジトリの場所として渡すS3 URIの`s3://`を`s3a://`に置き換える必要があります。

- Google GCSの場合:
  - "fs.s3a.access.key": Google GCSバケットにアクセスするために使用できるアクセスキー。
  - "fs.s3a.secret.key": Google GCSバケットにアクセスするために使用できるシークレットキー。
  - "fs.s3a.endpoint": Google GCSバケットにアクセスするために使用できるエンドポイント。

> **注意**
>
> StarRocksは、S3Aプロトコルに基づいてGoogle GCSのみでリポジトリを作成することをサポートしています。したがって、Google GCSでリポジトリを作成する場合は、`ON LOCATION`でリポジトリの場所として渡すGCS URIの接頭辞を`s3a://`に置き換える必要があります。

## 例

例1: Apache™ Hadoop®クラスタに`hdfs_repo`という名前のリポジトリを作成します。

```SQL
CREATE REPOSITORY hdfs_repo
WITH BROKER
ON LOCATION "hdfs://x.x.x.x:yyyy/repo_dir/backup"
PROPERTIES(
    "username" = "xxxxxxxx",
    "password" = "yyyyyyyy"
);
```

例2: Amazon S3バケット`bucket_s3`に読み取り専用のリポジトリ` s3_repo`を作成します。

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

例3: Google GCSバケット`bucket_gcs`に`gcs_repo`という名前のリポジトリを作成します。

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
