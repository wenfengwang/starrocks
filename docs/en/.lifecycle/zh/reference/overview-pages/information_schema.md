---
displayed_sidebar: English
---

# 信息架构

StarRocks 信息架构是 StarRocks 实例中的一个数据库。信息架构包含多个只读的系统定义视图，用于存储 StarRocks 实例维护的所有对象的大量元数据信息。StarRocks 信息架构基于 SQL-92 ANSI 信息架构，但增加了特定于 StarRocks 的视图和函数。

从 v3.2.0 开始，StarRocks 信息架构支持管理外部目录的元数据。

## 通过信息架构查看元数据

您可以通过查询 Information Schema 中的视图内容来查看 StarRocks 实例中的元数据信息。

以下示例通过查询名为 `tables` 的视图来检查 StarRocks 中名为 `table1` 的表的元数据信息。

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

## 信息架构中的视图

StarRocks 信息架构包含以下元数据视图：

| **视图**                                                    | **描述**                                              |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| [be_bvars](../information_schema/be_bvars.md)                                       | `be_bvars` 提供有关 bRPC 的统计信息。  |
| [be_cloud_native_compactions](../information_schema/be_cloud_native_compactions.md) | `be_cloud_native_compactions` 提供有关在共享数据群集的 CN（或 v3.0 的 BE）上运行的压缩事务的信息。 |
| [be_compactions](../information_schema/be_compactions.md)                           | `be_compactions` 提供有关压缩任务的统计信息。 |
| [character_sets](../information_schema/character_sets.md)                           | `character_sets` 标识可用的字符集。    |
| [collations](../information_schema/collations.md)                                   | `collations` 包含可用的排序规则。              |
| [column_privileges](../information_schema/column_privileges.md)                     | `column_privileges` 标识对当前启用的角色或由当前启用的角色授予的列的所有特权。 |
| [columns](../information_schema/columns.md)                                         | `columns` 包含有关所有表列（或视图列）的信息。 |
| [engines](../information_schema/engines.md)                                         | `engines` 提供有关存储引擎的信息。        |
| [events](../information_schema/events.md)                                           | `events` 提供有关事件管理器事件的信息。    |
| [global_variables](../information_schema/global_variables.md)                       | `global_variables` 提供有关全局变量的信息。 |
| [key_column_usage](../information_schema/key_column_usage.md)                       | `key_column_usage` 标识受某些唯一键、主键或外键约束约束的所有列。 |
| [load_tracking_logs](../information_schema/load_tracking_logs.md)                   | `load_tracking_logs` 提供加载作业的错误信息（如果有）。 |
| [loads](../information_schema/loads.md)                                             | `loads` 提供加载作业的结果。目前，您只能从此视图查看 [Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和 [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 作业的结果。|
| [materialized_views](../information_schema/materialized_views.md)                   | `materialized_views` 提供有关所有异步实例化视图的信息。 |
| [partitions](../information_schema/partitions.md)                                   | `partitions` 提供有关表分区的信息。    |
| [pipe_files](../information_schema/pipe_files.md)                                   | `pipe_files` 提供要通过指定管道加载的数据文件的状态。 |
| [pipes](../information_schema/pipes.md)                                             | `pipes` 提供有关存储在当前数据库或指定数据库中的所有管道的信息。 |
| [referential_constraints](../information_schema/referential_constraints.md)         | `referential_constraints` 包含所有引用（外键）约束。 |
| [routines](../information_schema/routines.md)                                       | `routines` 包含所有存储例程（存储过程和存储函数）。 |
| [schema_privileges](../information_schema/schema_privileges.md)                     | `schema_privileges` 提供有关数据库特权的信息。 |
| [schemata](../information_schema/schemata.md)                                       | `schemata` 提供有关数据库的信息。             |
| [session_variables](../information_schema/session_variables.md)                     | `session_variables` 提供有关会话变量的信息。 |
| [statistics](../information_schema/statistics.md)                                   | `statistics` 提供有关表索引的信息。       |
| [table_constraints](../information_schema/table_constraints.md)                     | `table_constraints` 描述哪些表具有约束。 |
| [table_privileges](../information_schema/table_privileges.md)                       | `table_privileges` 提供有关表权限的信息。 |
| [tables](../information_schema/tables.md)                                           | `tables` 提供有关表的信息。                  |
| [tables_config](../information_schema/tables_config.md)                             | `tables_config` 提供有关表配置的信息。 |
| [task_runs](../information_schema/task_runs.md)                                     | `task_runs` 提供有关异步任务执行的信息。 |
| [tasks](../information_schema/tasks.md)                                             | `tasks` 提供有关异步任务的信息。       |
| [triggers](../information_schema/triggers.md)                                       | `triggers` 提供有关触发器的信息。              |
| [user_privileges](../information_schema/user_privileges.md)                         | `user_privileges` 提供有关用户权限的信息。 |
| [views](../information_schema/views.md)                                             | `views` 提供有关所有用户定义视图的信息。   |

