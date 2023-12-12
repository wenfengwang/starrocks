---
displayed_sidebar: "Japanese"
---

# ALTER ROUTINE LOAD

## 説明

このコマンドは作成されたRoutine Loadジョブを変更するために使用されます。変更するジョブは**PAUSED**の状態である必要があります。ローディングジョブを一時停止させ、その後ジョブに`ALTER ROUTINE LOAD`操作を実行するために[PASUME](./PAUSE_ROUTINE_LOAD.md)(PAUSE ROUTINE LOAD)コマンドを実行できます。

## 構文

> 注意：角かっこ[]で囲まれた内容を指定する必要はありません。

```SQL
ALTER ROUTINE LOAD FOR [db.]<job_name>

[load_properties]

[job_properties]

FROM data_source

[data_source_properties]
```

## **パラメータ**

- **`[db.]<job_name>`**

変更したいジョブの名前。

- **load_properties**

インポートするデータのプロパティ。

構文:

```SQL
[COLUMNS TERMINATED BY '<terminator>'],

[COLUMNS ([<column_name> [, ...] ] [, column_assignment [, ...] ] )],

[WHERE <expr>],

[PARTITION ([ <partition_name> [, ...] ])]



column_assignment:

<column_name> = column_expression
```

1. カラムのセパレータを指定します。

   インポートしたいCSVデータのためのカラムセパレータを指定できます。例えば、カンマ(,)をカラムセパレータとして使用できます。

    ```SQL
    COLUMNS TERMINATED BY ","
    ```

   デフォルトのセパレータは`\t`です。

2. カラムのマッピングを指定します。

   ソースとデスティネーションテーブルのカラムのマッピングを指定し、派生カラムの作成方法を定義します。

   - マップされたカラム

   ソーステーブルのどのカラムが、デスティネーションテーブルのどのカラムに対応するかを順番に指定します。カラムをスキップしたい場合は、存在しないカラム名を指定できます。例えば、デスティネーションテーブルにはk1、k2、v1の3つのカラムがあります。ソーステーブルには4つのカラムがあり、その中で1番目、2番目、4番目のカラムがk2、k1、v1に対応しています。以下のようにコードを書くことができます。

    ```SQL
    COLUMNS (k2, k1, xxx, v1)
    ```

   `xxx` は存在しないカラムです。ソーステーブルの3番目のカラムをスキップするために使用されます。

   - 派生カラム

   `col_name = expr` で表されるカラムは派生カラムです。これらのカラムは `expr` を使用して生成されます。派生カラムは通常、マップされたカラムの後に配置されます。これは強制的なルールではありませんが、StarRocksは常に派生カラムの前にマップされたカラムを解析します。例えば、デスティネーションテーブルにはk1とk2を足したものである4番目のカラムv2があるとします。以下のようにコードを書くことができます。

    ```plaintext
    COLUMNS (k2, k1, xxx, v1, v2 = k1 + k2);
    ```

   CSVデータの場合、COLUMNS内のマップされたカラムの数はCSVファイル内のカラムの数に一致する必要があります。

3. フィルタ条件を指定します。

   不要なカラムをフィルタリングするためのフィルタ条件を指定できます。フィルタカラムはマップされたカラムまたは派生カラムであることができます。例えば、k1の値が100より大きく、k2の値が1000に等しい列からデータをインポートする必要がある場合、以下のようにコードを書くことができます。

    ```plaintext
    WHERE k1 > 100 and k2 = 1000
    ```

4. データをインポートするパーティションを指定します。

   パーティションが指定されていない場合、データはCSVデータ内のパーティションキーの値に基づいて自動的にStarRocksパーティションにインポートされます。例:

    ```plaintext
    PARTITION(p1, p2, p3)
    ```

- **job_properties**

変更したいジョブのパラメータ。現在、以下のパラメータを変更できます：

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

データソースのタイプ。現在、Kafkaデータソースのみがサポートされています。

- **data_source_properties**

データソースのプロパティ。以下のプロパティがサポートされています：

1. `kafka_partitions`

   消費されたKafkaパーティションのみを変更できます。

2. `kafka_offsets`

   まだ消費されていないパーティションオフセットを変更できます。

3. `property.group.id` や `property.group.id` などのカスタムプロパティ

> `kafka_partitions` では、すでに消費されたKafkaパーティションのみを指定できます。`kafka_offsets` では、まだ消費されていないパーティションオフセットのみを指定できます。

## 例

例1：`desired_concurrent_number` の値を1に変更します。このパラメータは、Kafkaデータを消費するために使用されるジョブの並列処理を指定します。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

PROPERTIES

(

    "desired_concurrent_number" = "1"

);
```

例2：`desired_concurrent_number` の値を10に変更し、パーティションオフセットとグループIDを変更します。

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

例3：フィルタ条件を `a > 0` に変更し、宛先パーティションを `p1` に設定します。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

WHERE a > 0

PARTITION (p1)
```