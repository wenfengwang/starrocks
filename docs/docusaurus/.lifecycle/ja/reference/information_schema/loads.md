---
displayed_sidebar: "Japanese"
---

# loads

`loads`は、ロードジョブの結果を提供します。このビューはStarRocks v3.1以降でサポートされています。現在、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)および[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)の結果のみを表示することができます。

`loads`には次のフィールドが提供されています:

| **フィールド**         | **説明**                                                     |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | StarRocksがロードジョブを識別するために割り当てた一意のIDです。           |
| LABEL                | ロードジョブのラベルです。                                       |
| DATABASE_NAME        | 目的のStarRocksテーブルが属するデータベースの名前です。                          |
| STATE                | ロードジョブの状態です。 有効な値: <ul><li>`PENDING`: ロードジョブが作成されました。</li><li>`QUEUEING`: ロードジョブはスケジュール待ちのキューにあります。</li><li>`LOADING`: ロードジョブが実行されています。</li><li>`PREPARED`: トランザクションが確定されました。</li><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul>詳細については、[非同期ロード](../../loading/Loading_intro.md#asynchronous-loading)を参照してください。 |
| PROGRESS             | ロードジョブのETLステージとLOADINGステージの進捗です。                           |
| TYPE                 | ロードジョブの種類です。 Broker Loadの場合、返り値は`BROKER`です。INSERTの場合、返り値は`INSERT`です。             |
| PRIORITY             | ロードジョブの優先度です。 有効な値: `HIGHEST`, `HIGH`, `NORMAL`, `LOW`, および `LOWEST`。 |
| SCAN_ROWS            | スキャンされたデータ行の数です。                                       |
| FILTERED_ROWS        | データ品質が不十分でフィルタリングされたデータ行の数です。                        |
| UNSELECTED_ROWS      | WHERE句で指定された条件によりフィルタリングされたデータ行の数です。                    |
| SINK_ROWS            | ロードされたデータ行の数です。                                       |
| ETL_INFO             | ロードジョブのETLの詳細です。Spark Loadの場合のみ非空の値が返されます。他のタイプのロードジョブでは、空の値が返されます。 |
| TASK_INFO            | ロードジョブのタスク実行の詳細です。`timeout`および`max_filter_ratio`の設定などが含まれます。 |
| CREATE_TIME          | ロードジョブが作成された時刻です。フォーマット:`yyyy-MM-dd HH:mm:ss`。例:`2023-07-24 14:58:58`。       |
| ETL_START_TIME       | ロードジョブのETLステージの開始時刻です。フォーマット:`yyyy-MM-dd HH:mm:ss`。例:`2023-07-24 14:58:58`。     |
| ETL_FINISH_TIME      | ロードジョブのETLステージの終了時刻です。フォーマット:`yyyy-MM-dd HH:mm:ss`。例:`2023-07-24 14:58:58`。     |
| LOAD_START_TIME      | ロードジョブのLOADINGステージの開始時刻です。フォーマット:`yyyy-MM-dd HH:mm:ss`。例:`2023-07-24 14:58:58`。   |
| LOAD_FINISH_TIME     | ロードジョブのLOADINGステージの終了時刻です。フォーマット:`yyyy-MM-dd HH:mm:ss`。例:`2023-07-24 14:58:58`。 |
| JOB_DETAILS          | ロードされたデータの詳細です。バイト数やファイル数などが含まれます。                       |
| ERROR_MSG            | ロードジョブのエラーメッセージです。ロードジョブでエラーが発生しない場合、`NULL`が返されます。                 |
| TRACKING_URL         | ロードジョブで検出された不適格なデータ行のサンプルにアクセスするためのURLです。`curl`または`wget`コマンドを使用してURLにアクセスし、不適格なデータ行のサンプルを取得できます。不適格なデータが検出されない場合は、`NULL`が返されます。 |
| TRACKING_SQL         | ロードジョブのトラッキングログをクエリするために使用できるSQLステートメントです。不適格なデータ行が関与する場合のみSQLステートメントが返されます。ロードジョブに不適格なデータ行が関与しない場合、`NULL`が返されます。 |
| REJECTED_RECORD_PATH | ロードジョブでフィルタリングされたすべての不適格なデータ行にアクセスできるパスです。パラメータ`log_rejected_record_num`で設定された不適格なデータ行の数によって、ログ記録される不適格なデータ行の数が決まります。`wget`コマンドを使用してパスにアクセスできます。ロードジョブに不適格なデータ行が関与しない場合、`NULL`が返されます。 |
