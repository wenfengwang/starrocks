---
displayed_sidebar: "Japanese"
---

# データの読み込み時にデータを変換する

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksでは、データの読み込み時にデータの変換がサポートされています。

この機能は、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)をサポートしていますが、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)はサポートされていません。

<InsertPrivNote />

このトピックでは、CSVデータを使用して、データの抽出と変換方法を説明します。選択した読み込み方法によってサポートされるデータファイルの形式は異なります。

> **注意**
>
> CSVデータの場合、テキスト区切り記号として、50バイトを超えないUTF-8文字列（カンマ（,）、タブ、またはパイプ（|）など）を使用できます。

## シナリオ

StarRocksテーブルにデータファイルを読み込む際、データファイルのデータが完全にStarRocksテーブルのデータにマッピングされない場合があります。この場合、データを抽出または変換する必要はありません。StarRocksは、データの読み込み中にデータを抽出および変換することができます。

- 読み込む必要のない列をスキップする。

  読み込む必要のない列をスキップすることができます。また、データファイルの列がStarRocksテーブルの列と異なる順序である場合、データファイルとStarRocksテーブルの間に列マッピングを作成することができます。

- 読み込みたくない行をフィルタリングする。

  読み込みたくない行をフィルタ条件で指定することができます。StarRocksは、フィルタ条件に一致しない行をフィルタリングします。

- 元の列から新しい列を生成する。

  生成列は、データファイルの元の列から計算される特殊な列です。生成列をStarRocksテーブルの列にマッピングすることができます。

- ファイルパスからパーティションフィールドの値を抽出する。

  データファイルがApache Hive™から生成された場合、ファイルパスからパーティションフィールドの値を抽出することができます。

## 前提条件

### Broker Load

[HDFSからデータを読み込む](../loading/hdfs_load.md)または[クラウドストレージからデータを読み込む](../loading/cloud_storage_load.md)の「背景情報」セクションを参照してください。

### Routine load

[Routine Load](./RoutineLoad.md)を選択する場合、Apache Kafka®クラスタにトピックが作成されていることを確認してください。`topic1`と`topic2`の2つのトピックを作成したと仮定します。

## データの例

1. ローカルファイルシステムにデータファイルを作成します。

   a. `file1.csv`という名前のデータファイルを作成します。このファイルは、ユーザーID、ユーザーの性別、イベント日付、およびイベントタイプを順に表す4つの列で構成されています。

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

   > **注意**
   >
   > v2.5.7以降、StarRocksはテーブルの作成またはパーティションの追加時にバケットの数（BUCKETS）を自動的に設定できるようになりました。バケットの数を手動で設定する必要はありません。詳細については、「[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)」を参照してください。

   a. `table1`という名前のテーブルを作成します。このテーブルは、`event_date`、`event_type`、および`user_id`の3つの列で構成されています。

      ```SQL
      MySQL [test_db]> CREATE TABLE table1
      (
          `event_date` DATE COMMENT "イベント日付",
          `event_type` TINYINT COMMENT "イベントタイプ",
          `user_id` BIGINT COMMENT "ユーザーID"
      )
      DISTRIBUTED BY HASH(user_id);
      ```

   b. `table2`という名前のテーブルを作成します。このテーブルは、`date`、`year`、`month`、および`day`の4つの列で構成されています。

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

3. `file1.csv`と`file2.csv`をHDFSクラスタの`/user/starrocks/data/input/`パスにアップロードし、`file1.csv`のデータをKafkaクラスタの`topic1`に、`file2.csv`のデータをKafkaクラスタの`topic2`に公開します。

## 読み込む必要のない列をスキップする

StarRocksテーブルに読み込むデータファイルには、StarRocksテーブルの列にマッピングできない列が含まれている場合があります。この場合、StarRocksは、データファイルからStarRocksテーブルの列にマッピングできる列のみを読み込むことができます。

この機能は、次のデータソースからのデータの読み込みをサポートしています。

- ローカルファイルシステム

- HDFSおよびクラウドストレージ
  
  > **注意**
  >
  > このセクションでは、HDFSを例として使用しています。

- Kafka

ほとんどの場合、CSVファイルの列には名前が付いていません。一部のCSVファイルでは、最初の行が列名で構成されていますが、StarRocksは最初の行の内容を列名ではなく一般のデータとして処理します。したがって、CSVファイルを読み込む場合は、ジョブ作成ステートメントまたはコマンドで一時的にCSVファイルの列を**順番に**名前付ける必要があります。これらの一時的に名前付けられた列は、StarRocksテーブルの列と**名前によって**マッピングされます。データファイルの列に関する次のポイントに注意してください。

- StarRocksテーブルの列にマッピングでき、一時的にStarRocksテーブルの列の名前を使用して一時的に名前付けられた列のデータは直接読み込まれます。

- StarRocksテーブルの列にマッピングできない列は無視され、これらの列のデータは読み込まれません。

- 一部の列がStarRocksテーブルの列にマッピングできるが、ジョブ作成ステートメントまたはコマンドで一時的に名前が付けられていない場合、読み込みジョブはエラーを報告します。

このセクションでは、`file1.csv`と`table1`を例に説明します。`file1.csv`の4つの列は、`user_id`、`user_gender`、`event_date`、および`event_type`として順番に一時的に名前が付けられます。`file1.csv`の一時的に名前が付けられた列のうち、`user_id`、`event_date`、および`event_type`は`table1`の特定の列にマッピングできますが、`user_gender`は`table1`のいずれの列にもマッピングできません。したがって、`user_id`、`event_date`、および`event_type`は`table1`に読み込まれますが、`user_gender`は読み込まれません。

### データの読み込み

#### ローカルファイルシステムからデータを読み込む

`file1.csv`がローカルファイルシステムに保存されている場合、次のコマンドを実行して[Stream Load](../loading/StreamLoad.md)ジョブを作成します。

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: user_id, user_gender, event_date, event_type" \
    -T file1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

> **注意**
>
> Stream Loadを選択した場合、`columns`パラメータを使用してデータファイルの列を一時的に名前付けて、データファイルとStarRocksテーブルの間に列マッピングを作成する必要があります。

詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

#### HDFSクラスタからデータを読み込む

`file1.csv`がHDFSクラスタに保存されている場合、次のステートメントを実行して[Broker Load](../loading/hdfs_load.md)ジョブを作成します。

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

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Kafkaクラスタからデータを読み込む

`file1.csv`のデータがKafkaクラスタの`topic1`に公開されている場合、次のステートメントを実行して[Routine Load](../loading/RoutineLoad.md)ジョブを作成します。

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

詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### データのクエリ

ローカルファイルシステム、HDFSクラスタ、またはKafkaクラスタからのデータの読み込みが完了したら、`table1`のデータをクエリして読み込みが成功したことを確認します。

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

## 読み込みたくない行をフィルタリングする

StarRocksテーブルにデータファイルを読み込む際、特定の行を読み込みたくない場合があります。この場合、WHERE句を使用して読み込みたい行を指定することができます。StarRocksは、WHERE句で指定されたフィルタ条件に一致しないすべての行をフィルタリングします。

この機能は、次のデータソースからのデータの読み込みをサポートしています。

- ローカルファイルシステム

- HDFSおよびクラウドストレージ
  > **注意**
  >
  > このセクションでは、HDFSを例として使用しています。

- Kafka

このセクションでは、`file1.csv`と`table1`を例に説明します。`file1.csv`から`table1`に読み込む際に、イベントタイプが`1`の行のみを読み込みたい場合、WHERE句を使用してフィルタ条件`event_type = 1`を指定することができます。

### データの読み込み

#### ローカルファイルシステムからデータを読み込む

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

#### HDFSクラスタからデータを読み込む

`file1.csv`がHDFSクラスタに保存されている場合、次のステートメントを実行して[Broker Load](../loading/hdfs_load.md)ジョブを作成します。

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

#### Kafkaクラスタからデータを読み込む

`file1.csv`のデータがKafkaクラスタの`topic1`に公開されている場合、次のステートメントを実行して[Routine Load](../loading/RoutineLoad.md)ジョブを作成します。

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

ローカルファイルシステム、HDFSクラスタ、またはKafkaクラスタからのデータの読み込みが完了したら、`table1`のデータをクエリして読み込みが成功したことを確認します。

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

StarRocksテーブルにデータファイルを読み込む際、データファイルの一部のデータは、データをStarRocksテーブルに読み込む前に変換する必要がある場合があります。この場合、ジョブ作成コマンドまたはステートメントで関数または式を使用してデータ変換を実装することができます。

この機能は、次のデータソースからのデータの読み込みをサポートしています。

- ローカルファイルシステム

- HDFSおよびクラウドストレージ
  > **注意**
  >
  > このセクションでは、HDFSを例として使用しています。

- Kafka

このセクションでは、`file2.csv`と`table2`を例に説明します。`file2.csv`は、日付を表す1つの列のみで構成されています。[year](../sql-reference/sql-functions/date-time-functions/year.md)、[month](../sql-reference/sql-functions/date-time-functions/month.md)、および[day](../sql-reference/sql-functions/date-time-functions/day.md)関数を使用して、`file2.csv`の各日付から年、月、および日を抽出し、抽出したデータを`table2`の`year`、`month`、および`day`の列に読み込むことができます。

### データの読み込み

#### ローカルファイルシステムからデータを読み込む

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
> - `columns`パラメータでは、まずデータファイルの**すべての列**に一時的に名前を付け、次に元の列から生成する新しい列に一時的に名前を付ける必要があります。前述の例では、`file2.csv`の唯一の列は`date`として一時的に名前が付けられ、`year=year(date)`、`month=month(date)`、および`day=day(date)`関数が呼び出されて、`year`、`month`、および`day`として一時的に名前が付けられた3つの新しい列が生成されます。
>
> - Stream Loadは`column_name = function(column_name)`をサポートしていませんが、`column_name = function(column_name)`をサポートしています。

詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

#### HDFSクラスタからデータを読み込む

`file2.csv`がHDFSクラスタに保存されている場合、次のステートメントを実行して[Broker Load](../loading/hdfs_load.md)ジョブを作成します。

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
> `column_list`パラメータを使用して、まずデータファイルの**すべての列**に一時的に名前を付け、SET句を使用して元の列から生成する新しい列に一時的に名前を付ける必要があります。前述の例では、`file2.csv`の唯一の列は`date`として`column_list`パラメータに一時的に名前が付けられ、SET句で`year=year(date)`、`month=month(date)`、および`day=day(date)`関数が呼び出されて、`year`、`month`、および`day`として一時的に名前が付けられた3つの新しい列が生成されます。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Kafkaクラスタからデータを読み込む

`file2.csv`のデータがKafkaクラスタの`topic2`に公開されている場合、次のステートメントを実行して[Routine Load](../loading/RoutineLoad.md)ジョブを作成します。

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
> `COLUMNS`パラメータを使用して、まずデータファイルの**すべての列**に一時的に名前を付け、次に元の列から生成する新しい列に一時的に名前を付ける必要があります。前述の例では、`file2.csv`の唯一の列は`date`として一時的に名前が付けられ、`year=year(date)`、`month=month(date)`、および`day=day(date)`関数が呼び出されて、`year`、`month`、および`day`として一時的に名前が付けられた3つの新しい列が生成されます。

詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### データのクエリ

ローカルファイルシステム、HDFSクラスタ、またはKafkaクラスタからのデータの読み込みが完了したら、`table2`のデータをクエリして読み込みが成功したことを確認します。

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

## ファイルパスからパーティションフィールドの値を抽出する

指定したファイルパスにパーティションフィールドが含まれている場合、`COLUMNS FROM PATH AS`パラメータを使用してファイルパスから抽出するパーティションフィールドを指定することができます。ファイルパスのパーティションフィールドは、データファイルの列と同等です。`COLUMNS FROM PATH AS`パラメータは、データの読み込みがHDFSクラスタからの場合にのみサポートされます。

たとえば、次のようなHiveから生成された4つのデータファイルを読み込む場合を考えます。

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

これらの4つのデータファイルは、HDFSクラスタの`/user/starrocks/data/input/`パスに保存されています。これらのデータファイルはすべて`date`パーティションフィールドでパーティション分割されており、イベントタイプとユーザーIDを表す2つの列で構成されています。

### HDFSクラスタからデータを読み込む

次のステートメントを実行して[Broker Load](../loading/hdfs_load.md)ジョブを作成します。このジョブでは、`/user/starrocks/data/input/`ファイルパスから`date`パーティションフィールドの値を抽出し、ワイルドカード（*）を使用してファイルパス内のすべてのデータファイルを`table1`に読み込むことができます。

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
> 前述の例では、指定したファイルパスの`date`パーティションフィールドは`table1`の`event_date`列と同等です。したがって、`date`パーティションフィールドを`event_date`列にマッピングするためにSET句を使用する必要があります。指定したファイルパスのパーティションフィールドがStarRocksテーブルの列と同じ名前を持つ場合、マッピングを作成するためにSET句を使用する必要はありません。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### データのクエリ

HDFSクラスタからのデータの読み込みが完了したら、`table1`のデータをクエリして読み込みが成功したことを確認します。

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
