---
displayed_sidebar: "Japanese"
---

# 情報スキーマ

StarRocksの`information_schema`は、各StarRocksインスタンス内にあるデータベースです。`information_schema`には、StarRocksインスタンスが保持するすべてのオブジェクトの詳細なメタデータ情報を格納するいくつかの読み取り専用のシステム定義テーブルが含まれています。

v3.2以降、StarRocksは`information_schema`を介して外部カタログメタデータを表示することがサポートされています。

## Information Schemaを介したメタデータの表示

`information_schema`内のテーブルのコンテンツをクエリすることで、StarRocksインスタンス内のメタデータ情報を表示できます。

次の例では、テーブル`tables`をクエリして、StarRocks内のテーブル`sr_member`に関するメタデータ情報を表示しています。

```Plain
mysql> SELECT * FROM information_schema.tables WHERE TABLE_NAME like 'sr_member'\G
*************************** 1. row ***************************
  TABLE_CATALOG: def
   TABLE_SCHEMA: sr_hub
     TABLE_NAME: sr_member
     TABLE_TYPE: BASE TABLE
         ENGINE: StarRocks
        VERSION: NULL
     ROW_FORMAT: NULL
     TABLE_ROWS: 6
 AVG_ROW_LENGTH: 542
    DATA_LENGTH: 3255
MAX_DATA_LENGTH: NULL
   INDEX_LENGTH: NULL
      DATA_FREE: NULL
 AUTO_INCREMENT: NULL
    CREATE_TIME: 2022-11-17 14:32:30
    UPDATE_TIME: 2022-11-17 14:32:55
     CHECK_TIME: NULL
TABLE_COLLATION: utf8_general_ci
       CHECKSUM: NULL
 CREATE_OPTIONS: NULL
  TABLE_COMMENT: OLAP
1 row in set (1.04 sec)
```

## Information Schemaテーブル

StarRocksは、`information_schema`内で提供されるテーブル`tables`, `tables_config`, `load_tracking_logs`のメタデータ情報を最適化しており、v3.1以降では`loads`テーブルを提供しています。

| **Information Schema table name** | **Description**                                              |
| --------------------------------- | ------------------------------------------------------------ |
| [tables](#tables)                            | テーブルの一般的なメタデータ情報を提供します。             |
| [tables_config](#tables_config)                     | StarRocks固有の追加のテーブルメタデータ情報を提供します。  |
| [load_tracking_logs](#load_tracking_logs)                | ロードジョブのエラー情報（あれば）を提供します。              |
| [loads](#loads)                             | v3.1以降でサポートされているロードジョブの結果を提供します。現在、このテーブルからは[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)と[Insert](../sql-reference/sql-statements/data-manipulation/INSERT.md)ジョブの結果のみを表示できます。                 |

### loads

`loads`で提供されるフィールドは次のとおりです：

| **Field**            | **Description**                                              |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | StarRocksがロードジョブを識別するために割り当てた一意のIDです。  |
| LABEL                | ロードジョブのラベル。                                        |
| DATABASE_NAME        | ディスティネーションのStarRocksテーブルが属するデータベースの名前です。 |
| STATE                | ロードジョブの状態。有効な値：<ul><li>`PENDING`：ロードジョブが作成されました。</li><li>`QUEUEING`：ロードジョブがスケジュールされるのを待っています。</li><li>`LOADING`：ロードジョブが実行中です。</li><li>`PREPARED`：トランザクションがコミットされました。</li><li>`FINISHED`：ロードジョブが成功しました。</li><li>`CANCELLED`：ロードジョブが失敗しました。</li></ul>詳細については、[非同期読み込み](../loading/Loading_intro.md#asynchronous-loading)を参照してください。 |
| PROGRESS             | ロードジョブのETLステージとLOADINGステージの進捗状況です。      |
| TYPE                 | ロードジョブの種類。Broker Loadの場合、返り値は`BROKER`です。INSERTの場合、返り値は`INSERT`です。 |
| PRIORITY             | ロードジョブの優先度。有効な値：`HIGHEST`、`HIGH`、`NORMAL`、`LOW`、`LOWEST`。 |
| SCAN_ROWS            | スキャンされたデータ行の数。                                    |
| FILTERED_ROWS        | データ品質が十分でないためにフィルタリングされたデータ行の数。     |
| UNSELECTED_ROWS      | WHERE句で指定された条件によってフィルタリングされたデータ行の数。  |
| SINK_ROWS            | 読み込まれたデータ行の数。                                       |
| ETL_INFO             | ロードジョブのETLの詳細。Spark Loadの場合のみ非空の値が返されます。他のタイプのロードジョブの場合、空の値が返されます。 |
| TASK_INFO            | ロードジョブのタスク実行の詳細、例えば`timeout`や`max_filter_ratio`の設定です。 |
| CREATE_TIME          | ロードジョブが作成された時刻。フォーマット：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_START_TIME       | ロードジョブのETLステージの開始時刻。フォーマット：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_FINISH_TIME      | ロードジョブのETLステージの終了時刻。フォーマット：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_START_TIME      | ロードジョブのLOADINGステージの開始時刻。フォーマット：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_FINISH_TIME     | ロードジョブのLOADINGステージの終了時刻。フォーマット：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| JOB_DETAILS          | ロードされたデータの詳細、例えばバイト数やファイル数です。 |
| ERROR_MSG            | ロードジョブのエラーメッセージ。エラーがない場合は、`NULL`が返されます。  |
| TRACKING_URL         | ロードジョブで検出された不適格データ行にアクセスできるURLです。`curl`や`wget`コマンドを使用してURLにアクセスし、不適格データ行を取得できます。不適格データがない場合は、`NULL`が返されます。 |
| TRACKING_SQL         | ロードジョブのトラッキングログをクエリするために使用できるSQLステートメントです。不適格データ行が関与するロードジョブの場合のみ、SQLステートメントが返されます。不適格データ行が関与しない場合は、`NULL`が返されます。 |
| REJECTED_RECORD_PATH | ロードジョブでフィルタリングされたすべての不適格データ行にアクセスできるパスです。`log_rejected_record_num`パラメータで設定された不適格データ行の数によってログに記録される不適格データ行の数が決まります。`wget`コマンドを使用してパスにアクセスできます。不適格データが関与しない場合は、`NULL`が返されます。 |

### tables

`tables`で提供されるフィールドは次のとおりです：

| **Field**       | **Description**                                              |
| --------------- | ------------------------------------------------------------ |
| TABLE_CATALOG   | テーブルを格納するカタログの名前。                             |
| TABLE_SCHEMA    | テーブルを格納するデータベースの名前。                        |
| TABLE_NAME      | テーブルの名前。                                             |
| TABLE_TYPE      | テーブルのタイプ。有効な値："BASE TABLE"または"VIEW"。          |
| ENGINE          | テーブルのエンジンタイプ。有効な値："StarRocks"、"MySQL"、"MEMORY"、または空の文字列。 |
| VERSION         | StarRocksで利用可能な機能に適用されます。                        |
| ROW_FORMAT      | StarRocksで利用可能な機能に適用されます。                        |
| TABLE_ROWS      | テーブルの行数。                                              |
| AVG_ROW_LENGTH  | テーブルの平均行長（サイズ）。`DATA_LENGTH` / `TABLE_ROWS`と等価です。単位：バイト。 |
| DATA_LENGTH     | テーブルのデータ長さ（サイズ）。単位：バイト。                  |
| MAX_DATA_LENGTH | StarRocksで利用可能な機能に適用されます。                        |
| INDEX_LENGTH    | StarRocksで利用可能な機能に適用されます。                        |
| DATA_FREE       | StarRocksで利用可能な機能に適用されます。                        |
| AUTO_INCREMENT  | StarRocksで利用可能な機能に適用されます。                        |
| CREATE_TIME     | テーブルが作成された時刻。                                       |
| UPDATE_TIME     | テーブルが最後に更新された時刻。                                  |
| CHECK_TIME      | テーブルで一貫性チェックが実行された最後の時刻。                    |
| TABLE_COLLATION | テーブルのデフォルト照合順序。                                    |
| CHECKSUM        | StarRocksで利用可能な機能に適用されます。                        |
| CREATE_OPTIONS  | StarRocksで利用可能な機能に適用されます。                        |
| TABLE_COMMENT   | テーブルに関するコメント。                                       |

### tables_config

`tables_config`で提供されるフィールドは次のとおりです：

| **Field**        | **Description**                                              |
| ---------------- | ------------------------------------------------------------ |
| TABLE_SCHEMA     | テーブルを格納するデータベースの名前。                        |
| TABLE_NAME       | テーブルの名前。                                             |
| TABLE_ENGINE     | テーブルのエンジンタイプ。                                    |
| TABLE_MODEL      | テーブルのタイプ。有効な値："DUP_KEYS"、"AGG_KEYS"、"UNQ_KEYS"、"PRI_KEYS"。 |
| PRIMARY_KEY      | Primary KeyテーブルまたはUnique Keyテーブルのプライマリキー。テーブルがPrimary KeyテーブルまたはUnique Keyテーブルでない場合は空の文字列が返されます。 |
| PARTITION_KEY    | テーブルのパーティション列。                         |
| DISTRIBUTE_KEY   | テーブルのバケット化された列。                          |
| DISTRIBUTE_TYPE  | テーブルのデータ分散方法。                    |
| DISTRIBUTE_BUCKET | テーブル内のバケット数。                              |
| SORT_KEY         | テーブルのソートキー。                                   |
| PROPERTIES       | テーブルのプロパティ。                                   |
| TABLE_ID         | テーブルのID。                                             |

## load_tracking_logs

この機能はStarRocks v3.0からサポートされています。

`load_tracking_logs`には次のフィールドがあります:

| **Field**     | **Description**                                                                       |
|---------------|---------------------------------------------------------------------------------------|
| JOB_ID        | ロードジョブのID。                                                               |
| LABEL         | ロードジョブのラベル。                                                            |
| DATABASE_NAME | ロードジョブが属するデータベース。                                            |
| TRACKING_LOG  | ロードジョブのエラーログ（あれば）。                                                  |
| Type          | ロードジョブのタイプ。有効な値: BROKER, INSERT, ROUTINE_LOAD, STREAM_LOAD。 |

## materialized_views

`materialized_views`には次のフィールドがあります:

| **Field**                            | **Description**                                              |
| ------------------------------------ | ------------------------------------------------------------ |
| MATERIALIZED_VIEW_ID                 | マテリアライズドビューのID                                  |
| TABLE_SCHEMA                         | マテリアライズドビューが存在するデータベース                |
| TABLE_NAME                           | マテリアライズドビューの名前                                |
| REFRESH_TYPE                         | マテリアライズドビューのリフレッシュタイプ、`ROLLUP`、`ASYNC`、`MANUAL`を含む |
| IS_ACTIVE                            | マテリアライズドビューがアクティブかどうかを示す。非アクティブなマテリアライズドビューはリフレッシュやクエリができません。 |
| INACTIVE_REASON                      | マテリアライズドビューが非アクティブである理由            |
| PARTITION_TYPE                       | マテリアライズドビューのパーティション戦略のタイプ      |
| TASK_ID                              | マテリアライズドビューのリフレッシュを担当するタスクのID  |
| TASK_NAME                            | マテリアライズドビューのリフレッシュを担当するタスクの名前  |
| LAST_REFRESH_START_TIME              | 最新のリフレッシュタスクの開始時刻                  |
| LAST_REFRESH_FINISHED_TIME           | 最新のリフレッシュタスクの終了時刻                    |
| LAST_REFRESH_DURATION                | 最新のリフレッシュタスクの期間                    |
| LAST_REFRESH_STATE                   | 最新のリフレッシュタスクのステータス                    |
| LAST_REFRESH_FORCE_REFRESH           | 最新のリフレッシュが強制リフレッシュであるかどうかを示す |
| LAST_REFRESH_START_PARTITION         | 最新のリフレッシュタスクの開始パーティション            |
| LAST_REFRESH_END_PARTITION           | 最新のリフレッシュタスクの終了パーティション            |
| LAST_REFRESH_BASE_REFRESH_PARTITIONS | 最新のリフレッシュタスクに関与するベーステーブルのパーティション |
| LAST_REFRESH_MV_REFRESH_PARTITIONS   | 最新のリフレッシュタスクでリフレッシュされたマテリアライズドビューのパーティション |
| LAST_REFRESH_ERROR_CODE              | 最新のリフレッシュタスクのエラーコード                |
| LAST_REFRESH_ERROR_MESSAGE           | 最新のリフレッシュタスクのエラーメッセージ             |
| TABLE_ROWS                           | マテリアライズドビュー内のデータ行数（おおよそのバックグラウンド統計に基づく） |
| MATERIALIZED_VIEW_DEFINITION         | マテリアライズドビューのSQL定義                    |