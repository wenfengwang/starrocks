---
displayed_sidebar: English
---

# Information Schema

StarRocksの`information_schema`は、各StarRocksインスタンス内のデータベースです。`information_schema`には、StarRocksインスタンスが保持するすべてのオブジェクトの詳細なメタデータ情報を格納する、読み取り専用のシステム定義テーブルがいくつか含まれています。

v3.2以降、StarRocksは`information_schema`を通じて外部カタログメタデータの表示をサポートしています。

## Information Schemaを通じたメタデータの表示

StarRocksインスタンス内のメタデータ情報を表示するには、`information_schema`のテーブルの内容を照会します。

次の例では、StarRocks内の`sr_member`という名前のテーブルに関するメタデータ情報を`tables`テーブルをクエリして表示します。

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

## Information Schemaのテーブル

StarRocksは、`tables`、`tables_config`、`load_tracking_logs`テーブルによって提供されるメタデータ情報を最適化し、v3.1以降`information_schema`に`loads`テーブルを提供しています：

| **Information Schemaのテーブル名** | **説明**                                              |
| --------------------------------- | ------------------------------------------------------------ |
| [tables](#tables)                            | テーブルの一般的なメタデータ情報を提供します。             |
| [tables_config](#tables_config)                     | StarRocksに特有の追加テーブルメタデータ情報を提供します。 |
| [load_tracking_logs](#load_tracking_logs)                | ロードジョブのエラー情報（ある場合）を提供します。 |
| [loads](#loads)                             | ロードジョブの結果を提供します。このテーブルはv3.1以降でサポートされています。現在、このテーブルから表示できるのは[BROKER_LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)と[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)ジョブの結果のみです。                 |

### loads

`loads`には以下のフィールドが提供されています：

| **フィールド**            | **説明**                                              |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | ロードジョブを識別するためにStarRocksによって割り当てられた一意のID。 |
| LABEL                | ロードジョブのラベル。                                   |
| DATABASE_NAME        | 宛先StarRocksテーブルが属するデータベースの名前。 |
| STATE                | ロードジョブの状態。有効な値：<ul><li>`PENDING`：ロードジョブが作成されました。</li><li>`QUEUEING`：ロードジョブがキューに入ってスケジュールを待っています。</li><li>`LOADING`：ロードジョブが実行中です。</li><li>`PREPARED`：トランザクションがコミットされました。</li><li>`FINISHED`：ロードジョブが成功しました。</li><li>`CANCELLED`：ロードジョブが失敗しました。</li></ul>詳細については、[非同期ローディング](../loading/Loading_intro.md#asynchronous-loading)を参照してください。 |
| PROGRESS             | ロードジョブのETLステージとLOADINGステージの進行状況。 |
| TYPE                 | ロードジョブのタイプ。Broker Loadの場合、戻り値は`BROKER`。INSERTの場合、戻り値は`INSERT`。 |
| PRIORITY             | ロードジョブの優先順位。有効な値：`HIGHEST`、`HIGH`、`NORMAL`、`LOW`、`LOWEST`。 |
| SCAN_ROWS            | スキャンされたデータ行の数。                    |
| FILTERED_ROWS        | データ品質が不十分でフィルタリングされたデータ行の数。 |
| UNSELECTED_ROWS      | WHERE句で指定された条件によりフィルタリングされたデータ行の数。 |
| SINK_ROWS            | ロードされたデータ行の数。                     |
| ETL_INFO             | ロードジョブのETL詳細。Spark Loadに対してのみ非空の値が返されます。他のタイプのロードジョブでは空の値が返されます。 |
| TASK_INFO            | ロードジョブのタスク実行の詳細、例えば`timeout`や`max_filter_ratio`の設定など。 |
| CREATE_TIME          | ロードジョブが作成された時刻。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_START_TIME       | ロードジョブのETLステージの開始時刻。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_FINISH_TIME      | ロードジョブのETLステージの終了時刻。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_START_TIME      | ロードジョブのLOADINGステージの開始時刻。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_FINISH_TIME     | ロードジョブのLOADINGステージの終了時刻。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| JOB_DETAILS          | ロードされたデータの詳細、例えばバイト数やファイル数など。 |
| ERROR_MSG            | ロードジョブのエラーメッセージ。エラーが発生していない場合は`NULL`が返されます。 |
| TRACKING_URL         | ロードジョブで検出された非適格データ行サンプルにアクセスできるURL。`curl`または`wget`コマンドを使用してURLにアクセスし、非適格データ行サンプルを取得できます。非適格データが検出されなかった場合は`NULL`が返されます。 |

| TRACKING_SQL         | ロードジョブのトラッキングログを照会するために使用できるSQLステートメントです。ロードジョブに不適格なデータ行が含まれている場合のみ、SQLステートメントが返されます。ロードジョブに不適格なデータ行が含まれていない場合は、`NULL`が返されます。 |
| REJECTED_RECORD_PATH | ロードジョブでフィルタリングされた不適格なデータ行にアクセスできるパスです。ログに記録される不適格なデータ行の数は、ロードジョブで設定された`log_rejected_record_num`パラメータによって決定されます。`wget`コマンドを使用してパスにアクセスできます。ロードジョブに不適格なデータ行が含まれていない場合は、`NULL`が返されます。 |

### tables

`tables`には以下のフィールドが提供されています：

| **フィールド**       | **説明**                                              |
| --------------- | ------------------------------------------------------------ |
| TABLE_CATALOG   | テーブルを格納するカタログの名前です。                   |
| TABLE_SCHEMA    | テーブルを格納するデータベースの名前です。                  |
| TABLE_NAME      | テーブルの名前です。                                           |
| TABLE_TYPE      | テーブルのタイプです。有効な値は "BASE TABLE" または "VIEW" です。     |
| ENGINE          | テーブルのエンジンタイプです。有効な値は "StarRocks", "MySQL", "MEMORY"、または空文字列です。 |
| VERSION         | StarRocksでは利用できない機能に適用されます。             |
| ROW_FORMAT      | StarRocksでは利用できない機能に適用されます。             |
| TABLE_ROWS      | テーブルの行数です。                                      |
| AVG_ROW_LENGTH  | テーブルの平均行長（サイズ）です。`DATA_LENGTH` / `TABLE_ROWS`に相当します。単位はバイトです。 |
| DATA_LENGTH     | テーブルのデータ長（サイズ）です。単位はバイトです。                 |
| MAX_DATA_LENGTH | StarRocksでは利用できない機能に適用されます。             |
| INDEX_LENGTH    | StarRocksでは利用できない機能に適用されます。             |
| DATA_FREE       | StarRocksでは利用できない機能に適用されます。             |
| AUTO_INCREMENT  | StarRocksでは利用できない機能に適用されます。             |
| CREATE_TIME     | テーブルが作成された時刻です。                          |
| UPDATE_TIME     | テーブルが最後に更新された時刻です。                     |
| CHECK_TIME      | テーブルで整合性チェックが最後に実行された時刻です。 |
| TABLE_COLLATION | テーブルのデフォルト照合順序です。                          |
| CHECKSUM        | StarRocksでは利用できない機能に適用されます。             |
| CREATE_OPTIONS  | StarRocksでは利用できない機能に適用されます。             |
| TABLE_COMMENT   | テーブルに関するコメントです。                                        |

### tables_config

`tables_config`には以下のフィールドが提供されています：

| **フィールド**        | **説明**                                              |
| ---------------- | ------------------------------------------------------------ |
| TABLE_SCHEMA     | テーブルを格納するデータベースの名前です。                  |
| TABLE_NAME       | テーブルの名前です。                                           |
| TABLE_ENGINE     | テーブルのエンジンタイプです。                                    |
| TABLE_MODEL      | テーブルのタイプです。有効な値は "DUP_KEYS", "AGG_KEYS", "UNQ_KEYS", "PRI_KEYS" です。 |
| PRIMARY_KEY      | プライマリキーテーブルまたはユニークキーテーブルのプライマリキーです。テーブルがプライマリキーテーブルまたはユニークキーテーブルでない場合は、空文字列が返されます。 |
| PARTITION_KEY    | テーブルのパーティションキーです。                       |
| DISTRIBUTE_KEY   | テーブルの分散キーです。                          |
| DISTRIBUTE_TYPE  | テーブルのデータ分散方法です。                   |
| DISTRIBUTE_BUCKET | テーブルのバケット数です。                              |
| SORT_KEY         | テーブルのソートキーです。                                      |
| PROPERTIES       | テーブルのプロパティです。                                     |
| TABLE_ID         | テーブルのIDです。                                             |

## load_tracking_logs

この機能はStarRocks v3.0以降でサポートされています。

`load_tracking_logs`には以下のフィールドが提供されています：

| **フィールド**     | **説明**                                                                       |
|---------------|---------------------------------------------------------------------------------------|
| JOB_ID        | ロードジョブのIDです。                                                               |
| LABEL         | ロードジョブのラベルです。                                                            |
| DATABASE_NAME | ロードジョブが属するデータベースの名前です。                                            |
| TRACKING_LOG  | ロードジョブのエラーログ（存在する場合）。                                                  |
| Type          | ロードジョブのタイプです。有効な値は "BROKER", "INSERT", "ROUTINE_LOAD", "STREAM_LOAD" です。 |

## materialized_views

`materialized_views`には以下のフィールドが提供されています：

| **フィールド**                            | **説明**                                              |
| ------------------------------------ | ------------------------------------------------------------ |
| MATERIALIZED_VIEW_ID                 | マテリアライズドビューのIDです。                                  |
| TABLE_SCHEMA                         | マテリアライズドビューが存在するデータベースの名前です。              |
| TABLE_NAME                           | マテリアライズドビューの名前です。                                |
| REFRESH_TYPE                         | マテリアライズドビューのリフレッシュタイプです。有効な値は "ROLLUP", "ASYNC", "MANUAL" です。 |
| IS_ACTIVE                            | マテリアライズドビューがアクティブかどうかを示します。非アクティブなマテリアライズドビューはリフレッシュやクエリができません。 |
| INACTIVE_REASON                      | マテリアライズドビューが非アクティブである理由です。            |
| PARTITION_TYPE                       | マテリアライズドビューのパーティションタイプです。      |
| TASK_ID                              | マテリアライズドビューのリフレッシュを担当するタスクのIDです。 |
| TASK_NAME                            | マテリアライズドビューのリフレッシュを担当するタスクの名前です。 |
| LAST_REFRESH_START_TIME              | 最新のリフレッシュタスクの開始時刻です。                   |
| LAST_REFRESH_FINISHED_TIME           | 最新のリフレッシュタスクの終了時刻です。                     |
| LAST_REFRESH_DURATION                | 最新のリフレッシュタスクの期間です。                     |
| LAST_REFRESH_STATE                   | 最新のリフレッシュタスクの状態です。                        |
| LAST_REFRESH_FORCE_REFRESH           | 最新のリフレッシュタスクが強制リフレッシュであったかどうかを示します。 |
| LAST_REFRESH_START_PARTITION         | 最新のリフレッシュタスクの開始パーティションです。          |
| LAST_REFRESH_END_PARTITION           | 最新のリフレッシュタスクの終了パーティションです。            |
| LAST_REFRESH_BASE_REFRESH_PARTITIONS | 最新のリフレッシュタスクに関連するベーステーブルのパーティションです。 |
| LAST_REFRESH_MV_REFRESH_PARTITIONS   | 最新のリフレッシュタスクでリフレッシュされたマテリアライズドビューのパーティションです。 |
| LAST_REFRESH_ERROR_CODE              | 最新のリフレッシュタスクのエラーコードです。                   |

| LAST_REFRESH_ERROR_MESSAGE           | 最新のリフレッシュタスクのエラーメッセージ                |
| TABLE_ROWS                           | マテリアライズド・ビュー内のデータ行数（概算のバックグラウンド統計に基づく） |
| MATERIALIZED_VIEW_DEFINITION         | マテリアライズド・ビューのSQL定義                      |
