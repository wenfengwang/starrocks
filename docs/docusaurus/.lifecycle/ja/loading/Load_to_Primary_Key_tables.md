---
displayed_sidebar: "Japanese"
---

# ロードを通じたデータの変更

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksが提供する[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)は、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、または[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を実行することで、StarRocksテーブルに対してデータ変更を行うことを可能にします。これらのデータ変更には、挿入、更新、削除が含まれます。ただし、プライマリキーテーブルでは[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)または[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を使用してデータを変更することはサポートされていません。

StarRocksは部分的な更新と条件付き更新もサポートしています。

<InsertPrivNote />

このトピックでは、CSVデータを例として使用し、StarRocksテーブルに対するデータ変更をロードを通じて行う方法について説明します。サポートされるデータファイル形式は、選択したロード方法によって異なります。

> **注記**
>
> CSVデータを使用する場合、テキストの区切り記号として、コンマ（,）、タブ、またはパイプ（|）などのUTF-8文字列を使用できます。ただし、その長さは50バイトを超えてはいけません。

## 実装

StarRocksが提供するプライマリキーテーブルはUPSERTおよびDELETEの操作をサポートしており、INSERTの操作をUPDATEの操作と区別しません。

ロードジョブを作成する際に、StarRocksは作成ステートメントまたはコマンドに`__op`というフィールドを追加することをサポートしています。`__op`フィールドは、実行したい操作のタイプを指定するために使用されます。

> **注記**
>
> テーブルを作成する際に、そのテーブルに`__op`という名前の列を追加する必要はありません。

`__op`フィールドの定義方法は、選択したロード方法によって異なります。

- Stream Loadを選択した場合は、`columns`パラメータを使用して`__op`フィールドを定義します。

- Broker Loadを選択した場合は、SET句を使用して`__op`フィールドを定義します。

- Routine Loadを選択した場合は、`COLUMNS`カラムを使用して`__op`フィールドを定義します。

`__op`フィールドを追加するかどうかは、行いたいデータ変更に基づいて決定できます。`__op`フィールドを追加しない場合は、操作タイプが自動的にUPSERTになります。主なデータ変更シナリオは次の通りです。

- ロードしたいデータファイルがUPSERT操作のみを含む場合、`__op`フィールドを追加する必要はありません。

- ロードしたいデータファイルが削除操作のみを含む場合、`__op`フィールドを追加し、操作タイプをDELETEとして指定する必要があります。

- ロードしたいデータファイルがUPSERTおよびDELETEの両方の操作を含む場合は、`__op`フィールドを追加し、データファイルに値が「0」または「1」である列が含まれていることを確認する必要があります。値が「0」の場合はUPSERT操作を、値が「1」の場合はDELETE操作を意味します。

## 使用上の注意

- データファイルの各行が同じ数の列を持つことを確認してください。

- データ変更に関与する列には、プライマリキー列が含まれている必要があります。

## 事前条件

### Broker Load

[HDFSからデータをロード](../loading/hdfs_load.md)または[クラウドストレージからデータをロード](../loading/cloud_storage_load.md)の「背景情報」セクションを参照してください。

### Routine Load

Routine Loadを選択した場合は、Apache Kafka®クラスターにトピックが作成されていることを確認してください。`topic1`、`topic2`、`topic3`、および`topic4`の4つのトピックが作成されていると仮定します。

## ベーシックオペレーション

このセクションでは、StarRocksテーブルに対するデータ変更をロードする例を提供します。詳細な構文およびパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### UPSERT

ロードしたいデータファイルがUPSERT操作のみを含む場合は、`__op`フィールドを追加する必要はありません。

> **注記**
>
> `__op`フィールドを追加する場合：
>
> - 操作タイプをUPSERTとして指定できます。
>
> - 操作タイプはデフォルトでUPSERTになるため、`__op`フィールドを空のままにすることもできます。

#### データの例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example1.csv`という名前のCSVファイルを作成します。ファイルは、ユーザーID、ユーザー名、およびユーザースコアを順に表す3つの列から構成されています。

      ```Plain
      101,Lily,100
      102,Rose,100
      ```

   b. `example1.csv`のデータをKafkaクラスターの`topic1`にパブリッシュします。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table1`という名前のプライマリキーテーブルを作成します。テーブルは、`id`、`name`、および`score`の3つの列から構成されており、そのうち`id`がプライマリキーです。

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

      > **注記**
      >
      > v2.5.7以降、StarRocksはテーブルの作成またはパーティションの追加時に、バケット数（BUCKETS）を自動的に設定できるようになりました。バケット数は手動で設定する必要はもうありません。詳細については、[バケット数を決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table1`にレコードを挿入します。

      ```SQL
      INSERT INTO table1 VALUES
          (101, 'Lily',80);
      ```

#### データのロード

`example1.csv`の中の`id`が`101`のレコードを`table1`に更新し、`id`が`102`のレコードを`table1`に挿入するためのロードジョブを実行します。

- Stream Loadジョブを実行します。

  - `__op`フィールドを含めたくない場合は、次のコマンドを実行します。

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label1" \
        -H "column_separator:," \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

  - `__op`フィールドを含めたい場合は、次のコマンドを実行します。

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

  - `__op`フィールドを含めたくない場合は、次のコマンドを実行します。

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

  - `__op`フィールドを含めたい場合は、次のコマンドを実行します。

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

  - `__op`フィールドを含めたくない場合は、次のコマンドを実行します。

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

  - `__op`フィールドを含めたい場合は、次のコマンドを実行します。
    ```SQL
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

データのロードが完了したら、`table1` のデータをクエリして、ロードが成功したことを確認します:

```SQL
SELECT * FROM table1;
+------+-------+------+
| id   | name  | score|
+------+-------+------+
| 101  | Lily  | 100  |
| 102  | Rose  | 100  |
+------+-------+------+
2 rows in set (0.02 sec)
```

上記のクエリ結果のように、`example1.csv` の `id` が `101` のレコードが`table1`に更新され、`example1.csv` の `id` が `102` のレコードが`table1`に挿入されました。

### 削除

ロードするデータファイルが削除操作のみである場合、`__op` フィールドを追加し、操作タイプを削除に指定する必要があります。

#### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに `example2.csv` という名前の CSV ファイルを作成します。このファイルには、ユーザーID、ユーザー名、ユーザースコアの3列が含まれています。

      ```Plain
      101,Jack,100
      ```

   b. `example2.csv` のデータをKafkaクラスターの`topic2`に公開します。

2. StarRocksテーブルを準備します。

   a. StarRocksテーブル`test_db`内に、プライマリキーが`id`である`table2`という名前のプライマリキーテーブルを作成します。テーブルには、`id`、`name`、`score`の3つの列が含まれます。

      ```SQL
      CREATE TABLE `table2`
            (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NOT NULL COMMENT "user name",
          `score` int(11) NOT NULL COMMENT "user score"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **注意**
      >
      > v2.5.7以降、StarRocksはテーブルやパーティションを作成する際に、バケツの数（BUCKETS）を自動的に設定できます。バケツの数を手動で設定する必要はもうありません。詳細については、[バケツの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets) を参照してください。

   b. `table2`に2つのレコードを挿入します。

      ```SQL
      INSERT INTO table2 VALUES
      (101, 'Jack', 100),
      (102, 'Bob', 90);
      ```

#### データのロード

`example2.csv` の `id` が `101` のレコードを `table2` から削除するためのロードジョブを実行します。

- Streamロードジョブを実行します。

  ```Bash
  curl --location-trusted -u <ユーザ名>:<パスワード> \
      -H "Expect:100-continue" \
      -H "label:label3" \
      -H "column_separator:," \
      -H "columns:__op='delete'" \
      -T example2.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
  ```

- Brokerロードジョブを実行します。

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

- Routineロードジョブを実行します。

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

ロードが完了したら、`table2` のデータを確認して、ロードが成功したことを確認します:

```SQL
SELECT * FROM table2;
+------+-------+------+
| id   | name  | score|
+------+-------+------+
| 102  | Bob   |  90  |
+------+-------+------+
1 row in set (0.00 sec)
```

上記のクエリ結果のように、`example2.csv` の `id` が `101` のレコードが`table2`から削除されました。

### UPSERT および DELETE

ロードするデータファイルがUPSERTおよびDELETEの両方の操作を含む場合、`__op` フィールドを追加し、データファイルに`0`または`1`の値を持つ列を含める必要があります。`0`の値はUPSERT操作を、`1`の値はDELETE操作を示します。

#### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに `example3.csv` という名前の CSV ファイルを作成します。このファイルには、ユーザーID、ユーザー名、ユーザースコア、および操作タイプの4列が含まれます。

      ```Plain
      101,Tom,100,1
      102,Sam,70,0
      103,Stan,80,0
      ```

   b. `example3.csv` のデータをKafkaクラスターの`topic3`に公開します。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`内に、プライマリキーが`id`である`table3`という名前のプライマリキーテーブルを作成します。テーブルには、`id`、`name`、`score`の3つの列が含まれます。

      ```SQL
      CREATE TABLE `table3`
      (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NOT NULL COMMENT "user name",
          `score` int(11) NOT NULL COMMENT "user score"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **注意**
      >
      > v2.5.7以降、StarRocksはテーブルやパーティションを作成する際に、バケツの数（BUCKETS）を自動的に設定できます。バケツの数を手動で設定する必要はもうありません。詳細については、[バケツの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets) を参照してください。

   b. `table3`に2つのレコードを挿入します。

      ```SQL
      INSERT INTO table3 VALUES
      (101, 'Tom', 100),
      (102, 'Sam', 90);
      ```

#### データのロード

`example3.csv` の `id` が `101` のレコードを `table3` から削除し、`example3.csv` の `id` が `102` のレコードを `table3` に更新し、`example3.csv` の `id` が `103` のレコードを`table3`に挿入するためのロードジョブを実行します。

- Streamロードジョブを実行します:

  ```Bash
  curl --location-trusted -u <ユーザ名>:<パスワード> \
      -H "Expect:100-continue" \
      -H "label:label4" \
      -H "column_separator:," \
      -H "columns: id, name, score, temp, __op = temp" \
      -T example3.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
  ```

  > **注意**
  >
  > 上記の例では、`example3.csv` の操作タイプを表す4番目の列を一時的に`temp`として名前を付け、`__op` フィールドを`columns`パラメータを使用して`temp`列にマップしました。そのため、StarRocksは`example3.csv`の4番目の列の値が`0`か`1`かに応じてUPSERTまたはDELETE操作を実行するかどうかを決定できます。

- Brokerロードジョブを実行します:

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

- Routineロードジョブを実行します:

  ```SQL
  CREATE ROUTINE LOAD test_db.table3 ON table3  
  COLUMNS(id, name, score, __op = 'upsert')
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

#### クエリデータ

ロードが完了したら、`table3` のデータをクエリして、ロードが成功したことを確認します:

```SQL
SELECT * FROM table3;
+------+-------+------+
| id   | name  | score|
+------+-------+------+
| 101  | Tom   | 100  |
| 102  | Sam   |  70  |
| 103  | Stan  |  80  |
+------+-------+------+
3 rows in set (0.02 sec)
```

上記のクエリ結果のように、`example3.csv` の `id` が `101` のレコードが`table3`から削除され、`example3.csv` の `id` が `102` のレコードが`table3`に更新され、`example3.csv` の `id` が `103` のレコードが`table3`に挿入されました。
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

ロードが完了したら、 `table3` のデータをクエリして、ロードが正常に完了したかを確認します。

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

前述のクエリ結果から分かるように、`example3.csv` の`id`が`101`のレコードは`table3`から削除され、`example3.csv`の`id`が`102`のレコードは`table3`に更新され、`example3.csv`の`id`が`103`のレコードは`table3`に挿入されました。

## 部分更新

v2.2以降、StarRocksはプライマリキー表の指定された列のみを更新する機能をサポートしています。このセクションでは、CSVを使用して部分更新の方法を説明します。

> **注意**
>
> 部分更新を実行する際は、更新対象の行が存在しない場合、StarRocksが新しい行を挿入し、データが更新されなかったため空白のフィールドにデフォルト値を入れます。

### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example4.csv`という名前のCSVファイルを作成します。このファイルには、ユーザーIDとユーザー名を表す2つの列が含まれます。

      ```Plain
      101,Lily
      102,Rose
      103,Alice
      ```

   b. `example4.csv`のデータをKafkaクラスターの`topic4`に発行します。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table4`というプライマリキー表を作成します。このテーブルには、`id`、`name`、`score`の3つの列が含まれます。`id`はプライマリキーです。

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
      > v2.5.7以降、StarRocksはテーブルを作成したりパーティションを追加したりする際にバケツの数（BUCKETS）を自動的に設定することができます。バケツ数を手動で設定する必要はありません。詳細については、[バケツ数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table4`にレコードを挿入します。

      ```SQL
      INSERT INTO table4 VALUES
          (101, 'Tom',80);
      ```

### データをロード

`example4.csv`の2つの列のデータを`table4`の`id`と`name`の列に更新するために、ロードを実行します。

- ストリームロードジョブを実行します:

  ```Bash
  curl --location-trusted -u <ユーザ名>:<パスワード> \
      -H "Expect:100-continue" \
      -H "label:label7" -H "column_separator:," \
      -H "partial_update:true" \
      -H "columns:id,name" \
      -T example4.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
  ```

  > **注意**
  >
  > ストリームロードを選択する場合、部分更新機能を有効にするために`partial_update`パラメータを`true`に設定する必要があります。さらに、更新対象の列を指定するために`columns`パラメータを使用する必要があります。

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

  > **注意**
  >
  > ブローカーロードを選択する場合、部分更新機能を有効にするために`partial_update`パラメータを`true`に設定する必要があります。さらに、更新対象の列を指定するために`column_list`パラメータを使用する必要があります。

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

  > **注意**
  >
  > ブローカーロードを選択する場合、部分更新機能を有効にするために`partial_update`パラメータを`true`に設定する必要があります。さらに、更新対象の列を指定するために`COLUMNS`パラメータを使用する必要があります。

### データをクエリ

ロードが完了したら、`table4`のデータをクエリして、ロードが正常に完了したかを確認します:

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

前述のクエリ結果から分かるように、`example4.csv`の`id`が`101`のレコードは`table4`に更新され、`example4.csv`の`id`が`102`と`103`のレコードは`table4`に挿入されました。

## 条件付き更新

StarRocks v2.5以降、プライマリキー表は条件付き更新をサポートしています。特定の非プライマリキー列を条件として指定することで、更新が有効になる条件を決定できます。これにより、指定された列で既存データレコードよりも新しいデータレコードは、新たなデータによって上書きされないようになります。

> **注意**
>
> - 同じデータのバッチに対して異なる列を更新条件として指定することはできません。
> - DELETE操作は条件付き更新をサポートしません。
> - v3.1.3以前のバージョンでは、部分更新と条件付き更新を同時に使用することはできませんが、v3.1.3以降、StarRocksは部分更新と条件付き更新を同時に使用することができます。
> - 条件付き更新はStream LoadとRoutine Loadのみをサポートしています。

### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに`example5.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、バージョン、ユーザースコアを表す3つの列が含まれます。

      ```Plain
      101,1,100
      102,3,100
      ```

   b. `example5.csv`のデータをKafkaクラスターの`topic5`に発行します。

2. StarRocksテーブルを準備します。

   a. StarRocksデータベース`test_db`に`table5`というプライマリキー表を作成します。このテーブルには、`id`、`version`、`score`の3つの列が含まれます。`id`はプライマリキーです。

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
      > v2.5.7以降、StarRocksはテーブルを作成したりパーティションを追加したりする際にバケツの数（BUCKETS）を自動的に設定することができます。バケツ数を手動で設定する必要はありません。詳細については、[バケツ数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   b. `table5`にレコードを挿入します。

      ```SQL
      INSERT INTO table5 VALUES
          (101, 2, 80),
          (102, 2, 90);
      ```

### データをロード
```yaml
      + {R}
      + {R}
    + {R}
  + {R}
```

```yaml
      + {T}
      + {T}
    + {T}
  + {T}
```