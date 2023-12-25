---
displayed_sidebar: Chinese
---

# materialized_views

`materialized_views` は、すべての非同期マテリアライズドビューに関する情報を提供します。

`materialized_views` は以下のフィールドを提供します：

| **フィールド**                       | **説明**                                         |
| ------------------------------------ | ------------------------------------------------ |
| MATERIALIZED_VIEW_ID                 | マテリアライズドビューのID。                     |
| TABLE_SCHEMA                         | マテリアライズドビューが存在するデータベース名。 |
| TABLE_NAME                           | マテリアライズドビューの名前。                   |
| REFRESH_TYPE                         | リフレッシュタイプ。`ROLLUP`、`ASYNC`、`MANUAL`が含まれます。 |
| IS_ACTIVE                            | アクティブかどうか。非アクティブなマテリアライズドビューはリフレッシュやクエリ書き換えが行われません。 |
| INACTIVE_REASON                      | 非アクティブの理由。                             |
| PARTITION_TYPE                       | マテリアライズドビューのパーティションタイプ。   |
| TASK_ID                              | マテリアライズドビューのリフレッシュタスクID。   |
| TASK_NAME                            | マテリアライズドビューのリフレッシュタスク名。   |
| LAST_REFRESH_START_TIME              | 最後のリフレッシュタスクが開始された時間。       |
| LAST_REFRESH_FINISHED_TIME           | 最後のリフレッシュタスクが終了した時間。         |
| LAST_REFRESH_DURATION                | 最後のリフレッシュタスクの持続時間。             |
| LAST_REFRESH_STATE                   | 最後のリフレッシュタスクの状態。                 |
| LAST_REFRESH_FORCE_REFRESH           | 最後のリフレッシュタスクが強制リフレッシュされたかどうか。 |
| LAST_REFRESH_START_PARTITION         | 最後のリフレッシュタスクが開始されたパーティション。 |
| LAST_REFRESH_END_PARTITION           | 最後のリフレッシュタスクが終了したパーティション。 |
| LAST_REFRESH_BASE_REFRESH_PARTITIONS | 最後のリフレッシュタスクのベーステーブルパーティション。 |
| LAST_REFRESH_MV_REFRESH_PARTITIONS   | 最後のリフレッシュタスクでリフレッシュされたパーティション。 |
| LAST_REFRESH_ERROR_CODE              | 最後のリフレッシュタスクのエラーコード。         |
| LAST_REFRESH_ERROR_MESSAGE           | 最後のリフレッシュタスクのエラーメッセージ。     |
| TABLE_ROWS                           | マテリアライズドビューの行数。バックグラウンドで統計された近似値です。 |
| MATERIALIZED_VIEW_DEFINITION         | マテリアライズドビューのSQL定義。                |
