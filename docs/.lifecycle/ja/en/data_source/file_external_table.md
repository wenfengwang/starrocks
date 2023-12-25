---
displayed_sidebar: English
---

# ファイル外部テーブル

ファイル外部テーブルは、特別なタイプの外部テーブルです。これにより、StarRocks にデータをロードすることなく、外部ストレージシステム内の Parquet および ORC データファイルに直接クエリを実行できます。また、ファイル外部テーブルはメタストアに依存しません。現在のバージョンでは、StarRocks は HDFS、Amazon S3、およびその他の S3 互換ストレージシステムをサポートしています。

この機能は StarRocks v2.5 からサポートされています。

## 制限

- ファイル外部テーブルは [default_catalog](../data_source/catalog/default_catalog.md) 内のデータベースに作成する必要があります。[SHOW CATALOGS](../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) を実行して、クラスター内で作成されたカタログを照会できます。
- Parquet、ORC、Avro、RCFile、および SequenceFile データファイルのみがサポートされています。
- ファイル外部テーブルは、ターゲットデータファイル内のデータをクエリするためにのみ使用できます。INSERT、DELETE、DROP などのデータ書き込み操作はサポートされていません。

## 前提条件

ファイル外部テーブルを作成する前に、StarRocks クラスターを設定して、StarRocks がターゲットデータファイルが保存されている外部ストレージシステムにアクセスできるようにする必要があります。ファイル外部テーブルに必要な設定は Hive カタログに必要な設定と同じですが、メタストアを設定する必要はありません。設定についての詳細は [Hive カタログ - 統合の準備](../data_source/catalog/hive_catalog.md#integration-preparations) を参照してください。

## データベースの作成（オプション）

StarRocks クラスタに接続した後、既存のデータベースにファイル外部テーブルを作成するか、新しいデータベースを作成してファイル外部テーブルを管理することができます。クラスタ内の既存のデータベースをクエリするには、[SHOW DATABASES](../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) を実行します。その後、`USE <db_name>` を実行してターゲットデータベースに切り替えることができます。

データベースを作成するための構文は次のとおりです。

```SQL
CREATE DATABASE [IF NOT EXISTS] <db_name>
```

## ファイル外部テーブルの作成

ターゲットデータベースにアクセスした後、このデータベースにファイル外部テーブルを作成できます。

### 構文

```SQL
CREATE EXTERNAL TABLE <table_name>
(
    <col_name> <col_type> [NULL | NOT NULL] [COMMENT "<comment>"]
) 
ENGINE=file
COMMENT ["<comment>"]
PROPERTIES
(
    FileLayoutParams,
    StorageCredentialParams
)
```

### パラメータ

| パラメータ        | 必須 | 説明                                                  |
| ---------------- | -------- | ------------------------------------------------------------ |
| table_name       | はい      | ファイル外部テーブルの名前。命名規則は以下の通りです：<ul><li>名前には文字、数字 (0-9)、およびアンダースコア (_) を含めることができます。文字で始まる必要があります。</li><li>名前の長さは 64 文字を超えてはいけません。</li></ul> |
| col_name         | はい      | ファイル外部テーブルの列名。ファイル外部テーブルの列名は、ターゲットデータファイルの列名と同じでなければなりませんが、大文字と小文字は区別されません。ファイル外部テーブルの列の順序は、ターゲットデータファイルの列の順序と異なる場合があります。 |
| col_type         | はい      | ファイル外部テーブルの列の型。このパラメータは、ターゲットデータファイルの列の型に基づいて指定する必要があります。詳細は [列の型のマッピング](#mapping-of-column-types) を参照してください。 |
| NULL \| NOT NULL | いいえ       | ファイル外部テーブルの列が NULL を許容するかどうか。<ul><li>NULL: NULL が許可されます。</li><li>NOT NULL: NULL は許可されません。</li></ul>この修飾子は、以下の規則に基づいて指定する必要があります：<ul><li>ターゲットデータファイルの列にこのパラメータが指定されていない場合、ファイル外部テーブルの列に指定しないか、または NULL を指定することができます。</li><li>ターゲットデータファイルの列に NULL が指定されている場合、ファイル外部テーブルの列にこのパラメータを指定しないか、または NULL を指定することができます。</li><li>ターゲットデータファイルの列に NOT NULL が指定されている場合、ファイル外部テーブルの列にも NOT NULL を指定する必要があります。</li></ul> |
| comment          | いいえ       | ファイル外部テーブルの列のコメント。            |
| ENGINE           | はい      | エンジンのタイプ。値は file に設定します。                   |
| comment          | いいえ       | ファイル外部テーブルの説明。                  |
| PROPERTIES       | はい      | <ul><li>`FileLayoutParams`: ターゲットファイルのパスとフォーマットを指定します。このプロパティは必須です。</li><li>`StorageCredentialParams`: オブジェクトストレージシステムへのアクセスに必要な認証情報を指定します。このプロパティは AWS S3 およびその他の S3 互換ストレージシステムにのみ必要です。</li></ul> |

#### FileLayoutParams

ターゲットデータファイルにアクセスするためのパラメータセット。

```SQL
"path" = "<file_path>",
"format" = "<file_format>",
"enable_recursive_listing" = "{ true | false }"
```

| パラメータ                | 必須 | 説明                                                  |
| ------------------------ | -------- | ------------------------------------------------------------ |
| path                     | はい      | データファイルのパス。<ul><li>HDFS に保存されているデータファイルの場合、パス形式は `hdfs://<HDFS の IP アドレス>:<ポート>/<パス>` です。デフォルトのポート番号は 8020 です。デフォルトポートを使用する場合は、指定する必要はありません。</li><li>AWS S3 またはその他の S3 互換ストレージシステムに保存されているデータファイルの場合、パス形式は `s3://<バケット名>/<フォルダ>/` です。</li></ul>パスを入力する際の注意点：<ul><li>パス内のすべてのファイルにアクセスしたい場合は、このパラメータをスラッシュ (`/`) で終わらせます。例：`hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data/`。クエリを実行すると、StarRocks はパス下のすべてのデータファイルをトラバースします。データファイルを再帰的にトラバースすることはありません。</li><li>単一のファイルにアクセスしたい場合は、そのファイルを直接指すパスを入力します。例：`hdfs://x.x.x.x/user/hive/warehouse/array2d_parq/data`。クエリを実行すると、StarRocks はそのデータファイルのみをスキャンします。</li></ul> |
| format                   | はい      | データファイルのフォーマット。有効な値は `parquet`、`orc`、`avro`、`rctext` または `rcbinary`、`sequence` です。 |
| enable_recursive_listing | いいえ       | 現在のパスの下のすべてのファイルを再帰的にトラバースするかどうかを指定します。デフォルト値は `false` です。 |

#### StorageCredentialParams（オプション）

StarRocks がターゲットストレージシステムと統合する方法に関するパラメータセット。このパラメータセットは**オプション**です。

ターゲットストレージシステムが AWS S3 またはその他の S3 互換ストレージの場合のみ `StorageCredentialParams` を設定する必要があります。

他のストレージシステムについては、`StorageCredentialParams`を無視しても構いません。

##### AWS S3

AWS S3に保存されたデータファイルにアクセスする必要がある場合は、`StorageCredentialParams`で以下の認証パラメータを設定します。

- インスタンスプロファイルベースの認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- 想定ロールベースの認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

```JavaScript
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<ARN of your assumed role>",
"aws.s3.region" = "<aws_s3_region>"
```

- IAMユーザーベースの認証方法を選択する場合、`StorageCredentialParams`を以下のように設定します：

```JavaScript
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

| パラメータ名                   | 必須 | 説明                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | はい      | AWS S3にアクセスする際にインスタンスプロファイルベースの認証方法または想定ロールベースの認証方法を有効にするかどうかを指定します。有効な値: `true`、`false`。デフォルト値: `false`。 |
| aws.s3.iam_role_arn         | はい      | AWS S3バケットに権限を持つIAMロールのARN。<br />想定ロールベースの認証方法でAWS S3にアクセスする場合、このパラメータを指定する必要があります。その後、StarRocksはターゲットデータファイルにアクセスする際にこのロールを引き受けます。 |
| aws.s3.region               | はい      | AWS S3バケットが存在するリージョン。例: us-west-1。 |
| aws.s3.access_key           | いいえ       | IAMユーザーのアクセスキー。IAMユーザーベースの認証方法でAWS S3にアクセスする場合、このパラメータを指定する必要があります。 |
| aws.s3.secret_key           | いいえ       | IAMユーザーのシークレットキー。IAMユーザーベースの認証方法でAWS S3にアクセスする場合、このパラメータを指定する必要があります。|

AWS S3へのアクセスに使用する認証方法の選択とAWS IAMコンソールでアクセス制御ポリシーを設定する方法については、[AWS S3へのアクセスのための認証パラメータ](../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)を参照してください。

##### S3互換ストレージ

MinIOなどのS3互換ストレージシステムにアクセスする必要がある場合は、`StorageCredentialParams`を以下のように設定して、成功した統合を確実にします：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下の表は、`StorageCredentialParams`で設定する必要があるパラメータを説明しています。

| パラメータ                         | 必須 | 説明                                                  |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl               | はい      | SSL接続を有効にするかどうかを指定します。<br />有効な値: `true`、`false`。デフォルト値: `true`。 |
| aws.s3.enable_path_style_access | はい      | パススタイルアクセスを有効にするかどうかを指定します。<br />有効な値: `true`、`false`。デフォルト値: `false`。MinIOの場合は、値を`true`に設定する必要があります。<br />パススタイルURLは次の形式を使用します: `https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例えば、US West (Oregon)リージョンに`DOC-EXAMPLE-BUCKET1`という名前のバケットを作成し、そのバケット内の`alice.jpg`オブジェクトにアクセスする場合、次のパススタイルURLを使用できます: `https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | はい      | AWS S3ではなく、S3互換ストレージシステムに接続するために使用されるエンドポイント。 |
| aws.s3.access_key               | はい      | IAMユーザーのアクセスキー。                         |
| aws.s3.secret_key               | はい      | IAMユーザーのシークレットキー。                         |

#### 列タイプのマッピング

以下の表は、ターゲットデータファイルとファイル外部テーブルの間の列タイプのマッピングを提供します。

| データファイル   | ファイル外部テーブル                                          |
| ----------- | ------------------------------------------------------------ |
| INT/INTEGER | INT                                                          |
| BIGINT      | BIGINT                                                       |
| TIMESTAMP   | DATETIME。<br />TIMESTAMPは、現在のセッションのタイムゾーン設定に基づいてタイムゾーンなしでDATETIMEに変換され、一部の精度が失われることに注意してください。 |
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

HDFSパスに格納されたParquetデータファイルをクエリするための`t0`という名前のファイル外部テーブルを作成します。

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

例1: ファイル外部テーブルを作成し、**インスタンスプロファイル**を使用してAWS S3内の**単一のParquetファイル**にアクセスします。

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

例2: ファイル外部テーブルを作成し、**想定ロール**を使用してAWS S3のターゲットファイルパス下の**すべてのORCファイル**にアクセスします。

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

例3: ファイル外部テーブルを作成し、**IAMユーザー**を使用してAWS S3のファイルパス下の**すべてのORCファイル**にアクセスします。

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

```SQL
SELECT <clause> FROM <file_external_table>
```

たとえば、[例 - HDFS](#examples)で作成したファイル外部テーブル`t0`からデータをクエリするには、次のコマンドを実行します：

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

テーブルのスキーマを表示するには [DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md) を使用し、テーブルを削除するには [DROP TABLE](../sql-reference/sql-statements/data-definition/DROP_TABLE.md) を使用できます。
