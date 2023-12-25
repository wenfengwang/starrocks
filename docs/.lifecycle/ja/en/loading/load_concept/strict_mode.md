---
displayed_sidebar: English
---

# 厳密モード

厳密モードは、データロードにおいて設定可能なオプションのプロパティです。これは、ロード動作と最終的にロードされるデータに影響を与えます。

このトピックでは、厳密モードとは何か、および厳密モードの設定方法について紹介します。

## 厳密モードを理解する

データロード中に、ソースカラムのデータ型が宛先カラムのデータ型と完全に一致しない場合があります。このような場合、StarRocksは、データ型が一致しないソースカラム値に対して変換を実行します。データ変換は、フィールドデータ型の不一致やフィールド長のオーバーフローなど、さまざまな問題により失敗することがあります。適切に変換されないソースカラム値は不適格なカラム値とされ、不適格なカラム値を含むソース行は「不適格な行」と呼ばれます。厳密モードは、データロード中に不適格な行をフィルタリングするかどうかを制御するために使用されます。

厳密モードの動作は以下の通りです：

- 厳密モードが有効の場合、StarRocksは適格な行のみをロードします。不適格な行をフィルタリングし、不適格な行の詳細を返します。
- 厳密モードが無効の場合、StarRocksは不適格なカラム値を`NULL`に変換し、これらの`NULL`値を含む不適格な行を適格な行と共にロードします。

以下の点に注意してください：

- 実際のビジネスシナリオでは、適格な行と不適格な行の両方に`NULL`値が含まれることがあります。宛先カラムが`NULL`値を許可しない場合、StarRocksはエラーを報告し、`NULL`値を含む行をフィルタリングします。

- [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、または[Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)ジョブでフィルタリングされる不適格な行の最大割合は、オプションのジョブプロパティ`max_filter_ratio`によって制御されます。[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)では、`max_filter_ratio`プロパティの設定はサポートされていません。

例えば、CSV形式のデータファイルから`\N`（`\N`は`NULL`値を表します）、`abc`、`2000`、および`1`の値を持つ4行をStarRocksテーブルにロードしたいとします。そして、宛先のStarRocksテーブルカラムのデータ型がTINYINT [-128, 127]であるとします。

- ソースカラム値`\N`は、TINYINTへの変換時に`NULL`に処理されます。

  > **注記**
  >
  > `\N`は、変換先のデータ型に関係なく、常に`NULL`に処理されます。

- ソースカラム値`abc`は、データ型がTINYINTではなく変換に失敗するため、`NULL`に処理されます。

- ソースカラム値`2000`は、TINYINTでサポートされる範囲を超えているため変換に失敗し、`NULL`に処理されます。

- ソースカラム値`1`は、TINYINT型の値`1`に適切に変換できます。

厳密モードが無効の場合、StarRocksは4行すべてをロードします。

厳密モードが有効の場合、StarRocksは`\N`または`1`を含む行のみをロードし、`abc`または`2000`を含む行をフィルタリングします。フィルタリングされた行は、`max_filter_ratio`パラメータによって指定されたデータ品質が不十分なためにフィルタリングされる行の最大割合にカウントされます。

### 厳密モード無効時の最終ロードデータ

| ソースカラム値 | TINYINTへの変換後のカラム値 | 宛先カラムがNULL値を許可する場合のロード結果 | 宛先カラムがNULL値を許可しない場合のロード結果 |
| ------------------- | --------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| \N                 | NULL                                    | `NULL`値がロードされます。                            | エラーが報告されます。                                        |
| abc                 | NULL                                    | `NULL`値がロードされます。                            | エラーが報告されます。                                        |
| 2000                | NULL                                    | `NULL`値がロードされます。                            | エラーが報告されます。                                        |
| 1                   | 1                                       | `1`値がロードされます。                               | `1`値がロードされます。                                     |

### 厳密モード有効時の最終ロードデータ

| ソースカラム値 | TINYINTへの変換後のカラム値 | 宛先カラムがNULL値を許可する場合のロード結果       | 宛先カラムがNULL値を許可しない場合のロード結果 |
| ------------------- | --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| \N                 | NULL                                    | `NULL`値がロードされます。                                  | エラーが報告されます。                                        |
| abc                 | NULL                                    | `NULL`値は許可されていないため、フィルタリングされます。 | エラーが報告されます。                                        |
| 2000                | NULL                                    | `NULL`値は許可されていないため、フィルタリングされます。 | エラーが報告されます。                                        |
| 1                   | 1                                       | `1`値がロードされます。                                     | `1`値がロードされます。                                     |

## 厳密モードの設定

[Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、または[Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)ジョブを実行してデータをロードする場合、`strict_mode`パラメータを使用してロードジョブの厳密モードを設定します。有効な値は`true`と`false`です。デフォルト値は`false`です。`true`は厳密モードを有効にし、`false`は厳密モードを無効にします。

[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を実行してデータをロードする場合、`enable_insert_strict`セッション変数を使用して厳密モードを設定します。有効な値は`true`と`false`です。デフォルト値は`true`です。`true`は厳密モードを有効にし、`false`は厳密モードを無効にします。

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

上記のコードスニペットはHDFSを例に使用しています。Broker Loadの詳細な構文とパラメータについては、[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

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

上記のコードスニペットはApache Kafka®を例に使用しています。Routine Loadの詳細な構文とパラメータについては、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

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

上記のコードスニペットはHDFSを例に使用しています。Spark Loadの詳細な構文とパラメータについては、[SPARK LOAD](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)を参照してください。

### INSERT

```SQL
SET enable_insert_strict = {true | false};
INSERT INTO <table_name> ...
```

INSERTの詳細な構文とパラメータについては、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。
