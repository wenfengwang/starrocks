---
displayed_sidebar: Chinese
---

# DROP MATERIALIZED VIEW

## 機能

マテリアライズドビューを削除します。

このコマンドは、作成中の同期マテリアライズドビューを削除するためには使用できません。作成中の同期マテリアライズドビューを削除するには、[同期マテリアライズドビューの削除](../../../using_starrocks/Materialized_view-single_table.md#同期マテリアライズドビューの削除)を参照してください。

> **注意**
>
> DROP権限を持つユーザーのみがマテリアライズドビューを削除できます。

## 文法

```SQL
DROP MATERIALIZED VIEW [IF EXISTS] [database.]mv_name
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| IF EXISTS      | いいえ   | このパラメータを指定すると、存在しないマテリアライズドビューを削除してもエラーは発生しません。指定しない場合、存在しないマテリアライズドビューを削除しようとするとエラーが発生します。 |
| mv_name        | はい     | 削除するマテリアライズドビューの名前。                       |

## 例

例1：存在するマテリアライズドビューを削除

1. 現在のデータベースに存在するマテリアライズドビューを確認します。

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

    ```Plain
    DROP MATERIALIZED VIEW order_mv1;
    ```

3. 削除後、再度現在のデータベースに存在するマテリアライズドビューを確認すると、該当するビューは表示されません。

    ```Plain
    MySQL > SHOW MATERIALIZED VIEWS;
    Empty set (0.01 sec)
    ```

例2：存在しないマテリアライズドビューを削除

- `IF EXISTS` パラメータを指定しない場合、現在のデータベースに属していないマテリアライズドビュー `k1_k2` を削除しようとするとエラーが発生します。

```Plain
MySQL > DROP MATERIALIZED VIEW k1_k2;
ERROR 1064 (HY000): Materialized view k1_k2 is not found
```

- `IF EXISTS` パラメータを指定した場合、現在のデータベースに属していないマテリアライズドビュー `k1_k2` を削除してもエラーは発生しません。

```Plain
MySQL > DROP MATERIALIZED VIEW IF EXISTS k1_k2;
Query OK, 0 rows affected (0.00 sec)
```
