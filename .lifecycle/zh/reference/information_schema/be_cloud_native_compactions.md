---
displayed_sidebar: English
---

# be_cloud_native_compactions

be_cloud_native_compactions 提供了在共享数据集群的 CN（或v3.0版本的 BE）上运行的压缩事务的信息。压缩事务在平板电脑级别被分为多个任务，be_cloud_native_compactions 中的每一行都对应压缩事务中的一个任务。

be_cloud_native_compactions 包含以下字段：

|字段|描述|
|---|---|
|BE_ID|CN (BE) 的 ID。|
|TXN_ID|压缩事务的 ID。它可以重复，因为每个压缩事务可能有多个任务。|
|TABLET_ID|任务对应的平板电脑的ID。|
|VERSION|输入到任务的数据的版本。|
|SKIPPED|是否跳过任务。|
|RUNS|任务执行次数。大于 1 的值表示已发生重试。|
|START_TIME|任务开始时间。|
|FINISH_TIME|任务完成时间。如果任务仍在进行中，则返回 NULL。|
|PROGRESS|进度百分比，范围从 0 到 100。|
|状态|任务状态。|
