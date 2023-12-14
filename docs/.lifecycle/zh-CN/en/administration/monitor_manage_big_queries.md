SET GLOBAL enable_pipeline_engine = true;
```


以下示例创建了一个名为 `bigQuery` 的资源组，将 CPU 时间上限设定为 `100` 秒，将扫描行数上限设定为 `100000`，将内存使用量上限设定为 `1073741824` 字节（1 GB）：

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

如果查询所需资源超过任何上限，查询将不会被执行，并返回错误。以下示例显示了当一个查询需要过多的扫描行数时返回的错误消息：

```Plain

ERROR 1064 (HY000): exceed big query scan_rows limit: current is 4 but limit is 1

```

如果这是您首次设置资源组，我们建议将上限设置相对较高，以免阻碍常规查询。在更好地了解大查询模式之后，您可以微调这些上限。

### 通过查询队列缓解系统超载

查询队列旨在在集群资源占用超过预设阈值时缓解系统超载恶化。您可以为最大并发数、内存使用量和 CPU 使用量设置阈值。当达到这些阈值中的任意一个时，StarRocks 将自动将即将到来的查询排队。挂起的查询要么在队列中等待执行，要么在达到预设资源阈值时被取消。更多信息，请参见[查询队列](../administration/query_queues.md)。

执行以下语句，以为 SELECT 查询启用查询队列：

```SQL
SET GLOBAL enable_query_queue_select = true;

```

启用查询队列功能之后，您可以定义触发查询队列的规则。

- 指定触发查询队列的并发阈值。

  以下示例将并发阈值设置为 `100`：

  ```SQL

  SET GLOBAL query_queue_concurrency_limit = 100;

  ```

- 指定触发查询队列的内存使用量比例阈值。

  以下示例将内存使用量比例阈值设置为 `0.9`：

  ```SQL

  SET GLOBAL query_queue_mem_used_pct_limit = 0.9;

  ```

- 指定触发查询队列的 CPU 使用量比例阈值。

  以下示例将 CPU 使用量千分比（CPU 使用量 * 1000）阈值设置为 `800`：

  ```SQL

  SET GLOBAL query_queue_cpu_used_permille_limit = 800;

  ```

您还可以通过配置最大队列长度和队列中每个挂起查询的超时时长来决定如何处理这些排队的查询。

- 指定最大查询队列长度。当达到此阈值时，将拒绝即将到来的查询。

  以下示例将查询队列长度设置为 `100`：

  ```SQL
  SET GLOBAL query_queue_max_queued_queries = 100;
  ```

- 指定队列中挂起查询的最大超时时长。当达到此阈值时，相应查询将被拒绝。

  以下示例将最大超时时长设置为 `480` 秒：

  ```SQL

  SET GLOBAL query_queue_pending_timeout_second = 480;
  ```


您可以使用[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md)来检查是否有查询挂起。

```Plain

mysql> SHOW PROCESSLIST;
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
| Id   | User | Host                | Db    | Command | ConnectionStartTime | Time | State | Info              | IsPending |

+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+

|    2 | root | xxx.xx.xxx.xx:xxxxx |       | Query   | 2022-11-24 18:08:29 |    0 | OK    | SHOW PROCESSLIST  | false     |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
```

如果 `IsPending` 为 `true`，则相应查询在查询队列中挂起。

## 实时监控大查询


从 v3.0 开始，StarRocks 支持查看当前在集群中处理的查询以及它们占用的资源。这使您能够在大查询绕过预防措施并导致意外系统超载的情况下监控集群。

### 通过 MySQL 客户端监控

1. 您可以使用[SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md)查看当前处理的查询（`current_queries`）。

   ```SQL

   SHOW PROC '/current_queries';

   ```

   StarRocks 返回每个查询的查询 ID（`QueryId`）、连接 ID（`ConnectionId`）以及查询的资源消耗，包括扫描数据大小（`ScanBytes`）、处理行数（`ProcessRows`）、CPU 时间（`CPUCostSeconds`）、内存使用量（`MemoryUsageBytes`）和执行时间（`ExecTime`）。

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


2. 您可以通过指定查询 ID 来进一步查看每个 BE 节点上的查询资源消耗。

   ```SQL
   SHOW PROC '/current_queries/<QueryId>/hosts';
   ```

   StarRocks 返回每个 BE 节点上的查询扫描数据大小（`ScanBytes`）、扫描行数（`ScanRows`）、CPU 时间（`CPUCostSeconds`）和内存使用量（`MemUsageBytes`）。

   ```Plain
   mysql> show proc '/current_queries/7c56495f-ae8b-11ed-8ebf-00163e00accc/hosts';
   +--------------------+-----------+-------------+----------------+---------------+
   | Host               | ScanBytes | ScanRows    | CpuCostSeconds | MemUsageBytes |
   +--------------------+-----------+-------------+----------------+---------------+
   | 172.26.34.185:8060 | 11.61 MB  | 356252 Rows | 52.93 Seconds  | 51.14 MB      |
   | 172.26.34.186:8060 | 14.66 MB  | 362646 Rows | 52.89 Seconds  | 50.44 MB      |
   | 172.26.34.187:8060 | 11.60 MB  | 356871 Rows | 52.91 Seconds  | 48.95 MB      |
   +--------------------+-----------+-------------+----------------+---------------+
   3 行记录(0.00 秒)


### 通过 FE 控制台监控

除了 MySQL 客户端，您还可以使用 FE 控制台进行可视化、交互式监控。

1. 使用以下 URL 在浏览器中导航至 FE 控制台：

   ```Bash
   http://<fe_IP>:<fe_http_port>/system?path=//current_queries
   ```

   ![FE 控制台1](../assets/console_1.png)

   您可以在**系统信息**页面查看当前正在处理的查询及其资源消耗情况。

2. 单击查询的**QueryID**。

   ![FE 控制台2](../assets/console_2.png)

   您可以在打开的页面上查看详细的、特定于节点的资源消耗信息。

### 手动终止大查询

如果有任何大查询绕过了您设置的预防措施，威胁到系统的可用性，您可以使用相应的连接 ID 在 [KILL](../sql-reference/sql-statements/Administration/KILL.md) 语句中手动终止它们：

```SQL
KILL QUERY <连接ID>;
```

## 分析大查询日志

从 v3.0 开始，StarRocks 支持大查询日志，存储在文件 **fe/log/fe.big_query.log** 中。相比于 StarRocks 审计日志，大查询日志额外打印了三个字段：

- `bigQueryLogCPUSecondThreshold`
- `bigQueryLogScanBytesThreshold`
- `bigQueryLogScanRowsThreshold`

这三个字段对应您定义的资源消耗阈值，用于确定查询是否为大查询。

要启用大查询日志，请执行以下语句：

```SQL
SET GLOBAL enable_big_query_log = true;
```

启用了大查询日志之后，您可以定义触发大查询日志的规则。

- 指定触发大查询日志的 CPU 时间阈值。

  以下示例将 CPU 时间阈值设置为 `600` 秒：

  ```SQL
  SET GLOBAL big_query_log_cpu_second_threshold = 600;
  ```

- 指定触发大查询日志的扫描数据大小阈值。

  以下示例将扫描数据大小阈值设置为 `10737418240` 字节（10 GB）：

  ```SQL
  SET GLOBAL big_query_log_scan_bytes_threshold = 10737418240;
  ```
  
- 指定触发大查询日志的扫描行数阈值。

  以下示例将扫描行数阈值设置为 `1500000000`：

  ```SQL
  SET GLOBAL big_query_log_scan_rows_threshold = 1500000000;
  ```

## 调整预防措施

通过实时监控和大查询日志获得的统计数据，您可以研究集群中被忽略的大查询（或者误判为大查询的常规查询）的模式，然后优化资源组和查询队列的设置。

如果显著比例的大查询符合某种 SQL 模式，并且您希望永久地禁止此 SQL 模式，您可以将此模式添加到 SQL 黑名单。StarRocks 拒绝与 SQL 黑名单中指定的任何模式匹配的所有查询，并返回错误。有关更多信息，请参阅 [管理 SQL 黑名单](../administration/Blacklist.md)。

要启用 SQL 黑名单，请执行以下语句：

```SQL
ADMIN SET FRONTEND CONFIG ("enable_sql_blacklist" = "true");
```

然后，可以使用 [ADD SQLBLACKLIST](../sql-reference/sql-statements/Administration/ADD_SQLBLACKLIST.md) 添加表示 SQL 模式的正则表达式到 SQL 黑名单。

以下示例将 `COUNT(DISTINCT)` 添加到 SQL 黑名单：

```SQL
ADD SQLBLACKLIST "SELECT COUNT(DISTINCT .+) FROM .+";
```