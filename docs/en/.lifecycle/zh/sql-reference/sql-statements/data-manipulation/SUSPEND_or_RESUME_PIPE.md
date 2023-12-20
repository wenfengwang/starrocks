---
displayed_sidebar: English
---

# 暂停或恢复 PIPE

## 描述

暂停或恢复一个管道：

- 当一个加载作业正在进行中（即处于 `RUNNING` 状态），暂停（`SUSPEND`）该作业的管道会中断作业。
- 当一个加载作业遇到错误时，恢复（`RESUME`）该作业的管道将会继续执行出错的作业。

## 语法

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { SUSPEND | RESUME [ IF SUSPENDED ] }
```

## 参数

### pipe_name

管道的名称。

## 示例

### 暂停一个管道

在名为 `mydatabase` 的数据库中暂停名为 `user_behavior_replica` 的管道（该管道处于 `RUNNING` 状态）：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica SUSPEND;
```

如果使用 [SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) 来查询管道，你可以看到它的状态已经变为 `SUSPEND`。

### 恢复一个管道

在名为 `mydatabase` 的数据库中恢复名为 `user_behavior_replica` 的管道：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica RESUME;
```

如果使用 [SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) 来查询管道，你可以看到它的状态已经变回 `RUNNING`。