---
displayed_sidebar: "日本語"
---

# HDFS からデータを読み込む

import LoadMethodIntro from '../assets/commonMarkdown/loadMethodIntro.md'

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks では、HDFS からデータを読み込むための以下のオプションを提供しています:

<LoadMethodIntro />

## 開始する前に

### ソースデータを準備する

StarRocks に読み込むソースデータが HDFS クラスターに適切に格納されていることを確認してください。このトピックでは、HDFS から StarRocks に `/user/amber/user_behavior_ten_million_rows.parquet` を読み込むことを想定しています。

### 権限を確認する

<InsertPrivNote />

### 接続の詳細を収集する

シンプル認証メソッドを使用して HDFS クラスターとの接続を確立することができます。シンプル認証を使用するには、HDFS クラスターの NameNode にアクセスできるアカウントのユーザー名とパスワードを収集する必要があります。

## INSERT+FILES() を使用する

このメソッドは v3.1 以降で利用可能であり、現在は Parquet および ORC ファイル形式のみをサポートしています。

### INSERT+FILES() の利点

[`FILES()`](../sql-reference/sql-functions/table-functions/files.md) は、指定したパス関連のプロパティに基づいてクラウドストレージに保存されたファイルを読み込み、ファイル内のデータのテーブルスキーマを推論し、そのデータをデータ行として返します。

`FILES()` を使用すると、次のことができます:

- [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して、HDFS からデータを直接クエリする。
- [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) (CTAS) を使用して、テーブルを作成しデータを読み込む。
- [INSERT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して、既存のテーブルにデータを読み込む。

### 典型的な例

#### SELECT を使用して直接 HDFS からクエリする

SELECT+`FILES()` を使用して、データセットの内容を事前にプレビューすることができます。例えば:

- データを保存せずにデータセットのプレビューを表示する。
- 最小値と最大値をクエリし、使用するデータ型を決定する。
- `NULL` 値をチェックする。

次の例は、HDFS クラスターに保存されているデータファイル `/user/amber/user_behavior_ten_million_rows.parquet` をクエリします:

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

システムは以下のクエリ結果を返します:

```plaintext
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
> 上記の列名は、Parquet ファイルによって提供されるものです。

#### CTAS を使用してテーブルを作成しデータを読み込む

これは以前の例の継続です。前のクエリを CREATE TABLE AS SELECT (CTAS) でラップして、スキーマ推論を使用して自動的にテーブルを作成し、その後データをテーブルに読み込みます。`FILES()` テーブル関数を Parquet ファイルとともに使用すると、テーブルの作成に必要な列名と型を指定する必要はありません。

> **注意**
>
> スキーマ推論を使用する場合の CREATE TABLE の構文では、レプリカの数を設定することはできません。そのため、テーブルを作成する前にレプリカの数を設定してください。以下の例は単一のレプリカを持つシステム向けです: 
>
> ```SQL
> ADMIN SET FRONTEND CONFIG ('default_replication_num' = "1");
> ```

データベースを作成して、それを使用します:

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

CTAS を使用してテーブルを作成し、データファイル `/user/amber/user_behavior_ten_million_rows.parquet` のデータをテーブルに読み込みます:

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

テーブルを作成した後、そのスキーマを [DESCRIBE](../sql-reference/sql-statements/Utility/DESCRIBE.md) を使用して表示できます:

```SQL
DESCRIBE user_behavior_inferred;
```

システムは以下のクエリ結果を返します:

```plaintext
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

スキーマ推論で推論されたスキーマと手動で作成したスキーマを比較します:

- データ型
- NULL 許容度
- キーフィールド

宛先テーブルのスキーマをよりよく制御し、より高速なクエリの実行を可能にするために、本番環境ではテーブルスキーマを手動で指定することを推奨します。

データがそれに正常に読み込まれたことを確認するためにテーブルをクエリします。例:

```SQL
SELECT * from user_behavior_inferred LIMIT 3;
```

その結果、次のクエリ結果が返され、データが正常に読み込まれていることが示されます:

```plaintext
+--------+--------+------------+--------------+---------------------+
| UserID | ItemID | CategoryID | BehaviorType | Timestamp           |
+--------+--------+------------+--------------+---------------------+
|     84 |  56257 |    1879194 | pv           | 2017-11-26 05:56:23 |
|     84 | 108021 |    2982027 | pv           | 2017-12-02 05:43:00 |
|     84 | 390657 |    1879194 | pv           + 2017-11-28 11:20:30 |
+--------+--------+------------+--------------+---------------------+
```

#### INSERT を使用して既存のテーブルにデータを読み込む

挿入するテーブルのカスタマイズが必要な場合、例えば:

- カラムのデータ型、NULL 設定、デフォルト値を設定
- キータイプと列
- データのパーティショニングとバケット化

> **注意**
>
> 最も効率的なテーブル構造を作成するには、データの使用方法と列の内容を知っている必要があります。このトピックではテーブルデザインについては取り上げていません。テーブルデザインに関する情報については、[テーブルタイプ](../table_design/StarRocks_table_design.md)を参照してください。

この例では、テーブルがどのようにクエリされるかと Parquet ファイル内のデータについての知識に基づいてテーブルを作成します。HDFS のデータを直接クエリすることで、例えば次のようなことができます:

- HDFS のデータセットのクエリにより、`Timestamp` 列に `datetime` データ型に一致するデータが含まれていることがわかりますので、以下の DDL で列の型が指定されています。
- HDFS のデータをクエリすることで、データセットに `NULL` 値が含まれていないことがわかりますので、DDL は列を NULL 許容に設定していません。
- 期待されるクエリタイプに関する知識に基づいて、ソートキーやバケット化された列が列 `UserID` に設定されています。データに対するあなたのユースケースはこれとは異なる場合がありますので、`UserID` の代わりにまたは追加で `ItemID` をソートキーとして使用する可能性があります。

データベースを作成して、それを使用します:

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

手動でテーブルを作成します (Parquet ファイルが HDFS から読み込むテーブルと同じスキーマであることを推奨します):

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

テーブルを作成した後、INSERT INTO SELECT FROM FILES() を使用してデータを読み込むことができます:

```SQL
INSERT INTO user_behavior_declared
```sql
SELECT * FROM FILES
(
    "path" = "hdfs://<hdfs_ip>:<hdfs_port>/user/amber/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

読み込みが完了したら、テーブルをクエリしてデータが正常に読み込まれたかどうかを確認できます。例：

```sql
SELECT * from user_behavior_declared LIMIT 3;
```

以下のクエリ結果が返され、データが正常に読み込まれたことが示されます：

```plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
|    107 | 1568743 |    4476428 | pv           | 2017-11-25 14:29:53 |
|    107 |  470767 |    1020087 | pv           | 2017-11-25 14:32:31 |
|    107 |  358238 |    1817004 | pv           | 2017-11-25 14:43:23 |
+--------+---------+------------+--------------+---------------------+
```

#### 読み込みの進行状況を確認する

`information_schema.loads` ビューから INSERT ジョブの進行状況をクエリできます。この機能はv3.1以降でサポートされています。例：

```sql
SELECT * FROM information_schema.loads ORDER BY JOB_ID DESC;
```

複数の読み込みジョブを提出した場合、ジョブに関連付けられた `LABEL` でフィルターできます。例：

```sql
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

`loads` ビューで提供されるフィールドに関する情報については、[Information Schema](../reference/information_schema/loads.md)を参照してください。

> **注意**
>
> INSERT は同期コマンドです。INSERT ジョブがまだ実行中の場合は、別のセッションを開いてその実行状況を確認する必要があります。

## Broker Load の使用

非同期の Broker Load プロセスは、HDFSへの接続、データの取得、およびStarRocksへのデータの格納を処理します。

この方法は、Parquet、ORC、およびCSVファイル形式をサポートしています。

### Broker Loadの利点

- Broker Loadは、読み込み中にUPSERTやDELETEなどの[データ変換](../loading/Etl_in_loading.md)や[データ変更](../loading/Load_to_Primary_Key_tables.md)のサポートを提供します。
- Broker Loadはバックグラウンドで実行され、クライアントが接続を続けてジョブを続行する必要はありません。
- デフォルトのタイムアウト時間が4時間のため、Broker Loadは長時間実行されるジョブに適しています。
- ParquetおよびORCファイル形式に加えて、Broker LoadはCSVファイルもサポートしています。

### データの流れ

![Workflow of Broker Load](../assets/broker_load_how-to-work_en.png)

1. ユーザーが読み込みジョブを作成します。
2. フロントエンド（FE）がクエリプランを作成し、プランをバックエンドノード（BE）に配布します。
3. バックエンドノード（BE）がソースからデータを取得し、データをStarRocksに読み込みます。

### 典型的な例

テーブルを作成し、HDFSからデータファイル `/user/amber/user_behavior_ten_million_rows.parquet` を読み込み、データ読み込みの進行状況と成功を確認します。

#### データベースとテーブルの作成

データベースを作成し、使用します。

```sql
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

テーブルを手動で作成します（HDFSから読み込みたいParquetファイルと同じスキーマを持つことをお勧めします）。

```sql
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

次のコマンドを実行して、データファイル `/user/amber/user_behavior_ten_million_rows.parquet` から `user_behavior` テーブルにデータを読み込むBroker Loadジョブを開始します。

```sql
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

このジョブには、以下のメインセクションがあります：

- `LABEL`：読み込みジョブの状態をクエリする際に使用される文字列。
- `LOAD` 宣言：ソースURI、ソースデータ形式、および宛先テーブル名。
- `BROKER`：ソースの接続の詳細。
- `PROPERTIES`：タイムアウト値とジョブに適用されるその他のプロパティ。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### ロードの進行状況をチェックする

`information_schema.loads` ビューから Broker Load ジョブの進行状況をクエリできます。この機能はv3.1以降でサポートされています。

```sql
SELECT * FROM information_schema.loads;
```

`loads` ビューで提供されるフィールドに関する情報については、[Information Schema](../reference/information_schema/loads.md)を参照してください。

複数の読み込みジョブを提出した場合、ジョブに関連付けられた `LABEL` でフィルターできます。例：

```sql
SELECT * FROM information_schema.loads WHERE LABEL = 'user_behavior';
```

以下の出力には、`user_behavior` のロードジョブの2つのエントリがあります：

- 最初のレコードは `CANCELLED` の状態を示しており、`ERROR_MSG` にステータスが `listPath failed` で失敗したことが表示されます。
- 2番目のレコードは `FINISHED` の状態を示しており、ジョブが成功したことを示しています。

```plaintext
JOB_ID|LABEL                                      |DATABASE_NAME|STATE    |PROGRESS           |TYPE  |PRIORITY|SCAN_ROWS|FILTERED_ROWS|UNSELECTED_ROWS|SINK_ROWS|ETL_INFO|TASK_INFO                                           |CREATE_TIME        |ETL_START_TIME     |ETL_FINISH_TIME    |LOAD_START_TIME    |LOAD_FINISH_TIME   |JOB_DETAILS                                                                                                                                                                                                                                                    |ERROR_MSG                             |TRACKING_URL|TRACKING_SQL|REJECTED_RECORD_PATH|
------+-------------------------------------------+-------------+---------+-------------------+------+--------+---------+-------------+---------------+---------+--------+----------------------------------------------------+-------------------+-------------------+-------------------+-------------------+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+------------+--------------------+
```SQL
SELECT * from user_behavior LIMIT 3;
```

The following query result is returned, indicating that the data has been successfully loaded:

```Plaintext
+--------+---------+------------+--------------+---------------------+
| UserID | ItemID  | CategoryID | BehaviorType | Timestamp           |
+--------+---------+------------+--------------+---------------------+
|    142 | 2869980 |    2939262 | pv           | 2017-11-25 03:43:22 |
|    142 | 2522236 |    1669167 | pv           | 2017-11-25 15:14:12 |
|    142 | 3031639 |    3607361 | pv           | 2017-11-25 15:19:25 |
+--------+---------+------------+--------------+---------------------+
```

## パイプの使用

v3.2から、StarRocksは現在ParquetおよびORCファイル形式のみをサポートしているパイプローディング方法を提供しています。

### パイプの利点

パイプは連続データローディングと大規模データローディングに理想的です。

- **マイクロバッチでの大規模データローディングは、データエラーによるリトライのコストを削減します。**

  パイプのサポートにより、StarRocksは、合計データ量が膨大な大量のデータファイルを効率的にロードすることができます。パイプはファイルをその数やサイズに基づいて自動的に分割し、ロードジョブを小さな連続的なタスクに分割します。このアプローチにより、1つのファイルのエラーがロードジョブ全体に影響を与えることはありません。パイプは各ファイルのロードステータスを記録するため、エラーを含むファイルを簡単に特定して修正することができます。データエラーに起因するリトライの必要性を最小限に抑えることで、このアプローチはコストを削減するのに役立ちます。

- **連続データローディングは、人員を減らすのに役立ちます。**

  パイプにより、新しいまたは更新されたデータファイルを特定の場所に書き込み、これらのファイルから新しいデータをStarRocksに継続的にロードできます。`"AUTO_INGEST" = "TRUE"`を指定してパイプジョブを作成した後、それは指定されたパスに保存されているデータファイルの変更を常に監視し、データファイルから新しいまたは更新されたデータを自動的にStarRocksのテーブルにロードします。

さらに、パイプは重複したデータローディングを防ぐためのファイルのユニーク性のチェックを実行します。ローディングプロセス中に、パイプはファイル名とダイジェストに基づいて各データファイルのユニーク性をチェックします。特定のファイル名とダイジェストを持つファイルがすでにパイプジョブによって処理された場合、パイプジョブは同じファイル名とダイジェストを持つすべての後続するファイルをスキップします。HDFSはファイルダイジェストとして`LastModifiedTime`を使用します。

各データファイルのロードステータスは、`information_schema.pipe_files`ビューに記録および保存されます。ビューに関連するパイプジョブが削除された後、そのジョブでロードされたファイルに関するレコードも削除されます。

### データフロー

![パイプのデータフロー](../assets/pipe_data_flow.png)

### パイプとINSERT+FILES()の違い

パイプジョブは、各データファイルのサイズと行数に基づいて1つ以上のトランザクションに分割されます。ユーザーはロードプロセス中に中間結果をクエリできます。一方、INSERT+`FILES()`ジョブは単一のトランザクションとして処理され、ユーザーはロードプロセス中にデータを表示することはできません。

### ファイルのローディング順序

各パイプジョブについて、StarRocksはファイルキューを維持し、ファイルをマイクロバッチとして取得およびロードします。パイプは、ファイルがアップロードされた順序と同じ順序でロードされることを保証しません。そのため、新しいデータが古いデータよりも先にロードされることがあります。

### 典型的な例

#### データベースおよびテーブルの作成

データベースを作成し、それに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

手動でテーブルを作成します（HDFSからロードしたいParquetファイルと同じスキーマを持つことをお勧めします）。

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

#### パイプジョブの開始

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

このジョブには4つの主要なセクションがあります。

- `pipe_name`：パイプの名前。パイプ名は、パイプが属するデータベース内で一意である必要があります。
- `INSERT_SQL`：指定されたソースデータファイルから宛先テーブルにデータをロードするために使用されるINSERT INTO SELECT FROM FILESステートメント。
- `PROPERTIES`：パイプの実行方法を指定する一連のオプションパラメータ。これには、`AUTO_INGEST`、`POLL_INTERVAL`、`BATCH_SIZE`、`BATCH_FILES`が含まれます。これらのプロパティを`"key" = "value"`形式で指定します。

詳細な構文とパラメータの説明については、[CREATE PIPE](../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)を参照してください。

#### ロード進捗の確認

- [SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)を使用してパイプジョブの進捗状況をクエリします。

  ```SQL
  SHOW PIPES;
  ```

  複数のロードジョブを提出した場合、ジョブに関連する`NAME`でフィルタリングできます。例：

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

  複数のロードジョブを提出した場合、ジョブに関連する`PIPE_NAME`でフィルタリングできます。例：

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

#### ファイルステータスを確認する

[`information_schema.pipe_files`](../reference/information_schema/pipe_files.md) ビューからロードされたファイルのロードステータスをクエリできます。

```SQL
SELECT * FROM information_schema.pipe_files;
```

複数のロードジョブを提出した場合は、ジョブに関連付けられた `PIPE_NAME` でフィルタリングできます。例:

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

#### パイプの管理

作成したパイプを変更、一時停止または再開、削除、またはクエリし、特定のデータファイルのロードを再試行できます。詳細については、[ALTER PIPE](../sql-reference/sql-statements/data-manipulation/ALTER_PIPE.md), [SUSPEND or RESUME PIPE](../sql-reference/sql-statements/data-manipulation/SUSPEND_or_RESUME_PIPE.md), [DROP PIPE](../sql-reference/sql-statements/data-manipulation/DROP_PIPE.md), [SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md), および [RETRY FILE](../sql-reference/sql-statements/data-manipulation/RETRY_FILE.md) を参照してください。