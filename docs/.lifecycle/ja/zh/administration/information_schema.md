---
displayed_sidebar: Chinese
---

# Information Schema

`information_schema` は StarRocks インスタンス内のデータベースです。このデータベースには、システムによって定義された複数のテーブルが含まれており、StarRocks インスタンス内のすべてのオブジェクトに関する豊富なメタデータ情報が格納されています。

v3.2 以降、StarRocks は `information_schema` を通じて External Catalog のメタデータを表示することをサポートしています。

## Information Schema を通じてメタデータ情報を表示する

`information_schema` のテーブルをクエリすることで、StarRocks インスタンス内のメタデータ情報を表示できます。

以下の例では、`tables` テーブルをクエリして、StarRocks 内の `sr_member` という名前のテーブルに関連するメタデータ情報を表示しています。

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

## Information Schema のテーブル

StarRocks は `information_schema` 内の `tables`、`tables_config`、`load_tracking_logs` テーブルが提供するメタデータ情報を最適化し、3.1 バージョンから `loads` テーブルを提供しています：

| **Information Schema テーブル名** | **説明**                                  |
| --------------------------- | ---------------------------------------- |
| [tables](#tables)                      | 一般的なテーブルメタデータ情報を提供します。                     |
| [tables_config](#tables_config)               | StarRocks 固有の追加テーブルメタデータ情報を提供します。     |
| [load_tracking_logs](#load_tracking_logs)          | インポートジョブに関連するエラー情報を提供します。                  |
| [loads](#loads)                       | インポートジョブの結果情報を提供します。3.1 バージョンからサポートされており、現在は [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) と [Insert](../sql-reference/sql-statements/data-manipulation/INSERT.md) インポートジョブの結果情報のみを表示できます。                  |

### loads

`loads` テーブルは以下のフィールドを提供します：

| **フィールド**             | **説明**                                                     |
| -------------------- | ------------------------------------------------------------ |
| JOB_ID               | インポートジョブの ID。StarRocks によって自動生成されます。                       |
| LABEL                | インポートジョブのラベル。                                             |
| DATABASE_NAME        | 対象の StarRocks テーブルが存在するデータベース名。                        |
| STATE                | インポートジョブの状態。以下を含みます：<ul><li>`PENDING`：インポートジョブが作成されました。</li><li>`QUEUEING`：インポートジョブが実行待ちです。</li><li>`LOADING`：インポートジョブが実行中です。</li><li>`PREPARED`：トランザクションがコミットされました。</li><li>`FINISHED`：インポートジョブが成功しました。</li><li>`CANCELLED`：インポートジョブが失敗しました。</li></ul>非同期インポートについては[こちら](../loading/Loading_intro.md#非同期インポート)を参照してください。 |
| PROGRESS             | インポートジョブの ETL フェーズと LOADING フェーズの進捗。                     |
| TYPE                 | インポートジョブのタイプ。Broker Load インポートの場合は `BROKER` を、INSERT インポートの場合は `INSERT` を返します。 |
| PRIORITY             | インポートジョブの優先度。`HIGHEST`、`HIGH`、`NORMAL`、`LOW`、`LOWEST` の範囲です。 |
| SCAN_ROWS            | スキャンされたデータ行の総数。                                           |
| FILTERED_ROWS        | データ品質が不適合でフィルタリングされたエラーデータ行の総数。                   |
| UNSELECTED_ROWS      | WHERE 句で指定された条件に基づいてフィルタリングされたデータ行の総数。              |
| SINK_ROWS            | インポートが完了したデータ行の総数。                                       |
| ETL_INFO             | ETL の詳細。このフィールドは Spark Load ジョブにのみ有効です。他のタイプのインポートジョブの場合は結果が空になります。 |
| TASK_INFO            | インポートジョブの詳細。例えば、インポートジョブのタイムアウト時間 (`timeout`) や最大許容比率 (`max_filter_ratio`) など。 |
| CREATE_TIME          | インポートジョブの作成時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_START_TIME       | ETL フェーズの開始時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| ETL_FINISH_TIME      | ETL フェーズの終了時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_START_TIME      | LOADING フェーズの開始時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| LOAD_FINISH_TIME     | LOADING フェーズの終了時間。形式：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |
| JOB_DETAILS          | インポートデータの詳細。インポートされたデータのサイズ（バイト単位）、ファイル数などを含みます。 |
| ERROR_MSG            | インポートジョブのエラーメッセージ。エラーがない場合は `NULL` を返します。            |
| TRACKING_URL         | 品質が不適合なデータのサンプリングにアクセスするための URL。`curl` コマンドまたは `wget` コマンドを使用してアクセスできます。品質が不適合なデータがインポートジョブに存在しない場合は `NULL` を返します。 |
| TRACKING_SQL         | インポートジョブのトラッキングログのクエリ文。不適合なデータが含まれているインポートジョブの場合のみクエリ文が返されます。不適合なデータが含まれていない場合は `NULL` を返します。 |
| REJECTED_RECORD_PATH | 品質が不適合なデータのアクセスパス。不適合なデータの結果数は、インポートジョブの `log_rejected_record_num` パラメータの設定によって決まります。`wget` コマンドを使用してアクセスできます。品質が不適合なデータがインポートジョブに存在しない場合は `NULL` を返します。 |

### tables

`tables` テーブルは以下のフィールドを提供します：

| **フィールド**        | **説明**                                                     |
| --------------- | ------------------------------------------------------------ |
| TABLE_CATALOG   | テーブルが属する Catalog 名。                                      |
| TABLE_SCHEMA    | テーブルが属するデータベース名。                                         |
| TABLE_NAME      | テーブル名。                                                       |
| TABLE_TYPE      | テーブルのタイプ。有効な値は「BASE TABLE」または「VIEW」です。                  |
| ENGINE          | テーブルのエンジンタイプ。有効な値は「StarRocks」、「MySQL」、「MEMORY」、または空文字列です。 |
| VERSION         | このフィールドは現在使用できません。                                             |
| ROW_FORMAT      | このフィールドは現在使用できません。                                             |
| TABLE_ROWS      | テーブルの行数。                                                   |
| AVG_ROW_LENGTH  | テーブルの平均行長（サイズ）。`DATA_LENGTH` / `TABLE_ROWS` で計算されます。単位はバイトです。 |
| DATA_LENGTH     | テーブルのデータファイルの長さ（サイズ）。単位はバイトです。                         |
| MAX_DATA_LENGTH | このフィールドは現在使用できません。                                             |
| INDEX_LENGTH    | このフィールドは現在使用できません。                                             |
| DATA_FREE       | このフィールドは現在使用できません。                                             |
| AUTO_INCREMENT  | このフィールドは現在使用できません。                                             |
| CREATE_TIME     | テーブルの作成時間。                                               |
| UPDATE_TIME     | テーブルの最後の更新時間。                                       |
| CHECK_TIME      | テーブルの最後の一貫性チェック時間。                           |
| TABLE_COLLATION | テーブルのデフォルトの照合順序。                                         |
| CHECKSUM        | このフィールドは現在使用できません。                                             |
| CREATE_OPTIONS  | このフィールドは現在使用できません。                                             |
| TABLE_COMMENT   | テーブルのコメント。                                               |

### tables_config

`tables_config` テーブルは以下のフィールドを提供します：

| **フィールド**         | **説明**                                                     |
| ---------------- | ------------------------------------------------------------ |
| TABLE_SCHEMA     | テーブルが属するデータベース名。                                         |
| TABLE_NAME       | テーブル名。                                                       |
| TABLE_ENGINE     | テーブルのエンジンタイプ。                                               |
| TABLE_MODEL      | テーブルのデータモデル。有効な値は「DUP_KEYS」、「AGG_KEYS」、「UNQ_KEYS」、または「PRI_KEYS」です。 |
| PRIMARY_KEY      | 主キーモデルまたは更新モデルテーブルの主キー。該当しない場合は空文字列が返されます。 |
| PARTITION_KEY    | テーブルのパーティションキー。                                                 |
| DISTRIBUTE_KEY   | テーブルのバケットキー。                                                 |
| DISTRIBUTE_TYPE  | テーブルのバケット方式。                                               |
| DISTRIBUTE_BUCKET | テーブルのバケット数。                                                 |
| SORT_KEY         | テーブルのソートキー。                                                 |
| PROPERTIES       | テーブルのプロパティ。                                                   |
| TABLE_ID         | テーブルの ID。                                                   |

### load_tracking_logs

この機能は StarRocks v3.0 からサポートされています。

`load_tracking_logs` テーブルは以下のフィールドを提供します：

| **フィールド**         | **説明**                                                     |
| ---------------- | ----------------------------------------------------------- |
| JOB_ID           | インポートジョブの ID。                                               |
| LABEL            | インポートジョブのラベル。                                            |
| DATABASE_NAME    | インポートジョブが属するデータベース名。                                      |
| TRACKING_LOG     | インポートジョブのエラーログ情報（存在する場合）。                                 |

### materialized_views

`materialized_views` テーブルは以下のフィールドを提供します：

| **フィールド**                             | **説明**                                         |
| ------------------------------------ | ------------------------------------------------ |
| MATERIALIZED_VIEW_ID                 | マテリアライズドビューの ID                                      |
| TABLE_SCHEMA                         | マテリアライズドビューが存在するデータベース名                         |
| TABLE_NAME                           | マテリアライズドビューの名前                                     |
| REFRESH_TYPE                         | リフレッシュタイプ。`ROLLUP`、`ASYNC`、`MANUAL` を含みます。    |
| IS_ACTIVE                            | アクティブかどうか。非アクティブなマテリアライズドビューはリフレッシュやクエリ改写されません。          |
| INACTIVE_REASON                      | 非アクティブの理由                                       |
| PARTITION_TYPE                       | マテリアライズドビューのパーティションタイプ                                 |
| TASK_ID                              | マテリアライズドビューのリフレッシュタスクの ID                            |
| TASK_NAME                            | マテリアライズドビューのリフレッシュタスクの名前                           |
| LAST_REFRESH_START_TIME              | 最後のリフレッシュタスクの開始時間                       |
| LAST_REFRESH_FINISHED_TIME           | 最後のリフレッシュタスクの終了時間                       |
| LAST_REFRESH_DURATION                | 最後のリフレッシュタスクの持続時間                       |
| LAST_REFRESH_STATE                   | 最後のリフレッシュタスクの状態                           |
| LAST_REFRESH_FORCE_REFRESH           | 最後のリフレッシュタスクが強制リフレッシュかどうか                     |
| LAST_REFRESH_START_PARTITION         | 最後のリフレッシュタスクの開始パーティション                       |
| LAST_REFRESH_END_PARTITION           | 最後のリフレッシュタスクの終了パーティション                       |
| LAST_REFRESH_BASE_REFRESH_PARTITIONS | 最後のリフレッシュタスクのベーステーブルのパーティション                       |
| LAST_REFRESH_MV_REFRESH_PARTITIONS   | 最後のリフレッシュタスクでリフレッシュされたパーティション                     |
| LAST_REFRESH_ERROR_CODE              | 最後のリフレッシュタスクのエラーコード                         |
| LAST_REFRESH_ERROR_MESSAGE           | 最後のリフレッシュタスクのエラーメッセージ                       |
| TABLE_ROWS                           | マテリアライズドビューのデータ行数。バックグラウンドで統計された近似値です。             |
| MATERIALIZED_VIEW_DEFINITION         | マテリアライズドビューの SQL 定義                              |
