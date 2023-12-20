---
displayed_sidebar: English
---

# 刷新物化视图

## 描述

手动刷新指定的异步物化视图或其内部分区。

> **注意**
> 您只能手动刷新采用**ASYNC**或**MANUAL**刷新策略的物化视图。您可以使用[SHOW MATERIALIZED VIEWS](../data-manipulation/SHOW_MATERIALIZED_VIEW.md)命令来检查异步物化视图的刷新策略。此操作需要对目标物化视图有**REFRESH**权限。

## 语法

```SQL
REFRESH MATERIALIZED VIEW [database.]mv_name
[PARTITION START ("<partition_start_date>") END ("<partition_end_date>")]
[FORCE]
[WITH { SYNC | ASYNC } MODE]
```

## 参数

|参数|必填|说明|
|---|---|---|
|mv_name|yes|要手动刷新的物化视图的名称。|
|PARTITION START () END ()|no|在一定时间间隔内手动刷新分区。|
|partition_start_date|no|要手动刷新的分区的开始日期。|
|partition_end_date|no|要手动刷新的分区的结束日期。|
|FORCE|no|如果指定该参数，StarRocks 会强制刷新相应的物化视图或分区。如果不指定该参数，StarRocks会自动判断数据是否更新，仅在需要时刷新分区。|
|WITH ... MODE|no|对刷新任务进行同步或异步调用。 SYNC表示同步调用刷新任务，StarRocks仅在任务成功或失败时返回任务结果。 ASYNC表示异步调用刷新任务，任务提交后StarRocks立即返回成功，让任务在后台异步执行。您可以通过查询StarRocks Information Schema 中的tasks 和task_runs 元数据视图来检查异步物化视图的刷新任务状态。详细信息请参见检查异步物化视图的执行状态。默认值：异步。从 v3.1.0 开始支持。|

> **注意**
> 当刷新基于外部目录创建的物化视图时，StarRocks刷新es所有分区中的物化视图。

## 示例

示例1：通过异步调用手动刷新指定的物化视图。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1;

REFRESH MATERIALIZED VIEW lo_mv1 WITH ASYNC MODE;
```

示例2：手动刷新指定物化视图的特定分区。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 
PARTITION START ("2020-02-01") END ("2020-03-01");
```

示例3：强制刷新指定物化视图的特定分区。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1
PARTITION START ("2020-02-01") END ("2020-03-01") FORCE;
```

示例4：通过同步调用手动刷新物化视图。

```Plain
REFRESH MATERIALIZED VIEW lo_mv1 WITH SYNC MODE;
```
