---
displayed_sidebar: "Japanese"
---

# データのロード時にデータを変換する

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksはデータのロード時にデータ変換をサポートしています。

この機能は[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)をサポートしていますが、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)をサポートしていません。

<InsertPrivNote />

このトピックでは、CSVデータを使用して、読み込み時にデータを抽出および変換する方法を説明します。選択したロード方法に応じてサポートされるデータファイルの形式は異なります。

> **注記**
>
> CSVデータの場合、テキストの区切り記号として、UTF-8文字列（例: カンマ(,)、タブ、またはパイプ(|)）を使用できます。長さは50バイト以下とします。

## シナリオ

StarRocksテーブルにデータファイルをロードする際、データファイルのデータが完全にStarRocksテーブルのデータにマップされない場合、StarRocksテーブルにロードする前にデータを抽出または変換する必要はありません。StarRocksはロード中にデータを抽出および変換することができます。

- ロードが必要ない列をスキップする
  
  ロードが不要な列をスキップすることができます。また、データファイルの列がStarRocksテーブルの列と異なる順序にある場合、データファイルとStarRocksテーブルの間で列マッピングを作成することができます。

- ロードが不要な行をフィルタリングする
  
  ロードが不要な行をフィルタリングするためのフィルター条件を指定できます。

- 元の列から新しい列を生成する
  
  生成列は、データファイルの元の列から計算される特別な列です。生成列をStarRocksテーブルの列にマッピングすることができます。

- ファイルパスからパーティションフィールド値を抽出する
  
  データファイルがApache Hive™で生成された場合、ファイルパスからパーティションフィールド値を抽出できます。

## 前提条件

### Broker Load

[HDFSからデータをロード](../loading/hdfs_load.md)または[クラウドストレージからデータをロード](../loading/cloud_storage_load.md)の「背景情報」セクションを参照してください。

### Routine Load

[Routine Load](./RoutineLoad.md)を選択する場合、Apache Kafka®クラスターにトピックが作成されていることを確認してください。`topic1`と`topic2`の2つのトピックが作成されていると仮定します。

## データ例

1. ローカルファイルシステムでデータファイルを作成します。

   a. `file1.csv`という名前のデータファイルを作成します。このファイルには、ユーザーID、ユーザーの性別、イベント日付、およびイベントタイプを表す4つの列が含まれています。

      ```Plain
      354,female,2020-05-20,1
      465,male,2020-05-21,2
      576,female,2020-05-22,1
      687,male,2020-05-23,2
      ```

   b. `file2.csv`という名前のデータファイルを作成します。このファイルには、日付を表す1つの列のみが含まれています。

      ```Plain
      2020-05-20
      2020-05-21
      2020-05-22
      2020-05-23
      ```

2. StarRocksデータベース`test_db`にテーブルを作成します。

   > **注記**
   >
   > v2.5.7以降、StarRocksはテーブルを作成したりパーティションを追加した際に、バケツの数（BUCKETS）を自動的に設定できるようになりました。これでバケツの数を手動で設定する必要がなくなりました。詳細な情報については、[バケツの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   a. `table1`という名前のテーブルを作成します。このテーブルには、`event_date`、`event_type`、および`user_id`の3つの列が含まれています。

      ```SQL
      MySQL [test_db]> CREATE TABLE table1
      (
          `event_date` DATE COMMENT "イベント日付",
          `event_type` TINYINT COMMENT "イベントタイプ",
          `user_id` BIGINT COMMENT "ユーザーID"
      )
      DISTRIBUTED BY HASH(user_id);
      ```

   b. `table2`という名前のテーブルを作成します。このテーブルには、`date`、`year`、`month`、`day`の4つの列が含まれています。

      ```SQL
      MySQL [test_db]> CREATE TABLE table2
      (
          `date` DATE COMMENT "日付",
          `year` INT COMMENT "年",
          `month` TINYINT COMMENT "月",
          `day` TINYINT COMMENT "日"
      )
      DISTRIBUTED BY HASH(date);
      ```

3. `file1.csv`および`file2.csv`をHDFSクラスターの`/user/starrocks/data/input/`パスにアップロードし、`file1.csv`のデータをKafkaクラスターの`topic1`に公開し、`file2.csv`のデータをKafkaクラスターの`topic2`に公開します。

## ロードが不要な列をスキップする

StarRocksテーブルにロードしたいデータファイルには、StarRocksテーブルの列にマップできない列が含まれる場合があります。この場合、StarRocksはデータファイルの列からStarRocksテーブルの列にマップできる列のみをロードすることをサポートしています。

この機能は、次のデータソースからのデータのロードをサポートしています:

- ローカルファイルシステム

- HDFSおよびクラウドストレージ
  
  > **注記**
  >
  > このセクションではHDFSを例としています。

- Kafka

ほとんどの場合、CSVファイルの列には名前が付いていません。一部のCSVファイルでは、最初の行が列名で構成されていますが、StarRocksは最初の行の内容を列名ではなく一般データとして処理します。そのため、CSVファイルをロードする際には、一時的にCSVファイルの列をジョブ作成ステートメントまたはコマンドで**順に**名前を付ける必要があります。これらの一時的に名前を付けられた列は、StarRocksテーブルの列と**名前で**マッピングされます。データファイルの列に関する次の点に注意してください。

- StarRocksテーブルの列にマッピングでき、一時的にStarRocksテーブルの列の名前を使用して名前が付けられた列のデータは直接ロードされます。

- StarRocksテーブルの列にマッピングできない列は無視され、これらの列のデータはロードされません。

- StarRocksテーブルの列にマッピングできるが、ジョブ作成ステートメントまたはコマンドで一時的な名前を付けられていない列がある場合、ロードジョブはエラーを報告します。

このセクションでは、`file1.csv`と`table1`を例にしています。`file1.csv`の4つの列は、順番に`user_id`、`user_gender`、`event_date`、`event_type`として一時的に名前が付けられています。`file1.csv`の一時的に名前が付けられた列のうち、`user_id`、`event_date`、`event_type`は`table1`の特定の列にマッピングできますが、`user_gender`は`table1`の任意の列にマッピングできません。そのため、`user_id`、`event_date`、`event_type`は`table1`にロードされますが、`user_gender`はロードされません。

### データのロード

#### ローカルファイルシステムからデータをロードする

`file1.csv`がローカルファイルシステムに保存されている場合、次のコマンドを実行して[Stream Load](../loading/StreamLoad.md)ジョブを作成します:

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: user_id, user_gender, event_date, event_type" \
    -T file1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

> **注記**
>
> Stream Loadを選択した場合は、データファイルの列とStarRocksテーブルの列の間で列マッピングを作成するために`columns`パラメータを使用する必要があります。

詳細な構文およびパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

#### HDFSクラスターからデータをロードする

`file1.csv`がHDFSクラスターに保存されている場合、次のステートメントを実行して[Broker Load](../loading/hdfs_load.md)ジョブを作成します:

```SQL
LOAD LABEL test_db.label1
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/file1.csv")
    INTO TABLE `table1`
    FORMAT AS "csv"
    COLUMNS TERMINATED BY ","
    (user_id, user_gender, event_date, event_type)
)
WITH BROKER "broker1";
```

> **注記**
>
> Broker Loadを選択した場合は、データファイルの列とStarRocksテーブルの列の間で列マッピングを作成するために`column_list`パラメータを使用する必要があります。

詳細な構文およびパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Kafkaクラスターからデータをロードする
"file1.csv"のデータがKafkaクラスターの "topic1" に公開されている場合、「[Routine Load](../loading/RoutineLoad.md)」ジョブを作成するには、次のステートメントを実行します。

```SQL
CREATE ROUTINE LOAD test_db.table101 ON table1
    COLUMNS TERMINATED BY ",",
    COLUMNS(user_id, user_gender, event_date, event_type)
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
    "kafka_topic" = "topic1",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

> **注意**
>
> 「Routine Load」を選択する場合、データファイルの列を一時的に名前を付けるために `COLUMNS` パラメータを使用して、「Routine Load」ジョブのデータファイルとStarRocksテーブルの間に列マッピングを作成する必要があります。

詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### データのクエリ

ローカルファイルシステム、HDFSクラスター、またはKafkaクラスターからのデータの読み込みが完了したら、「table1」のデータをクエリして、読み込みが成功したことを確認します。

```SQL
MySQL [test_db]> SELECT * FROM table1;
+------------+------------+---------+
| event_date | event_type | user_id |
+------------+------------+---------+
| 2020-05-22 |          1 |     576 |
| 2020-05-20 |          1 |     354 |
| 2020-05-21 |          2 |     465 |
| 2020-05-23 |          2 |     687 |
+------------+------------+---------+
4 行が表示されました (0.01 秒)
```

## 読み込む必要のない行を除外する

StarRocksテーブルにデータファイルを読み込む際に、データファイルの特定の行を読み込みたくない場合、`WHERE`句を使用して読み込む行を指定できます。StarRocksは、`WHERE`句で指定されたフィルタ条件を満たさないすべての行をフィルタリングします。

この機能では、次のデータソースからデータを読み込むことができます。

- ローカルファイルシステム

- HDFSおよびクラウドストレージ
  > **注意**
  >
  > このセクションでは、HDFSを例として使用しています。

- Kafka

このセクションでは、「file1.csv」と「table1」を例として使用しています。「file1.csv」から「table1」にロードするときに、イベントタイプが `1` の行のみをロードしたい場合、「event_type = 1」というフィルタ条件を指定することができます。

### データの読み込み

#### ローカルファイルシステムからデータを読み込む

「file1.csv」がローカルファイルシステムに保存されている場合、次のコマンドを実行して「Stream Load](../loading/StreamLoad.md)」ジョブを作成します。

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: user_id, user_gender, event_date, event_type" \
    -H "where: event_type=1" \
    -T file1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

#### HDFSクラスターからデータを読み込む

「file1.csv」がHDFSクラスターに保存されている場合、次のステートメントを実行して「Broker Load」ジョブを作成します。

```SQL
LOAD LABEL test_db.label2
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/file1.csv")
    INTO TABLE `table1`
    FORMAT AS "csv"
    COLUMNS TERMINATED BY ","
    (user_id, user_gender, event_date, event_type)
    WHERE event_type = 1
)
WITH BROKER "broker1";
```

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Kafkaクラスターからデータを読み込む

「file1.csv」のデータがKafkaクラスターの "topic1" に公開されている場合、「Routine Load](../loading/RoutineLoad.md)」ジョブを作成するには、次のステートメントを実行します。

```SQL
CREATE ROUTINE LOAD test_db.table102 ON table1
COLUMNS TERMINATED BY ",",
COLUMNS (user_id, user_gender, event_date, event_type)
WHERE event_type = 1
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
    "kafka_topic" = "topic1",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### データのクエリ

ローカルファイルシステム、HDFSクラスター、またはKafkaクラスターからのデータの読み込みが完了したら、「table1」のデータをクエリして、読み込みが成功したことを確認します。

```SQL
MySQL [test_db]> SELECT * FROM table1;
+------------+------------+---------+
| event_date | event_type | user_id |
+------------+------------+---------+
| 2020-05-20 |          1 |     354 |
| 2020-05-22 |          1 |     576 |
+------------+------------+---------+
2 行が表示されました (0.01 秒)
```

## オリジナルの列から新しい列を生成する

StarRocksテーブルにデータファイルをロードする際、データファイルの一部のデータは、StarRocksテーブルにロードする前に変換が必要な場合があります。この場合、ジョブ作成コマンドまたはステートメントで関数や式を使用してデータ変換を実装できます。

この機能では、次のデータソースからデータを読み込むことができます。

- ローカルファイルシステム

- HDFSおよびクラウドストレージ
  > **注意**
  >
  > このセクションでは、HDFSを例として使用しています。

- Kafka

このセクションでは、「file2.csv」と「table2」を例として使用しています。「file2.csv」は、日付を表す1列だけから構成されています。[year](../sql-reference/sql-functions/date-time-functions/year.md)、[month](../sql-reference/sql-functions/date-time-functions/month.md)、および[day](../sql-reference/sql-functions/date-time-functions/day.md)関数を使用して、「file2.csv」の各日付から年、月、日を抽出し、「table2」の「year」、「month」、「day」の列に抽出されたデータをロードできます。

### データの読み込み

#### ローカルファイルシステムからデータを読み込む

「file2.csv」がローカルファイルシステムに保存されている場合、次のコマンドを実行して「Stream Load](../loading/StreamLoad.md)」ジョブを作成します。

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns:date,year=year(date),month=month(date),day=day(date)" \
    -T file2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

> **注意**
>
> - 「columns」パラメータでは、まずデータファイルの**すべての列**に一時的な名前を付け、次にデータファイルのオリジナルの列から生成された新しい列に一時的な名前を付ける必要があります。前述の例のように、 `file2.csv` の唯一の列は一時的に `date` として名前が付けられており、 `year=date(date)`、`month=date(date)`、および `day=date(date)` 関数が呼び出され、それらが一時的に `year`、`month`、`day` として名前が付けられた新しい列を生成します。
>
> - Stream Loadは、「column_name = function(column_name)」ではなく、「column_name = function(column_name)」をサポートしています。

詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

#### HDFSクラスターからデータを読み込む

「file2.csv」がHDFSクラスターに保存されている場合、次のステートメントを実行して「Broker Load」ジョブを作成します。

```SQL
LOAD LABEL test_db.label3
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/file2.csv")
    INTO TABLE `table2`
    FORMAT AS "csv"
    COLUMNS TERMINATED BY ","
    (date)
    SET(year=year(date), month=month(date), day=day(date))
)
WITH BROKER "broker1";
```

> **注意**
>
>まず、`column_list`パラメータを使用して、データファイルの**すべての列**に一時的な名前を付け、その後、SET句を使用して、データファイルの元の列から生成したい新しい列に一時的な名前を付ける必要があります。前述の例では、`file2.csv`の唯一の列が`column_list`パラメータで一時的に`date`として名前が付けられ、次にSET句で`year=year(date)`、`month=month(date)`、`day=day(date)`関数が呼び出されて、一時的に`year`、`month`、`day`として名前が付けられた3つの新しい列が生成されています。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Kafkaクラスターからデータをロード

`file2.csv`のデータがKafkaクラスターの`topic2`に公開されている場合、次のステートメントを実行して[Routine Load](../loading/RoutineLoad.md)ジョブを作成します。

```SQL
CREATE ROUTINE LOAD test_db.table201 ON table2
    COLUMNS TERMINATED BY ",",
    COLUMNS(date,year=year(date),month=month(date),day=day(date))
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
    "kafka_topic" = "topic2",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

> **注記**
>
> `COLUMNS`パラメータでは、データファイルの**すべての列**に一時的な名前を付け、次にデータファイルの元の列から生成したい新しい列に一時的な名前を付ける必要があります。前述の例では、`file2.csv`の唯一の列が一時的に`date`として名前が付けられ、次に`year=year(date)`、`month=month(date)`、`day=day(date)`関数が呼び出されて、一時的に`year`、`month`、`day`として名前が付けられた3つの新しい列が生成されています。

詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### データをクエリ

ローカルファイルシステム、HDFSクラスター、またはKafkaクラスターからのデータのロードが完了した後、`table2`のデータをクエリしてロードが成功したことを確認してください。

```SQL
MySQL [test_db]> SELECT * FROM table2;
+------------+------+-------+------+
| date       | year | month | day  |
+------------+------+-------+------+
| 2020-05-20 | 2020 |  5    | 20   |
| 2020-05-21 | 2020 |  5    | 21   |
| 2020-05-22 | 2020 |  5    | 22   |
| 2020-05-23 | 2020 |  5    | 23   |
+------------+------+-------+------+
4 行が選択されました (0.01 秒)
```

## ファイルパスからパーティションフィールドの値を抽出する

指定したファイルパスにパーティションフィールドが含まれている場合、`COLUMNS FROM PATH AS`パラメータを使用して、ファイルパスから抽出したいパーティションフィールドを指定できます。ファイルパスのパーティションフィールドは、データファイルの列と同等です。`COLUMNS FROM PATH AS`パラメータは、データをHDFSクラスターからロードする場合のみサポートされています。

たとえば、Hiveから生成された次の4つのデータファイルをロードしたいとします。

```Plain
/user/starrocks/data/input/date=2020-05-20/data
1,354
/user/starrocks/data/input/date=2020-05-21/data
2,465
/user/starrocks/data/input/date=2020-05-22/data
1,576
/user/starrocks/data/input/date=2020-05-23/data
2,687
```

これらの4つのデータファイルは、HDFSクラスターの`/user/starrocks/data/input/`パスに保存されています。これらのデータファイルは、各々がパーティションフィールド`date`でパーティション化されており、2つの列で構成されており、順にイベントタイプとユーザーIDを表しています。

### HDFSクラスターからデータをロード

次のステートメントを実行して、`table1`にファイルパスの`/user/starrocks/data/input/`から`date`パーティションフィールドの値を抽出し、ワイルドカード(*)を使用してファイルパスのすべてのデータファイルを`table1`にロードする[Broker Load](../loading/hdfs_load.md)ジョブを作成します。

```SQL
LOAD LABEL test_db.label4
(
    DATA INFILE("hdfs://<fe_host>:<fe_http_port>/user/starrocks/data/input/date=*/*")
    INTO TABLE `table1`
    FORMAT AS "csv"
    COLUMNS TERMINATED BY ","
    (event_type, user_id)
    COLUMNS FROM PATH AS (date)
    SET(event_date = date)
)
WITH BROKER "broker1";
```

> **注記**
>
> 上記の例では、指定されたファイルパスの`date`パーティションフィールドは、`table1`の`event_date`列と同等です。したがって、`date`パーティションフィールドを`event_date`列にマップするためにSET句を使用する必要があります。指定したファイルパスのパーティションフィールドがStarRocksテーブルの列と同じ名前を持つ場合は、マッピングを作成するためにSET句を使用する必要はありません。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### データをクエリ

HDFSクラスターからのデータのロードが完了した後、`table1`のデータをクエリしてロードが成功したことを確認してください。

```SQL
MySQL [test_db]> SELECT * FROM table1;
+------------+------------+---------+
| event_date | event_type | user_id |
+------------+------------+---------+
| 2020-05-22 |          1 |     576 |
| 2020-05-20 |          1 |     354 |
| 2020-05-21 |          2 |     465 |
| 2020-05-23 |          2 |     687 |
+------------+------------+---------+
4 行が選択されました (0.01 秒)
```