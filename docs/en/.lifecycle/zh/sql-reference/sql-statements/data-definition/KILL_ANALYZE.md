---
displayed_sidebar: English
---

# KILL ANALYZE

## 描述

取消一个**正在运行**的采集任务，包括手动和自定义自动任务。

此语句从 v2.4 版本开始支持。

## 语法

```SQL
KILL ANALYZE <ID>
```

手动采集任务的任务 ID 可以通过 SHOW ANALYZE STATUS 命令获取。自定义采集任务的任务 ID 可以通过 SHOW ANALYZE JOB 命令获取。

## 参考资料

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md)

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)

有关为 CBO 收集统计信息的更多信息，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。