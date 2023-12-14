---
displayed_sidebar: "English"
---

# 查看分析任务状态

## 描述

查看采集任务的状态。

无法用此语句查看自定义采集任务的状态。若要查看自定义采集任务的状态，请使用 SHOW ANALYZE JOB。

自v2.4开始支持此语句。

## 语法

```SQL
SHOW ANALYZE STATUS [WHERE]
```

可以使用`LILE or WHERE`来过滤返回的信息。

此语句返回以下列。

| **列表名称** | **描述**                                                    |
| ------------- | ------------------------------------------------------------ |
| Id            | 采集任务的ID。                                               |
| Database      | 数据库名称。                                                 |
| Table         | 表名称。                                                     |
| Columns       | 列名。                                                      |
| Type          | 统计类型，包括FULL、SAMPLE和HISTOGRAM。                       |
| Schedule      | 调度类型。`ONCE`表示手动，`SCHEDULE`表示自动。                 |
| Status        | 任务状态。                                                   |
| StartTime     | 任务开始执行的时间。                                          |
| EndTime       | 任务执行结束的时间。                                           |
| Properties    | 自定义参数。                                                  |
| Reason        | 任务失败的原因。如果执行成功，则返回NULL。                     |

## 参考

[ANALYZE TABLE](../data-definition/ANALYZE_TABLE.md): 创建一个手动采集任务。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 取消正在运行的自定义采集任务。

有关为CBO收集统计信息的更多信息，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。