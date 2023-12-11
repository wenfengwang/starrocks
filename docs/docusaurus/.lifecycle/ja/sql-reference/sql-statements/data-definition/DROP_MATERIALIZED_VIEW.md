---
displayed_sidebar: "Japanese"
---

# MATERIALIZED VIEWの削除

## 説明

マテリアライズド・ビューを削除します。

このコマンドと同じプロセスで作成中の同期マテリアライズド・ビューを削除することはできません。この場合は、作成中の同期マテリアライズド・ビューを削除するには、[同期マテリアライズド・ビュー - 未完了のマテリアライズド・ビューの削除](../../../using_starrocks/Materialized_view.md#drop-an-unfinished-materialized-view)を参照してください。

> **注意**
>
> 基本テーブルが存在するデータベースで`DROP_PRIV`権限を持つユーザーのみがマテリアライズド・ビューを削除できます。

## 構文

```SQL
DROP MATERIALIZED VIEW [IF EXISTS] [database.]mv_name
```

角括弧[]内のパラメータはオプションです。

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| ------------- | ------------ | ------------------------------------------------------------ |
| IF EXISTS     | いいえ           | このパラメータが指定されている場合、StarRocksは存在しないマテリアライズド・ビューを削除する際に例外をスローしません。指定されていない場合、システムは存在しないマテリアライズド・ビューを削除しようとした際に例外をスローします。 |
| mv_name       | はい          | 削除するマテリアライズド・ビューの名前。                 |

## 例

例1: 既存のマテリアライズド・ビューの削除

1. データベース内のすべての既存のマテリアライズド・ビューを表示します。

  ```Plain
  MySQL > SHOW MATERIALIZED VIEWS\G
  *************************** 1. row ***************************
              id: 470740
          name: order_mv1
  database_name: default_cluster:sr_hub
        text: SELECT `sr_hub`.`orders`.`dt` AS `dt`, `sr_hub`.`orders`.`order_id` AS `order_id`, `sr_hub`.`orders`.`user_id` AS `user_id`, sum(`sr_hub`.`orders`.`cnt`) AS `total_cnt`, sum(`sr_hub`.`orders`.`revenue`) AS `total_revenue`, count(`sr_hub`.`orders`.`state`) AS `state_count` FROM `sr_hub`.`orders` GROUP BY `sr_hub`.`orders`.`dt`, `sr_hub`.`orders`.`order_id`, `sr_hub`.`orders`.`user_id`
          rows: 0
  1 rows in set (0.00 sec)
  ```

2. マテリアライズド・ビュー `order_mv1` を削除します。

  ```SQL
  DROP MATERIALIZED VIEW order_mv1;
  ```

3. 削除されたマテリアライズド・ビューが存在するか確認します。

  ```Plain
  MySQL > SHOW MATERIALIZED VIEWS;
  Empty set (0.01 sec)
  ```

例2: 存在しないマテリアライズド・ビューの削除

- パラメータ `IF EXISTS` が指定されている場合、存在しないマテリアライズド・ビューを削除してもStarRocksは例外をスローしません。

```Plain
MySQL > DROP MATERIALIZED VIEW IF EXISTS k1_k2;
Query OK, 0 rows affected (0.00 sec)
```

- パラメータ `IF EXISTS` が指定されていない場合、存在しないマテリアライズド・ビューを削除しようとした際にシステムは例外をスローします。

```Plain
MySQL > DROP MATERIALIZED VIEW k1_k2;
ERROR 1064 (HY000): Materialized view k1_k2 is not find
```