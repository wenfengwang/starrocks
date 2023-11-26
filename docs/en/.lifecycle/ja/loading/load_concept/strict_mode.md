---
displayed_sidebar: "Japanese"
---

# 厳密モード

厳密モードは、データのロードに設定できるオプションのプロパティです。ロードの動作と最終的にロードされるデータに影響を与えます。

このトピックでは、厳密モードとその設定方法について説明します。

## 厳密モードの理解

データのロード中、ソースの列のデータ型が宛先の列のデータ型と完全に一致しない場合があります。このような場合、StarRocksはデータ型が一致しないソースの列の値に対して変換を行います。データの変換は、フィールドのデータ型の不一致やフィールド長のオーバーフローなど、さまざまな問題により失敗する場合があります。適切に変換できないソースの列の値は、不適格な列の値と呼ばれ、不適格な列の値を含むソースの行は「不適格な行」と呼ばれます。厳密モードは、データのロード中に不適格な行をフィルタリングするかどうかを制御するために使用されます。

厳密モードの動作は次のようになります。

- 厳密モードが有効になっている場合、StarRocksは適格な行のみをロードします。不適格な行をフィルタリングし、不適格な行に関する詳細情報を返します。
- 厳密モードが無効になっている場合、StarRocksは不適格な列の値を`NULL`に変換し、これらの`NULL`値を含む不適格な行を適格な行とともにロードします。

以下の点に注意してください。

- 実際のビジネスシナリオでは、適格な行と不適格な行の両方に`NULL`値が含まれる場合があります。宛先の列が`NULL`値を許可しない場合、StarRocksはエラーを報告し、`NULL`値を含む行をフィルタリングします。

- [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、または[Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)ジョブに対してフィルタリングできる不適格な行の最大割合は、オプションのジョブプロパティ`max_filter_ratio`によって制御されます。[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)では、`max_filter_ratio`プロパティの設定はサポートされていません。

例えば、CSV形式のデータファイルからStarRocksテーブルに対して、`\N`（`NULL`値を示す）という値、`abc`、`2000`、`1`の値を持つ4つの行をロードしたい場合、宛先のStarRocksテーブルの列のデータ型がTINYINT [-128, 127]であるとします。

- ソースの列の値`\N`は、TINYINTに変換される際に`NULL`に変換されます。

  > **注意**
  >
  > `\N`は、宛先のデータ型に関係なく、常に`NULL`に変換されます。

- ソースの列の値`abc`は、TINYINTではないため変換に失敗し、`NULL`に変換されます。

- ソースの列の値`2000`は、TINYINTがサポートする範囲を超えているため変換に失敗し、`NULL`に変換されます。

- ソースの列の値`1`は、TINYINT型の値`1`に正しく変換されます。

厳密モードが無効になっている場合、StarRocksは4つの行すべてをロードします。

厳密モードが有効になっている場合、StarRocksは`\N`または`1`を保持する行のみをロードし、`abc`または`2000`を保持する行はフィルタリングします。フィルタリングされた行は、`max_filter_ratio`パラメータで指定されたデータ品質の不十分さによるフィルタリング可能な行の最大割合にカウントされます。

### 厳密モードが無効の場合の最終的にロードされるデータ

| ソースの列の値 | TINYINTに変換された列の値 | 宛先の列がNULL値を許可する場合のロード結果 | 宛先の列がNULL値を許可しない場合のロード結果 |
| ------------------- | --------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| \N                 | NULL                                    | 値`NULL`がロードされます。                            | エラーが報告されます。                                        |
| abc                 | NULL                                    | 値`NULL`がロードされます。                            | エラーが報告されます。                                        |
| 2000                | NULL                                    | 値`NULL`がロードされます。                            | エラーが報告されます。                                        |
| 1                   | 1                                       | 値`1`がロードされます。                               | 値`1`がロードされます。                                     |

### 厳密モードが有効の場合の最終的にロードされるデータ

| ソースの列の値 | TINYINTに変換された列の値 | 宛先の列がNULL値を許可する場合のロード結果       | 宛先の列がNULL値を許可しない場合のロード結果 |
| ------------------- | --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| \N                 | NULL                                    | 値`NULL`がロードされます。                                  | エラーが報告されます。                                        |
| abc                 | NULL                                    | 値`NULL`は許可されていないため、フィルタリングされます。 | エラーが報告されます。                                        |
| 2000                | NULL                                    | 値`NULL`は許可されていないため、フィルタリングされます。 | エラーが報告されます。                                        |
| 1                   | 1                                       | 値`1`がロードされます。                                     | 値`1`がロードされます。                                     |

## 厳密モードの設定

データをロードするために[Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、または[Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)ジョブを実行する場合、ロードジョブに対して厳密モードを設定するために`strict_mode`パラメータを使用します。有効な値は`true`と`false`です。デフォルト値は`false`です。値`true`は厳密モードを有効にし、値`false`は厳密モードを無効にします。

データをロードするために[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を実行する場合、厳密モードを設定するために`enable_insert_strict`セッション変数を使用します。有効な値は`true`と`false`です。デフォルト値は`true`です。値`true`は厳密モードを有効にし、値`false`は厳密モードを無効にします。

以下に例を示します。

### Stream Load

```Bash
curl --location-trusted -u <username>:<password> \
    -H "strict_mode: {true | false}" \
    -T <file_name> -XPUT \
    http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

Stream Loadの詳細な構文とパラメータについては、[STREAM LOAD](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

### Broker Load

```SQL
LOAD LABEL [<database_name>.]<label_name>
(
    DATA INFILE ("<file_path>"[, "<file_path>" ...])
    INTO TABLE <table_name>
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
)
PROPERTIES
(
    "strict_mode" = "{true | false}"
)
```

上記のコードスニペットはHDFSを例としています。Broker Loadの詳細な構文とパラメータについては、[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### Routine Load

```SQL
CREATE ROUTINE LOAD [<database_name>.]<job_name> ON <table_name>
PROPERTIES
(
    "strict_mode" = "{true | false}"
) 
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>[,<kafka_broker2_ip>:<kafka_broker2_port>...]",
    "kafka_topic" = "<topic_name>"
)
```

上記のコードスニペットはApache Kafka®を例としています。Routine Loadの詳細な構文とパラメータについては、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### Spark Load

```SQL
LOAD LABEL [<database_name>.]<label_name>
(
    DATA INFILE ("<file_path>"[, "<file_path>" ...])
    INTO TABLE <table_name>
)
WITH RESOURCE <resource_name>
(
    "spark.executor.memory" = "3g",
    "broker.username" = "<hdfs_username>",
    "broker.password" = "<hdfs_password>"
)
PROPERTIES
(
    "strict_mode" = "{true | false}"   
)
```

上記のコードスニペットはHDFSを例としています。Spark Loadの詳細な構文とパラメータについては、[SPARK LOAD](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)を参照してください。

### INSERT

```SQL
SET enable_insert_strict = {true | false};
INSERT INTO <table_name> ...
```

INSERTの詳細な構文とパラメータについては、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。
