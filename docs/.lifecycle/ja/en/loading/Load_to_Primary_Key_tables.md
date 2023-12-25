---
displayed_sidebar: English
---

# データの変更をロードを通じて行う

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksが提供する[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)では、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、または[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)ジョブを実行することで、StarRocksテーブルにデータ変更を加えることができます。これらのデータ変更には、挿入、更新、および削除が含まれます。ただし、プライマリキーテーブルでは、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)や[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用したデータの変更はサポートされていません。

StarRocksは部分更新と条件付き更新もサポートしています。

<InsertPrivNote />

このトピックでは、CSVデータを例に、ロードを通じてStarRocksテーブルにデータ変更を加える方法について説明します。サポートされるデータファイル形式は、選択したロード方法によって異なります。

> **注記**
>
> CSVデータの場合、50バイトを超えない長さのUTF-8文字列（カンマ(,)、タブ、パイプ(|)など）をテキスト区切り文字として使用できます。

## 実装

StarRocksが提供するプライマリキーテーブルはUPSERTとDELETE操作をサポートし、INSERT操作とUPDATE操作を区別しません。

ロードジョブを作成する際、StarRocksはジョブ作成ステートメントまたはコマンドに`__op`というフィールドを追加することをサポートしています。`__op`フィールドは、実行したい操作のタイプを指定するために使用されます。

> **注記**
>
> テーブルを作成する際に`__op`という名前のカラムを追加する必要はありません。

`__op`フィールドの定義方法は、選択したロード方法によって異なります：

- Stream Loadを選択した場合、`columns`パラメータを使用して`__op`フィールドを定義します。

- Broker Loadを選択した場合、SET句を使用して`__op`フィールドを定義します。

- Routine Loadを選択した場合、`COLUMNS`句を使用して`__op`フィールドを定義します。

`__op`フィールドの追加は、行いたいデータ変更に基づいて決定できます。`__op`フィールドを追加しない場合、操作タイプはデフォルトでUPSERTになります。主なデータ変更シナリオは以下の通りです：

- ロードしたいデータファイルにUPSERT操作のみが含まれる場合、`__op`フィールドを追加する必要はありません。

- ロードしたいデータファイルにDELETE操作のみが含まれる場合、`__op`フィールドを追加し、操作タイプをDELETEとして指定する必要があります。

- ロードしたいデータファイルにUPSERT操作とDELETE操作の両方が含まれる場合、`__op`フィールドを追加し、データファイルに`0`または`1`の値を持つカラムが含まれていることを確認する必要があります。`0`はUPSERT操作を示し、`1`はDELETE操作を示します。

## 使用上の注意

- データファイルの各行が同じ数のカラムを持っていることを確認してください。

- データ変更を伴うカラムには、プライマリキーカラムを含める必要があります。

## 前提条件

### Broker Load

[HDFSからのデータロード](../loading/hdfs_load.md)または[クラウドストレージからのデータロード](../loading/cloud_storage_load.md)の「背景情報」セクションを参照してください。

### Routine Load

Routine Loadを選択する場合、Apache Kafka®クラスターにトピックが作成されていることを確認してください。`topic1`、`topic2`、`topic3`、`topic4`という4つのトピックを作成したとします。

## 基本操作

このセクションでは、ロードを通じてStarRocksテーブルにデータ変更を加える方法の例を提供します。詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### UPSERT

ロードしたいデータファイルにUPSERT操作のみが含まれる場合、`__op`フィールドを追加する必要はありません。

> **注記**
>
> `__op`フィールドを追加する場合：
>
> - 操作タイプをUPSERTとして指定できます。
>
> - `__op`フィールドを空にしておくことができます。なぜなら、操作タイプはデフォルトでUPSERTに設定されているからです。

#### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example1.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、ユーザー名、ユーザースコアを順に表す3つのカラムで構成されています。

      ```Plain
      101,Lily,100
      102,Rose,100
      ```

   b. Kafkaクラスターの`topic1`に`example1.csv`のデータを公開します。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table1`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、`score`という3つのカラムで構成されており、`id`がプライマリキーです。

      ```SQL
      CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NOT NULL COMMENT "user name",
          `score` int(11) NOT NULL COMMENT "user score"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`) BUCKETS 10;
      ```

      > **注記**
      >
      > v2.5.7以降、StarRocksはテーブルを作成する際やパーティションを追加する際に、バケット数(BUCKETS)を自動的に設定することができます。バケット数を手動で設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table1`にレコードを挿入します。

      ```SQL
      INSERT INTO table1 VALUES
          (101, 'Lily', 80);
      ```

#### データのロード

`example1.csv`の`id`が`101`のレコードを`table1`に更新し、`example1.csv`の`id`が`102`のレコードを`table1`に挿入するためのロードジョブを実行します。

- Stream Loadジョブを実行します。
  
  - `__op`フィールドを含めない場合、以下のコマンドを実行します：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label1" \
        -H "column_separator:," \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

  - `__op`フィールドを含める場合、以下のコマンドを実行します：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label2" \
        -H "column_separator:," \
        -H "columns:__op='upsert'" \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

- Broker Loadジョブを実行します。

  - `__op`フィールドを含めない場合、以下のコマンドを実行します：

    ```SQL
    LOAD LABEL test_db.label1
    (
        DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/example1.csv")
        INTO TABLE table1
        COLUMNS TERMINATED BY ","
        FORMAT AS "csv"
    )
    WITH BROKER "broker1";
    ```

  - `__op`フィールドを含める場合、以下のコマンドを実行します：

    ```SQL
    LOAD LABEL test_db.label2
    (
        DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/example1.csv")
        INTO TABLE table1
        COLUMNS TERMINATED BY ","
        FORMAT AS "csv"
        SET (__op = 'upsert')
    )
    WITH BROKER "broker1";
    ```

- Routine Loadジョブを実行します。

  - `__op`フィールドを含めない場合、以下のコマンドを実行します：

    ```SQL
    CREATE ROUTINE LOAD test_db.table1 ON table1
    COLUMNS TERMINATED BY ",",
    COLUMNS (id, name, score)
    PROPERTIES
    (
        "desired_concurrent_number" = "3",
        "max_batch_interval" = "20",
        "max_batch_rows" = "250000",
        "max_error_number" = "1000"
    )
    FROM KAFKA
    (
        "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
        "kafka_topic" = "topic1",
        "property.kafka_default_offsets" = "OFFSET_BEGINNING"
    );
    ```

  - `__op` フィールドを含める場合は、次のコマンドを実行します。

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

#### データクエリ

ロードが完了したら、`table1` のデータを照会して、ロードが成功したことを確認します。

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

上記のクエリ結果に示されているように、`example1.csv` にある `id` が `101` のレコードは `table1` に更新され、`id` が `102` のレコードは `table1` に挿入されました。

### DELETE

ロードするデータファイルに DELETE 操作のみが含まれる場合は、`__op` フィールドを追加し、操作タイプを DELETE として指定する必要があります。

#### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに `example2.csv` という名前の CSV ファイルを作成します。このファイルは、ユーザー ID、ユーザー名、ユーザースコアを順に表す 3 つの列で構成されています。

      ```Plain
      101,Jack,100
      ```

   b. Kafka クラスターの `topic2` に `example2.csv` のデータをパブリッシュします。

2. StarRocks テーブルを準備します。

   a. StarRocks の `test_db` に `table2` という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、`score` の 3 つの列で構成され、`id` がプライマリキーです。

      ```SQL
      CREATE TABLE `table2`
            (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NOT NULL COMMENT "user name",
          `score` int(11) NOT NULL COMMENT "user score"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`) ON `table2`;
      ```

      > **注記**
      >
      > v2.5.7 以降、StarRocks はテーブルを作成またはパーティションを追加する際に、バケット数（BUCKETS）を自動的に設定できます。バケット数を手動で設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table2` に 2 つのレコードを挿入します。

      ```SQL
      INSERT INTO table2 VALUES
      (101, 'Jack', 100),
      (102, 'Bob', 90);
      ```

#### データロード

`example2.csv` にある `id` が `101` のレコードを `table2` から削除するロードジョブを実行します。

- ストリームロードジョブを実行します。

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label3" \
      -H "column_separator:," \
      -H "columns:__op='delete'" \
      -T example2.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
  ```

- ブローカーロードジョブを実行します。

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

- ルーチンロードジョブを実行します。

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

#### データクエリ

ロードが完了したら、`table2` のデータを照会して、ロードが成功したことを確認します。

```SQL
SELECT * FROM table2;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  102 | Bob  |    90 |
+------+------+-------+
1 row in set (0.00 sec)
```

上記のクエリ結果に示されているように、`example2.csv` にある `id` が `101` のレコードは `table2` から削除されました。

### UPSERT と DELETE

ロードするデータファイルに UPSERT 操作と DELETE 操作の両方が含まれる場合は、`__op` フィールドを追加し、データファイルに `0` または `1` の値を持つ列が含まれていることを確認する必要があります。`0` の値は UPSERT 操作を示し、`1` の値は DELETE 操作を示します。

#### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに `example3.csv` という名前の CSV ファイルを作成します。このファイルは、ユーザー ID、ユーザー名、ユーザースコア、および操作タイプを順に表す 4 つの列で構成されています。

      ```Plain
      101,Tom,100,1
      102,Sam,70,0
      103,Stan,80,0
      ```

   b. Kafka クラスターの `topic3` に `example3.csv` のデータをパブリッシュします。

2. StarRocks テーブルを準備します。

   a. StarRocks の `test_db` に `table3` という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、`score` の 3 つの列で構成され、`id` がプライマリキーです。

      ```SQL
      CREATE TABLE `table3`
      (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NOT NULL COMMENT "user name",
          `score` int(11) NOT NULL COMMENT "user score"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`) ON `table3`;
      ```

      > **注記**
      >
      > v2.5.7 以降、StarRocks はテーブルを作成またはパーティションを追加する際に、バケット数（BUCKETS）を自動的に設定できます。バケット数を手動で設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table3` に 2 つのレコードを挿入します。

      ```SQL
      INSERT INTO table3 VALUES
          (101, 'Tom', 100),
          (102, 'Sam', 90);
      ```

#### データロード

`example3.csv` にある `id` が `101` のレコードを `table3` から削除し、`id` が `102` のレコードを `table3` に更新し、`id` が `103` のレコードを `table3` に挿入するロードジョブを実行します。

- ストリームロードジョブを実行します。

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label4" \
      -H "column_separator:," \
      -H "columns: id, name, score, temp, __op = temp" \
      -T example3.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
  ```

  > **注記**
  >
  > 上記の例では、`example3.csv` の操作タイプを表す 4 番目の列は一時的に `temp` と名付けられ、`__op` フィールドは `columns` パラメータを使用して `temp` 列にマッピングされます。これにより、StarRocks は `example3.csv` の 4 番目の列の値が `0` または `1` かに応じて、UPSERT 操作または DELETE 操作を実行するかを決定できます。

- ブローカーロードジョブを実行します。

  ```Bash
  LOAD LABEL test_db.label4
  (
      data infile("hdfs://<hdfs_host>:<hdfs_port>/example3.csv")
      into table table3
      columns terminated by ","
      format as "csv"
      (id, name, score, temp)
      set (__op=temp)
  )
  with broker "broker1";
  ```

- ルーチンロードジョブを実行します。

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
      "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "test3",
      "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  );
  ```

#### データクエリ

ロードが完了したら、`table3` のデータを照会して、ロードが成功したことを確認します。

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


上記のクエリ結果に示されているように、`example3.csv`にある`id`が`101`のレコードは`table3`から削除され、`id`が`102`のレコードは`table3`に更新され、`id`が`103`のレコードは`table3`に挿入されています。

## 部分更新

v2.2以降、StarRocksはプライマリーキーテーブルの指定された列のみの更新をサポートしています。このセクションでは、CSVを例に、部分更新を実行する方法について説明します。

> **注意**
>
> 部分更新を実行する際、更新する行が存在しない場合、StarRocksは新しい行を挿入し、データ更新が挿入されないために空のフィールドにデフォルト値を入力します。

### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example4.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーIDとユーザー名を順番に表す2つの列で構成されています。

      ```Plain
      101,Lily
      102,Rose
      103,Alice
      ```

   b. Kafkaクラスターの`topic4`に`example4.csv`のデータをパブリッシュします。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table4`という名前のプライマリーキーテーブルを作成します。このテーブルは3つの列：`id`、`name`、`score`で構成され、`id`がプライマリーキーです。

      ```SQL
      CREATE TABLE `table4`
      (
          `id` int(11) NOT NULL COMMENT 'user ID',
          `name` varchar(65533) NOT NULL COMMENT 'user name',
          `score` int(11) NOT NULL COMMENT 'user score'
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`) BUCKETS 10;
      ```

      > **注記**
      >
      > v2.5.7以降、StarRocksはテーブルの作成時やパーティションの追加時にバケット数(BUCKETS)を自動的に設定できます。バケットの数を手動で設定する必要はありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table4`にレコードを挿入します。

      ```SQL
      INSERT INTO `table4` VALUES
          (101, 'Tom', 80);
      ```

### データのロード

`example4.csv`の2つの列のデータを`table4`の`id`と`name`の列に更新するためのロードを実行します。

- ストリームロードジョブを実行します：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label7" -H "column_separator:," \
      -H "partial_update:true" \
      -H "columns:id,name" \
      -T example4.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
  ```

  > **注記**
  >
  > ストリームロードを選択した場合は、`partial_update`パラメータを`true`に設定して、部分更新機能を有効にする必要があります。さらに、`columns`パラメータを使用して、更新する列を指定する必要があります。

- ブローカーロードジョブを実行します：

  ```SQL
  LOAD LABEL `test_db`.`table4`
  (
      DATA INFILE('hdfs://<hdfs_host>:<hdfs_port>/example4.csv')
      INTO TABLE `table4`
      FORMAT AS 'csv'
      (id, name)
  )
  WITH BROKER 'broker1'
  PROPERTIES
  (
      'partial_update' = 'true'
  );
  ```

  > **注記**
  >
  > ブローカーロードを選択した場合は、`partial_update`パラメータを`true`に設定して、部分更新機能を有効にする必要があります。さらに、`column_list`パラメータを使用して、更新する列を指定する必要があります。

- ルーチンロードジョブを実行します：

  ```SQL
  CREATE ROUTINE LOAD `test_db`.`table4` ON `table4`
  COLUMNS (id, name),
  COLUMNS TERMINATED BY ','
  PROPERTIES
  (
      'partial_update' = 'true'
  )
  FROM KAFKA
  (
      'kafka_broker_list' ='<kafka_broker_host>:<kafka_broker_port>',
      'kafka_topic' = 'topic4',
      'property.kafka_default_offsets' ='OFFSET_BEGINNING'
  );
  ```

  > **注記**
  >
  > ルーチンロードを選択した場合は、`partial_update`パラメータを`true`に設定して、部分更新機能を有効にする必要があります。さらに、`COLUMNS`パラメータを使用して、更新する列を指定する必要があります。

### データのクエリ

ロードが完了したら、`table4`のデータを照会して、ロードが成功したことを確認します：

```SQL
SELECT * FROM `table4`;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|  102 | Rose  |     0 |
|  101 | Lily  |    80 |
|  103 | Alice |     0 |
+------+-------+-------+
3 rows in set (0.01 sec)
```

上記のクエリ結果に示されているように、`example4.csv`にある`id`が`101`のレコードは`table4`に更新され、`id`が`102`と`103`のレコードは`table4`に挿入されています。

## 条件付き更新

StarRocks v2.5以降、プライマリーキーテーブルは条件付き更新をサポートしています。非プライマリーキーカラムを条件として指定して、更新が有効になるかどうかを判断できます。そのため、ソースレコードから宛先レコードへの更新は、ソースデータレコードが指定されたカラムで宛先データレコードよりも大きいか等しい値を持つ場合にのみ有効になります。

条件付き更新機能は、データの乱れを解決するために設計されています。ソースデータが乱れている場合、この機能を使用して、新しいデータが古いデータによって上書きされないようにすることができます。

> **注意**
>
> - 同じデータバッチに対して異なるカラムを更新条件として指定することはできません。
> - DELETE操作は条件付き更新をサポートしていません。
> - v3.1.3より前のバージョンでは、部分更新と条件付き更新を同時に使用することはできません。v3.1.3以降、StarRocksは部分更新と条件付き更新を同時に使用することをサポートしています。

### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example5.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、バージョン、およびユーザースコアを順番に表す3つの列で構成されています。

      ```Plain
      101,1,100
      102,3,100
      ```

   b. Kafkaクラスターの`topic5`に`example5.csv`のデータをパブリッシュします。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table5`という名前のプライマリーキーテーブルを作成します。このテーブルは3つの列：`id`、`version`、`score`で構成され、`id`がプライマリーキーです。

      ```SQL
      CREATE TABLE `table5`
      (
          `id` int(11) NOT NULL COMMENT 'user ID',
          `version` int NOT NULL COMMENT 'version',
          `score` int(11) NOT NULL COMMENT 'user score'
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`) DISTRIBUTED BY HASH(`id`) BUCKETS 10;
      ```

      > **注記**
      >
      > v2.5.7以降、StarRocksはテーブルの作成時やパーティションの追加時にバケット数(BUCKETS)を自動的に設定できます。バケットの数を手動で設定する必要はありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table5`にレコードを挿入します。

      ```SQL
      INSERT INTO `table5` VALUES
          (101, 2, 80),
          (102, 2, 90);
      ```

### データのロード

`example5.csv`から`table5`に`id`が`101`と`102`のレコードを更新し、それぞれのレコードの`version`値が現在の`version`値以上の場合にのみ更新が有効になるように指定するロードを実行します。

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

- ルーチンロードジョブを実行します：

  ```SQL
  CREATE ROUTINE LOAD `test_db`.`table5` ON `table5`
  COLUMNS (id, version, score),
  COLUMNS TERMINATED BY ','
  PROPERTIES
  (
      'merge_condition' = 'version'
  )
  FROM KAFKA
  (
      'kafka_broker_list' ='<kafka_broker_host>:<kafka_broker_port>',
      'kafka_topic' = 'topic5',
      'property.kafka_default_offsets' ='OFFSET_BEGINNING'
  );
  ```

- ブローカーロードジョブを実行します：

  ```SQL
  LOAD LABEL `test_db`.`table5`
  ( DATA INFILE ('s3://xxx.csv')
    INTO TABLE `table5` COLUMNS TERMINATED BY "," FORMAT AS "CSV"
  )
  WITH BROKER
  PROPERTIES
  (
      'merge_condition' = 'version'
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
2 rows in set (0.02 sec)
```

前述のクエリ結果に示されているように、`example5.csv`の`id`が`101`のレコードは`table5`に更新されておらず、`id`が`102`のレコードは`example5.csv`から`table5`に挿入されています。
