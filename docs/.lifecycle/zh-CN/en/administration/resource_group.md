---
displayed_sidebar: "Chinese"
---

# 资源组

本主题介绍了 StarRocks 的资源组功能。

![资源组](../assets/resource_group.png)

有了这个功能，您可以在单个集群中同时运行多个工作负载，包括短查询、即席查询、ETL 作业，从而节省部署多个集群的额外成本。从技术角度来看，执行引擎会根据用户的规定安排并发工作负载，并在它们之间隔离干扰。

资源组路线图：

- 从 v2.2 开始，StarRocks 支持限制查询的资源消耗，并在相同集群中的租户之间实现资源隔离和高效利用。
- 在 StarRocks v2.3 中，您可以进一步限制大查询的资源消耗，防止超大查询请求耗尽集群资源，以保证系统稳定性。
- StarRocks v2.5 支持限制数据加载 (INSERT) 的计算资源消耗。

|  | 内部表 | 外部表 | 大查询限制 | 短查询 | INSERT INTO，Broker Load  | 常规加载，流加载，架构更改 |
|---|---|---|---|---|---|---|
| 2.2 | √ | × | × | × | × | × |
| 2.3 | √ | √ | √ | √ | × | × |
| 2.4 | √ | √ | √ | √ | × | × |
| 2.5 及更高版本 | √ | √ | √ | √ | √ | × |

## 术语

本节描述了在使用资源组功能之前必须了解的术语。

### 资源组

每个资源组都是从特定 BE 获取的一组计算资源。您可以将集群的每个 BE 划分为多个资源组。当查询分配给资源组时，StarRocks 根据为资源组指定的资源配额，为资源组分配 CPU 和内存资源。

您可以使用以下参数为资源组在 BE 上指定 CPU 和内存资源配额：

- `cpu_core_limit`

  此参数指定可以分配给资源组的 BE 上 CPU 核心数的软限制。有效值：任何非零正整数。范围：(1, `avg_be_cpu_cores`]，其中 `avg_be_cpu_cores` 表示所有 BE 上 CPU 核心数的平均数。

  在实际业务场景中，分配给资源组的 CPU 核心数会根据 BE 上 CPU 核心的可用性按比例进行扩展。

  > **注意**
  >
  > 例如，您在提供 16 个 CPU 核心的 BE 上配置了三个资源组：rg1、rg2 和 rg3。三个资源组的 `cpu_core_limit` 值分别为 `2`、`6` 和 `8`。
  >
  > 如果 BE 的所有 CPU 核心都被占用，那么根据以下计算，可以分配给三个资源组的 CPU 核心数分别为 2、6 和 8：
  >
  > - rg1 的 CPU 核心数 = BE 上的 CPU 核心总数 × (2/16) = 2
  > - rg2 的 CPU 核心数 = BE 上的 CPU 核心总数 × (6/16) = 6
  > - rg3 的 CPU 核心数 = BE 上的 CPU 核心总数 × (8/16) = 8
  >
  > 如果 BE 上的 CPU 核心没有全部被占用，比如 rg1 和 rg2 被加载，但 rg3 没有，那么可以根据以下计算分配给 rg1 和 rg2 的 CPU 核心数分别为 4 和 12：
  >
  > - rg1 的 CPU 核心数 = BE 上的 CPU 核心总数 × (2/8) = 4
  > - rg2 的 CPU 核心数 = BE 上的 CPU 核心总数 × (6/8) = 12

- `mem_limit`

  此参数指定可用于 BE 提供的总内存中查询使用的内存百分比。有效值：(0, 1)。

  > **注意**
  >
  > 可用于查询的内存由 `query_pool` 参数表示。有关该参数的更多信息，请参见[内存管理](Memory_management.md)。

- `concurrency_limit`

  此参数指定资源组中的并发查询的上限。它用于避免由太多并发查询引起的系统过载。仅当设置大于 0 时此参数会生效。默认值：0。

- `max_cpu_cores`

  单个 BE 节点上此资源组的 CPU 核心限制。仅当设置大于 `0` 时此参数会生效。范围：[0, `avg_be_cpu_cores`]，其中 `avg_be_cpu_cores` 表示所有 BE 节点上 CPU 核心的平均数。默认值：0。

在上述资源消耗限制的基础上，您可以使用以下参数进一步限制大查询的资源消耗：

- `big_query_cpu_second_limit`：此参数指定单个 BE 上大查询的 CPU 上限时间。并发查询会将时间累加。单位为秒。仅当设置大于 0 时此参数会生效。默认值：0。
- `big_query_scan_rows_limit`：此参数指定单个 BE 上大查询的扫描行数上限。仅当设置大于 0 时此参数会生效。默认值：0。
- `big_query_mem_limit`：此参数指定单个 BE 上大查询的内存使用上限。单位为字节。仅当设置大于 0 时此参数会生效。默认值：0。

> **注意**
>
> 当运行在资源组中的查询超出上述大查询限制时，查询将带有错误终止。您也可以在 FE 节点 **fe.audit.log** 的 `ErrorCode` 列中查看错误消息。

您可以将资源组的 `type` 设置为 `short_query` 或 `normal`。

- 默认值为 `normal`。您不需要在参数 `type` 中指定 `normal`。
- 当查询命中 `short_query` 资源组时，BE 节点会为命中 `short_query` 的查询保留在 `short_query.cpu_core_limit` 中指定的 CPU 资源。命中 `normal` 资源组的查询对 CPU 资源的限制为 `BE 核心数 - short_query.cpu_core_limit`。
- 当没有查询命中 `short_query` 资源组时，对 `normal` 资源组的资源不做限制。

> **注意**
>
> - 在 StarRocks 集群中最多可以创建一个 `short_query` 资源组。
> - StarRocks 不会为 `short_query` 资源组设置硬限制的 CPU 资源上限。

### 分类器

每个分类器包含一个或多个可以与查询属性匹配的条件。StarRocks 根据匹配条件，识别最佳匹配每个查询的分类器，并分配资源以运行查询。

分类器支持以下条件：

- `user`：用户的名称。
- `role`：用户的角色。
- `query_type`：查询的类型。支持 `SELECT` 和 `INSERT`（从 v2.5 开始）。当 INSERT INTO 或 BROKER LOAD 任务命中 `query_type` 为 `insert` 的资源组时，BE 节点会为任务保留指定的 CPU 资源。
- `source_ip`：发起查询的 CIDR 块。
- `db`：查询访问的数据库。可以通过逗号分隔的字符串进行指定。
- `plan_cpu_cost_range`：查询的估算 CPU 成本范围。格式为 `(DOUBLE, DOUBLE]`。默认值为 NULL，表示没有此类限制。`fe.audit.log` 中的 `PlanCpuCost` 列表示查询的系统估算 CPU 成本。此参数从 v3.1.4 开始受支持。
- `plan_mem_cost_range`：查询的系统估算内存成本范围。格式为 `(DOUBLE, DOUBLE]`。默认值为 NULL，表示没有此类限制。`fe.audit.log` 中的 `PlanMemCost` 列表示查询的系统估算内存成本。此参数从 v3.1.4 开始受支持。

只有当分类器的一个或多个条件与查询的信息匹配时，分类器才匹配查询。如果有多个分类器匹配查询，StarRocks会计算查询与每个分类器之间的匹配度，并识别匹配度最高的分类器。

> **注意**
>
> 您可以通过在 FE 节点 **fe.audit.log** 的 `ResourceGroup` 列中查看查询所属的资源组，或者通过运行 `EXPLAIN VERBOSE <query>`，如 [查看查询的资源组](#view-the-resource-group-of-a-query) 中所述来查看。

StarRocks使用以下规则计算查询与分类器之间的匹配度：

- 如果分类器的 `user` 值与查询的值相同，则分类器的匹配度会增加 1。
- 如果分类器的 `role` 值与查询的值相同，则分类器的匹配度会增加 1。
- 如果分类器的`query_type`值与查询相同，则分类器的匹配度增加1加上以下计算获得的数字：1/分类器中的`query_type`字段数量。
- 如果分类器的`source_ip`值与查询相同，则分类器的匹配度增加1加上以下计算获得的数字：(32 - `cidr_prefix`)/64。
- 如果分类器的`db`值与查询相同，则分类器的匹配度增加10。
- 如果查询的CPU成本在`plan_cpu_cost_range`内，则分类器的匹配度增加1。
- 如果查询的内存成本在`plan_mem_cost_range`内，则分类器的匹配度增加1。

如果多个分类器匹配查询，则具有更多条件的分类器具有更高的匹配度。

```Plain
-- 分类器B具有比分类器A更多的条件。因此，分类器B的匹配度高于分类器A。

分类器A (user='Alice')


分类器B (user='Alice', source_ip = '192.168.1.0/24')
```

如果多个匹配的分类器具有相同数量的条件，则条件描述更准确的分类器具有更高的匹配度。

```Plain
-- 在分类器B中指定的CIDR块范围比分类器A小。因此，分类器B的匹配度高于分类器A。
分类器A (user='Alice', source_ip = '192.168.1.0/16')
分类器B (user='Alice', source_ip = '192.168.1.0/24')

-- 分类器C中指定的查询类型比分类器D中的更少。因此，分类器C的匹配度高于分类器D。
分类器C (user='Alice', query_type in ('select'))
分类器D (user='Alice', query_type in ('insert','select'))
```

如果多个分类器具有相同的匹配度，则会随机选择其中一个分类器。

```Plain
-- 如果一个查询同时查询db1和db2，并且分类器E和F是命中分类器中匹配度最高的分类器之一，将随机选择E和F中的一个。
分类器E (db='db1')
分类器F (db='db2')
```

## 隔离计算资源

您可以通过配置资源组和分类器来在查询之间隔离计算资源。

### 启用资源组

要使用资源组，必须为您的 StarRcosk 集群启用 Pipeline Engine：

```SQL
-- 在当前会话中启用 Pipeline Engine。
SET enable_pipeline_engine = true;
-- 在全局范围内启用 Pipeline Engine。
SET GLOBAL enable_pipeline_engine = true;
```

对于加载任务，您还需要设置 FE 配置项 `enable_pipeline_load` 以启用加载任务的 Pipeline engine。从 v2.5.0 开始支持该项。

```sql
ADMIN SET FRONTEND CONFIG ("enable_pipeline_load" = "true");
```

> **注意**
>
> 从 v3.1.0 开始，默认启用 Resource Group，并且会话变量 `enable_resource_group` 已被弃用。

### 创建资源组和分类器

执行以下语句以创建资源组，将资源组与分类器关联，并向资源组分配计算资源：

```SQL
CREATE RESOURCE GROUP <group_name> 
TO (
    user='string', 
    role='string', 
    query_type in ('select'), 
    source_ip='cidr'
) --创建分类器。如果创建了多个分类器，请使用逗号 (`,`) 分隔分类器。
WITH (
    "cpu_core_limit" = "INT",
    "mem_limit" = "m%",
    "concurrency_limit" = "INT",
    "type" = "str" --资源组的类型。将值设置为 normal。
);
```

示例：

```SQL
CREATE RESOURCE GROUP rg1
TO 
    (user='rg1_user1', role='rg1_role1', query_type in ('select'), source_ip='192.168.x.x/24'),
    (user='rg1_user2', query_type in ('select'), source_ip='192.168.x.x/24'),
    (user='rg1_user3', source_ip='192.168.x.x/24'),
    (user='rg1_user4'),
    (db='db1')
WITH (
    'cpu_core_limit' = '10',
    'mem_limit' = '20%',
    'big_query_cpu_second_limit' = '100',
    'big_query_scan_rows_limit' = '100000',
    'big_query_mem_limit' = '1073741824'
);
```

### 指定资源组（可选）

您可以直接为当前会话指定资源组。

```SQL
SET resource_group = 'group_name';
```

### 查看资源组和分类器

执行以下语句以查询所有资源组和分类器：

```SQL
SHOW RESOURCE GROUPS ALL;
```

执行以下语句以查询已登录用户的资源组和分类器：

```SQL
SHOW RESOURCE GROUPS;
```

执行以下语句以查询指定资源组及其分类器：

```SQL
SHOW RESOURCE GROUP group_name;
```

示例：

```plain
mysql> SHOW RESOURCE GROUPS ALL;
+------+--------+--------------+----------+------------------+--------+------------------------------------------------------------------------------------------------------------------------+
| Name | Id     | CPUCoreLimit | MemLimit | ConcurrencyLimit | Type   | Classifiers                                                                                                            |
+------+--------+--------------+----------+------------------+--------+------------------------------------------------------------------------------------------------------------------------+
| rg1  | 300039 | 10           | 20.0%    | 11               | NORMAL | (id=300040, weight=4.409375, user=rg1_user1, role=rg1_role1, query_type in (SELECT), source_ip=192.168.2.1/24)         |
| rg1  | 300039 | 10           | 20.0%    | 11               | NORMAL | (id=300041, weight=3.459375, user=rg1_user2, query_type in (SELECT), source_ip=192.168.3.1/24)                         |
| rg1  | 300039 | 10           | 20.0%    | 11               | NORMAL | (id=300042, weight=2.359375, user=rg1_user3, source_ip=192.168.4.1/24)                                                 |
| rg1  | 300039 | 10           | 20.0%    | 11               | NORMAL | (id=300043, weight=1.0, user=rg1_user4)                                                                                |
+------+--------+--------------+----------+------------------+--------+------------------------------------------------------------------------------------------------------------------------+
```

> **注意**
>
> 在上述示例中，`weight` 表示匹配度。

### 管理资源组和分类器

您可以修改每个资源组的资源配额。您还可以向资源组添加或删除分类器。

执行以下语句以修改现有资源组的资源配额：

```SQL
ALTER RESOURCE GROUP group_name WITH (
    'cpu_core_limit' = 'INT',
    'mem_limit' = 'm%'
);
```

执行以下语句以删除资源组：

```SQL
DROP RESOURCE GROUP group_name;
```

执行以下语句以向资源组添加分类器：

```SQL
ALTER RESOURCE GROUP <group_name> ADD (user='string', role='string', query_type in ('select'), source_ip='cidr');
```

执行以下语句以从资源组中删除分类器：

```SQL
ALTER RESOURCE GROUP <group_name> DROP (CLASSIFIER_ID_1, CLASSIFIER_ID_2, ...);
```

执行以下语句以删除资源组的所有分类器：

```SQL
ALTER RESOURCE GROUP <group_name> DROP ALL;
```

## 观察资源组

### 查看查询的资源组

您可以从 **fe.audit.log** 中的 `ResourceGroup` 列或执行 `EXPLAIN VERBOSE <query>` 后返回的 `RESOURCE GROUP` 列中查看特定查询任务匹配的资源组。它们指示特定查询任务匹配的资源组。

- 如果查询不在资源组的管理范围内，则该列的值为空字符串 `""`。
- 如果查询在资源组的管理范围内但未匹配任何分类器，则该列的值为空字符串 `""`。但是此查询会分配到默认资源组 `default_wg`。

`default_wg` 的资源限制如下：

- `cpu_core_limit`：1（对于 v2.3.7 或更早版本）或 BE 的 CPU 核数（对于 v2.3.7 之后的版本）。
- `mem_limit`：100%。
- `concurrency_limit`：0。
- `big_query_cpu_second_limit`：0。
- `big_query_scan_rows_limit`：0。
- `big_query_mem_limit`：0。

### 监控资源组

您可以为您的资源组设置 [监控及警报](Monitor_and_Alert.md)。

资源组相关的 FE 和 BE 指标如下。以下所有指标都具有指示其对应资源组的 `name` 标签。
以下FE指标仅在当前FE节点范围内提供统计信息：

| 指标                                          | 单位 | 类型          | 描述                                                        |
| ----------------------------------------------- | ---- | ------------- | ------------------------------------------------------------------ |
| starrocks_fe_query_resource_group               | 计数 | 瞬时 | 该资源组中历史运行的查询数量（包括当前运行的查询）。 |
| starrocks_fe_query_resource_group_latency       | 毫秒    | 瞬时 | 该资源组的查询延迟百分位数。标签 `type` 表示特定百分位数，包括 `mean`、`75_quantile`、`95_quantile`、`98_quantile`、`99_quantile`、`999_quantile`。 |
| starrocks_fe_query_resource_group_err           | 计数 | 瞬时 | 该资源组中遇到错误的查询数量。 |
| starrocks_fe_resource_group_query_queue_total   | 计数 | 瞬时 | 该资源组中历史排队的查询总数（包括当前运行的查询）。从v3.1.4版本开始支持此指标。仅在启用查询队列时有效，请参阅[查询队列](query_queues.md)获取详情。 |
| starrocks_fe_resource_group_query_queue_pending | 计数 | 瞬时 | 当前在该资源组队列中的查询数量。从v3.1.4版本开始支持此指标。仅在启用查询队列时有效，请参阅[查询队列](query_queues.md)获取详情。 |
| starrocks_fe_resource_group_query_queue_timeout | 计数 | 瞬时 | 在该资源组中在队列中超时的查询数量。从v3.1.4版本开始支持此指标。仅在启用查询队列时有效，请参阅[查询队列](query_queues.md)获取详情。 |

### BE指标

| 指标                                      | 单位     | 类型          | 描述                                                        |
| ----------------------------------------- | -------- | ------------- | ------------------------------------------------------------------ |
| resource_group_running_queries            | 计数    | 瞬时 | 当前在该资源组中运行的查询数量。   |
| resource_group_total_queries              | 计数    | 瞬时 | 该资源组中历史运行的查询总数（包括当前运行的查询）。  |
| resource_group_bigquery_count             | 计数    | 瞬时 | 在该资源组中触发大查询限制的查询数量。   |
| resource_group_concurrency_overflow_count | 计数    | 瞬时 | 在该资源组中触发 `concurrency_limit` 限制的查询数量。 |
| resource_group_mem_limit_bytes            | 字节    | 瞬时 | 该资源组的内存限制。                         |
| resource_group_mem_inuse_bytes            | 字节    | 瞬时 | 该资源组当前使用的内存。                   |
| resource_group_cpu_limit_ratio            | 百分比 | 瞬时 | 该资源组的 `cpu_core_limit` 与所有资源组的总 `cpu_core_limit` 的比率。   |
| resource_group_inuse_cpu_cores            | 计数     | 平均值     | 该资源组当前使用的CPU核数的估计值。这个值是一个近似估计。它代表基于两次连续指标收集统计信息计算的平均值。从v3.1.4版本开始支持此指标。 |
| resource_group_cpu_use_ratio              | 百分比 | 平均值     | **已弃用** 该资源组使用的管道线程时间片与所有资源组使用的管道线程时间片的比率。它代表基于两次连续指标收集统计信息计算的平均值。 |
| resource_group_connector_scan_use_ratio   | 百分比 | 平均值     | **已弃用** 外部表扫描线程时间片由该资源组使用的管道线程时间片的比率与所有资源组使用的管道线程时间片的比率。它代表基于两次连续指标收集统计信息计算的平均值。 |
| resource_group_scan_use_ratio             | 百分比 | 平均值     | **已弃用** 该资源组使用的内部表扫描线程时间片与所有资源组使用的管道线程时间片的比率。它代表基于两次连续指标收集统计信息计算的平均值。 |

### 查看资源组使用信息

从v3.1.4版本开始，StarRocks支持SQL语句 [SHOW USAGE RESOURCE GROUPS](../sql-reference/sql-statements/Administration/SHOW_USAGE_RESOURCE_GROUPS.md)，用于显示每个BE上每个资源组的使用信息。每个字段的描述如下：

- `Name`: 资源组的名称。
- `Id`: 资源组的ID。
- `Backend`: BE的IP或FQDN。
- `BEInUseCpuCores`: BE上该资源组当前使用的CPU核数。这个值是一个近似估计。
- `BEInUseMemBytes`: BE上该资源组当前使用的内存字节数。
- `BERunningQueries`: 该资源组在该BE上仍在运行的查询数量。

请注意：

- BE定期向Leader FE报告此资源使用信息，报告间隔由 `report_resource_usage_interval_ms` 指定，默认设置为1秒。
- 结果将只显示至少有一个 `BEInUseCpuCores`/`BEInUseMemBytes`/`BERunningQueries` 中有正数的行。换句话说，当资源组在BE上积极使用某些资源时才显示信息。

例子：

```Plain
MySQL [(none)]> SHOW USAGE RESOURCE GROUPS;
+------------+----+-----------+-----------------+-----------------+------------------+
| Name       | Id | Backend   | BEInUseCpuCores | BEInUseMemBytes | BERunningQueries |
+------------+----+-----------+-----------------+-----------------+------------------+
| default_wg | 0  | 127.0.0.1 | 0.100           | 1               | 5                |
+------------+----+-----------+-----------------+-----------------+------------------+
| default_wg | 0  | 127.0.0.2 | 0.200           | 2               | 6                |
+------------+----+-----------+-----------------+-----------------+------------------+
| wg1        | 0  | 127.0.0.1 | 0.300           | 3               | 7                |
+------------+----+-----------+-----------------+-----------------+------------------+
| wg2        | 0  | 127.0.0.1 | 0.400           | 4               | 8                |
+------------+----+-----------+-----------------+-----------------+------------------+
```

## 接下来该怎么办

配置资源组后，您可以管理内存资源和查询。了解更多信息，请参阅以下主题：

- [内存管理](../administration/Memory_management.md)

- [查询管理](../administration/Query_management.md)