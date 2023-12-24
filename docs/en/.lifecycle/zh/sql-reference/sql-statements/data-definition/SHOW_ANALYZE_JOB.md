---
displayed_sidebar: English
---

# 显示分析作业

## 描述

查看自定义采集任务的信息和状态。

默认情况下，StarRocks 会自动收集表的完整统计信息。它每 5 分钟检查一次数据更新。如果检测到数据更改，将自动触发数据收集。如果不想使用自动完整采集，可以将 FE 配置项 `enable_collect_full_statistic` 设置为 `false` 并自定义采集任务。

此语句从 v2.4 版本开始支持。

## 语法

```SQL
SHOW ANALYZE JOB [WHERE]
```

可以使用 WHERE 子句来筛选结果。该语句返回以下列。

| **列名**   | **描述**                                              |
| ------------ | ------------------------------------------------------------ |
| Id           | 采集任务的 ID。                               |
| Database     | 数据库名称。                                           |
| Table        | 表名。                                              |
| Columns      | 列名。                                            |
| Type         | 统计类型，包括 `FULL` 和 `SAMPLE`。       |
| Schedule     | 调度类型。自动任务的类型为 `SCHEDULE`。 |
| Properties   | 自定义参数。                                           |
| Status       | 任务状态，包括 PENDING、RUNNING、SUCCESS 和 FAILED。 |
| LastWorkTime | 上次采集时间。                             |
| Reason       | 任务失败的原因。如果任务执行成功，则返回 NULL。 |

## 例子

```SQL
-- 查看所有自定义采集任务。
SHOW ANALYZE JOB

-- 查看数据库 `test` 的自定义采集任务。
SHOW ANALYZE JOB WHERE `database` = 'test';
```

## 引用

[CREATE ANALYZE](../data-definition/CREATE_ANALYZE.md)：自定义自动采集任务。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md)：删除自定义采集任务。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：取消正在运行的自定义采集任务。

有关为 CBO 收集统计信息的更多信息，请参阅 [为 CBO 收集统计信息](../../../using_starrocks/Cost_based_optimizer.md)。