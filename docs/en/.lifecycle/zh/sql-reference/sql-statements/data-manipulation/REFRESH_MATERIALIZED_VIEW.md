---
displayed_sidebar: English
---

# 刷新物化视图

## 描述

手动刷新特定的异步物化视图或其内的分区。

> **警告**
> 您只能手动刷新采用**ASYNC**或**MANUAL**刷新策略的物化视图。您可以使用[SHOW MATERIALIZED VIEWS](../data-manipulation/SHOW_MATERIALIZED_VIEW.md)检查异步物化视图的刷新策略。此操作需要对目标物化视图有**REFRESH**权限。

## 语法

```SQL
REFRESH MATERIALIZED VIEW [database.]mv_name
[PARTITION START ("<partition_start_date>") END ("<partition_end_date>")]
[FORCE]
[WITH { SYNC | ASYNC } MODE]
```

## 参数

|**参数**|**必填**|**说明**|
|---|---|---|
|mv_name|是|要手动刷新的物化视图名称。|
|PARTITION START () END ()|否|手动刷新特定时间间隔内的分区。|
|partition_start_date|否|要手动刷新的分区的起始日期。|
|partition_end_date|否|要手动刷新的分区的结束日期。|
|FORCE|否|如果指定此参数，StarRocks 将强制刷新相应的物化视图或分区。如果未指定此参数，StarRocks 将自动判断数据是否已更新，并仅在必要时刷新分区。|
|WITH ... MODE|否|以同步或异步方式调用刷新任务。`SYNC` 表示以同步方式调用刷新任务，StarRocks 仅在任务成功或失败后返回任务结果。`ASYNC` 表示以异步方式调用刷新任务，任务提交后 StarRocks 立即返回成功，任务则在后台异步执行。您可以通过查询 StarRocks Information Schema 中的 `tasks` 和 `task_runs` 元数据视图来检查异步物化视图的刷新任务状态。更多信息，请参见[检查异步物化视图的执行状态](../../../using_starrocks/Materialized_view.md#check-the-execution-status-of-asynchronous-materialized-view)。默认值：`ASYNC`。从 v3.1.0 版本开始支持。|

> **警告**
> 当刷新基于外部目录创建的物化视图时，StarRocks 会刷新物化视图中的所有分区。

## 示例

示例 1：通过异步调用手动刷新特定物化视图。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1;

REFRESH MATERIALIZED VIEW lo_mv1 WITH ASYNC MODE;
```

示例 2：手动刷新特定物化视图的某些分区。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 
PARTITION START ("2020-02-01") END ("2020-03-01");
```

示例 3：强制刷新特定物化视图的某些分区。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1
PARTITION START ("2020-02-01") END ("2020-03-01") FORCE;
```

示例 4：通过同步调用手动刷新物化视图。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 WITH SYNC MODE;
```