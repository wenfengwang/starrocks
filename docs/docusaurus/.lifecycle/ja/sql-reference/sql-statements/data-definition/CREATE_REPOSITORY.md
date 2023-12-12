---
displayed_sidebar: "Japanese"
---

# リポジトリの作成

## 説明

[データのバックアップと復元](../../../administration/Backup_and_restore.md)のために使用されるリモートストレージシステムにリポジトリを作成します。

> **注意**
>
> ADMIN権限を持つユーザーのみがリポジトリを作成できます。

リポジトリを削除する詳細な手順については、[DROP REPOSITORY](../data-definition/DROP_REPOSITORY.md)を参照してください。

## 構文

```SQL
CREATE [READ ONLY] REPOSITORY <repository_name>
WITH BROKER
ON LOCATION "<repository_location>"
PROPERTIES ("key"="value", ...)
```

## パラメータ

| **パラメータ**       | **説明**                                              |
| ------------------- | ------------------------------------------------------------ |
| READ ONLY           | 読み取り専用リポジトリを作成します。読み取り専用リポジトリからデータを復元することができます。新しいクラスタのために同じリポジトリを作成し、データを移行する際には、新しいクラスタ用に読み取り専用リポジトリを作成し、復元権限のみを付与できます。 |
| repository_name     | リポジトリ名。                                             |
| repository_location | リポジトリのリモートストレージシステム内の場所。     |
| PROPERTIES          |リモートストレージシステムにアクセスするための認証方法。|

**PROPERTIES**:

StarRocksは、HDFS、AWS S3、およびGoogle GCSのリポジトリの作成をサポートしています。

- HDFSの場合:
  - "username": HDFSクラスタのNameNodeにアクセスするために使用するアカウントのユーザー名。
  - "password": HDFSクラスタのNameNodeにアクセスするために使用するアカウントのパスワード。

- AWS S3の場合:
  - "aws.s3.use_instance_profile": AWS S3にアクセスするためのインスタンスプロファイルおよび前提となる役割を認証方法として許可するかどうか。デフォルト: `false`。

    - AWS S3にアクセスするためにIAMユーザーベースの認証（アクセスキーとシークレットキー）を使用する場合は、このパラメータを指定する必要はありません。その代わりに、"aws.s3.access_key"、"aws.s3.secret_key"、および"aws.s3.endpoint"を指定する必要があります。
    - AWS S3へのインスタンスプロファイルを使用する場合は、このパラメータを`true`に設定し、"aws.s3.region"を指定する必要があります。
    - AWS S3への前提となる役割を使用する場合は、このパラメータを`true`に設定し、"aws.s3.iam_role_arn"および"aws.s3.region"を指定する必要があります。

  - "aws.s3.access_key": Amazon S3バケットにアクセスするために使用できるアクセスキーID。
  - "aws.s3.secret_key": Amazon S3バケットにアクセスするために使用できるシークレットアクセスキー。
  - "aws.s3.endpoint": Amazon S3バケットにアクセスするために使用できるエンドポイント。
  - "aws.s3.iam_role_arn": AWS S3バケットに格納されているデータファイルに特権を持つIAMロールのARN。AWS S3にアクセスするために前提となる役割を使用する場合は、このパラメータを指定する必要があります。その後、StarRocksは、Hiveカタログを使用してHiveデータを分析する際に、このロールを前提とします。
  - "aws.s3.region": AWS S3バケットが存在するリージョン。例: `us-west-1`。

> **注意**
>
> StarRocksは、S3Aプロトコルに基づいてAWS S3にのみリポジトリの作成をサポートしています。したがって、AWS S3にリポジトリを作成する際には、`ON LOCATION`に渡すS3 URI内の`S3://`を`s3a://`に置換する必要があります。

- Google GCSの場合:
  - "fs.s3a.access.key": Google GCSバケットにアクセスするために使用できるアクセスキー。
  - "fs.s3a.secret.key": Google GCSバケットにアクセスするために使用できるシークレットキー。
  - "fs.s3a.endpoint": Google GCSバケットにアクセスするために使用できるエンドポイント。

> **注意**
>
> StarRocksは、S3Aプロトコルに基づいてGoogle GCSにのみリポジトリの作成をサポートしています。したがって、Google GCSにリポジトリを作成する際には、`ON LOCATION`に渡すGCS URI内のプレフィックスを`s3a://`に置換する必要があります。

## 例

例1: Apache™ Hadoop®クラスタ内の`hdfs_repo`という名前のリポジトリを作成します。

```SQL
CREATE REPOSITORY hdfs_repo
WITH BROKER
ON LOCATION "hdfs://x.x.x.x:yyyy/repo_dir/backup"
PROPERTIES(
    "username" = "xxxxxxxx",
    "password" = "yyyyyyyy"
);
```

例2: Amazon S3バケット`bucket_s3`内の読み取り専用リポジトリ`read_only`を作成します。

```SQL
CREATE READ ONLY REPOSITORY read_only
WITH BROKER
ON LOCATION "s3a://bucket_s3/backup"
PROPERTIES(
    "aws.s3.access_key" = "XXXXXXXXXXXXXXXXX",
    "aws.s3.secret_key" = "yyyyyyyyyyyyyyyyy",
    "aws.s3.endpoint" = "s3.us-east-1.amazonaws.com"
);
```

例3: Google GCSバケット`bucket_gcs`内の`gcs_repo`という名前のリポジトリを作成します。

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