---
displayed_sidebar: English
---

# be_cloud_native_compactions

`be_cloud_native_compactions` 提供了关于在共享数据集群的 CN（或 v3.0 的 BE）上运行的压缩事务的信息。一个压缩事务在 tablet 级别被分为多个任务，`be_cloud_native_compactions` 中的每一行对应压缩事务中的一个任务。

`be_cloud_native_compactions` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|BE_ID|CN（BE）的 ID。|
|TXN_ID|压缩事务的 ID。它可能会重复，因为每个压缩事务可能包含多个任务。|
|TABLET_ID|任务对应的 tablet 的 ID。|
|VERSION|任务输入数据的版本。|
|SKIPPED|任务是否被跳过。|
|RUNS|任务执行的次数。大于 `1` 的值表示已经进行了重试。|
|START_TIME|任务开始时间。|
|FINISH_TIME|任务完成时间。如果任务仍在进行中，则返回 `NULL`。|
|PROGRESS|进度百分比，范围是 0 到 100。|
|STATUS|任务状态。|