---
displayed_sidebar: English
---

# ロード

`loads` はロードジョブの結果を提供します。このビューはStarRocks v3.1以降でサポートされています。現在、このビューから確認できるのは [Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) と [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) ジョブの結果のみです。

`loads` には以下のフィールドが提供されています：

| **フィールド**            | **説明**                                              |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | ロードジョブを識別するためにStarRocksによって割り当てられた一意のID。 |
| LABEL                | ロードジョブのラベル。                                   |
| DATABASE_NAME        | 宛先StarRocksテーブルが属するデータベースの名前。 |
| STATE                | ロードジョブの状態。有効な値：<ul><li>`PENDING`：ロードジョブが作成されました。</li><li>`QUEUEING`：ロードジョブがキューに入ってスケジュールを待っています。</li><li>`LOADING`：ロードジョブが実行中です。</li><li>`PREPARED`：トランザクションがコミットされました。</li><li>`FINISHED`：ロードジョブが成功しました。</li><li>`CANCELLED`：ロードジョブが失敗しました。</li></ul>詳細については、[非同期ロード](../../loading/Loading_intro.md#asynchronous-loading)を参照してください。 |
| PROGRESS             | ロードジョブのETLステージとLOADINGステージの進捗状況。 |
| TYPE                 | ロードジョブのタイプ。Broker Loadの場合、戻り値は`BROKER`。INSERTの場合、戻り値は`INSERT`。 |
| PRIORITY             | ロードジョブの優先度。有効な値：`HIGHEST`、`HIGH`、`NORMAL`、`LOW`、`LOWEST`。 |
| SCAN_ROWS            | スキャンされたデータ行の数。                    |
| FILTERED_ROWS        | データ品質が不十分でフィルタリングされたデータ行の数。 |
| UNSELECTED_ROWS      | WHERE句で指定された条件によりフィルタリングされたデータ行の数。 |
| SINK_ROWS            | ロードされたデータ行の数。                     |
| ETL_INFO             | ロードジョブのETL詳細。Spark Loadに対してのみ非空の値が返されます。他のロードジョブタイプでは空の値が返されます。 |
| TASK_INFO            | ロードジョブのタスク実行詳細、例えば`timeout`や`max_filter_ratio`の設定など。 |
| CREATE_TIME          | ロードジョブが作成された時刻。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_START_TIME       | ロードジョブのETLステージの開始時刻。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_FINISH_TIME      | ロードジョブのETLステージの終了時刻。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_START_TIME      | ロードジョブのLOADINGステージの開始時刻。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_FINISH_TIME     | ロードジョブのLOADINGステージの終了時刻。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| JOB_DETAILS          | ロードされたデータの詳細、例えばバイト数やファイル数など。 |
| ERROR_MSG            | ロードジョブのエラーメッセージ。エラーが発生していない場合は`NULL`が返されます。 |
| TRACKING_URL         | ロードジョブで検出された非適格データ行サンプルにアクセスできるURL。`curl`または`wget`コマンドを使用してURLにアクセスし、非適格データ行サンプルを取得できます。非適格データが検出されなかった場合は`NULL`が返されます。 |
| TRACKING_SQL         | ロードジョブのトラッキングログを照会するために使用できるSQLステートメント。非適格データ行が含まれるロードジョブの場合のみSQLステートメントが返されます。非適格データ行が含まれていない場合は`NULL`が返されます。 |
| REJECTED_RECORD_PATH | ロードジョブでフィルタリングされたすべての非適格データ行にアクセスできるパス。ログに記録される非適格データ行の数は、ロードジョブで設定された`log_rejected_record_num`パラメータによって決定されます。`wget`コマンドを使用してパスにアクセスできます。非適格データ行が含まれていない場合は`NULL`が返されます。 |
