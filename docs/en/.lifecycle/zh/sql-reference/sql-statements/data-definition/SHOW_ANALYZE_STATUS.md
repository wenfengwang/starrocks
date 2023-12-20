---
displayed_sidebar: English
---

# 显示分析状态

## 描述

查看采集任务的状态。

该语句不能用于查看自定义采集任务的状态。要查看自定义采集任务的状态，请使用 SHOW ANALYZE JOB。

该语句从 v2.4 版本开始支持。

## 语法

```SQL
SHOW ANALYZE STATUS [WHERE]
```

您可以使用 `LIKE` 或 `WHERE` 来过滤返回的信息。

该语句返回以下列。

|**列名**|**描述**|
|---|---|
|Id|采集任务的 ID。|
|Database|数据库名称。|
|Table|表名。|
|Columns|列名。|
|Type|统计类型，包括 FULL、SAMPLE 和 HISTOGRAM。|
|Schedule|调度类型。`ONCE` 表示手动，`SCHEDULE` 表示自动。|
|Status|任务状态。|
|StartTime|任务开始执行的时间。|
|EndTime|任务执行结束的时间。|
|Properties|自定义参数。|
|Reason|任务失败的原因。如果执行成功，则返回 NULL。|

## 参考资料

[ANALYZE TABLE](../data-definition/ANALYZE_TABLE.md)：创建一个手动采集任务。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：取消正在运行的自定义采集任务。

有关为 CBO 收集统计信息的更多信息，请参见[为 CBO 收集统计信息](../../../using_starrocks/Cost_based_optimizer.md)。