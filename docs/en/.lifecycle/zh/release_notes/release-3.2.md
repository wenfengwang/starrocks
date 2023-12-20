---
displayed_sidebar: English
---

# StarRocks 版本 3.2

## v3.2.0-RC01

发布日期：2023 年 11 月 15 日

### 新特性

#### 共享数据集群

- 支持在本地磁盘上[持久化主键表的索引](../table_design/table_types/primary_key_table.md)。
- 支持在多个本地磁盘之间均匀分布 Data Cache。

#### 数据湖分析

- 支持在 [Hive 目录](../data_source/catalog/hive_catalog.md)中创建和删除数据库和托管表，并支持使用 INSERT 或 INSERT OVERWRITE 将数据导出到 Hive 的托管表。
- 支持 [统一目录](../data_source/catalog/unified_catalog.md)，用户可以访问不同的表格格式（Hive、Iceberg、Hudi 和 Delta Lake），这些格式共享一个公共元存储，如 Hive 元存储或 AWS Glue。

#### 存储引擎、数据摄取和导出

- 添加了以下使用表函数 [FILES()](../sql-reference/sql-functions/table-functions/files.md) 加载的特性：
  - 从 Azure 或 GCP 加载 Parquet 和 ORC 格式数据。
  - 使用参数 `columns_from_path` 从文件路径中提取键/值对的值作为列的值。
  - 加载复杂数据类型，包括 ARRAY、JSON、MAP 和 STRUCT。
- 支持 dict_mapping 列属性，这可以显著简化构建全局字典过程中的加载步骤，加速精确 COUNT DISTINCT 计算。
- 支持使用 INSERT INTO FILES 将数据从 StarRocks 卸载到存储在 AWS S3 或 HDFS 中的 Parquet 格式文件。详细说明请参见[使用 INSERT INTO FILES 卸载数据](../unloading/unload_using_insert_into_files.md)。

#### SQL 参考

新增以下函数：

- 字符串函数：substring_index、url_extract_parameter、url_encode、url_decode 和 translate
- 日期函数：dayofweek_iso、week_iso、quarters_add、quarters_sub、milliseconds_add、milliseconds_sub、date_diff、jodatime_format、str_to_jodatime、to_iso8601、to_tera_date 和 to_tera_timestamp
- 模式匹配函数：regexp_extract_all
- 哈希函数：xx_hash3_64
- 聚合函数：approx_top_k
- 窗口函数：cume_dist、percent_rank 和 session_number
- 实用函数：dict_mapping 和 get_query_profile

#### 权限和安全

StarRocks 通过 Apache Ranger 支持访问控制，提供更高级别的数据安全性，并允许重用外部数据源的现有 Ranger 服务。与 Apache Ranger 集成后，StarRocks 启用以下访问控制方法：

- 当访问 StarRocks 中的内部表、外部表或其他对象时，可以根据 Ranger 中为 StarRocks 服务配置的访问策略执行访问控制。
- 当访问外部目录时，也可以利用原始数据源的对应 Ranger 服务（例如 Hive 服务）来控制访问（目前尚不支持将数据导出到 Hive 的访问控制）。

更多信息，请参见[使用 Apache Ranger 管理权限](../administration/ranger_plugin.md)。

### 改进

#### 物化视图

异步物化视图

- 创建：

  支持在视图或物化视图上创建的异步物化视图在视图、物化视图或其基表发生模式变化时自动刷新。

- 可观测性：

  支持异步物化视图的查询转储。

- 异步物化视图的刷新任务默认启用磁盘溢出功能，减少内存消耗。
- 数据一致性：

  - 新增属性 `query_rewrite_consistency` 用于异步物化视图创建。该属性定义了基于一致性检查的查询重写规则。
  - 新增属性 `force_external_table_query_rewrite` 用于基于外部目录的异步物化视图创建。该属性定义是否允许对基于外部目录创建的异步物化视图强制查询重写。
  详细信息请参见[创建物化视图](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)。

- 新增物化视图分区键的一致性检查。

  当用户创建包含 PARTITION BY 表达式的窗口函数的异步物化视图时，窗口函数的分区列必须与物化视图的分区列匹配。

#### 存储引擎、数据摄取和导出

- 优化了主键表的持久化索引，改进了内存使用逻辑，减少了 I/O 读写放大。[#24875](https://github.com/StarRocks/starrocks/pull/24875) [#27577](https://github.com/StarRocks/starrocks/pull/27577) [#28769](https://github.com/StarRocks/starrocks/pull/28769)
- 支持主键表跨本地磁盘的数据重新分配。
- 分区表支持基于分区时间范围和冷却时间的自动冷却。详细信息请参见[设置初始存储介质和自动存储冷却时间](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#specify-initial-storage-medium-automatic-storage-cooldown-time-replica-number)。
- 将数据写入主键表的加载作业的发布阶段从异步模式改为同步模式，使得加载作业完成后可以立即查询到加载的数据。详细信息请参见[enable_sync_publish](../administration/FE_configuration.md#enable_sync_publish)。

#### 查询

- 优化了 StarRocks 与 Metabase 和 Superset 的兼容性。支持将它们与外部目录集成。

#### SQL 参考

- array_agg 函数支持 DISTINCT 关键字。

### 开发者工具

- 支持异步物化视图的 Trace Query Profile，用于分析其透明重写。

### 兼容性变更

#### 行为变化

待更新。

#### 参数

- 新增了 Data Cache 的参数。

#### 系统变量

待更新。

### Bug 修复

修复了以下问题：

- 当调用 libcurl 时 BEs 崩溃。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 如果耗时过长，Schema Change 可能会失败，因为指定的 tablet 版本被垃圾回收处理了。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- 无法通过文件外部表访问 MinIO 或 AWS S3 中的 Parquet 文件。[#29873](https://github.com/StarRocks/starrocks/pull/29873)
- 在 `information_schema.columns` 中 ARRAY、MAP 和 STRUCT 类型的列显示不正确。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- 在 `information_schema.columns` 视图中，BINARY 或 VARBINARY 数据类型的 `DATA_TYPE` 和 `COLUMN_TYPE` 显示为 `unknown`。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### 升级须知

待更新。