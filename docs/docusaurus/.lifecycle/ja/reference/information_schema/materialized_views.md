---
displayed_sidebar: "Japanese"
---

# materialized_views

`materialized_views`はすべての非同期マテリアライズドビューに関する情報を提供します。

`materialized_views`には以下のフィールドが提供されます:

| **フィールド**                             | **説明**                                                     |
| ------------------------------------ | ------------------------------------------------------------ |
| MATERIALIZED_VIEW_ID                 | マテリアライズドビューのID。                                   |
| TABLE_SCHEMA                         | マテリアライズドビューが存在するデータベース。                 |
| TABLE_NAME                           | マテリアライズドビューの名前。                                 |
| REFRESH_TYPE                         | マテリアライズドビューのリフレッシュタイプ。有効な値：`ROLLUP`、`ASYNC`、`MANUAL`。|
| IS_ACTIVE                            | マテリアライズドビューがアクティブであるかどうかを示します。非アクティブなマテリアライズドビューはリフレッシュやクエリを実行できません。|
| INACTIVE_REASON                      | マテリアライズドビューが非アクティブである理由。               |
| PARTITION_TYPE                       | マテリアライズドビューのパーティション戦略のタイプ。               |
| TASK_ID                              | マテリアライズドビューをリフレッシュする責任があるタスクのID。                    |
| TASK_NAME                            | マテリアライズドビューをリフレッシュする責任があるタスクの名前。                 |
| LAST_REFRESH_START_TIME              | 最新のリフレッシュタスクの開始時間。                    |
| LAST_REFRESH_FINISHED_TIME           | 最新のリフレッシュタスクの終了時間。                    |
| LAST_REFRESH_DURATION                | 最新のリフレッシュタスクの所要時間。                    |
| LAST_REFRESH_STATE                   | 最新のリフレッシュタスクの状態。                       |
| LAST_REFRESH_FORCE_REFRESH           | 最新のリフレッシュタスクが強制リフレッシュであるかどうかを示します。 |
| LAST_REFRESH_START_PARTITION         | 最新のリフレッシュタスクの開始パーティション。         |
| LAST_REFRESH_END_PARTITION           | 最新のリフレッシュタスクの終了パーティション。           |
| LAST_REFRESH_BASE_REFRESH_PARTITIONS | 最新のリフレッシュタスクに関与するベーステーブルのパーティション。 |
| LAST_REFRESH_MV_REFRESH_PARTITIONS   | 最新のリフレッシュタスクでリフレッシュされたマテリアライズドビューのパーティション。 |
| LAST_REFRESH_ERROR_CODE              | 最新のリフレッシュタスクのエラーコード。                  |
| LAST_REFRESH_ERROR_MESSAGE           | 最新のリフレッシュタスクのエラーメッセージ。               |
| TABLE_ROWS                           | マテリアライズドビュー内のデータ行数(おおよそのバックグラウンド統計に基づく)。 |
| MATERIALIZED_VIEW_DEFINITION         | マテリアライズドビューのSQL定義。                     |