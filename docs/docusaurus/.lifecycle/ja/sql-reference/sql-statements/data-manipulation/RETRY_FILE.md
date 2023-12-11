# ファイルの再試行

## 説明

パイプ内のすべてのデータファイルまたは特定のデータファイルを再試行します。

## 構文

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { RETRY ALL | RETRY FILE '<file_name>' }
```

## パラメータ

### pipe_name

パイプの名前。

### file_name

再試行して読み込もうとするデータファイルの保存パスです。ファイルの完全な保存パスを指定する必要があります。`pipe_name` で指定したパイプに属していないファイルを指定した場合、エラーが返されます。

## 例

次の例は、`user_behavior_replica` という名前のパイプ内のすべてのデータファイルを再試行します。

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY ALL;
```

次の例は、`user_behavior_replica` という名前のパイプ内のデータファイル `s3://starrocks-datasets/user_behavior_ten_million_rows.parquet` を再試行します。

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY FILE 's3://starrocks-datasets/user_behavior_ten_million_rows.parquet';
```