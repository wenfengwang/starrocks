---
displayed_sidebar: English
---

# 恢复例行加载

从 '../../../assets/commonMarkdown/RoutineLoadPrivNote.md' 导入 RoutineLoadPrivNote

## 描述

恢复例行加载作业。该作业将暂时进入 **NEED_SCHEDULE** 状态，因为作业正在重新安排。一段时间后，作业将恢复到 **RUNNING** 状态，继续消费来自数据源的消息并加载数据。您可以使用 [SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md) 语句检查作业的信息。

<RoutineLoadPrivNote />

## 语法

```SQL
RESUME ROUTINE LOAD FOR [db_name.]<job_name>
```

## 参数

| **参数** | **必填** | **描述**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| db_name       |              | 例行加载作业所属的数据库的名称。 |
| job_name      | ✅            | 例行加载作业的名称。                            |

## 例子

在数据库 `example_db` 中恢复例行加载作业 `example_tbl1_ordertest1`。

```SQL
RESUME ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```
