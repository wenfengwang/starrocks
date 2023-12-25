---
displayed_sidebar: English
---

# 読み込み時のデータ変換

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、読み込み時のデータ変換をサポートしています。

この機能は、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)をサポートしていますが、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)はサポートしていません。

<InsertPrivNote />

このトピックでは、CSVデータを例に、読み込み時にデータを抽出および変換する方法について説明します。サポートされるデータファイルの形式は、選択した読み込み方法によって異なります。

> **注記**
>
> CSVデータの場合、長さが50バイトを超えないカンマ(,)、タブ、パイプ(|)などのUTF-8文字列をテキスト区切り文字として使用できます。

## シナリオ

StarRocksテーブルにデータファイルをロードする際、データファイルのデータがStarRocksテーブルのデータに完全にマッピングされない場合があります。この状況では、StarRocksテーブルにロードする前にデータを抽出または変換する必要はありません。StarRocksは、ロード中にデータを抽出して変換するのに役立ちます。

- ロードする必要のない列をスキップします。
  
  ロードする必要のない列はスキップできます。また、データファイルの列の順序がStarRocksテーブルの列と異なる場合は、データファイルとStarRocksテーブルの間に列マッピングを作成できます。

- ロードしたくない行をフィルタリングします。
  
  StarRocksは、ロードしたくない行をフィルタリングする条件を指定できます。

- 元の列から新しい列を生成します。
  
  生成された列は、データファイルの元の列から計算される特別な列です。生成された列をStarRocksテーブルの列にマッピングできます。

- ファイルパスからパーティションフィールドの値を抽出します。
  
  データファイルがApache Hive™から生成された場合、ファイルパスからパーティションフィールドの値を抽出できます。

## 前提条件

### Broker Load

[HDFSからのデータ読み込み](../loading/hdfs_load.md)または[クラウドストレージからのデータ読み込み](../loading/cloud_storage_load.md)の「背景情報」セクションを参照してください。

### Routine Load

[Routine Load](./RoutineLoad.md)を選択した場合、Apache Kafka®クラスターにトピックが作成されていることを確認してください。`topic1`と`topic2`という2つのトピックを作成したとします。

## データ例

1. ローカルファイルシステムにデータファイルを作成します。

   a. `file1.csv`という名前のデータファイルを作成します。このファイルは、ユーザーID、ユーザー性別、イベント日付、イベントタイプを順に表す4つの列で構成されています。

      ```Plain
      354,female,2020-05-20,1
      465,male,2020-05-21,2
      576,female,2020-05-22,1
      687,male,2020-05-23,2
      ```

   b. `file2.csv`という名前のデータファイルを作成します。このファイルは、日付を表す1つの列のみで構成されています。

      ```Plain
      2020-05-20
      2020-05-21
      2020-05-22
      2020-05-23
      ```

2. StarRocksデータベース`test_db`にテーブルを作成します。

   > **注記**
   >
   > v2.5.7以降、StarRocksはテーブルを作成する際やパーティションを追加する際にバケット数(BUCKETS)を自動的に設定するようになりました。バケット数を手動で設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   a. `event_date`、`event_type`、`user_id`の3つの列で構成される`table1`という名前のテーブルを作成します。

      ```SQL
      MySQL [test_db]> CREATE TABLE table1
      (
          `event_date` DATE COMMENT "event date",
          `event_type` TINYINT COMMENT "event type",
          `user_id` BIGINT COMMENT "user ID"
      )
      DISTRIBUTED BY HASH(user_id);
      ```

   b. `date`、`year`、`month`、`day`の4つの列で構成される`table2`という名前のテーブルを作成します。

      ```SQL
      MySQL [test_db]> CREATE TABLE table2
      (
          `date` DATE COMMENT "date",
          `year` INT COMMENT "year",
          `month` TINYINT COMMENT "month",
          `day` TINYINT COMMENT "day"
      )
      DISTRIBUTED BY HASH(date);
      ```

3. `file1.csv`と`file2.csv`をHDFSクラスターの`/user/starrocks/data/input/`パスにアップロードし、`file1.csv`のデータをKafkaクラスターの`topic1`に、`file2.csv`のデータを`topic2`に公開します。

## ロードする必要のない列をスキップする

StarRocksテーブルにロードするデータファイルには、StarRocksテーブルのどの列にもマッピングできない列が含まれている場合があります。この状況では、StarRocksはデータファイルからStarRocksテーブルの列にマッピングできる列のみをロードすることをサポートします。

この機能は、以下のデータソースからのデータのロードをサポートしています：

- ローカルファイルシステム

- HDFSおよびクラウドストレージ
  
  > **注記**
  >
  > このセクションでは、例としてHDFSを使用します。

- Kafka

ほとんどの場合、CSVファイルの列には名前がありません。一部のCSVファイルでは、最初の行が列名で構成されていますが、StarRocksは最初の行の内容を列名ではなく通常のデータとして処理します。したがって、CSVファイルをロードする際には、ジョブ作成ステートメントまたはコマンドでCSVファイルの列に**順番に**一時的な名前を付ける必要があります。これらの一時的に名前が付けられた列は、**名前によって**StarRocksテーブルの列にマッピングされます。データファイルの列に関する以下の点に注意してください：

- StarRocksテーブルの列の名前を使用してマッピングでき、一時的に名前が付けられた列のデータは直接ロードされます。

- StarRocksテーブルの列にマッピングできない列は無視され、これらの列のデータはロードされません。

- 一部の列がStarRocksテーブルの列にマッピングできるものの、ジョブ作成ステートメントまたはコマンドで一時的に名前が付けられていない場合、ロードジョブはエラーを報告します。

このセクションでは、`file1.csv`と`table1`を例に使用します。`file1.csv`の4つの列は、`user_id`、`user_gender`、`event_date`、`event_type`という順に一時的に名前が付けられます。`file1.csv`の一時的に名前が付けられた列のうち、`user_id`、`event_date`、`event_type`は`table1`の特定の列にマッピングできますが、`user_gender`はどの列にもマッピングできません。したがって、`user_id`、`event_date`、`event_type`は`table1`にロードされますが、`user_gender`はロードされません。

### データのロード

#### ローカルファイルシステムからのデータのロード


`file1.csv`がローカルファイルシステムに格納されている場合は、次のコマンドを実行して[Stream Load](../loading/StreamLoad.md)ジョブを作成します：

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
> Stream Loadを選択する場合、`columns`パラメータを使用してデータファイルの列に一時的な名前を付け、データファイルとStarRocksテーブル間の列マッピングを作成する必要があります。

詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

#### HDFSクラスタからデータをロードする

`file1.csv`がHDFSクラスタに保存されている場合、次のステートメントを実行して[Broker Load](../loading/hdfs_load.md)ジョブを作成します：

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
> Broker Loadを選択する場合、`column_list`パラメータを使用してデータファイルの列に一時的な名前を付け、データファイルとStarRocksテーブル間のカラムマッピングを作成する必要があります。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Kafkaクラスタからデータをロードする

`file1.csv`のデータがKafkaクラスタの`topic1`にパブリッシュされている場合、次のステートメントを実行して[Routine Load](../loading/RoutineLoad.md)ジョブを作成します：

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

> **注記**
>
> Routine Loadを選択する場合、`COLUMNS`パラメータを使用してデータファイルの列に一時的な名前を付け、データファイルとStarRocksテーブル間の列マッピングを作成する必要があります。

詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### データをクエリする

ローカルファイルシステム、HDFSクラスタ、またはKafkaクラスタからのデータロードが完了したら、`table1`のデータをクエリして、ロードが成功したことを確認します：

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

## ロードしたくない行をフィルタリングする

StarRocksテーブルにデータファイルをロードする際、特定の行をロードしたくない場合があります。このような場合、WHERE句を使用してロードしたい行を指定できます。StarRocksは、WHERE句で指定されたフィルタ条件を満たさないすべての行をフィルタリングします。

この機能は、以下のデータソースからのデータロードをサポートしています：

- ローカルファイルシステム

- HDFSおよびクラウドストレージ
  > **注記**
  >
  > このセクションでは、例としてHDFSを使用しています。

- Kafka

このセクションでは、`file1.csv`と`table1`を例に使用します。`file1.csv`からイベントタイプが`1`の行のみを`table1`にロードしたい場合、WHERE句を使用してフィルタ条件`event_type = 1`を指定できます。

### データをロードする

#### ローカルファイルシステムからデータをロードする

`file1.csv`がローカルファイルシステムに格納されている場合、次のコマンドを実行して[Stream Load](../loading/StreamLoad.md)ジョブを作成します：

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

#### HDFSクラスタからデータをロードする

`file1.csv`がHDFSクラスタに保存されている場合、次のステートメントを実行して[Broker Load](../loading/hdfs_load.md)ジョブを作成します：

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

#### Kafkaクラスタからデータをロードする

`file1.csv`のデータがKafkaクラスタの`topic1`にパブリッシュされている場合、次のステートメントを実行して[Routine Load](../loading/RoutineLoad.md)ジョブを作成します：

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

### データをクエリする

ローカルファイルシステム、HDFSクラスタ、またはKafkaクラスタからのデータロードが完了したら、`table1`のデータをクエリして、ロードが成功したことを確認します：

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

## 元の列から新しい列を生成する

データファイルをStarRocksテーブルにロードする際、データファイルの一部のデータはStarRocksテーブルにロードする前に変換が必要な場合があります。このような場合、ジョブ作成コマンドまたはステートメントで関数や式を使用してデータ変換を実行できます。

この機能は、以下のデータソースからのデータロードをサポートしています：

- ローカルファイルシステム

- HDFSおよびクラウドストレージ
  > **注記**
  >
  > このセクションでは、例としてHDFSを使用しています。

- Kafka

このセクションでは、例として `file2.csv` と `table2` を使用します。`file2.csv` は日付を表す1つの列のみで構成されています。[year](../sql-reference/sql-functions/date-time-functions/year.md)、[month](../sql-reference/sql-functions/date-time-functions/month.md)、および[day](../sql-reference/sql-functions/date-time-functions/day.md) 関数を使用して、`file2.csv` の各日付から年、月、日を抽出し、抽出したデータを `table2` の `year`、`month`、および `day` 列にロードできます。

### データの読み込み

#### ローカルファイルシステムからのデータの読み込み

`file2.csv` がローカルファイルシステムに格納されている場合、次のコマンドを実行して [Stream Load](../loading/StreamLoad.md) ジョブを作成します：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns:date,year=year(date),month=month(date),day=day(date)" \
    -T file2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

> **注記**
>
> - `columns` パラメータでは、最初にデータファイルの**すべての列**に一時的な名前を付け、次にデータファイルの元の列から生成する新しい列に一時的な名前を付ける必要があります。前述の例では、`file2.csv` の唯一の列は一時的に `date` と名付けられ、その後 `year=year(date)`、`month=month(date)`、`day=day(date)` 関数が呼び出されて、`year`、`month`、`day` という3つの新しい列が生成されます。
>
> - Stream Load は `column_name = function(column_name)` 形式をサポートしていませんが、`column_name = function(column_name)` 形式はサポートしています。

詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) を参照してください。

#### HDFSクラスタからのデータの読み込み

`file2.csv` がHDFSクラスタに保存されている場合、次のステートメントを実行して [Broker Load](../loading/hdfs_load.md) ジョブを作成します：

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

> **注記**
>
> `column_list` パラメータを使用して最初にデータファイルの**すべての列**に一時的な名前を付け、次にSET句を使用してデータファイルの元の列から生成する新しい列に一時的な名前を付ける必要があります。前述の例では、`file2.csv` の唯一の列は `column_list` パラメータで一時的に `date` と名付けられ、その後SET句で `year=year(date)`、`month=month(date)`、`day=day(date)` 関数が呼び出されて、`year`、`month`、`day` という3つの新しい列が生成されます。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

#### Kafkaクラスタからのデータの読み込み

`file2.csv` のデータがKafkaクラスタの `topic2` にパブリッシュされている場合、次のステートメントを実行して [Routine Load](../loading/RoutineLoad.md) ジョブを作成します：

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
> `COLUMNS` パラメータでは、最初にデータファイルの**すべての列**に一時的な名前を付け、次にデータファイルの元の列から生成する新しい列に一時的な名前を付ける必要があります。前述の例では、`file2.csv` の唯一の列は一時的に `date` と名付けられ、その後 `year=year(date)`、`month=month(date)`、`day=day(date)` 関数が呼び出されて、`year`、`month`、`day` という3つの新しい列が生成されます。

詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

### データのクエリ

ローカルファイルシステム、HDFSクラスタ、またはKafkaクラスタからのデータの読み込みが完了したら、`table2` のデータをクエリして、読み込みが成功したことを確認します：

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

## ファイルパスからパーティションフィールド値を抽出

指定したファイルパスにパーティションフィールドが含まれている場合、`COLUMNS FROM PATH AS` パラメータを使用してファイルパスから抽出したいパーティションフィールドを指定できます。ファイルパスのパーティションフィールドはデータファイルの列に相当します。`COLUMNS FROM PATH AS` パラメータは、HDFSクラスタからデータをロードする場合にのみサポートされます。

例えば、Hiveから生成された以下の4つのデータファイルをロードしたいとします：

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

これら4つのデータファイルはHDFSクラスタの `/user/starrocks/data/input/` パスに保存されており、それぞれがパーティションフィールド `date` でパーティション分けされており、イベントタイプとユーザーIDを表す2つの列で構成されています。

### HDFSクラスタからのデータの読み込み

次のステートメントを実行して [Broker Load](../loading/hdfs_load.md) ジョブを作成し、ファイルパス `/user/starrocks/data/input/` から `date` パーティションフィールド値を抽出し、ワイルドカード (*) を使用してファイルパス内のすべてのデータファイルを `table1` にロードするように指定します：

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
> 上記の例では、指定されたファイルパスのパーティションフィールド `date` は `table1` の `event_date` 列に相当します。したがって、`date` パーティションフィールドを `event_date` 列にマッピングするためにSET句を使用する必要があります。指定したファイルパスのパーティションフィールドがStarRocksテーブルの列と同じ名前である場合、マッピングを作成するためにSET句を使用する必要はありません。

構文およびパラメータの詳細な説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### データクエリ

HDFSクラスターからのデータロードが完了したら、`table1`のデータをクエリして、ロードが成功したことを確認します。

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

