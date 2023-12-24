---
displayed_sidebar: English
---

# StarRocks 3.2 版本

## v3.2.0-RC01

发布日期：2023年11月15日

### 新功能

#### 共享数据集群

- 支持在本地磁盘上为主键表创建持久索引，详情请参见[主键表的持久索引](../table_design/table_types/primary_key_table.md)。
- 支持将数据缓存均匀分布在多个本地磁盘之间。

#### 数据湖分析

- 支持在[Hive目录](../data_source/catalog/hive_catalog.md)中创建和删除数据库以及托管表，并支持使用INSERT或INSERT OVERWRITE将数据导出到Hive的托管表。
- 支持[统一目录](../data_source/catalog/unified_catalog.md)，用户可以访问不同的表格式（Hive、Iceberg、Hudi和Delta Lake），这些表格式共享一个类似于Hive元存储或AWS Glue的通用元存储。

#### 存储引擎、数据引入和导出

- 添加了使用表函数[FILES()](../sql-reference/sql-functions/table-functions/files.md)加载的以下功能：
  - 从Azure或GCP加载Parquet和ORC格式的数据。
  - 使用参数`columns_from_path`从文件路径中提取键/值对的值作为列的值。
  - 加载包括ARRAY、JSON、MAP和STRUCT在内的复杂数据类型。
- 支持dict_mapping列属性，可以显著简化全局字典构建过程中的加载过程，加速精确的COUNT DISTINCT计算。
- 支持使用INSERT INTO FILES将StarRocks中的数据卸载到存储在AWS S3或HDFS中的Parquet格式文件。有关详细说明，请参见[使用INSERT INTO FILES卸载数据](../unloading/unload_using_insert_into_files.md)。

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

StarRocks通过Apache Ranger支持访问控制，提供更高级别的数据安全性，并允许重用外部数据源的现有Ranger Service。与Apache Ranger集成后，StarRocks支持以下访问控制方法：

- 访问StarRocks中的内部表、外部表或其他对象时，可以根据为Ranger配置的StarRocks Service的访问策略强制执行访问控制。
- 访问外部目录时，访问控制还可以利用原始数据源的相应Ranger服务（如Hive服务）来控制访问（目前，尚不支持将数据导出到Hive的访问控制）。

有关更多信息，请参见[使用Apache Ranger管理权限](../administration/ranger_plugin.md)。

### 改进

#### 物化视图

异步物化视图

- 创建：

  支持在视图或物化视图发生架构更改时，自动刷新异步物化视图。

- 可观察性：

  支持异步物化视图的查询转储。

- 异步物化视图的刷新任务默认启用了磁盘溢出功能，从而减少内存消耗。
- 数据一致性：

  - 为异步物化视图创建添加了`query_rewrite_consistency`属性，该属性基于一致性检查定义了查询重写规则。
  - 为基于外部目录的异步物化视图创建添加了`force_external_table_query_rewrite`属性，该属性定义了是否允许强制查询重写。

  有关详细信息，请参见[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)。

- 添加了对物化视图的分区键的一致性检查。

  当用户使用包含PARTITION BY表达式的窗口函数创建异步物化视图时，窗口函数的分区列必须与物化视图的分区列匹配。

#### 存储引擎、数据引入和导出

- 通过改进内存使用逻辑和减少I/O读写放大，优化了主键表的持久索引。[#24875](https://github.com/StarRocks/starrocks/pull/24875)  [#27577](https://github.com/StarRocks/starrocks/pull/27577)  [#28769](https://github.com/StarRocks/starrocks/pull/28769)
- 支持主键表在本地磁盘间重新分发数据。
- 分区表支持根据分区时间范围和冷却时间自动冷却。有关详细信息，请参见[设置初始存储介质和自动存储冷却时间](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#specify-initial-storage-medium-automatic-storage-cooldown-time-replica-number)。
- 将写入主键表的加载作业的发布阶段从异步模式更改为同步模式。因此，加载作业完成后，可以立即查询加载的数据。有关详细信息，请参见[enable_sync_publish](../administration/FE_configuration.md#enable_sync_publish)。

#### 查询

- 优化了StarRocks与Metabase和Superset的兼容性。支持将它们与外部目录集成。

#### SQL参考

- array_agg支持关键字DISTINCT。

### 开发人员工具

- 支持对异步物化视图进行跟踪查询配置文件，可用于分析其透明重写。

### 兼容性更改

#### 行为更改

待更新。

#### 参数

- 为数据缓存添加了新参数。

#### 系统变量

待更新。

### Bug修复

修复了以下问题：

- BE在调用libcurl时崩溃。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 如果架构更改花费的时间过长，则可能会失败，因为指定的平板电脑版本由垃圾回收处理。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- 无法通过文件外部表访问MinIO或AWS S3中的Parquet文件。[#29873](https://github.com/StarRocks/starrocks/pull/29873)
- ARRAY、MAP和STRUCT类型列在`information_schema.columns`中未正确显示。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- `information_schema.columns`中BINARY或VARBINARY数据类型的`DATA_TYPE`和`COLUMN_TYPE`显示为`unknown`。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### 升级说明

待更新。
