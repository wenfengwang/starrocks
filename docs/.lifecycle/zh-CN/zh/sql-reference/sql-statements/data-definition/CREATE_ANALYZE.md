---
displayed_sidebar: "Chinese"
---

# 创建分析

## 功能

Create custom automatic collection tasks to collect CBO statistics information.

By default, StarRocks will automatically collect the full statistical information of the table periodically. The default check update time is once every 5 minutes. If data updates are found, the collection will be triggered automatically. If you do not want to use automatic full collection, you can set the FE configuration item `enable_collect_full_statistic` to `false`, and the system will stop automatic full collection and customize the collection according to the custom tasks you create.

Before creating a custom automatic collection task, you need to first disable automatic full collection `enable_collect_full_statistic=false`, otherwise the custom collection task will not take effect.

## 语法

```SQL
-- Regularly collect statistical information for all databases.
CREATE ANALYZE [FULL|SAMPLE] ALL PROPERTIES (property [,property])

-- Regularly collect statistical information for all tables in the specified database.
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name PROPERTIES (property [,property])

-- Regularly collect statistical information for the specified table and columns.
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name]) PROPERTIES (property [,property])
```

## 参数说明

- Collection type
  - FULL: Full collection.
  - SAMPLE: Sampling collection.
  - If the collection type is not specified, it defaults to full collection.

- `col_name`: Columns for collecting statistical information, multiple columns are separated by commas (,). If not specified, it represents collecting information for the entire table.

- PROPERTIES: Custom parameters of the collection task. If not configured, the default configuration in `fe.conf` will be used. The PROPERTIES actually used in the task execution can be viewed through the `Properties` column in [SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md).

| **PROPERTIES**                        | **类型** | **默认值** | **说明**                                                     |
| ------------------------------------- | -------- | ---------- | ------------------------------------------------------------ |
| statistic_auto_collect_ratio          | 浮点数   | 0.8        | Automatic collection threshold for statistical information health. If the health of statistical information is less than this threshold, automatic collection will be triggered. |
| statistics_max_full_collect_data_size | 整数     | 100        | Maximum size of automatically collected statistical information for a partition. Unit: GB. If a partition exceeds this value, it will abandon full statistical information collection and switch to sampling statistical information collection for the table. |
| statistic_sample_collect_rows         | 整数     | 200000     | Number of rows sampled in sampling collection. If the value of this parameter exceeds the actual number of table rows, a full collection will be performed. |

## 示例

Example 1: Automatic Full Collection

```SQL
-- Regularly collect full statistical information for all databases.
CREATE ANALYZE ALL;

-- Regularly collect full statistical information for all tables in the specified database.
CREATE ANALYZE DATABASE db_name;

-- Regularly collect full statistical information for all tables in the specified database.
CREATE ANALYZE FULL DATABASE db_name;

-- Regularly collect full statistical information for the specified table and columns.
CREATE ANALYZE TABLE tbl_name(c1, c2, c3);
```

Example 2: Automatic Sampling Collection

```SQL
-- Regularly collect sampling statistical information for all tables in the specified database using default configuration.
CREATE ANALYZE SAMPLE DATABASE db_name;

-- Regularly collect sampling statistical information for the specified table and columns, and set the sampling rows and health threshold.
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES(
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);
```

## 相关文档

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md): View the created custom automatic collection tasks.

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md): Delete the automatic collection task.

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): Cancel the running (Running) statistics collection task.

For more information on CBO statistics collection, refer to [CBO Statistics](../../../using_starrocks/Cost_based_optimizer.md).