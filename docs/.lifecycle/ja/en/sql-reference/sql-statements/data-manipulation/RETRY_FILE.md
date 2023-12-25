# リトライファイル

## 説明

パイプ内の全てのデータファイルまたは特定のデータファイルの読み込みを再試行します。

## 構文

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { RETRY ALL | RETRY FILE '<file_name>' }
```

## パラメータ

### pipe_name

パイプの名前です。

### file_name

再試行するデータファイルのストレージパスです。ファイルの完全なストレージパスを指定する必要があります。`pipe_name`で指定したパイプに属していないファイルを指定した場合、エラーが返されます。

## 例

以下の例では、`user_behavior_replica`という名前のパイプ内の全てのデータファイルの読み込みを再試行します：

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY ALL;
```

以下の例では、`user_behavior_replica`という名前のパイプで`s3://starrocks-datasets/user_behavior_ten_million_rows.parquet`データファイルの読み込みを再試行します：

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY FILE 's3://starrocks-datasets/user_behavior_ten_million_rows.parquet';
```
