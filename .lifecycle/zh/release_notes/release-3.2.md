---
displayed_sidebar: English
---

# 星石版本 3.2

## v3.2.0-RC01

发布日期：2023年11月15日

### 新特性

#### 共享数据集群

- 支持在[本地磁盘](../table_design/table_types/primary_key_table.md)上为主键表创建持久化索引。
- 支持在多个本地磁盘之间均匀分配数据缓存。

#### 数据湖分析

- 支持在[Hive目录](../data_source/catalog/hive_catalog.md)中创建和删除数据库及托管表，并支持使用INSERT或INSERT OVERWRITE命令将数据导出到Hive的托管表。
- 支持[统一目录](../data_source/catalog/unified_catalog.md)，用户可以访问不同的表格式（Hive、Iceberg、Hudi和Delta Lake），这些表格式共享一个公共的元数据存储，如Hive元数据存储或AWS Glue。

#### 存储引擎、数据摄取和导出

- 新增以下通过表函数[FILES()](../sql-reference/sql-functions/table-functions/files.md)进行数据加载的特性：
  - 从Azure或GCP加载Parquet和ORC格式的数据。
  - 使用参数columns_from_path从文件路径中提取键/值对，作为列的值。
  - 加载包括ARRAY、JSON、MAP和STRUCT在内的复杂数据类型。
- 支持dict_mapping列属性，它可以在构建全局字典期间显著简化数据加载过程，加快精确COUNT DISTINCT计算的速度。
- 支持使用INSERT INTO FILES将数据从StarRocks卸载到存储在AWS S3或HDFS中的Parquet格式文件。详细说明请参见[Unload data using INSERT INTO FILES](../unloading/unload_using_insert_into_files.md)。

#### SQL参考

新增以下函数：

- 字符串函数：substring_index、url_extract_parameter、url_encode、url_decode和translate
- 日期函数：dayofweek_iso、week_iso、quarters_add、quarters_sub、milliseconds_add、milliseconds_sub、date_diff、jodatime_format、str_to_jodatime、to_iso8601、to_tera_date和to_tera_timestamp
- 模式匹配函数：regexp_extract_all
- 哈希函数：xx_hash3_64
- 聚合函数：approx_top_k
- 窗口函数：cume_dist、percent_rank和session_number
- 实用函数：dict_mapping和get_query_profile

#### 权限和安全性

StarRocks通过Apache Ranger支持访问控制，提供更高级别的数据安全性，并允许复用外部数据源的现有Ranger服务。集成Apache Ranger之后，StarRocks可以实施以下访问控制方式：

- 当访问StarRocks中的内部表、外部表或其他对象时，可以基于在Ranger中为StarRocks服务配置的访问策略执行访问控制。
- 当访问外部目录时，也可以借助原始数据源的Ranger服务（例如Hive服务）进行访问控制（目前尚不支持将数据导出到Hive的访问控制）。

更多信息请参见[使用Apache Ranger管理权限](../administration/ranger_plugin.md)。

### 改进

#### 物化视图

异步物化视图

- 创建：

  支持在视图或物化视图上创建的异步物化视图在视图、物化视图或其基础表发生模式变更时自动刷新。

- 可观察性：

  支持异步物化视图的查询转储功能。

- 异步物化视图的刷新任务默认启用磁盘溢出功能，以减少内存消耗。
- 数据一致性：

  - 新增了用于创建异步物化视图的query_rewrite_consistency属性。此属性定义了基于一致性检查的查询重写规则。
  - 新增了force_external_table_query_rewrite属性，用于基于外部目录创建异步物化视图。该属性定义是否允许对基于外部目录创建的异步物化视图进行强制查询重写。
  详细信息请参见[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)。

- 为物化视图的分区键添加了一致性检查。

  当用户创建包含PARTITION BY表达式的窗口函数的异步物化视图时，窗口函数的分区列必须与物化视图的分区列匹配。

#### 存储引擎、数据摄取和导出

- 优化了主键表的持久化索引，提高了内存使用效率，同时降低了I/O读写放大。[#24875](https://github.com/StarRocks/starrocks/pull/24875)  [#27577](https://github.com/StarRocks/starrocks/pull/27577)  [#28769](https://github.com/StarRocks/starrocks/pull/28769)
- 支持主键表数据在本地磁盘之间的重新分配。
- 分区表支持基于分区时间范围和冷却时间自动冷却。详细信息请参见[设置初始存储介质和自动存储冷却时间](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#specify-initial-storage-medium-automatic-storage-cooldown-time-replica-number)。
- 将数据写入主键表的加载作业的发布阶段从异步模式改为同步模式。这样，加载作业完成后，可以立即查询到加载的数据。详细信息请参见[enable_sync_publish](../administration/FE_configuration.md#enable_sync_publish)。

#### 查询

- 优化了StarRocks与Metabase和Superset的兼容性，并支持将它们与外部目录集成。

#### SQL参考

- array_agg支持DISTINCT关键字。

### 开发者工具

- 支持异步物化视图的Trace Query Profile，用于分析其透明重写。

### 兼容性变更

#### 行为变更

待更新。

#### 参数

- 为数据缓存新增了新参数。

#### 系统变量

待更新。

### Bug修复

修复了以下问题：

- BEs crash when libcurl is invoked. [#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 如果耗时过长，Schema Change可能会失败，因为指定的平板电脑版本已被垃圾回收处理。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- 无法通过文件外部表访问MinIO或AWS S3中的Parquet文件。[#29873](https://github.com/StarRocks/starrocks/pull/29873)
- 在 `information_schema.columns` 中，ARRAY、MAP 和 STRUCT 类型的列显示不正确。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- 在`information_schema.columns`视图中，BINARY或VARBINARY数据类型的`DATA_TYPE`和`COLUMN_TYPE`显示为`未知`。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### 升级须知

待更新。
