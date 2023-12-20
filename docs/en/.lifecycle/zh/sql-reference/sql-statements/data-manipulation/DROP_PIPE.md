---
displayed_sidebar: English
---

# 删除管道

## 描述

删除一个管道及其相关的作业和元数据。执行此语句将不会撤销通过该管道已加载的数据。

## 语法

```SQL
DROP PIPE [IF EXISTS] [db_name.]<pipe_name>
```

## 参数

### db_name

管道所属的数据库名称。

### pipe_name

管道的名称。

## 示例

在名为 `mydatabase` 的数据库中删除名为 `user_behavior_replica` 的管道：

```SQL
USE mydatabase;
DROP PIPE user_behavior_replica;
```