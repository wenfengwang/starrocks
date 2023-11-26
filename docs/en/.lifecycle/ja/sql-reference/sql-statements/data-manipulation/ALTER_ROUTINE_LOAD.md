---
displayed_sidebar: "Japanese"
---

# ALTER ROUTINE LOAD（ルーチンロードの変更）

## 説明

このコマンドは、作成されたルーチンロードジョブを変更するために使用されます。変更するジョブは**PAUSED**状態である必要があります。ロードジョブを一時停止させ、ジョブに`ALTER ROUTINE LOAD`操作を実行するために[PASUME](./PAUSE_ROUTINE_LOAD.md)(PAUSE ROUTINE LOAD)コマンドを実行できます。

## 構文

> 注意: 角括弧[]で囲まれたコンテンツを指定する必要はありません。

```SQL
ALTER ROUTINE LOAD FOR [db.]<job_name>

[load_properties]

[job_properties]

FROM data_source

[data_source_properties]
```

## **パラメータ**

- **`[db.]<job_name>`**

変更したいジョブの名前です。

- **load_properties**

インポートするデータのプロパティです。

構文:

```SQL
[COLUMNS TERMINATED BY '<terminator>'],

[COLUMNS ([<column_name> [, ...] ] [, column_assignment [, ...] ] )],

[WHERE <expr>],

[PARTITION ([ <partition_name> [, ...] ])]



column_assignment:

<column_name> = column_expression
```

1. カラムの区切り文字を指定します。

   インポートしたいCSVデータのカラムの区切り文字を指定できます。例えば、カンマ(,)をカラムの区切り文字として使用することができます。

    ```SQL
    COLUMNS TERMINATED BY ","
    ```

   デフォルトの区切り文字は`\t`です。

2. カラムのマッピングを指定します。

   ソーステーブルとデスティネーションテーブルのカラムのマッピングを指定し、派生カラムの生成方法を定義します。

   - マッピングされたカラム

   ソーステーブルのどのカラムがデスティネーションテーブルのどのカラムに対応するかを順番に指定します。カラムをスキップしたい場合は、存在しないカラム名を指定することができます。例えば、デスティネーションテーブルにはk1、k2、v1の3つのカラムがあります。ソーステーブルには4つのカラムがあり、そのうちの1つ目、2つ目、4つ目のカラムがk2、k1、v1に対応している場合、以下のようにコードを記述することができます。

    ```SQL
    COLUMNS (k2, k1, xxx, v1)
    ```

   `xxx`は存在しないカラムです。ソーステーブルの3つ目のカラムをスキップするために使用されます。

   - 派生カラム

   `col_name = expr`と表されるカラムは派生カラムです。これらのカラムは`expr`を使用して生成されます。派生カラムは通常、マッピングされたカラムの後に配置されます。これは必須のルールではありませんが、StarRocksは常に派生カラムよりも先にマッピングされたカラムを解析します。デスティネーションテーブルにk1とk2を足した値で生成される4番目のカラムv2があると仮定します。以下のようにコードを記述することができます。

    ```plaintext
    COLUMNS (k2, k1, xxx, v1, v2 = k1 + k2);
    ```

   CSVデータの場合、COLUMNS内のマッピングされたカラムの数はCSVファイルのカラムの数と一致する必要があります。

3. フィルタ条件を指定します。

   不要なカラムをフィルタリングするためのフィルタ条件を指定できます。フィルタカラムはマッピングされたカラムまたは派生カラムのいずれかです。例えば、k1の値が100より大きく、k2の値が1000と等しいカラムからデータをインポートする必要がある場合、以下のようにコードを記述することができます。

    ```plaintext
    WHERE k1 > 100 and k2 = 1000
    ```

4. データをインポートするパーティションを指定します。

   パーティションが指定されていない場合、データはCSVデータのパーティションキーの値に基づいて自動的にStarRocksパーティションにインポートされます。例:

    ```plaintext
    PARTITION(p1, p2, p3)
    ```

- **job_properties**

変更したいジョブのパラメータです。現在、以下のパラメータを変更することができます:

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

データソースのプロパティです。以下のプロパティがサポートされています:

1. `kafka_partitions`

   消費されたKafkaパーティションのみを変更することができます。

2. `kafka_offsets`

   まだ消費されていないパーティションオフセットのみを変更することができます。

3. `property.group.id`や`property.group.id`などのカスタムプロパティ

> `kafka_partitions`にはまだ消費されていないKafkaパーティションのみを指定できます。`kafka_offsets`にはまだ消費されていないパーティションオフセットのみを指定できます。

## 例

例1: `desired_concurrent_number`の値を1に変更します。このパラメータはKafkaデータを消費するために使用されるジョブの並列性を指定します。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

PROPERTIES

(

    "desired_concurrent_number" = "1"

);
```

例2: `desired_concurrent_number`の値を10に変更し、パーティションオフセットとグループIDを変更します。

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

例3: フィルタ条件を`a > 0`に変更し、デスティネーションパーティションを`p1`に設定します。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

WHERE a > 0

PARTITION (p1)
```
