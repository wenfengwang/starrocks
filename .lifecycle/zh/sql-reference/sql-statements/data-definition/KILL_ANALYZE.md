---
displayed_sidebar: English
---

# 终止分析任务

## 描述

取消一个**正在执行**的数据收集任务，包括手动和自定义的自动任务。

此语句从v2.4版本开始支持。

## 语法

```SQL
KILL ANALYZE <ID>
```

手动收集任务的任务ID可通过SHOW ANALYZE STATUS命令获得。自定义收集任务的任务ID可通过SHOW ANALYZE JOB命令获得。

## 参考资料

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md)

[显示分析作业](../data-definition/SHOW_ANALYZE_JOB.md)

关于为成本基优化器（CBO）收集统计信息的更多信息，请参考[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。
