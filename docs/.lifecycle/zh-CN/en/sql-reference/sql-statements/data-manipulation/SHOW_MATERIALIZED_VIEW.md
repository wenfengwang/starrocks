---
displayed_sidebar: "英文"
---

# 显示物化视图

## 描述

显示所有或一个特定的异步物化视图。

自v3.0起，此语句的名称从SHOW MATERIALIZED VIEW 更改为SHOW MATERIALIZED VIEWS。

## 语法

```SQL
SHOW MATERIALIZED VIEWS
[FROM db_name]
[
WHERE NAME { = "mv_name" | LIKE "mv_name_matcher"}
]
```

方括号中的参数是可选的。

## 参数

| **参数**       | **必需** | **描述**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| db_name        | 否       | 物化视图所在数据库的名称。如果未指定此参数，默认使用当前数据库。  |
| mv_name        | 否       | 要显示的物化视图的名称。                                         |
| mv_name_matcher | 否      | 用于过滤物化视图的匹配器。                                       |

## 返回值

| **返回值**                  | **描述**                                                     |
| -------------------------- | ------------------------------------------------------------ |
| id                         | 物化视图的ID。                                                |
| database_name              | 物化视图所在数据库的名称。                                       |
| name                       | 物化视图的名称。                                               |
| refresh_type               | 物化视图的刷新类型，包括ROLLUP、MANUAL、ASYNC和INCREMENTAL。   |
| is_active                  | 物化视图状态是否活跃。有效值：`true`和`false`。                    |
| partition_type             | 物化视图的分区类型，包括RANGE和UNPARTITIONED。                     |
| task_id                    | 物化视图刷新任务的ID。                                         |
| task_name                  | 物化视图刷新任务的名称。                                       |
| last_refresh_start_time    | 物化视图最后一次刷新的开始时间。                                |
| last_refresh_finished_time | 物化视图最后一次刷新的结束时间。                                |
| last_refresh_duration      | 最后一次刷新所用时间。单位：秒。                                    |
| last_refresh_state         | 最后一次刷新的状态，包括PENDING、RUNNING、FAILED和SUCCESS。           |
| last_refresh_force_refresh | 最后一次刷新是否是FORCE刷新。                                   |
| last_refresh_start_partition | 物化视图最后一次刷新的开始分区。                                  |
| last_refresh_end_partition | 物化视图最后一次刷新的结束分区。                                  |
| last_refresh_base_refresh_partitions | 最后一次刷新中刷新的基础表分区。                                |
| last_refresh_mv_refresh_partitions | 最后一次刷新中刷新的物化视图分区。                             |
| last_refresh_error_code    | 物化视图最后一次刷新失败的错误代码（如果物化视图状态不是活跃的）。   |
| last_refresh_error_message | 物化视图最后一次刷新失败的原因（如果物化视图状态不是活跃的）。   |
| rows                       | 物化视图中的数据行数。                                          |
| text                       | 用于创建物化视图的语句。                                        |

## 示例

以下示例基于此业务场景：

```Plain
-- 创建表：customer
CREATE TABLE customer ( C_CUSTKEY     INTEGER NOT NULL,
                        C_NAME        VARCHAR(25) NOT NULL,
                        C_ADDRESS     VARCHAR(40) NOT NULL,
                        C_NATIONKEY   INTEGER NOT NULL,
                        C_PHONE       CHAR(15) NOT NULL,
                        C_ACCTBAL     double   NOT NULL,
                        C_MKTSEGMENT  CHAR(10) NOT NULL,
                        C_COMMENT     VARCHAR(117) NOT NULL,
                        PAD char(1) NOT NULL)
    ENGINE=OLAP
DUPLICATE KEY(`c_custkey`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`c_custkey`)
PROPERTIES (
"replication_num" = "1",
"storage_format" = "DEFAULT"
);

-- 创建物化视图：customer_mv
CREATE MATERIALIZED VIEW customer_mv
DISTRIBUTED BY HASH(c_custkey)
REFRESH MANUAL
PROPERTIES (
    "replication_num" = "1"
)
AS SELECT
              c_custkey, c_phone, c_acctbal, count(1) as c_count, sum(c_acctbal) as c_sum
   FROM
              customer
   GROUP BY c_custkey, c_phone, c_acctbal;

-- 刷新物化视图
REFRESH MATERIALIZED VIEW customer_mv;
```

示例 1：显示特定的物化视图。

```Plain
mysql> SHOW MATERIALIZED VIEWS WHERE NAME='customer_mv'\G;
*************************** 1. row ***************************
                        id: 10142
                      name: customer_mv
             database_name: test
              refresh_type: MANUAL
                 is_active: true
   last_refresh_start_time: 2023-02-17 10:27:33
last_refresh_finished_time: 2023-02-17 10:27:33
     last_refresh_duration: 0
        last_refresh_state: SUCCESS
             inactive_code: 0
           inactive_reason:
                      text: CREATE MATERIALIZED VIEW `customer_mv`
COMMENT "MATERIALIZED_VIEW"
DISTRIBUTED BY HASH(`c_custkey`)
REFRESH MANUAL
PROPERTIES (
"replication_num" = "1",
"storage_medium" = "HDD"
)
AS SELECT `customer`.`c_custkey`, `customer`.`c_phone`, `customer`.`c_acctbal`, count(1) AS `c_count`, sum(`customer`.`c_acctbal`) AS `c_sum`
FROM `test`.`customer`
GROUP BY `customer`.`c_custkey`, `customer`.`c_phone`, `customer`.`c_acctbal`;
                      rows: 0
1 row in set (0.11 sec)
```

示例 2：通过匹配名称显示物化视图。

```Plain
mysql> SHOW MATERIALIZED VIEWS WHERE NAME LIKE 'customer_mv'\G;
*************************** 1. row ***************************
                        id: 10142
                      name: customer_mv
             database_name: test
              refresh_type: MANUAL
                 is_active: true
   last_refresh_start_time: 2023-02-17 10:27:33
last_refresh_finished_time: 2023-02-17 10:27:33
     last_refresh_duration: 0
        last_refresh_state: SUCCESS
             inactive_code: 0
           inactive_reason:
                      text: CREATE MATERIALIZED VIEW `customer_mv`
COMMENT "MATERIALIZED_VIEW"
DISTRIBUTED BY HASH(`c_custkey`)
REFRESH MANUAL
PROPERTIES (
"replication_num" = "1",
"storage_medium" = "HDD"
)
AS SELECT `customer`.`c_custkey`, `customer`.`c_phone`, `customer`.`c_acctbal`, count(1) AS `c_count`, sum(`customer`.`c_acctbal`) AS `c_sum`
FROM `test`.`customer`
GROUP BY `customer`.`c_custkey`, `customer`.`c_phone`, `customer`.`c_acctbal`;
                      rows: 0
1 row in set (0.12 sec)
```