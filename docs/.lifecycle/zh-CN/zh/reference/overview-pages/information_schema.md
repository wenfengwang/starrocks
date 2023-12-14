---
displayed_sidebar: "Chinese"
---

# 信息模式

Information Schema 是StarRocks实例中的一个数据库。这个数据库包含了由系统定义的很多视图，这些视图中存储了关于StarRocks实例中所有对象的大量元数据信息。

从v3.2.0开始，Information Schema支持管理External Catalog中的元数据信息。

## 通过Information Schema查看元数据信息

您可以通过查询Information Schema中的视图来查看StarRocks实例中的元数据信息。

以下示例通过查询视图`tables`查看StarRocks中名为`table1`的表相关的元数据信息。

```Plain
MySQL > SELECT * FROM information_schema.tables WHERE TABLE_NAME like 'table1'\G
*************************** 1. row ***************************
  TABLE_CATALOG: def
   TABLE_SCHEMA: test_db
     TABLE_NAME: table1
     TABLE_TYPE: BASE TABLE
         ENGINE: StarRocks
        VERSION: NULL
     ROW_FORMAT: 
     TABLE_ROWS: 4
 AVG_ROW_LENGTH: 1657
    DATA_LENGTH: 6630
MAX_DATA_LENGTH: NULL
   INDEX_LENGTH: NULL
      DATA_FREE: NULL
 AUTO_INCREMENT: NULL
    CREATE_TIME: 2023-06-13 11:37:00
    UPDATE_TIME: 2023-06-13 11:38:06
     CHECK_TIME: NULL
TABLE_COLLATION: utf8_general_ci
       CHECKSUM: NULL
 CREATE_OPTIONS: 
  TABLE_COMMENT: 
1 row in set (0.01 sec)
```

## Information Schema中的视图

StarRocks Information Schema中包含以下视图：

| **视图名**                                                  | **描述**                                                     |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| [be_bvars](../information_schema/be_bvars.md)                                       | `be_bvars`提供关于bRPC的统计信息。                        |
| [be_cloud_native_compactions](../information_schema/be_cloud_native_compactions.md) | `be_cloud_native_compactions`提供关于存算一体集群的CN（或v3.0中的BE）上运行的Compaction事务的信息。 |
| [be_compactions](../information_schema/be_compactions.md)                           | `be_compactions`提供关于Compaction任务的统计信息。        |
| [character_sets](../information_schema/character_sets.md)                           | `character_sets`用于识别可用的字符集。                      |
| [collations](../information_schema/collations.md)                                   | `collations`包含可用的排序规则。                            |
| [column_privileges](../information_schema/column_privileges.md)                     | `column_privileges`用于识别当前启用的角色被授予的或由当前启用的角色授予的所有列权限。 |
| [columns](../information_schema/columns.md)                                         | `columns`包含有关所有表（或视图）中列的信息。               |
| [engines](../information_schema/engines.md)                                         | `engines`提供关于存储引擎的信息。                           |
| [events](../information_schema/events.md)                                           | `events`提供关于EventManager事件的信息。                 |
| [global_variables](../information_schema/global_variables.md)                       | `global_variables`提供关于全局变量的信息。                  |
| [key_column_usage](../information_schema/key_column_usage.md)                       | `key_column_usage`用于识别受某些唯一、主键或外键约束限制的所有列。 |
| [load_tracking_logs](../information_schema/load_tracking_logs.md)                   |提供导入作业相关的错误信息。                                 |
| [loads](../information_schema/loads.md)                                             |提供导入作业的结果信息。当前仅支持查看[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)和[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)导入作业的结果信息。 |
| [materialized_views](../information_schema/materialized_views.md)                   | `materialized_views`提供关于所有异步物化视图的信息。        |
| [partitions](../information_schema/partitions.md)                                   | `partitions`提供关于表分区的信息。                          |
| [pipe_files](../information_schema/pipe_files.md)                                   | `pipe_files`提供指定Pipe下数据文件的导入状态。            |
| [pipes](../information_schema/pipes.md)                                             | `pipes`提供当前数据库或指定数据库下所有Pipe的详细信息。   |
| [referential_constraints](../information_schema/referential_constraints.md)         | `referential_constraints`包含所有参照（外键）约束。         |
| [routines](../information_schema/routines.md)                                       | `routines`包含所有存储的过程（Routine），包括流程和函数。   |
| [schema_privileges](../information_schema/schema_privileges.md)                     | `schema_privileges`提供关于数据库权限的信息。               |
| [schemata](../information_schema/schemata.md)                                       | `schemata`提供关于数据库的信息。                            |
| [session_variables](../information_schema/session_variables.md)                     | `session_variables`提供关于Session变量的信息。            |
| [statistics](../information_schema/statistics.md)                                   | `statistics`提供关于表索引的信息。                          |
| [table_constraints](../information_schema/table_constraints.md)                     | `table_constraints`描述具有约束的表。                       |
| [table_privileges](../information_schema/table_privileges.md)                       | `table_privileges`提供关于表权限的信息。                    |
| [tables](../information_schema/tables.md)                                           | `tables`提供关于表的信息。                                  |
| [tables_config](../information_schema/tables_config.md)                             | `tables_config`提供关于表配置的信息。                       |
| [task_runs](../information_schema/task_runs.md)                                     | `task_runs`提供关于异步任务执行的信息。                     |
| [tasks](../information_schema/tasks.md)                                             | `tasks`提供关于异步任务的信息。                             |
| [triggers](../information_schema/triggers.md)                                       | `triggers`提供关于触发器的信息。                            |
| [user_privileges](../information_schema/user_privileges.md)                         | `user_privileges`提供关于用户权限的信息。                   |
| [views](../information_schema/views.md)                                             | `views`提供关于所有用户定义视图的信息。                     |