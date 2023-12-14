---
displayed_sidebar: "Chinese"
---

# 创建收集

## 描述

自定义自动收集CBO统计信息的收集任务。

默认情况下，StarRocks会自动收集表的完整统计信息，并每5分钟检查数据更新。如果检测到数据变更，将自动触发数据收集。如果您不希望使用自动完整收集，可以将FE配置项`enable_collect_full_statistic`设置为`false`，并自定义收集任务。

在创建自定义的自动收集任务之前，必须禁用自动完整收集(`enable_collect_full_statistic = false`)。否则，自定义任务将不会生效。

此语句从v2.4版本开始支持。

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

## 参数说明

- 收集类型
  - FULL：表示完整收集。
  - SAMPLE：表示采样收集。
  - 如果未指定收集类型，默认使用完整收集。

- `col_name`：要收集统计信息的列。用逗号(`,`)分隔多个列。如果未指定此参数，则将收集整个表。

- `PROPERTIES`：自定义参数。如果未指定`PROPERTIES`，将使用`fe.conf`中的默认设置。实际使用的属性可通过SHOW ANALYZE JOB输出的`Properties`列查看。

| **属性**                               | **类型** | **默认值** | **描述**                                              |
| ------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_auto_collect_ratio          | FLOAT    | 0.8               | 用于确定自动收集统计信息是否健康的阈值。如果统计信息未达到此阈值，将触发自动收集。 |
| statistics_max_full_collect_data_size | INT      | 100               | 自动收集收集数据的最大分区大小。单位：GB。如果分区超过此值，将放弃完整收集，改为采样收集。 |
| statistic_sample_collect_rows         | INT      | 200000            | 收集的最小行数。如果参数值超过表中的实际行数，将进行完整收集。 |

## 示例

示例1：自动完整收集

```SQL
-- 自动收集所有数据库的完整统计信息。
CREATE ANALYZE ALL;

-- 自动收集数据库的完整统计信息。
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

-- 收集表中指定列的统计信息，指定统计信息健康度和收集行数。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES(
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);
```

## 引用

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)：查看自定义收集任务的状态。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md)：删除自定义收集任务。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：取消正在运行的自定义收集任务。

有关CBO收集统计信息的更多信息，请参见[为CBO收集统计信息](../../../using_starrocks/Cost_based_optimizer.md)。