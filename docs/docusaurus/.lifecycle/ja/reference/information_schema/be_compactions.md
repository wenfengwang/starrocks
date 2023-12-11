---
displayed_sidebar: "Japanese"
---

# be_compactions

`be_compactions`はコンパクションタスクの統計情報を提供します。

`be_compactions`には、次のフィールドが提供されます：

| **フィールド**                    | **説明**                                               |
| --------------------------------- | ------------------------------------------------------- |
| BE_ID                             | BEのID。                                                |
| CANDIDATES_NUM                    | コンパクションタスクの候補数。                          |
| BASE_COMPACTION_CONCURRENCY       | 実行中のベースコンパクションタスクの数。                |
| CUMULATIVE_COMPACTION_CONCURRENCY | 実行中の累積コンパクションタスクの数。                  |
| LATEST_COMPACTION_SCORE           | 直近のコンパクションタスクのコンパクションスコア。      |
| CANDIDATE_MAX_SCORE               | タスク候補の最大コンパクションスコア。                  |
| MANUAL_COMPACTION_CONCURRENCY     | 実行中のマニュアルコンパクションタスクの数。            |
| MANUAL_COMPACTION_CANDIDATES_NUM  | マニュアルコンパクションタスクの候補数。                |