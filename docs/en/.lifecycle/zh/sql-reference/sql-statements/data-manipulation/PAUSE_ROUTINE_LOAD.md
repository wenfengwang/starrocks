---
displayed_sidebar: English
---

# 暂停 Routine Load 作业

import RoutineLoadPrivNote from '../../../assets/commonMarkdown/RoutineLoadPrivNote.md'

## 描述

暂停一个 Routine Load 作业，但不会终止该作业。您可以执行 [RESUME ROUTINE LOAD](./RESUME_ROUTINE_LOAD.md) 来恢复它。在作业被暂停后，您可以执行 [SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md) 和 [ALTER ROUTINE LOAD](./ALTER_ROUTINE_LOAD.md) 来查看和修改有关暂停作业的信息。

<RoutineLoadPrivNote />


## 语法

```SQL
PAUSE ROUTINE LOAD FOR [db_name.]<job_name>;
```

## 参数

|参数|必填|描述|
|---|---|---|
|db_name||Routine Load 作业所属的数据库名称。|
|job_name|✅|Routine Load 作业的名称。一张表可能有多个 Routine Load 作业，建议使用可识别信息，例如 Kafka 主题名称或您创建作业时的时间，来设置一个有意义的 Routine Load 作业名称，以区分多个例行加载作业。Routine Load 作业的名称在同一数据库内必须唯一。|

## 示例

在数据库 `example_db` 中暂停 Routine Load 作业 `example_tbl1_ordertest1`。

```sql
PAUSE ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```