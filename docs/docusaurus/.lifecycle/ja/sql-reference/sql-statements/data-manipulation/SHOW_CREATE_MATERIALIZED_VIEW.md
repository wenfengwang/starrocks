---
displayed_sidebar: "Japanese"
---

# マテリアライズド・ビューの作成を表示

## 説明

特定の非同期マテリアライズド・ビューの定義を表示します。

## 構文

```SQL
SHOW CREATE MATERIALIZED VIEW [database.]<mv_name>
```

角かっこ[]内のパラメータは任意です。

## パラメータ

| **パラメータ** | **必須** | **説明**                                      |
| ------------- | ------------ | ------------------------------------------ |
| mv_name       | はい          | 表示するマテリアライズド・ビューの名前。 |

## 戻り値

| **戻り値**             | **説明**                                |
| ------------------------ | ---------------------------------------- |
| Materialized View        | マテリアライズド・ビューの名前。          |
| Create Materialized View | マテリアライズド・ビューの定義。         |

## 例

例 1: 特定のマテリアライズド・ビューの定義を表示する

```Plain
MySQL > SHOW CREATE MATERIALIZED VIEW lo_mv1\G
*************************** 1. row ***************************
       Materialized View: lo_mv1
Create Materialized View: CREATE MATERIALIZED VIEW `lo_mv1`
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