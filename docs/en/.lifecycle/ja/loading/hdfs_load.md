---
displayed_sidebar: "Japanese"
---

# HDFSからデータをロードする

StarRocksでは、次のオプションを使用してHDFSからデータをロードすることができます。

- [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)+[`FILES()`](../sql-reference/sql-functions/table-functions/files.md)を使用した同期ロード
- [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用した非同期ロード
- [Pipe](../sql-reference/sql-statements/data-manipulation/CREATE_PIPE.md)を使用した連続非同期ロード

これらのオプションは、それぞれ独自の利点を持っており、以下のセクションで詳細に説明されています。

ほとんどの場合、INSERT+`FILES()`メソッドを使用することをおすすめします。このメソッドは非常に使いやすく、ParquetおよびORCファイル形式のみをサポートしています。

ただし、INSERT+`FILES()`メソッドは現在、ParquetおよびORCファイル形式のみをサポートしています。したがって、CSVなどの他のファイル形式のデータをロードする必要がある場合や、データのロード中にDELETEなどのデータ変更を行う必要がある場合は、Broker Loadを使用することができます。

合計データ量が非常に大きい（たとえば100 GB以上、または1 TB以上）場合は、Pipeメソッドを使用することをおすすめします。Pipeは、ファイルの数やサイズに基づいてファイルを分割し、ロードジョブを小さな連続的なタスクに分割します。これにより、1つのファイルのエラーがロードジョブ全体に影響を与えることがなくなり、データエラーによる再試行の必要性を最小限に抑えることができます。

## 開始する前に

### ソースデータを準備する

StarRocksにロードするソースデータがHDFSクラスタに適切に格納されていることを確認してください。このトピックでは、HDFSから`/user/amber/user_behavior_ten_million_rows.parquet`をStarRocksにロードすることを想定しています。

### 権限を確認する

StarRocksテーブルにデータをロードするには、そのStarRocksテーブルに対してINSERT権限を持つユーザーとしてのみデータをロードすることができます。INSERT権限がない場合は、StarRocksクラスタに接続するために使用するユーザーにINSERT権限を付与するための[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)の手順に従ってください。

### 接続の詳細を収集する

HDFSクラスタへの接続には、シンプルな認証方法を使用することができます。シンプルな認証を使用するには、HDFSクラスタのNameNodeにアクセスするために使用できるアカウントのユーザー名とパスワードを収集する必要があります。

## INSERT+FILES()を使用する

このメソッドはv3.1以降で使用可能で、現在はParquetおよびORCファイル形式のみをサポートしています。

### INSERT+FILES()の利点

[`FILES()`](../sql-reference/sql-functions/table-functions/files.md)は、指定したパス関連のプロパティに基づいてクラウドストレージに保存されているファイルを読み取り、ファイル内のデータのテーブルスキーマを推測し、データをデータ行として返すことができます。

`FILES()`を使用すると、次のことができます。

- [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用してHDFSからデータを直接クエリする。
- [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)（CTAS）を使用してテーブルを作成し、データをロードする。
- [INSERT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して既存のテーブルにデータをロードする。

### 典型的な例

#### SELECTを使用してHDFSから直接クエリする

SELECT+`FILES()`を使用してHDFSから直接クエリすることで、テーブルの内容をプレビューすることができます。たとえば、次のようなことができます。

- データを保存せずにデータセットのプレビューを取得する。
- 最小値と最大値をクエリし、使用するデータ型を決定する。
- `NULL`値をチェックする。

次の例では、HDFSクラスタに保存されているデータファイル`/user/amber/user_behavior_ten_million_rows.parquet`をクエリしています。

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

システムは次のクエリ結果を返します。

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
> 上記の列名は、Parquetファイルによって提供される列名です。

#### CTASを使用してテーブルを作成し、データをロードする

これは前の例の続きです。前のクエリをCREATE TABLE AS SELECT（CTAS）でラップして、スキーマ推論を使用してテーブルの作成を自動化します。これにより、StarRocksはテーブルスキーマを推測し、必要なテーブルを作成し、データをテーブルにロードします。Parquet形式では、列名は必要ありません。

> **注意**
>
> スキーマ推論を使用する場合のCREATE TABLEの構文では、レプリカの数を設定することはできないため、テーブルを作成する前に設定してください。以下の例は、シングルレプリカのシステム用のものです。
>
> ```SQL
> ADMIN SET FRONTEND CONFIG ('default_replication_num' = "1");
> ```

データベースを作成し、それに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

CTASを使用してテーブルを作成し、データファイル`/user/amber/user_behavior_ten_million_rows.parquet`のデータをテーブルにロードします。

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

テーブルを作成した後、[DESCRIBE](../sql-reference/sql-statements/Utility/DESCRIBE.md)を使用してそのスキーマを表示できます。

```SQL
DESCRIBE user_behavior_inferred;
```

システムは次のクエリ結果を返します。

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

スキーマ推論で推測されたスキーマと手動で作成したスキーマを比較します。

- データ型
- NULL許容
- キーフィールド

実稼働環境では、宛先テーブルのスキーマをより制御し、クエリのパフォーマンスを向上させるために、テーブルスキーマを手動で指定することをおすすめします。

データが正常にロードされたことを確認するために、テーブルをクエリします。例：

```SQL
SELECT * from user_behavior_inferred LIMIT 3;
```

次のクエリ結果が返され、データが正常にロードされていることが示されます。

```Plaintext
+--------+--------+------------+--------------+---------------------+
| UserID | ItemID | CategoryID | BehaviorType | Timestamp           |
+--------+--------+------------+--------------+---------------------+
|     84 |  56257 |    1879194 | pv           | 2017-11-26 05:56:23 |
|     84 | 108021 |    2982027 | pv           | 2017-12-02 05:43:00 |
|     84 | 390657 |    1879194 | pv           | 2017-11-28 11:20:30 |
+--------+--------+------------+--------------+---------------------+
```

#### INSERTを使用して既存のテーブルにデータをロードする

挿入先のテーブルをカスタマイズする場合があります。たとえば、次のような場合です。

- 列のデータ型、NULL許容設定、デフォルト値を指定する。
- キータイプと列を指定する。
- データのパーティショニングとバケット化を指定する。

> **注意**
>
> 最も効率的なテーブル構造を作成するには、データの使用方法と列の内容についての知識が必要です。このトピックではテーブル設計について説明していません。テーブル設計についての情報については、[テーブルの種類](../table_design/StarRocks_table_design.md)を参照してください。

この例では、クエリのタイプとParquetファイルのデータに関する知識に基づいて、テーブルを作成します。Parquetファイルのデータに関する知識は、HDFSでファイルを直接クエリすることで得ることができます。

- HDFSのデータセットのクエリによって、`Timestamp`列には`datetime`データ型に一致するデータが含まれていることがわかりますので、次のDDLで列の型を指定します。
- HDFSのデータをクエリすることで、データセットに`NULL`値が含まれていないことがわかるため、DDLでは列をNULL許容に設定しません。
- 予想されるクエリタイプに基づいて、ソートキーとバケット化列を`UserID`列に設定します。このデータに対しては、使用用途によっては、ソートキーとして`UserID`の他に`ItemID`を使用するか、または`UserID`の代わりに`ItemID`を使用するかもしれません。

データベースを作成し、それに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

テーブルを手動で作成します（Parquetファイルからロードするテーブルと同じスキーマを持つことをおすすめします）。

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

テーブルを作成した後、INSERT INTO SELECT FROM FILES()でテーブルにデータをロードします。

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

ロードが完了した後、テーブルをクエリしてデータが正常にロードされたことを確認できます。例：

```SQL
SELECT * from user_behavior_declared LIMIT 3;
```

次のクエリ結果が返され、データが正常にロードされていることが示されます。

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

`information_schema.loads`ビューからINSERTジョブの進捗状況をクエリすることができます。この機能はv3.1以降でサポートされています。例：

```SQL
SELECT * FROM information_schema.loads ORDER BY JOB_ID DESC;
```

複数のロードジョブを送信した場合は、ジョブに関連付けられた`LABEL`でフィルタリングすることができます。例：

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

`loads`ビューで提供されるフィールドの詳細については、[Information Schema](../reference/information_schema/loads.md)を参照してください。

> **注意**
>
> INSERTは同期コマンドです。INSERTジョブがまだ実行中の場合は、別のセッションを開いて実行状態を確認する必要があります。

## Broker Loadを使用する

非同期のBroker Loadプロセスは、HDFSへの接続、データの取得、およびデータのStarRocksへの保存を処理します。

このメソッドはParquet、ORC、およびCSVファイル形式をサポートしています。

### Broker Loadの利点

- Broker Loadは、ロード中にETLやUPSERT、DELETEなどのデータ変換やデータ変更をサポートしています。
- Broker Loadはバックグラウンドで実行され、ジョブが継続するためにクライアントが接続し続ける必要はありません。
- Broker Loadは、デフォルトのタイムアウトが4時間に設定されている長時間実行ジョブに適しています。
- ParquetおよびORCファイル形式に加えて、Broker LoadはCSVファイルもサポートしています。

### データフロー

![Broker Loadのワークフロー](../assets/broker_load_how-to-work_en.png)

1. ユーザーがロードジョブを作成します。
2. フロントエンド（FE）がクエリプランを作成し、プランをバックエンドノード（BE）に配布します。
3. BEがソースからデータを取得し、データをStarRocksにロードします。

### 典型的な例

#### データベースとテーブルを作成する

データベースを作成し、それに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

手動でテーブルを作成します（HDFSからロードするParquetファイルと同じスキーマを持つことをおすすめします）。

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

次のコマンドを実行して、データファイル`/user/amber/user_behavior_ten_million_rows.parquet`からデータを取得し、`user_behavior`テーブルにロードするBroker Loadジョブを開始します。

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

このジョブには、次の4つのメインセクションがあります。

- `LABEL`：ジョブの状態をクエリする際に使用する文字列。
- `LOAD`宣言：ソースURI、ソースデータ形式、および宛先テーブル名。
- `BROKER`：ソースの接続詳細。
- `PROPERTIES`：タイムアウト値およびロードジョブに適用するその他のプロパティ。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### ロードの進捗状況を確認する

[SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)を使用してPipeジョブの進捗状況をクエリすることができます。

```SQL
SHOW PIPES;
```

複数のロードジョブを送信した場合は、ジョブに関連付けられた`NAME`でフィルタリングすることができます。例：

```SQL
SHOW PIPES WHERE NAME = "user_behavior_replica" \G
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

[`information_schema.pipes`](../reference/information_schema/pipes.md)ビューからPipeジョブの進捗状況をクエリすることもできます。

```SQL
SELECT * FROM information_schema.pipes;
```

複数のロードジョブを送信した場合は、ジョブに関連付けられた`PIPE_NAME`でフィルタリングすることができます。例：

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

[`information_schema.pipe_files`](../reference/information_schema/pipe_files.md)ビューから、ロードされたファイルのロードステータスをクエリすることができます。

```SQL
SELECT * FROM information_schema.pipe_files;
```

複数のロードジョブを送信した場合は、ジョブに関連付けられた`PIPE_NAME`でフィルタリングすることができます。例：
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
1 行がセットされました (0.02 秒)
```

#### パイプの管理

作成したパイプを変更、一時停止または再開、削除、または特定のデータファイルをロードし直すことができます。詳細については、[ALTER PIPE](../sql-reference/sql-statements/data-manipulation/ALTER_PIPE.md)、[SUSPEND or RESUME PIPE](../sql-reference/sql-statements/data-manipulation/SUSPEND_or_RESUME_PIPE.md)、[DROP PIPE](../sql-reference/sql-statements/data-manipulation/DROP_PIPE.md)、[SHOW PIPES](../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)、および[RETRY FILE](../sql-reference/sql-statements/data-manipulation/RETRY_FILE.md)を参照してください。
