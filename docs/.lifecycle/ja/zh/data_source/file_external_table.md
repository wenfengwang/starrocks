---
displayed_sidebar: Chinese
---

# ファイル外部テーブル

ファイル外部テーブル (File External Table) は特別な外部テーブルです。ファイル外部テーブルを通じて、外部ストレージシステム上の Parquet や ORC 形式のデータファイルを直接クエリし、データのインポートは不要です。また、ファイル外部テーブルは Metastore に依存しません。StarRocks が現在サポートしている外部ストレージシステムには、HDFS、Amazon S3、S3 プロトコル互換のオブジェクトストレージ、アリババクラウドの OSS、テンセントクラウドの COS が含まれます。

この機能は StarRocks 2.5 バージョンからサポートされています。

## 使用制限

- 現在、[default_catalog](../data_source/catalog/default_catalog.md) 内のデータベースでのみファイル外部テーブルを作成でき、external catalog はサポートされていません。クラスター内の catalog は [SHOW CATALOGS](../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) で確認できます。
- Parquet、ORC、Avro、RCFile、SequenceFile 形式のデータファイルのクエリのみをサポートしています。
- 現在、**読み取りのみ**をサポートしており、INSERT、DELETE、DROP などの**書き込み**操作はサポートしていません。

## 前提条件

ファイル外部テーブルを作成する前に、StarRocks で適切な設定を行い、クラスターがデータファイルがある外部ストレージシステムにアクセスできるようにする必要があります。具体的な設定手順は Hive catalog と同じです（Metastore の設定は不要です）。詳細は [Hive catalog - 準備作業](../data_source/catalog/hive_catalog.md#準備作業) を参照してください。

## データベースの作成（オプション）

StarRocks クラスターに接続した後、既存のデータベースにファイル外部テーブルを作成することも、新しいデータベースを作成してファイル外部テーブルを管理することもできます。クラスター内のデータベースは [SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) で確認し、`USE <db_name>` を実行して目的のデータベースに切り替えます。

データベースを作成する構文は以下の通りです。

```SQL
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## ファイル外部テーブルの作成

### 構文

現在のデータベースに切り替えた後、以下の構文を使用してファイル外部テーブルを作成できます。

```SQL
CREATE EXTERNAL TABLE <table_name> 
(
    <col_name> <col_type> [NULL | NOT NULL] [COMMENT "<comment>"]
) 
ENGINE=FILE
COMMENT ["comment"]
PROPERTIES
(
    FileLayoutParams,
    StorageCredentialParams
)
```

### パラメータ説明

| パラメータ       | 必須 | 説明                                                         |
| --------------- | ---- | ------------------------------------------------------------ |
| table_name      | はい | ファイル外部テーブルの名前。命名規則は以下の通りです：<ul><li> 英字 (a-z または A-Z)、数字 (0-9)、アンダースコア (_) で構成され、英字で始まる必要があります。</li><li> 全体の長さは 64 文字を超えてはいけません。</li></ul> |
| col_name        | はい | ファイル外部テーブルの列名。列名は大文字小文字を区別せず、データファイル内の名前と一致する必要があります。列の順序は一致する必要はありません。 |
| col_type        | はい | ファイル外部テーブルの列の型。[列型マッピング](#列型マッピング)に従って記入してください。 |
| NULL \| NOT NULL | いいえ | ファイル外部テーブルの列が NULL を許容するかどうか。<ul><li> NULL: NULL を許容する。</li><li> NOT NULL: NULL を許容しない。</li></ul> このパラメータは以下のルールに従って指定してください：<ul><li> データファイルの列でこのパラメータが指定されていない場合、ファイル外部テーブルの列は指定しなくてもよいか、NULL として指定できます。</li><li> データファイルの列が NULL として指定されている場合、ファイル外部テーブルの列は指定しなくてもよいか、NULL として指定できます。</li><li> データファイルの列が NOT NULL として指定されている場合、ファイル外部テーブルの列は NOT NULL として指定する必要があります。</li></ul> |
| comment         | いいえ | ファイル外部テーブルの列のコメント。                         |
| ENGINE          | はい | ENGINE のタイプ。`FILE` としてください。                     |
| comment         | いいえ | ファイル外部テーブルのコメント。                             |
| PROPERTIES      | はい | テーブルのプロパティ。<ul><li> `FileLayoutParams`: データファイルのパスと形式を指定するために必要です。</li><li> `StorageCredentialParams`: 外部ストレージシステムにアクセスするために必要な認証パラメータを設定します。AWS S3 または S3 プロトコル互換のオブジェクトストレージの場合のみ必要です。</li></ul> |

#### FileLayoutParams

```SQL
"path" = "<file_path>",
"format" = "<file_format>",
"enable_recursive_listing" = "{ true | false }"
```

| パラメータ                 | 必須 | 説明                                                         |
| ------------------------ | ---- | ------------------------------------------------------------ |
| path                     | はい | データファイルのパス。<ul><li> HDFS 上のファイルの場合、パスの形式は `hdfs://<HDFSのIPアドレス>:<ポート番号>/<パス>` です。ポート番号はデフォルトで 8020 なので、デフォルトを使用する場合はパスに指定する必要はありません。</li><li> Amazon S3 または S3 プロトコル互換のオブジェクトストレージ上のファイルの場合、パスの形式は `s3://<バケット名>/<フォルダ>/` です。</li></ul> パスを指定する際は、以下の点に注意してください：<ul><li> パス下のすべてのファイルを走査する場合は、パスの最後に '/' を付けてください。例：`hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data/`。クエリ時、StarRocks はそのパス下のすべてのファイルを走査しますが、再帰的には行いません。</li><li> 単一のファイルのみをクエリする場合は、パスを直接ファイル名に設定してください。例：`hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data`。クエリ時、StarRocks はそのファイルを直接スキャンします。</li></ul> |
| format                   | はい | データファイルの形式。`parquet`、`orc`、`avro`、`rctext`、`rcbinary`、`sequence` のいずれかを指定してください。 |
| enable_recursive_listing | いいえ | パス下のすべてのファイルを再帰的にクエリするかどうか。デフォルト値は `false` です。 |

#### `StorageCredentialParams`（オプション）

外部オブジェクトストレージにアクセスするために必要な認証パラメータを設定します。

データファイルが AWS S3 または S3 プロトコル互換のオブジェクトストレージに保存されている場合のみ必要です。

他のファイルストレージの場合は `StorageCredentialParams` を省略できます。

##### AWS S3

データファイルが AWS S3 に保存されている場合、`StorageCredentialParams` で以下の認証パラメータを設定する必要があります：

- Instance Profile を使用して認証および認可を行う場合

```SQL
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- Assumed Role を使用して認証および認可を行う場合

```SQL
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<iam_role_arn>",
"aws.s3.region" = "<aws_s3_region>"
```

- IAM User を使用して認証および認可を行う場合

```SQL
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

| パラメータ                        | 必須 | 説明                                                         |
| --------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい | Instance Profile または Assumed Role の認証方式を使用するかどうか。<br />`true` または `false` を指定します。デフォルト値は `false` です。 |

| aws.s3.iam_role_arn         | 否       | AWS S3 Bucket にアクセスする権限を持つ IAM Role の ARN。<br />Assume Role 認証方式で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。StarRocks がターゲットのデータファイルにアクセスする際に、この IAM Role を使用します。 |
| aws.s3.region               | はい       | AWS S3 Bucket のリージョン。例：us-west-1。                  |
| aws.s3.access_key           | 否       | IAM User の Access Key。<br />IAM User 認証方式で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | 否       | IAM User の Secret Key。<br />IAM User 認証方式で AWS S3 にアクセスする場合、このパラメータを指定する必要があります。 |

AWS S3 へのアクセスに使用する認証方式の選択方法や、AWS IAM コンソールでアクセス制御ポリシーを設定する方法については、[AWS S3 への認証パラメータ](../integrations/authenticate_to_aws_resources.md#AWS-S3-への認証パラメータ)を参照してください。

##### 阿里云 OSS

データファイルが阿里云 OSS に保存されている場合、`StorageCredentialParams` に以下の認証パラメータを設定する必要があります：

```SQL
"aliyun.oss.access_key" = "<user_access_key>",
"aliyun.oss.secret_key" = "<user_secret_key>",
"aliyun.oss.endpoint" = "<oss_endpoint>" 
```

| パラメータ                        | 必須 | 説明                                                         |
| ------------------------------- | ---- | ------------------------------------------------------------ |
| aliyun.oss.endpoint             | はい      | 阿里云 OSS のエンドポイント。例：`oss-cn-beijing.aliyuncs.com`。エンドポイントとリージョンの関係を調べるには、[アクセスドメインとデータセンター](https://help.aliyun.com/document_detail/31837.html)を参照してください。 |
| aliyun.oss.access_key           | はい      | 阿里云アカウントまたは RAM ユーザーの AccessKey ID を指定します。取得方法については、[AccessKey の取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。 |
| aliyun.oss.secret_key           | はい      | 阿里云アカウントまたは RAM ユーザーの AccessKey Secret を指定します。取得方法については、[AccessKey の取得](https://help.aliyun.com/document_detail/53045.html)を参照してください。 |

##### S3 プロトコル互換のオブジェクトストレージ

データファイルが S3 プロトコル互換のオブジェクトストレージ（例：MinIO）に保存されている場合、`StorageCredentialParams` に以下の認証パラメータを設定する必要があります：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

| パラメータ                        | 必須 | 説明                                                         |
| ------------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | はい      | SSL 接続を有効にするかどうか。<br />値の範囲：`true` または `false`。デフォルト値：`true`。 |
| aws.s3.enable_path_style_access | はい      | パススタイルアクセス（Path-Style Access）を有効にするかどうか。<br />値の範囲：`true` または `false`。デフォルト値：`false`。MinIO では `true` に設定する必要があります。<br />パススタイルの URL は以下の形式を使用します：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、米国西部（オレゴン）リージョンに `DOC-EXAMPLE-BUCKET1` という名前のバケットを作成し、そのバケット内の `alice.jpg` オブジェクトにアクセスしたい場合、以下のパススタイルの URL を使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | はい      | S3 プロトコル互換のオブジェクトストレージにアクセスするためのエンドポイント。 |

### 列タイプマッピング

ファイル外部テーブルを作成する際には、データファイルの列タイプに基づいてファイル外部テーブルの列タイプを指定する必要があります。具体的なマッピング関係は以下の通りです。

| データファイル  | ファイル外部テーブル                                         |
| --------- | ------------------------------------------------------------ |
| INT       | INT                                                          |
| BIGINT    | BIGINT                                                       |
| TIMESTAMP | DATETIME <br />注意：TIMESTAMP を DATETIME に変換すると精度が失われ、現在のセッション設定に基づいてタイムゾーンのない DATETIME に変換されます。 |
| STRING    | STRING                                                       |
| VARCHAR   | VARCHAR                                                      |
| CHAR      | CHAR                                                         |
| DOUBLE    | DOUBLE                                                       |
| FLOAT     | FLOAT                                                        |
| DECIMAL   | DECIMAL                                                      |
| BOOLEAN   | BOOLEAN                                                      |
| ARRAY     | ARRAY                                                        |

### 作成例

#### HDFS

ファイル外部テーブル `t0` を作成し、HDFS 上に保存されている Parquet データファイルにアクセスします。

```SQL
USE db_example;
CREATE EXTERNAL TABLE t0
(
    name string, 
    id int
) 
ENGINE=file
PROPERTIES 
(
    "path"="hdfs://x.x.x.x:8020/user/hive/warehouse/person_parq/", 
    "format"="parquet"
);
```

#### AWS S3

例 1：ファイル外部テーブル `table_1` を作成し、AWS S3 上の**単一の Parquet** データファイルにアクセスします。Instance Profile を使用して認証と認可を行います。

```SQL
USE db_example;
CREATE EXTERNAL TABLE table_1
(
    name string, 
    id int
) 
ENGINE=file
PROPERTIES 
(
    "path" = "s3://bucket-test/folder1/raw_0.parquet", 
    "format" = "parquet",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "us-west-2" 
);
```

例 2：ファイル外部テーブル `table_1` を作成し、AWS S3 上の特定のパスにある**すべての ORC データファイル**にアクセスします。Assume Role を使用して認証と認可を行います。

```SQL
USE db_example;
CREATE EXTERNAL TABLE table_1
(
    name string, 
    id int
) 
ENGINE=file
PROPERTIES 
(
    "path" = "s3://bucket-test/folder1/", 
    "format" = "orc",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "arn:aws:iam::51234343412:role/role_name_in_aws_iam",
    "aws.s3.region" = "us-west-2" 
);
```

例 3：ファイル外部テーブル `table_1` を作成し、AWS S3 上の特定のパスにある**すべての ORC データファイル**にアクセスします。IAM User を使用して認証と認可を行います。

```SQL
USE db_example;
CREATE EXTERNAL TABLE table_1
(
    name string, 
    id int
) 
ENGINE=file
PROPERTIES 
(
    "path" = "s3://bucket-test/folder1/", 
    "format" = "orc",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>",
    "aws.s3.region" = "us-west-2" 
);
```

## ファイル外部テーブルのクエリ

構文：

```sql
SELECT <clause> FROM <file_external_table>
```

例えば、HDFS の例で作成されたファイル外部テーブル `t0` をクエリするには、次のコマンドを実行します：

```plain
SELECT * FROM t0;

+--------+------+
| name   | id   |
+--------+------+
| jack   |    2 |
| lily   |    1 |
+--------+------+
2 rows in set (0.08 sec)
```

## ファイル外部テーブルの管理

ファイル外部テーブルの情報やテーブル構造を確認するには [DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md) を実行することができます。また、[DROP TABLE](../sql-reference/sql-statements/data-definition/DROP_TABLE.md) を使用してファイル外部テーブルを削除することもできます。
