---
displayed_sidebar: "Chinese"
---

# be_compactions

`be_compactions`提供压实任务的统计信息。

`be_compactions`提供以下字段：

| **字段**                         | **描述**                                          |
| --------------------------------- | ------------------------------------------------------- |
| BE_ID                             | BE的ID。                                           |
| CANDIDATES_NUM                    | 压实任务的候选数。              |
| BASE_COMPACTION_CONCURRENCY       | 正在运行的基本压实任务数。       |
| CUMULATIVE_COMPACTION_CONCURRENCY | 正在运行的累积压实任务数。 |
| LATEST_COMPACTION_SCORE           | 最新压实任务的压实分。           |
| CANDIDATE_MAX_SCORE               | 任务候选的最大压实分。     |
| MANUAL_COMPACTION_CONCURRENCY     | 正在运行的手动压实任务数。     |
| MANUAL_COMPACTION_CANDIDATES_NUM  | 手动压实任务的候选数。       |