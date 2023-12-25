---
displayed_sidebar: Chinese
---

# ロード

インポートジョブの結果情報を提供します。このビューは StarRocks v3.1 バージョンからサポートされています。現在、[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) と [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) インポートジョブの結果情報のみを表示できます。

`loads` は以下のフィールドを提供します：

| **フィールド**       | **説明**                                                     |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | インポートジョブの ID。StarRocks によって自動生成されます。 |
| LABEL                | インポートジョブのラベル。                                   |
| DATABASE_NAME        | 対象 StarRocks テーブルが存在するデータベースの名前。        |
| STATE                | インポートジョブの状態。以下を含みます：<ul><li>`PENDING`：インポートジョブが作成されました。</li><li>`QUEUEING`：インポートジョブが実行待ちです。</li><li>`LOADING`：インポートジョブが実行中です。</li><li>`PREPARED`：トランザクションがコミットされました。</li><li>`FINISHED`：インポートジョブが成功しました。</li><li>`CANCELLED`：インポートジョブが失敗しました。</li></ul>非同期インポートについては[こちら](../../loading/Loading_intro.md#非同期インポート)を参照してください。 |
| PROGRESS             | インポートジョブの ETL フェーズと LOADING フェーズの進捗。   |
| TYPE                 | インポートジョブのタイプ。Broker Load の場合は `BROKER`、INSERT の場合は `INSERT` を返します。 |
| PRIORITY             | インポートジョブの優先度。`HIGHEST`、`HIGH`、`NORMAL`、`LOW`、`LOWEST` の範囲です。 |
| SCAN_ROWS            | スキャンされたデータ行の総数。                               |
| FILTERED_ROWS        | データ品質が不適合でフィルタリングされたエラーデータ行の総数。 |
| UNSELECTED_ROWS      | WHERE 句で指定された条件によりフィルタリングされたデータ行の総数。 |
| SINK_ROWS            | インポートが完了したデータ行の総数。                         |
| ETL_INFO             | ETL の詳細。このフィールドは Spark Load ジョブにのみ有効です。他のタイプのインポートジョブの場合は結果が空になります。 |
| TASK_INFO            | インポートジョブの詳細。例えば、インポートジョブのタイムアウト時間 (`timeout`) や最大許容フィルタ比率 (`max_filter_ratio`) など。 |
| CREATE_TIME          | インポートジョブの作成時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_START_TIME       | ETL フェーズの開始時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_FINISH_TIME      | ETL フェーズの終了時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_START_TIME      | LOADING フェーズの開始時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_FINISH_TIME     | LOADING フェーズの終了時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| JOB_DETAILS          | インポートデータの詳細。インポートされたデータのサイズ（バイト単位）、ファイル数などを含みます。 |
| ERROR_MSG            | インポートジョブのエラーメッセージ。エラーがない場合は `NULL` を返します。 |
| TRACKING_URL         | 品質が不適合なデータのサンプルにアクセスするための URL。`curl` コマンドまたは `wget` コマンドでアクセスできます。品質が不適合なデータがインポートジョブに存在しない場合は `NULL` を返します。 |
| TRACKING_SQL         | インポートジョブのトラッキングログを照会する SQL 文。不適合なデータが含まれているインポートジョブの場合のみ、照会文が返されます。不適合なデータが含まれていない場合は `NULL` を返します。 |
| REJECTED_RECORD_PATH | 品質が不適合なデータにアクセスするための URL。返される不適合なデータの結果数は、インポートジョブの `log_rejected_record_num` パラメータの設定によって決まります。`wget` コマンドでアクセスできます。品質が不適合なデータがインポートジョブに存在しない場合は `NULL` を返します。 |
