---
displayed_sidebar: "中文"
---

# SHOW PROC

## 描述

显示StarRocks集群的某些指标。

## 语法

```SQL
SHOW PROC { '/backends' | '/compute_nodes' | '/dbs' 
          | '/jobs' | '/statistic' | '/tasks' | '/frontends' 
          | '/brokers' | '/resources' | '/load_error_hub' 
          | '/transactions' | '/monitor' | '/current_queries' 
          | '/current_backend_instances' | '/cluster_balance' 
          | '/routine_loads' | '/colocation_group' | '/catalog' }
```

## 参数

| **参数**                | **描述**              |
| ---------------------------- | ------------------------------------------------------------ |
| '/backends'                  | 显示集群中BE节点的信息。           |
| '/compute_nodes'             | 显示集群中CN节点的信息。           |
| '/dbs'                       | 显示集群中数据库的信息。           |
| '/jobs'                      | 显示集群中作业的信息。             |
| '/statistic'                 | 显示集群中每个数据库的统计信息。    |
| '/tasks'                     | 显示集群中所有通用任务的总数以及失败的任务。 |
| '/frontends'                 | 显示集群中FE节点的信息。           |
| '/brokers'                   | 显示集群中Broker节点的信息。       |
| '/resources'                 | 显示集群中资源的信息。             |
| '/load_error_hub'            | 显示集群的加载错误中心的配置，用于管理加载作业的错误消息。 |
| '/transactions'              | 显示集群中事务的信息。             |
| '/monitor'                   | 显示集群中的监控信息。             |
| '/current_queries'           | 显示当前FE节点上运行的查询信息。   |
| '/current_backend_instances' | 显示在集群中处理请求的BE节点。     |
| '/cluster_balance'           | 显示集群中的负载平衡信息。         |
| '/routine_loads'             | 显示集群中例行负载的信息。         |
| '/colocation_group'          | 显示集群中Colocate Join组的信息。  |
| '/catalog'                   | 显示集群中目录的信息。             |

## 示例

示例1: 显示集群中BE节点的信息。

```Plain
mysql> SHOW PROC '/backends'\G
*************************** 1. row ***************************
            BackendId: 10004
                   IP: xxx.xx.92.200
        HeartbeatPort: 9354
               BePort: 9360
             HttpPort: 8338
             BrpcPort: 8360
        LastStartTime: 2023-04-21 09:56:10
        LastHeartbeat: 2023-04-21 09:56:10
                Alive: true
 SystemDecommissioned: false
ClusterDecommissioned: false
            TabletNum: 2199
     DataUsedCapacity: 0.000 
        AvailCapacity: 584.578 GB
        TotalCapacity: 1.968 TB
              UsedPct: 71.00 %
       MaxDiskUsedPct: 71.00 %
               ErrMsg: 
              Version: BRANCH-3.0-RELEASE-8eb8705
               Status: {"lastSuccessReportTabletsTime":"N/A"}
    DataTotalCapacity: 584.578 GB
          DataUsedPct: 0.00 %
             CpuCores: 16
    NumRunningQueries: 0
           MemUsedPct: 0.52 %
           CpuUsedPct: 0.0 %
```

| **返回**            | **描述**              |
| --------------------- | ------------------------------------------------------------ |
| BackendId             | BE节点的ID。                                           |
| IP                    | BE节点的IP地址。                                      |
| HeartbeatPort         | BE节点的心跳服务端口。                                 |
| BePort                | BE节点的Thrift服务器端口。                             |
| HttpPort              | BE节点的HTTP服务器端口。                               |
| BrpcPort              | BE节点的bRPC端口。                                    |
| LastStartTime         | BE节点上次启动的时间。                                 |
| LastHeartbeat         | BE节点上次收到心跳的时间。                             |
| Alive                 | BE节点是否活跃。                                       |
| SystemDecommissioned  | BE节点是否已停止使用。                                 |
| ClusterDecommissioned | BE节点是否在集群中已停止使用。                         |
| TabletNum             | BE节点中的Tablet数。                                   |
| DataUsedCapacity      | BE节点中用于数据的存储容量。                           |
| AvailCapacity         | BE节点中可用的存储容量。                               |
| TotalCapacity         | BE节点中总的存储容量。                                 |
| UsedPct               | BE节点存储容量使用百分比。                             |
| MaxDiskUsedPct        | BE节点存储容量的最大使用百分比。                       |
| ErrMsg                | BE节点中的错误消息。                                   |
| Version               | BE节点的StarRocks版本。                                |
| Status                | BE节点的状态信息，包括BE节点报告Tablets的最后时间。    |
| DataTotalCapacity     | 使用和可用数据存储容量的总和。`DataUsedCapacity`和`AvailCapacity`的总和。 |
| DataUsedPct           | 数据存储占总数据容量的百分比（DataUsedCapacity/DataTotalCapacity）。 |
| CpuCores              | BE节点中的CPU核心数。                                  |
| NumRunningQueries     | 当前集群中正在运行的查询数。                           |
| MemUsedPct            | 当前内存使用百分比。                                          |
| CpuUsedPct            | 当前CPU使用百分比。              |

示例2: 显示集群中数据库的信息。

```Plain
mysql> SHOW PROC '/dbs';
+---------+------------------------+----------+----------------+--------------------------+---------------------+
| DbId    | DbName                 | TableNum | Quota          | LastConsistencyCheckTime | ReplicaQuota        |
+---------+------------------------+----------+----------------+--------------------------+---------------------+
| 1       | information_schema     | 22       | 8388608.000 TB | NULL                     | 9223372036854775807 |
| 840997  | tpcds_100g             | 25       | 1024.000 GB    | NULL                     | 1073741824          |
| 1275196 | _statistics_           | 3        | 8388608.000 TB | 2022-09-06 23:00:58      | 9223372036854775807 |
| 1286207 | tpcds_n                | 24       | 8388608.000 TB | NULL                     | 9223372036854775807 |
| 1381289 | test                   | 6        | 8388608.000 TB | 2022-01-14 23:10:18      | 9223372036854775807 |
| 6186781 | test_stddev            | 1        | 8388608.000 TB | 2022-09-06 23:00:58      | 9223372036854775807 |
+---------+------------------------+----------+----------------+--------------------------+---------------------+
```

| **返回**               | **描述**                                   |
| ------------------------ | ---------------------------------- |
| DbId                     | 数据库ID。                                      |
| DbName                   | 数据库名称。                                    |
| TableNum                 | 数据库中的表数量。                             |
| Quota                    | 数据库的存储配额。                             |
| LastConsistencyCheckTime | 上次执行一致性检查的时间。                    |
| ReplicaQuota             | 数据库的数据副本配额。                         |

示例3: 显示集群中作业的信息。

```Plain
mysql> SHOW PROC '/jobs';
+-------+--------------------------------------+
| DbId  | DbName                               |
+-------+--------------------------------------+
| 10005 | default_cluster:_statistics_         |
| 0     | default_cluster:information_schema   |
| 12711 | default_cluster:starrocks_audit_db__ |
+-------+--------------------------------------+
3 rows in set (0.00 sec)

mysql> SHOW PROC '/jobs/10005';
+---------------+---------+---------+----------+-----------+-------+
| JobType       | Pending | Running | Finished | Cancelled | Total |
+---------------+---------+---------+----------+-----------+-------+
| load          | 0       | 0       | 3        | 0         | 3     |
| rollup        | 0       | 0       | 0        | 0         | 0     |
| schema_change | 0       | 0       | 0        | 0         | 0     |
| export        | 0       | 0       | 0        | 0         | 0     |
+---------------+---------+---------+----------+-----------+-------+
4 rows in set (0.00 sec)
```

| **返回** | **描述**                    |
| ---------- | ---------------------- |
| DbId       | Database ID.                       |
| DbName     | Database name.                     |
| JobType    | Job type.                          |
| Pending    | Number of jobs that are pending.   |
| Running    | Number of jobs that are running.   |
| Finished   | Number of jobs that are finished.  |
| Cancelled  | Number of jobs that are cancelled. |
| Total      | Total number of jobs.              |

```plaintext

mysql> SHOW PROC '/statistic';
+--------+----------------------------------------------------------+----------+--------------+----------+-----------+------------+--------------------+-----------------------+------------------+---------------------+
| DbId   | DbName                                                   | TableNum | PartitionNum | IndexNum | TabletNum | ReplicaNum | UnhealthyTabletNum | InconsistentTabletNum | CloningTabletNum | ErrorStateTabletNum |
+--------+----------------------------------------------------------+----------+--------------+----------+-----------+------------+--------------------+-----------------------+------------------+---------------------+
| 10004  | _statistics_                                             | 3        | 3            | 3        | 30        | 60         | 0                  | 0                     | 0                | 0                   |
| 1      | information_schema                                       | 0        | 0            | 0        | 0         | 0          | 0                  | 0                     | 0                | 0                   |
| 92498  | stream_load_test_db_03afc714_b1cb_11ed_a82c_00163e237e98 | 0        | 0            | 0        | 0         | 0          | 0                  | 0                     | 0                | 0                   |
| 92542  | stream_load_test_db_79876e92_b1da_11ed_b50e_00163e237e98 | 1        | 1            | 1        | 3         | 3          | 0                  | 0                     | 0                | 0                   |
| 115476 | testdb                                                   | 0        | 0            | 0        | 0         | 0          | 0                  | 0                     | 0                | 0                   |
| 10002  | zq_test                                                  | 8        | 8            | 8        | 5043      | 7063       | 0                  | 0                     | 0                | 2                   |
| Total  | 6                                                        | 12       | 12           | 12       | 5076      | 7126       | 0                  | 0                     | 0                | 2                   |
+--------+----------------------------------------------------------+----------+--------------+----------+-----------+------------+--------------------+-----------------------+------------------+---------------------+
7 rows in set (0.01 sec)


mysql> show proc '/statistic/10002';
+------------------+---------------------+----------------+-------------------+
| UnhealthyTablets | InconsistentTablets | CloningTablets | ErrorStateTablets |
+------------------+---------------------+----------------+-------------------+
| []               | []                  | []             | [116703, 116706]  |
+------------------+---------------------+----------------+-------------------+
```

| **Return**            | **Description**                                              |
| --------------------- | ------------------------------------------------------------ |
| DbId                  | Database ID.                                                 |
| DbName                | Database name.                                               |
| TableNum              | Number of tables in the database.                            |
| PartitionNum          | Number of partitions in the database.                        |
| IndexNum              | Number of indexes in the database.                           |
| TabletNum             | Number of tablets in the database.                           |
| ReplicaNum            | Number of replicas in the database.                          |
| UnhealthyTabletNum    | Number of unfinished (unhealthy) tablets in the database during data redistribution. |
| InconsistentTabletNum | Number of inconsistent tablets in the database.              |
| CloningTabletNum      | Number of tablets that are being cloned in the database.     |
| ErrorStateTabletNum   | In a Primary Key type table, the number of tablets in Error state. |
| ErrorStateTablets     | In a Primary Key type table, the IDs of the tablets in Error state. |

Example 5: Shows the total number of all generic tasks and the failed tasks in the cluster.

```plaintext
mysql> SHOW PROC '/tasks';
+-------------------------+-----------+----------+
| TaskType                | FailedNum | TotalNum |
+-------------------------+-----------+----------+
| CREATE                  | 0         | 0        |
| DROP                    | 0         | 0        |
| PUSH                    | 0         | 0        |
| CLONE                   | 0         | 0        |
| STORAGE_MEDIUM_MIGRATE  | 0         | 0        |
| ROLLUP                  | 0         | 0        |
| SCHEMA_CHANGE           | 0         | 0        |
| CANCEL_DELETE           | 0         | 0        |
| MAKE_SNAPSHOT           | 0         | 0        |
| RELEASE_SNAPSHOT        | 0         | 0        |
| CHECK_CONSISTENCY       | 0         | 0        |
| UPLOAD                  | 0         | 0        |
| DOWNLOAD                | 0         | 0        |
| CLEAR_REMOTE_FILE       | 0         | 0        |
| MOVE                    | 0         | 0        |
| REALTIME_PUSH           | 0         | 0        |
| PUBLISH_VERSION         | 0         | 0        |
| CLEAR_ALTER_TASK        | 0         | 0        |
| CLEAR_TRANSACTION_TASK  | 0         | 0        |
| RECOVER_TABLET          | 0         | 0        |
| STREAM_LOAD             | 0         | 0        |
| UPDATE_TABLET_META_INFO | 0         | 0        |
| ALTER                   | 0         | 0        |
| INSTALL_PLUGIN          | 0         | 0        |
| UNINSTALL_PLUGIN        | 0         | 0        |
| NUM_TASK_TYPE           | 0         | 0        |
| Total                   | 0         | 0        |
+-------------------------+-----------+----------+
```

| **Return** | **Description**         |
| ---------- | ----------------------- |
| TaskType   | Task type.              |
| FailedNum  | Number of failed tasks. |
| TotalNum   | Total number of tasks.  |

Example 6: Shows the information of FE nodes in the cluster.

```plaintext
mysql> SHOW PROC '/frontends';
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
| Name                             | IP            | EditLogPort | HttpPort | QueryPort | RpcPort | Role     | ClusterId  | Join  | Alive | ReplayedJournalId | LastHeartbeat | IsHelper | ErrMsg        | StartTime | Version |
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
| xxx.xx.xx.xxx_9009_1600088918395 | xxx.xx.xx.xxx | 9009        | 7390     | 0         | 0       | FOLLOWER | 1747363037 | false | false | 0                 | NULL          | true     | got exception | NULL      | NULL    |
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
```

| **Return**        | **Description**                                        |
| ----------------- | ------------------------------------------------------ |
| Name              | FE node name.                                          |
| IP                | IP address of the FE node.                             |
| EditLogPort       | Port for communication between FE nodes.               |
| HttpPort          | HTTP Server port of the FE node.                       |
| QueryPort         | MySQL Server port of the FE node.                      |
| RpcPort           | RPC port of the FE node.                               |
| Role              | Role of the FE node (Leader, Follower, or Observer).   |
| ClusterId         | Cluster ID.                                            |
| Join              | If the FE node has joined a cluster.                   |
| Alive             | If the FE node is alive.                               |
| ReplayedJournalId | The largest metadata ID that the FE node has replayed. |
| LastHeartbeat     | The last time when the FE node sent a heartbeat.       |
| IsHelper          | If the FE node is the BDBJE helper node.               |
| ErrMsg            | Error messages in the FE node.                         |
| StartTime         | Time when the FE node is started.                      |
| Version           | StarRocks version of the FE node.                      |

Example 7: Shows the information of Broker nodes in the cluster.

```plaintext
mysql> SHOW PROC '/brokers';
+-------------+---------------+------+-------+---------------+---------------------+--------+
| Name        | IP            | Port | Alive | LastStartTime | LastUpdateTime      | ErrMsg |
+-------------+---------------+------+-------+---------------+---------------------+--------+
| hdfs_broker | xxx.xx.xx.xxx | 8500 | true  | NULL          | 2022-10-10 16:37:59 |        |
| hdfs_broker | xxx.xx.xx.xxx | 8500 | true  | NULL          | 2022-10-10 16:37:59 |        |
| hdfs_broker | xxx.xx.xx.xxx | 8500 | true  | NULL          | 2022-10-10 16:37:59 |        |
+-------------+---------------+------+-------+---------------+---------------------+--------+
```

```plain
| **返回**    | **描述**                 |
| ----------- | ------------------------ |
| Name        | Broker节点名称。         |
| IP          | Broker节点的IP地址。     |
| Port        | Broker节点的Thrift服务器端口。该端口用于接收请求。 |
| Alive       | Broker节点是否存活。     |
| LastStartTime | Broker节点最后启动时间。  |
| LastUpdateTime | Broker节点最后更新时间。  |

| ErrMsg      | Broker节点的错误消息。   |

示例8：显示集群中资源的信息。

```Plain
mysql> SHOW PROC '/resources';
+-------------------------+--------------+---------------------+------------------------------+
| Name                    | ResourceType | Key                 | Value                        |
+-------------------------+--------------+---------------------+------------------------------+
| hive_resource_stability | hive         | hive.metastore.uris | thrift://xxx.xx.xxx.xxx:9083 |
| hive2                   | hive         | hive.metastore.uris | thrift://xxx.xx.xx.xxx:9083  |

+-------------------------+--------------+---------------------+------------------------------+
```

| **返回**    | **描述**    |
| ----------- | ---------- |
| Name        | 资源名称。  |

| ResourceType | 资源类型。  |

| Key         | 资源键。   |
| Value      | 资源值。    |

示例9：显示集群中事务的信息。

```Plain
mysql> SHOW PROC '/transactions';
+-------+--------------------------------------+
| DbId  | DbName                               |

+-------+--------------------------------------+
| 10005 | default_cluster:_statistics_         |
| 12711 | default_cluster:starrocks_audit_db__ |
+-------+--------------------------------------+
2 rows in set (0.00 sec)

mysql> SHOW PROC '/transactions/10005';
+----------+--------+
| State    | Number |

+----------+--------+
| running  | 0      |
| finished | 4      |
+----------+--------+
2 rows in set (0.00 sec)
```

| **返回**  | **描述**                |

| --------- | ---------------------- |
| DbId      | 数据库ID。              |
| DbName    | 数据库名称。            |
| State     | 事务的状态。           |
| Number    | 事务数量。             |

示例10：显示集群中的监控信息。


```Plain
mysql> SHOW PROC '/monitor';
+------+------+
| Name | Info |

+------+------+

| jvm  |      |
+------+------+
```

| **返回** | **描述**  |
| ------- | ---------- |
| Name    | JVM名称。   |
| Info    | JVM信息。   |

示例11：显示集群中的负载均衡信息。

```Plain
mysql> SHOW PROC '/cluster_balance';
+-------------------+--------+

| Item              | Number |
+-------------------+--------+
| cluster_load_stat | 1      |
| working_slots     | 3      |

| sched_stat        | 1      |

| priority_repair   | 0      |
| pending_tablets   | 2001   |
| running_tablets   | 0      |
| history_tablets   | 1000   |
+-------------------+--------+
```

| **返回** | **描述**                                |
| ------- | -------------------------------------- |
| Item    | `cluster_balance`中的子命令项。         |
| Number  | `cluster_balance`中每个子命令的数量。   |

示例12：显示集群中Colocate Join组的信息。

```Plain
mysql> SHOW PROC '/colocation_group';

+-----------------+----------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------+------------+----------------+-------------+----------+
| GroupId         | GroupName                  | TableIds                                                                                                                                          | BucketsNum | ReplicationNum | DistCols    | IsStable |
+-----------------+----------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------+------------+----------------+-------------+----------+
| 24010.177354    | 24010_lineitem_str_g1      | 177672                                                                                                                                            | 12         | 1              | varchar(-1) | true     |
| 24010.182146    | 24010_lineitem_str_g2      | 182144                                                                                                                                            | 192        | 1              | varchar(-1) | true     |
| 1439318.1735496 | 1439318_group_agent_uid    | 1735677, 1738390                                                                                                                                  | 12         | 2              | bigint(20)  | true     |
| 24010.37804     | 24010_gsdaf2449s9e         | 37802                                                                                                                                             | 192        | 1              | int(11)     | true     |
| 174844.175370   | 174844_groupa4             | 175368, 591307, 591362, 591389, 591416                                                                                                            | 12         | 1              | int(11)     | true     |
| 24010.30587     | 24010_group2               | 30585, 30669                                                                                                                                      | 12         | 1              | int(11)     | true     |

| 10005.181366    | 10005_lineorder_str_normal | 181364                                                                                                                                            | 192        | 1              | varchar(-1) | true     |

| 1904968.5973175 | 1904968_groupa2            | 5973173                                                                                                                                           | 12         | 1              | int(11)     | true     |
| 24010.182535    | 24010_lineitem_str_g3      | 182533                                                                                                                                            | 192        | 1              | varchar(-1) | true     |
+-----------------+----------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------+------------+----------------+-------------+----------+
```

| **返回**    | **描述**                            |
| ----------- | ---------------------------------- |
| GroupId     | Colocate Join组ID。                 |
| GroupName   | Colocate Join组名称。               |
| TableIds    | Colocate Join组中的表ID。           |

| BucketsNum | Colocate Join组中的buckets数量。    |
| ReplicationNum | Colocate Join组中的复制数量。     |
| DistCols    | Colocate Join组的分布列。          |
| IsStable   | Colocate Join组是否稳定。          |
