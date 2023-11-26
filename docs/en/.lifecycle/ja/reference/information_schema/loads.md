---
displayed_sidebar: "Japanese"
---

# ロード

`loads`はロードジョブの結果を提供します。このビューはStarRocks v3.1以降でサポートされています。現在、このビューからは[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)と[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)の結果のみを表示することができます。

`loads`には以下のフィールドが提供されます：

| **フィールド**          | **説明**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| JOB_ID                 | StarRocksがロードジョブを識別するために割り当てた一意のIDです。 |
| LABEL                  | ロードジョブのラベルです。                                    |
| DATABASE_NAME          | デスティネーションのStarRocksテーブルが所属するデータベースの名前です。 |
| STATE                  | ロードジョブの状態です。有効な値は次のとおりです：<ul><li>`PENDING`：ロードジョブが作成されました。</li><li>`QUEUEING`：ロードジョブがスケジュール待ちのキューにあります。</li><li>`LOADING`：ロードジョブが実行中です。</li><li>`PREPARED`：トランザクションがコミットされました。</li><li>`FINISHED`：ロードジョブが成功しました。</li><li>`CANCELLED`：ロードジョブが失敗しました。</li></ul>詳細については、[非同期ロード](../../loading/Loading_intro.md#asynchronous-loading)を参照してください。 |
| PROGRESS               | ロードジョブのETLステージとLOADINGステージの進捗状況です。   |
| TYPE                   | ロードジョブのタイプです。Broker Loadの場合、戻り値は`BROKER`です。INSERTの場合、戻り値は`INSERT`です。 |
| PRIORITY               | ロードジョブの優先度です。有効な値は`HIGHEST`、`HIGH`、`NORMAL`、`LOW`、`LOWEST`です。 |
| SCAN_ROWS              | スキャンされたデータ行の数です。                             |
| FILTERED_ROWS          | データ品質が不十分なためにフィルタリングされたデータ行の数です。 |
| UNSELECTED_ROWS        | WHERE句で指定された条件によりフィルタリングされたデータ行の数です。 |
| SINK_ROWS              | ロードされたデータ行の数です。                               |
| ETL_INFO               | ロードジョブのETLの詳細です。Spark Loadの場合のみ、空でない値が返されます。他のタイプのロードジョブでは、空の値が返されます。 |
| TASK_INFO              | ロードジョブのタスク実行の詳細です。`timeout`や`max_filter_ratio`の設定などが含まれます。 |
| CREATE_TIME            | ロードジョブが作成された時刻です。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_START_TIME         | ロードジョブのETLステージの開始時刻です。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_FINISH_TIME        | ロードジョブのETLステージの終了時刻です。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_START_TIME        | ロードジョブのLOADINGステージの開始時刻です。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_FINISH_TIME       | ロードジョブのLOADINGステージの終了時刻です。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| JOB_DETAILS            | ロードされたデータの詳細情報です。バイト数やファイル数などが含まれます。 |
| ERROR_MSG              | ロードジョブのエラーメッセージです。ロードジョブにエラーがない場合、`NULL`が返されます。 |
| TRACKING_URL           | ロードジョブで検出された不適格なデータ行にアクセスできるURLです。`curl`コマンドや`wget`コマンドを使用してURLにアクセスし、不適格なデータ行のサンプルを取得することができます。不適格なデータが検出されない場合、`NULL`が返されます。 |
| TRACKING_SQL           | ロードジョブのトラッキングログをクエリするために使用できるSQL文です。ロードジョブに不適格なデータ行が関与していない場合、`NULL`が返されます。 |
| REJECTED_RECORD_PATH   | ロードジョブでフィルタリングされたすべての不適格なデータ行にアクセスできるパスです。不適格なデータ行の数は、ロードジョブで設定された`log_rejected_record_num`パラメータによって決まります。`wget`コマンドを使用してパスにアクセスすることができます。ロードジョブに不適格なデータ行が関与していない場合、`NULL`が返されます。 |
