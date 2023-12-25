---
displayed_sidebar: Chinese
---

# be_compactions

`be_compactions` はCompactionタスクの統計情報を提供します。

`be_compactions` は以下のフィールドを提供します：

| **フィールド**                    | **説明**                                      |
| --------------------------------- | --------------------------------------------- |
| BE_ID                             | BEのIDです。                                  |
| CANDIDATES_NUM                    | 現在実行待ちのCompactionタスクの数です。      |
| BASE_COMPACTION_CONCURRENCY       | 現在実行中のBase Compactionタスクの数です。   |
| CUMULATIVE_COMPACTION_CONCURRENCY | 現在実行中のCumulative Compactionタスクの数です。 |
| LATEST_COMPACTION_SCORE           | 最後のCompactionタスクのCompaction Scoreです。 |
| CANDIDATE_MAX_SCORE               | 実行待ちのタスクの最大Compaction Scoreです。   |
| MANUAL_COMPACTION_CONCURRENCY     | 現在実行中の手動Compactionタスクの数です。    |
| MANUAL_COMPACTION_CANDIDATES_NUM  | 現在実行待ちの手動Compactionタスクの数です。  |
