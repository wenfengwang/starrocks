---
displayed_sidebar: Chinese
---

# HDFSからのインポート

import LoadMethodIntro from '../assets/commonMarkdown/loadMethodIntro.md'

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、以下の方法でHDFSからデータをインポートできます:

<LoadMethodIntro />

## 準備

### データソースの準備

インポートするデータがHDFSクラスタに保存されていることを確認してください。このドキュメントでは、インポートするデータファイルが `/user/amber/user_behavior_ten_million_rows.parquet` であると仮定しています。

### 権限の確認

<InsertPrivNote />

### リソースアクセス設定の取得

HDFSにはシンプル認証方式でアクセスでき、事前にHDFSクラスタのNameNodeノードにアクセスするためのユーザー名とパスワードを取得する必要があります。

## INSERT+FILES()を使用したインポート

この機能はバージョン3.1からサポートされています。現在、ParquetとORCファイル形式のみをサポートしています。

### INSERT+FILES()の利点

`FILES()`は指定されたデータパスなどのパラメータをもとにデータを読み取り、データファイルの形式や列情報などからテーブル構造を自動的に推測し、最終的にデータファイル内のデータを行として返します。

`FILES()`を使用することで、あなたは以下を行うことができます:

- [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して、HDFSから直接データをクエリします。
- [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)（CTASとも呼ばれます）ステートメントを使用して、自動的にテーブルを作成し、データをインポートします。
- 手動でテーブルを作成し、[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) を使用してデータをインポートします。

### 操作例

#### SELECTを使用して直接データをクエリする

SELECT+`FILES()`を使用して、HDFS内のデータを直接クエリし、テーブルを作成する前にインポートするデータを全体的に理解することができます。その利点は以下の通りです:

- データを保存する必要なく、それを閲覧することができます。
- データの最大値や最小値を確認し、どのデータ型を使用する必要があるかを決定できます。
- データに`NULL`値が含まれているかどうかを確認できます。

例えば、HDFSに保存されているデータファイル `/user/amber/user_behavior_ten_million_rows.parquet` をクエリするには：

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

> **説明**
>
> 上記の結果における列名は、元のParquetファイルで定義された列名です。

#### CTASを使用して自動的にテーブルを作成し、データをインポートする

この例は前の例の続きです。この例では、CREATE TABLE AS SELECT（CTAS）ステートメントに前の例のSELECTクエリを埋め込むことで、StarRocksは自動的にテーブル構造を推測し、テーブルを作成し、新しいテーブルにデータをインポートすることができます。Parquet形式のファイルには列名とデータ型が含まれているため、列名やデータ型を指定する必要はありません。

> **説明**
>
> テーブル構造の推測機能を使用する場合、CREATE TABLEステートメントではレプリケーション数を設定することはできません。したがって、テーブルを作成する前にレプリケーション数を設定する必要があります。例えば、以下のコマンドを使用してレプリケーション数を`1`に設定できます：
>
> ```SQL
> ADMIN SET FRONTEND CONFIG ('default_replication_num' = "1");
> ```

以下のステートメントを使用してデータベースを作成し、そのデータベースに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

CTASを使用して自動的にテーブルを作成し、データファイル `/user/amber/user_behavior_ten_million_rows.parquet` からデータを新しいテーブルにインポートします：

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

テーブル作成後、[DESCRIBE](../sql-reference/sql-statements/Utility/DESCRIBE.md) を使用して新しいテーブルの構造を確認できます：

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

システムによって推測されたテーブル構造と手動で作成したテーブル構造を以下の点で比較します：

- データ型
- `NULL`値を許可するかどうか
- キーとして定義されたフィールド

本番環境では、目標とするテーブルの構造をより良く制御し、より高いクエリ性能を実現するために、手動でテーブルを作成し、テーブル構造を指定することをお勧めします。

新しく作成されたテーブルのデータをクエリして、データが正常にインポートされたことを確認できます。例えば：

```SQL
SELECT * from user_behavior_inferred LIMIT 3;
```

システムは以下のクエリ結果を返し、データが正常にインポートされたことを示します：

```Plaintext
+--------+--------+------------+--------------+---------------------+
| UserID | ItemID | CategoryID | BehaviorType | Timestamp           |
+--------+--------+------------+--------------+---------------------+
|     84 |  56257 |    1879194 | pv           | 2017-11-26 05:56:23 |
|     84 | 108021 |    2982027 | pv           | 2017-12-02 05:43:00 |
|     84 | 390657 |    1879194 | pv           | 2017-11-28 11:20:30 |
+--------+--------+------------+--------------+---------------------+
```

#### INSERTを使用して手動でテーブルを作成し、データをインポートする

実際のビジネスシナリオでは、目標とするテーブルの構造をカスタマイズする必要があるかもしれません。これには以下が含まれます：

- 各列のデータ型とデフォルト値、および`NULL`値を許可するかどうか
- どの列をキーとして定義するか、およびそれらの列のデータ型
- データのパーティション分割とバケット分割

> **説明**
>
> 効率的なテーブル構造を設計するためには、テーブル内のデータの用途や各列の内容について深く理解する必要があります。このドキュメントではテーブル設計について詳しくは述べませんが、詳細については[テーブル設計](../table_design/StarRocks_table_design.md)を参照してください。

この例では、Parquet形式のソースファイル内のデータの特徴や、将来のクエリ用途などに基づいて目標とするテーブルを定義し、作成する方法を主に示しています。テーブルを作成する前に、HDFSに保存されているソースファイルを確認して、ソースファイル内のデータの特徴を理解することができます。例えば：

- ソースファイルには`datetime`型の`Timestamp`列が含まれているため、テーブル作成ステートメントにも`datetime`型の`Timestamp`列を定義する必要があります。
- ソースファイルのデータには`NULL`値が含まれていないため、テーブル作成ステートメントで`NULL`値を許可する列を定義する必要はありません。
- クエリされたデータ型に基づいて、テーブル作成ステートメントで`UserID`列をソートキーおよびバケットキーとして定義できます。実際のビジネスシナリオに応じて、`ItemID`などの他の列を定義するか、`UserID`と他の列の組み合わせをソートキーとして定義することもできます。

以下のステートメントを使用してデータベースを作成し、そのデータベースに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

以下のステートメントを使用して手動でテーブルを作成します（テーブル構造はHDFSに保存されているインポートするデータの構造と一致することをお勧めします）：

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

テーブル作成後、INSERT INTO SELECT FROM FILES()を使用してテーブルにデータをインポートできます：

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

インポートが完了したら、新しく作成したテーブルのデータをクエリして、データが正常にインポートされたことを確認できます。例えば：

```SQL
SELECT * FROM user_behavior_declared LIMIT 3;
```

システムは以下のクエリ結果を返し、データが正常にインポートされたことを示しています：

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
|    107 | 1568743 |    4476428 | pv           | 2017-11-25 14:29:53 |
|    107 |  470767 |    1020087 | pv           | 2017-11-25 14:32:31 |
|    107 |  358238 |    1817004 | pv           | 2017-11-25 14:43:23 |
+--------+---------+------------+--------------+---------------------+
```

#### インポート進捗の確認

`information_schema.loads` ビューを使用して、インポートジョブの進捗を確認できます。この機能はバージョン 3.1 以降でサポートされています。例えば：

```SQL
SELECT * FROM information_schema.loads ORDER BY JOB_ID DESC;
```

複数のインポートジョブを提出した場合は、`LABEL` を使用して、確認したいジョブをフィルタリングできます。例えば：

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

`loads` ビューが提供するフィールドの詳細については、[Information Schema](../reference/information_schema/loads.md) を参照してください。

> **注記**
>
> INSERT 文は同期コマンドであるため、ジョブが実行中の場合は、別のセッションを開いて INSERT ジョブの実行状況を確認する必要があります。

## Broker Load を使用したインポート

Broker Load は非同期のインポート方法であり、HDFS との接続を確立し、データを取得して StarRocks に保存します。

現在、Parquet、ORC、CSV の三つのファイル形式をサポートしています。

### Broker Load の利点

- Broker Load はインポートプロセス中に[データ変換](../loading/Etl_in_loading.md)や [UPSERT、DELETE などのデータ変更操作](../loading/Load_to_Primary_Key_tables.md)をサポートしています。
- Broker Load はバックグラウンドで実行され、クライアントが接続を維持していなくても、インポートジョブが中断されないことを保証します。
- Broker Load ジョブのデフォルトタイムアウトは 4 時間で、データ量が多く、インポート実行時間が長いシナリオに適しています。
- Parquet と ORC ファイル形式に加えて、Broker Load は CSV ファイル形式もサポートしています。

### 動作原理

![Broker Load 原理図](../assets/broker_load_how-to-work_zh.png)

1. ユーザーがインポートジョブを作成します。
2. FE がクエリプランを生成し、それを分割して各 BE に割り当てて実行します。
3. 各 BE がデータソースからデータを取得し、StarRocks にインポートします。

### 操作例

StarRocks テーブルを作成し、HDFS クラスターからデータファイル `/user/amber/user_behavior_ten_million_rows.parquet` のデータを取得するインポートジョブを開始し、インポートプロセスと結果が成功したことを確認します。

#### データベースとテーブルの作成

以下のステートメントでデータベースを作成し、そのデータベースに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

以下のステートメントで手動でテーブルを作成します（テーブル構造は、HDFS クラスターに保存されているインポート対象のデータ構造と一致することをお勧めします）：

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

#### インポートジョブの提出

以下のコマンドを実行して Broker Load ジョブを作成し、データファイル `/user/amber/user_behavior_ten_million_rows.parquet` のデータをテーブル `user_behavior` にインポートします：

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

インポートステートメントには四つの部分が含まれています：

- `LABEL`：インポートジョブのラベルで、文字列型であり、インポートジョブの状態をクエリするために使用できます。
- `LOAD` 宣言：ソースデータファイルの URI、ソースデータファイルの形式、およびターゲットテーブルの名前など、ジョブの記述情報を含みます。
- `BROKER`：データソースへの接続認証情報の設定です。
- `PROPERTIES`：タイムアウト時間などのオプションのジョブ属性を指定するために使用されます。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

#### インポート進捗の確認

`information_schema.loads` ビューを使用して、インポートジョブの進捗を確認できます。この機能はバージョン 3.1 以降でサポートされています。

```SQL
SELECT * FROM information_schema.loads;
```

`loads` ビューが提供するフィールドの詳細については、[Information Schema](../reference/information_schema/loads.md) を参照してください。

複数のインポートジョブを提出した場合は、`LABEL` を使用して、確認したいジョブをフィルタリングできます。例えば：

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'user_behavior';
```

例えば、以下の戻り値には、インポートジョブ `user_behavior` に関する二つのレコードがあります：

- 最初のレコードは、インポートジョブの状態が `CANCELLED` であることを示しています。レコード内の `ERROR_MSG` フィールドを通じて、`listPath failed` がジョブエラーの原因であることが確認できます。
- 二番目のレコードは、インポートジョブの状態が `FINISHED` であり、ジョブが成功したことを示しています。

```Plaintext
JOB_ID|LABEL                                      |DATABASE_NAME|STATE    |PROGRESS           |TYPE  |PRIORITY|SCAN_ROWS|FILTERED_ROWS|UNSELECTED_ROWS|SINK_ROWS|ETL_INFO|TASK_INFO                                           |CREATE_TIME        |ETL_START_TIME     |ETL_FINISH_TIME    |LOAD_START_TIME    |LOAD_FINISH_TIME   |JOB_DETAILS                                                                                                                                                                                                                                                    |ERROR_MSG                             |TRACKING_URL|TRACKING_SQL|REJECTED_RECORD_PATH|
------+-------------------------------------------+-------------+---------+-------------------+------+--------+---------+-------------+---------------+---------+--------+----------------------------------------------------+-------------------+-------------------+-------------------+-------------------+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+------------+--------------------+
 10121|user_behavior                              |mydatabase   |CANCELLED|ETL:N/A; LOAD:N/A  |BROKER|NORMAL  |        0|            0|              0|        0|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:59:30|                   |                   |                   |2023-08-10 14:59:34|{"All backends":{},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":0,"InternalTableLoadRows":0,"ScanBytes":0,"ScanRows":0,"TaskNumber":0,"Unfinished backends":{}}                                                                                        |type:ETL_RUN_FAIL; msg:listPath failed|            |            |                    |

 10106|user_behavior                              |mydatabase   |FINISHED |ETL:100%; LOAD:100%|BROKER|NORMAL  | 86953525|            0|              0| 86953525|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:50:15|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:55:10|{"All backends":{"a5fe5e1d-d7d0-4826-ba99-c7348f9a5f2f":[10004]},"FileNumber":1,"FileSize":1225637388,"InternalTableLoadBytes":2710603082,"InternalTableLoadRows":86953525,"ScanBytes":1225637388,"ScanRows":86953525,"TaskNumber":1,"Unfinished backends":{"a5|                                      |            |            |                    |
```

インポートジョブが完了した後、テーブル内でデータをクエリして、データが正常にインポートされたかを確認できます。例えば：

```SQL
SELECT * from user_behavior LIMIT 3;
```

システムは以下のクエリ結果を返し、データが正常にインポートされたことを示しています：

```Plaintext
+--------+--------+------------+--------------+---------------------+
| UserID | ItemID | CategoryID | BehaviorType | Timestamp           |
+--------+--------+------------+--------------+---------------------+
|     58 | 158350 |    2355072 | pv           | 2017-11-27 13:06:51 |
|     58 | 158590 |    3194735 | pv           | 2017-11-27 02:21:04 |
|     58 | 215073 |    3002561 | pv           | 2017-11-30 10:55:42 |
+--------+--------+------------+--------------+---------------------+
```

## Pipe を通じたインポート

バージョン3.2から、StarRocksはPipeを通じたインポート方法を提供し、現在はParquetとORCファイル形式のみをサポートしています。

### Pipeの利点

Pipeは、大規模なバッチデータのインポートや、継続的なデータのインポートに適しています：

- **大規模なバッチインポートで、エラーのリトライコストを削減。**

  インポートする必要があるデータファイルが多く、データ量が大きい場合、Pipeはファイルの数やサイズに応じて、ディレクトリ内のファイルを自動的に分割し、大きなインポートジョブを複数の小さなシリアルインポートタスクに分割します。単一ファイルのデータエラーが全体のインポートジョブの失敗を引き起こすことはありません。また、Pipeは各ファイルのインポート状態を記録します。インポートが終了した後、エラーのあったデータファイルを修正し、修正後のデータファイルを再インポートするだけで済みます。これにより、データエラーのリトライコストを削減できます。

- **継続的なインポートを中断せずに行い、人的操作コストを削減。**

  新規または変更されたデータファイルを特定のフォルダに書き込み、その新規データを継続的にStarRocksにインポートする必要がある場合、`"AUTO_INGEST" = "TRUE"`を指定したステートメントで、Pipeに基づく継続的なインポートジョブを作成するだけで済みます。そのPipeは、指定されたパス下のデータファイルの変更を継続的に監視し、新規または変更されたデータファイルを自動的にStarRocksのターゲットテーブルにインポートします。

さらに、Pipeはファイルのユニークネスを判断し、重複データのインポートを避けます。インポートプロセス中に、Pipeはファイル名とファイルに対応するダイジェスト値に基づいてデータファイルが重複していないかを判断します。もしファイル名とファイルダイジェスト値が同じPipeインポートジョブで既に処理されていた場合、後続のインポートは既に処理されたファイルを自動的にスキップします。なお、HDFSは`LastModifiedTime`をファイルダイジェストとして使用します。

インポートプロセス中のファイル状態は`information_schema.pipe_files`ビューに記録され、このビューを通じてPipeインポートジョブの各ファイルのインポート状態を確認できます。もし関連するPipeジョブが削除された場合、そのビューの関連記録も同期してクリアされます。

### 動作原理

![Pipeの動作原理](../assets/pipe_data_flow.png)

### PipeとINSERT+FILES()の違い

Pipeインポート操作は、各データファイルのサイズと行数に基づいて、一つまたは複数のトランザクションに分割され、インポートプロセス中の中間結果がユーザーに表示されます。INSERT+`FILES()`インポート操作は一つの全体トランザクションであり、インポートプロセス中のデータはユーザーに表示されません。

### ファイルのインポート順序

Pipeインポート操作は内部でファイルキューを維持し、キューから対応するファイルをバッチで取り出してインポートします。Pipeはファイルのインポート順序とファイルのアップロード順序が一致することを保証できませんので、新しいデータが古いデータよりも早くインポートされる可能性があります。

### 操作例

#### データベースとテーブルの作成

以下のステートメントでデータベースを作成し、そのデータベースに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

以下のステートメントで手動でテーブルを作成します（テーブル構造はHDFSに保存されているインポート対象データ構造と一致することをお勧めします）：

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

#### インポートジョブの提出

以下のコマンドを実行してPipeジョブを作成し、データファイル`/user/amber/user_behavior_ten_million_rows.parquet`のデータをテーブル`user_behavior_replica`にインポートします：

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

インポートステートメントには以下の4つの部分が含まれています：

- `pipe_name`：Pipeの名前。この名前はPipeが所属するデータベース内で一意でなければなりません。
- `INSERT_SQL`：INSERT INTO SELECT FROM FILESステートメント。指定されたソースデータファイルからデータをターゲットテーブルにインポートするために使用されます。
- `PROPERTIES`：`AUTO_INGEST`、`POLL_INTERVAL`、`BATCH_SIZE`、`BATCH_FILES`など、Pipeの実行を制御するいくつかのパラメータを設定するために使用されます。形式は`"key" = "value"`です。

詳細な構文とパラメータの説明については、[CREATE PIPE](../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)を参照してください。

#### インポート進捗の確認

- [SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)を使用して、現在のデータベース内のインポートジョブを確認します。

  ```SQL
  SHOW PIPES;
  ```

  複数のインポートジョブを提出した場合、`NAME`を使用して確認したいインポートジョブをフィルタリングできます。例えば：

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

- [`information_schema.pipes`](../reference/information_schema/pipes.md)ビューを使用して、現在のデータベース内のインポートジョブを確認します。

  ```SQL
  SELECT * FROM information_schema.pipes;
  ```

  複数のインポートジョブを提出した場合、`PIPE_NAME`を使用して確認したいインポートジョブをフィルタリングできます。例えば：

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

#### インポートされたファイル情報の確認

[`information_schema.pipe_files`](../reference/information_schema/pipe_files.md)ビューを使用して、インポートされたファイル情報を確認できます。

```SQL
SELECT * FROM information_schema.pipe_files;
```

複数のインポートジョブを提出した場合、`PIPE_NAME`を使用して確認したいジョブのファイル情報をフィルタリングできます。例えば：

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

#### インポートジョブの管理

Pipe インポートジョブを作成した後、必要に応じてこれらのジョブを変更、一時停止または再開、削除、照会、および再インポートを試みることができます。詳細は [ALTER PIPE](../sql-reference/sql-statements/data-manipulation/ALTER_PIPE.md)、[SUSPEND or RESUME PIPE](../sql-reference/sql-statements/data-manipulation/SUSPEND_or_RESUME_PIPE.md)、[DROP PIPE](../sql-reference/sql-statements/data-manipulation/DROP_PIPE.md)、[SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)、[RETRY FILE](../sql-reference/sql-statements/data-manipulation/RETRY_FILE.md) を参照してください。
