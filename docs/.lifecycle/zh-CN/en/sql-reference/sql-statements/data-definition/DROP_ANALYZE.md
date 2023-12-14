---
displayed_sidebar: "Chinese"
---

# DROP ANALYZE（删除分析）

## 描述

删除自定义收集任务。

默认情况下，StarRocks自动收集表的完整统计信息。它每5分钟检查任何数据更新。如果检测到数据更改，将自动触发数据收集。如果不想使用自动完整收集，可以将FE配置项`enable_collect_full_statistic`设置为`false`并自定义收集任务。

此语句支持v2.4。

## 语法

```SQL
DROP ANALYZE <ID>
```

可以使用SHOW ANALYZE JOB语句获取任务ID。

## 示例

```SQL
DROP ANALYZE 266030;
```

## 参考

[CREATE ANALYZE（创建分析）](../data-definition/CREATE_ANALYZE.md)：自定义自动收集任务。

[SHOW ANALYZE JOB（显示分析任务）](../data-definition/SHOW_ANALYZE_JOB.md)：查看自动收集任务的状态。

[KILL ANALYZE（终止分析）](../data-definition/KILL_ANALYZE.md)：取消正在运行的自定义收集任务。

有关为CBO收集统计信息的更多信息，请参阅[Gather statistics for CBO（为CBO收集统计信息）](../../../using_starrocks/Cost_based_optimizer.md)。