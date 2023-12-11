---
displayed_sidebar: "Japanese"
---

# ロードを通じたデータの変更

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

スターロックスで提供される[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)を使用すると、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、または[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) ジョブを実行して、StarRocksテーブルにデータ変更を加えることができます。これにより、挿入、更新、削除などのデータ変更が可能になります。ただし、プライマリキーテーブルでは、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)や[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用したデータの変更はサポートされていません。

スターロックスでは、部分更新と条件付き更新もサポートされています。

<InsertPrivNote />

このトピックでは、CSVデータを使用して、StarRocksテーブルにデータ変更を行う方法を説明します。利用するロード手法によって、サポートされるデータファイル形式は異なります。

> **注記**
>
> CSVデータを使用する場合、50バイトを超えないコンマ(,)、タブ、またはパイプ(|)などのUTF-8文字列をテキスト区切り文字として使用できます。

## 実装

スターロックスで提供されるプライマリキーテーブルは、UPSERTとDELETEの操作をサポートしており、INSERT操作とUPDATE操作を区別しません。

ロードジョブを作成する際、スターロックスでは、ジョブ作成ステートメントまたはコマンドに`__op`というフィールドを追加することをサポートしています。`__op`フィールドは、実行する操作の種類を指定するために使用されます。

> **注記**
>
> テーブルを作成する際、`__op`という列を追加する必要はありません。

`__op`フィールドの定義方法は、利用するロード手法によって異なります。

- Stream Loadを選択する場合、`columns`パラメータを使用して`__op`フィールドを定義します。

- Broker Loadを選択する場合、SET句を使用して`__op`フィールドを定義します。

- Routine Loadを選択する場合、`COLUMNS`列を使用して`__op`フィールドを定義します。

`__op`フィールドを追加するかどうかは、行いたいデータ変更に基づいて決定できます。`__op`フィールドを追加しない場合、操作の種類はUPSERTにデフォルトで設定されます。主なデータ変更シナリオは次のとおりです。

- ロードしたいデータファイルがUPSERT操作のみを含む場合、`__op`フィールドを追加する必要はありません。

- ロードしたいデータファイルが削除操作のみを含む場合、`__op`フィールドを追加し、操作タイプをDELETEとして指定する必要があります。

- ロードしたいデータファイルがUPSERTとDELETEの両方の操作を含む場合、`__op`フィールドを追加し、データファイルに値が`0`または`1`の列が含まれるようにする必要があります。`0`の値はUPSERT操作を示し、`1`の値はDELETE操作を示します。

## 使用上の注意

- データファイルの各行には同じ列数が含まれていることを確認してください。

- データ変更に関連する列には、プライマリキー列を含める必要があります。

## 必要条件

### Broker Load

[HDFSからデータをロード](../loading/hdfs_load.md)または[クラウドストレージからデータをロード](../loading/cloud_storage_load.md)の「バックグラウンド情報」セクションを参照してください。

### Routine Load

Routine Loadを選択する場合は、Apache Kafka®クラスタにトピックが作成されていることを確認してください。`topic1`、`topic2`、`topic3`、および`topic4`の4つのトピックが作成されていると仮定します。

## 基本操作

このセクションでは、StarRocksテーブルに対するデータ変更の例を提供します。詳細な構文やパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### UPSERT

ロードしたいデータファイルがUPSERT操作のみを含む場合、`__op`フィールドを追加する必要はありません。

> **注記**
>
> `__op`フィールドを追加する場合：
>
> - 操作タイプをUPSERTとして指定できます。
>
> - `__op`フィールドを空のままにすることもできます。なぜなら、操作の種類はUPSERTにデフォルトで設定されるからです。

#### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example1.csv`という名前のCSVファイルを作成します。このファイルには、順にユーザーID、ユーザー名、ユーザースコアを表す3つの列が含まれています。

      ```Plain
      101,Lily,100
      102,Rose,100
      ```

   b. `example1.csv`のデータをKafkaクラスタの`topic1`に公開します。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table1`という名前のプライマリキーテーブルを作成します。このテーブルには、`id`、`name`、`score`の3つの列が含まれており、`id`がプライマリキーです。

      ```SQL
      CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NOT NULL COMMENT "user name",
          `score` int(11) NOT NULL COMMENT "user score"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **注記**
      >
      > v2.5.7以降、StarRocksはテーブルの作成やパーティションの追加時にバケツ数（BUCKETS）を自動的に設定できます。バケツ数を手動で設定する必要はもはやありません。詳細については、[バケット数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table1`にレコードを挿入します。

      ```SQL
      INSERT INTO table1 VALUES
          (101, 'Lily',80);
      ```

#### データのロード

`example1.csv`内の`id`が`101`のレコードを`table1`に更新し、`example1.csv`内の`id`が`102`のレコードを`table1`に挿入するためのロードジョブを実行します。

- Stream Loadジョブを実行します。

  - `__op`フィールドを含めたくない場合、次のコマンドを実行してください:

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label1" \
        -H "column_separator:," \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

  - `__op`フィールドを含めたい場合、次のコマンドを実行してください:

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label2" \
        -H "column_separator:," \
        -H "columns:__op ='upsert'" \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

- Broker Loadジョブを実行します。

  - `__op`フィールドを含めたくない場合、次のコマンドを実行してください:

    ```SQL
    LOAD LABEL test_db.label1
    (
        data infile("hdfs://<hdfs_host>:<hdfs_port>/example1.csv")
        into table table1
        columns terminated by ","
        format as "csv"
    )
    with broker "broker1";
    ```

  - `__op`フィールドを含めたい場合、次のコマンドを実行してください:

    ```SQL
    LOAD LABEL test_db.label2
    (
        data infile("hdfs://<hdfs_host>:<hdfs_port>/example1.csv")
        into table table1
        columns terminated by ","
        format as "csv"
        set (__op = 'upsert')
    )
    with broker "broker1";
    ```

- Routine Loadジョブを実行します。

  - `__op`フィールドを含めたくない場合、次のコマンドを実行してください:

    ```SQL
    CREATE ROUTINE LOAD test_db.table1 ON table1
    COLUMNS TERMINATED BY ",",
    COLUMNS (id, name, score)
    PROPERTIES
    (
        "desired_concurrent_number" = "3",
        "max_batch_interval" = "20",
        "max_batch_rows"= "250000",
        "max_error_number" = "1000"
    )
    FROM KAFKA
    (
        "kafka_broker_list" ="<kafka_broker_host>:<kafka_broker_port>",
        "kafka_topic" = "test1",
        "property.kafka_default_offsets" ="OFFSET_BEGINNING"
    );
    ```

  - `__op`フィールドを含めたい場合、次のコマンドを実行してください:

    ```SQL
    CREATE ROUTINE LOAD test_db.table1 ON table1
    COLUMNS TERMINATED BY ",",
    COLUMNS (id, name, score, __op ='upsert')
    PROPERTIES
    (
        "desired_concurrent_number" = "3",
        "max_batch_interval" = "20",
        "max_batch_rows"= "250000",
        "max_error_number" = "1000"
    )
    FROM KAFKA
    (
        "kafka_broker_list" ="<kafka_broker_host>:<kafka_broker_port>",
        "kafka_topic" = "test1",
        "property.kafka_default_offsets" ="OFFSET_BEGINNING"
    );
    ```

#### クエリデータ

ロードが完了したら、`table1` のデータをクエリして、ロードが成功したかどうかを確認します。

```SQL
SELECT * FROM table1;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  101 | Lily |   100 |
|  102 | Rose |   100 |
+------+------+-------+
2 rows in set (0.02 sec)
```

前述のクエリ結果に示されているように、`example1.csv` の `id` が `101` のレコードは `table1` に更新され、`example1.csv` の `id` が `102` のレコードは `table1` に挿入されました。

### 削除

ロードするデータファイルが削除操作のみを含む場合は、`__op` フィールドを追加し、操作の種類を DELETE と指定する必要があります。

#### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに `example2.csv` という名前の CSV ファイルを作成します。このファイルには、ユーザー ID、ユーザー名、そしてユーザースコアを順に表す三つの列が含まれています。

      ```Plain
      101,Jack,100
      ```

   b. `example2.csv` のデータを Kafka クラスタの `topic2` にパブリッシュします。

2. StarRocks テーブルを準備します。

   a. StarRocks テーブル `test_db` に `table2` という名前のプライマリキー・テーブルを作成します。このテーブルには、`id`、`name`、そして`score` という三つの列が含まれており、`id` がプライマリキーです。

      ```SQL
      CREATE TABLE `table2`
            (
          `id` int(11) NOT NULL COMMENT "ユーザーID",
          `name` varchar(65533) NOT NULL COMMENT "ユーザー名",
          `score` int(11) NOT NULL COMMENT "ユーザースコア"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **注**
      >
      > v2.5.7 以降、StarRocks はテーブルの作成またはパーティションの追加時にバケット数 (BUCKETS) を自動的に設定できます。バケット数を手動で設定する必要はもはやありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table2` に2つのレコードを挿入します。

      ```SQL
      INSERT INTO table2 VALUES
      (101, 'Jack', 100),
      (102, 'Bob', 90);
      ```

#### データのロード

`example2.csv` の `id` が `101` のレコードを `table2` から削除するためのロードジョブを実行します。

- ストリームロードジョブを実行する。

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label3" \
      -H "column_separator:," \
      -H "columns:__op='delete'" \
      -T example2.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
  ```

- ブローカーロードジョブを実行する。

  ```SQL
  LOAD LABEL test_db.label3
  (
      data infile("hdfs://<hdfs_host>:<hdfs_port>/example2.csv")
      into table table2
      columns terminated by ","
      format as "csv"
      set (__op = 'delete')
  )
  with broker "broker1";  
  ```

- ルーチンロードジョブを実行する。

  ```SQL
  CREATE ROUTINE LOAD test_db.table2 ON table2
  COLUMNS(id, name, score, __op = 'delete')
  PROPERTIES
  (
      "desired_concurrent_number" = "3",
      "max_batch_interval" = "20",
      "max_batch_rows"= "250000",
      "max_error_number" = "1000"
  )
  FROM KAFKA
  (
      "kafka_broker_list" ="<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "test2",
      "property.kafka_default_offsets" ="OFFSET_BEGINNING"
  );
  ```

#### クエリデータ

ロードが完了したら、`table2` のデータをクエリして、ロードが成功したかどうかを確認します。

```SQL
SELECT * FROM table2;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  102 | Bob  |    90 |
+------+------+-------+
1 row in set (0.00 sec)
```

前述のクエリ結果に示されているように、`example2.csv` の `id` が `101` のレコードは `table2` から削除されました。

### UPSERT および DELETE

ロードするデータファイルが UPSERT および DELETE の両方の操作を含む場合は、`__op` フィールドを追加し、データファイルに値が `0` または `1` である列を含める必要があります。`0` の値は UPSERT 操作を示し、`1` の値は DELETE 操作を示します。

#### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに `example3.csv` という名前の CSV ファイルを作成します。このファイルには、ユーザー ID、ユーザー名、ユーザースコア、および操作タイプを順に表す四つの列が含まれています。

      ```Plain
      101,Tom,100,1
      102,Sam,70,0
      103,Stan,80,0
      ```

   b. `example3.csv` のデータを Kafka クラスタの `topic3` にパブリッシュします。

2. StarRocks テーブルを準備します。

   a. StarRocks データベース `test_db` に `table3` という名前のプライマリキーテーブルを作成します。このテーブルには、`id`、`name`、そして`score` という三つの列が含まれており、`id` がプライマリキーです。

      ```SQL
      CREATE TABLE `table3`
      (
          `id` int(11) NOT NULL COMMENT "ユーザーID",
          `name` varchar(65533) NOT NULL COMMENT "ユーザー名",
          `score` int(11) NOT NULL COMMENT "ユーザースコア"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **注**
      >
      > v2.5.7 以降、StarRocks はテーブルの作成またはパーティションの追加時にバケット数 (BUCKETS) を自動的に設定できます。バケット数を手動で設定する必要はもはやありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table3` に2つのレコードを挿入します。

      ```SQL
      INSERT INTO table3 VALUES
          (101, 'Tom', 100),
          (102, 'Sam', 90);
      ```

#### データのロード

`example3.csv` の `id` が `101` のレコードを `table3` から削除し、`example3.csv` の `id` が `102` のレコードを `table3` に更新し、そして `example3.csv` の `id` が `103` のレコードを `table3` に挿入するためのロードジョブを実行します。

- ストリームロードジョブを実行する。

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label4" \
      -H "column_separator:," \
      -H "columns: id, name, score, temp, __op = temp" \
      -T example3.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
  ```

  > **注**
  >
  > 前述の例では、`example3.csv` の操作タイプを表す4番目の列を一時的に`temp`という名前にし、`columns` パラメータを使用して `__op` フィールドを `temp` 列にマッピングすることで、`example3.csv` の4番目の列の値が `0` または `1` に応じて UPSERT 操作または DELETE 操作を行うかどうかを StarRocks が決定できるようにしています。

- ブローカーロードジョブを実行する。

  ```Bash
  LOAD LABEL test_db.label4
  (
      data infile("hdfs://<hdfs_host>:<hdfs_port>/example1.csv")
      into table table1
      columns terminated by ","
      format as "csv"
      (id, name, score, temp)
      set (__op=temp)
  )
  with broker "broker1";
  ```

- ルーチンロードジョブを実行する。

  ```SQL
  CREATE ROUTINE LOAD test_db.table3 ON table3
```
  COLUMNS(id, name, score, temp, __op = temp)
  PROPERTIES
  (
      "desired_concurrent_number" = "3",
      "max_batch_interval" = "20",
      "max_batch_rows"= "250000",
      "max_error_number" = "1000"
  )
  FROM KAFKA
  (
      "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "test3",
      "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  );
  ```

#### クエリデータ

ロードが完了したら、`table3` のデータをクエリして、ロードが成功したことを確認します。

```SQL
SELECT * FROM table3;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  102 | Sam  |    70 |
|  103 | Stan |    80 |
+------+------+-------+
2 行が選択されました (0.01 秒)
```

前述のクエリの結果によると、`example3.csv` の`id` が `101` のレコードは `table3` から削除され、`example3.csv` の`id` が `102` のレコードは `table3` に更新され、`example3.csv` の`id` が `103` のレコードは `table3` に挿入されました。

## 部分的な更新

v2.2 以降、StarRocks はプライマリキー テーブルの指定された列のみを更新する機能をサポートしています。このセクションでは、CSV を使用して部分的な更新を実行する方法について説明します。

> **NOTICE**
>
> 部分的な更新を実行する際、更新対象の行が存在しない場合、StarRocks は新しい行を挿入し、データが更新されなかったために空のフィールドにはデフォルト値が入ります。

### データの例

1. データファイルの準備

   a. ローカルファイルシステムに `example4.csv` という名前の CSV ファイルを作成します。このファイルは、ユーザー ID とユーザー名を順に表す 2 列で構成されています。

      ```Plain
      101,Lily
      102,Rose
      103,Alice
      ```

   b. `example4.csv` のデータを Kafka クラスタの `topic4` に発行します。

2. StarRocks テーブルの準備

   a. StarRocks データベース `test_db` に `table4` という名前のプライマリキー テーブルを作成します。このテーブルには `id`、`name`、`score` の 3 列があり、`id` がプライマリキーです。

      ```SQL
      CREATE TABLE `table4`
      (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NOT NULL COMMENT "user name",
          `score` int(11) NOT NULL COMMENT "user score"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **NOTE**
      >
      > v2.5.7 以降、StarRocks はテーブルを作成したりパーティションを追加したりする際に、自動的にバケット数 (BUCKETS) を設定できます。バケット数を手動で設定する必要はもうありません。詳細 は [データ分布を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets) を参照してください。

   b. `table4` にレコードを挿入します。

      ```SQL
      INSERT INTO table4 VALUES
          (101, 'Tom',80);
      ```

### データのロード

`example4.csv` の 2 列のデータを `table4` の `id` と `name` の列にアップデートするためのロードを実行します。

- ストリームロードを実行します:

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label7" -H "column_separator:," \
      -H "partial_update:true" \
      -H "columns:id,name" \
      -T example4.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
  ```

  > **NOTE**
  >
  > ストリームロードを選択した場合、`partial_update` パラメータを `true` に設定して部分的な更新機能を有効にする必要があります。さらに、更新する列を指定するために `columns` パラメータを使用する必要があります。

- ブローカーロードジョブを実行します:

  ```SQL
  LOAD LABEL test_db.table4
  (
      data infile("hdfs://<hdfs_host>:<hdfs_port>/example4.csv")
      into table table4
      format as "csv"
      (id, name)
  )
  WITH BROKER "broker1"
  PROPERTIES
  (
      "partial_update" = "true"
  );
  ```

  > **NOTE**
  >
  > ブローカーロードを選択した場合、`partial_update` パラメータを `true` に設定して部分的な更新機能を有効にする必要があります。さらに、更新する列を指定するために `column_list` パラメータを使用する必要があります。

- ルーチンロードジョブを実行します:

  ```SQL
  CREATE ROUTINE LOAD test_db.table4 on table4
  COLUMNS (id, name),
  COLUMNS TERMINATED BY ','
  PROPERTIES
  (
      "partial_update" = "true"
  )
  FROM KAFKA
  (
      "kafka_broker_list" ="<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "test4",
      "property.kafka_default_offsets" ="OFFSET_BEGINNING"
  );
  ```

  > **NOTE**
  >
  > ブローカーロードを選択した場合、`partial_update` パラメータを `true` に設定して部分的な更新機能を有効にする必要があります。さらに、更新する列を指定するために `COLUMNS` パラメータを使用する必要があります。

### データのクエリ

ロードが完了したら、`table4` のデータをクエリして、ロードが成功したことを確認します。

```SQL
SELECT * FROM table4;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|  102 | Rose  |     0 |
|  101 | Lily  |    80 |
|  103 | Alice |     0 |
+------+-------+-------+
3 行が選択されました (0.01 秒)
```

前述のクエリの結果によると、`example4.csv` の`id` が `101` のレコードが `table4` に更新され、`example4.csv` の`id` が `102` と `103` のレコードが `table4` に挿入されました。

## 条件付きの更新

StarRocks v2.5 以降、プライマリキー テーブルは条件付きの更新をサポートしています。指定された非プライマリキー 列を条件として指定することで、更新が有効になるかどうかを判断できます。つまり、指定された列において、ソースデータレコードが宛先データレコード以上の値を持つ場合にのみソースから宛先への更新が実行されます。

条件付きの更新機能はデータの不整合を解消するためのものです。ソースデータが不整合な場合、この機能を使用して新しいデータが古いデータによって上書きされることを防ぎます。

> **NOTICE**
>
> - 同じバッチのデータに異なる列を更新条件として指定することはできません。
> - DELETE 操作は条件付きの更新をサポートしません。
> - v3.1.3 より前のバージョンでは、部分的な更新と条件付きの更新を同時に使用することはできません。v3.1.3 以降、StarRocks は部分的な更新と条件付きの更新を同時に使用することができます。
> - 条件付きの更新は、ストリームロードとルーチンロードのみがサポートしています。

### データの例

1. データファイルを準備

   a. ローカルファイルシステムに `example5.csv` という名前の CSV ファイルを作成します。このファイルは、ユーザー ID、バージョン、およびユーザースコアを順に表す 3 列で構成されています。

      ```Plain
      101,1,100
      102,3,100
      ```

   b. `example5.csv` のデータを Kafka クラスタの `topic5` に発行します。

2. StarRocks テーブルを準備

   a. StarRocks データベース `test_db` に `table5` という名前のプライマリキー テーブルを作成します。このテーブルには `id`、`version`、`score` の 3 列があり、`id` がプライマリキーです。

      ```SQL
      CREATE TABLE `table5`
      (
          `id` int(11) NOT NULL COMMENT "user ID", 
          `version` int NOT NULL COMMENT "version",
          `score` int(11) NOT NULL COMMENT "user score"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`) DISTRIBUTED BY HASH(`id`);
      ```

      > **NOTE**
      >
      > v2.5.7 以降、StarRocks はテーブルを作成したりパーティションを追加したりする際に、自動的にバケット数 (BUCKETS) を設定できます。バケット数を手動で設定する必要はもうありません。詳細 は [データ分布を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets) を参照してください。

   b. `table5` にレコードを挿入します。

      ```SQL
      INSERT INTO table5 VALUES
          (101, 2, 80),
          (102, 2, 90);
      ```

### データのロード

レコードの`id`値がそれぞれ`101`および`102`のレコードを、`example5.csv`から`table5`に更新するためのロードを実行し、それぞれの2つのレコードの`version`の値が現在の`version`の値以上の場合にのみ更新が行われるように指定します。

- ストリームロードジョブを実行します：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label10" \
      -H "column_separator:," \
      -H "merge_condition:version" \
      -T example5.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
  ```

- 通常のロードジョブを実行します：

  ```SQL
  CREATE ROUTINE LOAD test_db.table5 on table5
  COLUMNS (id, version, score),
  COLUMNS TERMINATED BY ','
  PROPERTIES
  (
      "merge_condition" = "version"
  )
  FROM KAFKA
  (
      "kafka_broker_list" ="<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "topic5",
      "property.kafka_default_offsets" ="OFFSET_BEGINNING"
  );
  ```

### データのクエリ

ロードが完了したら、`table5`のデータをクエリして、ロードが成功したことを確認してください：

```SQL
SELECT * FROM table5;
+------+------+-------+
| id   | version | score |
+------+------+-------+
|  101 |       2 |   80 |
|  102 |       3 |  100 |
+------+------+-------+
2 rows in set (0.02 sec)
```

前のクエリ結果に示されているように、`example5.csv`の`id`が`101`のレコードは`table5`に更新されず、`example5.csv`の`id`が`102`のレコードが`table5`に挿入されました。