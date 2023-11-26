---
displayed_sidebar: "Japanese"
---

# パイプの作成

## 説明

指定されたソースデータファイルから指定された宛先テーブルにデータをロードするためにシステムが使用するINSERT INTO SELECT FROM FILESステートメントを定義するための新しいパイプを作成します。

## 構文

```SQL
CREATE [OR REPLACE] PIPE [db_name.]<pipe_name> 
[PROPERTIES ("<key>" = "<value>"[, "<key> = <value>" ...])]
AS <INSERT_SQL>
```

## パラメータ

### db_name

パイプが所属するデータベースの名前です。

### pipe_name

パイプの名前です。パイプ名は、パイプが作成されるデータベース内で一意である必要があります。

### INSERT_SQL

指定されたソースデータファイルから宛先テーブルにデータをロードするために使用されるINSERT INTO SELECT FROM FILESステートメントです。

FILES()テーブル関数の詳細については、[FILES](../../../sql-reference/sql-functions/table-functions/files.md)を参照してください。

### PROPERTIES

パイプの実行方法を指定するオプションのパラメータのセットです。形式: `"key" = "value"`。

| プロパティ      | デフォルト値 | 説明                                                  |
| :------------ | :------------ | :----------------------------------------------------------- |
| AUTO_INGEST   | `TRUE`        | 自動的な増分データロードを有効にするかどうかを指定します。有効な値: `TRUE` および `FALSE`。このパラメータを `TRUE` に設定すると、自動的な増分データロードが有効になります。このパラメータを `FALSE` に設定すると、システムはジョブ作成時に指定されたソースデータファイルの内容のみをロードし、その後の新しいまたは更新されたファイルの内容はロードされません。大量ロードの場合、このパラメータを `FALSE` に設定できます。 |
| POLL_INTERVAL | `10` (秒) | 自動的な増分データロードのポーリング間隔です。   |
| BATCH_SIZE    | `1GB`         | バッチとしてロードされるデータのサイズです。パラメータ値に単位を含めない場合、デフォルトのバイト単位が使用されます。 |
| BATCH_FILES   | `256`         | バッチとしてロードされるソースデータファイルの数です。     |

## 例

現在のデータベースに`user_behavior_replica`という名前のパイプを作成し、サンプルデータセット`s3://starrocks-datasets/user_behavior_ten_million_rows.parquet`のデータを`user_behavior_replica`テーブルにロードします。

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
> 上記のコマンドで`AAA`と`BBB`を自分の認証情報に置き換えてください。有効な`aws.s3.access_key`と`aws.s3.secret_key`は、どのAWS認証ユーザーでもオブジェクトが読み取り可能なため、使用できます。

この例では、IAMユーザーベースの認証方法と、StarRocksテーブルと同じスキーマを持つParquetファイルを使用しています。他の認証方法とCREATE PIPEの使用方法の詳細については、[AWSリソースへの認証](../../../integrations/authenticate_to_aws_resources.md)と[FILES](../../../sql-reference/sql-functions/table-functions/files.md)を参照してください。
