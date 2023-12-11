---
displayed_sidebar: "Japanese"
---

# リポジトリの作成

## 説明

リモートストレージシステムにリポジトリを作成し、[データのバックアップとリストア](../../../administration/Backup_and_restore.md)用のデータスナップショットを保存するために使用します。

> **注意**
>
> 管理者権限を持つユーザーのみがリポジトリを作成できます。

リポジトリの削除の詳細な手順については、[DROP REPOSITORY](../data-definition/DROP_REPOSITORY.md)を参照してください。

## 構文

```SQL
CREATE [READ ONLY] REPOSITORY <リポジトリ名>
WITH BROKER
ON LOCATION "<リポジトリの場所>"
PROPERTIES ("key"="value", ...)
```

## パラメータ

| **パラメータ**       | **説明**                                              |
| ------------------- | ------------------------------------------------------------ |
| READ ONLY           | 読み取り専用のリポジトリを作成します。データのリストアは読み取り専用のリポジトリからのみ実行できます。データを移行するために同じリポジトリを2つのクラスターに作成する場合、新しいクラスター用に読み取り専用のリポジトリを作成し、RESTORE権限のみを付与できます。|
| リポジトリ名     | リポジトリ名。                                             |
| リポジトリの場所 | リモートストレージシステム内のリポジトリの場所。     |
| PROPERTIES          | リモートストレージシステムにアクセスするための資格情報方法。 |

**PROPERTIES**:

StarRocksは、HDFS、AWS S3、Google GCSにリポジトリを作成することをサポートしています。

- HDFSの場合：
  - "username": HDFSクラスターのNameNodeにアクセスするために使用するアカウントのユーザー名。
  - "password": HDFSクラスターのNameNodeにアクセスするために使用するアカウントのパスワード。

- AWS S3の場合：
  - "aws.s3.use_instance_profile": AWS S3へのアクセスの資格情報方法として、インスタンスプロファイルと仮定される役割を許可するかどうか。デフォルト： `false`。
  
    - AWS S3へのアクセスにIAMユーザーベースの資格情報（アクセスキーとシークレットキー）を使用する場合、このパラメータを指定する必要はなく、"aws.s3.access_key"、"aws.s3.secret_key"、"aws.s3.endpoint"を指定する必要があります。
    - AWS S3へのアクセスにインスタンスプロファイルを使用する場合、このパラメータを`true`に設定し、"aws.s3.region"を指定する必要があります。
    - AWS S3へのアクセスに仮定される役割を使用する場合、このパラメータを`true`に設定し、"aws.s3.iam_role_arn"と"aws.s3.region"を指定する必要があります。
  
  - "aws.s3.access_key": Amazon S3バケットにアクセスするために使用できるアクセスキーID。
  - "aws.s3.secret_key": Amazon S3バケットにアクセスするために使用できるシークレットアクセスキー。
  - "aws.s3.endpoint": Amazon S3バケットにアクセスするために使用できるエンドポイント。
  - "aws.s3.iam_role_arn": AWS S3バケット上で特権を持つIAMロールのARN。AWS S3へのアクセス方法として、仮定される役割を使用する場合は、このパラメータを指定する必要があります。その後、StarRocksはHiveカタログを使用してHiveデータを分析する際に、この役割を仮定します。
  - "aws.s3.region": AWS S3バケットが存在するリージョン。例: `us-west-1`。

> **注意**
>
> StarRocksは、S3Aプロトコルに従ってAWS S3にのみリポジトリの作成をサポートしています。したがって、AWS S3にリポジトリを作成する場合は、「ON LOCATION」で渡すS3 URI内の`s3://`を`s3a://`に置換する必要があります。

- Google GCSの場合：
  - "fs.s3a.access.key": Google GCSバケットにアクセスするために使用できるアクセスキー。
  - "fs.s3a.secret.key": Google GCSバケットにアクセスするために使用できるシークレットキー。
  - "fs.s3a.endpoint": Google GCSバケットにアクセスするために使用できるエンドポイント。

> **注意**
>
> StarRocksは、S3Aプロトコルに従ってGoogle GCSにのみリポジトリの作成をサポートしています。したがって、Google GCSにリポジトリを作成する場合は、「ON LOCATION」で渡すGCS URI内のプレフィックスを`s3a://`に置換する必要があります。

## 例

例1: Apache™ Hadoop®クラスター内の`hdfs_repo`という名前のリポジトリを作成します。

```SQL
CREATE REPOSITORY hdfs_repo
WITH BROKER
ON LOCATION "hdfs://x.x.x.x:yyyy/repo_dir/backup"
PROPERTIES(
    "username" = "xxxxxxxx",
    "password" = "yyyyyyyy"
);
```

例2: Amazon S3バケット`bucket_s3`内の読み取り専用リポジトリ`s3_repo`を作成します。

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