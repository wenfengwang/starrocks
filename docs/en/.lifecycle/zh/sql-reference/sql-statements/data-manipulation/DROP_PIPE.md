---
displayed_sidebar: English
---

# 删除管道

## 描述

删除管道以及相关的作业和元数据。在管道上执行此语句不会撤销已通过此管道加载的数据。

## 语法

```SQL
DROP PIPE [IF EXISTS] [db_name.]<pipe_name>
```

## 参数

### db_name

管道所属的数据库的名称。

### pipe_name

管道的名称。

## 例子

删除名为 `user_behavior_replica` 的管道，该管道位于名为 `mydatabase` 的数据库中：

```SQL
USE mydatabase;
DROP PIPE user_behavior_replica;