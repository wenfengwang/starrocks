---
displayed_sidebar: "Chinese"
---

# StarRocks版本3.0

## 3.0.8

发布日期：2023年11月17日

### 功能优化

- `INFORMATION_SCHEMA.COLUMNS`表支持显示ARRAY、MAP、STRUCT类型的字段。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

### 问题修复

修复了如下问题：

- 执行 `show proc '/current_queries';` 时，如果某个查询刚开始执行，可能会引起BE Crash。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 带Sort Key的主键模型表持续高频导入时，会小概率出现Compaction失败。[#26486](https://github.com/StarRocks/starrocks/pull/26486)
- 修改LDAP认证用户的密码后，使用新密码登录时报错。
- 如果提交的Broker Load作业包含过滤条件，在数据导入过程中，某些情况下会出现BE crash。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- SHOW GRANTS时报`unknown error`。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 当使用函数`cast()`时数据类型转换前后类型一致，某些类型下会导致BE crash。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- BINARY或VARBINARY类型在`information_schema.columns`视图里面的`DATA_TYPE`和`COLUMN_TYPE`显示为`unknown`。[#32678](https://github.com/StarRocks/starrocks/pull/32678)
- 长时间向持久化索引打开的主键模型表高频导入，可能会引起BE crash。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- Query Cache开启后查询结果有错。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- 重启后RESTORE的表中的数据和BACKUP前的表中的数据在某些情况下会不一致。[#33567](https://github.com/StarRocks/starrocks/pull/33567)
- 执行RESTORE时如果Compaction正好同时在进行，可能引起BE crash。[#32902](https://github.com/StarRocks/starrocks/pull/32902)

## 3.0.7

发布日期：2023年10月18日

### 功能优化

- 窗口函数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMP支持ORDER BY子句和Window子句。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- 主键模型数据写入时的publish过程由异步改为同步，导入作业成功返回后数据立即可见。[#27055](https://github.com/StarRocks/starrocks/pull/27055)
- DECIMAL类型数据查询结果越界时，返回报错而不是NULL。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 执行带有不合法注释的SQL命令时，命令返回结果与MySQL保持一致。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 对于单个分区列的Range分区或者表达式分区的StarRocks表，SQL语句的谓词中包含分区列的表达式也可用于分区裁剪。[#30421](https://github.com/StarRocks/starrocks/pull/30421)

### 问题修复

修复了如下问题：

- 并发进行库和表的创建删除操作，某些情况下会引起表找不到而导致数据写入报错。[#28985](https://github.com/StarRocks/starrocks/pull/28985)
- 某些情况下使用UDF会存在内存泄露。[#29467](https://github.com/StarRocks/starrocks/pull/29467) [#29465](https://github.com/StarRocks/starrocks/pull/29465)
- ORDER BY子句中包含聚合函数时报错“java.lang.IllegalStateException: null”。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 如果Hive Catalog是多级目录，且数据存储在腾讯云COS中，会导致查询结果不正确。[#30363](https://github.com/StarRocks/starrocks/pull/30363)
- 如果ARRAY<STRUCT>类型的数据中STRUCT的某些子列缺失，在读取数据时因为填充默认数据长度错误会导致BE Crash。[#30263](https://github.com/StarRocks/starrocks/pull/30263)
- 升级Berkeley DB Java Edition的版本，避免安全漏洞。[#30029](https://github.com/StarRocks/starrocks/pull/30029)
- 主键模型表导入时，如果Truncate操作和查询并发，在有些情况下会报错“java.lang.NullPointerException”。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- 如果Schema Change执行时间过长，会因为Tablet版本被垃圾回收而失败。 [#31376](https://github.com/StarRocks/starrocks/pull/31376)
- 如果表字段为`NOT NULL`但没有设置默认值，使用CloudCanal导入时会报错“Unsupported dataFormat value is: \N”。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- 存算分离模式下，表的Key信息没有在`information_schema.COLUMNS`中记录，导致使用Flink Connector导入数据时DELETE操作无法执行。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- 升级时如果某些列的类型也升级了（比如Decimal升级到Decimal v3），某些特定特征的表在Compaction时会导致BE crash。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- 使用Flink Connector导入数据时，如果并发高且HTTP和Scan线程数受限，会发生卡死。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- 调用libcurl时会引起BE Crash。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 向主键模型表增加BITMAP类型的列时报错。[#31763](https://github.com/StarRocks/starrocks/pull/31763)

## 3.0.6

发布日期：2023年9月12日

### 行为变更

- 聚合函数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)的分隔符必须使用`SEPARATOR`关键字声明。

### 新增特性

- 聚合函数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)支持使用DISTINCT关键词和ORDER BY子句。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- 分区中数据可以随着时间推移自动进行降冷操作（[List分区方式](../table_design/list_partitioning.md)还不支持）。[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 功能优化

- 对所有复合谓词以及 WHERE 子句中的表达式支持隐式转换，可通过[会话变量](../reference/System_variable.md) `enable_strict_type` 控制是否打开隐式转换（默认取值为 `false`）。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- 统一 FE 和 BE 中 STRING 转换成 INT 的处理逻辑。[#29969](https://github.com/StarRocks/starrocks/pull/29969)

### 问题修复

修复了如下问题：

- 如果 `enable_orc_late_materialization` 设置为 `true`，使用 Hive Catalog 查询 ORC 文件中 STRUCT 类型的数据时结果异常。[#27971](https://github.com/StarRocks/starrocks/pull/27971)
- Hive Catalog 查询时，如果 WHERE 子句中使用分区列且包含 OR 条件，查询结果不正确。 [#28876](https://github.com/StarRocks/starrocks/pull/28876)
- RESTful API `show_data` 对于云原生表的返回信息不正确。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- 如果集群为[存算分离架构](https://docs.starrocks.io/zh/docs/3.0/deployment/deploy_shared_data/)，数据存储在 Azure Blob Storage 上，并且已经建表，则回滚到 3.0 时 FE 无法启动。 [#29433](https://github.com/StarRocks/starrocks/pull/29433)
- 向用户赋予 Iceberg Catalog 下某表权限后，该用户查询该表时显示没有权限。[#29173](https://github.com/StarRocks/starrocks/pull/29173)
- [BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md) 和 [HLL](../sql-reference/sql-statements/data-types/HLL.md) 类型的列在 [SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md) 查询结果中返回的 `Default` 字段值不正确。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- 在线修改 FE 动态参数 `max_broker_load_job_concurrency` 不生效。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- Refresh 物化视图，同时并发地修改物化视图的刷新策略，有概率会导致 FE 无法启动。[#29691](https://github.com/StarRocks/starrocks/pull/29691)
- 执行 `select count(distinct(int+double)) from table_name` 会报错 `unknown error`。 [#30054](https://github.com/StarRocks/starrocks/pull/30054)
- 主键模型表 Restore 之后，BE 重启后元数据发生错误，导致元数据不一致。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 3.0.5

发布日期：2023 年 8 月 16 日

### 新增特性

- 支持聚合函数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)。
- 支持[窗口函数](../sql-reference/sql-functions/Window_function.md) COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMP。

### 功能优化

- 在报错信息 `xxx too many versions xxx` 中增加了如何处理的建议说明。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 动态分区新增支持分区粒度为年。[#28386](https://github.com/StarRocks/starrocks/pull/28386)
- [INSERT OVERWRITE 中使用表达式分区时](../table_design/expression_partitioning.md#%E5%AF%BC%E5%85%A5%E6%95%B0%E6%8D%AE%E8%87%B3%E5%88%86%E5%8C%BA)，分区字段大小写不敏感。[#28309](https://github.com/StarRocks/starrocks/pull/28309)

### 问题修复

修复了如下问题：

- FE 中表级别 scan 统计信息错误，导致表查询和导入的 metrics 信息不正确。 [#27779](https://github.com/StarRocks/starrocks/pull/27779)
- 分区表中修改 sort key 列后查询结果不稳定。[#27850](https://github.com/StarRocks/starrocks/pull/27850)
- Restore 后同一个 tablet 在 BE 和 FE 上的 version 不一致。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- 建 Colocation 表时如果不指定 buckets 数量，则 bucket 数量自动推断为 0，后续添加分区会失败。[#27086](https://github.com/StarRocks/starrocks/pull/27086)
- 当 INSERT INTO SELECT 的 SELECT 结果集为空时，SHOW LOAD 显示导入任务状态为 `CANCELED`。[#26913](https://github.com/StarRocks/starrocks/pull/26913)
- 当 sub_bitmap 函数的输入值不是 BITMAP 类型时，会导致 BE crash。[#27982](https://github.com/StarRocks/starrocks/pull/27982)
- 更新 AUTO_INCREMENT 列时会导致 BE crash。[#27199](https://github.com/StarRocks/starrocks/pull/27199)
- 物化视图 Outer join 和 Anti join 改写错误。 [#28028](https://github.com/StarRocks/starrocks/pull/28028)
- 主键模型部分列更新时平均 row size 预估不准导致内存占用过多。 [#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 激活失效物化视图时可能导致 FE crash。[#27959](https://github.com/StarRocks/starrocks/pull/27959)
- 查询无法改写至基于 Hudi Catalog 外部表创建的物化视图。[#28023](https://github.com/StarRocks/starrocks/pull/28023)
- 删除 Hive 表后，即使手动更新元数据缓存，仍然可以查询到 Hive 表数据。[#28223](https://github.com/StarRocks/starrocks/pull/28223)
- 当异步物化视图的刷新策略为手动刷新且同步调用刷新任务（SYNC MODE）时，手动刷新后 `information_schema.task_runs` 表中有多条 INSERT OVERWRITE 的记录。[#28060](https://github.com/StarRocks/starrocks/pull/28060)
- LabelCleaner 线程卡死导致 FE 内存泄漏。 [#28311](https://github.com/StarRocks/starrocks/pull/28311) [#28636](https://github.com/StarRocks/starrocks/pull/28636)

## 3.0.4

发布日期：2023 年 7 月 18 日

### 新增特性

- 查询和物化视图的 Join 类型不同时，也支持对查询进行改写。[#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 功能优化

- 优化异步物化视图的手动刷新策略。支持通过 REFRESH MATERIALIZED VIEW WITH SYNC MODE 同步调用物化视图刷新任务。[#25904](https://github.com/StarRocks/starrocks/pull/25904)
- 如果查询的字段不包含在物化视图的 output 列但是包含在其谓词条件中，仍可使用该物化视图进行查询改写。[#23028](https://github.com/StarRocks/starrocks/issues/23028)
- [Switch to Trino Syntax](../reference/System_variable.md) `set sql_dialect = 'trino';`, query table aliases are case-insensitive. [#26094](https://github.com/StarRocks/starrocks/pull/26094) [#25282](https://github.com/StarRocks/starrocks/pull/25282)
- `Information_schema.tables_config` table added `table_id` field. You can query the database and table name to which the tablets belong based on the `table_id` field in the `Information_schema` tables `tables_config` and `be_tablets`. [#24061](https://github.com/StarRocks/starrocks/pull/24061)

### Problem Fixes

Fixed the following issues:

- When rewriting the query of the sum aggregate function to a single-table materialized view, the query result may be incorrect due to type inference issues. [#25512](https://github.com/StarRocks/starrocks/pull/25512)
- When using SHOW PROC to view tablet information in the store-separate mode, an error occurs.
- When the inserted data length exceeds the CHAR length defined by the STRUCT, the insertion does not respond. [#25942](https://github.com/StarRocks/starrocks/pull/25942)
- When INSERT INTO SELECT contains FULL JOIN, the returned result is missing. [#26603](https://github.com/StarRocks/starrocks/pull/26603)
- When using the ALTER TABLE command to modify the `default.storage_medium` property of a table, an error `ERROR xxx: Unknown table property xxx` occurs. [#25870](https://github.com/StarRocks/starrocks/issues/25870)
- Broker Load reports an error when importing an empty file. [#26212](https://github.com/StarRocks/starrocks/pull/26212)
- Occasionally, BE offline will freeze. [#26509](https://github.com/StarRocks/starrocks/pull/26509)

## 3.0.3

Release Date: June 28, 2023

### Feature Optimization

- Synchronization of StarRocks external table metadata is now performed during data loading. [#24739](https://github.com/StarRocks/starrocks/pull/24739)
- For tables using [expression partitioning](../table_design/expression_partitioning.md), INSERT OVERWRITE supports specifying the partition. [#25005](https://github.com/StarRocks/starrocks/pull/25005)
- Optimized error messages when adding partitions to non-partitioned tables. [#25266](https://github.com/StarRocks/starrocks/pull/25266)

### Problem Fixes

Fixed the following issues:

- If Parquet files contain complex types, errors will occur when filtering the maximum and minimum values. [#23976](https://github.com/StarRocks/starrocks/pull/23976)
- The library or table has been dropped, but the write task is still in the queue. [#24801](https://github.com/StarRocks/starrocks/pull/24801)
- FE restarts will occasionally cause BE crashes. [#25037](https://github.com/StarRocks/starrocks/pull/25037)
- Import and query occasionally freeze when "enable_profile = true". [#25060](https://github.com/StarRocks/starrocks/pull/25060)
- When the cluster does not have 3 Alive BEs, the error message for INSERT OVERWRITE is inaccurate. [#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 3.0.2

Release Date: June 13, 2023

### Feature Optimization

- After the Union query is rewritten by a materialized view, predicates can also be pushed down. [#23312](https://github.com/StarRocks/starrocks/pull/23312)
- Optimized [automatic bucketing strategy](../table_design/Data_distribution.md#determining-the-number-of-buckets) for tables. [#24543](https://github.com/StarRocks/starrocks/pull/24543)
- Removed the dependency of NetworkTime on the system clock to solve the problem of abnormal estimation of Exchange network time caused by system clock deviation. [#24858](https://github.com/StarRocks/starrocks/pull/24858)

### Problem Fixes

Fixed the following issues:

- Schema change occasionally freezes when data import is also in progress. [#23456](https://github.com/StarRocks/starrocks/pull/23456)
- Query error occurs when `pipeline_profile_level = 0`. [#23873](https://github.com/StarRocks/starrocks/pull/23873)
- Failed to create table when `cloud_native_storage_type` is configured as S3.
- LDAP accounts can log in without passwords. [#24862](https://github.com/StarRocks/starrocks/pull/24862)
- CANCEL LOAD fails when the table does not exist. [#24922](https://github.com/StarRocks/starrocks/pull/24922)

### Upgrade Notes

- If there is a database named `starrocks` in your system, please rename it using ALTER DATABASE RENAME before upgrading.
- 支持[导入时自动创建分区和使用分区表达式定义分区规则](https://docs.starrocks.io/zh-cn/3.0/table_design/automatic_partitioning)，提高了分区创建的易用性和灵活性。
- Primary Key 模型表支持更丰富的[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md) 和[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md) 语法，包括使用 CTE 和对多表的引用。
- Broker Load 和 INSERT INTO 增加 Load Profile，支持通过 profile 查看并分析导入作业详情。使用方法与[查看分析Query Profile](../administration/query_profile.md) 相同。

**数据湖分析**

- [Preview] 支持 Presto/Trino 兼容模式，可以自动改写 Presto/Trino 的 SQL。参见[系统变量](../reference/System_variable.md) 中的 `sql_dialect`。
- [Preview] 支持[JDBC Catalog](../data_source/catalog/jdbc_catalog.md)。
- 支持使用[SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 命令来手动选择 Catalog。

**权限与安全**

- 提供了新版的完整 RBAC 功能，支持角色的继承和默认角色。更多信息，参见[权限系统总览](../administration/privilege_overview.md)。
- 提供更多权限管理对象和更细粒度的权限。更多信息，参见[权限项](../administration/privilege_item.md)。

**查询**

<!-- - [Preview] 支持大查询的算子落盘，可以在内存不足时利用磁盘空间来保证查询稳定执行成功。
- [Query Cache](../using_starrocks/query_cache.md) 支持更多使用场景，包括各种 Broadcast Join、Bucket Shuffle Join 场景。-->
- 支持[Global UDF](../sql-reference/sql-functions/JAVA_UDF.md).
- 动态自适应并行度，可以根据查询并发度自适应调节`pipeline_dop`.

**SQL 语句和函数**

- 新增如下权限相关 SQL 语句：[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md)、[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md)、[SHOW USERS](../sql-reference/sql-statements/account-management/SHOW_USERS.md)。
- 新增半结构化数据分析相关函数：[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)、[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md).
- [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md) 支持 ORDER BY。
- 窗口函数[lead](../sql-reference/sql-functions/Window_function.md#使用-lead-窗口函数)、[lag](../sql-reference/sql-functions/Window_function.md#使用-lag-窗口函数) 支持 IGNORE NULLS。
- 新增[BINARY/VARBINARY 数据类型](../sql-reference/sql-statements/data-types/BINARY.md)，新增[to_binary](../sql-reference/sql-functions/binary-functions/to_binary.md)、[from_binary](../sql-reference/sql-functions/binary-functions/from_binary.md) 函数。
- 新增字符串函数[replace](../sql-reference/sql-functions/string-functions/replace.md)、[hex_decode_binary](../sql-reference/sql-functions/string-functions/hex_decode_binary.md)、[hex_decode_string](../sql-reference/sql-functions/string-functions/hex_decode_string.md).
- 新增加密函数[base64_decode_binary](../sql-reference/sql-functions/crytographic-functions/base64_decode_binary.md)、[base64_decode_string](../sql-reference/sql-functions/crytographic-functions/base64_decode_string.md).
- 新增数学函数[sinh](../sql-reference/sql-functions/math-functions/sinh.md)、[cosh](../sql-reference/sql-functions/math-functions/cosh.md)、[tanh](../sql-reference/sql-functions/math-functions/tanh.md).
- 新增工具函数[current_role](../sql-reference/sql-functions/utility-functions/current_role.md).

### 功能优化

**部署**

- 更新 3.0 版本的 Docker 镜像和相关[部署文档](../quick_start/deploy_with_docker.md)。 [#20623](https://github.com/StarRocks/starrocks/pull/20623) [#21021](https://github.com/StarRocks/starrocks/pull/21021)

**存储与导入**

- 数据导入提供了更丰富的 CSV 格式参数，包括`skip_header`、`trim_space`、`enclose` 和`escape`。参见[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 和[ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)。
- [Primary Key 模型表](../table_design/table_types/primary_key_table.md)解耦了主键和排序键，支持通过`ORDER BY`单独指定排序键。
- 优化 Primary Key 模型表在大数据导入、部分列更新、以及开启持久化索引等场景的内存占用。 [#12068](https://github.com/StarRocks/starrocks/pull/12068) [#14187](https://github.com/StarRocks/starrocks/pull/14187) [#15729](https://github.com/StarRocks/starrocks/pull/15729)
- 提供异步 ETL 命令接口，支持创建异步 INSERT 任务。更多信息，参考[INSERT](../loading/InsertInto.md) 和[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)。 ([#20609](https://github.com/StarRocks/starrocks/issues/20609))

**物化视图**

- 优化[物化视图](../using_starrocks/Materialized_view.md)的改写能力：
  - 支持 view delta join, 可以改写。
  - 支持对 Outer Join 和 Cross Join 的改写。
  - 优化带分区时 UNION 的 SQL rewrite。
- 完善物化视图的构建能力：支持 CTE、SELECT * 、UNION。
- 优化[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)命令的返回信息。
- 提升物化视图构建时的分区创建效率。([#21167](https://github.com/StarRocks/starrocks/pull/21167))

**查询**

- 完善算子对 Pipeline 的支持，所有算子都支持 Pipeline。
- 完善[大查询定位](../administration/monitor_manage_big_queries.md)。[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md) 支持查看 CPU 内存信息，增加大查询日志。
- 优化 Outer Join Reorder 能力。
- 优化 SQL 解析阶段的报错信息，查询的报错位置更明确，信息更清晰。

**数据湖分析**

- 优化元数据统计信息收集。
- Hive Catalog、Iceberg Catalog、Hudi Catalog 和 Delta Lake Catalog 支持通过[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)查看外表的创建信息。

### 问题修复

修复了以下问题：

- StarRocks 源文件的许可 header 中部分 url 无法访问。[#2224](https://github.com/StarRocks/starrocks/issues/2224)
- SELECT 查询出现 unknown error。 [#19731](https://github.com/StarRocks/starrocks/issues/19731)
- 支持 SHOW/SET CHARACTER。 [#17480](https://github.com/StarRocks/starrocks/issues/17480)
- 当导入的内容超过 StarRocks 字段长度时，ErrorURL 提示信息不准确。提示源端字段为空，而不是内容超长。 [#14](https://github.com/StarRocks/DataX/issues/14)
- 支持 show full fields from 'table'。 [#17233](https://github.com/StarRocks/starrocks/issues/17233)
- Partition pruning causing MV rewrite failures. [#14641](https://github.com/StarRocks/starrocks/issues/14641)
- MV rewrite fails when the MV creation statement contains count(distinct) and count(distinct) operates on the distribution column. [#16558](https://github.com/StarRocks/starrocks/issues/16558)
- Using VARCHAR as the partition column of a materialized view causing FE to start abnormally. [#19366](https://github.com/StarRocks/starrocks/issues/19366)
- Incorrect handling of IGNORE NULLS for window functions [lead](../sql-reference/sql-functions/Window_function.md#使用-lead-窗口函数) and [lag](../sql-reference/sql-functions/Window_function.md#使用-lag-窗口函数). [#21001](https://github.com/StarRocks/starrocks/pull/21001)
- Conflicts between inserting temporary partitions and automatic partition creation. [#21222](https://github.com/StarRocks/starrocks/issues/21222)

### Behavior Changes

- After RBAC upgrade, it will be compatible with previous users and privileges, but there are significant changes in related syntax such as [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) and [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md).
- Renamed SHOW MATERIALIZED VIEW to [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md).
- The following [reserved keywords](../sql-reference/sql-statements/keywords.md) have been added: AUTO_INCREMENT, CURRENT_ROLE, DEFERRED, ENCLOSE, ESCAPE, IMMEDIATE, PRIVILEGES, SKIP_HEADER, TRIM_SPACE, VARBINARY.

### Upgrade Considerations

**It is mandatory to upgrade from version 2.5 to 3.0, otherwise rollback will not be supported (in theory, upgrading from versions below 2.5 to 3.0 is also supported. But for safety, please upgrade from 2.5 to reduce risks.)**

#### BDBJE Upgrade

The 3.0 version has upgraded the BDB library, but BDB does not support rollback, so the BDB library in version 3.0 needs to be used even after rollback.

1. After replacing the FE package with the old version, place `fe/lib/starrocks-bdb-je-18.3.13.jar` from version 3.0 into the `fe/lib` directory of version 2.5.
2. Delete `fe/lib/je-7.*.jar`.

#### Permission System Upgrade

Upgrading to 3.0 will default to using the new RBAC permission management system, and after the upgrade, only rollback to 2.5 will be supported.

After rollback, you need to manually execute the [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) command to trigger the creation of a new image and wait for the image to synchronize to all follower nodes. If this command is not executed, it may cause some rollbacks to fail. ALTER SYSTEM CREATE IMAGE is supported in version 2.5.3 and later.