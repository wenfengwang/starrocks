---
displayed_sidebar: Chinese
---

# データ変更を実現するためのインポート

StarRocks の[プライマリキーモデル](../table_design/table_types/primary_key_table.md)は、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、または [Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) インポートジョブを通じて、StarRocks テーブルに対するデータ変更をサポートしています。これには、データの挿入、更新、削除が含まれます。[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) インポートジョブや [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) ステートメントを使用して StarRocks テーブルに対するデータ変更はサポートされていません。

StarRocks は、部分更新 (Partial Update) と条件付き更新 (Conditional Update) もサポートしています。

> **注意**
>
> インポート操作には、対象テーブルの INSERT 権限が必要です。もし INSERT 権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を参照してユーザーに権限を付与してください。

この記事では、CSV 形式のデータファイルを例に、インポートを通じてデータ変更を実現する方法を紹介します。具体的にサポートされるデータファイルのタイプは、選択したインポート方法によって異なります。

> **説明**
>
> CSV 形式のデータについて、StarRocks は最大 50 バイトの UTF-8 エンコード文字列を列区切り文字として設定することをサポートしており、これにはカンマ (,)、タブ、パイプ (|) などが含まれます。

## 内部実装

StarRocks のプライマリキーモデルは現在、UPSERT と DELETE 操作をサポートしており、INSERT と UPDATE 操作を区別することはできません。

インポートジョブを作成する際、StarRocks はインポートジョブの作成ステートメントまたはコマンドに `__op` フィールドを追加することをサポートしており、これにより操作タイプを指定できます。

> **説明**
>
> StarRocks テーブルを作成する際に `__op` 列を追加する必要はありません。

インポート方法によって `__op` フィールドの定義方法が異なります：

- Stream Load インポートを使用する場合は、`columns` パラメータを通じて `__op` フィールドを定義する必要があります。

- Broker Load インポートを使用する場合は、SET 句を通じて `__op` フィールドを定義する必要があります。

- Routine Load インポートを使用する場合は、`COLUMNS` パラメータを通じて `__op` フィールドを定義する必要があります。

データ変更操作を行う場合、`__op` フィールドを追加するかどうかを選択できます。`__op` フィールドを追加しない場合、デフォルトで UPSERT 操作となります。主なデータ変更操作シナリオは以下の通りです：

- データファイルが UPSERT 操作のみを含む場合は、`__op` フィールドを追加する必要はありません。

- データファイルが DELETE 操作のみを含む場合は、`__op` フィールドを追加し、操作タイプを DELETE として指定する必要があります。

- データファイルに UPSERT と DELETE 操作の両方が含まれている場合は、`__op` フィールドを追加し、データファイルに操作タイプを表す列が含まれていることを確認する必要があります。ここで、`0` は UPSERT 操作を、`1` は DELETE 操作を表します。

## 使用説明

- インポートするデータファイルの各行の列数が同じであることを確認する必要があります。

- 更新される列にはプライマリキー列が含まれている必要があります。

## 前提条件

### Broker Load

[从 HDFS 导入](../loading/hdfs_load.md)または[从云存储导入](../loading/cloud_storage_load.md)の「背景情報」セクションを参照してください。

### Routine Load

Routine Load を使用してデータをインポートする場合、Apache Kafka® クラスターに Topic が作成されていることを確認する必要があります。この記事では、`topic1`、`topic2`、`topic3`、`topic4` の 4 つの Topic がデプロイされていると仮定しています。

## 基本操作

以下の例を通じて、具体的なインポート操作を示します。Stream Load、Broker Load、Routine Load を使用してデータをインポートする詳細な構文とパラメータについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

### UPSERT

データファイルが UPSERT 操作のみを含む場合は、`__op` フィールドを追加する必要はありません。

`__op` フィールドを追加する場合：

- `__op` を UPSERT 操作として指定することができます。

- 何も指定せずに、StarRocks はデフォルトで UPSERT 操作としてインポートします。

#### データ例

1. データファイルを準備します。

   a. ローカルファイルシステムに CSV 形式のデータファイル `example1.csv` を作成します。ファイルにはユーザー ID、ユーザー名、ユーザースコアを表す 3 列が含まれています。以下のようになります：

      ```Plain
      101,Lily,100
      102,Rose,100
      ```

   b. `example1.csv` ファイルのデータを Kafka クラスターの `topic1` にアップロードします。

2. StarRocks テーブルを準備します。

   a. `test_db` データベースに `table1` という名前のプライマリキーモデルテーブルを作成します。テーブルには `id`、`name`、`score` の 3 列が含まれ、それぞれユーザー ID、ユーザー名、ユーザースコアを表し、プライマリキーは `id` 列です。以下のようになります：

      ```SQL
      CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "ユーザー ID",
          `name` varchar(65533) NOT NULL COMMENT "ユーザー名",
          `score` int(11) NOT NULL COMMENT "ユーザースコア"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

      > **説明**
      >
      > バージョン 2.5.7 以降、StarRocks はテーブル作成とパーティション追加時に自動的にバケット数 (BUCKETS) を設定することをサポートしており、手動でバケット数を設定する必要はありません。詳細は [バケット数の決定](../table_design/Data_distribution.md#确定分桶数量) を参照してください。

   b. `table1` テーブルに以下のデータを挿入します：

      ```SQL
      INSERT INTO table1 VALUES
          (101, 'Lily',80);
      ```

#### データインポート

インポートを通じて、`example1.csv` ファイルの `id` が `101` のデータを `table1` テーブルに更新し、`example1.csv` ファイルの `id` が `102` のデータを `table1` テーブルに挿入します。

- Stream Load を使用してインポートする場合：
  - `__op` フィールドを追加しない：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label1" \
        -H "column_separator:," \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

  - `__op` フィールドを追加する：

    ```Bash
    curl --location-trusted -u <username>:<password> \
        -H "Expect:100-continue" \
        -H "label:label2" \
        -H "column_separator:," \
        -H "columns:__op ='upsert'" \
        -T example1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
    ```

- Broker Load を使用してインポートする場合：

  - `__op` フィールドを追加しない：

    ```SQL
    LOAD LABEL test_db.label1
    (
        data infile("hdfs://<hdfs_host>:<hdfs_port>/example1.csv")
        into table table1
        columns terminated by ","
        format as "csv"
    )
    with broker
    ```

  - `__op` フィールドを追加する：

    ```SQL
    LOAD LABEL test_db.label2
    (
        data infile("hdfs://<hdfs_host>:<hdfs_port>/example1.csv")
        into table table1
        columns terminated by ","
        format as "csv"
        set (__op = 'upsert')
    )
    with broker
    ```

- Routine Load を使用してインポートする場合：

  - `__op` フィールドを追加しない：

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

  - `__op` フィールドを追加する：

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
        "property.kafka_default_offsets" = "OFFSET_BEGINNING"
    );
    ```

#### データのクエリ

インポートが完了した後、`table1` のデータを以下のようにクエリします：

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

クエリ結果から、`example1.csv` ファイルの `id` が `101` のデータが `table1` に更新され、`id` が `102` のデータが `table1` に挿入されたことがわかります。

### DELETE

データファイルが DELETE 操作のみを含む場合、`__op` フィールドを追加し、操作タイプを DELETE として指定する必要があります。

#### データサンプル

1. データファイルの準備。

   a. ローカルファイルシステムに CSV 形式のデータファイル `example2.csv` を作成します。このファイルには3つの列が含まれ、それぞれユーザーID、ユーザー名、ユーザースコアを表します。以下のようになります：

      ```Plain
      101,Jack,100
      ```

   b. `example2.csv` ファイルのデータを Kafka クラスタの `topic2` にアップロードします。

2. StarRocks テーブルの準備。

   a. `test_db` データベースに `table2` という名前のプライマリキーモデルのテーブルを作成します。テーブルには `id`、`name`、`score` の3つの列が含まれ、それぞれユーザーID、ユーザー名、ユーザースコアを表し、プライマリキーは `id` 列です。以下のようになります：

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

      > **説明**
      >
      > 2.5.7 バージョンから、StarRocks はテーブル作成とパーティション追加時に自動的にバケット数 (BUCKETS) を設定する機能をサポートしています。手動でバケット数を設定する必要はありません。詳細は [バケット数の決定](../table_design/Data_distribution.md#バケット数の決定) を参照してください。

   b. `table2` にデータを挿入します。以下のようになります：

      ```SQL
      INSERT INTO table2 VALUES
          (101, 'Jack', 100),
          (102, 'Bob', 90);
      ```

#### データのインポート

インポートを通じて、`example2.csv` ファイルの `id` が `101` のデータを `table2` から削除します。

- Stream Load を使用してインポート：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label3" \
      -H "column_separator:," \
      -H "columns:__op='delete'" \
      -T example2.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
  ```

- Broker Load を使用してインポート：

  ```SQL
  LOAD LABEL test_db.label3
  (
      data infile("hdfs://<hdfs_host>:<hdfs_port>/example2.csv")
      into table table2
      columns terminated by ","
      format as "csv"
      set (__op = 'delete')
  )
  with broker  
  ```

- Routine Load を使用してインポート：

  ```SQL
  CREATE ROUTINE LOAD test_db.table2 ON table2
  COLUMNS(id, name, score, __op = 'delete')
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
      "kafka_topic" = "test2",
      "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  );
  ```

#### データのクエリ

インポートが完了した後、`table2` のデータを以下のようにクエリします：

```SQL
SELECT * FROM table2;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|  102 | Bob  |    90 |
+------+------+-------+
1 row in set (0.00 sec)
```

クエリ結果から、`example2.csv` ファイルの `id` が `101` のデータが `table2` から削除されたことがわかります。

### UPSERT と DELETE

データファイルに UPSERT と DELETE の操作が同時に含まれる場合、`__op` フィールドを追加し、操作タイプを表す列がデータファイルに含まれていることを確認する必要があります。この列の値は `0` または `1` で、`0` は UPSERT 操作を、`1` は DELETE 操作を表します。

#### データサンプル

1. データファイルの準備。

   a. ローカルファイルシステムに CSV 形式のデータファイル `example3.csv` を作成します。このファイルには4つの列が含まれ、それぞれユーザーID、ユーザー名、ユーザースコア、操作タイプを表します。以下のようになります：

      ```Plain
      101,Tom,100,1
      102,Sam,70,0
      103,Stan,80,0
      ```

   b. `example3.csv` ファイルのデータを Kafka クラスタの `topic3` にアップロードします。

2. StarRocks テーブルの準備。

   a. `test_db` データベースに `table3` という名前のプライマリキーモデルのテーブルを作成します。テーブルには `id`、`name`、`score` の3つの列が含まれ、それぞれユーザーID、ユーザー名、ユーザースコアを表し、プライマリキーは `id` 列です。以下のようになります：

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

      > **説明**
      >
      > 2.5.7 バージョンから、StarRocks はテーブル作成とパーティション追加時に自動的にバケット数 (BUCKETS) を設定する機能をサポートしています。手動でバケット数を設定する必要はありません。詳細は [バケット数の決定](../table_design/Data_distribution.md#バケット数の決定) を参照してください。

   b. `table3` にデータを挿入します。以下のようになります：

      ```SQL
      INSERT INTO table3 VALUES
          (101, 'Tom', 100),
          (102, 'Sam', 90);
      ```

#### データのインポート

インポートを通じて、`example3.csv` ファイルの `id` が `101` のデータを `table3` から削除し、`id` が `102` のデータを `table3` に更新し、`id` が `103` のデータを `table3` に挿入します：

- Stream Load を使用してインポート：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label4" \
      -H "column_separator:," \
      -H "columns: id, name, score, temp, __op = temp" \
      -T example3.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
  ```

  > **説明**
  >
  > 上記の例では、`columns` パラメータを使用して `example3.csv` ファイルの第4列を `temp` として一時的に命名し、その後 `__op` フィールドを `temp` 列と同じ値に設定します。これにより、StarRocks は `example3.csv` ファイルの第4列の値が `0` か `1` かに基づいて、UPSERT 操作か DELETE 操作かを判断します。

- Broker Load を使用してインポート：

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
  with broker
  ```

- Routine Load を使用してインポート：

  ```SQL
  CREATE ROUTINE LOAD test_db.table3 ON table3
  COLUMNS(id, name, score, temp, __op = temp)
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
      "kafka_topic" = "test3",
      "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  );
  ```

#### データのクエリ

インポートが完了した後、`table3` のデータを以下のようにクエリします：

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
`example3.csv`ファイルの`id`が`101`のデータは`table3`から削除され、`id`が`102`のデータは`table3`に更新され、`id`が`103`のデータは`table3`に挿入されました。

## 部分更新

StarRocks v2.2以降、主キーモデルのテーブルは部分更新（Partial Update）をサポートしています。特定の列のみを更新することを選択できます。ここではCSV形式のデータファイルを例に説明します。

> **注意**
>
> 部分更新モードでは、更新する行が存在しない場合、StarRocksは新しい行を挿入し、欠落している列にデフォルト値を自動的に埋めます。

### データ例

1. データファイルの準備。

   a. ローカルファイルシステムに`example4.csv`というCSV形式のデータファイルを作成します。ファイルには2列が含まれ、それぞれユーザーIDとユーザー名を表しています。以下のようになります：

      ```Plain
      101,Lily
      102,Rose
      103,Alice
      ```

   b. `example4.csv`ファイルのデータをKafkaクラスタの`topic4`にアップロードします。

2. StarRocksテーブルの準備。

   a. `test_db`データベースに`table4`という名前の主キーモデルテーブルを作成します。テーブルには`id`、`name`、`score`の3列が含まれ、それぞれユーザーID、ユーザー名、ユーザースコアを表し、主キーは`id`列です。以下のようになります：

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

      > **説明**
      >
      > 2.5.7バージョン以降、StarRocksはテーブル作成とパーティション追加時にバケット数（BUCKETS）を自動的に設定することをサポートしています。バケット数を手動で設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#バケット数の決定)を参照してください。

   b. `table4`に以下のデータを挿入します：

      ```SQL
      INSERT INTO table4 VALUES
          (101, 'Tom', 80);
      ```

### データのインポート

インポートを通じて、`example4.csv`の2列のデータを`table4`の`id`と`name`の2列に更新します。

- Stream Loadを使用してインポート：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label7" -H "column_separator:," \
      -H "partial_update:true" \
      -H "columns:id,name" \
      -T example4.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
  ```

  > **説明**
  >
  > Stream Loadを使用してデータをインポートする場合、`partial_update`を`true`に設定して部分更新機能を有効にする必要があります。また、`columns`で更新するデータの列名を宣言する必要があります。

- Broker Loadを使用してインポート：

  ```SQL
  LOAD LABEL test_db.table4
  (
      DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/example4.csv")
      INTO TABLE table4
      FORMAT AS "csv"
      (id, name)
  )
  WITH BROKER
  PROPERTIES
  (
      "partial_update" = "true"
  );
  ```

  > **説明**
  >
  > Broker Loadを使用してデータをインポートする場合、`partial_update`を`true`に設定して部分更新機能を有効にする必要があります。また、`column_list`で更新するデータの列名を宣言する必要があります。

- Routine Loadを使用してインポート：

  ```SQL
  CREATE ROUTINE LOAD test_db.table4 ON table4
  COLUMNS (id, name),
  COLUMNS TERMINATED BY ','
  PROPERTIES
  (
      "partial_update" = "true"
  )
  FROM KAFKA
  (
      "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "test4",
      "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  );
  ```

  > **説明**
  >
  > Routine Loadを使用してデータをインポートする場合、`partial_update`を`true`に設定して部分更新機能を有効にする必要があります。また、`COLUMNS`で更新するデータの列名を宣言する必要があります。

### データのクエリ

インポートが完了した後、`table4`のデータを以下のようにクエリします：

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

クエリの結果から、`example4.csv`ファイルの`id`が`101`のデータは`table4`に更新され、`id`が`102`と`103`のデータは`table4`に挿入されたことがわかります。

## 条件付き更新

StarRocks v2.5以降、主キーモデルのテーブルは条件付き更新（Conditional Update）をサポートしています。更新条件として特定の非主キー列を指定でき、インポートされたデータのその列の値が現在の値以上の場合にのみ更新が有効になります。

条件付き更新機能は、データの順序が乱れた問題を解決するために使用されます。上流のデータに乱れがある場合、条件付き更新機能を使用して、新しいデータが古いデータによって上書きされないように保証できます。

> **説明**
>
> - 同一バッチのインポートデータに異なる条件を指定することはできません。
>
> - 削除操作はサポートされていません。
>
> - 3.1.3バージョン以前では、StarRocksは条件付き更新と部分更新を同時に使用することはサポートされていませんでした。3.1.3バージョン以降から、条件付き更新と部分更新を同時に使用できるようになりました。
>

### データ例

1. データファイルの準備。

   a. ローカルファイルシステムに`example5.csv`というCSV形式のデータファイルを作成します。ファイルには3列が含まれ、それぞれユーザーID、バージョン番号、ユーザースコアを表しています。以下のようになります：

      ```Plain
      101,1,100
      102,3,100
      ```

   b. `example5.csv`ファイルのデータをKafkaクラスタの`topic5`にアップロードします。

2. StarRocksテーブルの準備。

   a. `test_db`データベースに`table5`という名前の主キーモデルテーブルを作成します。テーブルには`id`、`version`、`score`の3列が含まれ、それぞれユーザーID、バージョン番号、ユーザースコアを表し、主キーは`id`列です。以下のようになります：

      ```SQL
      CREATE TABLE `table5`
      (
          `id` int(11) NOT NULL COMMENT "ユーザーID",
          `version` int NOT NULL COMMENT "バージョン番号",
          `score` int(11) NOT NULL COMMENT "ユーザースコア"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`) DISTRIBUTED BY HASH(`id`);
      ```

      > **説明**
      >
      > 2.5.7バージョン以降、StarRocksはテーブル作成とパーティション追加時にバケット数（BUCKETS）を自動的に設定することをサポートしています。バケット数を手動で設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#バケット数の決定)を参照してください。

   b. `table5`に以下のデータを挿入します：

      ```SQL
      INSERT INTO table5 VALUES
          (101, 2, 80),
          (102, 2, 90);
      ```

### データのインポート

インポートを通じて、`example5.csv`ファイルの`id`が`101`および`102`のデータを`table5`に更新します。`merge_condition`を`version`列として指定し、インポートされたデータの`version`が`table5`の対応する行の`version`以上の場合にのみ更新が有効になるようにします。

- Stream Loadを使用してインポート：

  ```Bash
  curl --location-trusted -u <username>:<password> \
      -H "Expect:100-continue" \
      -H "label:label10" \
      -H "column_separator:," \
      -H "merge_condition:version" \
      -T example5.csv -XPUT \
      http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
  ```

- Routine Loadを使用してインポート：

  ```SQL
  CREATE ROUTINE LOAD test_db.table5 ON table5
  COLUMNS (id, version, score),
  COLUMNS TERMINATED BY ','
  PROPERTIES
  (
      "merge_condition" = "version"
  )
  FROM KAFKA
  (
      "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
      "kafka_topic" = "topic5",
      "property.kafka_default_offsets" = "OFFSET_BEGINNING"
  );
  ```

- Broker Loadを使用してインポート：

  ```SQL
  LOAD LABEL test_db.table5
  ( DATA INFILE ("s3://xxx.csv")
    INTO TABLE table5 COLUMNS TERMINATED BY "," FORMAT AS "CSV"
  )
  WITH BROKER
  PROPERTIES
  (
      "merge_condition" = "version"
  );
  ```

### データのクエリ

インポートが完了した後、`table5` のデータを以下のようにクエリします：

```SQL
SELECT * FROM table5;
+------+------+-------+
| id   | version | score |
+------+------+-------+
|  101 |       2 |   80 |
|  102 |       3 |  100 |
+------+------+-------+
2行がセットされました (0.02秒)
```

クエリ結果から、`example5.csv` ファイルの `id` が `101` のデータは更新されていないことがわかりますが、`id` が `102` のデータは更新されました。
