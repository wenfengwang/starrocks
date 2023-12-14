---
displayed_sidebar: "Chinese"
---

# KILL ANALYZE

## 描述

取消运行中的收集任务，包括手动和自定义自动任务。

此语句从v2.4起受支持。

## 语法

```SQL
KILL ANALYZE <ID>
```

手动收集任务的任务ID可以从SHOW ANALYZE STATUS中获取。自定义收集任务的任务ID可以从SHOW ANALYZE JOB中获取。

## 参考

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md)

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)

有关为CBO收集统计信息的更多信息，请参阅[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。