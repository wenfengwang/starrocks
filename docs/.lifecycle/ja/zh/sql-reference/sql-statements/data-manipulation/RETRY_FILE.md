# リトライファイル

## 機能

指定されたPipe内の全てのデータファイル、または特定のデータファイルのインポートを再試行します。

## 構文

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { RETRY ALL | RETRY FILE '<file_name>' }
```

## パラメータ説明

### pipe_name

Pipeの名前です。

### file_name

再試行するデータファイルの保存パスです。ここでは完全なパスを指定する必要があります。指定されたファイルが現在指定されているPipeに属していない場合は、エラーが返されます。

## 例

`user_behavior_replica`という名前のPipeに含まれる全てのデータファイルのインポートを再試行します：

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY ALL;
```

`user_behavior_replica`という名前のPipeに含まれるデータファイル`s3://starrocks-datasets/user_behavior_ten_million_rows.parquet`のインポートを再試行します：

```SQL
ALTER PIPE [ IF EXISTS ] user_behavior_replica RETRY FILE 's3://starrocks-datasets/user_behavior_ten_million_rows.parquet';
```
