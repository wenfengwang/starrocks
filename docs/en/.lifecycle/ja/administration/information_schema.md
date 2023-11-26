---
displayed_sidebar: "Japanese"
---

# インフォメーションスキーマ

StarRocksの`information_schema`は、各StarRocksインスタンス内のデータベースです。`information_schema`には、StarRocksインスタンスが保持するすべてのオブジェクトの詳細なメタデータ情報を格納するいくつかの読み取り専用のシステム定義テーブルが含まれています。

v3.2以降、StarRocksは`information_schema`を介して外部カタログのメタデータを表示することをサポートしています。

## インフォメーションスキーマを介してメタデータを表示する

`information_schema`のテーブルの内容をクエリして、StarRocksインスタンス内のメタデータ情報を表示することができます。

以下の例では、テーブル`sr_member`のメタデータ情報を`tables`テーブルをクエリして表示しています。

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

## インフォメーションスキーマのテーブル

StarRocksは、`tables`、`tables_config`、および`load_tracking_logs`テーブルが提供するメタデータ情報を最適化し、v3.1以降、`information_schema`に`loads`テーブルを提供しています。

| **インフォメーションスキーマテーブル名** | **説明**                                                     |
| --------------------------------------- | ------------------------------------------------------------ |
| [tables](#tables)                       | テーブルの一般的なメタデータ情報を提供します。                 |
| [tables_config](#tables_config)         | StarRocks固有の追加のテーブルメタデータ情報を提供します。       |
| [load_tracking_logs](#load_tracking_logs) | ロードジョブのエラー情報（ある場合）を提供します。             |
| [loads](#loads)                         | ロードジョブの結果を提供します。このテーブルはv3.1以降でサポートされています。現在、このテーブルからは[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)および[Insert](../sql-reference/sql-statements/data-manipulation/INSERT.md)ジョブの結果のみを表示できます。 |

### loads

`loads`テーブルには、次のフィールドが提供されています。

| **フィールド**       | **説明**                                                     |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | StarRocksがロードジョブを識別するために割り当てた一意のIDです。 |
| LABEL                | ロードジョブのラベルです。                                   |
| DATABASE_NAME        | デスティネーションのStarRocksテーブルが所属するデータベースの名前です。 |
| STATE                | ロードジョブの状態です。有効な値:<ul><li>`PENDING`: ロードジョブが作成されました。</li><li>`QUEUEING`: ロードジョブがスケジュール待ちのキューにあります。</li><li>`LOADING`: ロードジョブが実行中です。</li><li>`PREPARED`: トランザクションがコミットされました。</li><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul>詳細については、[非同期ロード](../loading/Loading_intro.md#asynchronous-loading)を参照してください。 |
| PROGRESS             | ロードジョブのETLステージとLOADINGステージの進捗状況です。    |
| TYPE                 | ロードジョブのタイプです。Broker Loadの場合、返り値は`BROKER`です。INSERTの場合、返り値は`INSERT`です。 |
| PRIORITY             | ロードジョブの優先度です。有効な値: `HIGHEST`、`HIGH`、`NORMAL`、`LOW`、`LOWEST`です。 |
| SCAN_ROWS            | スキャンされたデータ行の数です。                             |
| FILTERED_ROWS        | データ品質が不十分なためにフィルタリングされたデータ行の数です。 |
| UNSELECTED_ROWS      | WHERE句で指定された条件によりフィルタリングされたデータ行の数です。 |
| SINK_ROWS            | ロードされたデータ行の数です。                               |
| ETL_INFO             | ロードジョブのETLの詳細です。Spark Loadの場合のみ、空でない値が返されます。他のタイプのロードジョブの場合、空の値が返されます。 |
| TASK_INFO            | ロードジョブのタスク実行の詳細（`timeout`、`max_filter_ratio`など）です。 |
| CREATE_TIME          | ロードジョブが作成された時刻です。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| ETL_START_TIME       | ロードジョブのETLステージの開始時刻です。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| ETL_FINISH_TIME      | ロードジョブのETLステージの終了時刻です。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| LOAD_START_TIME      | ロードジョブのLOADINGステージの開始時刻です。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| LOAD_FINISH_TIME     | ロードジョブのLOADINGステージの終了時刻です。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |
| JOB_DETAILS          | ロードされたデータの詳細（バイト数、ファイル数など）です。 |
| ERROR_MSG            | ロードジョブのエラーメッセージです。ロードジョブにエラーが発生しなかった場合、`NULL`が返されます。 |
| TRACKING_URL         | ロードジョブで検出された不適格なデータ行のサンプルにアクセスできるURLです。`curl`コマンドや`wget`コマンドを使用してURLにアクセスし、不適格なデータ行のサンプルを取得することができます。不適格なデータが検出されない場合、`NULL`が返されます。 |
| TRACKING_SQL         | ロードジョブのトラッキングログをクエリするために使用できるSQLステートメントです。ロードジョブに不適格なデータ行が関与する場合のみ、SQLステートメントが返されます。ロードジョブに不適格なデータ行が関与しない場合、`NULL`が返されます。 |
| REJECTED_RECORD_PATH | ロードジョブでフィルタリングされたすべての不適格なデータ行にアクセスできるパスです。ログに記録される不適格なデータ行の数は、ロードジョブで構成された`log_rejected_record_num`パラメータによって決まります。`wget`コマンドを使用してパスにアクセスすることができます。ロードジョブに不適格なデータ行が関与しない場合、`NULL`が返されます。 |

### tables

`tables`テーブルには、次のフィールドが提供されています。

| **フィールド**       | **説明**                                                     |
| -------------------- | ------------------------------------------------------------ |
| TABLE_CATALOG        | テーブルを格納するカタログの名前です。                         |
| TABLE_SCHEMA         | テーブルを格納するデータベースの名前です。                     |
| TABLE_NAME           | テーブルの名前です。                                         |
| TABLE_TYPE           | テーブルのタイプです。有効な値: "BASE TABLE"または"VIEW"です。 |
| ENGINE               | テーブルのエンジンタイプです。有効な値: "StarRocks"、"MySQL"、"MEMORY"、または空の文字列です。 |
| VERSION              | StarRocksでは使用できない機能に適用されます。                 |
| ROW_FORMAT           | StarRocksでは使用できない機能に適用されます。                 |
| TABLE_ROWS           | テーブルの行数です。                                          |
| AVG_ROW_LENGTH       | テーブルの平均行長（サイズ）です。`DATA_LENGTH` / `TABLE_ROWS`と同等です。単位: バイト。 |
| DATA_LENGTH          | テーブルのデータ長（サイズ）です。単位: バイト。               |
| MAX_DATA_LENGTH      | StarRocksでは使用できない機能に適用されます。                 |
| INDEX_LENGTH         | StarRocksでは使用できない機能に適用されます。                 |
| DATA_FREE            | StarRocksでは使用できない機能に適用されます。                 |
| AUTO_INCREMENT       | StarRocksでは使用できない機能に適用されます。                 |
| CREATE_TIME          | テーブルが作成された時刻です。                                |
| UPDATE_TIME          | テーブルが最後に更新された時刻です。                          |
| CHECK_TIME           | テーブルの整合性チェックが実行された最後の時刻です。           |
| TABLE_COLLATION      | テーブルのデフォルトの照合順序です。                          |
| CHECKSUM             | StarRocksでは使用できない機能に適用されます。                 |
| CREATE_OPTIONS       | StarRocksでは使用できない機能に適用されます。                 |
| TABLE_COMMENT        | テーブルのコメントです。                                      |

### tables_config

`tables_config`テーブルには、次のフィールドが提供されています。

| **フィールド**       | **説明**                                                     |
| -------------------- | ------------------------------------------------------------ |
| TABLE_SCHEMA         | テーブルを格納するデータベースの名前です。                     |
| TABLE_NAME           | テーブルの名前です。                                         |
| TABLE_ENGINE         | テーブルのエンジンタイプです。                                |
| TABLE_MODEL          | テーブルのタイプです。有効な値: "DUP_KEYS"、"AGG_KEYS"、"UNQ_KEYS"、または"PRI_KEYS"です。 |
| PRIMARY_KEY          | プライマリキーまたはユニークキーのテーブルのプライマリキーです。テーブルがプライマリキーまたはユニークキーのテーブルでない場合、空の文字列が返されます。 |
| PARTITION_KEY        | テーブルのパーティショニング列です。                          |
| DISTRIBUTE_KEY       | テーブルのバケット化列です。                                 |
| DISTRIBUTE_TYPE      | テーブルのデータ分散方法です。                                |
| DISTRIBUTE_BUCKET    | テーブルのバケット数です。                                    |
| SORT_KEY             | テーブルのソートキーです。                                    |
| PROPERTIES           | テーブルのプロパティです。                                    |
| TABLE_ID             | テーブルのIDです。                                            |

## load_tracking_logs

この機能はStarRocks v3.0以降でサポートされています。

`load_tracking_logs`テーブルには、次のフィールドが提供されています。

| **フィールド**       | **説明**                                                     |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | ロードジョブのIDです。                                       |
| LABEL                | ロードジョブのラベルです。                                   |
| DATABASE_NAME        | ロードジョブが所属するデータベースです。                      |
| TRACKING_LOG         | ロードジョブのエラーログ（ある場合）です。                     |
| Type                 | ロードジョブのタイプです。有効な値: BROKER、INSERT、ROUTINE_LOAD、STREAM_LOADです。 |

## materialized_views

`materialized_views`テーブルには、次のフィールドが提供されています。

| **フィールド**                            | **説明**                                                     |
| ---------------------------------------- | ------------------------------------------------------------ |
| MATERIALIZED_VIEW_ID                      | マテリアライズドビューのIDです。                               |
| TABLE_SCHEMA                              | マテリアライズドビューが所属するデータベースです。             |
| TABLE_NAME                                | マテリアライズドビューの名前です。                             |
| REFRESH_TYPE                              | マテリアライズドビューのリフレッシュタイプです。`ROLLUP`、`ASYNC`、`MANUAL`を含みます。 |
| IS_ACTIVE                                 | マテリアライズドビューがアクティブかどうかを示します。非アクティブなマテリアライズドビューはリフレッシュやクエリを実行することができません。 |
| INACTIVE_REASON                           | マテリアライズドビューが非アクティブである理由です。           |
| PARTITION_TYPE                            | マテリアライズドビューのパーティショニング戦略のタイプです。   |
| TASK_ID                                   | マテリアライズドビューのリフレッシュを担当するタスクのIDです。 |
| TASK_NAME                                 | マテリアライズドビューのリフレッシュを担当するタスクの名前です。 |
| LAST_REFRESH_START_TIME                   | 直近のリフレッシュタスクの開始時刻です。                       |
| LAST_REFRESH_FINISHED_TIME                | 直近のリフレッシュタスクの終了時刻です。                       |
| LAST_REFRESH_DURATION                     | 直近のリフレッシュタスクの所要時間です。                       |
| LAST_REFRESH_STATE                        | 直近のリフレッシュタスクの状態です。                           |
| LAST_REFRESH_FORCE_REFRESH                | 直近のリフレッシュタスクが強制リフレッシュであるかどうかを示します。 |
| LAST_REFRESH_START_PARTITION              | 直近のリフレッシュタスクの開始パーティションです。             |
| LAST_REFRESH_END_PARTITION                | 直近のリフレッシュタスクの終了パーティションです。             |
| LAST_REFRESH_BASE_REFRESH_PARTITIONS      | 直近のリフレッシュタスクに関与するベーステーブルのパーティションです。 |
| LAST_REFRESH_MV_REFRESH_PARTITIONS        | 直近のリフレッシュタスクでリフレッシュされたマテリアライズドビューのパーティションです。 |
| LAST_REFRESH_ERROR_CODE                   | 直近のリフレッシュタスクのエラーコードです。                   |
| LAST_REFRESH_ERROR_MESSAGE                | 直近のリフレッシュタスクのエラーメッセージです。               |
| TABLE_ROWS                                | マテリアライズドビューのデータ行数です。背景統計に基づいています。 |
| MATERIALIZED_VIEW_DEFINITION              | マテリアライズドビューのSQL定義です。                          |
