---
displayed_sidebar: "Japanese"
---

# 情報スキーマ

StarRocksの`information_schema`は、StarRocksの各インスタンス内にあるデータベースです。`information_schema`には、StarRocksインスタンスが保持するすべてのオブジェクトの詳細なメタデータ情報を格納するいくつかの読み取り専用のシステム定義テーブルが含まれています。

v3.2以降、StarRocksは`information_schema`を介して外部カタログメタデータを表示することをサポートしています。

## 情報スキーマを介したメタデータの表示

`information_schema`のテーブルのコンテンツをクエリして、StarRocksインスタンス内のメタデータ情報を表示することができます。

次の例は、テーブル`tables`をクエリして、StarRocks内のテーブル`sr_member`に関するメタデータ情報を表示しています。

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

## 情報スキーマテーブル

StarRocksは、`information_schema`内のテーブル`tables`、`tables_config`、`load_tracking_logs`のメタデータ情報を最適化しており、v3.1以降から`loads`テーブルを提供しています。

| **情報スキーマテーブル名** | **説明**                                                     |
| --------------------------------- | ------------------------------------------------------------ |
| [tables](#tables)                            | テーブルの一般的なメタデータ情報を提供します。             |
| [tables_config](#tables_config)                     | StarRocks固有の追加のテーブルメタデータ情報を提供します。 |
| [load_tracking_logs](#load_tracking_logs)                | ロードジョブのエラー情報（あれば）を提供します。      |
| [loads](#loads)                             | v3.1以降でサポートされる、ロードジョブの結果を提供します。現時点では、このテーブルから[BROKER_LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)および[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)ジョブの結果を表示することができます。                 |

### loads

`loads`では、以下のフィールドが提供されています：

| **フィールド**            | **説明**                                               |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | StarRocksがロードジョブを識別するために割り当てた一意のIDです。 |
| LABEL                | ロードジョブのラベルです。                                    |
| DATABASE_NAME        | 対象のStarRocksテーブルが所属するデータベースの名前です。     |
| STATE                | ロードジョブの状態です。有効な値:<ul><li>`PENDING`: ロードジョブが作成されています。</li><li>`QUEUEING`: ロードジョブがスケジュール待ちのキューにあります。</li><li>`LOADING`: ロードジョブが実行されています。</li><li>`PREPARED`: トランザクションがコミットされました。</li><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul>詳細については、[非同期ロード](../loading/Loading_intro.md#asynchronous-loading)を参照してください。 |
| PROGRESS             | ロードジョブのETLステージとLOADINGステージの進行状況です。          |
| TYPE                 | ロードジョブのタイプです。Broker Loadの場合、戻り値は`BROKER`です。INSERTの場合、戻り値は`INSERT`です。 |
| PRIORITY             | ロードジョブの優先度です。有効な値:`HIGHEST`、`HIGH`、`NORMAL`、`LOW`、`LOWEST`。 |
| SCAN_ROWS            | スキャンされたデータ行の数です。                                  |
| FILTERED_ROWS        | データ品質が不十分でフィルタリングされたデータ行の数です。     |
| UNSELECTED_ROWS      | WHERE句で指定された条件によってフィルタリングされたデータ行の数です。 |
| SINK_ROWS            | ロードされたデータ行の数です。                                     |
| ETL_INFO             | ロードジョブのETLの詳細です。Spark Loadの場合のみ、空でない値が返されます。その他の種類のロードジョブでは、空の値が返されます。 |
| TASK_INFO            | ロードジョブのタスク実行の詳細（`timeout`、`max_filter_ratio`の設定など）です。 |
| CREATE_TIME          | ロードジョブの作成時刻です。フォーマット:`yyyy-MM-dd HH:mm:ss`。例:`2023-07-24 14:58:58`。 |
| ETL_START_TIME       | ロードジョブのETLステージの開始時刻です。フォーマット:`yyyy-MM-dd HH:mm:ss`。例:`2023-07-24 14:58:58`。 |
| ETL_FINISH_TIME      | ロードジョブのETLステージの終了時刻です。フォーマット:`yyyy-MM-dd HH:mm:ss`。例:`2023-07-24 14:58:58`。 |
| LOAD_START_TIME      | ロードジョブのLOADINGステージの開始時刻です。フォーマット:`yyyy-MM-dd HH:mm:ss`。例:`2023-07-24 14:58:58`。 |
| LOAD_FINISH_TIME     | ロードジョブのLOADINGステージの終了時刻です。フォーマット:`yyyy-MM-dd HH:mm:ss`。例:`2023-07-24 14:58:58`。 |
| JOB_DETAILS          | 読み込まれたデータの詳細（バイト数やファイル数など）です。    |
| ERROR_MSG            | ロードジョブのエラーメッセージです。ロードジョブにエラーがない場合は、`NULL`が返されます。 |
| TRACKING_URL         | ロードジョブで検出された不適格なデータ行へアクセスできるURLです。`curl`または`wget`コマンドを使用してURLにアクセスし、不適格なデータ行のサンプルを取得できます。不適格なデータが検出されない場合は、`NULL`が返されます。 |
| TRACKING_SQL         | ロードジョブの追跡ログをクエリするために使用できるSQL文です。ロードジョブが不適格なデータ行を含む場合のみ、SQL文が返されます。ロードジョブが不適格なデータ行を含まない場合は、`NULL`が返されます。 |
| REJECTED_RECORD_PATH | ロードジョブでフィルタリングされたすべての不適格なデータ行にアクセスできるパスです。`log_rejected_record_num`パラメータで設定された不適格なデータ行の数によって、記録された不適格なデータ行の数が決定されます。`wget`コマンドを使用してパスにアクセスできます。ロードジョブが不適格なデータ行を含まない場合は、`NULL`が返されます。 |

### tables

`tables`では、以下のフィールドが提供されています：

| **フィールド**       | **説明**                                               |
| --------------- | ------------------------------------------------------------ |
| TABLE_CATALOG   | テーブルを格納するカタログの名前です。                    |
| TABLE_SCHEMA    | テーブルを格納するデータベースの名前です。                |
| TABLE_NAME      | テーブルの名前です。                                      |
| TABLE_TYPE      | テーブルの種類です。有効な値:「BASE TABLE」または「VIEW」。      |
| ENGINE          | テーブルのエンジンタイプです。有効な値:「StarRocks」、「MySQL」、「MEMORY」、または空の文字列。 |
| VERSION         | StarRocksで使用できない機能に適用されます。                      |
| ROW_FORMAT      | StarRocksで使用できない機能に適用されます。                      |
| TABLE_ROWS      | テーブルの行数です。                                         |
| AVG_ROW_LENGTH  | テーブルの平均行長（サイズ）です。`DATA_LENGTH` / `TABLE_ROWS`と同等です。単位: バイト。 |
| DATA_LENGTH     | テーブルのデータ長（サイズ）です。単位: バイト。                |
| MAX_DATA_LENGTH | StarRocksで使用できない機能に適用されます。                      |
| INDEX_LENGTH    | StarRocksで使用できない機能に適用されます。                      |
| DATA_FREE       | StarRocksで使用できない機能に適用されます。                      |
| AUTO_INCREMENT  | StarRocksで使用できない機能に適用されます。                      |
| CREATE_TIME     | テーブルが作成された時刻です。                                 |
| UPDATE_TIME     | テーブルが最後に更新された時刻です。                            |
| CHECK_TIME      | テーブルについての整合性チェックが最後に実行された時刻です。            |
| TABLE_COLLATION | テーブルのデフォルト照合順序です。                            |
| CHECKSUM        | StarRocksで使用できない機能に適用されます。                      |
| CREATE_OPTIONS  | StarRocksで使用できない機能に適用されます。                      |
| TABLE_COMMENT   | テーブルのコメントです。                                      |

### tables_config

`tables_config`では、以下のフィールドが提供されています：

| **フィールド**        | **説明**                                               |
| ---------------- | ------------------------------------------------------------ |
| TABLE_SCHEMA     | テーブルを格納するデータベースの名前です。                |
| TABLE_NAME       | テーブルの名前です。                                      |
| TABLE_ENGINE     | テーブルのエンジンタイプです。                               |
| TABLE_MODEL      | テーブルの種類です。有効な値:「DUP_KEYS」、「AGG_KEYS」、「UNQ_KEYS」、「PRI_KEYS」。 |
| PRIMARY_KEY      | Primary KeyテーブルまたはUnique Keyテーブルのプライマリキー。テーブルがPrimary KeyテーブルまたはUnique Keyテーブルでない場合は空の文字列が返されます。 |
| PARTITION_KEY    | テーブルのパーティショニング列。                                  |
| DISTRIBUTE_KEY   | テーブルのバケット化列。                                         |
| DISTRIBUTE_TYPE  | テーブルのデータ分散方法。                                      |
| DISTRIBUTE_BUCKET | テーブル内のバケット数。                                         |
| SORT_KEY         | テーブルのソートキー。                                            |
| PROPERTIES       | テーブルのプロパティ。                                            |
| TABLE_ID         | テーブルのID。                                                    |

## load_tracking_logs

この機能はStarRocks v3.0以降でサポートされています。

`load_tracking_logs`には、次のフィールドが提供されます:

| **Field**     | **Description**                                                                       |
|---------------|---------------------------------------------------------------------------------------|
| JOB_ID        | ロードジョブのID。                                                                     |
| LABEL         | ロードジョブのラベル。                                                                 |
| DATABASE_NAME | ロードジョブが属するデータベース。                                                    |
| TRACKING_LOG  | ロードジョブのエラーログ（あれば）。                                                  |
| Type          | ロードジョブのタイプ。有効な値: BROKER、INSERT、ROUTINE_LOAD、およびSTREAM_LOAD。  |

## materialized_views

`materialized_views`には、次のフィールドが提供されます:

| **Field**                            | **Description**                                              |
| ------------------------------------ | ------------------------------------------------------------ |
| MATERIALIZED_VIEW_ID                 | マテリアライズドビューのID                                   |
| TABLE_SCHEMA                         | マテリアライズドビューが存在するデータベース                 |
| TABLE_NAME                           | マテリアライズドビューの名前                                 |
| REFRESH_TYPE                         | マテリアライズドビューのリフレッシュタイプ。`ROLLUP`、`ASYNC`、`MANUAL`を含む |
| IS_ACTIVE                            | マテリアライズドビューがアクティブかどうかを示します。非アクティブなマテリアライズドビューはリフレッシュやクエリができません。 |
| INACTIVE_REASON                      | マテリアライズドビューが非アクティブになっている理由        |
| PARTITION_TYPE                       | マテリアライズドビューのパーティション戦略のタイプ          |
| TASK_ID                              | マテリアライズドビューのリフレッシュを担当するタスクのID     |
| TASK_NAME                            | マテリアライズドビューのリフレッシュを担当するタスクの名前   |
| LAST_REFRESH_START_TIME              | 最新のリフレッシュタスクの開始時間                           |
| LAST_REFRESH_FINISHED_TIME           | 最新のリフレッシュタスクの終了時間                           |
| LAST_REFRESH_DURATION                | 最新のリフレッシュタスクの所要時間                           |
| LAST_REFRESH_STATE                   | 最新のリフレッシュタスクの状態                               |
| LAST_REFRESH_FORCE_REFRESH           | 最新のリフレッシュタスクが強制リフレッシュであるかどうかを示します |
| LAST_REFRESH_START_PARTITION         | 最新のリフレッシュタスクの開始パーティション                 |
| LAST_REFRESH_END_PARTITION           | 最新のリフレッシュタスクの終了パーティション                 |
| LAST_REFRESH_BASE_REFRESH_PARTITIONS | 最新のリフレッシュタスクに関与するベーステーブルのパーティション |
| LAST_REFRESH_MV_REFRESH_PARTITIONS   | 最新のリフレッシュタスクでリフレッシュされたマテリアライズドビューのパーティション |
| LAST_REFRESH_ERROR_CODE              | 最新のリフレッシュタスクのエラーコード                       |
| LAST_REFRESH_ERROR_MESSAGE           | 最新のリフレッシュタスクのエラーメッセージ                   |
| TABLE_ROWS                           | マテリアライズドビュー内のデータ行数（おおまかなバックグラウンド統計に基づく） |
| MATERIALIZED_VIEW_DEFINITION         | マテリアライズドビューのSQL定義                             |