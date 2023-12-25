---
displayed_sidebar: English
---

# be_compactions

`be_compactions` はコンパクションタスクに関する統計情報を提供します。

`be_compactions` で提供されるフィールドは以下の通りです：

| **フィールド**                     | **説明**                                               |
| --------------------------------- | ------------------------------------------------------- |
| BE_ID                             | BEのIDです。                                           |
| CANDIDATES_NUM                    | コンパクションタスクの候補数です。                      |
| BASE_COMPACTION_CONCURRENCY       | 実行中のベースコンパクションタスクの数です。           |
| CUMULATIVE_COMPACTION_CONCURRENCY | 実行中の累積コンパクションタスクの数です。             |
| LATEST_COMPACTION_SCORE           | 最新のコンパクションタスクのスコアです。               |
| CANDIDATE_MAX_SCORE               | タスク候補の最大コンパクションスコアです。             |
| MANUAL_COMPACTION_CONCURRENCY     | 実行中の手動コンパクションタスクの数です。             |
| MANUAL_COMPACTION_CANDIDATES_NUM  | 手動コンパクションタスクの候補数です。                 |
