---
displayed_sidebar: "Japanese"
---

# 厳格モード

厳格モードは、データロードのために構成できるオプションのプロパティであります。それは読み込み動作と最終的な読み込まれたデータに影響を与えます。

このトピックでは、厳格モードとその設定方法について紹介します。

## 厳格モードの理解

データの読み込み中、ソース列のデータ型が宛先列のデータ型と完全に一致しない場合があります。このような場合、StarRocks は、データ型が一致しないソース列値に対して変換を行います。データの変換には、一致しないフィールドのデータ型やフィールド長のオーバーフローなど、さまざまな問題が原因で失敗することがあります。適切に変換できないソース列値は、無資格の列値であり、それらの無資格の列値を含むソース行は「無資格の行」と呼ばれます。厳格モードは、データの読み込み中に無資格の行をフィルタリングするかどうかを制御するために使用されます。

厳格モードの動作は次のとおりです。

- 厳格モードが有効になっている場合、StarRocks は、無資格の行のみを読み込みます。無資格の行をフィルタリングし、無資格の行に関する詳細を返します。
- 厳格モードが無効になっている場合、StarRocks は、無資格の列値を `NULL` に変換し、これらの `NULL` 値を含む無資格の行を適切に変換できる行とともに読み込みます。

次の点に注意してください。

- 実際のビジネスシナリオでは、有資格と無資格の行の両方に `NULL` 値が含まれる場合があります。宛先の列が `NULL` 値を許可していない場合、StarRocks はエラーを報告し、`NULL` 値を含む行をフィルタリングします。

- [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、または [Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) ジョブでフィルタリングできる無資格の行の最大割合は、オプションのジョブプロパティ `max_filter_ratio` で制御されます。[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) は `max_filter_ratio` プロパティの設定をサポートしていません。

例えば、CSV 形式のデータファイルから StarRocks テーブルに、それぞれ `\N`（`NULL` 値を示す）, `abc`, `2000`, `1` の値を保持する 4 行を読み込む場合、宛先の StarRocks テーブル列のデータ型が TINYINT [-128, 127] であるとします。

- ソース列値 `\N` は、TINYINT への変換時に `NULL` に処理されます。

  > **注意**
  >
  > `\N` は、宛先のデータ型に関係なく、常に `NULL` に処理されます。

- ソース列値 `abc` は、そのデータ型が TINYINT でないため、変換に失敗し、`NULL` に処理されます。

- ソース列値 `2000` は、TINYINT でサポートされている範囲を超えているため、変換に失敗し、`NULL` に処理されます。

- ソース列値 `1` は、TINYINT 型の値 `1` に正しく変換されます。

厳格モードが無効の場合、StarRocks は 4 つの行すべてを読み込みます。

厳格モードが有効の場合、StarRocks は、`\N` や `1` を含む行のみを読み込み、`abc` や `2000` を含む行をフィルタリングします。フィルタリングされた行は、`max_filter_ratio` パラメータによって指定された不適切なデータ品質の最大割合に対してカウントされます。

### 厳格モードが無効の場合の最終的な読み込まれたデータ

| ソース列値 | TINYINT への変換後の列値 | 宛先列が NULL 値を許可している場合の読み込み結果 | 宛先列が NULL 値を許可していない場合の読み込み結果 |
| --------------- | ---------------------------------------- | ------------------------------------------------ | ------------------------------------------------------ |
| \N        | NULL                                 | 値 `NULL` が読み込まれます。            | エラーが報告されます。 |
| abc     | NULL                                 | 値 `NULL` が読み込まれます。              | エラーが報告されます。       |
| 2000  | NULL                                 | 値 `NULL` が読み込まれます。             | エラーが報告されます。       |
| 1         | 1                                      | 値 `1` が読み込まれます。                 | 値 `1` が読み込まれます。      |

### 厳格モードが有効の場合の最終的な読み込まれたデータ

| ソース列値 | TINYINT への変換後の列値 | 宛先列が NULL 値を許可している場合の読み込み結果       | 宛先列が NULL 値を許可していない場合の読み込み結果 |
| --------------- | ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| \N        | NULL                                 | 値 `NULL` が読み込まれます。                                          | エラーが報告されます。 |
| abc     | NULL                                 | 値 `NULL` は許可されていないため、フィルタリングされます。    | エラーが報告されます。 |
| 2000  | NULL                                 | 値 `NULL` は許可されていないため、フィルタリングされます。    | エラーが報告されます。 |
| 1         | 1                                      | 値 `1` が読み込まれます。                                             | 値 `1` が読み込まれます。      |

## 厳格モードの設定

データを読み込むために [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、または [Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) ジョブを実行する場合、読み込みジョブの厳格モードを設定するには `strict_mode` パラメータを使用します。有効な値は `true` と `false` です。デフォルト値は `false` です。値 `true` は厳格モードを有効にし、値 `false` は厳格モードを無効にします。

データを読み込むために [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を実行する場合、厳格モードを設定するために `enable_insert_strict` セッション変数を使用します。有効な値は `true` と `false` です。デフォルト値は `true` です。値 `true` は厳格モードを有効にし、値 `false` は厳格モードを無効にします。

以下に例を示します。

### Stream Load

```Bash
curl --location-trusted -u <username>:<password> \
    -H "strict_mode: {true | false}" \
    -T <file_name> -XPUT \
    http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

Stream Load の構文やパラメータの詳細については、[STREAM LOAD](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) を参照してください。

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

上記のコードスニペットでは HDFS を例にしています。Broker Load の構文やパラメータの詳細については、[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

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

上記のコードスニペットでは Apache Kafka® を例にしています。Routine Load の構文やパラメータの詳細については、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

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

上記のコードスニペットでは HDFS を例にしています。Spark Load の構文やパラメータの詳細については、[SPARK LOAD](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) を参照してください。

### INSERT

```SQL
SET enable_insert_strict = {true | false};
INSERT INTO <table_name> ...
```

INSERT の構文やパラメータの詳細については、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。