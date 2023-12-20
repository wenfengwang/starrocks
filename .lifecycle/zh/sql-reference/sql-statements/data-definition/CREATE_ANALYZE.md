---
displayed_sidebar: English
---

# 创建分析任务

## 描述

自定义自动采集任务，用于搜集CBO统计信息。

默认情况下，StarRocks会自动搜集表的全部统计信息，并且每5分钟检查一次是否有数据更新。如果侦测到数据变动，系统将自动触发数据搜集。如果您不希望使用自动的全量搜集功能，可以将FE配置项enable_collect_full_statistic设为false，并自行定制搜集任务。

在创建自定义的自动搜集任务之前，必须先关闭自动的全量搜集（enable_collect_full_statistic = false），否则自定义任务将不会生效。

该指令从v2.4版本开始支持。

## 语法

```SQL
-- Automatically collect stats of all databases.
CREATE ANALYZE [FULL|SAMPLE] ALL PROPERTIES (property [,property])

-- Automatically collect stats of all tables in a database.
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name
PROPERTIES (property [,property])

-- Automatically collect stats of specified columns in a table.
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
PROPERTIES (property [,property])
```

## 参数描述

- 搜集类型
  - FULL：指全量搜集。
  - SAMPLE：指采样搜集。
  - 如果未指定搜集类型，默认将采用全量搜集。

- col_name：需要搜集统计信息的列名。使用逗号（,）分隔多个列名。如果未指定此参数，则会搜集整个表的数据。

- PROPERTIES：自定义参数。如果未指定PROPERTIES，则将采用fe.conf中的默认设置。实际使用的属性可以通过SHOW ANALYZE JOB命令输出中的Properties列来查看。

|属性|类型|默认值|描述|
|---|---|---|---|
|statistic_auto_collect_ratio|FLOAT|0.8|判断自动收集统计信息是否健康的阈值。如果统计信息健康状况低于此阈值，则会触发自动收集。|
|statistics_max_full_collect_data_size|INT|100|自动收集收集数据的最大分区的大小。单位：GB。如果分区超过此值，则放弃全量采集，改为采样采集。|
|statistic_sample_collect_rows|INT|200000|要收集的最小行数。如果参数值超过表中的实际行数，则执行完整收集。|

## 示例

示例1：自动全量搜集

```SQL
-- Automatically collect full stats of all databases.
CREATE ANALYZE ALL;

-- Automatically collect full stats of a database.
CREATE ANALYZE DATABASE db_name;

-- Automatically collect full stats of all tables in a database.
CREATE ANALYZE FULL DATABASE db_name;

-- Automatically collect full stats of specified columns in a table.
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 
```

示例2：自动采样搜集

```SQL
-- Automatically collect stats of all tables in a database using default settings.
CREATE ANALYZE SAMPLE DATABASE db_name;

-- Automatically collect stats of specified columns in a table, with statistics health and the number of rows to collect specified.
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES(
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);
```

## 参考资料

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)：查看自定义搜集任务的状态。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md)：删除一个自定义收集任务。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：终止正在执行的自定义搜集任务。

有关搜集CBO统计信息的更多资料，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。
