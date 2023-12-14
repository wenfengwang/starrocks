---
displayed_sidebar: "Chinese"
---

# be_cloud_native_compactions

`be_cloud_native_compactions`提供有关在共享数据集群的CNs（或v3.0的BEs）上运行的压实事务的信息。压实事务在表格级别被分成多个任务，`be_cloud_native_compactions`中的每一行对应于压实事务中的一个任务。

`be_cloud_native_compactions`提供以下字段：

| **字段**    | **描述**                                                     |
| ----------- | ------------------------------------------------------------ |
| BE_ID       | CN（BE）的ID。                                               |
| TXN_ID      | 压实事务的ID。它可能是重复的，因为每个压实事务可能有多个任务。                |
| TABLET_ID   | 任务对应的表格的ID。                                          |
| VERSION     | 作为输入到任务的数据的版本。                                  |
| SKIPPED     | 任务是否被跳过。                                             |
| RUNS        | 任务执行次数。大于`1`的值表示已发生重试。                    |
| START_TIME  | 任务开始时间。                                                |
| FINISH_TIME | 任务完成时间。如果任务仍在进行中，则返回`NULL`。             |
| PROGRESS    | 进度百分比，范围从0到100。                                   |
| STATUS      | 任务状态。                                                   |
