---
displayed_sidebar: Chinese
---

# SHOW CREATE MATERIALIZED VIEWの表示

## 機能

指定された[非同期マテリアライズドビュー](../../../using_starrocks/Materialized_view.md)の定義を表示します。

:::tip

この操作には権限は必要ありません。

:::

## 文法

```SQL
SHOW CREATE MATERIALIZED VIEW [db_name.]mv_name
```

## パラメータ

| **パラメータ** | **必須** | **説明**                     |
| -------------- | -------- | ---------------------------- |
| db_name        | いいえ   | データベース名。指定しない場合は、現在のデータベース内の指定されたマテリアライズドビューの定義を表示します。 |
| mv_name        | はい     | 定義を表示するマテリアライズドビューの名前。 |

## 戻り値

| **戻り値**                 | **説明**         |
| -------------------------- | ---------------- |
| Materialized View          | マテリアライズドビューの名前。 |
| Create Materialized View   | マテリアライズドビューの定義。 |

## 例

### 例1：指定されたマテリアライズドビューの定義を表示

```Plain
MySQL > SHOW CREATE MATERIALIZED VIEW lo_mv1\G
*************************** 1. row ***************************
       Materialized View: lo_mv1
Create Materialized View: CREATE MATERIALIZED VIEW `lo_mv1`
COMMENT "MATERIALIZED_VIEW"
DISTRIBUTED BY HASH(`lo_orderkey`) BUCKETS 10 
REFRESH ASYNC
PROPERTIES (
"replication_num" = "3",
"storage_medium" = "HDD"
)
AS SELECT `wlc_test`.`lineorder`.`lo_orderkey` AS `lo_orderkey`, `wlc_test`.`lineorder`.`lo_custkey` AS `lo_custkey`, sum(`wlc_test`.`lineorder`.`lo_quantity`) AS `total_quantity`, sum(`wlc_test`.`lineorder`.`lo_revenue`) AS `total_revenue`, count(`wlc_test`.`lineorder`.`lo_shipmode`) AS `shipmode_count` FROM `wlc_test`.`lineorder` GROUP BY `wlc_test`.`lineorder`.`lo_orderkey`, `wlc_test`.`lineorder`.`lo_custkey` ORDER BY `wlc_test`.`lineorder`.`lo_orderkey` ASC ;
1 row in set (0.01 sec)
```
