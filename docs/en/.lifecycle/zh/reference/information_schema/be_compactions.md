---
displayed_sidebar: English
---

# be_compactions

`be_compactions` 提供关于压缩任务的统计信息。

`be_compactions` 中提供以下字段：

|**字段**|**描述**|
|---|---|
|BE_ID|BE 的 ID。|
|CANDIDATES_NUM|压缩任务的候选数量。|
|BASE_COMPACTION_CONCURRENCY|正在运行的基础压缩任务数量。|
|CUMULATIVE_COMPACTION_CONCURRENCY|正在运行的累计压缩任务数量。|
|LATEST_COMPACTION_SCORE|最后一次压缩任务的压缩得分。|
|CANDIDATE_MAX_SCORE|任务候选的最高压缩得分。|
|MANUAL_COMPACTION_CONCURRENCY|正在运行的手动压缩任务数量。|
|MANUAL_COMPACTION_CANDIDATES_NUM|手动压缩任务的候选数量。|