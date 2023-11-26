---
displayed_sidebar: "Japanese"
---

# materialized_views

`materialized_views`はすべての非同期マテリアライズドビューに関する情報を提供します。

`materialized_views`には以下のフィールドが提供されます：

| **フィールド**                          | **説明**                                                      |
| ------------------------------------ | ------------------------------------------------------------ |
| MATERIALIZED_VIEW_ID                 | マテリアライズドビューのID。                                 |
| TABLE_SCHEMA                         | マテリアライズドビューが存在するデータベース。             |
| TABLE_NAME                           | マテリアライズドビューの名前。                               |
| REFRESH_TYPE                         | マテリアライズドビューのリフレッシュタイプ。有効な値：`ROLLUP`、`ASYNC`、`MANUAL`。 |
| IS_ACTIVE                            | マテリアライズドビューがアクティブかどうかを示します。非アクティブなマテリアライズドビューはリフレッシュやクエリを実行できません。 |
| INACTIVE_REASON                      | マテリアライズドビューが非アクティブな理由。           |
| PARTITION_TYPE                       | マテリアライズドビューのパーティショニング戦略のタイプ。     |
| TASK_ID                              | マテリアライズドビューのリフレッシュを担当するタスクのID。 |
| TASK_NAME                            | マテリアライズドビューのリフレッシュを担当するタスクの名前。 |
| LAST_REFRESH_START_TIME              | 直近のリフレッシュタスクの開始時刻。                  |
| LAST_REFRESH_FINISHED_TIME           | 直近のリフレッシュタスクの終了時刻。                    |
| LAST_REFRESH_DURATION                | 直近のリフレッシュタスクの所要時間。                    |
| LAST_REFRESH_STATE                   | 直近のリフレッシュタスクの状態。                       |
| LAST_REFRESH_FORCE_REFRESH           | 直近のリフレッシュタスクが強制リフレッシュかどうかを示します。 |
| LAST_REFRESH_START_PARTITION         | 直近のリフレッシュタスクの開始パーティション。         |
| LAST_REFRESH_END_PARTITION           | 直近のリフレッシュタスクの終了パーティション。           |
| LAST_REFRESH_BASE_REFRESH_PARTITIONS | 直近のリフレッシュタスクに関与するベーステーブルのパーティション。 |
| LAST_REFRESH_MV_REFRESH_PARTITIONS   | 直近のリフレッシュタスクでリフレッシュされたマテリアライズドビューのパーティション。 |
| LAST_REFRESH_ERROR_CODE              | 直近のリフレッシュタスクのエラーコード。                  |
| LAST_REFRESH_ERROR_MESSAGE           | 直近のリフレッシュタスクのエラーメッセージ。               |
| TABLE_ROWS                           | マテリアライズドビューのデータ行数。背景統計に基づくおおよその値です。 |
| MATERIALIZED_VIEW_DEFINITION         | マテリアライズドビューのSQL定義。                     |
