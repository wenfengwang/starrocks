---
displayed_sidebar: Chinese
---

# ALTER ROUTINE LOAD

## 機能

この構文は、すでに作成された定期的なインポートジョブを変更するために使用され、**PAUSED** 状態のジョブのみを変更することができます。[PAUSE](./PAUSE_ROUTINE_LOAD.md) コマンドを使用してインポートタスクを一時停止し、`ALTER ROUTINE LOAD` 操作を行うことができます。

## 文法

```sql
ALTER ROUTINE LOAD FOR [db.]job_name
[load_properties]
[job_properties]
FROM data_source
[data_source_properties]
```

1. **[db.] job_name**

    変更するジョブ名を指定します。

2. **load_properties**

    インポートデータを記述するために使用されます。文法：

    ```sql
    [COLUMNS TERMINATED BY '<terminator>'],
    [COLUMNS ([<column_name> [, ...] ] [, column_assignment [, ...] ] )],
    [WHERE <expr>],
    [PARTITION ([ <partition_name> [, ...] ])]

    column_assignment:
    <column_name> = column_expression
    ```

    1. 列区切り文字を設定

        CSV形式のデータに対して、列区切り文字を指定することができます。例えば、列区切り文字をコンマ "," に指定する場合は以下のようにします。

        ```sql
        COLUMNS TERMINATED BY ","
        ```

        デフォルトは：\t。

    2. 列のマッピング関係を指定

        ソースデータの列のマッピング関係を指定し、派生列の生成方法を定義します。

        1. マッピング列：

            ソースデータの各列が、ターゲットテーブルのどの列に対応するかを順番に指定します。スキップしたい列には、存在しない列名を指定することができます。
            例えば、ターゲットテーブルに k1, k2, v1 の3列があり、ソースデータには4列あり、そのうち1、2、4列目がそれぞれ k2, k1, v1 に対応している場合、以下のように記述します：

            ```SQL
            COLUMNS (k2, k1, xxx, v1)
            ```

            ここで xxx は存在しない列で、ソースデータの3列目をスキップするために使用されます。

        2. 派生列：

            col_name = expr の形式で表される列を派生列と呼びます。つまり、expr を計算してターゲットテーブルの対応する列の値を導き出すことができます。
            派生列は通常、マッピング列の後に配置されますが、これは強制的な規定ではありませんが、StarRocks は常にマッピング列を先に解析し、その後で派生列を解析します。
            前の例に続いて、ターゲットテーブルに v2 という4列目があり、v2 は k1 と k2 の和によって生成される場合、以下のように記述できます：

            ```plain text
            COLUMNS (k2, k1, xxx, v1, v2 = k1 + k2);
            ```

        CSV形式のデータに対しては、COLUMNS のマッピング列の数はデータの列の数と一致している必要があります。

    3. フィルタ条件を指定

        不要な列をフィルタリングするための条件を指定するために使用されます。フィルタリング列はマッピング列または派生列であることができます。
        例えば、k1 が 100 より大きく、k2 が 1000 に等しい列のみをインポートしたい場合は、以下のように記述します：

        ```plain text
        WHERE k1 > 100 and k2 = 1000
        ```

    4. インポートするパーティションを指定

        **インポート先テーブル** のどの partition にインポートするかを指定します。指定しない場合は、対応する partition に自動的にインポートされます。
        例：

        ```plain text
        PARTITION(p1, p2, p3)
        ```

3. **job_properties**

    変更する必要のあるジョブパラメータを指定します。現在、以下のパラメータの変更のみをサポートしています：

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

4. **data_source**

    データソースのタイプ。現在サポートされているのは：

    KAFKA

5. **data_source_properties**

    データソースの関連属性。現在、以下のみをサポートしています：
    1. `kafka_partitions`
    2. `kafka_offsets`
    3. カスタム property、例えば `property.group.id`

注意：

`kafka_partitions` と `kafka_offsets` は、消費待ちの Kafka Partition の Offset を変更するために使用され、現在消費されている partition のみを変更することができます。新しい partition を追加することはできません。

## 例

1. `desired_concurrent_number` を 1 に変更し、このパラメータを調整して Kafka の消費並列度を調整します。詳細な調整方法については、[並列度を調整して Routine load の Kafka データ消費速度を上げる](https://forum.starrocks.com/t/topic/1675) を参照してください。

    ```sql
    ALTER ROUTINE LOAD FOR db1.label1
    PROPERTIES
    (
        "desired_concurrent_number" = "1"
    );
    ```

2. `desired_concurrent_number` を 10 に変更し、partition の offset を変更し、group id を変更します。

    ```sql
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

3. インポートのフィルタ条件を `a > 0` に変更し、インポートする partition を `p1` に指定します。

    ```sql
    ALTER ROUTINE LOAD FOR db1.label1
    WHERE a > 0
    PARTITION (p1)
    ```
