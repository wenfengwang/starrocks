---
displayed_sidebar: English
---

# CREATE ANALYZE

## 描述

自定义自动采集任务，用于收集CBO统计数据。

默认情况下，StarRocks会自动收集表的完整统计信息。它每5分钟检查一次数据更新。如果检测到数据变化，将自动触发数据收集。如果您不想使用自动全量采集，可以将FE配置项`enable_collect_full_statistic`设置为`false`，并自定义采集任务。

在创建自定义自动采集任务之前，您必须关闭自动全量采集（`enable_collect_full_statistic = false`）。否则，自定义任务将无法生效。

该语句从v2.4版本开始支持。

## 语法

```SQL
-- 自动收集所有数据库的统计信息。
CREATE ANALYZE [FULL|SAMPLE] ALL PROPERTIES (property [,property])

-- 自动收集数据库中所有表的统计信息。
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name
PROPERTIES (property [,property])

-- 自动收集表中指定列的统计信息。
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
PROPERTIES (property [,property])
```

## 参数描述

- 收集类型
  - FULL：表示全量收集。
  - SAMPLE：表示采样收集。
  - 如果未指定收集类型，默认为全量收集。

- `col_name`：需要收集统计信息的列。用逗号（`,`）分隔多个列。如果未指定此参数，则收集整个表的统计信息。

- `PROPERTIES`：自定义参数。如果未指定`PROPERTIES`，则使用`fe.conf`中的默认设置。实际使用的属性可以通过`SHOW ANALYZE JOB`的输出中的`Properties`列查看。

|**PROPERTIES**|**类型**|**默认值**|**描述**|
|---|---|---|---|
|statistic_auto_collect_ratio|FLOAT|0.8|用于判断自动收集的统计信息是否健康的阈值。如果统计信息的健康度低于此阈值，将触发自动收集。|
|statistics_max_full_collect_data_size|INT|100|自动收集可以收集数据的最大分区大小。单位：GB。如果分区超过此值，将放弃全量收集，改为采样收集。|
|statistic_sample_collect_rows|INT|200000|要收集的最小行数。如果参数值超过您表中的实际行数，将执行全量收集。|

## 示例

示例1：自动全量收集

```SQL
-- 自动收集所有数据库的完整统计信息。
CREATE ANALYZE ALL;

-- 自动收集一个数据库的完整统计信息。
CREATE ANALYZE DATABASE db_name;

-- 自动收集数据库中所有表的完整统计信息。
CREATE ANALYZE FULL DATABASE db_name;

-- 自动收集表中指定列的完整统计信息。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 
```

示例2：自动采样收集

```SQL
-- 使用默认设置自动收集数据库中所有表的统计信息。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- 自动收集表中指定列的统计信息，并指定统计信息健康度和要收集的行数。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES(
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);
```

## 参考资料

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)：查看自定义采集任务的状态。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md)：删除自定义采集任务。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：取消正在执行的自定义采集任务。

有关为CBO收集统计信息的更多信息，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。