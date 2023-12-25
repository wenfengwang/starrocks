---
displayed_sidebar: Chinese
---

# ストリクトモード

ストリクトモード (Strict Mode) はインポート操作のオプションで、設定によって StarRocks がデータをインポートする際の挙動や、最終的に StarRocks にインポートされるデータに影響を与えます。

このドキュメントでは、ストリクトモードとは何か、およびストリクトモードの設定方法について説明します。

## ストリクトモードの紹介

インポートプロセス中、元の列とターゲット列のデータ型が完全に一致しない場合があります。そのような場合、StarRocks はデータ型が一致しない元の列の値を変換します。変換プロセス中に、フィールドタイプの不一致やフィールドの長さ超過などの変換エラーが発生することがあります。変換に失敗したフィールドを「エラーフィールド」と呼び、エラーフィールドを含むデータ行を「エラーのあるデータ行」と呼びます。ストリクトモードは、これらのエラーのあるデータ行をインポートプロセス中にフィルタリングするかどうかを制御するために使用されます。

ストリクトモードのフィルタリング戦略は以下の通りです：

- ストリクトモードを有効にすると、StarRocks はエラーのあるデータ行をフィルタリングし、正しいデータ行のみをインポートし、エラーデータの詳細を返します。

- ストリクトモードを無効にすると、StarRocks は変換に失敗したエラーフィールドを `NULL` 値に変換し、これら `NULL` 値を含むエラーのあるデータ行を正しいデータ行と共にインポートします。

以下の二点に注意してください：

- 実際のインポートプロセス中、正しいデータ行とエラーのあるデータ行の両方に `NULL` 値が存在する可能性があります。ターゲット列が `NULL` 値を許可しない場合、StarRocks はエラーを報告し、これら `NULL` 値を含むデータ行をフィルタリングします。

- [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) および [Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) において、インポートジョブが許容できるデータ品質不良によりフィルタリングされるエラーのあるデータ行の最大比率は、ジョブのオプションパラメータ `max_filter_ratio` によって制御されます。[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 方式のインポートは現在 `max_filter_ratio` パラメータをサポートしていません。

以下は CSV 形式のデータファイルを例にして、ストリクトモードの効果を説明します。ターゲット列のデータ型が TINYINT [-128, 127] だと仮定します。`\N`（null 値を表す）、`abc`、`2000`、`1` の四つの元の列値を例にします：

- 元の列値 `\N` は TINYINT 型に変換されると `NULL` になります。

  > **注記**
  >
  > 元の列値 `\N` はターゲット列のデータ型に関係なく、変換後は常に `NULL` になります。

- 元の列値 `abc` はターゲット列のデータ型 TINYINT と一致しないため、変換に失敗し `NULL` になります。

- 元の列値 `2000` はターゲット列のデータ型 TINYINT の許容範囲外であるため、変換に失敗し `NULL` になります。

- 元の列値 `1` は正常に TINYINT 型の数値 `1` に変換できます。

ストリクトモードを無効にすると、すべてのデータ行がインポートされます。ストリクトモードを有効にすると、`\N` または `1` を含むデータ行のみがインポートされ、`abc` または `2000` を含むデータ行はフィルタリングされます。フィルタリングされたデータ行はインポートジョブのパラメータ `max_filter_ratio` が許容するデータ品質不良によりフィルタリングされるデータ行に記録されます。

**ストリクトモードを無効にした場合のインポート結果**

| 元の列値の例 | TINYINT 型に変換後の列値 | ターゲット列が NULL 値を許可する場合のインポート結果 | ターゲット列が NULL 値を許可しない場合のインポート結果 |
| ------------ | --------------------------- | -------------------------- | ---------------------------- |
| \N          | NULL                        | `NULL` 値をインポート。           | エラー。                       |
| abc          | NULL                        | `NULL` 値をインポート。           | エラー。                       |
| 2000         | NULL                        | `NULL` 値をインポート。           | エラー。                       |
| 1            | 1                           | `1` をインポート。                 | `1` をインポート。                   |

**ストリクトモードを有効にした場合のインポート結果**

| 元の列値の例 | TINYINT 型に変換後の列値 | ターゲット列が NULL 値を許可する場合のインポート結果 | ターゲット列が NULL 値を許可しない場合のインポート結果 |
| ------------ | --------------------------- | ------------------------ | -------------------------- |
| \N          | NULL                        | `NULL` 値をインポート。         | エラー。                     |
| abc          | NULL                        | `NULL` 値は無効で、フィルタリングされます。  | エラー。                     |
| 2000         | NULL                        | `NULL` 値は無効で、フィルタリングされます。  | エラー。                     |
| 1            | 1                           | `1` をインポート。               | `1` をインポート。                 |

## ストリクトモードの設定

[Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、および [Spark Load](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) を使用してデータインポートを実行する際には、パラメータ `strict_mode` を通じてストリクトモードを設定する必要があります。パラメータの値は `true` または `false` です。デフォルト値は `false` です。`true` は有効、`false` は無効を意味します。

[INSERT](../../loading/InsertInto.md) を使用してデータインポートを実行する際には、セッション変数 `enable_insert_strict` を通じてストリクトモードを設定する必要があります。変数の値は `true` または `false` です。デフォルト値は `true` です。`true` は有効、`false` は無効を意味します。

以下は、異なるインポート方法を使用する際のストリクトモードの設定方法を説明します。

### Stream Load

```Bash
curl --location-trusted -u <username>:<password> \
    -H "strict_mode: {true | false}" \
    -T <file_name> -XPUT \
    http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

Stream Load の構文とパラメータの説明については、[STREAM LOAD](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) を参照してください。

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

ここでは HDFS データソースを例にしています。Broker Load の構文とパラメータの説明については、[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

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

ここでは Apache Kafka® データソースを例にしています。Routine Load の構文とパラメータの説明については、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

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

ここでは HDFS データソースを例にしています。Spark Load の構文とパラメータの説明については、[SPARK LOAD](../../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) を参照してください。

### INSERT

```SQL
SET enable_insert_strict = {true | false};
INSERT INTO <table_name> ...
```

INSERT の構文とパラメータの説明については、[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。
