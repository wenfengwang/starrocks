# ファイルを再試行

## 説明

パイプですべてのデータファイルまたは特定のデータファイルを再試行します。

## 構文

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { RETRY ALL | RETRY FILE '<file_name>' }
```

## パラメーター

### pipe_name

パイプの名前です。

### file_name

再試行して読み込もうとするデータファイルの格納パスです。ファイルの完全な格納パスを指定する必要があります。指定したファイルが`pipe_name`で指定したパイプに属していない場合、エラーが返されます。

## 例

次の例では、`user_behavior_replica`という名前のパイプですべてのデータファイルを再試行します：

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY ALL;
```

次の例では、`user_behavior_replica`という名前のパイプで`'s3://starrocks-datasets/user_behavior_ten_million_rows.parquet'`というデータファイルを再試行します：

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY FILE 's3://starrocks-datasets/user_behavior_ten_million_rows.parquet';
```