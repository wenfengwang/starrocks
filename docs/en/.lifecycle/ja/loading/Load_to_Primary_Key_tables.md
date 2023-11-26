---
displayed_sidebar: "Japanese"
---

# ロードを介してデータを変更する

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksが提供する[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)を使用すると、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、または[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)ジョブを実行することで、StarRocksテーブルに対してデータの変更を行うことができます。これらのデータの変更には、挿入、更新、削除が含まれます。ただし、プライマリキーテーブルでは、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)または[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用してデータを変更することはサポートされていません。

StarRocksは部分更新と条件付き更新もサポートしています。

<InsertPrivNote />

このトピックでは、CSVデータを使用して、ロードを介してStarRocksテーブルにデータの変更を行う方法について説明します。選択したロード方法によってサポートされるデータファイルの形式は異なります。

> **注意**
>
> CSVデータの場合、テキスト区切り文字として、50バイトを超えないUTF-8文字列（カンマ（,）、タブ、またはパイプ（|）など）を使用できます。

## 実装

StarRocksが提供するプライマリキーテーブルは、UPSERTおよびDELETE操作をサポートしており、INSERT操作とUPDATE操作を区別しません。

ロードジョブを作成する際、StarRocksはジョブ作成ステートメントまたはコマンドに`__op`というフィールドを追加することをサポートしています。`__op`フィールドは、実行する操作のタイプを指定するために使用されます。

> **注意**
>
> テーブルを作成する際に、そのテーブルに`__op`という名前の列を追加する必要はありません。

`__op`フィールドの定義方法は、選択したロード方法によって異なります。

- Stream Loadを選択した場合、`__op`フィールドは`columns`パラメータを使用して定義します。

- Broker Loadを選択した場合、`__op`フィールドはSET句を使用して定義します。

- Routine Loadを選択した場合、`__op`フィールドは`COLUMNS`列を使用して定義します。

データの変更を行うかどうかは、`__op`フィールドを追加するかどうかによって決定することができます。`__op`フィールドを追加しない場合、操作タイプはデフォルトでUPSERTになります。主なデータ変更シナリオは次のとおりです。

- ロードするデータファイルがUPSERT操作のみを含む場合、`__op`フィールドを追加する必要はありません。

- ロードするデータファイルがDELETE操作のみを含む場合、`__op`フィールドを追加し、操作タイプをDELETEに指定する必要があります。

- ロードするデータファイルがUPSERTおよびDELETE操作の両方を含む場合、`__op`フィールドを追加し、データファイルに`0`または`1`の値を持つ列が含まれていることを確認する必要があります。`0`の値はUPSERT操作を示し、`1`の値はDELETE操作を示します。

## 使用上の注意

- データファイルの各行には、同じ数の列が含まれていることを確認してください。

- データの変更に関与する列には、プライマリキーカラムが含まれている必要があります。

## 前提条件

### Broker Load

[HDFSからデータをロードする](../loading/hdfs_load.md)または[クラウドストレージからデータをロードする](../loading/cloud_storage_load.md)の「背景情報」セクションを参照してください。

### Routine Load

Routine Loadを選択した場合、Apache Kafka®クラスタにトピックが作成されていることを確認してください。`topic1`、`topic2`、`topic3`、`topic4`の4つのトピックが作成されていると仮定します。

## 基本操作

このセクションでは、ロードを介してStarRocksテーブルにデータの変更を行う方法の例を提供します。詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### UPSERT

ロードするデータファイルがUPSERT操作のみを含む場合、`__op`フィールドを追加する必要はありません。

> **注意**
>
> `__op`フィールドを追加する場合は、次のことに注意してください。
>
> - 操作タイプをUPSERTとして指定できます。
>
> - `__op`フィールドを空のままにすることができます。なぜなら、操作タイプはデフォルトでUPSERTになるからです。

#### データの例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example1.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、ユーザー名、ユーザースコアを順に表す3つの列で構成されています。

      ```Plain
      101,Lily,100
      102,Rose,100
      ```

   b. `example1.csv`のデータをKafkaクラスタの`topic1`に公開します。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table1`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、`score`の3つの列で構成されており、`id`がプライマリキーです。

      ```SQL
      CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "ユーザーID",
          `name` varchar(65533) NOT NULL COMMENT "ユーザー名",
          `score` int(11) NOT NULL COMMENT "ユーザースコア"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **注意**
      >
      > StarRocksは、テーブルまたはパーティションを作成する際にバケットの数（BUCKETS）を自動的に設定することができます。バケットの数を手動で設定する必要はありません。詳細については、[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table1`にレコードを挿入します。

      ```SQL
      INSERT INTO table1 VALUES
          (101, 'Lily',80);
      ```

#### データのロード

`example1.csv`の`id`が`101`のレコードを`table1`に更新し、`example1.csv`の`id`が`102`のレコードを`table1`に挿入するために、ロードジョブを実行します。

- Stream Loadジョブを実行します。

  - `__op`フィールドを含めたくない場合、次のコマンドを実行します。

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label1" \
        -H "column_separator:," \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

  - `__op`フィールドを含めたい場合、次のコマンドを実行します。

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

  - `__op`フィールドを含めたくない場合、次のコマンドを実行します。

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

  - `__op`フィールドを含めたい場合、次のコマンドを実行します。

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

  - `__op`フィールドを含めたくない場合、次のコマンドを実行します。

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

  - `__op`フィールドを含めたい場合、次のコマンドを実行します。

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

#### データのクエリ

ロードが完了したら、`table1`のデータをクエリして、ロードが成功したことを確認します。

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

上記のクエリ結果に示されているように、`example1.csv`の`id`が`101`のレコードが`table1`に更新され、`example1.csv`の`id`が`102`のレコードが`table1`に挿入されました。

### DELETE

ロードするデータファイルがDELETE操作のみを含む場合、`__op`フィールドを追加し、操作タイプをDELETEに指定する必要があります。

#### データの例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example2.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、ユーザー名、ユーザースコアを順に表す3つの列で構成されています。

      ```Plain
      101,Jack,100
      ```

   b. `example2.csv`のデータをKafkaクラスタの`topic2`に公開します。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table2`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、`score`の3つの列で構成されており、`id`がプライマリキーです。

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

      > **注意**
      >
      > StarRocksは、テーブルまたはパーティションを作成する際にバケットの数（BUCKETS）を自動的に設定することができます。バケットの数を手動で設定する必要はありません。詳細については、[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table2`に2つのレコードを挿入します。

      ```SQL
      INSERT INTO table2 VALUES
      (101, 'Jack', 100),
      (102, 'Bob', 90);
      ```

#### データのロード

`example2.csv`の`id`が`101`のレコードを`table2`から削除するために、ロードジョブを実行します。

- Stream Loadジョブを実行します。

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label3" \
      -H "column_separator:," \
      -H "columns:__op='delete'" \
      -T example2.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
  ```

- Broker Loadジョブを実行します。

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

- Routine Loadジョブを実行します。

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

#### データのクエリ

ロードが完了したら、`table2`のデータをクエリして、ロードが成功したことを確認します。

```SQL
SELECT * FROM table2;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  102 | Bob  |    90 |
+------+------+-------+
1 row in set (0.00 sec)
```

上記のクエリ結果に示されているように、`example2.csv`の`id`が`101`のレコードが`table2`から削除されました。

### UPSERTとDELETE

ロードするデータファイルがUPSERTおよびDELETE操作の両方を含む場合、`__op`フィールドを追加し、データファイルに`0`または`1`の値を持つ列が含まれていることを確認する必要があります。`0`の値はUPSERT操作を示し、`1`の値はDELETE操作を示します。

#### データの例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example3.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、ユーザー名、ユーザースコア、および操作タイプを順に表す4つの列で構成されています。

      ```Plain
      101,Tom,100,1
      102,Sam,70,0
      103,Stan,80,0
      ```

   b. `example3.csv`のデータをKafkaクラスタの`topic3`に公開します。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table3`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、`score`の3つの列で構成されており、`id`がプライマリキーです。

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

      > **注意**
      >
      > StarRocksは、テーブルまたはパーティションを作成する際にバケットの数（BUCKETS）を自動的に設定することができます。バケットの数を手動で設定する必要はありません。詳細については、[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table3`に2つのレコードを挿入します。

      ```SQL
      INSERT INTO table3 VALUES
          (101, 'Tom', 100),
          (102, 'Sam', 90);
      ```

#### データのロード

`example3.csv`の`id`が`101`のレコードを`table3`から削除し、`example3.csv`の`id`が`102`のレコードを`table3`に更新し、`example3.csv`の`id`が`103`のレコードを`table3`に挿入するために、ロードジョブを実行します。

- Stream Loadジョブを実行します。

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label4" \
      -H "column_separator:," \
      -H "columns: id, name, score, temp, __op = temp" \
      -T example3.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
  ```

  > **注意**
  >
  > 上記の例では、`example3.csv`の操作タイプを一時的に`temp`という名前にし、`__op`フィールドを`columns`パラメータを使用して`temp`列にマッピングしています。これにより、`example3.csv`の各レコードの4番目の列の値が`0`または`1`であるかどうかに応じて、StarRocksがUPSERT操作またはDELETE操作を実行するかどうかを決定することができます。

- Broker Loadジョブを実行します。

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

- Routine Loadジョブを実行します。

  ```SQL
  CREATE ROUTINE LOAD test_db.table3 ON table3
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
      "kafka_broker_list" ="<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "test3",
      "property.kafka_default_offsets" ="OFFSET_BEGINNING"
  );
  ```

#### データのクエリ

ロードが完了したら、`table3`のデータをクエリして、ロードが成功したことを確認します。

```SQL
SELECT * FROM table3;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  102 | Sam  |    70 |
|  103 | Stan |    80 |
+------+------+-------+
2 rows in set (0.01 sec)
```

上記のクエリ結果に示されているように、`example3.csv`の`id`が`101`のレコードが`table3`から削除され、`example3.csv`の`id`が`102`のレコードが`table3`に更新され、`example3.csv`の`id`が`103`のレコードが`table3`に挿入されました。

## 部分更新

v2.2以降、StarRocksはプライマリキーテーブルの指定された列のみを更新する部分更新をサポートしています。このセクションでは、CSVを例にして部分更新の方法を説明します。

> **注意**
>
> 部分更新を実行する場合、更新対象の行が存在しない場合、StarRocksは新しい行を挿入し、データの更新が行われないために空のフィールドにデフォルト値を入れます。

### データの例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example4.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーIDとユーザー名を順に表す2つの列で構成されています。

      ```Plain
      101,Lily
      102,Rose
      103,Alice
      ```

   b. `example4.csv`のデータをKafkaクラスタの`topic4`に公開します。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table4`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、`score`の3つの列で構成されており、`id`がプライマリキーです。

      ```SQL
      CREATE TABLE `table4`
      (
          `id` int(11) NOT NULL COMMENT "ユーザーID",
          `name` varchar(65533) NOT NULL COMMENT "ユーザー名",
          `score` int(11) NOT NULL COMMENT "ユーザースコア"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **注意**
      >
      > StarRocksは、テーブルまたはパーティションを作成する際にバケットの数（BUCKETS）を自動的に設定することができます。バケットの数を手動で設定する必要はありません。詳細については、[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table4`にレコードを挿入します。

      ```SQL
      INSERT INTO table4 VALUES
          (101, 'Tom',80);
      ```

### データのロード

`example4.csv`の2つの列のデータを`table4`の`id`列と`name`列に更新するために、ロードを実行します。

- Stream Loadジョブを実行します。

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label7" -H "column_separator:," \
      -H "partial_update:true" \
      -H "columns:id,name" \
      -T example4.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
  ```

  > **注意**
  >
  > Stream Loadを選択した場合、部分更新機能を有効にするために`partial_update`パラメータを`true`に設定する必要があります。さらに、更新する列を指定するために`columns`パラメータを使用する必要があります。

- Broker Loadジョブを実行します。

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

  > **注意**
  >
  > Broker Loadを選択した場合、部分更新機能を有効にするために`partial_update`パラメータを`true`に設定する必要があります。さらに、更新する列を指定するために`column_list`パラメータを使用する必要があります。

- Routine Loadジョブを実行します。

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

  > **注意**
  >
  > Broker Loadを選択した場合、部分更新機能を有効にするために`partial_update`パラメータを`true`に設定する必要があります。さらに、更新する列を指定するために`COLUMNS`パラメータを使用する必要があります。

### データのクエリ

ロードが完了したら、`table4`のデータをクエリして、ロードが成功したことを確認します。

```SQL
SELECT * FROM table4;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|  102 | Rose  |     0 |
|  101 | Lily  |    80 |
|  103 | Alice |     0 |
+------+-------+-------+
3 rows in set (0.01 sec)
```

上記のクエリ結果に示されているように、`example4.csv`の`id`が`101`のレコードが`table4`に更新され、`example4.csv`の`id`が`102`および`103`のレコードが`table4`に挿入されました。

## 条件付き更新

StarRocks v2.5以降、プライマリキーテーブルは条件付き更新をサポートしています。指定した非プライマリキーカラムを条件として指定することで、更新が有効になるかどうかを判断することができます。したがって、ソースレコードから宛先レコードへの更新は、指定した列の値がソースデータレコードで現在のバージョンの値以上である場合にのみ有効になります。

条件付き更新機能は、データの不整合を解決するために設計されています。ソースデータが不整合している場合、この機能を使用して新しいデータが古いデータに上書きされないようにすることができます。

> **注意**
>
> - 同じデータバッチに対して異なる列を更新条件として指定することはできません。
> - DELETE操作は条件付き更新をサポートしていません。
> - v3.1.3より前のバージョンでは、部分更新と条件付き更新を同時に使用することはできません。v3.1.3以降、StarRocksは部分更新と条件付き更新を同時に使用することができます。
> - 条件付き更新は、Stream LoadとRoutine Loadのみサポートしています。

### データの例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example5.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、バージョン、およびユーザースコアを順に表す3つの列で構成されています。

      ```Plain
      101,1,100
      102,3,100
      ```

   b. `example5.csv`のデータをKafkaクラスタの`topic5`に公開します。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table5`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`version`、`score`の3つの列で構成されており、`id`がプライマリキーです。

      ```SQL
      CREATE TABLE `table5`
      (
          `id` int(11) NOT NULL COMMENT "ユーザーID", 
          `version` int NOT NULL COMMENT "バージョン",
          `score` int(11) NOT NULL COMMENT "ユーザースコア"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`) DISTRIBUTED BY HASH(`id`);
      ```

      > **注意**
      >
      > StarRocksは、テーブルまたはパーティションを作成する際にバケットの数（BUCKETS）を自動的に設定することができます。バケットの数を手動で設定する必要はありません。詳細については、[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table5`にレコードを挿入します。

      ```SQL
      INSERT INTO table5 VALUES
          (101, 2, 80),
          (102, 2, 90);
      ```

### データのロード

`example5.csv`の`id`値が`101`および`102`のレコードを、それぞれのレコードの`version`値が現在の`version`値以上である場合にのみ、`table5`に更新するために、ロードジョブを実行します。

- Stream Loadジョブを実行します。

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label10" \
      -H "column_separator:," \
      -H "merge_condition:version" \
      -T example5.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
  ```

- Routine Loadジョブを実行します。

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

ロードが完了したら、`table5`のデータをクエリして、ロードが成功したことを確認します。

```SQL
SELECT * FROM table5;
+------+------+-------+
| id   | version | score |
+------+------+-------+
|  101 |       2 |   80 |
|  102 |       3 |  100 |
+------+------+-------+
2行がセットになっています（0.02秒）

前のクエリ結果に示されているように、`example5.csv`の`id`が`101`のレコードは`table5`に更新されていませんが、`id`が`102`のレコードは`table5`に挿入されました。
