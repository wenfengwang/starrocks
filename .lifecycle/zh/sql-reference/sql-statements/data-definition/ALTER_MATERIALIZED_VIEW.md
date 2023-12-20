---
displayed_sidebar: English
---

# 修改物化视图

## 描述

此SQL语句能够：

- 更改异步物化视图的名称。
- 更改异步物化视图的刷新策略。
- 将异步物化视图的状态设置为活跃或非活跃。
- 在两个异步物化视图之间进行原子级别的交换。
- 修改异步物化视图的属性。
  您可以使用此SQL语句修改以下属性：

  - 分区TTL（生存时间）数值
  - 分区刷新次数
  - 资源组
  - 自动刷新分区的限制
  - 排除的触发器表
  - 物化视图重写的陈旧秒数
  - 唯一性约束
  - 外键约束
  - 数据分布策略
  - 所有与会话变量相关的属性。关于会话变量的信息，请参见[System variables](../../../reference/System_variable.md)文档。

:::提示

执行此操作需要对目标物化视图拥有**ALTER**权限。您可以遵循[GRANT](../account-management/GRANT.md)语句的指引来授予此权限。

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

方括号[]内的参数是可选的。

## 参数

|参数|必填|说明|
|---|---|---|
|mv_name|yes|要更改的物化视图的名称。|
|new_refresh_scheme_desc|no|新的刷新策略，详细信息请参见 SQL 参考 - 创建物化视图 - 参数。|
|new_mv_name|no|物化视图的新名称。|
|ACTIVE|no|将物化视图的状态设置为活动。如果物化视图的任何基表发生更改（例如删除并重新创建），StarRocks 会自动将物化视图设置为非活动状态，以防止原始元数据与更改后的基表不匹配的情况。非活动物化视图不能用于查询加速或查询重写。您可以在更改基表后使用此 SQL 激活物化视图。|
|INACTIVE|no|将物化视图的状态设置为非活动。无法刷新非活动的异步物化视图。但您仍然可以将其作为表进行查询。|
|SWAP WITH|no|在必要的一致性检查后与另一个异步物化视图执行原子交换。|
|key|no|要更改的属性名称，请参阅 SQL 参考 - 创建物化视图 - 参数了解详细信息。注意如果要更改物化视图的会话变量相关属性，则必须添加会话。属性的前缀，例如 session.query_timeout。您不需要指定非会话属性的前缀，例如 mv_rewrite_staleness_second。|
|value|no|要更改的属性的值。|

## 示例

示例1：修改物化视图的名称。

```SQL
ALTER MATERIALIZED VIEW lo_mv1 RENAME lo_mv1_new_name;
```

示例2：修改物化视图的刷新间隔。

```SQL
ALTER MATERIALIZED VIEW lo_mv2 REFRESH ASYNC EVERY(INTERVAL 1 DAY);
```

示例3：修改物化视图的属性。

```SQL
-- Change mv1's query_timeout to 40000 seconds.
ALTER MATERIALIZED VIEW mv1 SET ("session.query_timeout" = "40000");
-- Change mv1's mv_rewrite_staleness_second to 600 seconds.
ALTER MATERIALIZED VIEW mv1 SET ("mv_rewrite_staleness_second" = "600");
```

示例4：将物化视图的状态设置为活跃。

```SQL
ALTER MATERIALIZED VIEW order_mv ACTIVE;
```

示例5：在物化视图order_mv和order_mv1之间进行原子交换。

```SQL
ALTER MATERIALIZED VIEW order_mv SWAP WITH order_mv1;
```
