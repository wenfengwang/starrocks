---
displayed_sidebar: "Chinese"
---

# 刷新物化视图

## 描述

手动刷新特定的异步物化视图或其中的分区。

> **注意**
>
> 您只能手动刷新采用ASYNC或MANUAL刷新策略的物化视图。您可以使用[SHOW MATERIALIZED VIEWS](../data-manipulation/SHOW_MATERIALIZED_VIEW.md)来检查异步物化视图的刷新策略。

## 语法

```SQL
REFRESH MATERIALIZED VIEW [database.]mv_name
[PARTITION START ("<partition_start_date>") END ("<partition_end_date>")]
[FORCE]
[WITH { SYNC | ASYNC } MODE]
```

## 参数

| **参数**                    | **必需** | **描述**                                        |
| ------------------------- | ------------ | ------------------------------------------------------ |
| mv_name                   | 是           | 要手动刷新的物化视图名称。 |
| PARTITION START () END () | 否           | 手动刷新指定时间间隔内的分区。 |
| partition_start_date      | 否           | 手动刷新的分区的起始日期。  |
| partition_end_date        | 否           | 手动刷新的分区的结束日期。    |
| FORCE                     | 否           | 如果指定此参数，StarRocks将强制刷新对应的物化视图或分区。如果不指定此参数，StarRocks会自动判断数据是否已更新，并仅在需要时刷新分区。  |
| WITH ... MODE             | 否           | 进行刷新任务的同步或异步调用。`SYNC`表示进行刷新任务的同步调用，StarRocks只有在任务成功或失败时才返回任务结果。`ASYNC`表示进行刷新任务的异步调用，StarRocks在提交任务后立即返回成功，并将任务异步在后台执行。您可以通过查询StarRocks的信息模式中的 `tasks` 和 `task_runs` 元数据视图来检查异步物化视图的刷新任务执行状态。有关更多信息，请参阅[检查异步物化视图的执行状态](../../../using_starrocks/Materialized_view.md#check-the-execution-status-of-asynchronous-materialized-view)。默认值：`ASYNC`。从v3.1.0开始支持。 |

> **注意**
>
> 刷新基于外部目录创建的物化视图时，StarRocks会刷新物化视图中的所有分区。

## 示例

示例1：通过异步调用手动刷新特定物化视图。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1;

REFRESH MATERIALIZED VIEW lo_mv1 WITH ASYNC MODE;
```

示例2：手动刷新特定物化视图的某些分区。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 
PARTITION START ("2020-02-01") END ("2020-03-01");
```

示例3：强制刷新特定物化视图的某些分区。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1
PARTITION START ("2020-02-01") END ("2020-03-01") FORCE;
```

示例4：通过同步调用手动刷新物化视图。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 WITH SYNC MODE;
```