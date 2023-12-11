---
displayed_sidebar: "Japanese"
---

# HDFSからデータをロード

import LoadMethodIntro from '../assets/commonMarkdown/loadMethodIntro.md'

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksでは、以下のオプションを提供していますHDFSからデータをロードするための：

<LoadMethodIntro />

## 開始する前に

### ソースデータを整える

StarRocksにロードしたいソースデータがHDFSクラスターに正しく格納されていることを確認してください。このトピックでは、HDFSからStarRocksに`/user/amber/user_behavior_ten_million_rows.parquet`をロードすることを前提としています。

### 権限を確認する

<InsertPrivNote />

### 接続詳細を収集する

簡単な認証メソッドを使用してHDFSクラスターとの接続を確立できます。簡単な認証を使用するには、HDFSクラスターのNameNodeにアクセスするために使用できるアカウントのユーザー名とパスワードを収集する必要があります。

## INSERT+FILES()を使用する

この方法はv3.1以降で利用可能で、現在はParquetおよびORCファイル形式のみをサポートしています。

### INSERT+FILES()の利点

[`FILES()`](../sql-reference/sql-functions/table-functions/files.md)は、指定したパス関連のプロパティに基づいてクラウドストレージに保存されているファイルを読み込み、ファイル内のデータのテーブルスキーマを推論し、その後、ファイルからデータをデータ行として返します。

`FILES()`を使用すると、次のことができます：

- [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、直接HDFSからデータをクエリする。
- [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)（CTAS）を使用して、テーブルを作成してデータをロードする。
- [INSERT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、既存のテーブルにデータをロードする。

### 典型的な例

#### SELECTを使用して直接HDFSからクエリする

SELECT+`FILES()`を使用して直接HDFSからクエリを行うと、テーブルを作成する前にデータセットの内容をプレビューできます。例：

- データを格納せずにデータセットのプレビューを取得する。
- 最小値と最大値をクエリしてデータ型を決定する。
- `NULL`値を確認する。

次の例では、HDFSクラスターに格納されているデータファイル`/user/amber/user_behavior_ten_million_rows.parquet`をクエリします：

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

システムは次のクエリ結果を返します：

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
| 543711 |  829192 |    2355072 | pv           | 2017-11-27 08:22:37 |
| 543711 | 2056618 |    3645362 | pv           | 2017-11-27 10:16:46 |
| 543711 | 1165492 |    3645362 | pv           | 2017-11-27 10:17:00 |
+--------+---------+------------+--------------+---------------------+
```

> **注意**
>
> 上記で返されたカラム名はParquetファイルによって提供されていることに注意してください。

#### CTASを使用してテーブルを作成してデータをロードする

これは前述の例の続きです。前のクエリをCREATE TABLE AS SELECT（CTAS）でラップすると、スキーマ推論を使用してテーブル作成を自動化し、希望するテーブルを作成し、データをロードします。Parquetファイルを使用する場合、`FILES()`テーブル関数を使用すると列名と型を指定する必要はありません。

> **注意**
>
> スキーマ推論を使用する際のCREATE TABLEの構文では、レプリカ数を設定することはできません。したがって、テーブルを作成する前に設定してください。以下の例は単一のレプリカを持つシステム用です：

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

テーブルを作成した後、[DESCRIBE](../sql-reference/sql-statements/Utility/DESCRIBE.md)を使用してそのスキーマを表示できます：

```SQL
DESCRIBE user_behavior_inferred;
```

システムは次のクエリ結果を返します：

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

スキーマ推論されたスキーマと手動で作成されたスキーマを比較してください：

- データ型
- NULL可能
- キーフィールド

宛先テーブルのスキーマをより良く制御し、クエリのパフォーマンスを向上させるために、本番環境ではテーブルスキーマを手動で指定することをお勧めします。

データが正常にロードされたことを確認するために、テーブルをクエリしてください。例：

```SQL
SELECT * from user_behavior_inferred LIMIT 3;
```

次のクエリ結果が返され、データが正常にロードされたことが示されます：

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

挿入先のテーブルをカスタマイズしたい場合は、たとえば以下を指定する：

- カラムのデータ型、NULL可能な設定、またはデフォルト値
- キータイプとカラム
- データのパーティショニングとバケティング

> **注意**
>
> 最も効率的なテーブル構造を作成するには、データの使用方法と列の内容を知る必要があります。本トピックではテーブル設計については説明していません。テーブル設計については、[Table types](../table_design/StarRocks_table_design.md)を参照してください。

この例では、HDFSのデータセットのクエリにより、`Timestamp`列が`datetime`データ型に一致するデータを含んでいることがわかるため、以下のDDLでカラムタイプが指定されています。
- HDFS内のデータをクエリすることで、データセットに`NULL`値がないことがわかりますので、DDLではどのカラムもnullableとしていません。
- 期待されるクエリ種類の知識に基づいて、ソートキーとバケティングカラムは`UserID`に設定されています。このデータについては異なる場合がありますので、ソートキーとして`UserID`の他に`ItemID`を使用することを決定するかもしれません。

データベースを作成し、それに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

手動でテーブルを作成します（Parquetファイルと同じスキーマを持つことをお勧めします）：

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

テーブルを作成した後、INSERT INTO SELECT FROM FILES()でロードできます：

```SQL
INSERT INTO user_behavior_declared
```SQL
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

ロードが完了したら、テーブルをクエリしてデータが正常にロードされたことを確認できます。例：

```SQL
SELECT * from user_behavior_declared LIMIT 3;
```

次のクエリ結果が返され、データが正常にロードされたことが示されます：

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
|    107 | 1568743 |    4476428 | pv           | 2017-11-25 14:29:53 |
|    107 |  470767 |    1020087 | pv           | 2017-11-25 14:32:31 |
|    107 |  358238 |    1817004 | pv           | 2017-11-25 14:43:23 |
+--------+---------+------------+--------------+---------------------+
```

#### ロードの進捗状況を確認する

`information_schema.loads` ビューから INSERT ジョブの進捗状況をクエリできます。この機能は v3.1 以降でサポートされています。例：

```SQL
SELECT * FROM information_schema.loads ORDER BY JOB_ID DESC;
```

複数のロードジョブを送信した場合は、ジョブに関連付けられた `LABEL` でフィルタリングできます。例：

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

`loads` ビューで提供されるフィールドの情報については、「[Information Schema](../reference/information_schema/loads.md)」を参照してください。

> **注意**
>
> INSERT は同期コマンドです。INSERT ジョブがまだ実行中の場合は、別のセッションを開いて実行ステータスを確認する必要があります。

## ブローカーロードを使用する

非同期のブローカーロードプロセスは、HDFSへの接続、データの取得、およびデータのStarRocksへの保存を処理します。

この方法は、Parquet、ORC、およびCSVファイル形式をサポートしています。

### ブローカーロードの利点

- ブローカーロードは[データ変換](../loading/Etl_in_loading.md)および[データ変更](../loading/Load_to_Primary_Key_tables.md)をサポートしており、ローディング中にUPSERTやDELETEなどの操作が可能です。
- ブローカーロードはバックグラウンドで実行され、ジョブが継続するためにクライアントが接続を維持する必要はありません。
- ブローカーロードはデフォルトのタイムアウトが4時間に及ぶ長時間のジョブに適しています。
- ParquetおよびORCファイル形式に加えて、ブローカーロードはCSVファイルもサポートしています。

### データフロー

![ブローカーロードのワークフロー](../assets/broker_load_how-to-work_en.png)

1. ユーザーがロードジョブを作成します。
2. フロントエンド（FE）がクエリプランを作成し、プランをバックエンドノード（BE）に配信します。
3. BEはソースからデータを取得し、データをStarRocksにロードします。

### 典型的な例

テーブルを作成し、HDFSからデータファイル `/user/amber/user_behavior_ten_million_rows.parquet` を取得するロードプロセスを開始し、データのロードの進行および成功を確認します。

#### データベースおよびテーブルの作成

データベースを作成して、それに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

手動でテーブルを作成します（HDFSからロードしたいParquetファイルと同じスキーマを持つことをお勧めします）：

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

#### ブローカーロードの開始

次のコマンドを実行して、データファイル `/user/amber/user_behavior_ten_million_rows.parquet` から `user_behavior` テーブルにデータをロードするブローカーロードジョブを開始します：

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

- `LABEL`：ロードジョブの状態をクエリする際に使用する文字列です。
- `LOAD` 宣言：ソースURI、ソースデータ形式、および宛先テーブル名です。
- `BROKER`：ソースの接続詳細です。
- `PROPERTIES`：タイムアウト値およびロードジョブに適用する他のプロパティです。

詳しい構文およびパラメータの説明については、「[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)」を参照してください。

#### ロードの進捗状況を確認する

`information_schema.loads` ビューからブローカーロードジョブの進捗状況をクエリできます。この機能は v3.1 以降でサポートされています。

```SQL
SELECT * FROM information_schema.loads;
```

`LOADS` ビューで提供されるフィールドの情報については、「[Information Schema](../reference/information_schema/loads.md)」を参照してください。

複数のロードジョブを送信した場合は、ジョブに関連付けられた `LABEL` でフィルタリングできます。例：

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'user_behavior';
```

以下の出力には、ロードジョブ `user_behavior` についての2つのエントリがあります：

- 最初のレコードは `CANCELLED` の状態を示しています。`ERROR_MSG` にスクロールすると、`listPath failed` によってジョブが失敗したことがわかります。
- 2番目のレコードは `FINISHED` の状態を示しており、ジョブが成功したことを意味します。

```Plaintext
JOB_ID|LABEL                                      |DATABASE_NAME|STATE    |PROGRESS           |TYPE  |PRIORITY|SCAN_ROWS|FILTERED_ROWS|UNSELECTED_ROWS|SINK_ROWS|ETL_INFO|TASK_INFO                                           |CREATE_TIME        |ETL_START_TIME     |ETL_FINISH_TIME    |LOAD_START_TIME    |LOAD_FINISH_TIME   |JOB_DETAILS                                                                                                                                                                                                                                                    |ERROR_MSG                             |TRACKING_URL|TRACKING_SQL|REJECTED_RECORD_PATH|
------+-------------------------------------------+-------------+---------+-------------------+------+--------+---------+-------------+---------------+---------+--------+----------------------------------------------------+-------------------+-------------------+-------------------+-------------------+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+------------+--------------------+
```
```plaintext
+--------+---------+------------+--------------+---------------------+
| ユーザーID | アイテムID | カテゴリID | 行動タイプ | タイムスタンプ         |
+--------+---------+------------+--------------+---------------------+
|    142 | 2869980 |    2939262 | pv           | 2017-11-25 03:43:22 |
|    142 | 2522236 |    1669167 | pv           | 2017-11-25 15:14:12 |
|    142 | 3031639 |    3607361 | pv           | 2017-11-25 15:19:25 |
+--------+---------+------------+--------------+---------------------+
```

## パイプの使用

v3.2から、StarRocksでは現在ParquetおよびORCファイルフォーマットのみをサポートしているパイプロード方法が提供されます。

### パイプの利点

パイプは、連続的なデータロードと大規模なデータロードに適しています。

- **マイクロバッチによる大規模なデータロードは、データエラーによるリトライコストを減らすのに役立ちます。**

  パイプのサポートを受けて、StarRocksは合計データ量が多い多数のデータファイルを効率的にロードできます。パイプは、ファイルの数やサイズに基づいてファイルを自動的に分割し、ロードジョブを小さな順次のタスクに分割します。このアプローチにより、1つのファイルのエラーがロードジョブ全体に影響を与えることはありません。各ファイルのロード状態はパイプによって記録されるため、エラーを含むファイルを簡単に特定して修正できます。データエラーによるリトライの必要性を最小限に抑えることで、このアプローチはコストを削減します。

- **連続的なデータロードは人手を減らすのに役立ちます。**

  パイプによって、新しいまたは更新されたデータファイルを特定の場所に書き込み、それらのファイルから新しいデータをStarRocksに連続的にロードできます。`"AUTO_INGEST" = "TRUE"`を指定してパイプジョブを作成した後、指定されたパスに格納されているデータファイルの変更を常に監視し、そのファイルから新しいまたは更新されたデータを自動的にロードできます。

さらに、パイプは重複データロードを防ぐためにファイルの一意性チェックを実行します。ロードプロセス中、パイプはファイル名とダイジェストに基づいて、各データファイルの一意性をチェックします。特定のファイル名とダイジェストを持つファイルがすでにパイプジョブによって処理されている場合、そのパイプジョブは同じファイル名とダイジェストを持つすべての後続ファイルをスキップします。HDFSはファイルのダイジェストとして`LastModifiedTime`を使用します。

各データファイルのロード状態は`information_schema.pipe_files`ビューに記録され、ビューに関連付けられたパイプジョブが削除されると、そのジョブでロードされたファイルに関するレコードも削除されます。

### データフロー

![Pipe data flow](../assets/pipe_data_flow.png)

### パイプとINSERT+FILES()の違い

パイプジョブは、各データファイルのサイズと行数に基づいて1つ以上のトランザクションに分割されます。ユーザーはロードプロセス中に中間結果をクエリできます。一方、INSERT+`FILES()`ジョブは単一のトランザクションとして処理され、ユーザーはロードプロセス中にデータを表示できません。

### ファイルのロードシーケンス

各パイプジョブに対して、StarRocksはファイルキューを維持し、データファイルをマイクロバッチとして取得してロードします。パイプは、ファイルがアップロードされた順序でロードされることを保証しません。したがって、新しいデータが古いデータよりも先にロードされる可能性があります。

### 典型的な例

#### データベースとテーブルを作成

データベースを作成してそれに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

手動でテーブルを作成します（HDFSからロードしたいParquetファイルと同じスキーマを持つテーブルを作成することをお勧めします）。

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

#### パイプジョブを開始

次のコマンドを実行して、データファイル`/user/amber/user_behavior_ten_million_rows.parquet`から`user_behavior_replica`テーブルにデータをロードするパイプジョブを開始します。

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

このジョブには4つのメインセクションがあります。

- `pipe_name`：パイプの名前。パイプ名は、そのパイプが属するデータベース内で一意である必要があります。
- `INSERT_SQL`：指定したソースデータファイルから宛先テーブルにデータをロードするために使用されるINSERT INTO SELECT FROM FILESステートメント。
- `PROPERTIES`：パイプの実行方法を指定する一連のオプションパラメータ。これには、`AUTO_INGEST`、`POLL_INTERVAL`、`BATCH_SIZE`、`BATCH_FILES`などが含まれます。これらのプロパティは、`"key" = "value"`フォーマットで指定します。

詳細な構文とパラメータの説明については、[CREATE PIPE](../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)を参照してください。

#### ロード進捗状況の確認

- [SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)を使用してパイプジョブの進捗状況をクエリします。

  ```SQL
  SHOW PIPES;
  ```

  複数のロードジョブを提出している場合は、ジョブに関連する`NAME`でフィルタリングできます。例：

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

- [`information_schema.pipes`](../reference/information_schema/pipes.md)ビューからパイプジョブの進捗状況をクエリします。

  ```SQL
  SELECT * FROM information_schema.pipes;
  ```

  複数のロードジョブを提出している場合は、ジョブに関連する`PIPE_NAME`でフィルタリングできます。例：

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

#### ファイルのステータスを確認する

[`information_schema.pipe_files`](../reference/information_schema/pipe_files.md) ビューからロードされたファイルのロードステータスをクエリできます。

```SQL
SELECT * FROM information_schema.pipe_files;
```

複数のロードジョブを提出した場合は、ジョブに関連付けられた `PIPE_NAME` でフィルタリングできます。例：

```SQL
SELECT * FROM information_schema.pipe_files WHERE pipe_name = 'user_behavior_replica' \G
*************************** 1. 行 ***************************
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
1 行 in set (0.02 秒)
```

#### パイプの管理

作成したパイプを変更、中断、再開、削除、またはクエリし、特定のデータファイルのロードを再試行できます。詳細については、[ALTER PIPE](../sql-reference/sql-statements/data-manipulation/ALTER_PIPE.md), [SUSPEND or RESUME PIPE](../sql-reference/sql-statements/data-manipulation/SUSPEND_or_RESUME_PIPE.md), [DROP PIPE](../sql-reference/sql-statements/data-manipulation/DROP_PIPE.md), [SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md), および [RETRY FILE](../sql-reference/sql-statements/data-manipulation/RETRY_FILE.md) を参照してください。