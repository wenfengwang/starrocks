---
displayed_sidebar: English
---

# 刷新物化视图

## 描述

手动刷新特定的异步物化视图或其中的分区。

> **注意**
>
> 只能手动刷新采用 ASYNC 或 MANUAL 刷新策略的物化视图。您可以使用 [SHOW MATERIALIZED VIEWS](../data-manipulation/SHOW_MATERIALIZED_VIEW.md) 来检查异步物化视图的刷新策略。
> 此操作需要对目标物化视图具有 REFRESH 特权。

## 语法

```SQL
REFRESH MATERIALIZED VIEW [database.]mv_name
[PARTITION START ("<partition_start_date>") END ("<partition_end_date>")]
[FORCE]
[WITH { SYNC | ASYNC } MODE]
```

## 参数

| **参数**             | **必填** | **描述**                                        |
| ------------------------- | ------------ | ------------------------------------------------------ |
| mv_name                   | 是          | 要手动刷新的物化视图的名称。 |
| PARTITION START () END () | 否           | 在特定时间间隔内手动刷新分区。 |
| partition_start_date      | 否           | 要手动刷新的分区的开始日期。  |
| partition_end_date        | 否           | 要手动刷新的分区的结束日期。    |
| FORCE                     | 否           | 如果指定此参数，StarRocks 将强制刷新相应的物化视图或分区。如果不指定此参数，StarRocks 将自动判断数据是否已更新，并仅在需要时刷新分区。  |
| WITH ... MODE             | 否           | 进行同步或异步调用刷新任务。`SYNC` 表示进行同步调用刷新任务，StarRocks 仅在任务成功或失败时返回任务结果。`ASYNC` 表示进行异步调用刷新任务，StarRocks 在提交任务后立即返回成功，将任务异步在后台执行。您可以通过查询 StarRocks 信息模式中的 `tasks` 和 `task_runs` 元数据视图来检查异步物化视图的刷新任务状态。有关详细信息，请参阅[检查异步物化视图的执行状态](../../../using_starrocks/Materialized_view.md#check-the-execution-status-of-asynchronous-materialized-view)。默认值：`ASYNC`。从 v3.1.0 开始支持。

> **注意**
>
> 刷新基于外部目录创建的物化视图时，StarRocks 会刷新物化视图中的所有分区。

## 例子

示例 1：通过异步调用手动刷新特定的物化视图。

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
