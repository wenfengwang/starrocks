---
displayed_sidebar: English
---

# 终止分析

## 描述

取消**正在运行**的收集任务，包括手动和自定义自动任务。

此语句从 v2.4 版本开始得到支持。

## 语法

```SQL
KILL ANALYZE <ID>
```

手动收集任务的任务 ID 可以从 SHOW ANALYZE STATUS 中获取。自定义收集任务的任务 ID 可以从 SHOW ANALYZE JOB 中获取。

## 引用

[显示分析状态](../data-definition/SHOW_ANALYZE_STATUS.md)

[显示分析作业](../data-definition/SHOW_ANALYZE_JOB.md)

有关为 CBO 收集统计信息的更多信息，请参阅 [为 CBO 收集统计信息](../../../using_starrocks/Cost_based_optimizer.md)。