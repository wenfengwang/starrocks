---
displayed_sidebar: "Chinese"
---

# 显示创建物化视图

## 描述

显示特定异步物化视图的定义。

## 语法

```SQL
SHOW CREATE MATERIALIZED VIEW [数据库.]<mv_name>
```

方括号[]中的参数是可选的。

## 参数

| **参数**    | **必需** | **描述**                         |
| ----------- | -------- | -------------------------------- |
| mv_name     | 是       | 要显示的物化视图的名称。         |

## 返回

| **返回**                | **描述**                      |
| ------------------------ | ----------------------------- |
| 物化视图                | 物化视图的名称。              |
| 创建物化视图            | 物化视图的定义。              |

## 示例

示例1：显示特定物化视图的定义

```Plain
MySQL > SHOW CREATE MATERIALIZED VIEW lo_mv1\G
*************************** 1. row ***************************
       物化视图: lo_mv1
创建物化视图: CREATE MATERIALIZED VIEW `lo_mv1`
COMMENT "MATERIALIZED_VIEW"
DISTRIBUTED BY HASH(`lo_orderkey`) 
REFRESH ASYNC
PROPERTIES (
"replication_num" = "3",
"storage_medium" = "HDD"
)
AS SELECT `wlc_test`.`lineorder`.`lo_orderkey` AS `lo_orderkey`, `wlc_test`.`lineorder`.`lo_custkey` AS `lo_custkey`, sum(`wlc_test`.`lineorder`.`lo_quantity`) AS `total_quantity`, sum(`wlc_test`.`lineorder`.`lo_revenue`) AS `total_revenue`, count(`wlc_test`.`lineorder`.`lo_shipmode`) AS `shipmode_count` FROM `wlc_test`.`lineorder` GROUP BY `wlc_test`.`lineorder`.`lo_orderkey`, `wlc_test`.`lineorder`.`lo_custkey` ORDER BY `wlc_test`.`lineorder`.`lo_orderkey` ASC ;
1 row in set (0.01 sec)
```