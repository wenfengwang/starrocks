---
displayed_sidebar: English
---

# 分析表

## 描述

创建手动收集任务以收集成本基础优化器（CBO）的统计信息。默认情况下，手动收集是同步操作。您也可以设置为异步操作。在异步模式下，执行ANALYZE TABLE后，系统会立即返回该语句是否执行成功。但是，收集任务会在后台运行，无需等待结果。您可以通过执行SHOW ANALYZE STATUS命令来检查任务的状态。对于数据量大的表，适合使用异步收集；而对于数据量小的表，则适合使用同步收集。

**手动收集任务创建后仅执行一次，无需删除。**

该语句从v2.4版本开始支持。

### 手动收集基础统计信息

更多关于基础统计信息的内容，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)。

#### 语法

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
PROPERTIES (property [,property])
```

#### 参数描述

- 收集类型
  - FULL：表示全量收集。
  - SAMPLE：表示采样收集。
  - 如果未指定收集类型，默认为全量收集。

- col_name：要收集统计信息的列名。用逗号（,）分隔多个列名。如果未指定此参数，默认收集整个表。

- [WITH SYNC | ASYNC MODE]：指定手动收集任务是以同步模式还是异步模式运行。如果未指定此参数，默认为同步模式。

- PROPERTIES：自定义参数。如果未指定PROPERTIES，则使用fe.conf文件中的默认设置。可以通过SHOW ANALYZE STATUS输出的Properties列来查看实际使用的属性。

|属性|类型|默认值|描述|
|---|---|---|---|
|statistic_sample_collect_rows|INT|200000|采样收集要收集的最小行数。如果参数值超过表中的实际行数，则执行完整收集。|

#### 示例

示例1：手动全量收集

```SQL
-- Manually collect full stats of a table using default settings.
ANALYZE TABLE tbl_name;

-- Manually collect full stats of a table using default settings.
ANALYZE FULL TABLE tbl_name;

-- Manually collect stats of specified columns in a table using default settings.
ANALYZE TABLE tbl_name(c1, c2, c3);
```

示例2：手动采样收集

```SQL
-- Manually collect partial stats of a table using default settings.
ANALYZE SAMPLE TABLE tbl_name;

-- Manually collect stats of specified columns in a table, with the number of rows to collect specified.
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

### 手动收集直方图

更多关于直方图的内容，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md#histogram)。

#### 语法

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
PROPERTIES (property [,property]);
```

#### 参数描述

- col_name：要收集统计信息的列名。用逗号（,）分隔多个列名。如果未指定此参数，默认收集整个表。收集直方图时，此参数是必需的。

- [WITH SYNC | ASYNC MODE]：指定手动收集任务是以同步模式还是异步模式运行。如果未指定此参数，默认为同步模式。

- WITH N BUCKETS：N是直方图收集的桶数。如果未指定，将使用fe.conf文件中的默认值。

- PROPERTIES：自定义参数。如果未指定PROPERTIES，则使用fe.conf文件中的默认设置。可以通过SHOW ANALYZE STATUS输出的Properties列来查看实际使用的属性。

|属性|类型|默认值|描述|
|---|---|---|---|
|statistic_sample_collect_rows|INT|200000|要收集的最小行数。如果参数值超过表中的实际行数，则执行完整收集。|
|histogram_buckets_size|LONG|64|直方图的默认存储桶编号。|
|histogram_mcv_size|INT|100|直方图最常见值 (MCV) 的数量。|
|histogram_sample_ratio|FLOAT|0.1|直方图的采样率。|
|histogram_max_sample_row_count|LONG|10000000|为直方图收集的最大行数。|

收集直方图的行数由多个参数决定，取statistic_sample_collect_rows和表行数乘以histogram_sample_ratio之间的较大值。但该数值不能超过histogram_max_sample_row_count所指定的值。如果超过，将以histogram_max_sample_row_count为准。

#### 示例

```SQL
-- Manually collect histograms on v1 using the default settings.
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- Manually collect histograms on v1 and v2, with 32 buckets, 32 MCVs, and 50% sampling ratio.
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

## 参考资料

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md)：查看手动收集任务的状态。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：取消正在运行的手动收集任务。

更多关于为CBO收集统计信息的内容，请参见[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)。
