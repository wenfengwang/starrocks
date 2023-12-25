---
displayed_sidebar: English
---

# リポジトリの作成

## 説明

リモートストレージシステムにリポジトリを作成し、[データのバックアップと復元](../../../administration/Backup_and_restore.md)用のデータスナップショットを保存します。

> **注意**
>
> ADMIN権限を持つユーザーのみがリポジトリを作成できます。

リポジトリの削除の詳細については、[DROP REPOSITORY](../data-definition/DROP_REPOSITORY.md)を参照してください。

## 構文

```SQL
CREATE [READ ONLY] REPOSITORY <repository_name>
WITH BROKER
ON LOCATION "<repository_location>"
PROPERTIES ("key"="value", ...)
```

## パラメーター

| **パラメーター**       | **説明**                                              |
| ------------------- | ------------------------------------------------------------ |
| READ ONLY           | 読み取り専用リポジトリを作成します。読み取り専用リポジトリからのみデータを復元できることに注意してください。データを移行するために2つのクラスターで同じリポジトリを作成する場合、新しいクラスターに読み取り専用リポジトリを作成し、RESTORE権限のみを付与することができます。|
| repository_name     | リポジトリ名。                                             |
| repository_location | リモートストレージシステム内のリポジトリの場所。     |
| PROPERTIES          |リモートストレージシステムへのアクセスに使用する認証方法。 |

**PROPERTIES**:

StarRocksは、HDFS、AWS S3、Google GCSでのリポジトリ作成をサポートしています。

- HDFSの場合:
  - "username": HDFSクラスターのNameNodeにアクセスするためのアカウントのユーザー名。
  - "password": HDFSクラスターのNameNodeにアクセスするためのアカウントのパスワード。

- AWS S3の場合:
  - "aws.s3.use_instance_profile": インスタンスプロファイルと仮定されたロールをAWS S3へのアクセスの認証方法として使用するかどうか。デフォルト: `false`。

    - IAMユーザーベースの認証情報（アクセスキーとシークレットキー）を使用してAWS S3にアクセスする場合、このパラメーターは指定する必要はありませんが、"aws.s3.access_key"、"aws.s3.secret_key"、および"aws.s3.endpoint"を指定する必要があります。
    - インスタンスプロファイルを使用してAWS S3にアクセスする場合、このパラメーターを`true`に設定し、"aws.s3.region"を指定する必要があります。
    - 仮定されたロールを使用してAWS S3にアクセスする場合、このパラメーターを`true`に設定し、"aws.s3.iam_role_arn"と"aws.s3.region"を指定する必要があります。
  
  - "aws.s3.access_key": Amazon S3バケットにアクセスするためのアクセスキーID。
  - "aws.s3.secret_key": Amazon S3バケットにアクセスするためのシークレットアクセスキー。
  - "aws.s3.endpoint": Amazon S3バケットにアクセスするためのエンドポイント。
  - "aws.s3.iam_role_arn": データファイルが保存されているAWS S3バケットに権限を持つIAMロールのARN。AWS S3にアクセスするための認証方法として仮定されたロールを使用する場合、このパラメーターを指定する必要があります。その後、StarRocksはHiveカタログを使用してHiveデータを分析する際にこのロールを仮定します。
  - "aws.s3.region": AWS S3バケットが存在するリージョン。例: `us-west-1`.

> **注記**
>
> StarRocksはS3Aプロトコルに従ってのみAWS S3でのリポジトリ作成をサポートしています。したがって、AWS S3でリポジトリを作成する場合、`ON LOCATION`でリポジトリの場所として渡すS3 URIの`s3://`を`s3a://`に置き換える必要があります。

- Google GCSの場合:
  - "fs.s3a.access.key": Google GCSバケットにアクセスするためのアクセスキー。
  - "fs.s3a.secret.key": Google GCSバケットにアクセスするためのシークレットキー。
  - "fs.s3a.endpoint": Google GCSバケットにアクセスするためのエンドポイント。

> **注記**
>
> StarRocksはS3Aプロトコルに従ってのみGoogle GCSでのリポジトリ作成をサポートしています。したがって、Google GCSでリポジトリを作成する場合、`ON LOCATION`でリポジトリの場所として渡すGCS URIのプレフィックスを`s3a://`に置き換える必要があります。

## 例

例1: Apache™ Hadoop®クラスターに`hdfs_repo`という名前のリポジトリを作成します。

```SQL
CREATE REPOSITORY hdfs_repo
WITH BROKER
ON LOCATION "hdfs://x.x.x.x:yyyy/repo_dir/backup"
PROPERTIES(
    "username" = "xxxxxxxx",
    "password" = "yyyyyyyy"
);
```

例2: Amazon S3バケット`bucket_s3`に`READ ONLY`リポジトリ`s3_repo`を作成します。

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
