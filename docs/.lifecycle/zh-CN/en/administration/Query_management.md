---
displayed_sidebar: "Chinese"
---

# 查询管理

## 用户连接数

`Property` 用于用户粒度。要设置客户端和前端之间的最大连接数，请使用以下命令。

```sql
SET PROPERTY [FOR 'user'] 'key' = 'value' [, 'key' = 'value']
```

用户属性包括分配给用户的资源。此处设置的属性为用户的，而不是 `user_identity` 的。也就是说，如果使用 `CREATE USER` 语句创建了两个用户 `jack'@'％` 和 `jack'@'192.%`，则 `SET PROPERTY` 语句可用于用户 `jack`，而不是 `jack'@'％` 或 `jack'@'192.%`。

示例 1:

```sql
对于用户 `jack`，将最大连接数更改为 1000
SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';

检查根用户的连接限制
SHOW PROPERTY FOR 'root'; 
```

## 与查询相关的会话变量

会话变量可以通过 'key' = 'value' 设置，可以限制当前会话中的并发性、内存和其他查询参数。例如：

- parallel_fragment_exec_instance_num

  查询的并行度，默认值为 1。它表示每个 BE 上的片段实例数。您可以将其设置为 BE 的 CPU 核心数的一半，以提高查询性能。

- query_mem_limit

  查询的内存限制，在查询报告内存不足时可以进行调整。

- load_mem_limit

  导入的内存限制，在导入作业报告内存不足时可以进行调整。

示例 2:

```sql
set parallel_fragment_exec_instance_num = 8; 
set query_mem_limit = 137438953472;
```

## 数据库存储容量配额

数据库存储容量配额默认为无限。您可以使用 `alter database` 来更改配额值。

```sql
ALTER DATABASE db_name SET DATA QUOTA quota;
```

配额单位: B/K/KB/M/MB/G/GB/T/TB/P/PB

示例 3:

```sql
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## 终止查询

要终止特定连接上的查询，请使用以下命令：

```sql
kill connection_id;
```

`connection_id` 可以通过 `show processlist;` 或 `select connection_id();` 查看。

```plain text
 show processlist;
+------+------------+---------------------+-----------------+---------------+---------+------+-------+------+
| Id   | User       | Host                | Cluster         | Db            | Command | Time | State | Info |
+------+------------+---------------------+-----------------+---------------+---------+------+-------+------+
|    1 | starrocksmgr | 172.26.34.147:56208 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  129 | root       | 172.26.92.139:54818 | default_cluster |               | Query   |    0 |       |      |
|  114 | test       | 172.26.34.147:57974 | default_cluster | ssb_100g      | Query   |    3 |       |      |
|    3 | starrocksmgr | 172.26.34.147:57268 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  100 | root       | 172.26.34.147:58472 | default_cluster | ssb_100       | Sleep   |  637 |       |      |
|  117 | starrocksmgr | 172.26.34.147:33790 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|    6 | starrocksmgr | 172.26.34.147:57632 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  119 | starrocksmgr | 172.26.34.147:33804 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  111 | root       | 172.26.92.139:55472 | default_cluster |               | Sleep   | 2758 |       |      |
+------+------------+---------------------+-----------------+---------------+---------+------+-------+------+
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