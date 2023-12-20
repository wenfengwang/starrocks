---
displayed_sidebar: English
---

# 恢复例行加载作业

import RoutineLoadPrivNote from '../../../assets/commonMarkdown/RoutineLoadPrivNote.md'

## 描述

恢复一个例行加载作业。作业将因重新调度而暂时进入 **NEED_SCHEDULE** 状态。经过一段时间后，作业将恢复至 **RUNNING** 状态，继续从数据源消费消息并加载数据。您可以使用 [SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md) 语句来查看作业的信息。

<RoutineLoadPrivNote />


## 语法

```SQL
RESUME ROUTINE LOAD FOR [db_name.]<job_name>
```

## 参数

|**参数**|**必填**|**说明**|
|---|---|---|
|db_name||例行加载作业所属的数据库名称。|
|job_name|✅|例行加载作业的名称。|

## 示例

在数据库 `example_db` 中恢复例行加载作业 `example_tbl1_ordertest1`。

```SQL
RESUME ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```