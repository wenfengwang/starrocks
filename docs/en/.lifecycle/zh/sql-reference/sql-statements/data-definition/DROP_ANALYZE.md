---
displayed_sidebar: English
---

# 删除分析任务

## 描述

删除自定义采集任务。

默认情况下，StarRocks 会自动收集表的完整统计信息。系统每5分钟检查一次数据更新。如果检测到数据变化，将自动触发数据收集。如果您不希望使用自动全量采集，可以将 FE 配置项 `enable_collect_full_statistic` 设置为 `false` 并自定义一个采集任务。

该语句从 v2.4 版本开始支持。

## 语法

```SQL
DROP ANALYZE <ID>
```

可以通过使用 SHOW ANALYZE JOB 语句来获取任务 ID。

## 示例

```SQL
DROP ANALYZE 266030;
```

## 参考资料

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md)：自定义一个自动采集任务。

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)：查看自动采集任务的状态。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：取消一个正在运行的自定义采集任务。

有关为 CBO 收集统计信息的更多信息，请参阅 [Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。