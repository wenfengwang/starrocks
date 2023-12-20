---
displayed_sidebar: English
---

# 暂停或恢复管道

## 描述

暂停或恢复管道操作：

- 当加载作业正在进行（即处于RUNNING状态）时，暂停（SUSPEND）作业的管道会中断作业。
- 当加载作业遇到错误时，恢复（RESUME）作业的管道可以继续执行出错的作业。

## 语法

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { SUSPEND | RESUME [ IF SUSPENDED ] }
```

## 参数

### pipe_name

管道的名称。

## 示例

### 暂停管道

在名为mydatabase的数据库中暂停名为user_behavior_replica的管道（该管道处于RUNNING状态）：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica SUSPEND;
```

如果你使用[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)命令查询管道，你会看到它的状态已经变更为`SUSPEND`。

### 恢复管道

在名为mydatabase的数据库中恢复名为user_behavior_replica的管道：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica RESUME;
```

如果你使用[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)命令查询管道，你会看到它的状态已经变更为`RUNNING`。
