---
displayed_sidebar: English
---

# 分析表

## 描述

创建一个用于收集 CBO 统计信息的手动收集任务。默认情况下，手动收集是同步操作。您也可以将其设置为异步操作。在异步模式下，执行 ANALYZE TABLE 后，系统会立即返回该语句是否成功。但是，收集任务将在后台运行，您不必等待结果。您可以通过运行 SHOW ANALYZE STATUS 来检查任务的状态。异步收集适用于数据量较大的表，而同步收集适用于数据量较小的表。

**手动收集任务在创建后仅运行一次。您无需删除手动收集任务。**

此语句从 v2.4 版本开始支持。

### 手动收集基本统计信息

有关基本统计信息的更多信息，请参阅 [为 CBO 收集统计信息](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)。

#### 语法

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
PROPERTIES (property [,property])
```

#### 参数说明

- 收集类型
  - FULL：表示全量收集。
  - SAMPLE：表示抽样收集。
  - 如果未指定收集类型，则默认使用全量收集。

- `col_name`：要收集统计信息的列。用逗号（,）分隔多个列。如果未指定此参数，则收集整个表。

- [WITH SYNC | ASYNC MODE]：指定手动收集任务以同步模式还是异步模式运行。如果未指定此参数，则默认使用同步收集。

- `PROPERTIES`：自定义参数。如果未指定 `PROPERTIES`，则使用 `fe.conf` 文件中的默认设置。实际使用的属性可以通过 SHOW ANALYZE STATUS 输出中的 `Properties` 列查看。

| **属性**                | **类型** | **默认值** | **描述**                                              |
| ----------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows | INT      | 200000            | 采样收集的最小行数。如果参数值超过表中的实际行数，则执行全量收集。 |

#### 例子

示例 1：手动进行全量收集

```SQL
-- 使用默认设置手动收集表的完整统计信息。
ANALYZE TABLE tbl_name;

-- 使用默认设置手动收集表的完整统计信息。
ANALYZE FULL TABLE tbl_name;

-- 使用默认设置手动收集表中指定列的统计信息。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

示例 2：手动进行抽样收集

```SQL
-- 使用默认设置手动收集表的部分统计信息。
ANALYZE SAMPLE TABLE tbl_name;

-- 使用默认设置手动收集表中指定列的统计信息，并指定要收集的行数。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

### 手动收集直方图

有关直方图的更多信息，请参阅 [为 CBO 收集统计信息](../../../using_starrocks/Cost_based_optimizer.md#histogram)。

#### 语法

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
PROPERTIES (property [,property]);
```

#### 参数说明

- `col_name`：要收集统计信息的列。用逗号（,）分隔多个列。如果未指定此参数，则收集整个表。此参数对于直方图是必需的。

- [WITH SYNC | ASYNC MODE]：指定手动收集任务以同步模式还是异步模式运行。如果未指定此参数，则默认使用同步收集。

- `WITH N BUCKETS`：`N` 是用于直方图收集的桶数。如果未指定，将使用 `fe.conf` 中的默认值。

- PROPERTIES：自定义参数。如果未指定 `PROPERTIES`，则使用 `fe.conf` 中的默认设置。实际使用的属性可以通过 SHOW ANALYZE STATUS 输出中的 `Properties` 列查看。

| **属性**                 | **类型** | **默认值** | **描述**                                              |
| ------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows  | INT      | 200000            | 要收集的最小行数。如果参数值超过表中的实际行数，则执行全量收集。 |
| histogram_buckets_size         | LONG     | 64                | 直方图的默认桶数。                   |
| histogram_mcv_size             | INT      | 100               | 直方图的最常见值（MCV）数量。      |
| histogram_sample_ratio         | FLOAT    | 0.1               | 直方图的采样比率。                          |
| histogram_max_sample_row_count | LONG     | 10000000          | 直方图要收集的最大行数。       |

直方图的收集行数由多个参数控制。它取 `statistic_sample_collect_rows` 和表行数 * `histogram_sample_ratio` 之间的较大值。该数字不能超过 `histogram_max_sample_row_count` 指定的值。如果超过该值，将优先使用 `histogram_max_sample_row_count`。

#### 例子

```SQL
-- 使用默认设置手动收集 v1 的直方图。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- 使用 32 个桶、32 个 MCV 和 50% 的采样比率手动收集 v1 和 v2 的直方图。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

## 引用

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md)：查看手动收集任务的状态。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：取消正在运行的手动收集任务。

有关为 CBO 收集统计信息的更多信息，请参阅 [为 CBO 收集统计信息](../../../using_starrocks/Cost_based_optimizer.md)。
