---
displayed_sidebar: English
---

# 暂停或恢复管道

## 描述

暂停或恢复管道：

- 当加载作业正在进行中（即处于 `RUNNING` 状态）时，暂停 (`SUSPEND`) 作业的管道会中断该作业。
- 当加载作业遇到错误时，恢复 (`RESUME`) 作业的管道将继续运行出错的作业。

## 语法

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { SUSPEND | RESUME [ IF SUSPENDED ] }
```

## 参数

### pipe_name

管道的名称。

## 例子

### 暂停管道

在名为 `mydatabase` 的数据库中暂停处于 `RUNNING` 状态的 `user_behavior_replica` 管道：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica SUSPEND;
```

如果使用 [SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) 查询管道，可以看到其状态已更改为 `SUSPEND`。

### 恢复管道

在名为 `mydatabase` 的数据库中恢复 `user_behavior_replica` 管道：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica RESUME;
```

如果使用 [SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md) 查询管道，可以看到其状态已更改为 `RUNNING`。
