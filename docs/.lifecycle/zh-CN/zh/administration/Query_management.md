---
displayed_sidebar: "中文"
---

# 管理查询

本文介绍如何管理查询。

## 管理用户连接数

Property 是针对用户粒度的属性设置项。通过设置用户属性，可以分配给用户的资源等。这里所指的用户属性是指针对 user 而不是 user_identity 的属性。

您可以通过以下命令管理特定用户的客户端到 FE 的最大连接数。

```sql
SET PROPERTY [FOR 'user'] 'key' = 'value'[, ...];
```

以下例子将用户 jack 的最大连接数修改为 1000。

```sql
SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';
```

您可以通过以下命令查看特定用户的连接数限制。

```sql
SHOW PROPERTY FOR 'user';
```

## 设置与查询相关的 Session 变量

您可以设置与查询相关的 Session 级别变量，以此调整当前 Session 查询的并发度、内存等。

### 调整查询并发度

如果需要调整查询并发度，建议修改与 Pipeline 执行引擎相关的变量。

> **说明**
>
> - 自2.2版本起，正式推出了 Pipeline 引擎。
> - 自3.0版本起，支持根据查询并发度自动调整 `pipeline_dop`。

```sql
SET enable_pipeline_engine = true;
SET pipeline_dop = 0;
```

| 参数                               | 说明                                                         |
| -------------------------------- | ----------------------------------------------------------- |
| enable_pipeline_engine           | 是否启用 Pipeline 执行引擎。true：启用（默认项），false：禁用。     |
| pipeline_dop                     | Pipeline 实例的并行数。推荐设置默认值0，即系统将自动调整每个 pipeline 的并行度。您也可以将其设置为大于0的数值，通常为 BE 节点 CPU 物理核心数的一半。 |

您还可以通过设置实例的并行数量来调整查询并发度。

```sql
SET GLOBAL parallel_fragment_exec_instance_num = INT;
```

`parallel_fragment_exec_instance_num`：Fragment 实例的并行数量。一个 Fragment 实例占用一个 BE 节点上的一个 CPU，因此一个查询的并行度就是 Fragment 实例的并行数量。如果您想提高一个查询的并行度，可以将此参数设置为 BE CPU 核心数的一半。

> 注意：
>
> - 实际应用中，Fragment 实例的并行数量有上限，等于一张表在单个 BE 中的 Tablet 数量。举例来说，如果一张表有3个分区和32个分桶，在4个 BE 节点上，则单个 BE 节点的 Tablet 数为 32 * 3 / 4 = 24。因此，这个 BE 节点上的 Fragment 实例最大并行数量为24。这种情况下，即使将该参数设置为 `32`，实际应用的并行数仍然为24。
> - 在高并发场景下，CPU 资源通常已充分利用，因此建议将 Fragment 实例的并行数量设置为 `1`，减少不同查询间的资源竞争，从而提高整体查询效率。

### 调整查询内存上限

您可以通过以下命令来调整查询的内存上限。

```sql
SET query_mem_limit = INT;
```

`query_mem_limit`：单个查询的内存限制，单位是 Byte。建议设置为 17179869184（16GB）以上。

## 调整数据库存储容量 Quota

在默认设置下，每个数据库的存储容量是无限制的。您可以通过以下命令来进行调整。

```sql
ALTER DATABASE db_name SET DATA QUOTA quota;
```

> 说明：`quota` 的单位可以是 `B`，`K`，`KB`，`M`，`MB`，`G`，`GB`，`T`，`TB`，`P`，或 `PB`。

示例：

```sql
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

更多详细信息，请参考 [ALTER DATABASE](../sql-reference/sql-statements/data-definition/ALTER_DATABASE.md)

## 停止查询

您可以通过以下命令停止某个连接上的查询。

```sql
KILL connection_id;
```

`connection_id`：特定连接的 ID。您可以通过 `SHOW processlist;` 或者 `select connection_id();` 来查看。

示例：

```plain text
mysql> show processlist;
+------+--------------+---------------------+-----------------+-------------------+---------+------+-------+------+
| Id   | User         | Host                | Cluster         | Db                | Command | Time | State | Info |
+------+--------------+---------------------+-----------------+-------------------+---------+------+-------+------+
|    1 | starrocksmgr | 172.26.34.147:56208 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  129 | root         | 172.26.92.139:54818 | default_cluster |                   | Query   |    0 |       |      |
|  114 | test         | 172.26.34.147:57974 | default_cluster | ssb_100g          | Query   |    3 |       |      |
|    3 | starrocksmgr | 172.26.34.147:57268 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  100 | root         | 172.26.34.147:58472 | default_cluster | ssb_100           | Sleep   |  637 |       |      |
|  117 | starrocksmgr | 172.26.34.147:33790 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|    6 | starrocksmgr | 172.26.34.147:57632 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  119 | starrocksmgr | 172.26.34.147:33804 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  111 | root         | 172.26.92.139:55472 | default_cluster |                   | Sleep   | 2758 |       |      |
+------+--------------+---------------------+-----------------+-------------------+---------+------+-------+------+
9 rows in set (0.00 sec)

mysql> select connection_id();
+-----------------+
| CONNECTION_ID() |
+-----------------+
|              98 |
+-----------------+


mysql> kill 114;
Query OK, 0 rows affected (0.02 sec)
```

> 说明
>
> `Info` 栏展示对应的 SQL 语句。如果由于 SQL 语句较长而被截断，您可以使用 `SHOW FULL processlist;` 来查看完整的 SQL 语句。