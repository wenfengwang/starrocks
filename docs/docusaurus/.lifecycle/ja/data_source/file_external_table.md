---
displayed_sidebar: "Japanese"
---

# 外部ファイルテーブル

外部ファイルテーブルは特別な外部テーブルの一種です。これにより、StarRocksにデータをロードせずに外部ストレージシステムに保存されているParquetやORC形式のデータファイルを直接クエリできます。さらに、外部ファイルテーブルはメタストアに依存しません。現在のバージョンでは、StarRocksは次の外部ストレージシステムをサポートしています：HDFS、Amazon S3、および他のS3互換のストレージシステム。

この機能はStarRocks v2.5からサポートされています。

## 制限

- 外部ファイルテーブルは[default_catalog](../data_source/catalog/default_catalog.md)内のデータベースに作成する必要があります。クラスタで作成されたカタログをクエリするには、[SHOW CATALOGS](../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を実行できます。
- Parquet、ORC、Avro、RCFile、およびSequenceFile形式のデータファイルのみがサポートされています。
- ファイル外部テーブルはターゲットのデータファイルのクエリにのみ使用できます。INSERT、DELETE、およびDROPなどのデータ書き込み操作はサポートされていません。

## 前提条件

外部ファイルテーブルを作成する前に、StarRocksクラスタを構成して、StarRocksがターゲットデータファイルが保存されている外部ストレージシステムにアクセスできるようにする必要があります。外部ファイルテーブルに必要な構成はHiveカタログと同じですが、メタストアを構成する必要はありません。構成に関する詳細については、[Hiveカタログ - 統合準備](../data_source/catalog/hive_catalog.md#integration-preparations)を参照してください。

## データベースの作成（オプション）

StarRocksクラスタに接続した後、既存のデータベースに外部ファイルテーブルを作成するか、新しいデータベースを作成して管理できます。クラスタ内の既存のデータベースをクエリするには、[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を実行します。その後、`USE <db_name>`を実行してターゲットのデータベースに切り替えることができます。

データベースを作成する構文は次のとおりです。

```SQL
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## 外部ファイルテーブルの作成

ターゲットのデータベースにアクセスした後、このデータベースに外部ファイルテーブルを作成できます。

### 構文

```SQL
CREATE EXTERNAL TABLE <table_name>
(
    <col_name> <col_type> [NULL | NOT NULL] [COMMENT "<comment>"]
) 
ENGINE=file
COMMENT ["comment"]
PROPERTIES
(
    FileLayoutParams,
    StorageCredentialParams
)
```

### パラメータ

| パラメータ        | 必須     | 説明                                                         |
| ---------------- | -------- | ------------------------------------------------------------ |
| table_name       | はい     | 外部ファイルテーブルの名前。次の命名規則に従います：<ul><li> 名前には文字、数字（0-9）、アンダースコア（_）を含めることができます。ただし、文字で始めなければなりません。</li><li> 名前は64文字を超えてはいけません。</li></ul> |
| col_name         | はい     | 外部ファイルテーブルの列名。外部ファイルテーブルの列名は、ターゲットのデータファイルの列名と同じである必要がありますが、大文字小文字は区別されません。外部ファイルテーブルの列の順序は、ターゲットのデータファイルの順序と異なる場合があります。 |
| col_type         | はい     | 外部ファイルテーブルの列のデータ型。このパラメータは、ターゲットのデータファイルの列のデータ型に基づいて指定する必要があります。詳細については、[列のデータ型のマッピング](#mapping-of-column-types)を参照してください。 |
| NULL \| NOT NULL | いいえ    | 外部ファイルテーブルの列がNULLであることを許可するかどうか。<ul><li>NULL：NULLを許可します。</li><li>NOT NULL：NULLは許可されません。</li></ul>次のルールに基づいてこの修飾子を指定する必要があります：<ul><li>ターゲットのデータファイルの列に対してこのパラメータが指定されていない場合、外部ファイルテーブルの列についても指定しないか、指定してもNULLを指定することができます。</li><li>ターゲットのデータファイルの列にNULLが指定されている場合、外部ファイルテーブルの列についてもこのパラメータを指定しないか、列に対してNULLを指定することができます。</li><li>ターゲットのデータファイルの列にNOT NULLが指定されている場合、外部ファイルテーブルの列にもNOT NULLを指定する必要があります。</li></ul> |
| comment          | いいえ    | 外部ファイルテーブルの列のコメント。                     |
| ENGINE          | はい     | エンジンのタイプ。値をfileに設定します。                   |
| comment          | いいえ    | 外部ファイルテーブルの説明。                             |
| PROPERTIES       | はい     | <ul><li>`FileLayoutParams`：ターゲットファイルのパスと形式を指定します。このプロパティは必須です。</li><li>`StorageCredentialParams`：オブジェクトストレージシステムへのアクセスに必要な認証情報を指定します。AWS S3および他のS3互換のストレージシステムについてのみ、このプロパティが必要です。</li></ul> |

#### FileLayoutParams

ターゲットデータファイルにアクセスするためのパラメータのセット。

```SQL
"path" = "<file_path>",
"format" = "<file_format>"
"enable_recursive_listing" = "{ true | false }"
```

| パラメータ        | 必須     | 説明                                                         |
| ---------------- | -------- | ------------------------------------------------------------ |
| path            | はい      | データファイルのパス。<ul><li>データファイルがHDFSに保存されている場合、パスの形式は`hdfs://<IP HDFSのアドレス>:<ポート>/<パス>`です。デフォルトのポート番号は8020です。デフォルトのポートを使用する場合、ポート番号を指定する必要はありません。</li><li>データファイルがAWS S3または他のS3互換のストレージシステムに保存されている場合、パスの形式は`s3://<バケット名>/<フォルダ>/`です。</li></ul>次のルールに注意してパスを入力してください：<ul><li>パス内のすべてのファイルにアクセスする場合は、パラメータをスラッシュ（`/`）で終了するようにします。例：`hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data/`。クエリを実行すると、StarRocksはパスの下にあるすべてのデータファイルをトラバースします。再帰を使用してデータファイルをトラバースしません。</li><li>単一のファイルにアクセスする場合は、このファイルを直接指すパスを入力します。例：`hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data`。クエリを実行すると、StarRocksはこのデータファイルのみをスキャンします。</li></ul> |
| format            | はい      | データファイルの形式。有効な値：`parquet`、`orc`、`avro`、`rctext`または`rcbinary`、`sequence`。 |
| enable_recursive_listing | いいえ | 現在のパスの下にあるすべてのファイルを再帰的にトラバースするかどうかを指定します。デフォルト値：`false`。 |

#### StorageCredentialParams（オプション）

ターゲットストレージシステムとの統合に関するパラメータのセット。このパラメータセットは**オプション**です。

ターゲットストレージシステムがAWS S3または他のS3互換のストレージである場合は、`StorageCredentialParams`を構成する必要があります。

その他のストレージシステムの場合、`StorageCredentialParams`は無視できます。

##### AWS S3

AWS S3に保存されているデータファイルにアクセスする必要がある場合は、`StorageCredentialParams`で次の認証パラメータを構成してください。

- インスタンスプロファイルベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成します：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- 仮定されたロールベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成します：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<あなたの仮定されたロールのARN>",
"aws.s3.region" = "<aws_s3_region>"
```

- IAMユーザーベースの認証メソッドを選択する場合は、`StorageCredentialParams`を次のように構成します：

```JavaScript
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

| パラメータ名        | 必須     | 説明                                                              |
| ------------------ | -------- |------------------------------------------------------------------- |
| aws.s3.use_instance_profile | はい | AWS S3にアクセスする際にインスタンスプロファイルベースの認証メソッドと仮定されたロールベースの認証メソッドを有効にするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。 |
| aws.s3.iam_role_arn | はい | AWS S3バケットに特権を持つIAMロールのARN。<br />AWS S3にアクセスするために仮定されたロールベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。StarRocksは、ターゲットのデータファイルにアクセスする際にこのロールを仮定します。 |
| aws.s3.region | はい | AWS S3バケットのリージョン。例：us-west-1。 |
| aws.s3.access_key | いいえ | IAMユーザーのアクセスキー。IAMユーザーベースの認証メソッドを使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key | いいえ | IAMユーザーのシークレットキー。IAMユーザーベースの認証メソッドを使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。|

AWS S3へのアクセス認証方法の選択と、AWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS S3へのアクセスの認証パラメータ](../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージ

MinIOなどのS3互換ストレージシステムへのアクセスが必要な場合、成功した統合を確実にするために、次のように`StorageCredentialParams`を構成します。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

次の表に、`StorageCredentialParams`で構成する必要があるパラメータを説明します。

| Parameter                       | Required | Description                                                  |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | Yes      | SSL接続を有効にするかどうかを指定します。 <br />有効な値: `true` と `false`。 デフォルト値: `true`。 |
| aws.s3.enable_path_style_access | Yes      | パス形式のアクセスを有効にするかどうかを指定します。<br />有効な値: `true` と `false`。デフォルト値: `false`。MinIOの場合、値を `true` に設定する必要があります。<br />パス形式のURLは次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、米国西部（オレゴン）リージョンで `DOC-EXAMPLE-BUCKET1` というバケットを作成し、そのバケットで `alice.jpg` オブジェクトにアクセスしたい場合、次のパス形式のURLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | Yes      | AWS S3の代わりにS3互換ストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key               | Yes      | IAMユーザーのアクセスキー。                         |
| aws.s3.secret_key               | Yes      | IAMユーザーのシークレットキー。                         |

#### カラムタイプのマッピング

次の表に、ターゲットデータファイルとファイル外部テーブル間のカラムタイプのマッピングを示します。

| Data file   | File external table                                          |
| ----------- | ------------------------------------------------------------ |
| INT/INTEGER | INT                                                          |
| BIGINT      | BIGINT                                                       |
| TIMESTAMP   | DATETIME. <br />TIMESTAMPは、現在のセッションのタイムゾーン設定に基づいてタイムゾーンを持たないDATETIMEに変換され、一部の精度が失われます。 |
| STRING      | STRING                                                       |
| VARCHAR     | VARCHAR                                                      |
| CHAR        | CHAR                                                         |
| DOUBLE      | DOUBLE                                                       |
| FLOAT       | FLOAT                                                        |
| DECIMAL     | DECIMAL                                                      |
| BOOLEAN     | BOOLEAN                                                      |
| ARRAY       | ARRAY                                                        |
| MAP         | MAP                                                          |
| STRUCT      | STRUCT                                                       |

### 例

#### HDFS

HDFSパスに保存されているParquetデータファイルをクエリするためのファイル外部テーブル`t0`を作成します。

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

Example 1: ファイル外部テーブルを作成し、**インスタンスプロファイル**を使用してAWS S3の**単一のParquetファイル**にアクセスします。

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

Example 2: ファイル外部テーブルを作成し、**アサムドロール**を使用してAWS S3のターゲットファイルパスの**すべてのORCファイル**にアクセスします。

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

Example 3: ファイル外部テーブルを作成し、**IAMユーザー**を使用してAWS S3のファイルパスの下にある**すべてのORCファイル**にアクセスします。

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
    "aws.s3.secret_key" = "<iam_user_access_key>",
    "aws.s3.region" = "us-west-2" 
);
```

## ファイル外部テーブルのクエリ

構文:

```SQL
SELECT <clause> FROM <file_external_table>
```

たとえば、[例 - HDFS](#examples)で作成したファイル外部テーブル`t0`からデータをクエリするには、次のコマンドを実行します。

```plain
SELECT * FROM t0;

+--------+------+
| name   | id   |
+--------+------+
| jack   |    2 |
| lily   |    1 |
+--------+------+
2 行が返されました (0.08 秒)
```

## ファイル外部テーブルの管理

[DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md)を使用してテーブルのスキーマを表示したり、[DROP TABLE](../sql-reference/sql-statements/data-definition/DROP_TABLE.md)を使用してテーブルを削除したりすることができます。