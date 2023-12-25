---
displayed_sidebar: English
---

# ALTER ROUTINE LOAD

## 説明

このコマンドは、作成されたルーチンロードジョブを変更するために使用されます。変更するジョブは、**PAUSED** 状態でなければなりません。[PAUSE](./PAUSE_ROUTINE_LOAD.md)（PAUSE ROUTINE LOAD）コマンドを実行して、ロードジョブを一時停止してから `ALTER ROUTINE LOAD` 操作をそのジョブに対して実行することができます。

## 構文

> 注: 角括弧 [] で囲まれた内容を指定する必要はありません。

```SQL
ALTER ROUTINE LOAD FOR [db.]<job_name>

[load_properties]

[job_properties]

FROM data_source

[data_source_properties]
```

## **パラメーター**

- **`[db.]<job_name>`**

変更したいジョブの名前です。

- **load_properties**

インポートされるデータのプロパティです。

構文：

```SQL
[COLUMNS TERMINATED BY '<terminator>'],

[COLUMNS ([<column_name> [, ...] ] [, column_assignment [, ...] ] )],

[WHERE <expr>],

[PARTITION ([ <partition_name> [, ...] ])]



column_assignment:

<column_name> = column_expression
```

1. 列区切り文字を指定します。

   インポートしたいCSVデータの列区切り文字を指定できます。例えば、コンマ (,) を列区切り文字として使用することができます。

    ```SQL
    COLUMNS TERMINATED BY ","
    ```

   デフォルトの区切り文字は `\t` です。

2. 列マッピングを指定します。

   ソーステーブルとデスティネーションテーブルの列のマッピングを指定し、派生列がどのように生成されるかを定義します。

   - マッピングされた列

   ソーステーブルのどの列がデスティネーションテーブルのどの列に対応するかを順番に指定します。列をスキップしたい場合は、存在しない列名を指定することができます。例えば、デスティネーションテーブルには k1、k2、v1 の3つの列があり、ソーステーブルには4つの列があり、そのうちの1番目、2番目、4番目の列が k2、k1、v1 に対応している場合、コードは以下のようになります。

    ```SQL
    COLUMNS (k2, k1, xxx, v1)
    ```

   `xxx` は存在しない列で、ソーステーブルの3番目の列をスキップするために使用されます。

   - 派生列

   `col_name = expr` 形式の列は派生列です。これらの列は `expr` を使用して生成されます。派生列は通常、マッピングされた列の後に配置されます。これは必須のルールではありませんが、StarRocksは常にマッピングされた列を派生列より先に解析します。デスティネーションテーブルに4番目の列 v2 があり、これは k1 と k2 を合計して生成されると仮定します。コードは以下のようになります。

    ```plaintext
    COLUMNS (k2, k1, xxx, v1, v2 = k1 + k2);
    ```

   CSVデータの場合、COLUMNSに指定されたマッピングされた列の数はCSVファイルの列の数と一致する必要があります。

3. フィルタ条件を指定します。

   不要なデータをフィルタリングするための条件を指定できます。フィルタリングされる列はマッピングされた列または派生列です。例えば、k1 の値が 100 より大きく、k2 の値が 1000 である列からデータをインポートする必要がある場合、以下のようにコードを記述できます。

    ```plaintext
    WHERE k1 > 100 AND k2 = 1000
    ```

4. データをインポートするパーティションを指定します。

   パーティションが指定されていない場合、データはCSVデータのパーティションキー値に基づいてStarRocksのパーティションに自動的にインポートされます。例えば：

    ```plaintext
    PARTITION(p1, p2, p3)
    ```

- **job_properties**

変更したいジョブパラメータです。現在、以下のパラメータを変更することができます：

1. `desired_concurrent_number`
2. `max_error_number`
3. `max_batch_interval`
4. `max_batch_rows`
5. `max_batch_size`
6. `jsonpaths`
7. `json_root`
8. `strip_outer_array`
9. `strict_mode`
10. `timezone`

- **data_source**

データソースのタイプです。現在、Kafkaデータソースのみがサポートされています。

- **data_source_properties**

データソースのプロパティです。以下のプロパティがサポートされています：

1. `kafka_partitions`

   消費されたKafkaパーティションのみを変更できます。

2. `kafka_offsets`

   まだ消費されていないパーティションオフセットのみを変更できます。

3. カスタムプロパティ、例えば `property.group.id` や `property.group.id`

> `kafka_partitions` で指定できるのは消費されたKafkaパーティションのみです。`kafka_offsets` で指定できるのはまだ消費されていないパーティションオフセットのみです。

## 例

例 1: `desired_concurrent_number` の値を 1 に変更します。このパラメータは、Kafkaデータを消費するために使用されるジョブの並列性を指定します。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

PROPERTIES

(

    "desired_concurrent_number" = "1"

);
```

例 2: `desired_concurrent_number` の値を 10 に変更し、パーティションオフセットとグループIDを変更します。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

PROPERTIES

(

    "desired_concurrent_number" = "10"

)

FROM kafka

(

    "kafka_partitions" = "0, 1, 2",

    "kafka_offsets" = "100, 200, 100",

    "property.group.id" = "new_group"

);
```

例 3: フィルタ条件を `a > 0` に変更し、宛先パーティションを `p1` に設定します。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

WHERE a > 0

PARTITION (p1)
```
