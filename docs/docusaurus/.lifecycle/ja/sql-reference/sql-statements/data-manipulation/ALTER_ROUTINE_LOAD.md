---
displayed_sidebar: "Japanese"
---

# ALTER ROUTINE LOAD（ルーチンロードを変更する）

## 説明

このコマンドは、作成されたルーチンロードのジョブを修正するために使用されます。修正するジョブは**PAUSED** ステートである必要があります。ロードジョブを一時停止させ、その後ジョブに対して`ALTER ROUTINE LOAD`操作を行うために[PASUME](./PAUSE_ROUTINE_LOAD.md)(PAUSE ROUTINE LOAD)コマンドを実行できます。

## 構文

> 注：角括弧[]で囲まれた内容を指定する必要はありません。

```SQL
ALTER ROUTINE LOAD FOR [db.]<job_name>

[load_properties]

[job_properties]

FROM data_source

[data_source_properties]
```

## **パラメータ**

- **`[db.]<job_name>`**

修正したいジョブの名前です。

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

1. カラムセパレータを指定する。

   インポートしたいCSVデータのためのカラムセパレータを指定できます。たとえば、カンマ(,)をカラムセパレータとして使用できます。

    ```SQL
    COLUMNS TERMINATED BY ","
    ```

   デフォルトのセパレータは `\t` です。

2. カラムマッピングを指定する。

   ソーステーブルと宛先テーブルのカラムのマッピングを指定し、導出列の生成方法を定義します。

   - マッピングされたカラム

   ソーステーブルのどのカラムが宛先テーブルのどのカラムに対応するかを指定します。特定のカラムをスキップしたい場合は、存在しないカラム名を指定できます。たとえば、宛先テーブルには k1、k2、v1 の 3 つのカラムがあります。ソーステーブルには 4 つのカラムがあり、そのうちの 1 番目、2 番目、4 番目の列が k2、k1、v1 に対応しています。次のようにコードを記述できます。

    ```SQL
    COLUMNS (k2, k1, xxx, v1)
    ```

   `xxx` は存在しない列です。ソーステーブルの 3 番目の列をスキップするために使用されます。

   - 導出列

   `col_name = expr` で表される列は導出列です。これらの列は `expr` を使用して生成されます。通常、導出列はマッピングされた列の後に配置されます。これは強制的なルールではないが、StarRocksは常に導出列をマッピングされた列よりも先に解析します。たとえば、宛先テーブルに 4 番目の列 v2 があり、それは k1 と k2 を加算して生成されます。次のようにコードを記述できます。

    ```plaintext
    COLUMNS (k2, k1, xxx, v1, v2 = k1 + k2);
    ```

   CSV データの場合、COLUMNS内のマッピングされた列の数は、CSV ファイル内の列の数と一致している必要があります。

3. フィルタ条件を指定する。

   不要な列をフィルタリングするためのフィルタ条件を指定できます。フィルタ列は、マッピングされた列または導出列になります。たとえば、k1 の値が 100 より大きくて k2 の値が 1000 に等しい列のデータのみをインポートする必要がある場合、次のようにコードを記述できます。

    ```plaintext
    WHERE k1 > 100 and k2 = 1000
    ```

4. インポートしたいパーティションを指定する。

   パーティションが指定されていない場合、データは CSV データのパーティションキー値に基づいて自動的に StarRocks パーティションにインポートされます。例:

    ```plaintext
    PARTITION(p1, p2, p3)
    ```

- **job_properties**

修正したいジョブのパラメータ。現在、次のパラメータを修正できます:

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

データソースのタイプ。現在、Kafka データソースのみがサポートされています。

- **data_source_properties**

データソースのプロパティ。次のプロパティがサポートされています:

1. `kafka_partitions`

   すでに消費された Kafka パーティションを修正できます。

2. `kafka_offsets`

   まだ消費されていないパーティションオフセットを修正できます。

3. `property.group.id` などのカスタムプロパティ

> `kafka_partitions` ではすでに消費された Kafka パーティションのみを指定できます。`kafka_offsets` ではまだ消費されていないパーティションオフセットのみを指定できます。

## 例

例1: `desired_concurrent_number` の値を 1 に変更する。このパラメータは、Kafka データを消費するために使用されるジョブの並行性を指定します。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

PROPERTIES

(

    "desired_concurrent_number" = "1"

);
```

例2: `desired_concurrent_number` の値を 10 に変更し、パーティションオフセットとグループ ID を修正する。

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

例3: フィルタ条件を `a > 0` に変更し、宛先パーティションを `p1` に設定する。

```SQL
ALTER ROUTINE LOAD FOR db1.label1

WHERE a > 0

PARTITION (p1)
```