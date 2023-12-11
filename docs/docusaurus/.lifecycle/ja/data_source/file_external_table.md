---
displayed_sidebar: "Japanese"
---

# ファイル外部テーブル

ファイル外部テーブルは特別な外部テーブルの一種です。これを使用すると、StarRocksにデータをロードせずに、外部ストレージシステムに保存されているParquetおよびORC形式のデータファイルを直接クエリできます。また、ファイル外部テーブルはメタストアに依存しません。現在のバージョンでは、StarRocksは次の外部ストレージシステムをサポートしています：HDFS、Amazon S3、およびその他のS3互換のストレージシステム。

この機能はStarRocks v2.5からサポートされています。

## 制限

- ファイル外部テーブルは、[default_catalog](../data_source/catalog/default_catalog.md)内のデータベースに作成する必要があります。クラスタで作成されたカタログをクエリするには、[SHOW CATALOGS](../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を実行できます。
- Parquet、ORC、Avro、RCFile、およびSequenceFileのデータファイルのみがサポートされています。
- ファイル外部テーブルは、対象のデータファイルのクエリにのみ使用できます。INSERT、DELETE、およびDROPなどのデータ書き込み操作はサポートされていません。

## 前提条件

ファイル外部テーブルを作成する前に、StarRocksクラスタを構成して、StarRocksが対象のデータファイルが格納されている外部ストレージシステムにアクセスできるようにする必要があります。ファイル外部テーブルに必要な構成は、Hiveカタログに必要な構成と同じですが、メタストアを構成する必要はありません。詳細については、[Hiveカタログ−統合の準備](../data_source/catalog/hive_catalog.md#integration-preparations)を参照してください。

## データベースの作成（オプション）

StarRocksクラスタに接続した後、既存のデータベースにファイル外部テーブルを作成するか、新しいデータベースを作成して、ファイル外部テーブルを管理できます。クラスタ内の既存のデータベースをクエリするには、[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を実行できます。その後、`USE <db_name>`を実行して、対象のデータベースに切り替えることができます。

データベースを作成する構文は次のとおりです。

```SQL
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## ファイル外部テーブルの作成

対象のデータベースにアクセスした後、そのデータベース内にファイル外部テーブルを作成できます。

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

| パラメータ        | 必須     | 説明                                                        |
| ---------------- | -------- | ------------------------------------------------------------ |
| table_name       | はい     | ファイル外部テーブルの名前。次の命名規則に従います：<ul><li>名前には、英字、数字（0-9）、アンダースコア（_）を含めることができます。英字で始まらなければなりません。</li><li>名前は64文字を超えることはできません。</li></ul> |
| col_name         | はい     | ファイル外部テーブルの列名。ファイル外部テーブルの列名は、対象のデータファイルの列名と同じである必要がありますが、大文字小文字は区別されません。ファイル外部テーブルの列の順序は、対象のデータファイルの順序と異なる場合があります。 |
| col_type         | はい     | ファイル外部テーブルの列のタイプ。このパラメータは、対象のデータファイルの列のタイプに基づいて指定する必要があります。詳細については、「[列のタイプのマッピング](#mapping-of-column-types)」を参照してください。 |
| NULL \| NOT NULL | いいえ   | ファイル外部テーブルの列がNULLであることを許可するかどうか。<ul><li>NULL：NULLが許可されます。</li><li>NOT NULL：NULLが許可されません。</li></ul>次の規則に基づいてこの変更子を指定する必要があります：<ul><li>対象のデータファイルの列に対してこのパラメータが指定されていない場合、ファイル外部テーブルの列にも指定しないか、ファイル外部テーブルの列にNULLを指定しないでください。</li><li>対象のデータファイルの列にNULLが指定されている場合、ファイル外部テーブルの列にもこのパラメータを指定しないか、ファイル外部テーブルの列にNULLを指定しないでください。</li><li>対象のデータファイルの列にNOT NULLが指定されている場合は、ファイル外部テーブルの列にもNOT NULLを指定する必要があります。</li></ul> |
| comment          | いいえ   | ファイル外部テーブルの列のコメント。                        |
| ENGINE           | はい     | エンジンの種類。値をfileに設定します。                      |
| comment          | いいえ   | ファイル外部テーブルの説明。                                 |
| PROPERTIES       | はい     | <ul><li>`FileLayoutParams`: 対象ファイルのパスとフォーマットを指定します。このプロパティは必須です。</li><li>`StorageCredentialParams`：オブジェクトストレージシステムにアクセスするために必要な認証情報を指定します。このプロパティはAWS S3およびその他のS3互換のストレージシステムにのみ必要です。</li></ul> |

#### FileLayoutParams

対象データファイルにアクセスするためのパラメータのセット。

```SQL
"path" = "<file_path>",
"format" = "<file_format>"
"enable_recursive_listing" = "{ true | false }"
```

| パラメータ        | 必須     | 説明                                                        |
| ---------------- | -------- | ------------------------------------------------------------ |
| path             | はい     | データファイルのパス。 <ul><li>データファイルがHDFSに格納されている場合、パス形式は `hdfs://<HDFSのIPアドレス>:<ポート>/<パス>` です。デフォルトのポート番号は8020です。デフォルトのポートを使用する場合は、指定する必要はありません。</li><li>データファイルがAWS S3またはその他のS3互換のストレージシステムに格納されている場合、パス形式は `s3://<バケット名>/<フォルダ>/` です。</li></ul>パスを入力する際は次のルールに注意してください：<ul><li>パス内のすべてのファイルにアクセスしたい場合は、このパラメータをスラッシュ（`/`）で終了させてください。例：`hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data/`。クエリを実行すると、StarRocksはパス内のすべてのデータファイルを走査します。再帰を使用してデータファイルを走査しません。</li><li>単一のファイルにアクセスしたい場合は、このファイルを直接指すパスを入力してください。例： `hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data`。クエリを実行すると、StarRocksはこのデータファイルのみをスキャンします。</li></ul> |
| format           | はい     | データファイルのフォーマット。有効な値：`parquet`、`orc`、`avro`、`rctext`または`rcbinary`、`sequence`。 |
| enable_recursive_listing | いいえ   | 現在のパスの下にあるすべてのファイルを再帰的に走査するかどうかを指定します。デフォルト値：`false`。 |

#### StorageCredentialParams（オプション）

対象のストレージシステムとの統合に関する一連のパラメータ。このパラメータセットは**オプション**です。

対象のストレージシステムがAWS S3またはその他のS3互換のストレージシステムの場合、`StorageCredentialParams`を構成する必要があります。

他のストレージシステムの場合、`StorageCredentialParams`を無視できます。

##### AWS S3

AWS S3に格納されているデータファイルにアクセスする必要がある場合は、次の認証パラメータを`StorageCredentialParams`で構成してください。

- インスタンスプロファイルベースの認証メソッドを選択した場合、`StorageCredentialParams`を次のように構成します：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- アサムドロールベースの認証メソッドを選択した場合、`StorageCredentialParams`を次のように構成します：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<あなたの仮定される役割のARN>",
"aws.s3.region" = "<aws_s3_region>"
```

- IAMユーザーベースの認証メソッドを選択した場合、`StorageCredentialParams`を次のように構成します：

```JavaScript
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

| パラメータ名                | 必須       | 説明                                                         |
| --------------------------- | ---------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい       | AWS S3にアクセスする際にインスタンスプロファイルベースの認証メソッドおよびアサムドロールベースの認証メソッドを有効にするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。 |
| aws.s3.iam_role_arn         | はい       | AWS S3バケットに権限を持つIAMロールのARN。<br />AWS S3にアサムドロールベースの認証メソッドでアクセスする場合は、このパラメータを指定する必要があります。その後、StarRocksは対象データファイルにアクセスする際にこの役割を仮定します。 |
| aws.s3.region               | はい       | AWS S3バケットが存在するリージョン。例：us-west-1。          |
| aws.s3.access_key           | いいえ     | IAMユーザーのアクセスキー。AWS S3にアクセスするためのIAMユーザーベースの認証メソッドを使用する場合は、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ     | IAMユーザーのシークレットキー。IAMユーザーベースの認証メソッドを使用してAWS S3にアクセスする場合は、このパラメータを指定する必要があります。 |

AWS S3へのアクセスに使用する認証方法の選択とAWS IAMコンソールでのアクセス制御ポリシーの構成方法については、[AWS S3へのアクセスの認証パラメータ](../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージ

MinIOなどのS3互換ストレージシステムにアクセスする必要がある場合は、成功した統合を確保するために、以下のように`StorageCredentialParams`を構成します。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表には、`StorageCredentialParams`で構成する必要があるパラメータが記載されています。

| パラメータ                     | 必須     | 説明                                                     |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | はい      | SSL接続を有効にするかどうかを指定します。<br />有効な値: `true` および `false`。 デフォルト値: `true`。 |
| aws.s3.enable_path_style_access | はい      | パス形式アクセスを有効にするかどうかを指定します。<br />有効な値: `true` および `false`。 デフォルト値: `false`。MinIOの場合は、値を`true`に設定する必要があります。<br />パス形式URLは次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。 たとえば、米国西部（オレゴン）リージョンで`DOC-EXAMPLE-BUCKET1`という名前のバケットを作成し、そのバケットで`alice.jpg`オブジェクトにアクセスしたい場合、次のパス形式URLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | はい      | AWS S3の代わりにS3互換ストレージシステムに接続するために使用するエンドポイント。 |
| aws.s3.access_key               | はい      | IAMユーザーのアクセスキー。                         |
| aws.s3.secret_key               | はい     | IAMユーザーのシークレットキー。                         |

#### カラムタイプのマッピング

以下の表は、ターゲットデータファイルとファイル外部テーブル間のカラムタイプのマッピングを提供しています。

| データファイル   | ファイル外部テーブル                                          |
| ----------- | ------------------------------------------------------------ |
| INT/INTEGER | INT                                                          |
| BIGINT      | BIGINT                                                       |
| TIMESTAMP   | DATETIME。 <br />TIMESTAMPは、現在のセッションのタイムゾーン設定に基づいてタイムゾーンを持たないDATETIMEに変換され、一部の精度が失われます。 |
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

HDFSパスに格納されているParquetデータファイルをクエリするためのファイル外部テーブル`t0`を作成します。

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

例1: ファイル外部テーブルを作成し、AWS S3の**単一のParquetファイル**にアクセスするために**インスタンスプロファイル**を使用します。

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

例2: ファイル外部テーブルを作成し、**ターゲットファイルパス**内のすべてのORCファイルにアクセスするために**仮定された役割**を使用します。

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

例3: ファイル外部テーブルを作成し、AWS S3のファイルパス内の全てのORCファイルにアクセスするために**IAMユーザー**を使用します。

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

たとえば、[Examples - HDFS](#examples)で作成したファイル外部テーブル`t0`からデータをクエリする場合は、次のコマンドを実行します:

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

[DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md)を使用してテーブルのスキーマを表示したり、[DROP TABLE](../sql-reference/sql-statements/data-definition/DROP_TABLE.md)を使用してテーブルを削除したりできます。