---
displayed_sidebar: English
---

# materialized_views

`materialized_views` は、すべての非同期マテリアライズドビューに関する情報を提供します。

`materialized_views` には、以下のフィールドが含まれています：

| **フィールド**                            | **説明**                                              |
| ------------------------------------ | ------------------------------------------------------------ |
| MATERIALIZED_VIEW_ID                 | マテリアライズドビューのID。                                 |
| TABLE_SCHEMA                         | マテリアライズドビューが存在するデータベース。             |
| TABLE_NAME                           | マテリアライズドビューの名前。                               |
| REFRESH_TYPE                         | マテリアライズドビューのリフレッシュタイプ。有効な値は `ROLLUP`、`ASYNC`、`MANUAL`です。 |
| IS_ACTIVE                            | マテリアライズドビューがアクティブかどうかを示します。非アクティブなマテリアライズドビューは、リフレッシュやクエリができません。 |
| INACTIVE_REASON                      | マテリアライズドビューが非アクティブである理由。           |
| PARTITION_TYPE                       | マテリアライズドビューのパーティションタイプ。     |
| TASK_ID                              | マテリアライズドビューのリフレッシュを担当するタスクのID。 |
| TASK_NAME                            | マテリアライズドビューのリフレッシュを担当するタスクの名前。 |
| LAST_REFRESH_START_TIME              | 最新のリフレッシュタスクの開始時刻。                  |
| LAST_REFRESH_FINISHED_TIME           | 最新のリフレッシュタスクの終了時刻。                    |
| LAST_REFRESH_DURATION                | 最新のリフレッシュタスクの所要時間。                    |
| LAST_REFRESH_STATE                   | 最新のリフレッシュタスクの状態。                       |
| LAST_REFRESH_FORCE_REFRESH           | 最新のリフレッシュタスクが強制リフレッシュだったかどうかを示します。 |
| LAST_REFRESH_START_PARTITION         | 最新のリフレッシュタスクの開始パーティション。         |
| LAST_REFRESH_END_PARTITION           | 最新のリフレッシュタスクの終了パーティション。           |
| LAST_REFRESH_BASE_REFRESH_PARTITIONS | 最新のリフレッシュタスクに関連するベーステーブルのパーティション。 |
| LAST_REFRESH_MV_REFRESH_PARTITIONS   | 最新のリフレッシュタスクでリフレッシュされたマテリアライズドビューのパーティション。 |
| LAST_REFRESH_ERROR_CODE              | 最新のリフレッシュタスクのエラーコード。                  |
| LAST_REFRESH_ERROR_MESSAGE           | 最新のリフレッシュタスクのエラーメッセージ。               |
| TABLE_ROWS                           | マテリアライズドビューのデータ行数（概算のバックグラウンド統計に基づく）。 |
| MATERIALIZED_VIEW_DEFINITION         | マテリアライズドビューのSQL定義。                     |
