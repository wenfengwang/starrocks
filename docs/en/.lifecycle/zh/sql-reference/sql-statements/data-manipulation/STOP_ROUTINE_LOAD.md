---
displayed_sidebar: English
---

# 停止例程加载

从 '../../../assets/commonMarkdown/RoutineLoadPrivNote.md' 导入 RoutineLoadPrivNote

## 描述

停止一个例程加载作业。

<RoutineLoadPrivNote />

::: 警告

- 已停止的例程加载作业无法恢复。因此，在执行此语句时请谨慎操作。
- 如果只需要暂停例程加载作业，可以执行[PAUSE ROUTINE LOAD](./PAUSE_ROUTINE_LOAD.md)。

:::

## 语法

```SQL
STOP ROUTINE LOAD FOR [db_name.]<job_name>
```

## 参数

| **参数** | **必填** | **描述**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       |              | 例程加载作业所属的数据库的名称。 |
| job_name      | ✅            | 例程加载作业的名称。                            |

## 例子

停止数据库 `example_db` 中的例程加载作业 `example_tbl1_ordertest1`。

```SQL
STOP ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;