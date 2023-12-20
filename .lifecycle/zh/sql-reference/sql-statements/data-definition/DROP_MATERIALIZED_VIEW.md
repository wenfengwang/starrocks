---
displayed_sidebar: English
---

# 删除物化视图

## 说明

此操作用于删除物化视图。

当前不支持使用此命令删除正在创建中的同步物化视图。若需删除正在创建中的同步物化视图，请参见[Synchronous materialized View - Drop an unfinished materialized view](../../../using_starrocks/Materialized_view-single_table.md#drop-an-unfinished-synchronous-materialized-view)获取详细指导。

:::提示

执行此操作需要对目标物化视图具有DROP权限。

:::

## 语法

```SQL
DROP MATERIALIZED VIEW [IF EXISTS] [database.]mv_name
```

方括号[]内的参数为可选项。

## 参数

|参数|必填|说明|
|---|---|---|
|IF EXISTS|no|如果指定该参数，StarRocks 在删除不存在的物化视图时不会抛出异常。如果不指定该参数，系统在删除不存在的物化视图时会抛出异常。|
|mv_name|yes|要删除的物化视图的名称。|

## 示例

示例 1：删除已存在的物化视图

1. 查看数据库中所有已存在的物化视图。

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

2. 删除名为order_mv1的物化视图。

```SQL
DROP MATERIALIZED VIEW order_mv1;
```

3. 核实已删除的物化视图是否仍存在。

```Plain
MySQL > SHOW MATERIALIZED VIEWS;
Empty set (0.01 sec)
```

示例 2：删除一个不存在的物化视图

- 若指定了IF EXISTS参数，当删除一个不存在的物化视图时，StarRocks不会报错。

```Plain
MySQL > DROP MATERIALIZED VIEW IF EXISTS k1_k2;
Query OK, 0 rows affected (0.00 sec)
```

- 若未指定IF EXISTS参数，系统在删除一个不存在的物化视图时将报错。

```Plain
MySQL > DROP MATERIALIZED VIEW k1_k2;
ERROR 1064 (HY000): Materialized view k1_k2 is not find
```
