---
displayed_sidebar: English
---

# 资源组

本主题介绍 StarRocks 的资源组功能。

![resource group](../assets/resource_group.png)

通过此功能，您可以在单个集群中同时运行多个工作负载，包括短查询、即席查询、ETL 作业，以节省部署多个集群的额外成本。从技术角度来看，执行引擎会根据用户的指定来调度并发工作负载，并隔离它们之间的干扰。

资源组的路线图：

- 自 v2.2 起，StarRocks 支持限制查询的资源消耗，实现同一集群内租户之间的隔离和资源的高效利用。
- 在 StarRocks v2.3 中，您可以进一步限制大查询的资源消耗，防止集群资源因过大的查询请求而耗尽，以保证系统稳定性。
- StarRocks v2.5 支持限制数据加载（INSERT）的计算资源消耗。

|内部表|外部表|大查询限制|短查询|INSERT INTO、Broker Load|Routine Load、Stream Load、Schema Change|
|---|---|---|---|---|---|
|2.2|√|×|×|×|×|×|
|2.3|√|√|√|√|×|×|
|2.4|√|√|√|√|×|×|
|2.5 及以后|√|√|√|√|√|×|

## 术语

本节描述了在使用资源组功能之前您必须了解的术语。

### 资源组

每个资源组是来自特定 BE 的一组计算资源。您可以将集群的每个 BE 划分为多个资源组。当查询分配给资源组时，StarRocks 会根据您为资源组指定的资源配额为资源组分配 CPU 和内存资源。

您可以使用以下参数为 BE 上的资源组指定 CPU 和内存资源配额：

- `cpu_core_limit`

  此参数指定 BE 上可分配给资源组的 CPU 核心数量的软限制。有效值：任何非零正整数。范围：(1, `avg_be_cpu_cores`]，其中 `avg_be_cpu_cores` 表示所有 BE 的平均 CPU 核心数。

  在实际业务场景中，分配给资源组的 CPU 核心数量会根据 BE 上 CPU 核心的可用性进行比例缩放。

    > **注意**
    > 例如，您在提供 16 个 CPU 核心的 BE 上配置三个资源组：rg1、rg2 和 rg3。三个资源组的 `cpu_core_limit` 值分别为 `2`、`6` 和 `8`。
    > 如果 BE 的所有 CPU 核心都被占用，可以分配给三个资源组的 CPU 核心数分别为 2、6 和 8，基于以下计算：
  - rg1 的 CPU 核心数 = BE 上的 CPU 核心总数 × (2/16) = 2
  - rg2 的 CPU 核心数 = BE 上的 CPU 核心总数 × (6/16) = 6
  - rg3 的 CPU 核心数 = BE 上的 CPU 核心总数 × (8/16) = 8
    > 如果 BE 的 CPU 核心没有全部被占用，例如 rg1 和 rg2 正在运行但 rg3 没有时，可以分配给 rg1 和 rg2 的 CPU 核心数分别为 4 和 12，基于以下计算：
  - rg1 的 CPU 核心数 = BE 上的 CPU 核心总数 × (2/8) = 4
  - rg2 的 CPU 核心数 = BE 上的 CPU 核心总数 × (6/8) = 12

- `mem_limit`

  此参数指定可用于查询的内存占 BE 提供的总内存的百分比。有效值：(0, 1)。

    > **注意**
    > 可用于查询的内存量由 `query_pool` 参数指示。有关该参数的更多信息，请参阅[内存管理](Memory_management.md)。

- `concurrency_limit`

  此参数指定资源组中并发查询的上限。它用于避免因并发查询过多而导致系统过载。此参数仅在设置大于 0 时生效。默认值：0。

- `max_cpu_cores`

  此资源组在单个 BE 节点上的 CPU 核心限制。仅当设置大于 `0` 时才生效。范围：[0, `avg_be_cpu_cores`]，其中 `avg_be_cpu_cores` 表示所有 BE 节点的平均 CPU 核心数。默认值：0。

在上述资源消耗限制的基础上，您可以进一步限制大查询的资源消耗，通过以下参数：

- `big_query_cpu_second_limit`：此参数指定单个 BE 上大查询的 CPU 时间上限。并发查询会累加时间。单位是秒。此参数仅在设置大于 0 时生效。默认值：0。
- `big_query_scan_rows_limit`：此参数指定单个 BE 上大查询的扫描行数上限。此参数仅在设置大于 0 时生效。默认值：0。
- `big_query_mem_limit`：此参数指定单个 BE 上大查询的内存使用上限。单位是字节。此参数仅在设置大于 0 时生效。默认值：0。

> **注意**
> 当资源组中运行的查询超出上述大查询限制时，查询将因错误而终止。您还可以在 FE 节点 **fe.audit.log** 的 `ErrorCode` 列中查看错误消息。

您可以将资源组 `type` 设置为 `short_query` 或 `normal`。

- 默认值是 `normal`。您不需要在参数 `type` 中指定 `normal`。
- 当查询命中 `short_query` 资源组时，BE 节点保留 `short_query.cpu_core_limit` 中指定的 CPU 资源。为命中 `normal` 资源组的查询保留的 CPU 资源限制为 `BE 核心数 - short_query.cpu_core_limit`。
- 当没有查询命中 `short_query` 资源组时，对 `normal` 资源组的资源不施加限制。

> **警告**
- 您最多可以在 StarRocks 集群中创建一个短查询资源组。
- StarRocks 没有为 `short_query` 资源组设置 CPU 资源的硬性上限。

### 分类器

每个分类器都包含一个或多个可以与查询属性相匹配的条件。StarRocks 根据匹配条件识别与每个查询最匹配的分类器，并分配用于运行查询的资源。

分类器支持以下条件：

- `user`：用户的名称。
- `role`：用户的角色。
- `query_type`：查询的类型。支持 `SELECT` 和 `INSERT`（从 v2.5 起）。当 INSERT INTO 或 BROKER LOAD 任务命中 `query_type` 为 `insert` 的资源组时，BE 节点为任务保留指定的 CPU 资源。
- `source_ip`：发起查询的 CIDR 块。
- `db`：查询访问的数据库。可以用逗号 `,` 分隔的字符串来指定。
- `plan_cpu_cost_range`：查询的估计 CPU 成本范围。格式为 `(DOUBLE, DOUBLE]`。默认值为 NULL，表示没有此限制。`fe.audit.log` 中的 `PlanCpuCost` 列表示系统对查询的 CPU 成本的估计。从 v3.1.4 起支持此参数。
- `plan_mem_cost_range`：系统估计的查询内存成本范围。格式为 `(DOUBLE, DOUBLE]`。默认值为 NULL，表示没有此限制。`fe.audit.log` 中的 `PlanMemCost` 列表示系统对查询的内存成本的估计。从 v3.1.4 起支持此参数。

仅当分类器的一个或所有条件与查询的信息匹配时，分类器才匹配查询。如果多个分类器与查询匹配，StarRocks 会计算查询与每个分类器之间的匹配度，并识别匹配度最高的分类器。

> **注意**
> 您可以在 FE 节点 **fe.audit.log** 的 `ResourceGroup` 列中查看查询所属的资源组，或者通过执行 `EXPLAIN VERBOSE <query>` 来查看，如[查看查询的资源组](#view-the-resource-group-of-a-query)中所述。

StarRocks 使用以下规则计算查询和分类器之间的匹配度：

- 如果分类器具有与查询相同的 `user` 值，则分类器的匹配度增加 1。
- 如果分类器与查询具有相同的 `role` 值，则分类器的匹配度增加 1。
- 如果分类器的 `query_type` 值与查询相同，则分类器的匹配度增加 1 加上通过以下计算得到的数字：1/分类器中 `query_type` 字段的数量。
- 如果分类器的 `source_ip` 值与查询相同，则分类器的匹配度增加 1 加上通过以下计算获得的数字：(32 - `cidr_prefix`)/64。
- 如果分类器的 `db` 值与查询相同，则分类器的匹配度增加 10。
- 如果查询的 CPU 成本落在 `plan_cpu_cost_range` 内，则分类器的匹配度增加 1。
- 如果查询的内存成本落在 `plan_mem_cost_range` 内，则分类器的匹配度增加 1。

如果多个分类器匹配一个查询，条件数量较多的分类器匹配度较高。

```Plain
-- 分类器 B 比分类器 A 有更多的条件。因此，分类器 B 的匹配度高于分类器 A。


classifier A (user='Alice')


classifier B (user='Alice', source_ip = '192.168.1.0/24')
```

如果多个匹配分类器的条件数量相同，条件描述越准确的分类器匹配程度越高。

```Plain
-- 分类器 B 指定的 CIDR 块范围比分类器 A 小。因此，分类器 B 的匹配度高于分类器 A。
classifier A (user='Alice', source_ip = '192.168.1.0/16')
classifier B (user='Alice', source_ip = '192.168.1.0/24')

-- 分类器 C 指定的查询类型比分类器 D 少。因此，分类器 C 的匹配度高于分类器 D。
classifier C (user='Alice', query_type in ('select'))
classifier D (user='Alice', query_type in ('insert', 'select'))
```

如果多个分类器的匹配度相同，将随机选择其中一个分类器。

```Plain
-- 如果一个查询同时查询 db1 和 db2，并且分类器 E 和 F 在命中的分类器中具有
-- 最高的匹配度，将随机选择 E 或 F 其中之一。
classifier E (db='db1')
classifier F (db='db2')
```

## 隔离计算资源

您可以通过配置资源组和分类器来隔离查询之间的计算资源。

### 启用资源组

要使用资源组，您必须为 StarRocks 集群启用 Pipeline Engine：

```SQL
-- 在当前会话中启用 Pipeline Engine。
SET enable_pipeline_engine = true;
-- 全局启用 Pipeline Engine。
SET GLOBAL enable_pipeline_engine = true;
```

对于加载任务，还需要设置 FE 配置项 `enable_pipeline_load` 来启用 Pipeline 引擎加载任务。此项自 v2.5.0 起支持。

```sql
ADMIN SET FRONTEND CONFIG ("enable_pipeline_load" = "true");
```

> **注意**
> 从 v3.1.0 起，资源组默认启用，会话变量 `enable_resource_group` 已弃用。
```
> 从 v3.1.0 版本开始，默认启用资源组功能，`enable_resource_group` 会话变量已不推荐使用。

### 创建资源组和分类器

执行以下 SQL 语句以创建资源组，将资源组与分类器关联，并为资源组分配计算资源：

```SQL
CREATE RESOURCE GROUP <group_name> 
TO (
    user='string', 
    role='string', 
    query_type in ('select'), 
    source_ip='cidr'
) -- 创建一个分类器。如果您创建了多个分类器，请用逗号（`,`）分隔。
WITH (
    "cpu_core_limit" = "INT",
    "mem_limit" = "m%",
    "concurrency_limit" = "INT",
    "type" = "str" -- 资源组的类型。将值设置为 normal。
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

执行以下 SQL 语句查询所有资源组和分类器：

```SQL
SHOW RESOURCE GROUPS ALL;
```

执行以下 SQL 语句查询当前登录用户的资源组和分类器：

```SQL
SHOW RESOURCE GROUPS;
```

执行以下 SQL 语句查询指定资源组及其分类器：

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
> 在上述示例中，`weight` 表示匹配程度。

### 管理资源组和分类器

您可以修改每个资源组的资源配额。您还可以向资源组中添加或删除分类器。

执行以下 SQL 语句以修改现有资源组的资源配额：

```SQL
ALTER RESOURCE GROUP group_name WITH (
    'cpu_core_limit' = 'INT',
    'mem_limit' = 'm%'
);
```

执行以下 SQL 语句以删除资源组：

```SQL
DROP RESOURCE GROUP group_name;
```

执行以下 SQL 语句以向资源组中添加分类器：

```SQL
ALTER RESOURCE GROUP <group_name> ADD (user='string', role='string', query_type in ('select'), source_ip='cidr');
```

执行以下 SQL 语句以从资源组中删除分类器：

```SQL
ALTER RESOURCE GROUP <group_name> DROP (CLASSIFIER_ID_1, CLASSIFIER_ID_2, ...);
```

执行以下 SQL 语句以删除资源组的所有分类器：

```SQL
ALTER RESOURCE GROUP <group_name> DROP ALL;
```

## 观察资源组

### 查看查询的资源组

您可以从 **fe.audit.log** 中的 `ResourceGroup` 列或从执行 `EXPLAIN VERBOSE <query>` 后返回的 `RESOURCE GROUP` 列查看查询命中的资源组。它们指示特定查询任务匹配的资源组。

- 如果查询不受资源组管理，则该列值为空字符串 `""`。
- 如果查询受资源组管理但不匹配任何分类器，则列值为空字符串 `""`。但该查询被分配到默认资源组 `default_wg`。

`default_wg` 的资源限制如下：

- `cpu_core_limit`：1（对于 v2.3.7 或更早版本）或 BE 的 CPU 核心数（对于 v2.3.7 之后的版本）。
- `mem_limit`：100%。
- `concurrency_limit`：0。
- `big_query_cpu_second_limit`：0。
- `big_query_scan_rows_limit`：0。
- `big_query_mem_limit`：0。

### 监控资源组

您可以为您的资源组设置[监控和报警](Monitor_and_Alert.md)。

资源组相关的 FE 和 BE 指标如下。下面的所有指标都有一个 `name` 标签，指示其对应的资源组。

### FE 指标

以下 FE 指标仅提供当前 FE 节点内的统计信息：

| 指标 | 单位 | 类型 | 描述 |
|---|---|---|---|
| starrocks_fe_query_resource_group | 计数 | 瞬时 | 此资源组中历史上运行的查询数（包括当前正在运行的查询）。|
| starrocks_fe_query_resource_group_latency | 毫秒 | 瞬时 | 此资源组的查询延迟百分位。标签 `type` 指示特定的百分位数，包括 `mean`、`75_quantile`、`95_quantile`、`98_quantile`、`99_quantile`、`999_quantile`。|
| starrocks_fe_query_resource_group_err | 计数 | 瞬时 | 此资源组中遇到错误的查询数量。|
| starrocks_fe_resource_group_query_queue_total | 计数 | 瞬时 | 历史上在此资源组中排队的查询总数（包括当前正在运行的查询）。从 v3.1.4 版本开始支持此指标。仅当启用查询队列时才有效，详细信息请参阅[查询队列](query_queues.md)。|
| starrocks_fe_resource_group_query_queue_pending | 计数 | 瞬时 | 当前在此资源组队列中的查询数。从 v3.1.4 版本开始支持此指标。仅当启用查询队列时才有效，请参阅[查询队列](query_queues.md)了解详细信息。|
| starrocks_fe_resource_group_query_queue_timeout | 计数 | 瞬时 | 此资源组中在队列中超时的查询数。从 v3.1.4 版本开始支持此指标。仅当启用查询队列时才有效，请参阅[查询队列](query_queues.md)了解详细信息。|

### BE 指标

| 指标 | 单位 | 类型 | 描述 |
|---|---|---|---|
| resource_group_running_queries | 计数 | 瞬时 | 当前在此资源组中运行的查询数。|
| resource_group_total_queries | 计数 | 瞬时 | 此资源组中历史上运行的查询数（包括当前正在运行的查询）。|
| resource_group_bigquery_count | 计数 | 瞬时 | 此资源组中触发大查询限制的查询数。|
| resource_group_concurrency_overflow_count | 计数 | 瞬时 | 此资源组中触发 `concurrency_limit` 限制的查询数。|
| resource_group_mem_limit_bytes | 字节 | 瞬时 | 此资源组的内存限制。|
| resource_group_mem_inuse_bytes | 字节 | 瞬时 | 此资源组当前使用的内存。|
| resource_group_cpu_limit_ratio | 百分比 | 瞬时 | 此资源组的 `cpu_core_limit` 与所有资源组的总 `cpu_core_limit` 的比率。|
| resource_group_inuse_cpu_cores | 计数 | 平均 | 此资源组正在使用的 CPU 核心的估计数量。该值是一个近似估计值。它表示根据两个连续指标采集的统计信息计算出的平均值。从 v3.1.4 版本开始支持此指标。|
| resource_group_cpu_use_ratio | 百分比 | 平均 | **已弃用** 此资源组使用的 Pipeline 线程时间片与所有资源组使用的 Pipeline 线程时间片总数的比率。它表示根据两个连续指标采集的统计信息计算出的平均值。|
| resource_group_connector_scan_use_ratio | 百分比 | 平均 | **已弃用** 此资源组使用的外部表扫描线程时间片与所有资源组使用的总 Pipeline 线程时间片的比率。它表示根据两个连续指标采集的统计信息计算出的平均值。|
| resource_group_scan_use_ratio | 百分比 | 平均 | **已弃用** 此资源组使用的内部表扫描线程时间片与所有资源组使用的 Pipeline 线程总时间片的比率。它表示根据两个连续指标采集的统计信息计算出的平均值。|

### 查看资源组使用信息

从 v3.1.4 版本开始，StarRocks 支持 SQL 语句 [SHOW USAGE RESOURCE GROUPS](../sql-reference/sql-statements/Administration/SHOW_USAGE_RESOURCE_GROUPS.md)，用于显示跨 BE 的每个资源组的使用信息。各字段的说明如下：

- `Name`：资源组的名称。
- `Id`：资源组的 ID。
- `Backend`：BE 的 IP 或 FQDN。
- `BEInUseCpuCores`：此 BE 上的该资源组当前使用的 CPU 核心数。该值是一个近似估计值。
- `BEInUseMemBytes`：此 BE 上的此资源组当前正在使用的内存字节数。
- `BERunningQueries`：来自该资源组且仍在该 BE 上运行的查询数。

请注意：

- BE 按照 `report_resource_usage_interval_ms` 中指定的时间间隔定期向 Leader FE 报告此资源使用信息，默认情况下设置为 1 秒。
- 结果将仅显示 `BEInUseCpuCores`/`BEInUseMemBytes`/`BERunningQueries` 中至少有一个为正数的行。换句话说，只有当资源组正在积极使用 BE 上的某些资源时，才会显示该信息。

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
```
```Plain
| wg1        | 0  | 127.0.0.1 | 0.300           | 3               | 7                |
+------------+----+-----------+-----------------+-----------------+------------------+
| wg2        | 0  | 127.0.0.1 | 0.400           | 4               | 8                |
+------------+----+-----------+-----------------+-----------------+------------------+
```

## 下一步该做什么

在配置资源组之后，您可以对内存资源和查询进行管理。更多相关信息，请参考以下主题：

- [内存管理](../administration/Memory_management.md)

- [查询管理](../administration/Query_management.md)