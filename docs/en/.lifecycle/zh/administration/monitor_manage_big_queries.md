---
displayed_sidebar: English
---

# 监控和管理大型查询

本主题介绍如何在您的 StarRocks 集群中监控和管理大型查询。

大型查询包括扫描过多行或占用过多 CPU 和内存资源的查询。如果不加以限制，它们很容易耗尽集群资源并导致系统过载。为了解决这个问题，StarRocks 提供了一系列措施来监控和管理大型查询，防止查询垄断集群资源。

StarRocks 处理大型查询的总体思路如下：

1. 使用资源组和查询队列为大型查询设置自动预防措施。
2. 实时监控大型查询，并终止那些绕过预防措施的查询。
3. 分析审计日志和大型查询日志，研究大型查询的模式，并对您之前设置的预防机制进行微调。

该功能从 v3.0 版本开始支持。

## 设置大型查询的预防措施

StarRocks 为处理大型查询提供了两种预防工具——资源组和查询队列。您可以使用资源组阻止大型查询的执行。另一方面，查询队列可以帮助您在并发阈值或资源限制达到时对传入的查询进行排队，防止系统过载。

### 通过资源组过滤大型查询

资源组能够自动识别并终止大型查询。创建资源组时，您可以指定查询所允许的 CPU 时间、内存使用量或扫描行数的上限。对于触及资源组的所有查询，任何超出资源要求的查询都将被拒绝并返回错误。更多信息请参见[资源隔离](../administration/resource_group.md)。

在创建资源组之前，您必须执行以下语句以启用资源组功能所依赖的 Pipeline Engine：

```SQL
SET GLOBAL enable_pipeline_engine = true;
```

以下示例创建了一个名为 `bigQuery` 的资源组，该资源组将 CPU 时间上限设置为 `100` 秒，扫描行数上限设置为 `100000`，内存使用上限设置为 `1073741824` 字节（1 GB）：

```SQL
CREATE RESOURCE GROUP bigQuery
TO 
    (db='sr_hub')
WITH (
    'cpu_core_limit' = '10',
    'mem_limit' = '20%',
    'big_query_cpu_second_limit' = '100',
    'big_query_scan_rows_limit' = '100000',
    'big_query_mem_limit' = '1073741824'
);
```

如果查询所需的资源超过任何限制，该查询将不会被执行，并会返回错误。以下示例显示了当查询请求的扫描行数过多时返回的错误消息：

```Plain
ERROR 1064 (HY000): exceed big query scan_rows limit: current is 4 but limit is 1
```

如果这是您第一次设置资源组，我们建议您设置相对较高的限制，以免妨碍常规查询。在您更好地了解大型查询模式之后，可以对这些限制进行微调。

### 通过查询队列缓解系统过载

查询队列旨在缓解当集群资源占用超过预设阈值时的系统过载。您可以设置最大并发数、内存使用量和 CPU 使用量的阈值。当任一阈值达到时，StarRocks 会自动将传入的查询排入队列。待处理的查询要么在队列中等待执行，要么在达到预设的资源阈值时被取消。更多信息请参见[查询队列](../administration/query_queues.md)。

执行以下语句以启用 SELECT 查询的查询队列：

```SQL
SET GLOBAL enable_query_queue_select = true;
```

在启用查询队列功能之后，您可以定义触发查询队列的规则。

- 指定触发查询队列的并发阈值。

  下面的示例将并发阈值设置为 `100`：

  ```SQL
  SET GLOBAL query_queue_concurrency_limit = 100;
  ```

- 指定触发查询队列的内存使用比例阈值。

  下面的示例将内存使用比例阈值设置为 `0.9`：

  ```SQL
  SET GLOBAL query_queue_mem_used_pct_limit = 0.9;
  ```

- 指定触发查询队列的 CPU 使用比例阈值。

  下面的示例将 CPU 使用比例千分比（CPU 使用率 * 1000）阈值设置为 `800`：

  ```SQL
  SET GLOBAL query_queue_cpu_used_permille_limit = 800;
  ```

您还可以通过配置队列中每个待处理查询的最大队列长度和超时时间来决定如何处理这些排队的查询。

- 指定最大查询队列长度。当达到此阈值时，新传入的查询将被拒绝。

  下面的示例将查询队列长度设置为 `100`：

  ```SQL
  SET GLOBAL query_queue_max_queued_queries = 100;
  ```

- 指定队列中待处理查询的最大超时时间。当达到此阈值时，相应的查询将被拒绝。

  下面的示例将最大超时时间设置为 `480` 秒：

  ```SQL
  SET GLOBAL query_queue_pending_timeout_second = 480;
  ```

您可以使用 [SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md) 检查查询是否处于待处理状态。

```Plain
mysql> SHOW PROCESSLIST;
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
| Id   | User | Host                | Db    | Command | ConnectionStartTime | Time | State | Info              | IsPending |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
|    2 | root | xxx.xx.xxx.xx:xxxxx |       | Query   | 2022-11-24 18:08:29 |    0 | OK    | SHOW PROCESSLIST  | false     |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
```

如果 `IsPending` 为 `true`，则相应的查询正在查询队列中等待。

## 实时监控大型查询

从 v3.0 版本开始，StarRocks 支持查看集群中当前正在处理的查询及其占用的资源。这允许您监控集群，以防任何大型查询绕过预防措施，导致意外的系统过载。

```
### 通过 MySQL 客户端监控

1. 您可以使用 [SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md) 查看当前正在处理的查询（`current_queries`）。

   ```SQL
   SHOW PROC '/current_queries';
   ```

   StarRocks 返回查询 ID（`QueryId`）、连接 ID（`ConnectionId`）以及每个查询的资源消耗，包括扫描数据大小（`ScanBytes`）、处理行数（`ProcessRows`）、CPU 时间（`CPUCostSeconds`）、内存使用量（`MemoryUsageBytes`）和执行时间（`ExecTime`）。

   ```Plain
   mysql> SHOW PROC '/current_queries';
   +--------------------------------------+--------------+------------+------+-----------+----------------+----------------+------------------+----------+
   | QueryId                              | ConnectionId | Database   | User | ScanBytes | ProcessRows    | CPUCostSeconds | MemoryUsageBytes | ExecTime |
   +--------------------------------------+--------------+------------+------+-----------+----------------+----------------+------------------+----------+
   | 7c56495f-ae8b-11ed-8ebf-00163e00accc | 4            | tpcds_100g | root | 37.88 MB  | 1075769 Rows   | 11.13 Seconds  | 146.70 MB        | 3804     |
   | 7d543160-ae8b-11ed-8ebf-00163e00accc | 6            | tpcds_100g | root | 13.02 GB  | 487873176 Rows | 81.23 Seconds  | 6.37 GB          | 2090     |
   +--------------------------------------+--------------+------------+------+-----------+----------------+----------------+------------------+----------+
   2 rows in set (0.01 sec)
   ```

2. 您可以通过指定查询 ID 来进一步检查查询在每个 BE 节点上的资源消耗。

   ```SQL
   SHOW PROC '/current_queries/<QueryId>/hosts';
   ```

   StarRocks 返回每个 BE 节点上查询的扫描数据大小（`ScanBytes`）、扫描行数（`ScanRows`）、CPU 时间（`CPUCostSeconds`）和内存使用情况（`MemUsageBytes`）。

   ```Plain
   mysql> show proc '/current_queries/7c56495f-ae8b-11ed-8ebf-00163e00accc/hosts';
   +--------------------+-----------+-------------+----------------+---------------+
   | Host               | ScanBytes | ScanRows    | CpuCostSeconds | MemUsageBytes |
   +--------------------+-----------+-------------+----------------+---------------+
   | 172.26.34.185:8060 | 11.61 MB  | 356252 Rows | 52.93 Seconds  | 51.14 MB      |
   | 172.26.34.186:8060 | 14.66 MB  | 362646 Rows | 52.89 Seconds  | 50.44 MB      |
   | 172.26.34.187:8060 | 11.60 MB  | 356871 Rows | 52.91 Seconds  | 48.95 MB      |
   +--------------------+-----------+-------------+----------------+---------------+
   3 rows in set (0.00 sec)
   ```

### 通过 FE 控制台监控

除了 MySQL 客户端外，您还可以使用 FE 控制台进行可视化、交互式监控。

1. 使用以下 URL 在浏览器中导航到 FE 控制台：

   ```Bash
   http://<fe_IP>:<fe_http_port>/system?path=//current_queries
   ```

   ![FE 控制台 1](../assets/console_1.png)

   您可以在 **系统信息** 页面查看当前正在处理的查询及其资源消耗。

2. 点击查询的 **QueryID**。

   ![FE 控制台 2](../assets/console_2.png)

   您可以在出现的页面上查看详细的、特定于节点的资源消耗信息。

### 手动终止大型查询

如果任何大型查询绕过了您设置的预防措施并威胁系统可用性，您可以使用相应的连接 ID 在 [KILL](../sql-reference/sql-statements/Administration/KILL.md) 语句中手动终止它们：

```SQL
KILL QUERY <ConnectionId>;
```

## 分析大型查询日志

从 v3.0 版本开始，StarRocks 支持大型查询日志（Big Query Logs），这些日志存储在文件 **fe/log/fe.big_query.log** 中。与 StarRocks 审计日志相比，大型查询日志打印额外的三个字段：

- `bigQueryLogCPUSecondThreshold`
- `bigQueryLogScanBytesThreshold`
- `bigQueryLogScanRowsThreshold`

这三个字段对应于您定义的资源消耗阈值，用于确定查询是否为大型查询。

要启用大型查询日志，请执行以下语句：

```SQL
SET GLOBAL enable_big_query_log = true;
```

启用大型查询日志后，您可以定义触发大型查询日志的规则。

- 指定触发大型查询日志的 CPU 时间阈值。

  以下示例将 CPU 时间阈值设置为 `600` 秒：

  ```SQL
  SET GLOBAL big_query_log_cpu_second_threshold = 600;
  ```

- 指定触发大型查询日志的扫描数据大小阈值。

  以下示例将扫描数据大小阈值设置为 `10737418240` 字节（10 GB）：

  ```SQL
  SET GLOBAL big_query_log_scan_bytes_threshold = 10737418240;
  ```

- 指定触发大型查询日志的扫描行数阈值。

  以下示例将扫描行数阈值设置为 `1500000000`：

  ```SQL
  SET GLOBAL big_query_log_scan_rows_threshold = 1500000000;
  ```

## 微调预防措施

通过实时监控和大型查询日志的统计数据，您可以研究集群中遗漏的大型查询（或误诊为大型查询的常规查询）的模式，进而优化资源组和查询队列的设置。

如果相当比例的大型查询符合某种 SQL 模式，并且您希望永久禁止此 SQL 模式，则可以将此模式添加到 SQL 黑名单。StarRocks 会拒绝所有与 SQL 黑名单中指定的模式匹配的查询，并返回错误。有关详细信息，请参阅[管理 SQL 黑名单](../administration/Blacklist.md)。

要启用 SQL 黑名单，请执行以下语句：

```SQL
ADMIN SET FRONTEND CONFIG ("enable_sql_blacklist" = "true");
```

然后您可以使用 [ADD SQLBLACKLIST](../sql-reference/sql-statements/Administration/ADD_SQLBLACKLIST.md) 将代表 SQL 模式的正则表达式添加到 SQL 黑名单。

以下示例将 `COUNT(DISTINCT)` 添加到 SQL 黑名单：

```SQL
ADD SQLBLACKLIST "SELECT COUNT(DISTINCT .+) FROM .+";
```
以下示例将 `COUNT(DISTINCT)` 添加到 SQL 黑名单：

```SQL
ADD SQLBLACKLIST "SELECT COUNT(DISTINCT .+) FROM .+";
```