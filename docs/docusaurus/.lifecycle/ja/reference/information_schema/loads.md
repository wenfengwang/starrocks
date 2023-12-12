---
displayed_sidebar: "英語"
---

# loads

`loads`はロードジョブの結果を提供します。このビューはStarRocks v3.1以降でサポートされています。現在、このビューで確認できるのは[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)と[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ジョブの結果のみです。

以下のフィールドが`loads`で提供されます。

| **フィールド**            | **説明**                                              |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | StarRocksによって割り当てられたロードジョブを識別するための一意のID。 |
| LABEL                | ロードジョブのラベル。                                   |
| DATABASE_NAME        | 対象のStarRocksテーブルが属するデータベースの名前。 |
| STATE                | ロードジョブの状態。有効な値は：<ul><li>`PENDING`: ロードジョブが作成されました。</li><li>`QUEUEING`: ロードジョブがスケジュールされるのを待っているキューにあります。</li><li>`LOADING`: ロードジョブが実行中です。</li><li>`PREPARED`: トランザクションがコミットされました。</li><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul>詳しくは[非同期ローディング](../../loading/Loading_intro.md#asynchronous-loading)を参照してください。 |
| PROGRESS             | ロードジョブのETLステージとLOADINGステージの進行状況。 |
| TYPE                 | ロードジョブのタイプ。Broker Loadの場合、返り値は`BROKER`です。INSERTの場合は`INSERT`となります。 |
| PRIORITY             | ロードジョブの優先度。有効な値は`HIGHEST`, `HIGH`, `NORMAL`, `LOW`, `LOWEST`。 |
| SCAN_ROWS            | スキャンされたデータ行の数。                    |
| FILTERED_ROWS        | データの品質が不十分であるためにフィルタリングされたデータ行の数。 |
| UNSELECTED_ROWS      | WHERE節で指定された条件によりフィルタリングされたデータ行の数。 |
| SINK_ROWS            | ロードされたデータ行の数。                     |
| ETL_INFO             | ロードジョブのETLの詳細。Spark Loadの場合に限り空でない値が返されます。その他のロードジョブタイプでは、空の値が返されます。 |
| TASK_INFO            | ロードジョブのタスク実行の詳細、例えば`timeout`や`max_filter_ratio`の設定など。 |
| CREATE_TIME          | ロードジョブが作成された時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_START_TIME       | ロードジョブのETLステージの開始時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_FINISH_TIME      | ロードジョブのETLステージの終了時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_START_TIME      | ロードジョブのLOADINGステージの開始時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_FINISH_TIME     | ロードジョブのLOADINGステージの終了時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| JOB_DETAILS          | ロードされたデータの詳細、例えばバイト数やファイル数など。 |
| ERROR_MSG            | ロードジョブのエラーメッセージ。ロードジョブがエラーに遭遇していない場合、`NULL`が返されます。 |
| TRACKING_URL         | ロードジョブで検出された不適格データ行サンプルにアクセスできるURL。`curl`または`wget`コマンドを使ってURLにアクセスし、不適格データ行サンプルを取得できます。不適格データが検出されない場合、`NULL`が返されます。 |
| TRACKING_SQL         | ロードジョブの追跡ログを問い合わせるために使用できるSQL文。ロードジョブに不適格データ行が含まれている場合のみSQL文が返されます。ロードジョブに不適格データ行が含まれていない場合、`NULL`が返されます。 |
| REJECTED_RECORD_PATH | ロードジョブでフィルタリングされたすべての不適格データ行にアクセスできるパス。ログに記録される不適格データ行の数は、ロードジョブで設定されている`log_rejected_record_num`パラメータによって決まります。`wget`コマンドを使ってパスにアクセスできます。ロードジョブに不適格データ行が含まれていない場合、`NULL`が返されます。 |