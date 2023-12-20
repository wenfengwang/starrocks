---
displayed_sidebar: English
---

# 删除物化视图

## 描述

删除一个物化视图。

您不能使用此命令删除正在创建过程中的同步物化视图。要删除一个正在创建过程中的同步物化视图，请参阅[Synchronous materialized View - Drop an unfinished materialized view](../../../using_starrocks/Materialized_view-single_table.md#drop-an-unfinished-synchronous-materialized-view)获取更多指导。

:::tip

此操作需要对目标物化视图有 DROP 权限。

:::

## 语法

```SQL
DROP MATERIALIZED VIEW [IF EXISTS] [database.]mv_name
```

方括号[]内的参数是可选的。

## 参数

|**参数**|**是否必须**|**描述**|
|---|---|---|
|IF EXISTS|否|如果指定了此参数，当删除一个不存在的物化视图时，StarRocks 不会抛出异常。如果未指定此参数，当删除一个不存在的物化视图时，系统将会抛出异常。|
|mv_name|是|要删除的物化视图的名称。|

## 示例

示例 1：删除一个现有的物化视图

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

2. 删除物化视图 `order_mv1`。

```SQL
DROP MATERIALIZED VIEW order_mv1;
```

3. 检查已删除的物化视图是否存在。

```Plain
MySQL > SHOW MATERIALIZED VIEWS;
Empty set (0.01 sec)
```

示例 2：删除一个不存在的物化视图

- 如果指定了 `IF EXISTS` 参数，当删除一个不存在的物化视图时，StarRocks 不会抛出异常。

```Plain
MySQL > DROP MATERIALIZED VIEW IF EXISTS k1_k2;
Query OK, 0 rows affected (0.00 sec)
```

- 如果未指定 `IF EXISTS` 参数，当删除一个不存在的物化视图时，系统将会抛出异常。

```Plain
MySQL > DROP MATERIALIZED VIEW k1_k2;
ERROR 1064 (HY000): Materialized view k1_k2 is not find
```