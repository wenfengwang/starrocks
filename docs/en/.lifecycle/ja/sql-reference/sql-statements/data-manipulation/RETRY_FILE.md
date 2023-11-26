# ファイルの再試行

## 説明

パイプ内のすべてのデータファイルまたは特定のデータファイルを再試行して読み込みます。

## 構文

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { RETRY ALL | RETRY FILE '<file_name>' }
```

## パラメータ

### pipe_name

パイプの名前です。

### file_name

再試行して読み込みたいデータファイルのストレージパスです。ファイルの完全なストレージパスを指定する必要があります。指定したファイルが `pipe_name` で指定したパイプに属していない場合、エラーが返されます。

## 例

以下の例では、`user_behavior_replica` という名前のパイプ内のすべてのデータファイルを再試行して読み込みます。

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY ALL;
```

以下の例では、`user_behavior_replica` という名前のパイプ内のデータファイル `s3://starrocks-datasets/user_behavior_ten_million_rows.parquet` を再試行して読み込みます。

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY FILE 's3://starrocks-datasets/user_behavior_ten_million_rows.parquet';
```
