---
displayed_sidebar: English
---

# 监控和管理大型查询

本主题介绍如何在您的 StarRocks 集群中监控和管理大型查询。

所谓大型查询，指的是扫描行数过多或占用大量 CPU 和内存资源的查询。若不加以限制，这些查询极易耗尽集群资源并引发系统过载。为解决此问题，StarRocks 提供了一系列措施来监控和管理大型查询，避免它们垄断集群资源。

StarRocks 处理大型查询的整体思路如下：

1. 通过资源组和查询队列，为大型查询设置自动化的预防措施。
2. 实时监控大型查询，并终止那些绕过预防措施的查询。
3. 分析审计日志和大型查询日志，以研究大型查询的模式，并对之前设置的预防机制进行微调。

该功能从 v3.0 版本开始支持。

## 对大型查询设置预防措施

StarRocks 为处理大型查询提供了两种预防措施：资源组和查询队列。您可以利用资源组来阻止大型查询的执行。而查询队列则可以在并发阈值或资源限制达到时，帮助您将传入的查询进行排队，以防止系统过载。

### 通过资源组筛选大型查询

资源组能够自动识别并终止大型查询。在创建资源组时，您可以指定查询的CPU时间、内存使用量或扫描行数的上限。在所有触发资源组限制的查询中，任何需要更多资源的查询都会被拒绝并返回错误。更多信息，请参见[资源隔离](../administration/resource_group.md)。

在创建资源组之前，您必须执行以下语句以启用资源组功能所依赖的 Pipeline Engine：

```SQL
SET GLOBAL enable_pipeline_engine = true;
```

以下示例创建了一个名为 bigQuery 的资源组，限制了 CPU 时间的上限为 100 秒，扫描行数的上限为 100000，内存使用的上限为 1073741824 字节（1 GB）：

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

如果查询所需资源超过任一限制，该查询将不会被执行，并会返回错误信息。以下示例展示了当查询请求的扫描行数过多时，返回的错误信息：

```Plain
ERROR 1064 (HY000): exceed big query scan_rows limit: current is 4 but limit is 1
```

如果您是首次设置资源组，我们建议您设定较高的限制，以免影响正常查询。在您对大型查询模式有了更深入的了解后，可以适当调整这些限制。

### 通过查询队列减轻系统负载

查询队列旨在缓解当集群资源占用超过预设阈值时的系统负载问题。您可以为最大并发数、内存使用率和 CPU 使用率设置阈值。当任一阈值达到时，StarRocks会自动将传入的查询排入队列。排队中的查询要么等待执行，要么在达到预设的资源阈值时被取消。更多详情，请参见[查询队列](../administration/query_queues.md)部分。

执行以下语句以对 SELECT 查询启用查询队列功能：

```SQL
SET GLOBAL enable_query_queue_select = true;
```

启用查询队列功能后，您可以开始定义触发查询队列的规则。

- 设定触发查询队列的并发阈值。

  以下示例将并发阈值设为 100：

  ```SQL
  SET GLOBAL query_queue_concurrency_limit = 100;
  ```

- 设定触发查询队列的内存使用率阈值。

  以下示例将内存使用率阈值设为 0.9：

  ```SQL
  SET GLOBAL query_queue_mem_used_pct_limit = 0.9;
  ```

- 设定触发查询队列的 CPU 使用率阈值。

  以下示例将 CPU 使用率千分比（CPU 使用率 * 1000）阈值设为 800：

  ```SQL
  SET GLOBAL query_queue_cpu_used_permille_limit = 800;
  ```

您还可以通过配置队列中每个待处理查询的最大队列长度和超时时间来决定如何处理这些排队的查询。

- 设定最大查询队列长度。当达到此阈值时，新传入的查询将被拒绝。

  以下示例将查询队列长度设为 100：

  ```SQL
  SET GLOBAL query_queue_max_queued_queries = 100;
  ```

- 设定队列中待处理查询的最大超时时间。当达到此阈值时，相应的查询将被拒绝。

  以下示例将最大超时时间设为 480 秒：

  ```SQL
  SET GLOBAL query_queue_pending_timeout_second = 480;
  ```

您可以使用[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md)命令检查查询是否处于待处理状态。

```Plain
mysql> SHOW PROCESSLIST;
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
| Id   | User | Host                | Db    | Command | ConnectionStartTime | Time | State | Info              | IsPending |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
|    2 | root | xxx.xx.xxx.xx:xxxxx |       | Query   | 2022-11-24 18:08:29 |    0 | OK    | SHOW PROCESSLIST  | false     |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
```

如果 IsPending 为 true，表示相关查询正在查询队列中等待。

## 实时监控大型查询

从 v3.0 版本起，StarRocks 支持查看集群中当前正在处理的查询及其占用的资源。这允许您监控集群，防止任何大型查询绕过预防措施，导致系统出现意外过载。

### 通过 MySQL 客户端监控

1. 您可以使用[SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md)命令查看当前正在处理的查询（`current_queries`）。

   ```SQL
   SHOW PROC '/current_queries';
   ```

   StarRocks 返回查询 ID（QueryId）、连接 ID（ConnectionId）以及每个查询的资源消耗情况，包括扫描数据大小（ScanBytes）、处理的行数（ProcessRows）、CPU 时间（CPUCostSeconds）、内存使用量（MemoryUsageBytes）和执行时间（ExecTime）。

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

2. 您可以通过指定查询 ID，进一步查看每个 BE 节点上查询的资源消耗。

   ```SQL
   SHOW PROC '/current_queries/<QueryId>/hosts';
   ```

   StarRocks 返回每个 BE 节点上查询的扫描数据大小（ScanBytes）、扫描行数（ScanRows）、CPU 时间（CPUCostSeconds）和内存使用量（MemUsageBytes）。

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

除了 MySQL 客户端之外，您还可以使用 FE 控制台进行可视化的交互式监控。

1. 通过以下 URL 在浏览器中访问 FE 控制台：

   ```Bash
   http://<fe_IP>:<fe_http_port>/system?path=//current_queries
   ```

   ![FE 控制台 1](../assets/console_1.png)

   您可以在**系统信息**页面查看当前正在处理的查询及其资源消耗情况。

2. 点击**QueryID**的查询。

   ![FE 控制台 2](../assets/console_2.png)

   在随后出现的页面上，您可以查看详细的、针对特定节点的资源消耗信息。

### 手动终止大型查询

如果任何大型查询绕过您设置的预防措施，威胁到系统的可用性，您可以使用对应的连接 ID 在[KILL](../sql-reference/sql-statements/Administration/KILL.md)语句中手动终止它们：

```SQL
KILL QUERY <ConnectionId>;
```

## 分析大型查询日志

从 v3.0 版本起，StarRocks 支持大型查询日志，这些日志存储在 **fe/log/fe.big_query.log** 文件中。与 StarRocks 的审计日志相比，大型查询日志额外记录了三个字段：

- bigQueryLogCPUSecondThreshold
- bigQueryLogScanBytesThreshold
- bigQueryLogScanRowsThreshold

这三个字段分别对应您定义的资源消耗阈值，用于判断查询是否属于大型查询。

要启用大型查询日志，请执行以下语句：

```SQL
SET GLOBAL enable_big_query_log = true;
```

启用大型查询日志后，您可以开始定义触发大型查询日志的规则。

- 设定触发大型查询日志的 CPU 时间阈值。

  以下示例将 CPU 时间阈值设为 600 秒：

  ```SQL
  SET GLOBAL big_query_log_cpu_second_threshold = 600;
  ```

- 设定触发大型查询日志的扫描数据大小阈值。

  以下示例将扫描数据大小阈值设为 10737418240 字节（10 GB）：

  ```SQL
  SET GLOBAL big_query_log_scan_bytes_threshold = 10737418240;
  ```

- 设定触发大型查询日志的扫描行数阈值。

  以下示例将扫描行数阈值设为 1500000000：

  ```SQL
  SET GLOBAL big_query_log_scan_rows_threshold = 1500000000;
  ```

## 微调预防措施

通过分析实时监控和大型查询日志中的统计数据，您可以了解集群中被遗漏的大型查询（或被误判为大型查询的常规查询）的模式，然后针对资源组和查询队列的设置进行优化。

如果大量的大型查询符合某个特定的 SQL 模式，并且您希望永久禁止该 SQL 模式，您可以将其添加到 SQL **黑名单**中。StarRocks 会拒绝所有与 SQL **黑名单**中指定模式匹配的查询，并返回错误信息。更多详情，请参见[管理 SQL 黑名单](../administration/Blacklist.md)部分。

要启用 SQL 黑名单，请执行以下语句：

```SQL
ADMIN SET FRONTEND CONFIG ("enable_sql_blacklist" = "true");
```

然后，您可以使用[ADD SQLBLACKLIST](../sql-reference/sql-statements/Administration/ADD_SQLBLACKLIST.md)命令将代表 SQL 模式的正则表达式添加到 SQL 黑名单中。

以下示例将 COUNT(DISTINCT) 添加到 SQL 黑名单：

```SQL
ADD SQLBLACKLIST "SELECT COUNT(DISTINCT .+) FROM .+";
```
