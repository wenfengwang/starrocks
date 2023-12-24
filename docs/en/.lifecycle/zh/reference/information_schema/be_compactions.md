---
displayed_sidebar: English
---

# be_compactions

`be_compactions` 提供压缩任务的统计信息。

`be_compactions` 中提供以下字段：

| **字段**                         | **描述**                                         |
| --------------------------------- | ------------------------------------------------------- |
| BE_ID                             | BE 的 ID。                                           |
| CANDIDATES_NUM                    | 压缩任务的候选者数。              |
| BASE_COMPACTION_CONCURRENCY       | 正在运行的基本压缩任务数。       |
| CUMULATIVE_COMPACTION_CONCURRENCY | 正在运行的累积压缩任务数。 |
| LATEST_COMPACTION_SCORE           | 上一个压缩任务的压缩分数。           |
| CANDIDATE_MAX_SCORE               | 任务候选者的最大压缩得分。     |
| MANUAL_COMPACTION_CONCURRENCY     | 正在运行的手动压缩任务数。     |
| MANUAL_COMPACTION_CANDIDATES_NUM  | 手动压缩任务的候选者数。       |
