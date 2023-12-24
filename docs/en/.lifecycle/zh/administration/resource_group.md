---
displayed_sidebar: English
---

# 资源组

本主题描述了 StarRocks 的资源组功能。

![资源组](../assets/resource_group.png)

借助此功能，您可以在单个集群中同时运行多个工作负载，包括短查询、即席查询、ETL 作业，从而节省部署多个集群的额外成本。从技术角度来看，执行引擎会根据用户的规范调度并发工作负载，并隔离它们之间的干扰。

资源组的路线图：

- 从 v2.2 开始，StarRocks 支持限制查询的资源消耗，实现同一集群租户之间的资源隔离和高效使用。
- 在 StarRocks v2.3 中，您可以进一步限制大查询的资源消耗，防止集群资源因查询请求过大而耗尽，以保证系统的稳定性。
- StarRocks v2.5 支持限制数据加载的计算资源消耗 （INSERT）。

|  | 内部表 | 外部表 | 大查询限制 | 短查询 | INSERT INTO，代理加载  | 例程加载、流加载、架构更改 |
|---|---|---|---|---|---|---|
| 2.2 | √ | × | × | × | × | × |
| 2.3 | √ | √ | √ | √ | × | × |
| 2.4 | √ | √ | √ | √ | × | × |
| 2.5 及更高版本 | √ | √ | √ | √ | √ | × |

## 术语

本部分描述了在使用资源组功能之前必须了解的术语。

### 资源组

每个资源组都是来自特定 BE 的一组计算资源。您可以将集群的每个 BE 划分为多个资源组。当查询分配给资源组时，StarRocks 会根据您为资源组指定的资源配额为该资源组分配 CPU 和内存资源。

可以使用以下参数为 BE 上的资源组指定 CPU 和内存资源配额：

- `cpu_core_limit`

  此参数指定可分配给 BE 上资源组的 CPU 内核数的软限制。有效值：任意非零正整数。范围：（1, `avg_be_cpu_cores`]，其中 `avg_be_cpu_cores` 表示所有 BE 的平均 CPU 内核数。

  在实际业务场景中，分配给资源组的 CPU 核心数会根据 BE 上 CPU 核心的可用性按比例进行扩展。

  > **注意**
  >
  > 例如，在提供 16 个 CPU 内核的 BE 上配置三个资源组：rg1、rg2 和 rg3。这三个资源组的 `cpu_core_limit` 分别为 `2`、 `6` 和 `8`。
  >
  > 如果 BE 的 CPU 核心数全部被占用，则根据以下计算，可分配给三个资源组的 CPU 核心数分别为 2、6 和 8：
  >
  > - rg1 的 CPU 内核数 = BE 上的 CPU 内核总数 × (2/16) = 2
  > - rg2 的 CPU 内核数 = BE 上的 CPU 内核总数 × (6/16) = 6
  > - rg3 的 CPU 内核数 = BE 上的 CPU 内核总数 × (8/16) = 8
  >
  > 如果 BE 的 CPU 核心数没有全部被占用，例如加载了 rg1 和 rg2 但未加载 rg3 时，根据以下计算，可以分配给 rg1 和 rg2 的 CPU 核心数分别为 4 和 12：
  >
  > - rg1 的 CPU 内核数 = BE 上的 CPU 内核总数 × (2/8) = 4
  > - rg2 的 CPU 内核数 = BE 上的 CPU 内核总数 × (6/8) = 12

- `mem_limit`

  此参数指定 BE 提供的总内存中可用于查询的内存百分比。有效值：（0, 1）。

  > **注意**
  >
  > 可用于查询的内存量由 `query_pool` 参数指示。有关该参数的详细信息，请参阅 [内存管理](Memory_management.md)。

- `concurrency_limit`

  该参数指定资源组中并发查询的上限。用于避免并发查询过多导致系统过载。该参数仅在设置为大于 0 时生效。默认值：0。

- `max_cpu_cores`

  此资源组在单个 BE 节点上的 CPU 核心限制。仅当设置为大于 0 时，它才会生效。范围：[0, `avg_be_cpu_cores`]，其中 `avg_be_cpu_cores` 表示所有 BE 节点的平均 CPU 内核数。默认值：0。

在上述资源消耗限制的基础上，可以通过以下参数进一步限制大查询的资源消耗：

- `big_query_cpu_second_limit`：该参数指定单个 BE 上大型查询的 CPU 时间上限。并发查询会使时间相加。单位为秒。该参数仅在设置为大于 0 时生效。默认值：0。
- `big_query_scan_rows_limit`：此参数指定单个 BE 上大型查询的扫描行计数上限。该参数仅在设置为大于 0 时生效。默认值：0。
- `big_query_mem_limit`：指定单个 BE 上大查询的内存使用上限。单位为字节。该参数仅在设置为大于 0 时生效。默认值：0。

> **注意**
>
> 当资源组中运行的查询超过上述大查询限制时，查询将终止并显示错误。您还可以在 FE 节点 **fe.audit.log** 的 `ErrorCode` 列中查看错误消息。

可以将资源组的 `type` 设置为 `short_query` 或 `normal`。

- 缺省值为 `normal`。您不需要在参数 `type` 中指定 `normal`。
- 当查询命中 `short_query` 资源组时，BE 节点会保留 `short_query.cpu_core_limit` 中指定的 CPU 资源。为命中资源组的查询保留的 CPU 资源 `normal` 限制为 `BE core - short_query.cpu_core_limit`。
- 当资源组没有查询命中 `short_query` 时，不会对资源组的资源施加任何限制，`normal`。

> **注意**
>
> - 一个 StarRocks 集群最多可以创建一个短查询资源组。
> - StarRocks 没有为资源组设置 CPU 资源的硬上限 `short_query`。

### 分类器

每个分类器都包含一个或多个可以与查询属性匹配的条件。StarRocks 会根据匹配条件确定每个查询最匹配的分类器，并分配运行查询的资源。

分类器支持以下条件：

- `user`：用户的名称。
- `role`：用户的角色。
- `query_type`：查询的类型。`SELECT` 和 `INSERT`（从 v2.5 开始）被支持。当 INSERT INTO 或 BROKER LOAD 任务命中具有 `query_type` 为 `insert` 的资源组时，BE 节点会为这些任务保留指定的 CPU 资源。
- `source_ip`：从中发起查询的 CIDR 块。
- `db`：查询访问的数据库。它可以通过用逗号分隔的字符串来指定。
- `plan_cpu_cost_range`：查询的估计 CPU 开销范围。格式为 `(DOUBLE, DOUBLE]`。默认值为 NULL，表示没有此类限制。`fe.audit.log` 中的 `PlanCpuCost` 列表示系统对查询的 CPU 开销的估计值。从 v3.1.4 开始支持此参数。
- `plan_mem_cost_range`：查询的系统估计内存成本范围。格式为 `(DOUBLE, DOUBLE]`。默认值为 NULL，表示没有此类限制。`fe.audit.log` 中的 `PlanMemCost` 列表示系统对查询的内存成本的估计值。从 v3.1.4 开始支持此参数。

仅当分类器的一个或所有条件与查询相关信息匹配时，分类器才与查询匹配。如果一个查询有多个分类器匹配，StarRocks 会计算查询与每个分类器之间的匹配度，并识别匹配度最高的分类器。

> **注意**
>
> 您可以在 FE 节点 **fe.audit.log** 的 `ResourceGroup` 列中查看查询所属的资源组，也可以通过运行 `EXPLAIN VERBOSE <query>`，如[查看查询的资源组中所述](#view-the-resource-group-of-a-query)。

StarRocks 使用以下规则计算查询与分类器之间的匹配度：

- 如果分类器的 `user` 值与查询相同，则分类器的匹配度增加 1。
- 如果分类器的 `role` 值与查询相同，则分类器的匹配度增加 1。
- 如果分类器的 `query_type` 值与查询相同，则分类器的匹配度将增加 1 加上从以下计算中获得的数字：1/分类器中的 `query_type` 字段数。
- 如果分类器的 `source_ip` 值与查询相同，则分类器的匹配度将增加 1 加上从以下计算中获得的数字：（32 - `cidr_prefix`）/64。
- 如果分类器的 `db` 值与查询相同，则分类器的匹配度将增加 10。
- 如果查询的 CPU 开销在 `plan_cpu_cost_range` 范围内，则分类器的匹配度增加 1。
- 如果查询的内存开销在 `plan_mem_cost_range` 范围内，则分类器的匹配度增加 1。

如果多个分类器与一个查询匹配，则条件数较多的分类器具有更高的匹配度。

```Plain
-- 分类器 B 的条件数多于分类器 A。因此，分类器 B 的匹配度高于分类器 A。


分类器 A (user='Alice')


分类器 B (user='Alice', source_ip = '192.168.1.0/24')
```

如果多个匹配分类器具有相同数量的条件，则条件描述更准确的分类器具有更高的匹配度。

```Plain
-- 在分类器 B 中指定的 CIDR 块范围小于分类器 A。因此，分类器 B 的匹配度高于分类器 A。
分类器 A (user='Alice', source_ip = '192.168.1.0/16')
分类器 B (user='Alice', source_ip = '192.168.1.0/24')

-- 分类器 C 中指定的查询类型少于分类器 D。因此，分类器 C 的匹配度高于分类器 D。
分类器 C (user='Alice', query_type in ('select'))
分类器 D (user='Alice', query_type in ('insert','select'))
```

如果多个分类器具有相同的匹配度，则将随机选择其中一个分类器。

```Plain
-- 如果一个查询同时查询 db1 和 db2，并且分类器 E 和 F 在命中分类器中具有最高的匹配度，则将随机选择 E 和 F 中的一个。
分类器 E (db='db1')
分类器 F (db='db2')
```

## 隔离计算资源

您可以通过配置资源组和分类器，将计算资源隔离在查询之间。

### 启用资源组

若要使用资源组，必须为 StarRcosk 群集启用管道引擎：

```SQL
-- 在当前会话中启用管道引擎。
SET enable_pipeline_engine = true;
-- 全局启用管道引擎。
SET GLOBAL enable_pipeline_engine = true;
```

对于加载任务，还需要设置 FE 配置项 `enable_pipeline_load` ，以启用加载任务的 Pipeline 引擎。从 v2.5.0 开始支持此项。

```sql

ADMIN SET FRONTEND CONFIG ("enable_pipeline_load" = "true");
```

> **注意**
>
> 从 v3.1.0 版本开始，默认情况下启用了资源组，并且会话变量 `enable_resource_group` 已被弃用。

### 创建资源组和分类器

执行以下语句以创建资源组，将资源组与分类器关联，并为资源组分配计算资源：

```SQL
CREATE RESOURCE GROUP <group_name> 
TO (
    user='string', 
    role='string', 
    query_type in ('select'), 
    source_ip='cidr'
) --创建分类器。如果创建多个分类器，请使用逗号（`,`）分隔分类器。
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

执行以下语句以查询登录用户的资源组和分类器：

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
> 在上述示例中，`weight` 表示匹配程度。

### 管理资源组和分类器

您可以修改每个资源组的资源配额。您还可以向资源组中添加或删除分类器。

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

执行以下语句以向资源组中添加分类器：

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

您可以从 **fe.audit.log** 中的 `ResourceGroup` 列或执行 `EXPLAIN VERBOSE <query>` 后返回的 `RESOURCE GROUP` 列中查看查询命中的资源组。它们指示特定查询任务匹配的资源组。

- 如果查询不在资源组的管理之下，则列值为空字符串 `""`。
- 如果查询受资源组管理，但与任何分类器都不匹配，则列值为空字符串 `""`。但此查询已分配给默认资源组 `default_wg`。

`default_wg` 的资源限制如下：

- `cpu_core_limit`：1（对于 v2.3.7 或更早版本）或 BE 的 CPU 核心数（对于 v2.3.7 之后的版本）。
- `mem_limit`: 100%.
- `concurrency_limit`: 0.
- `big_query_cpu_second_limit`: 0.
- `big_query_scan_rows_limit`: 0.
- `big_query_mem_limit`: 0。

### 监视资源组

您可以为您的资源组设置[监视和警报](Monitor_and_Alert.md)。

FE 和 BE 相关的资源组指标如下。以下所有指标都有一个 `name` 标签，指示其对应的资源组。

### FE 指标

以下 FE 指标仅提供当前 FE 节点内的统计信息：

| 度量                                          | 单位 | 类型          | 描述                                                        |
| ----------------------------------------------- | ---- | ------------- | ------------------------------------------------------------------ |
| starrocks_fe_query_resource_group               | 计数 | 瞬时 | 历史上在此资源组中运行的查询数（包括当前正在运行的查询）。 |
| starrocks_fe_query_resource_group_latency       | ms    | 瞬时 | 此资源组的查询延迟百分位数。标签 `type` 表示特定的百分位数，包括 `mean`、 `75_quantile`、 `95_quantile`、 `98_quantile`、 `99_quantile`、 `999_quantile`。 |
| starrocks_fe_query_resource_group_err           | 计数 | 瞬时 | 此资源组中遇到错误的查询数。 |
| starrocks_fe_resource_group_query_queue_total   | 计数 | 瞬时 | 此资源组中历史上排队的查询总数（包括当前正在运行的查询）。从 v3.1.4 开始支持此指标。仅当启用查询队列时才可用，有关详细信息，请参阅[查询队列](query_queues.md) 。 |
| starrocks_fe_resource_group_query_queue_pending | 计数 | 瞬时 | 此资源组队列中当前存在的查询数。从 v3.1.4 开始支持此指标。仅当启用查询队列时才有效，详见[查询队列](query_queues.md) 。 |
| starrocks_fe_resource_group_query_queue_timeout | 计数 | 瞬时 | 此资源组中在队列中超时的查询数。从 v3.1.4 开始支持此指标。仅当启用查询队列时才有效，详见[查询队列](query_queues.md) 。 |

### BE 指标

| 度量                                      | 单位     | 类型          | 描述                                                        |
| ----------------------------------------- | -------- | ------------- | ------------------------------------------------------------------ |
| resource_group_running_queries            | 计数    | 瞬时 | 此资源组中当前运行的查询数。   |
| resource_group_total_queries              | 计数    | 瞬时 | 历史上在此资源组中运行的查询数（包括当前正在运行的查询）。 |
| resource_group_bigquery_count             | 计数    | 瞬时 | 此资源组中触发大查询限制的查询数。 |
| resource_group_concurrency_overflow_count | 计数    | 瞬时 | 此资源组中触发 `concurrency_limit` 限制的查询数。 |
| resource_group_mem_limit_bytes            | 字节    | 瞬时 | 此资源组的内存限制。                         |
| resource_group_mem_inuse_bytes            | 字节    | 瞬时 | 此资源组当前正在使用的内存。               |
| resource_group_cpu_limit_ratio            | 百分比 | 瞬时 | 此资源组的 `cpu_core_limit` 与所有资源组 `cpu_core_limit` 总数的比率。|
| resource_group_inuse_cpu_cores            | 计数     | 平均     | 此资源组正在使用的估计 CPU 核心数。此值是近似估计值。它表示根据两个连续指标集合的统计数据计算得出的平均值。从 v3.1.4 开始支持此指标。 |
| resource_group_cpu_use_ratio              | 百分比 | 平均     | **已弃用**：此资源组使用的管道线程时间片与所有资源组使用的管道线程时间片总数的比率。它表示根据两个连续指标集合的统计数据计算得出的平均值。|
| resource_group_connector_scan_use_ratio   | 百分比 | 平均     | **已弃用**：此资源组使用的外部表扫描线程时间片与所有资源组使用的管道线程时间片总数的比率。它表示根据两个连续指标集合的统计数据计算得出的平均值。|
| resource_group_scan_use_ratio             | 百分比 | 平均     | **已弃用**：此资源组使用的内部表扫描线程时间片与所有资源组使用的管道线程时间片总数的比率。它表示根据两个连续指标集合的统计数据计算得出的平均值。|

### 查看资源组使用信息

从 v3.1.4 开始，StarRocks 支持 SQL 语句 [SHOW USAGE RESOURCE GROUPS](../sql-reference/sql-statements/Administration/SHOW_USAGE_RESOURCE_GROUPS.md)，用于显示每个 BE 上每个资源组的使用信息。各字段的说明如下：

- `Name`：资源组的名称。
- `Id`：资源组的 ID。
- `Backend`：BE 的 IP 或 FQDN。
- `BEInUseCpuCores`：此资源组在此 BE 上当前使用的 CPU 核心数。此值是近似估计值。
- `BEInUseMemBytes`：此资源组在此 BE 上当前使用的内存字节数。
- `BERunningQueries`：此资源组在此 BE 上仍在运行的查询数。

请注意：

- BE 按照 `report_resource_usage_interval_ms` 中指定的时间间隔定期向 Leader FE 报告此资源使用信息，默认设置为 1 秒。
- 结果仅显示 `BEInUseCpuCores`/`BEInUseMemBytes`/`BERunningQueries` 中至少有一个为正数的行。换言之，仅当资源组主动在 BE 上使用一些资源时，才会显示该信息。

示例：

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

## 接下来该做什么

在配置资源组之后，您可以管理内存资源和查询。有关更多信息，请参阅以下主题：

- [内存管理](../administration/Memory_management.md)

- [查询管理](../administration/Query_management.md)