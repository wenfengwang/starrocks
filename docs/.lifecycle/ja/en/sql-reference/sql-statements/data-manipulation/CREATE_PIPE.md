---
displayed_sidebar: English
---

# CREATE PIPE

## 説明

指定されたソースデータファイルから宛先テーブルにデータをロードするためにシステムが使用するINSERT INTO SELECT FROM FILESステートメントを定義する新しいパイプを作成します。

## 構文

```SQL
CREATE [OR REPLACE] PIPE [db_name.]<pipe_name> 
[PROPERTIES ("<key>" = "<value>"[, "<key>" = "<value>" ...])]
AS <INSERT_SQL>
```

## パラメーター

### db_name

パイプが属するデータベースのユニークな名前。

> **注意**
>
> 各パイプは特定のデータベースに属しています。パイプが属するデータベースを削除すると、パイプもデータベースと共に削除され、データベースが復旧してもパイプは復旧できません。

### pipe_name

パイプの名前です。パイプ名は、パイプが作成されるデータベース内でユニークでなければなりません。

### INSERT_SQL

指定されたソースデータファイルから宛先テーブルにデータをロードするために使用されるINSERT INTO SELECT FROM FILESステートメントです。

FILES()テーブル関数の詳細については、[FILES](../../../sql-reference/sql-functions/table-functions/files.md)を参照してください。

### PROPERTIES

パイプの実行方法を指定するオプションのパラメーターのセットです。フォーマット: `"key" = "value"`。

| プロパティ      | デフォルト値 | 説明                                                  |
| :------------ | :------------ | :----------------------------------------------------------- |
| AUTO_INGEST   | `TRUE`        | 自動的な増分データロードを有効にするかどうか。有効な値: `TRUE` と `FALSE`。このパラメーターを`TRUE`に設定すると、自動増分データロードが有効になります。このパラメーターを`FALSE`に設定すると、ジョブ作成時に指定されたソースデータファイルの内容のみがロードされ、その後の新規または更新されたファイルの内容はロードされません。バルクロードの場合、このパラメーターを`FALSE`に設定できます。 |
| POLL_INTERVAL | `10` (秒) | 自動増分データロードのポーリング間隔です。   |
| BATCH_SIZE    | `1GB`         | バッチとしてロードされるデータのサイズです。パラメーター値に単位を含めない場合、デフォルトの単位はバイトです。 |
| BATCH_FILES   | `256`         | バッチとしてロードされるソースデータファイルの数です。     |

## 例

現在のデータベースに`user_behavior_replica`という名前のパイプを作成し、サンプルデータセット`s3://starrocks-datasets/user_behavior_ten_million_rows.parquet`のデータを`user_behavior_replica`テーブルにロードします：

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

> **注記**
>
> 上記のコマンドで`AAA`と`BBB`をあなたの認証情報に置き換えてください。任意の有効な`aws.s3.access_key`と`aws.s3.secret_key`を使用できます。オブジェクトはAWS認証ユーザーなら誰でも読み取り可能です。

この例ではIAMユーザーベースの認証方法と、StarRocksテーブルと同じスキーマを持つParquetファイルを使用しています。他の認証方法とCREATE PIPEの使用方法の詳細については、[AWSリソースへの認証](../../../integrations/authenticate_to_aws_resources.md)と[FILES](../../../sql-reference/sql-functions/table-functions/files.md)を参照してください。
