---
displayed_sidebar: "English"
---

# 挂起或恢复管道

## 描述

挂起或恢复管道：

- 当加载作业正在进行中（即处于 `RUNNING` 状态）时，挂起该作业的管道（使用 `SUSPEND`）将会中断该作业。
- 当加载作业遇到错误时，恢复该作业的管道（使用 `RESUME`）将继续运行出错的作业。

## 语法

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { SUSPEND | RESUME [ IF SUSPENDED ] }
```

## 参数

### pipe_name

管道的名称。

## 示例

### 挂起管道

在名为 `mydatabase` 的数据库中，挂起名为 `user_behavior_replica`（处于 `RUNNING` 状态）的管道：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica SUSPEND;
```

如果使用 [SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) 查询该管道，您可以看到其状态已更改为 `SUSPEND`。

### 恢复管道

恢复名为 `user_behavior_replica` 的管道（在名为 `mydatabase` 的数据库中）：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica RESUME;
```

如果使用 [SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) 查询该管道，您可以看到其状态已更改为 `RUNNING`。