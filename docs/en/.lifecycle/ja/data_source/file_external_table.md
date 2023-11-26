---
displayed_sidebar: "Japanese"
---

# ファイル外部テーブル

ファイル外部テーブルは、特殊な種類の外部テーブルです。これにより、データをStarRocksにロードせずに、外部ストレージシステムのParquetおよびORCデータファイルを直接クエリできます。また、ファイル外部テーブルはメタストアに依存しません。現在のバージョンでは、StarRocksは次の外部ストレージシステムをサポートしています：HDFS、Amazon S3、およびその他のS3互換のストレージシステム。

この機能は、StarRocks v2.5からサポートされています。

## 制限事項

- ファイル外部テーブルは、[default_catalog](../data_source/catalog/default_catalog.md)内のデータベースに作成する必要があります。クラスタで作成されたカタログをクエリするには、[SHOW CATALOGS](../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)を実行できます。
- Parquet、ORC、Avro、RCFile、およびSequenceFileのデータファイルのみがサポートされています。
- ファイル外部テーブルは、対象のデータファイル内のデータをクエリするためにのみ使用できます。INSERT、DELETE、およびDROPなどのデータ書き込み操作はサポートされていません。

## 前提条件

ファイル外部テーブルを作成する前に、StarRocksクラスタを構成して、StarRocksが対象のデータファイルが格納されている外部ストレージシステムにアクセスできるようにする必要があります。ファイル外部テーブルに必要な構成は、Hiveカタログに必要な構成と同じですが、メタストアの構成は必要ありません。構成についての詳細は、[Hiveカタログ - 統合の準備](../data_source/catalog/hive_catalog.md#integration-preparations)を参照してください。

## データベースの作成（オプション）

StarRocksクラスタに接続した後、既存のデータベースにファイル外部テーブルを作成するか、新しいデータベースを作成してファイル外部テーブルを管理することができます。クラスタ内の既存のデータベースをクエリするには、[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)を実行します。その後、`USE <db_name>`を実行して対象のデータベースに切り替えることができます。

データベースを作成するための構文は次のとおりです。

```SQL
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## ファイル外部テーブルの作成

対象のデータベースにアクセスした後、このデータベース内にファイル外部テーブルを作成することができます。

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

| パラメータ        | 必須 | 説明                                                         |
| ---------------- | ---- | ------------------------------------------------------------ |
| table_name       | Yes  | ファイル外部テーブルの名前です。命名規則は次のとおりです：<ul><li>名前には、文字、数字（0-9）、およびアンダースコア（_）を含めることができます。ただし、文字で始める必要があります。</li><li>名前の長さは64文字を超えることはできません。</li></ul> |
| col_name         | Yes  | ファイル外部テーブルの列名です。ファイル外部テーブルの列名は、対象のデータファイルの列名と同じである必要がありますが、大文字と小文字は区別されません。ファイル外部テーブルの列の順序は、対象のデータファイルの順序と異なる場合があります。 |
| col_type         | Yes  | ファイル外部テーブルの列の型です。このパラメータは、対象のデータファイルの列の型に基づいて指定する必要があります。詳細については、[列の型のマッピング](#mapping-of-column-types)を参照してください。 |
| NULL \| NOT NULL | No   | ファイル外部テーブルの列がNULLであることを許可するかどうかを指定します。 <ul><li>NULL：NULLが許可されます。</li><li>NOT NULL：NULLは許可されません。</li></ul>次のルールに基づいてこの修飾子を指定する必要があります：<ul><li>対象のデータファイルの列にこのパラメータが指定されていない場合、ファイル外部テーブルの列には指定しないか、ファイル外部テーブルの列にNULLを指定することができます。</li><li>対象のデータファイルの列にNULLが指定されている場合、ファイル外部テーブルの列にはこのパラメータを指定しないか、ファイル外部テーブルの列にNULLを指定することができます。</li><li>対象のデータファイルの列にNOT NULLが指定されている場合、ファイル外部テーブルの列にもNOT NULLを指定する必要があります。</li></ul> |
| comment          | No   | ファイル外部テーブルの列のコメントです。                      |
| ENGINE           | Yes  | エンジンのタイプです。値をfileに設定します。                   |
| comment          | No   | ファイル外部テーブルの説明です。                              |
| PROPERTIES       | Yes  | <ul><li>`FileLayoutParams`：対象ファイルのパスと形式を指定します。このプロパティは必須です。</li><li>`StorageCredentialParams`：オブジェクトストレージシステムにアクセスするために必要な認証情報を指定します。このプロパティは、AWS S3およびその他のS3互換のストレージシステムにのみ必要です。</li></ul> |

#### FileLayoutParams

対象データファイルにアクセスするためのパラメータのセットです。

```SQL
"path" = "<file_path>",
"format" = "<file_format>"
"enable_recursive_listing" = "{ true | false }"
```

| パラメータ                | 必須 | 説明                                                         |
| ------------------------ | ---- | ------------------------------------------------------------ |
| path                     | Yes  | データファイルのパスです。<ul><li>データファイルがHDFSに格納されている場合、パスの形式は`hdfs://<HDFSのIPアドレス>:<ポート>/<パス>`です。デフォルトのポート番号は8020です。デフォルトのポートを使用する場合は、指定する必要はありません。</li><li>データファイルがAWS S3またはその他のS3互換のストレージシステムに格納されている場合、パスの形式は`s3://<バケット名>/<フォルダ>/`です。</li></ul>次のルールに注意してパスを入力します：<ul><li>パス内のすべてのファイルにアクセスする場合は、このパラメータをスラッシュ（`/`）で終了させてください。たとえば、`hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data/`のようにします。クエリを実行すると、StarRocksはパスの下にあるすべてのデータファイルをトラバースします。再帰を使用してデータファイルをトラバースしません。</li><li>単一のファイルにアクセスする場合は、このファイルを直接指すパスを入力します。たとえば、`hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data`のようにします。クエリを実行すると、StarRocksはこのデータファイルのみをスキャンします。</li></ul> |
| format                   | Yes  | データファイルの形式です。有効な値：`parquet`、`orc`、`avro`、`rctext`または`rcbinary`、および`sequence`。 |
| enable_recursive_listing | No   | 現在のパスの下にあるすべてのファイルを再帰的にトラバースするかどうかを指定します。デフォルト値：`false`。 |

#### StorageCredentialParams（オプション）

対象ストレージシステムとの統合方法に関するパラメータのセットです。このパラメータセットは**オプション**です。

対象ストレージシステムがAWS S3またはその他のS3互換のストレージである場合、`StorageCredentialParams`を構成する必要があります。

他のストレージシステムの場合、`StorageCredentialParams`は無視してください。

##### AWS S3

AWS S3に格納されているデータファイルにアクセスする必要がある場合は、`StorageCredentialParams`で次の認証パラメータを構成します。

- インスタンスプロファイルベースの認証方法を選択した場合、`StorageCredentialParams`を次のように構成します：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- アサムドロールベースの認証方法を選択した場合、`StorageCredentialParams`を次のように構成します：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<あなたのアサムドロールのARN>",
"aws.s3.region" = "<aws_s3_region>"
```

- IAMユーザーベースの認証方法を選択した場合、`StorageCredentialParams`を次のように構成します：

```JavaScript
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

| パラメータ名                  | 必須 | 説明                                                         |
| --------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes  | AWS S3にアクセスする際にインスタンスプロファイルベースの認証方法とアサムドロールベースの認証方法を有効にするかどうかを指定します。有効な値：`true`および`false`。デフォルト値：`false`。 |
| aws.s3.iam_role_arn         | Yes  | AWS S3バケットに対する特権を持つIAMロールのARN。<br />AWS S3へのアクセスにアサムドロールベースの認証方法を使用する場合、このパラメータを指定する必要があります。その後、StarRocksは対象のデータファイルにアクセスする際にこのロールを仮定します。 |
| aws.s3.region               | Yes  | AWS S3バケットが存在するリージョン。例：us-west-1。 |
| aws.s3.access_key           | No   | IAMユーザーのアクセスキー。IAMユーザーベースの認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | No   | IAMユーザーのシークレットキー。IAMユーザーベースの認証方法を使用してAWS S3にアクセスする場合、このパラメータを指定する必要があります。|

AWS S3へのアクセス方法の選択方法とAWS IAMコンソールでアクセス制御ポリシーを構成する方法については、[AWS S3へのアクセスの認証パラメータ](../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換のストレージ

MinIOなどのS3互換のストレージシステムにアクセスする必要がある場合は、次のように`StorageCredentialParams`を構成して、統合が成功するようにします。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

次の表に、`StorageCredentialParams`で構成する必要のあるパラメータを説明します。

| パラメータ                       | 必須 | 説明                                                         |
| ------------------------------- | ---- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | Yes  | SSL接続を有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`true`。 |
| aws.s3.enable_path_style_access | Yes  | パススタイルアクセスを有効にするかどうかを指定します。<br />有効な値：`true`および`false`。デフォルト値：`false`。MinIOの場合、値を`true`に設定する必要があります。<br />パススタイルURLは次の形式を使用します：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。たとえば、US West（Oregon）リージョンで`DOC-EXAMPLE-BUCKET1`というバケットを作成し、そのバケットの`alice.jpg`オブジェクトにアクセスする場合、次のパススタイルURLを使用できます：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | Yes  | AWS S3ではなく、S3互換のストレージシステムに接続するために使用するエンドポイントです。 |
| aws.s3.access_key               | Yes  | IAMユーザーのアクセスキー。                         |
| aws.s3.secret_key               | Yes  | IAMユーザーのシークレットキー。                         |

#### 列の型のマッピング

次の表は、対象のデータファイルとファイル外部テーブルの列の型のマッピングを示しています。

| データファイル   | ファイル外部テーブル                                          |
| ----------- | ------------------------------------------------------------ |
| INT/INTEGER | INT                                                          |
| BIGINT      | BIGINT                                                       |
| TIMESTAMP   | DATETIME。<br />TIMESTAMPは、現在のセッションのタイムゾーン設定に基づいてタイムゾーンなしのDATETIMEに変換され、一部の精度が失われます。 |
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

HDFSパスに格納されているParquetデータファイルをクエリするためのファイル外部テーブル「t0」を作成します。

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

例1：AWS S3の単一のParquetファイルにアクセスするためのファイル外部テーブルを作成し、**インスタンスプロファイル**を使用します。

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

例2：AWS S3のターゲットファイルパスの下にあるすべてのORCファイルにアクセスするためのファイル外部テーブルを作成し、**アサムドロール**を使用します。

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

例3：AWS S3のターゲットファイルパスの下にあるすべてのORCファイルにアクセスするためのファイル外部テーブルを作成し、**IAMユーザー**を使用します。

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

構文：

```SQL
SELECT <clause> FROM <file_external_table>
```

たとえば、[例 - HDFS](#hdfs)で作成したファイル外部テーブル「t0」からデータをクエリするには、次のコマンドを実行します。

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

[DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md)を使用してテーブルのスキーマを表示したり、[DROP TABLE](../sql-reference/sql-statements/data-definition/DROP_TABLE.md)を使用してテーブルを削除したりすることができます。
