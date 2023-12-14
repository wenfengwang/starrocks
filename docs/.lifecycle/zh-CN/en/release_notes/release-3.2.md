---
displayed_sidebar: "Chinese"
---

# StarRocks 版本 3.2

## v3.2.0-RC01

发布日期：2023年11月15日

### 新功能

#### 共享数据集群

- 支持在本地磁盘上为主键表持久索引](../table_design/table_types/primary_key_table.md)。
- 支持将数据缓存均匀分布在多个本地磁盘上。

#### 数据湖分析

- 支持在[Hive目录](../data_source/catalog/hive_catalog.md)中创建和删除数据库和托管表，并支持使用INSERT或INSERT OVERWRITE将数据导出到Hive的托管表。
- 支持[Unified Catalog](../data_source/catalog/unified_catalog.md)，用户可以访问使用共同的元数据存储（如Hive元数据存储或AWS Glue）的不同表格式（Hive、Iceberg、Hudi和Delta Lake）。

#### 存储引擎、数据摄入和导出

- 增加了使用表函数[FILES()](../sql-reference/sql-functions/table-functions/files.md)的以下功能：
  - 从Azure或GCP加载Parquet和ORC格式的数据。
  - 使用参数`columns_from_path`从文件路径提取键值对的值作为列的值。
  - 加载包括ARRAY、JSON、MAP和STRUCT在内的复杂数据类型。
- 支持dict_mapping列属性，可以在构建全局字典时显着简化加载过程，加速确切的COUNT DISTINCT计算。
- 通过使用INSERT INTO FILES，支持从StarRocks将数据卸载到存储在AWS S3或HDFS中的Parquet格式文件。有关详细说明，请参见[使用INSERT INTO FILES卸载数据](../unloading/unload_using_insert_into_files.md)。

#### SQL参考

增加了以下函数：

- 字符串函数：substring_index、url_extract_parameter、url_encode、url_decode和translate
- 日期函数：dayofweek_iso、week_iso、quarters_add、quarters_sub、milliseconds_add、milliseconds_sub、date_diff、jodatime_format、str_to_jodatime、to_iso8601、to_tera_date和to_tera_timestamp
- 模式匹配函数：regexp_extract_all
- 哈希函数：xx_hash3_64
- 聚合函数：approx_top_k
- 窗口函数：cume_dist、percent_rank和session_number
- 实用函数：dict_mapping和get_query_profile

#### 权限和安全

StarRocks通过Apache Ranger支持访问控制，提供更高级别的数据安全，并允许重用外部数据源的现有Ranger Service。集成Apache Ranger后，StarRocks可以实现以下访问控制方法：

- 访问StarRocks中的内部表、外部表或其他对象时，可以根据Ranger中为StarRocks服务配置的访问策略来强制执行访问控制。
- 访问外部目录时，访问控制还可以利用原始数据源的相应Ranger服务（如Hive服务）来控制访问（当前尚不支持将数据导出到Hive的访问控制）。

有关更多信息，请参见[使用Apache Ranger管理权限](../administration/ranger_plugin.md)。

### 改进

#### 材料化视图

异步材料化视图

- 创建：

  支持在视图或材料化视图发生架构更改时自动刷新异步材料化视图。
  
- 可观察性：

  支持异步材料化视图的查询转储。

- 默认启用Spill to Disk功能以减少异步材料化视图刷新任务的内存消耗。
- 数据一致性：

  - 为异步材料化视图创建添加了`query_rewrite_consistency`属性。该属性基于一致性检查定义查询重写规则。
  - 为基于外部目录的异步材料化视图创建添加了`force_external_table_query_rewrite`属性。该属性定义是否允许强制对异步材料化视图进行查询重写。

  有关详细信息，请参见[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)。

- 为材料化视图的分区键添加了一致性检查。

  用户创建包含PARTITION BY表达式的具有窗口函数的异步材料化视图时，窗口函数的分区列必须与材料化视图的分区列相匹配。

#### 存储引擎、数据摄入和导出

- 通过改进内存使用逻辑，优化了主键表的持久索引，减少了I/O读写放大。[#24875](https://github.com/StarRocks/starrocks/pull/24875)  [#27577](https://github.com/StarRocks/starrocks/pull/27577)  [#28769](https://github.com/StarRocks/starrocks/pull/28769)
- 支持在本地磁盘上对主键表进行数据重新分布。
- 分区表支持基于分区时间范围和冷却时间的自动冷却。有关详细信息，请参见[指定初始存储介质和自动存储冷却时间复制的数量](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#specify-initial-storage-medium-automatic-storage-cooldown-time-replica-number)。
- 写入数据到主键表的加载作业的Publish阶段从异步模式更改为同步模式。因此，数据加载完成后可立即查询。有关详细信息，请参见[enable_sync_publish](../administration/Configuration.md#enable_sync_publish)。

#### 查询

- 优化了StarRocks与Metabase和Superset的兼容性。支持与外部目录集成。

#### SQL 参考

- array_agg支持关键字DISTINCT。

### 开发人员工具

- 支持异步材料化视图的Trace Query Profile，可用于分析其透明重写。

### 兼容性更改

#### 行为更改

待更新。

#### 参数

- 为Data Cache添加新参数。

#### 系统变量

待更新。

### Bug 修复

修复了以下问题：

- 当调用libcurl时，BE会崩溃。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 如果指定的tablet版本由垃圾收集处理，那么架构更改可能会失败，因为它花费了过长的时间。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- 通过文件外部表无法访问MinIO或AWS S3中的Parquet文件。[#29873](https://github.com/StarRocks/starrocks/pull/29873)
- `information_schema.columns`中未正确显示ARRAY、MAP和STRUCT类型的列。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- `information_schema.columns`视图中BINARY或VARBINARY数据类型的`DATA_TYPE`和`COLUMN_TYPE`显示为`unknown`。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### 升级注意事项

待更新。