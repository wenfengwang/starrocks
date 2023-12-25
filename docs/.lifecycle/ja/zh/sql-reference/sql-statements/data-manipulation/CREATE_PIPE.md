---
displayed_sidebar: Chinese
---

# CREATE PIPE

## 機能

Pipeを作成し、データのインポートを実現するINSERT INTO SELECT FROM FILESステートメントを定義します。

## 文法

```SQL
CREATE [OR REPLACE] PIPE [db_name.]<pipe_name> 
[PROPERTIES ("<key>" = "<value>"[, "<key>" = "<value>" ...])]
AS <INSERT_SQL>
```

## パラメータ説明

### db_name

Pipeが属するデータベースの名前。

### pipe_name

Pipeの名前。この名前は、Pipeが存在するデータベース内で一意でなければなりません。

> **NOTICE**
>
> 各Pipeは一つのデータベースに属しています。Pipeが存在するデータベースを削除すると、そのPipeも同時に削除され、データベースの復元と共にPipeは復元されません。

### INSERT_SQL

INSERT INTO SELECT FROM FILESステートメントで、指定されたソースデータファイルからターゲットテーブルにデータをインポートします。

FILES()テーブル関数の使用方法については、[FILES](../../../sql-reference/sql-functions/table-functions/files.md)を参照してください。

### PROPERTIES

Pipeの実行を制御するいくつかのパラメータ。形式：`"key" = "value"`。

| パラメータ      | デフォルト値  | 説明                                                         |
| :------------ | :------------ | :----------------------------------------------------------- |
| AUTO_INGEST   | `TRUE`        | 自動増分インポートを有効にするかどうか。値の範囲：`TRUE` と `FALSE`。`TRUE` は自動増分インポートを有効にすることを意味します。`FALSE` は、ジョブ開始時に指定されたデータファイルの内容のみをインポートし、後に追加または変更されたファイルの内容はインポートしません。バッチインポートの場合は、`FALSE` に設定することができます。 |
| POLL_INTERVAL | `10` (秒)     | 自動増分インポートのポーリング間隔。                         |
| BATCH_SIZE    | `1 GB`        | インポートバッチのサイズ。パラメータの値に単位が指定されていない場合は、デフォルトの単位Byteが使用されます。 |
| BATCH_FILES   | `256`         | インポートバッチのファイル数。                               |

## 例

現在のデータベースに `user_behavior_replica` という名前のPipeを作成し、`s3://starrocks-datasets/user_behavior_ten_million_rows.parquet` のデータを `user_behavior_replica` テーブルにインポートします：

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

> **説明**
>
> 上記のコマンド例の `AAA` と `BBB` を実際の有効なAccess KeyとSecret Keyに置き換えて、アクセス資格情報として使用します。ここで使用されているデータオブジェクトは、すべての合法的なAWSユーザーに公開されているため、任意の実際の有効なAccess KeyとSecret Keyを入力することができます。

この例は、IAM Userに基づく認証方式を例としており、ParquetソースファイルがStarRocksターゲットテーブルの構造と同じであると仮定しています。認証方式とステートメントの詳細については、[AWS認証情報の設定](../../../integrations/authenticate_to_aws_resources.md)と[FILES](../../../sql-reference/sql-functions/table-functions/files.md)を参照してください。
