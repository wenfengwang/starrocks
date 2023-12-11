---
displayed_sidebar: "Japanese"
---

# パイプの作成

## 説明

指定されたソースデータファイルからのデータを指定した宛先テーブルにロードするためにシステムで使用されるINSERT INTO SELECT FROM FILESステートメントを定義する新しいパイプを作成します。

## 構文

```SQL
CREATE [OR REPLACE] PIPE [db_name.]<pipe_name> 
[PROPERTIES ("<key>" = "<value>"[, "<key> = <value>" ...])]
AS <INSERT_SQL>
```

## パラメータ

### db_name

パイプが属するデータベースの一意の名前です。

> **注意**
>
> 各パイプは特定のデータベースに属しています。パイプが属するデータベースを削除すると、パイプはデータベースと共に削除され、データベースが回復してもパイプは回復できません。

### pipe_name

パイプの名前です。パイプの作成されたデータベース内で、パイプ名はユニークである必要があります。

### INSERT_SQL

指定されたソースデータファイルから宛先テーブルにデータをロードするために使用されるINSERT INTO SELECT FROM FILESステートメントです。

FILES()テーブル関数の詳細については、[FILES](../../../sql-reference/sql-functions/table-functions/files.md)を参照してください。

### PROPERTIES

パイプの実行方法を指定する一連のオプションパラメータです。フォーマット: `"key" = "value"`。

| プロパティ      | デフォルト値 | 説明                                                  |
| :------------ | :------------ | :----------------------------------------------------------- |
| AUTO_INGEST   | `TRUE`        | 自動インクリメンタルデータロードを有効にするかどうか。有効な値: `TRUE` および `FALSE`。このパラメータを`TRUE` に設定すると、自動インクリメンタルデータロードが有効になります。このパラメータを`FALSE` に設定した場合、システムはジョブ作成時に指定されたソースデータファイルの内容のみをロードし、その後の新しいまたは更新されたファイルの内容はロードされません。バルクロードの場合、このパラメータを`FALSE` に設定できます。 |
| POLL_INTERVAL | `10` (second) | 自動インクリメンタルデータロードのポーリング間隔。   |
| BATCH_SIZE    | `1GB`         | バッチとしてロードされるデータのサイズ。パラメータ値に単位を含めない場合、デフォルト単位バイトが使用されます。 |
| BATCH_FILES   | `256`         | バッチとしてロードされるソースデータファイルの数。     |

## 例

指定されたサンプルデータセット`s3://starrocks-datasets/user_behavior_ten_million_rows.parquet`のデータを`user_behavior_replica`テーブルにロードするために、現在のデータベースに`user_behavior_replica`という名前のパイプを作成します。

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
> 上記のコマンドで`AAA`と`BBB`をあなたの認証情報で置き換えてください。任意の正当な`aws.s3.access_key`および`aws.s3.secret_key`を使用できます。該当オブジェクトはAWS認証ユーザーによって読み取ることができます。

この例ではIAMユーザーベースの認証方法と、StarRocksテーブルと同じスキーマを持つParquetファイルが使用されています。その他の認証方法やCREATE PIPEの使用方法の詳細については、[AWSリソースへの認証](../../../integrations/authenticate_to_aws_resources.md)および[FILES](../../../sql-reference/sql-functions/table-functions/files.md)を参照してください。