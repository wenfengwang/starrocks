---
displayed_sidebar: "Chinese"
---

# 删除物化视图

## 描述

删除一个物化视图。

不能删除正在创建过程中的同步物化视图。要删除正在创建中的同步物化视图，请参见[Synchronous materialized View - Drop an unfinished materialized view](../../../using_starrocks/Materialized_view.md#drop-an-unfinished-materialized-view)获取进一步的说明。

> **警告**
>
> 只有在基表所在的数据库中具有`DROP_PRIV`特权的用户才能删除物化视图。

## 语法

```SQL
DROP MATERIALIZED VIEW [IF EXISTS] [database.]mv_name
```

括号内的参数是可选的。

## 参数

| **参数**   | **必须** | **描述**                                                     |
| ---------- | -------- | ------------------------------------------------------------ |
| IF EXISTS  | 否       | 如果指定了这个参数，StarRocks 在删除不存在的物化视图时不会抛出异常。如果没有指定这个参数，系统在删除不存在的物化视图时会抛出异常。 |
| mv_name    | 是       | 要删除的物化视图的名称。                                      |

## 示例

示例 1：删除现有的物化视图

1. 查看数据库中所有现有的物化视图。

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

2. 删除物化视图`order_mv1`。

  ```SQL
  DROP MATERIALIZED VIEW order_mv1;
  ```

3. 检查删除的物化视图是否存在。

  ```Plain
  MySQL > SHOW MATERIALIZED VIEWS;
  Empty set (0.01 sec)
  ```

示例 2：删除不存在的物化视图

- 如果指定了`IF EXISTS`参数，StarRocks 在删除不存在的物化视图时不会抛出异常。

```Plain
MySQL > DROP MATERIALIZED VIEW IF EXISTS k1_k2;
Query OK, 0 rows affected (0.00 sec)
```

- 如果没有指定`IF EXISTS`参数，系统在删除不存在的物化视图时会抛出异常。

```Plain
MySQL > DROP MATERIALIZED VIEW k1_k2;
ERROR 1064 (HY000): Materialized view k1_k2 is not find
```