---
displayed_sidebar: English
---

# 信息模式

StarRocks信息模式是StarRocks实例中的一个数据库。信息模式包含了多个只读的、系统定义的视图，用于存储StarRocks实例维护的所有对象的丰富元数据信息。StarRocks信息模式基于SQL-92 ANSI信息模式，但增加了一些特定于StarRocks的视图和函数。

从v3.2.0版本开始，StarRocks信息模式支持管理外部目录的元数据。

## 通过信息模式查看元数据

您可以通过查询信息模式中的视图内容来查看StarRocks实例中的元数据信息。

以下示例展示了如何通过查询名为tables的视图来检查StarRocks中名为table1的表的元数据信息。

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

## 信息模式中的视图

StarRocks信息模式包含以下元数据视图：

|查看|说明|
|---|---|
|be_bvars|be_bvars 提供有关 bRPC 的统计信息。|
|be_cloud_native_compactions|be_cloud_native_compactions 提供有关在共享数据集群的 CN（或 v3.0 的 BE）上运行的压缩事务​​的信息。|
|be_compactions|be_compactions 提供有关压缩任务的统计信息。|
|character_sets|character_sets 标识可用的字符集。|
|collat​​ions|collat​​ions 包含可用的排序规则。|
|column_privileges|column_privileges 标识在列上授予当前启用的角色或由当前启用的角色授予的所有权限。|
|columns|columns 包含有关所有表列（或视图列）的信息。|
|engines|engines 提供有关存储引擎的信息。|
|events|events 提供有关事件管理器事件的信息。|
|global_variables|global_variables 提供有关全局变量的信息。|
|key_column_usage|key_column_usage 标识受某些唯一、主键或外键约束限制的所有列。|
|load_tracking_logs|load_tracking_logs 提供加载作业的错误信息（如果有）。|
|loads|loads 提供加载作业的结果。目前，您只能从此视图查看 Broker Load 和 INSERT 作业的结果。|
|materialized_views|materialized_views 提供有关所有异步物化视图的信息。|
|partitions|partitions 提供有关表分区的信息。|
|pipe_files|pipe_files 提供要通过指定管道加载的数据文件的状态。|
|pipes|pipes 提供有关存储在当前或指定数据库中的所有管道的信息。|
|referential_constraints|referential_constraints 包含所有引用（外键）约束。|
|例程|例程包含所有存储例程（存储过程和存储函数）。|
|schema_privileges|schema_privileges 提供有关数据库权限的信息。|
|schemata|schemata 提供有关数据库的信息。|
|session_variables|session_variables 提供有关会话变量的信息。|
|statistics|statistics 提供有关表索引的信息。|
|table_constraints|table_constraints 描述哪些表具有约束。|
|table_privileges|table_privileges 提供有关表权限的信息。|
|tables|tables 提供有关表的信息。|
|tables_config|tables_config 提供有关表配置的信息。|
|task_runs|task_runs 提供有关异步任务执行的信息。|
|tasks|tasks 提供有关异步任务的信息。|
|triggers|triggers 提供有关触发器的信息。|
|user_privileges|user_privileges 提供有关用户权限的信息。|
|views|views 提供有关所有用户定义视图的信息。|
