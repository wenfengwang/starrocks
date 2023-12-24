---
displayed_sidebar: English
---

# DROP ANALYZE

## 描述

删除自定义采集任务。

默认情况下，StarRocks 会自动收集表的完整统计信息。它每 5 分钟检查一次数据更新情况。如果检测到数据变化，将自动触发数据收集。如果不想使用自动完整采集，可以将 FE 配置项 `enable_collect_full_statistic` 设置为 `false` 并自定义一个采集任务。

此语句从 v2.4 版本开始支持。

## 语法

```SQL
DROP ANALYZE <ID>
```

可以使用 SHOW ANALYZE JOB 语句获取任务 ID。

## 例子

```SQL
DROP ANALYZE 266030;
```

## 引用

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md)：自定义一个自动采集任务。

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)：查看自动采集任务的状态。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：取消正在运行的自定义采集任务。

有关为 CBO 收集统计信息的更多信息，请参阅 [为 CBO 收集统计信息](../../../using_starrocks/Cost_based_optimizer.md)。