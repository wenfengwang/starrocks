---
displayed_sidebar: English
---

# 展示分析作业

## 描述

查看自定义采集任务的信息及状态。

默认情况下，StarRocks 会自动收集表的全部统计信息。系统每5分钟检查一次数据更新情况。如果侦测到数据有变动，便会自动触发数据采集。如果您不希望使用自动全量采集，可以将 FE 配置项 enable_collect_full_statistic 设为 false，并自定义一个采集任务。

此语句从 v2.4 版本开始支持。

## 语法

```SQL
SHOW ANALYZE JOB [WHERE]
```

您可以使用 WHERE 子句来过滤结果。语句将返回以下列信息。

|专栏|描述|
|---|---|
|Id|采集任务的ID。|
|数据库|数据库名称。|
|表|表名称。|
|列|列名称。|
|类型|统计数据的类型，包括FULL和SAMPLE。|
|计划|计划的类型。类型是自动任务的 SCHEDULE。|
|属性|自定义参数。|
|状态|任务状态，包括 PENDING、RUNNING、SUCCESS 和 FAILED。|
|LastWorkTime|上次采集的时间。|
|原因|任务失败的原因。如果任务执行成功，则返回 NULL。|

## 示例

```SQL
-- View all the custom collection tasks.
SHOW ANALYZE JOB

-- View custom collection tasks of database `test`.
SHOW ANALYZE JOB where `database` = 'test';
```

## 参考资料

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md)：自定义一个自动采集任务。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md)：删除一个自定义采集任务。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：取消一个正在运行的自定义采集任务。

有关为 CBO 收集统计信息的更多信息，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。
