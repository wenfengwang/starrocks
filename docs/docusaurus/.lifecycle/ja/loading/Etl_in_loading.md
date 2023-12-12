---
displayed_sidebar: "Japanese"
---

# ロード時のデータ変換

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksはロード時のデータ変換をサポートします。

この機能は[ストリームロード](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[ブローカーロード](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および[ルーチンロード](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)をサポートしていますが、[スパークロード](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)をサポートしていません。

<InsertPrivNote />

このトピックでは、CSVデータを使用して、ロード時にデータを抽出および変換する方法を説明します。選択したロード方法によってサポートされるデータファイルの形式は異なります。

> **注記**
>
> CSVデータの場合、テキスト区切り記号として、50バイトを超えないUTF-8文字列（例: カンマ（,）、タブ、またはパイプ（|））を使用できます。

## シナリオ

StarRocksテーブルにデータファイルをロードする際、データファイルのデータが完全にStarRocksテーブルのデータにマッピングされない場合、StarRocksテーブルにロードする前にデータを抽出または変換する必要はありません。StarRocksはロード中にデータを抽出および変換することができます。

- ロードする必要のない列をスキップ
  
  ロードする必要のない列をスキップできます。また、データファイルの列がStarRocksテーブルの列と異なる順序にある場合、データファイルとStarRocksテーブルの間で列マッピングを作成できます。

- ロードする必要のない行をフィルタリング
  
  ロードする必要のない行をフィルタ条件を指定してフィルタリングできます。

- オリジナル列から新しい列を生成
  
  生成列はオリジナル列から計算される特別な列です。生成列をStarRocksテーブルの列にマッピングできます。

- ファイルパスからパーティションフィールドの値を抽出
  
  データファイルがApache Hive™から生成された場合、ファイルパスからパーティションフィールドの値を抽出できます。

## 前提条件

### ブローカーロード

[HDFSからデータをロード](../loading/hdfs_load.md)または[クラウドストレージからデータをロード](../loading/cloud_storage_load.md)の「背景情報」セクションを参照してください。

### ルーチンロード

[ルーチンロード](./RoutineLoad.md)を選択した場合、Apache Kafka®クラスターにトピックが作成されていることを確認してください。`topic1`および`topic2`の2つのトピックが作成されていると仮定します。

## データ例

1. ローカルファイルシステムでデータファイルを作成します。

   a. `file1.csv`という名前のデータファイルを作成します。ファイルは、ユーザーID、ユーザーの性別、イベント日付、およびイベントタイプを順に表す4つの列から構成されています。

      ```Plain
      354,female,2020-05-20,1
      465,male,2020-05-21,2
      576,female,2020-05-22,1
      687,male,2020-05-23,2
      ```

   b. `file2.csv`という名前のデータファイルを作成します。ファイルは日付を表す1つの列のみから構成されています。

      ```Plain
      2020-05-20
      2020-05-21
      2020-05-22
      2020-05-23
      ```

2. StarRocksデータベース`test_db`でテーブルを作成します。

   > **注記**
   >
   > v2.5.7から、テーブルを作成したりパーティションを追加する際に、StarRocksはバケツの数（BUCKETS）を自動的に設定できます。バケツの数を手動で設定する必要はもはやありません。詳細は、[バケツの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   a. `table1`という名前のテーブルを作成します。テーブルは、`event_date`、`event_type`、および`user_id`の3つの列から構成されています。

      ```SQL
      MySQL [test_db]> CREATE TABLE table1
      (
          `event_date` DATE COMMENT "イベント日",
          `event_type` TINYINT COMMENT "イベントタイプ",
          `user_id` BIGINT COMMENT "ユーザーID"
      )
      DISTRIBUTED BY HASH(user_id);
      ```

   b. `table2`という名前のテーブルを作成します。テーブルは、`date`, `year`, `month`, および`day`の4つの列から構成されています。

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

## ロードする必要のない列をスキップ

StarRocksテーブルにロードしたいデータファイルには、StarRocksテーブルの列にマッピングできない列が含まれる場合があります。この場合、StarRocksはデータファイルの列からStarRocksテーブルの列にマッピングできる列のみをロードすることをサポートします。

この機能は、次のデータソースからのデータのロードをサポートしています。

- ローカルファイルシステム

- HDFSおよびクラウドストレージ
  
  > **注記**
  
  このセクションではHDFSを例としています。

- Kafka

ほとんどの場合、CSVファイルの列には名前が付いていません。一部のCSVファイルでは、最初の行が列名で構成されていますが、StarRocksは最初の行の内容を列名ではなく共通データとして処理します。したがって、CSVファイルをロードする際には、CSVファイルの列を一時的にジョブ作成文またはコマンドで**順序通り**に名前を付ける必要があります。これらの一時的に名前を付けた列は、StarRocksテーブルの列と**名前によって**マッピングされます。データファイルの列に関する次の点に注意してください。

- StarRocksテーブルの列にマッピングでき、列を一時的な名前でStarRocksテーブルの列と同じ名前でマッピングされている列のデータは直接ロードされます。

- StarRocksテーブルの列にマッピングできない列は無視され、これらの列のデータはロードされません。

- StarRocksテーブルの列にマッピングできる列でも、ジョブ作成文またはコマンドで一時的な名前を付けていない列の場合、ロードジョブはエラーを報告します。

このセクションでは、`file1.csv`と`table1`を例として使用します。`file1.csv`の4つの列は、順に`user_id`、`user_gender`、`event_date`、および`event_type`として一時的に名前が付けられています。`file1.csv`の一時的に名前が付けられた列のうち、`user_id`、`event_date`、および`event_type`は`table1`の特定の列にマッピングされますが、`user_gender`は`table1`のいずれの列にもマッピングされません。したがって、`user_id`、`event_date`、および`event_type`は`table1`にロードされますが、`user_gender`はロードされません。

### データのロード

#### ローカルファイルシステムからデータをロード

`file1.csv`がローカルファイルシステムに保存されている場合は、次のコマンドを実行して[ストリームロード](../loading/StreamLoad.md)ジョブを作成します:

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
> ストリームロードを選択した場合、`columns`パラメータを使用してデータファイルの列に一時的な名前を付けてデータファイルとStarRocksテーブルの間に列マッピングを作成する必要があります。

詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

#### HDFSクラスターからデータをロード

`file1.csv`がHDFSクラスターに保存されている場合は、次のステートメントを実行して[ブローカーロード](../loading/hdfs_load.md)ジョブを作成します:

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
> ブローカーロードを選択した場合、`column_list`パラメータを使用してデータファイルの列に一時的な名前を付けてデータファイルとStarRocksテーブルの間に列マッピングを作成する必要があります。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Kafkaクラスターからデータをロード
`file1.csv`のデータがあなたのKafkaクラスターの`topic1`に公開されている場合、次のステートメントを実行して[Routine Load](../loading/RoutineLoad.md)ジョブを作成します。

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
> Routine Loadを選択した場合、`COLUMNS`パラメータを使用して、一時的にデータファイルの列に名前を付けて、データファイルとStarRocksテーブルの間に列マッピングを作成する必要があります。

詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### データのクエリ

ローカルファイルシステム、HDFSクラスター、またはKafkaクラスターからデータをロードした後、`table1`のデータをクエリして、ロードが成功したことを確認します。

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
4 rows in set (0.01 sec)
```

## ロードしない行をフィルタリング

StarRocksテーブルにデータファイルをロードする際、特定の行をロードしたくない場合があります。このような場合、`WHERE`句を使用してロードしたい行を指定できます。StarRocksは`WHERE`句で指定されたフィルタ条件を満たさないすべての行をフィルタリングします。

この機能は、次のデータソースからデータをロードすることをサポートしています。

- ローカルファイルシステム

- HDFSとクラウドストレージ
  > **注意**
  >
  > このセクションではHDFSを例にしています。

- Kafka

このセクションでは`file1.csv`および`table1`を例にしています。`file1.csv`から`table1`にロードする際に、イベントタイプが`1`である行のみをロードしたい場合、`WHERE`句を使用して`event_type = 1`のフィルタ条件を指定できます。

### データのロード

#### ローカルファイルシステムからデータをロード

`file1.csv`がローカルファイルシステムに保存されている場合、次のコマンドを実行して[Stream Load](../loading/StreamLoad.md)ジョブを作成します。

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

#### HDFSクラスターからデータをロード

`file1.csv`がHDFSクラスターに保存されている場合、次のステートメントを実行して[Broker Load](../loading/hdfs_load.md)ジョブを作成します。

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

#### Kafkaクラスターからデータをロード

`file1.csv`のデータがあなたのKafkaクラスターの`topic1`に公開されている場合、次のステートメントを実行して[Routine Load](../loading/RoutineLoad.md)ジョブを作成します。

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

ローカルファイルシステム、HDFSクラスター、またはKafkaクラスターからデータをロードした後、`table1`のデータをクエリして、ロードが成功したことを確認します。

```SQL
MySQL [test_db]> SELECT * FROM table1;
+------------+------------+---------+
| event_date | event_type | user_id |
+------------+------------+---------+
| 2020-05-20 |          1 |     354 |
| 2020-05-22 |          1 |     576 |
+------------+------------+---------+
2 rows in set (0.01 sec)
```

## 元の列から新しい列を生成

StarRocksテーブルにデータファイルをロードする際、データファイルのデータの一部には、StarRocksテーブルにロードする前に変換が必要な場合があります。このような場合、ジョブ作成コマンドまたはステートメントで関数や式を使用してデータ変換を実装できます。

この機能は、次のデータソースからデータをロードすることをサポートしています。

- ローカルファイルシステム

- HDFSとクラウドストレージ
  > **注意**
  >
  > このセクションではHDFSを例にしています。

- Kafka

このセクションでは`file2.csv`および`table2`を例にしています。`file2.csv`には、日付を表す1列のみが含まれています。 `file2.csv`から各日付の年、月、日を抽出し、抽出したデータを`table2`の`year`、`month`、および`day`の列にロードするために、[year](../sql-reference/sql-functions/date-time-functions/year.md)、[month](../sql-reference/sql-functions/date-time-functions/month.md)、および[day](../sql-reference/sql-functions/date-time-functions/day.md)関数を使用できます。

### データのロード

#### ローカルファイルシステムからデータをロード

`file2.csv`がローカルファイルシステムに保存されている場合、次のコマンドを実行して[Stream Load](../loading/StreamLoad.md)ジョブを作成します。

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
> - `columns`パラメータでは、まずデータファイルの**すべての列**に一時的に名前を付け、次にデータファイルの元の列から生成する新しい列に一時的に名前を付ける必要があります。前述の例のように、`file2.csv`の唯一の列は一時的に`date`として名前が付けられ、その後`year=year(date)`、`month=month(date)`、`day=day(date)`関数が呼び出されて、一時的に`year`、`month`、`day`と名付けられた3つの新しい列が生成されます。
>
> - Stream Loadは、`column_name = function(column_name)`をサポートしておらず、`column_name = function(column_name)`をサポートしています。

詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

#### HDFSクラスターからデータをロード

`file2.csv`がHDFSクラスターに保存されている場合、次のステートメントを実行して[Broker Load](../loading/hdfs_load.md)ジョブを作成します。

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
まず、`column_list`パラメータを使用してデータファイルの**すべての列**に一時的な名前を付け、その後、SET句を使用してデータファイルの元の列から生成したい新しい列に一時的な名前を付けます。前述の例のように、`file2.csv`の唯一の列は`column_list`パラメータで一時的に`date`という名前が付けられ、その後、SET句で`year=year(date)`、`month=month(date)`、`day=day(date)`関数が呼び出され、一時的に`year`、`month`、`day`という名前が付けられた3つの新しい列が生成されます。

詳細な構文およびパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Kafkaクラスタからデータをロードする

`file2.csv`のデータがKafkaクラスタの`topic2`に発行されている場合は、次のステートメントを実行して[Routine Load](../loading/RoutineLoad.md)ジョブを作成します。

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

> **注意**
>
> `COLUMNS`パラメータでは、まずデータファイルの**すべての列**に一時的な名前を付け、その後、元のデータファイルの列から生成したい新しい列に一時的な名前を付ける必要があります。前述の例のように、`file2.csv`の唯一の列は一時的に`date`という名前が付けられ、その後`year=year(date)`、`month=month(date)`、`day=day(date)`関数が呼び出されて一時的に`year`、`month`、`day`という名前が付けられた3つの新しい列が生成されます。

詳細な構文およびパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### データのクエリ

ローカルファイルシステム、HDFSクラスタ、またはKafkaクラスタからのデータのロードが完了したら、`table2`のデータをクエリして、ロードが成功していることを確認します。

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
4 rows in set (0.01 sec)
```

## ファイルパスからのパーティションフィールド値の抽出

指定したファイルパスにパーティションフィールドが含まれている場合は、`COLUMNS FROM PATH AS`パラメータを使用してファイルパスから抽出するパーティションフィールドを指定できます。ファイルパスのパーティションフィールドは、データファイルの列に相当します。`COLUMNS FROM PATH AS`パラメータは、データをHDFSクラスタからロードする場合にのみサポートされています。

たとえば、Hiveから生成された次の4つのデータファイルをロードしたいとします:

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

これらの4つのデータファイルはHDFSクラスタの`/user/starrocks/data/input/`パスに保存されています。それぞれのデータファイルはパーティションフィールド`date`でパーティション分けされており、2つの列（イベントタイプとユーザーID）で構成されています。

### HDFSクラスタからデータをロードする

以下のステートメントを実行して、指定したファイルパスから`date`パーティションフィールド値を抽出し、ワイルドカード(*)を使用してファイルパスのすべてのデータファイルを`table1`にロードする[Broker Load](../loading/hdfs_load.md)ジョブを作成します。

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

> **注意**
>
> 前述の例のように、指定したファイルパスの`date`パーティションフィールドは`table1`の`event_date`列に相当します。そのため、`date`パーティションフィールドを`event_date`列にマップするためにSET句を使用する必要があります。指定したファイルパスのパーティションフィールドがStarRocksテーブルの列と同じ名前を持つ場合、マッピングを作成するためにSET句を使用する必要はありません。

詳細な構文およびパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### データのクエリ

HDFSクラスタからのデータのロードが完了したら、`table1`のデータをクエリして、ロードが成功していることを確認します。

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
4 rows in set (0.01 sec)
```