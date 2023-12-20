---
displayed_sidebar: English
---

# 查询管理

## 用户连接数量

该属性是针对用户粒度进行设置的。要设置客户端与前端（FE）之间的最大连接数，请使用以下命令。

```sql
SET PROPERTY [FOR 'user'] 'key' = 'value' [, 'key' = 'value']
```

用户属性包括分配给用户的资源。这里所设置的属性是针对用户本身，而不是针对 user_identity。也就是说，如果通过 CREATE USER 语句创建了两个用户 jack'@'%' 和 jack'@'192.%'，那么 SET PROPERTY 语句可以作用于用户 jack，而不能作用于 jack'@'%' 或 jack'@'192.%'。

示例1：

```sql
For the user `jack`, change the maximum number of connections to 1000
SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';

Check the connection limit for the root user
SHOW PROPERTY FOR 'root'; 
```

## 查询相关的会话变量

会话变量可以通过 '键' = '值' 的方式设置，能够限制当前会话中的并发数、内存以及其他查询参数。例如：

- parallel_fragment_exec_instance_num

  查询的并行度，默认值为1。它指每个后端（BE）上的分片实例数量。您可以将此值设置为 BE 的 CPU 核心数的一半，以提升查询性能。

- query_mem_limit

  查询的内存限制，当查询报告内存不足时，可以进行调整。

- load_mem_limit

  数据导入的内存限制，当数据导入作业报告内存不足时，可以进行调整。

示例2：

```sql
set parallel_fragment_exec_instance_num  = 8; 
set query_mem_limit  = 137438953472;
```

## 数据库存储容量配额

数据库存储的容量配额默认是无限制的。您可以通过使用 alter database 命令来更改配额值。

```sql
ALTER DATABASE db_name SET DATA QUOTA quota;
```

配额单位有：B/K/KB/M/MB/G/GB/T/TB/P/PB

示例3：

```sql
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## 终止查询

要终止特定连接上的查询，可以使用以下命令：

```sql
kill connection_id;
```

可以通过 show processlist 查看 connection_id；或者使用 select connection_id(); 获取。

```plain
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
