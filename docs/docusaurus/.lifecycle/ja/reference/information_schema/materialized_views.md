---
displayed_sidebar: "Japanese"
---

# materialized_views

`materialized_views`はすべての非同期マテリアライズド・ビューに関する情報を提供します。

`materialized_views`には、以下のフィールドが提供されます:

| **Field**                            | **Description**                                              |
| ------------------------------------ | ------------------------------------------------------------ |
| MATERIALIZED_VIEW_ID                 | マテリアライズド・ビューのID。                                 |
| TABLE_SCHEMA                         | マテリアライズド・ビューが存在するデータベース。             |
| TABLE_NAME                           | マテリアライズド・ビューの名前。                              |
| REFRESH_TYPE                         | マテリアライズド・ビューの更新タイプ。有効な値: `ROLLUP`, `ASYNC`, `MANUAL`。 |
| IS_ACTIVE                            | マテリアライズド・ビューがアクティブかどうかを示します。非アクティブなマテリアライズド・ビューは更新またはクエリできません。 |
| INACTIVE_REASON                      | マテリアライズド・ビューが非アクティブである理由。           |
| PARTITION_TYPE                       | マテリアライズド・ビューのパーティショニング戦略の種類。     |
| TASK_ID                              | マテリアライズド・ビューを更新するタスクのID。               |
| TASK_NAME                            | マテリアライズド・ビューを更新するタスクの名前。             |
| LAST_REFRESH_START_TIME              | 最近の更新タスクの開始時間。                                 |
| LAST_REFRESH_FINISHED_TIME           | 最近の更新タスクの終了時間。                                 |
| LAST_REFRESH_DURATION                | 最近の更新タスクの実行時間。                                 |
| LAST_REFRESH_STATE                   | 最近の更新タスクの状態。                                     |
| LAST_REFRESH_FORCE_REFRESH           | 最近の更新タスクが強制更新であるかどうかを示します。         |
| LAST_REFRESH_START_PARTITION         | 最近の更新タスクの開始パーティション。                       |
| LAST_REFRESH_END_PARTITION           | 最近の更新タスクの終了パーティション。                       |
| LAST_REFRESH_BASE_REFRESH_PARTITIONS | 最近の更新タスクに関与する基本テーブルのパーティション。       |
| LAST_REFRESH_MV_REFRESH_PARTITIONS   | 最近の更新タスクで更新されたマテリアライズド・ビューのパーティション。 |
| LAST_REFRESH_ERROR_CODE              | 最近の更新タスクのエラーコード。                             |
| LAST_REFRESH_ERROR_MESSAGE           | 最近の更新タスクのエラーメッセージ。                         |
| TABLE_ROWS                           | マテリアライズド・ビュー内のデータ行数（おおよそのバックグラウンド統計に基づく）。 |
| MATERIALIZED_VIEW_DEFINITION         | マテリアライズド・ビューのSQL定義。                           |