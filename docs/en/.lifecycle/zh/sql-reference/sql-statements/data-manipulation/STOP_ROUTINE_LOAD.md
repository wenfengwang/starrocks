---
displayed_sidebar: English
---

# 停止常规加载作业

import RoutineLoadPrivNote from '../../../assets/commonMarkdown/RoutineLoadPrivNote.md'

## 描述

停止一个常规加载（Routine Load）作业。

<RoutineLoadPrivNote />


::: warning

- 一旦停止，常规加载作业将无法恢复。因此，在执行此语句时请谨慎操作。
- 如果您只是需要暂停常规加载作业，可以执行 [PAUSE ROUTINE LOAD](./PAUSE_ROUTINE_LOAD.md)。

:::

## 语法

```SQL
STOP ROUTINE LOAD FOR [db_name.]<job_name>
```

## 参数

|**参数**|**必填**|**描述**|
|---|---|---|
|db_name||常规加载作业所属的数据库名称。|
|job_name|✅|常规加载作业的名称。|

## 示例

在数据库 `example_db` 中停止常规加载作业 `example_tbl1_ordertest1`。

```SQL
STOP ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```