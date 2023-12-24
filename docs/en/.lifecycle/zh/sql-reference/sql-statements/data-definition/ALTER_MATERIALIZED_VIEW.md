---
displayed_sidebar: English
---

# 修改物化视图

## 描述

此 SQL 语句可以：

- 修改异步物化视图的名称。
- 修改异步物化视图的刷新策略。
- 将异步物化视图的状态更改为活动或非活动。
- 在两个异步物化视图之间执行原子交换。
- 修改异步物化视图的属性。

  您可以使用此 SQL 语句来修改以下属性：

  - `partition_ttl_number`
  - `partition_refresh_number`
  - `resource_group`
  - `auto_refresh_partitions_limit`
  - `excluded_trigger_tables`
  - `mv_rewrite_staleness_second`
  - `unique_constraints`
  - `foreign_key_constraints`
  - `colocate_with`
  - 所有与会话变量相关的属性。有关会话变量的信息，请参阅 [系统变量](../../../reference/System_variable.md)。

:::提示

此操作需要对目标物化视图具有 ALTER 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。

:::

## 语法

```SQL
ALTER MATERIALIZED VIEW [db_name.]<mv_name> 
    { RENAME [db_name.]<new_mv_name> 
    | REFRESH <new_refresh_scheme_desc> 
    | ACTIVE | INACTIVE 
    | SWAP WITH [db_name.]<mv2_name>
    | SET ( "<key>" = "<value>"[,...]) }
```

方括号中的参数是可选的。

## 参数

| **参数**           | **必填** | **描述**                                              |
| ----------------------- | ------------ | ------------------------------------------------------------ |
| mv_name                 | 是          | 要修改的物化视图的名称。                  |
| new_refresh_scheme_desc | 否           | 新的刷新策略，请参阅 [SQL 参考 - 创建物化视图 - 参数](../data-definition/CREATE_MATERIALIZED_VIEW.md#parameters) 了解详情。 |
| new_mv_name             | 否           | 物化视图的新名称。                          |
| ACTIVE                  | 否           |将物化视图的状态设置为活动。如果任何基表发生更改，例如删除并重新创建，StarRocks 会自动将物化视图设置为非活动状态，以防止原始元数据与更改后的基表不匹配。非活动物化视图不能用于查询加速或查询重写。您可以使用此 SQL 在更改基表后激活物化视图。 |
| INACTIVE                | 否           | 将物化视图的状态设置为非活动。非活动的异步物化视图无法刷新，但仍可作为表进行查询。 |
| SWAP WITH               | 否           | 在必要的一致性检查后，与另一个异步物化视图执行原子交换。 |
| key                     | 否           | 要修改的属性名称，请参阅 [SQL 参考 - 创建物化视图 - 参数](../data-definition/CREATE_MATERIALIZED_VIEW.md#parameters) 了解详情。<br />**注意**<br />如果要修改物化视图的会话变量相关属性，必须在属性前加上 `session.` 前缀，例如 `session.query_timeout`。对于非会话属性，例如 `mv_rewrite_staleness_second`，则无需指定前缀。 |
| value                   | 否           | 要修改的属性值。                         |

## 示例

示例 1：修改物化视图的名称。

```SQL
ALTER MATERIALIZED VIEW lo_mv1 RENAME lo_mv1_new_name;
```

示例 2：修改物化视图的刷新间隔。

```SQL
ALTER MATERIALIZED VIEW lo_mv2 REFRESH ASYNC EVERY(INTERVAL 1 DAY);
```

示例 3：修改物化视图的属性。

```SQL
-- 将 mv1 的 query_timeout 修改为 40000 秒。
ALTER MATERIALIZED VIEW mv1 SET ("session.query_timeout" = "40000");
-- 将 mv1 的 mv_rewrite_staleness_second 修改为 600 秒。
ALTER MATERIALIZED VIEW mv1 SET ("mv_rewrite_staleness_second" = "600");
```

示例 4：将物化视图的状态修改为活动。

```SQL
ALTER MATERIALIZED VIEW order_mv ACTIVE;
```

示例 5：在物化视图 `order_mv` 和 `order_mv1` 之间执行原子交换。

```SQL
ALTER MATERIALIZED VIEW order_mv SWAP WITH order_mv1;
```
