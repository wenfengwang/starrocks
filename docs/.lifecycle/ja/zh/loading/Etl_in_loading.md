---
displayed_sidebar: Chinese
---

# データインポート中のデータ変換の実現

StarRocksは、データインポート中にデータ変換を実現することをサポートしています。

現在サポートされているインポート方法には、[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および [Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)があります。[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)によるインポート方法は現在サポートされていません。

> **注意**
>
> インポート操作には対象テーブルのINSERT権限が必要です。ユーザーアカウントにINSERT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を参照してユーザーに権限を付与してください。

この記事では、CSV形式のデータファイルを例に、データインポート中にデータ変換を実現する方法について説明します。どのデータファイル形式の変換がサポートされているかは、選択したインポート方法によって異なります。

> **説明**
>
> CSV形式のデータについては、StarRocksは最大50バイトのUTF-8エンコーディング文字列を列区切り文字として設定することをサポートしており、一般的なカンマ(,)、タブ、パイプ(|)を含みます。

## 適用シナリオ

StarRocksテーブルにデータをインポートする際、時にはStarRocksテーブルの内容とソースデータファイルの内容が完全に一致しないことがあります。以下のいくつかの状況では、外部のETL作業を行う必要なく、StarRocksがインポートプロセス中にデータの抽出と変換を完了することを支援できます：

- インポート不要な列をスキップする。
  
  一方で、この機能によりインポート不要な列をスキップできます。他方で、StarRocksテーブルとソースデータファイルの列の順序が一致しない場合、この機能を使用して両者間の列マッピング関係を確立できます。

- インポート不要な行をフィルタリングする。
  
  インポート時にフィルタ条件を指定して、インポート不要な行をスキップし、必要な行のみをインポートできます。

- 派生列を生成する。
  
  派生列とは、ソースデータファイル内の列に対する計算後に生成される新しい列のことです。この機能により、計算後に生成された新しい列をStarRocksテーブルに落とすことができます。

- ファイルパスからパーティションフィールドの内容を取得する。
  
  Apache Hive™のパーティションパス命名方式をサポートし、StarRocksがファイルパスからパーティション列の内容を取得できるようにします。

## 前提条件

### Broker Load

[从 HDFS 导入](../loading/hdfs_load.md)または[从云存储导入](../loading/cloud_storage_load.md)の「背景情報」セクションを参照してください。

### Routine Load

[Routine Load](./RoutineLoad.md)を使用してデータをインポートする場合、Apache Kafka®クラスターにTopicが作成されていることを確認する必要があります。この記事では、`topic1`と`topic2`の2つのTopicがデプロイされていると仮定しています。

## データサンプル

1. ローカルファイルシステムでデータファイルを作成します。

   a. `file1.csv`という名前のデータファイルを作成し、ユーザーID、ユーザー性別、イベント日付、イベントタイプを表す4列を含むファイルです。以下のようになります：

   ```Plain
   354,female,2020-05-20,1
   465,male,2020-05-21,2
   576,female,2020-05-22,1
   687,male,2020-05-23,2
   ```

   b. `file2.csv`という名前のデータファイルを作成し、日付を表すタイムスタンプ形式の列のみを含むファイルです。以下のようになります：

   ```Plain
   2020-05-20
   2020-05-21
   2020-05-22
   2020-05-23
   ```

2. `test_db`データベースでStarRocksテーブルを作成します。

   > **説明**
   >
   > バージョン2.5.7以降、StarRocksはテーブル作成とパーティション追加時に自動的にバケット数（BUCKETS）を設定することをサポートしており、手動でバケット数を設定する必要はありません。詳細は[确定分桶数量](../table_design/Data_distribution.md#确定分桶数量)を参照してください。

   a. `user_id`、`event_date`、`event_type`の3列を含む`table1`という名前のテーブルを作成します。以下のようになります：

      ```SQL
      CREATE TABLE table1
      (
          `user_id` BIGINT COMMENT "ユーザーID",
          `event_date` DATE COMMENT "イベント日付",
          `event_type` TINYINT COMMENT "イベントタイプ"
      )
      DISTRIBUTED BY HASH(user_id);
      ```

   b. `date`、`year`、`month`、`day`の4列を含む`table2`という名前のテーブルを作成します。以下のようになります：

      ```SQL
      CREATE TABLE table2
      (
          `date` DATE COMMENT "日付",
          `year` INT COMMENT "年",
          `month` TINYINT COMMENT "月",
          `day` TINYINT COMMENT "日"
      )
      DISTRIBUTED BY HASH(date);
      ```

3. `file1.csv`と`file2.csv`のファイルをHDFSクラスターの`/user/starrocks/data/input/`パスにアップロードし、`file1.csv`と`file2.csv`のファイル内のデータをそれぞれApache Kafka®クラスターの`topic1`と`topic2`にアップロードします。

## インポート不要な列をスキップする

ソースデータファイルには、StarRocksテーブルに存在しない列が含まれている可能性があります。StarRocksは、StarRocksテーブルに存在する列のみをインポートし、不要な列を無視することをサポートしています。

この機能は以下のデータソースをサポートしています：

- ローカルファイルシステム

- HDFSおよび外部クラウドストレージシステム
  > **説明**
  >
  > ここではHDFSを例に説明します。

- Kafka

通常、CSVファイル内のソースデータファイルの列には名前が付けられていません。一部のCSVファイルでは、最初の行に列名が記載されていますが、実際にはStarRocksは認識しておらず、通常のデータとして処理されます。したがって、CSV形式のデータをインポートする際には、インポートコマンドまたはステートメント内でソースデータファイルの列に**順番に**一時的な名前を付ける必要があります。これらの一時的な名前が付けられた列は、StarRocksテーブルの列と**名前によって**対応付けられます。ここで以下の点に注意する必要があります：

- ソースデータファイルとStarRocksテーブルの両方に存在し、名前が同じ列は、そのデータが直接インポートされます。

- ソースデータファイルに存在するがStarRocksテーブルには存在しない列は、インポートプロセス中に無視されます。

- StarRocksテーブルに存在するが宣言されていない列がある場合、エラーが発生します。

このセクションでは、`file1.csv`ファイルと`table1`テーブルを例に説明します。`file1.csv`ファイル内の4列を順に`user_id`、`user_gender`、`event_date`、`event_type`と一時的に名付けると仮定します。その中で、`file1.csv`ファイル内の`user_id`、`event_date`、`event_type`という名前の列は`table1`テーブルで対応する列を見つけることができるため、これらの列に対応するデータはすべて`table1`テーブルにインポートされます。一方、`file1.csv`ファイル内の`user_gender`という名前の列は`table1`テーブルには存在しないため、インポート時に無視されます。

### データをインポートする

#### ローカルファイルシステムからインポートする

`file1.csv`ファイルがローカルファイルシステムに保存されている場合、以下のステートメントを使用して[Stream Load](../loading/StreamLoad.md)インポートジョブを作成し、データをインポートできます：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: user_id, user_gender, event_date, event_type" \
    -T file1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

> **説明**
>
> `columns`パラメータは、データファイル内の列に一時的な名前を付けて、StarRocksテーブルの列にマッピングするために使用されます。

詳細な構文とパラメータの紹介については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

#### HDFSからインポートする

`file1.csv`ファイルがHDFSに保存されている場合、以下のステートメントを使用して[Broker Load](../loading/hdfs_load.md)インポートジョブを作成し、データをインポートできます：

```SQL
LOAD LABEL test_db.label1
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/file1.csv")
    INTO TABLE `table1`
    FORMAT AS "csv"
    COLUMNS TERMINATED BY ","
    (user_id, user_gender, event_date, event_type)
)
WITH BROKER
```

> **説明**
>
> `column_list`パラメータは、データファイル内の列に一時的な名前を付けて、StarRocksテーブルの列にマッピングするために使用されます。

詳細な構文とパラメータの紹介については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Kafkaからインポートする

`file1.csv`ファイル内のデータがKafkaクラスターの`topic1`に保存されている場合、以下のステートメントを使用して[Routine Load](../loading/RoutineLoad.md)インポートジョブを作成し、データをインポートできます：

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

> **説明**
>
> `COLUMNS`パラメータは、データ内の列に一時的な名前を付けて、StarRocksテーブルの列にマッピングするために使用されます。

詳細な構文とパラメータの紹介については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

### データをクエリする

ローカルファイルシステム、HDFS、またはKafkaから上記のデータをインポートした後、`table1`テーブルをクエリすると以下のようになります：

```SQL
SELECT * FROM table1;
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

## インポート不要な行をフィルタリングする

ソースデータファイルには、StarRocksテーブルに不要な行が含まれている可能性があります。StarRocksは、WHERE句を使用してどの行をインポートするかを指定し、条件に合わないデータはフィルタリングされることをサポートしています。

この機能は以下のデータソースをサポートしています：

- ローカルファイルシステム


- HDFS と外部クラウドストレージシステム
  > **説明**
  >
  > ここでは HDFS を例に説明します。

- Kafka

このセクションでは、`file1.csv` ファイルと `table1` テーブルを例に説明します。`file1.csv` ファイルのイベントタイプを表す列の値が `1` の行のみを `table1` テーブルにインポートしたい場合、WHERE 句を使用してフィルタ条件 `event_type = 1` を指定できます。

### データのインポート

#### ローカルファイルシステムからのインポート

`file1.csv` ファイルがローカルファイルシステムにある場合、以下のコマンドで [Stream Load](../loading/StreamLoad.md) インポートジョブを作成してデータをインポートできます：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: user_id, user_gender, event_date, event_type" \
    -H "where: event_type=1" \
    -T file1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

詳細な文法とパラメータについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) を参照してください。

#### HDFS からのインポート

`file1.csv` ファイルが HDFS 上にある場合、以下のコマンドで [Broker Load](../loading/hdfs_load.md) インポートジョブを作成してデータをインポートできます：

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
WITH BROKER
```

詳細な文法とパラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

#### Kafka からのインポート

`file1.csv` ファイルのデータが Kafka クラスタの `topic1` にある場合、以下のコマンドで [Routine Load](../loading/RoutineLoad.md) インポートジョブを作成してデータをインポートできます：

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

詳細な文法とパラメータについては、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

### データのクエリ

ローカルファイルシステム、HDFS、または Kafka から上記のデータをインポートした後、`table1` テーブルを以下のようにクエリします：

```SQL
SELECT * FROM table1;
+------------+------------+---------+
| event_date | event_type | user_id |
+------------+------------+---------+
| 2020-05-20 |          1 |     354 |
| 2020-05-22 |          1 |     576 |
+------------+------------+---------+
2 rows in set (0.01 sec)
```

## 派生列の生成

ソースデータファイルのデータは、StarRocks テーブルにインポートする前に変換が必要な場合があります。StarRocks では、インポートコマンドまたはステートメント内で関数を使用してデータ変換を実行できます。

この機能は以下のデータソースをサポートしています：

- ローカルファイルシステム

- HDFS と外部クラウドストレージシステム
  > **説明**
  >
  > ここでは HDFS を例に説明します。

- Kafka

このセクションでは、`file2.csv` ファイルと `table2` テーブルを例に説明します。`file2.csv` ファイルには日付を表すタイムスタンプ形式のデータが1列だけ含まれており、[year](../sql-reference/sql-functions/date-time-functions/year.md)、[month](../sql-reference/sql-functions/date-time-functions/month.md)、[day](../sql-reference/sql-functions/date-time-functions/day.md) 関数を使用して `file2.csv` ファイルのデータを抽出し、`table2` テーブルの `year`、`month`、`day` 列にそれぞれ格納できます。

### データのインポート

#### ローカルファイルシステムからのインポート

`file2.csv` ファイルがローカルファイルシステムにある場合、以下のコマンドで [Stream Load](../loading/StreamLoad.md) インポートジョブを作成してデータをインポートできます：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns:date,year=year(date),month=month(date),day=day(date)" \
    -T file2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

> **説明**
>
> - `columns` パラメータを使用して、ソースデータファイルに含まれる**すべての列**を最初に宣言し、その後で派生列を宣言する必要があります。上記の例では、`columns` パラメータで `file2.csv` ファイルに含まれる唯一の列を一時的に `date` と宣言し、その後で関数を使用して変換される派生列 `year=year(date)`、`month=month(date)`、`day=day(date)` を宣言しています。
>
> - `column_name = function(column_name)` の形式はサポートされていません。必要に応じて派生列の前の列をリネームすることができます。例えば `column_name = func(temp_column_name)` のようにします。

詳細な文法とパラメータについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) を参照してください。

#### HDFS からのインポート

`file2.csv` ファイルが HDFS 上にある場合、以下のコマンドで [Broker Load](../loading/hdfs_load.md) インポートジョブを作成してデータをインポートできます：

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
WITH BROKER
```

> **説明**
>
> - 最初に `column_list` パラメータを使用してソースデータファイルに含まれるすべての列を宣言し、その後で SET 句を使用して派生列を宣言する必要があります。上記の例では、`column_list` パラメータで `file2.csv` ファイルに含まれる唯一の列を一時的に `date` と宣言し、その後で SET 句を使用して関数を使用して変換される派生列 `year=year(date)`、`month=month(date)`、`day=day(date)` を宣言しています。

詳細な文法とパラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

#### Kafka からのインポート

`file2.csv` ファイルのデータが Kafka クラスタの `topic2` にある場合、以下のコマンドで [Routine Load](../loading/RoutineLoad.md) インポートジョブを作成してデータをインポートできます：

```SQL
CREATE ROUTINE LOAD test_db.table2 ON table2
    COLUMNS TERMINATED BY ",",
    COLUMNS(date,year=year(date),month=month(date),day=day(date))
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker_host>:<kafka_broker_port>",
    "kafka_topic" = "topic2",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING"
);
```

> **説明**
>
> - 最初に `COLUMNS` パラメータを使用してソースデータファイルに含まれるすべての列を宣言し、その後で派生列を宣言する必要があります。上記の例では、`COLUMNS` パラメータで `file2.csv` ファイルに含まれる唯一の列を一時的に `date` と宣言し、その後で関数を使用して変換される派生列 `year=year(date)`、`month=month(date)`、`day=day(date)` を宣言しています。

詳細な文法とパラメータについては、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

### データのクエリ

ローカルファイルシステム、HDFS、または Kafka から上記のデータをインポートした後、`table2` テーブルを以下のようにクエリします：

```SQL
SELECT * FROM table2;
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

## ファイルパスからパーティションフィールドの内容を取得する

指定されたファイルパスにパーティションフィールドが存在する場合、StarRocks は `COLUMNS FROM PATH AS` パラメータを使用して、ファイルパスからどのパーティションフィールドの情報を抽出するかを指定することができます。これはソースデータファイルの列と同等です。このパラメータは HDFS からデータをインポートする際にのみ使用できます。

例えば、Hive によって生成された4つのデータファイルをインポートする場合、これらのファイルは HDFS 上の `/user/starrocks/data/input/` パスに保存されており、各データファイルは `date` パーティションフィールドに従ってパーティション分けされています。また、各データファイルにはイベントタイプとユーザーIDを表す2列のみが含まれています。以下のようになります：

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

### データのインポート

以下のコマンドで [Broker Load](../loading/hdfs_load.md) インポートジョブを作成し、`/user/starrocks/data/input/` パスのパーティションフィールド `date` の情報を取得し、ワイルドカード (*) を使用してこのパスのすべてのデータファイルを `table1` テーブルにインポートできます：

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
WITH BROKER
```

> **説明**
>
> 上記の例では、指定されたファイルパスのパーティションフィールド `date` が `table1` テーブルの `event_date` 列に対応しているため、SET 句を使用して `date` から `event_date` へのマッピングを行う必要があります。指定されたファイルパスのパーティションフィールドが StarRocks テーブルの列名と同じ場合、SET 句を使用してマッピングを指定する必要はありません。

詳細な文法とパラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

### データのクエリ

HDFS から上記のデータをインポートした後、`table1` テーブルを以下のようにクエリします：

```SQL
SELECT * FROM table1 WHERE event_date IS NOT NULL;
```
SELECT * FROM table1;
+------------+------------+---------+
| event_date | event_type | user_id |
+------------+------------+---------+
| 2020-05-22 |          1 |     576 |
| 2020-05-20 |          1 |     354 |
| 2020-05-21 |          2 |     465 |
| 2020-05-23 |          2 |     687 |
+------------+------------+---------+
4行がセットされました (0.01秒)