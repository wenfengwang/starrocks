```yaml
---
displayed_sidebar: "Japanese"
---

# 厳格モード

厳格モードはデータロードに設定できるオプションプロパティです。ロード動作と最終的なロードされたデータに影響を与えます。

このトピックでは、厳格モードとその設定方法について紹介します。

## 厳格モードの理解

データのロード中、ソース列のデータ型は、宛先列のデータ型と完全に一致しない場合があります。このような場合、StarRocksはデータ型が一致しないソース列の値に対して変換を行います。データの変換には、一致しないフィールドのデータ型やフィールド長のオーバーフローなどのさまざまな問題によって失敗する可能性があります。適切に変換できなかったソース列の値は、不適格な列値とされ、これらの不適格な列値を含むソース行は「不適格な行」と呼ばれます。厳格モードは、データのロード中に不適格な行をフィルタリングするかどうかを制御するために使用されます。

厳格モードの動作は、次のようになります。

- 厳格モードが有効になっている場合、StarRocksは有資格な行のみをロードします。不適格な行をフィルタリングし、不適格な行に関する詳細を返します。
- 厳格モードが無効になっている場合、StarRocksは不適格な列値を`NULL`に変換し、これらの`NULL`値を含む不適格な行を有資格な行と一緒にロードします。

次の点に注意してください。

- 実際のビジネスシナリオでは、有資格な行と不適格な行の両方に`NULL`値が含まれる場合があります。宛先列が`NULL`値を許可しない場合、StarRocksはエラーを報告し、`NULL`値を含む行をフィルタリングします。

- [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、または[Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)ジョブでフィルタリングできる不適格な行の最大割合は、オプションのジョブプロパティ`max_filter_ratio`によって制御されます。[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)では`max_filter_ratio`プロパティの設定はサポートされていません。

たとえば、CSV形式のデータファイルからStarRocksテーブルに、`\N`（`NULL`値を示す）を保持する4つの行、`abc`、`2000`、`1`の値をそれぞれ1つの列にロードしたい場合、宛先のStarRocksテーブル列のデータ型がTINYINT [-128, 127]であるとします。

- ソース列の値`\N`は、TINYINTに変換される際に`NULL`に処理されます。

  > **注意**
  >
  > 宛先のデータ型に関係なく、`\N`は常に`NULL`に処理されます。

- ソース列の値`abc`は、そのデータ型がTINYINTではないため変換が失敗するため、`NULL`に処理されます。

- ソース列の値`2000`は、TINYINTがサポートする範囲を超えているため、変換に失敗し、`NULL`に処理されます。

- ソース列の値`1`は、TINYINT型の値`1`に適切に変換できます。

厳格モードが無効になっている場合、StarRocksは4つの行すべてをロードします。

厳格モードが有効になっている場合、StarRocksは`\N`または`1`を保持する行のみをロードし、`abc`または`2000`を保持する行をフィルタリングします。不適格な行は`max_filter_ratio`パラメータによって指定された不十分なデータ品質の行の最大割合に対してカウントされます。

### 厳格モードが無効の最終的なロードデータ

| ソース列の値 | TINYINTへの変換時の列の値 | 宛先列がNULL値を許可する場合のロード結果 | 宛先列がNULL値を許可しない場合のロード結果 |
| ------------- | --------------------------- | ------------------------------------------- | ------------------------------------------- |
| \N            | NULL                        | 値`NULL`がロードされます。                 | エラーが報告されます。                      |
| abc           | NULL                        | 値`NULL`がロードされます。                 | エラーが報告されます。                      |
| 2000          | NULL                        | 値`NULL`がロードされます。                 | エラーが報告されます。                      |
| 1             | 1                           | 値`1`がロードされます。                   | 値`1`がロードされます。                     |

### 厳格モードが有効の最終的なロードデータ

| ソース列の値 | TINYINTへの変換時の列の値 | 宛先列がNULL値を許可する場合のロード結果 | 宛先列がNULL値を許可しない場合のロード結果 |
| ------------- | --------------------------- | ------------------------------------------- | ------------------------------------------- |
| \N            | NULL                        | 値`NULL`がロードされます。                 | エラーが報告されます。                      |
| abc           | NULL                        | 値`NULL`は許可されず、したがってフィルタリングされます。 | エラーが報告されます。                      |
| 2000          | NULL                        | 値`NULL`は許可されず、したがってフィルタリングされます。 | エラーが報告されます。                      |
| 1             | 1                           | 値`1`がロードされます。                   | 値`1`がロードされます。                     |

## 厳格モードの設定

データをロードするために[Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、または[Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)ジョブを実行する場合は、`strict_mode`パラメータを使用してロードジョブの厳格モードを設定します。有効な値は`true`と`false`です。デフォルト値は`false`です。値`true`は厳格モードを有効にし、値`false`は厳格モードを無効にします。

データをロードするために[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を実行する場合は、`enable_insert_strict`セッション変数を使用して厳格モードを設定します。有効な値は`true`と`false`です。デフォルト値は`true`です。値`true`は厳格モードを有効にし、値`false`は厳格モードを無効にします。

以下に具体的な例を示します。

### Stream Load

```Bash
curl --location-trusted -u <username>:<password> \
    -H "strict_mode: {true | false}" \
    -T <file_name> -XPUT \
    http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

Stream Loadの構文やパラメータの詳細については、[STREAM LOAD](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

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

上記のコードスニペットはHDFSを例にしています。Broker Loadの構文やパラメータの詳細については、[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

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

上記のコードスニペットはApache Kafka®を例にしています。Routine Loadの構文やパラメータの詳細については、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

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

上記のコードスニペットはHDFSを例にしています。Spark Loadの構文やパラメータの詳細については、[SPARK LOAD](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)を参照してください。

### INSERT

```SQL
SET enable_insert_strict = {true | false};
INSERT INTO <table_name> ...
```

INSERTの構文やパラメータの詳細については、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。
```