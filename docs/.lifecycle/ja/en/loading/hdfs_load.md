---
displayed_sidebar: English
---

# HDFSからデータを読み込む

import LoadMethodIntro from '../assets/commonMarkdown/loadMethodIntro.md'

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、HDFSからデータをロードするための次のオプションを提供します。

<LoadMethodIntro />

## 始める前に

### ソースデータを準備する

StarRocksにロードするソースデータがHDFSクラスタに正しく保存されていることを確認してください。このトピックでは、HDFSから`/user/amber/user_behavior_ten_million_rows.parquet`をStarRocksにロードすることを前提としています。

### 権限を確認する

<InsertPrivNote />

### 接続の詳細を収集する

シンプル認証方法を使用してHDFSクラスターとの接続を確立できます。シンプル認証を使用するには、HDFSクラスタのNameNodeへのアクセスに使用できるアカウントのユーザー名とパスワードを収集する必要があります。

## INSERT+FILES()を使用する

このメソッドはv3.1以降で使用可能で、現在はParquetとORCファイル形式のみをサポートしています。

### INSERT+FILES()の利点

[`FILES()`](../sql-reference/sql-functions/table-functions/files.md)は、指定したパス関連のプロパティに基づいてクラウドストレージに保存されているファイルを読み取り、ファイル内のデータのテーブルスキーマを推測し、その後ファイルからデータをデータ行として返します。

`FILES()`を使用すると、次のことができます：

- [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用してHDFSから直接データをクエリする。
- [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) (CTAS)を使用してテーブルを作成およびロードする。
- [INSERT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して既存のテーブルにデータをロードする。

### 典型的な例

#### SELECTを使用してHDFSから直接クエリする

SELECT+`FILES()`を使用してHDFSから直接クエリすることで、テーブルを作成する前にデータセットの内容をよく確認できます。例えば：

- データを保存せずにデータセットのプレビューを取得する。
- 最小値と最大値をクエリして、使用するデータ型を決定する。
- `NULL`値をチェックする。

次の例では、HDFSクラスタに格納されているデータファイル`/user/amber/user_behavior_ten_million_rows.parquet`をクエリします。

```SQL
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
)
LIMIT 3;
```

システムは以下のクエリ結果を返します：

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
| 543711 |  829192 |    2355072 | pv           | 2017-11-27 08:22:37 |
| 543711 | 2056618 |    3645362 | pv           | 2017-11-27 10:16:46 |
| 543711 | 1165492 |    3645362 | pv           | 2017-11-27 10:17:00 |
+--------+---------+------------+--------------+---------------------+
```

> **注記**
>
> 上記で返された列名はParquetファイルによって提供されます。

#### CTASを使用してテーブルを作成およびロードする

これは前の例の続きです。前のクエリはCREATE TABLE AS SELECT (CTAS)でラップされ、スキーマ推論を使用してテーブルの作成が自動化されます。つまり、StarRocksはテーブルスキーマを推測し、必要なテーブルを作成してからデータをテーブルにロードします。Parquetファイルを使用する場合、`FILES()`テーブル関数では列の名前と型を指定する必要はありません。

> **注記**
>
> スキーマ推論を使用する場合のCREATE TABLEの構文ではレプリカ数の設定ができないため、テーブル作成前に設定してください。以下の例は、レプリカが1つのシステムの場合です：
>
> ```SQL
> ADMIN SET FRONTEND CONFIG ('default_replication_num' = "1");
> ```

データベースを作成し、それに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

CTASを使用してテーブルを作成し、データファイル`/user/amber/user_behavior_ten_million_rows.parquet`のデータをテーブルにロードします：

```SQL
CREATE TABLE user_behavior_inferred AS
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

テーブルを作成したら、[DESCRIBE](../sql-reference/sql-statements/Utility/DESCRIBE.md)を使用してそのスキーマを表示できます：

```SQL
DESCRIBE user_behavior_inferred;
```

システムは以下のクエリ結果を返します：

```Plaintext
+--------------+------------------+------+-------+---------+-------+
| Field        | Type             | Null | Key   | Default | Extra |
+--------------+------------------+------+-------+---------+-------+
| UserID       | bigint           | YES  | true  | NULL    |       |
| ItemID       | bigint           | YES  | true  | NULL    |       |
| CategoryID   | bigint           | YES  | true  | NULL    |       |
| BehaviorType | varchar(1048576) | YES  | false | NULL    |       |
| Timestamp    | varchar(1048576) | YES  | false | NULL    |       |
+--------------+------------------+------+-------+---------+-------+
```

推論されたスキーマと手動で作成したスキーマを比較します：

- データ型
- NULL許容
- キーフィールド

変換先テーブルのスキーマをより適切に制御し、クエリのパフォーマンスを向上させるために、運用環境ではテーブルスキーマを手動で指定することをお勧めします。

テーブルをクエリして、データがロードされていることを確認します。例：

```SQL
SELECT * from user_behavior_inferred LIMIT 3;
```

データが正常にロードされたことを示す以下のクエリ結果が返されます：

```Plaintext
+--------+--------+------------+--------------+---------------------+
| UserID | ItemID | CategoryID | BehaviorType | Timestamp           |
+--------+--------+------------+--------------+---------------------+
|     84 |  56257 |    1879194 | pv           | 2017-11-26 05:56:23 |
|     84 | 108021 |    2982027 | pv           | 2017-12-02 05:43:00 |
|     84 | 390657 |    1879194 | pv           | 2017-11-28 11:20:30 |
+--------+--------+------------+--------------+---------------------+
```

#### INSERTを使用して既存のテーブルにロードする

たとえば、挿入先のテーブルをカスタマイズすることができます：

- 列のデータ型、NULL許容設定、またはデフォルト値
- キータイプと列
- データのパーティショニングとバケッティング

> **注記**
>
> 最も効率的なテーブル構造を作成するには、データの使用方法と列の内容に関する知識が必要です。このトピックではテーブル設計については扱いません。テーブル設計についての詳細は、[テーブルタイプ](../table_design/StarRocks_table_design.md)を参照してください。

この例では、テーブルのクエリ方法とParquetファイル内のデータに関する知識に基づいてテーブルを作成しています。Parquetファイル内のデータに関する知識は、HDFSでファイルを直接クエリすることで得られます。

- HDFSでのデータセットのクエリは、`Timestamp`列に`datetime`データ型に一致するデータが含まれていることを示しているため、列の型は次のDDLで指定されます。
- HDFSでのデータのクエリにより、データセットに`NULL`値がないことがわかるため、DDLはどの列もNULL許容として設定しません。
- 想定されるクエリの種類に関する知識に基づいて、ソートキーとバケット列が`UserID`列に設定されます。このデータのユースケースは異なる場合があるため、ソートキーとして`ItemID`を加えて、または`UserID`の代わりに使用することを決定するかもしれません。

データベースを作成し、それに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

テーブルを手動で作成します（テーブルのスキーマはHDFSからロードするParquetファイルと同じであることをお勧めします）：

```SQL
CREATE TABLE user_behavior_declared
(
    UserID int(11),
    ItemID int(11),
    CategoryID int(11),
    BehaviorType varchar(65533),
    Timestamp datetime
)
ENGINE = OLAP 
DUPLICATE KEY(UserID)
DISTRIBUTED BY HASH(UserID)
PROPERTIES
(
    "replication_num" = "1"
);
```

テーブルを作成したら、INSERT INTO SELECT FROM FILES()でロードできます：

```SQL
INSERT INTO user_behavior_declared
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

読み込みが完了したら、テーブルをクエリして、データがロードされたことを確認できます。例：

```SQL
SELECT * FROM user_behavior_declared LIMIT 3;
```

データが正常に読み込まれたことを示す次のクエリ結果が返されます。

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
|    107 | 1568743 |    4476428 | pv           | 2017-11-25 14:29:53 |
|    107 |  470767 |    1020087 | pv           | 2017-11-25 14:32:31 |
|    107 |  358238 |    1817004 | pv           | 2017-11-25 14:43:23 |
+--------+---------+------------+--------------+---------------------+
```

#### ロード進捗の確認

`information_schema.loads` ビューからINSERTジョブの進捗を照会できます。この機能はv3.1以降でサポートされています。例：

```SQL
SELECT * FROM information_schema.loads ORDER BY JOB_ID DESC;
```

複数のロードジョブをサブミットした場合は、ジョブに関連付けられた `LABEL` でフィルタリングできます。例：

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'insert_0d86c3f9-851f-11ee-9c3e-00163e044958' \G
*************************** 1. row ***************************
              JOB_ID: 10214
               LABEL: insert_0d86c3f9-851f-11ee-9c3e-00163e044958
       DATABASE_NAME: mydatabase
               STATE: FINISHED
            PROGRESS: ETL:100%; LOAD:100%
                TYPE: INSERT
            PRIORITY: NORMAL
           SCAN_ROWS: 10000000
       FILTERED_ROWS: 0
     UNSELECTED_ROWS: 0
           SINK_ROWS: 10000000
            ETL_INFO:
           TASK_INFO: resource:N/A; timeout(s):300; max_filter_ratio:0.0
         CREATE_TIME: 2023-11-17 15:58:14
      ETL_START_TIME: 2023-11-17 15:58:14
     ETL_FINISH_TIME: 2023-11-17 15:58:14
     LOAD_START_TIME: 2023-11-17 15:58:14
    LOAD_FINISH_TIME: 2023-11-17 15:58:18
         JOB_DETAILS: {"All backends":{"0d86c3f9-851f-11ee-9c3e-00163e044958":[10120]},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":311710786,"InternalTableLoadRows":10000000,"ScanBytes":581574034,"ScanRows":10000000,"TaskNumber":1,"Unfinished backends":{"0d86c3f9-851f-11ee-9c3e-00163e044958":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: NULL
        TRACKING_SQL: NULL
REJECTED_RECORD_PATH: NULL
```

`loads` ビューで提供されるフィールドについての情報は、[情報スキーマ](../reference/information_schema/loads.md)を参照してください。

> **注記**
>
> INSERTは同期コマンドです。INSERTジョブがまだ実行中の場合は、その実行状態を確認するために別のセッションを開く必要があります。

## Broker Loadの使用

非同期のBroker Loadプロセスは、HDFSへの接続、データの取得、およびStarRocksへのデータの保存を処理します。

この方法は、Parquet、ORC、CSVファイル形式をサポートしています。

### Broker Loadの利点

- Broker Loadは、ロード中のUPSERTおよびDELETE操作などの[データ変換](../loading/Etl_in_loading.md)と[データ変更](../loading/Load_to_Primary_Key_tables.md)をサポートしています。
- Broker Loadはバックグラウンドで実行され、クライアントはジョブが続行されるために接続を維持する必要はありません。
- Broker Loadは長時間実行されるジョブに適しており、デフォルトのタイムアウトは4時間です。
- ParquetとORCファイル形式に加えて、Broker LoadはCSVファイルもサポートしています。

### データフロー

![Broker Loadのワークフロー](../assets/broker_load_how-to-work_en.png)

1. ユーザーがロードジョブを作成します。
2. フロントエンド(FE)はクエリプランを作成し、バックエンドノード(BE)にプランを配布します。
3. BEはソースからデータを取得し、StarRocksにデータをロードします。

### 典型的な例

テーブルを作成し、HDFSからデータファイル`/user/amber/user_behavior_ten_million_rows.parquet`を取得するロードプロセスを開始し、データロードの進行状況と成功を確認します。

#### データベースとテーブルの作成

データベースを作成し、それに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

テーブルを手動で作成します（HDFSからロードするParquetファイルと同じスキーマを持つテーブルを作成することをお勧めします）：

```SQL
CREATE TABLE user_behavior
(
    UserID int(11),
    ItemID int(11),
    CategoryID int(11),
    BehaviorType varchar(65533),
    Timestamp datetime
)
ENGINE = OLAP 
DUPLICATE KEY(UserID)
DISTRIBUTED BY HASH(UserID)
PROPERTIES
(
    "replication_num" = "1"
);
```

#### Broker Loadを開始する

次のコマンドを実行して、データファイル`/user/amber/user_behavior_ten_million_rows.parquet`から`user_behavior`テーブルにデータをロードするBroker Loadジョブを開始します：

```SQL
LOAD LABEL user_behavior
(
    DATA INFILE("hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet")
    INTO TABLE user_behavior
    FORMAT AS "parquet"
 )
 WITH BROKER
(
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
)
PROPERTIES
(
    "timeout" = "72000"
);
```

このジョブには4つの主要なセクションがあります：

- `LABEL`：ロードジョブの状態を照会する際に使用される文字列。
- `LOAD`宣言：ソースURI、ソースデータ形式、および宛先テーブル名。
- `BROKER`：ソースへの接続詳細。
- `PROPERTIES`：タイムアウト値およびロードジョブに適用するその他のプロパティ。

詳細な構文とパラメーターの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### ロード進捗の確認

`information_schema.loads`ビューからBroker Loadジョブの進捗を照会できます。この機能はv3.1以降でサポートされています。

```SQL
SELECT * FROM information_schema.loads;
```

`loads`ビューで提供されるフィールドについての情報は、[情報スキーマ](../reference/information_schema/loads.md)を参照してください。

複数のロードジョブをサブミットした場合は、ジョブに関連付けられた`LABEL`でフィルタリングできます。例：

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'user_behavior';
```

以下の出力には、ロードジョブ`user_behavior`の2つのエントリがあります：

- 最初のレコードは`CANCELLED`の状態を示しています。`ERROR_MSG`までスクロールすると、`listPath failed`が原因でジョブが失敗したことがわかります。
- 2番目のレコードは`FINISHED`の状態を示しており、ジョブが成功したことを意味します。

```Plaintext
JOB_ID|LABEL                                      |DATABASE_NAME|STATE    |PROGRESS           |TYPE  |PRIORITY|SCAN_ROWS|FILTERED_ROWS|UNSELECTED_ROWS|SINK_ROWS|ETL_INFO|TASK_INFO                                           |CREATE_TIME        |ETL_START_TIME     |ETL_FINISH_TIME    |LOAD_START_TIME    |LOAD_FINISH_TIME   |JOB_DETAILS                                                                                                                                                                                                                                                    |ERROR_MSG                             |TRACKING_URL|TRACKING_SQL|REJECTED_RECORD_PATH|
------+-------------------------------------------+-------------+---------+-------------------+------+--------+---------+-------------+---------------+---------+--------+----------------------------------------------------+-------------------+-------------------+-------------------+-------------------+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+------------+--------------------+
 10121|user_behavior                              |mydatabase   |CANCELLED|ETL:N/A; LOAD:N/A  |BROKER|NORMAL  |        0|            0|              0|        0|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:59:30|                   |                   |                   |2023-08-10 14:59:34|{"All backends":{},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":0,"InternalTableLoadRows":0,"ScanBytes":0,"ScanRows":0,"TaskNumber":0,"Unfinished backends":{}}                                                                                        |type:ETL_RUN_FAIL; msg:listPath failed|            |            |                    |
 10106|user_behavior                              |mydatabase   |FINISHED |ETL:100%; LOAD:100%|BROKER|NORMAL  | 86953525|            0|              0| 86953525|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:50:15|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:55:10|{"All backends":{"a5fe5e1d-d7d0-4826-ba99-c7348f9a5f2f":[10004]},"FileNumber":1,"FileSize":1225637388,"InternalTableLoadBytes":2710603082,"InternalTableLoadRows":86953525,"ScanBytes":1225637388,"ScanRows":86953525,"TaskNumber":1,"Unfinished backends":{"a5|                                      |            |            |                    |
```

ロードジョブが完了したことを確認したら、宛先テーブルの一部をチェックして、データが正常にロードされたかどうかを確認できます。例：

```SQL
SELECT * FROM user_behavior LIMIT 3;
```

データが正常にロードされたことを示す、次のクエリ結果が返されます。

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
|    142 | 2869980 |    2939262 | pv           | 2017-11-25 03:43:22 |
|    142 | 2522236 |    1669167 | pv           | 2017-11-25 15:14:12 |
|    142 | 3031639 |    3607361 | pv           | 2017-11-25 15:19:25 |
+--------+---------+------------+--------------+---------------------+
```

## Pipeの使用

v3.2から、StarRocksはPipeロード方式を提供し、現在ParquetとORCファイル形式のみをサポートしています。

### Pipeの利点

Pipeは連続データロードと大規模データロードに理想的です。

- **マイクロバッチによる大規模データロードは、データエラーによるリトライコストを削減します。**

  Pipeを利用することで、StarRocksは大量のデータファイルを効率的にロードできます。Pipeはファイルの数やサイズに基づいて自動的に分割し、ロードジョブを小さな連続タスクに分解します。このアプローチにより、一つのファイルのエラーがロードジョブ全体に影響を与えないようになります。各ファイルのロード状態はPipeによって記録され、エラーが含まれるファイルを簡単に特定して修正することができます。データエラーによるリトライの必要性を最小限に抑えることで、コストを削減するのに役立ちます。

- **連続データロードは人手を削減します。**

  Pipeは、新規または更新されたデータファイルを特定の場所に書き込み、これらのファイルからStarRocksに新しいデータを継続的にロードするのに役立ちます。`"AUTO_INGEST" = "TRUE"`を指定してPipeジョブを作成すると、指定されたパスに保存されているデータファイルの変更を常に監視し、データファイルから新規または更新されたデータをStarRocksテーブルに自動的にロードします。

さらに、Pipeはファイルのユニーク性チェックを行い、データの重複ロードを防ぎます。ロードプロセス中、Pipeはファイル名とダイジェストに基づいて各データファイルのユニーク性をチェックします。特定のファイル名とダイジェストを持つファイルが既にPipeジョブによって処理されている場合、Pipeジョブは同じファイル名とダイジェストを持つ後続のファイルをすべてスキップします。HDFSでは`LastModifiedTime`をファイルダイジェストとして使用します。

各データファイルのロード状態は記録され、`information_schema.pipe_files`ビューに保存されます。ビューに関連付けられたPipeジョブが削除されると、そのジョブでロードされたファイルに関するレコードも削除されます。

### データフロー

![Pipeデータフロー](../assets/pipe_data_flow.png)

### PipeとINSERT+FILES()の違い

Pipeジョブは、各データファイルのサイズと行数に基づいて1つ以上のトランザクションに分割されます。ユーザーはロードプロセス中に中間結果をクエリできます。一方、INSERT+`FILES()`ジョブは単一トランザクションとして処理され、ユーザーはロードプロセス中にデータを表示できません。

### ファイルのロード順序

StarRocksは、各Pipeジョブに対してファイルキューを維持し、そこからマイクロバッチとしてデータファイルを取得してロードします。Pipeはデータファイルがアップロードされた順序と同じ順序でロードされることを保証しません。したがって、新しいデータが古いデータよりも先にロードされる可能性があります。

### 典型的な例

#### データベースとテーブルの作成

データベースを作成し、切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

手動でテーブルを作成します（HDFSからロードするParquetファイルと同じスキーマを持つテーブルを作成することを推奨します）。

```SQL
CREATE TABLE user_behavior_replica
(
    UserID int(11),
    ItemID int(11),
    CategoryID int(11),
    BehaviorType varchar(65533),
    Timestamp datetime
)
ENGINE = OLAP 
DUPLICATE KEY(UserID)
DISTRIBUTED BY HASH(UserID)
PROPERTIES
(
    "replication_num" = "1"
);
```

#### Pipeジョブの開始

次のコマンドを実行して、データファイル`/user/amber/user_behavior_ten_million_rows.parquet`から`user_behavior_replica`テーブルへのデータロードを開始するPipeジョブを開始します。

```SQL
CREATE PIPE user_behavior_replica
PROPERTIES
(
    "AUTO_INGEST" = "TRUE"
)
AS
INSERT INTO user_behavior_replica
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
); 
```

このジョブには4つの主要セクションがあります。

- `pipe_name`: パイプの名前です。パイプ名は、パイプが属するデータベース内で一意でなければなりません。
- `INSERT_SQL`: 指定されたソースデータファイルから宛先テーブルへデータをロードするために使用されるINSERT INTO SELECT FROM FILESステートメントです。
- `PROPERTIES`: パイプを実行する方法を指定するオプションのパラメータセットです。これには`AUTO_INGEST`、`POLL_INTERVAL`、`BATCH_SIZE`、`BATCH_FILES`が含まれます。これらのプロパティは`"key" = "value"`形式で指定します。

詳細な構文とパラメータの説明については、[CREATE PIPE](../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)を参照してください。

#### ロード進行状況の確認

- [SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)を使用してPipeジョブの進行状況を照会します。

  ```SQL
  SHOW PIPES;
  ```

  複数のロードジョブを提出した場合、ジョブに関連付けられた`NAME`でフィルタリングできます。例えば：

  ```SQL
  SHOW PIPES WHERE NAME = 'user_behavior_replica' \G
  *************************** 1. row ***************************
  DATABASE_NAME: mydatabase
        PIPE_ID: 10252
      PIPE_NAME: user_behavior_replica
          STATE: RUNNING
     TABLE_NAME: mydatabase.user_behavior_replica
    LOAD_STATUS: {"loadedFiles":1,"loadedBytes":132251298,"loadingFiles":0,"lastLoadedTime":"2023-11-17 16:13:22"}
     LAST_ERROR: NULL
   CREATED_TIME: 2023-11-17 16:13:15
  1 row in set (0.00 sec)
  ```

- [`information_schema.pipes`](../reference/information_schema/pipes.md)ビューからPipeジョブの進行状況を照会します。

  ```SQL
  SELECT * FROM information_schema.pipes;
  ```

  複数のロードジョブを提出した場合、ジョブに関連付けられた`PIPE_NAME`でフィルタリングできます。例えば：

  ```SQL
  SELECT * FROM information_schema.pipes WHERE pipe_name = 'user_behavior_replica' \G
  *************************** 1. row ***************************
  DATABASE_NAME: mydatabase
        PIPE_ID: 10252
      PIPE_NAME: user_behavior_replica
          STATE: RUNNING
     TABLE_NAME: mydatabase.user_behavior_replica
    LOAD_STATUS: {"loadedFiles":1,"loadedBytes":132251298,"loadingFiles":0,"lastLoadedTime":"2023-11-17 16:13:22"}
     LAST_ERROR:
   CREATED_TIME: 2023-11-17 16:13:15
  1 row in set (0.00 sec)
  ```

#### ファイルステータスの確認

[`information_schema.pipe_files`](../reference/information_schema/pipe_files.md)ビューからロードされたファイルのロードステータスを照会できます。

```SQL
SELECT * FROM information_schema.pipe_files;
```

複数のロードジョブを提出した場合、ジョブに関連付けられた`PIPE_NAME`でフィルタリングできます。例えば：

```SQL
SELECT * FROM information_schema.pipe_files WHERE pipe_name = 'user_behavior_replica' \G
*************************** 1. row ***************************
   DATABASE_NAME: mydatabase
         PIPE_ID: 10252
       PIPE_NAME: user_behavior_replica
       FILE_NAME: hdfs://172.26.195.67:9000/user/amber/user_behavior_ten_million_rows.parquet
    FILE_VERSION: 1700035418838
       FILE_SIZE: 132251298
   LAST_MODIFIED: 2023-11-15 08:03:38
      LOAD_STATE: FINISHED
     STAGED_TIME: 2023-11-17 16:13:16
 START_LOAD_TIME: 2023-11-17 16:13:17
FINISH_LOAD_TIME: 2023-11-17 16:13:22
       ERROR_MSG:
1 row in set (0.02 sec)
```

#### Pipeの管理


作成したパイプを変更、中断、再開、削除するか、またはクエリを実行して、特定のデータファイルのロードを再試行することができます。詳細については、[ALTER PIPE](../sql-reference/sql-statements/data-manipulation/ALTER_PIPE.md)、[SUSPEND または RESUME PIPE](../sql-reference/sql-statements/data-manipulation/SUSPEND_or_RESUME_PIPE.md)、[DROP PIPE](../sql-reference/sql-statements/data-manipulation/DROP_PIPE.md)、[SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)、および [RETRY FILE](../sql-reference/sql-statements/data-manipulation/RETRY_FILE.md) を参照してください。
