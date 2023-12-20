---
displayed_sidebar: English
---

# 查看分析状态

## 描述

查看采集任务的状态。

此语句无法用来查看自定义采集任务的状态。若要查看自定义采集任务的状态，请使用 SHOW ANALYZE JOB 命令。

此语句自 v2.4 版本起提供支持。

## 语法

```SQL
SHOW ANALYZE STATUS [WHERE]
```

您可以使用 LIKE 或 WHERE 子句来过滤返回的信息。

此语句将返回以下列：

|列表名称|描述|
|---|---|
|Id|采集任务的ID。|
|数据库|数据库名称。|
|表|表名称。|
|列|列名称。|
|类型|统计数据的类型，包括FULL、SAMPLE、HISTOGRAM。|
|计划|计划的类型。 ONCE 表示手动，SCHEDULE 表示自动。|
|状态|任务的状态。|
|StartTime|任务开始执行的时间。|
|EndTime|任务执行结束的时间。|
|属性|自定义参数。|
|原因|任务失败的原因。如果执行成功则返回NULL。|

## 参考资料

[ANALYZE TABLE](../data-definition/ANALYZE_TABLE.md)：创建一个手动采集任务。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：取消正在执行的自定义采集任务。

有关为 CBO 收集统计信息的更多信息，请参阅[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。
