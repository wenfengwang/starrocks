---
displayed_sidebar: "Japanese"
---

# パイプの作成

## 説明

指定されたソースデータファイルから宛先テーブルにデータをロードするためにシステムで使用されるINSERT INTO SELECT FROM FILESステートメントを定義する新しいパイプを作成します。

## 構文

```SQL
CREATE [OR REPLACE] PIPE [db_name.]<pipe_name> 
[PROPERTIES ("<key>" = "<value>"[, "<key> = <value>" ...])]
AS <INSERT_SQL>
```

## パラメータ

### db_name

パイプが所属するデータベースのユニークな名前。

> **注意**
>
> 各パイプは特定のデータベースに属しています。パイプが属するデータベースを削除すると、パイプはデータベースとともに削除され、データベースが復元されてもパイプを復元することはできません。

### pipe_name

パイプの名前。パイプ名は、作成されたデータベース内で一意である必要があります。

### INSERT_SQL

指定されたソースデータファイルから宛先テーブルにデータをロードするために使用されるINSERT INTO SELECT FROM FILESステートメント。

ファイルに関する詳細情報については、[FILES](../../../sql-reference/sql-functions/table-functions/files.md)を参照してください。

### PROPERTIES

パイプの実行方法を指定するオプションのパラメータのセット。形式: `"key" = "value"`。

| プロパティ     | デフォルト値 | 説明                                                     |
| :------------ | :------------ | :----------------------------------------------------------- |
| AUTO_INGEST   | `TRUE`        | 自動増分データロードを有効にするかどうか。有効な値: `TRUE` および `FALSE`。このパラメータを`TRUE`に設定すると、自動増分データロードが有効になります。このパラメータを`FALSE`に設定すると、システムはジョブ作成時に指定されたソースデータファイルのコンテンツのみをロードし、その後の新しいまたは更新されたファイルのコンテンツはロードされません。大量ロードの場合は、このパラメータを`FALSE`に設定できます。 |
| POLL_INTERVAL | `10` (秒)    | 自動増分データロードのポーリング間隔。   |
| BATCH_SIZE    | `1GB`         | バッチとしてロードするデータのサイズ。パラメータ値に単位が含まれていない場合、デフォルトのバイト単位が使用されます。 |
| BATCH_FILES   | `256`         | バッチとしてロードするソースデータファイルの数。     |

## 例

現在のデータベースに`user_behavior_replica`という名前のパイプを作成し、サンプルデータセット`s3://starrocks-datasets/user_behavior_ten_million_rows.parquet`のデータを`user_behavior_replica`テーブルにロードする場合:

```SQL
CREATE PIPE user_behavior_replica
PROPERTIES
(
    "AUTO_INGEST" = "TRUE"
)
AS
INSERT INTO user_behavior_replica
SELECT * FROM FILES
(
    "path" = "s3://starrocks-datasets/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "aws.s3.region" = "us-east-1",
    "aws.s3.access_key" = "AAAAAAAAAAAAAAAAAAAA",
    "aws.s3.secret_key" = "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
); 
```

> **注意**
>
> 上記のコマンドに対して、`AAA`および`BBB`を自身の資格情報に置き換えてください。有効な`aws.s3.access_key`および`aws.s3.secret_key`はどちらも使用できます。これは、オブジェクトがAWS認証ユーザーによって読み取り可能であるためです。

この例では、IAMユーザーベースの認証方法と、StarRocksテーブルと同じスキーマを持つParquetファイルを使用しています。他の認証方法やCREATE PIPEの使用方法についての詳細については、[AWSリソースへの認証](../../../integrations/authenticate_to_aws_resources.md)および[FILES](../../../sql-reference/sql-functions/table-functions/files.md)を参照してください。