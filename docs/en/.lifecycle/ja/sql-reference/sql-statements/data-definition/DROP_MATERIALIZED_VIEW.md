---
displayed_sidebar: "Japanese"
---

# DROP MATERIALIZED VIEW

## 説明

マテリアライズドビューを削除します。

このコマンドで作成中の同期マテリアライズドビューを削除することはできません。作成中の同期マテリアライズドビューを削除するには、[同期マテリアライズドビュー - 未完了のマテリアライズドビューを削除する](../../../using_starrocks/Materialized_view.md#drop-an-unfinished-materialized-view)を参照してください。

> **注意**
>
> ベーステーブルが存在するデータベースで `DROP_PRIV` 権限を持つユーザーのみがマテリアライズドビューを削除できます。

## 構文

```SQL
DROP MATERIALIZED VIEW [IF EXISTS] [database.]mv_name
```

角括弧 [] 内のパラメータはオプションです。

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| ------------- | ---------- | ------------------------------------------------------------ |
| IF EXISTS     | いいえ       | このパラメータが指定されている場合、存在しないマテリアライズドビューを削除する際に StarRocks は例外をスローしません。このパラメータが指定されていない場合、存在しないマテリアライズドビューを削除する際にシステムは例外をスローします。 |
| mv_name       | はい         | 削除するマテリアライズドビューの名前。                        |

## 例

例1: 既存のマテリアライズドビューを削除する

1. データベース内のすべての既存のマテリアライズドビューを表示します。

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

2. マテリアライズドビュー `order_mv1` を削除します。

  ```SQL
  DROP MATERIALIZED VIEW order_mv1;
  ```

3. 削除されたマテリアライズドビューが存在するかどうかを確認します。

  ```Plain
  MySQL > SHOW MATERIALIZED VIEWS;
  Empty set (0.01 sec)
  ```

例2: 存在しないマテリアライズドビューを削除する

- パラメータ `IF EXISTS` が指定されている場合、存在しないマテリアライズドビューを削除する際に StarRocks は例外をスローしません。

```Plain
MySQL > DROP MATERIALIZED VIEW IF EXISTS k1_k2;
Query OK, 0 rows affected (0.00 sec)
```

- パラメータ `IF EXISTS` が指定されていない場合、存在しないマテリアライズドビューを削除する際にシステムは例外をスローします。

```Plain
MySQL > DROP MATERIALIZED VIEW k1_k2;
ERROR 1064 (HY000): Materialized view k1_k2 is not find
```
