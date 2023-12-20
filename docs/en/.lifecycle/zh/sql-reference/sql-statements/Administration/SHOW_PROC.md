---
displayed_sidebar: English
---

# SHOW PROC

## 描述

显示 StarRocks 集群的某些指标。

:::tip

此操作需要 SYSTEM 级别的 OPERATE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。

:::

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

|**参数**|**描述**|
|---|---|
|'/backends'|显示集群中 BE 节点的信息。|
|'/compute_nodes'|显示集群中 CN 节点的信息。|
|'/dbs'|显示集群中数据库的信息。|
|'/jobs'|显示集群中作业的信息。|
|'/statistic'|显示集群中每个数据库的统计信息。|
|'/tasks'|显示集群中所有通用任务和失败任务的总数。|
|'/frontends'|显示集群中 FE 节点的信息。|
|'/brokers'|显示集群中 Broker 节点的信息。|
|'/resources'|显示集群中资源的信息。|
|'/load_error_hub'|显示集群的 Load Error Hub 配置，用于管理加载作业的错误消息。|
|'/transactions'|显示集群中事务的信息。|
|'/monitor'|显示集群中的监控信息。|
|'/current_queries'|显示当前 FE 节点上运行的查询信息。|
|'/current_backend_instances'|显示集群中正在处理请求的 BE 节点。|
|'/cluster_balance'|显示集群中的负载均衡信息。|
|'/routine_loads'|显示集群中 Routine Load 的信息。|
|'/colocation_group'|显示集群中 Colocate Join 组的信息。|
|'/catalog'|显示集群中目录的信息。|

## 示例

示例 1：显示集群中 BE 节点的信息。

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

|**返回**|**描述**|
|---|---|
|BackendId|BE 节点的 ID。|
|IP|BE 节点的 IP 地址。|
|HeartbeatPort|BE 节点的心跳服务端口。|
|BePort|BE 节点的 Thrift Server 端口。|
|HttpPort|BE 节点的 HTTP Server 端口。|
|BrpcPort|BE 节点的 bRPC 端口。|
|LastStartTime|BE 节点最后一次启动的时间。|
|LastHeartbeat|BE 节点最后一次接收心跳的时间。|
|Alive|BE 节点是否存活。|
|SystemDecommissioned|BE 节点是否已经退役。|
|ClusterDecommissioned|BE 节点在集群中是否已经退役。|
|TabletNum|BE 节点中的平板数量。|
|DataUsedCapacity|BE 节点中用于数据的存储容量。|
|AvailCapacity|BE 节点中的可用存储容量。|
|TotalCapacity|BE 节点的总存储容量。|
|UsedPct|BE 节点存储容量的使用百分比。|
|MaxDiskUsedPct|BE 节点存储容量使用的最大百分比。|
|ErrMsg|BE 节点中的错误信息。|
|Version|BE 节点的 StarRocks 版本。|
|Status|BE 节点的状态信息，包括最后一次成功报告平板的时间。|
|DataTotalCapacity|已用和可用数据存储容量的总和。是 DataUsedCapacity 和 AvailCapacity 的总和。|
|DataUsedPct|数据存储占总数据容量的百分比（DataUsedCapacity / DataTotalCapacity）。|
|CpuCores|BE 节点中的 CPU 核心数。|
|NumRunningQueries|集群中当前正在运行的查询数量。|
|MemUsedPct|当前的内存使用百分比。|
|CpuUsedPct|当前的 CPU 使用百分比。|

示例 2：显示集群中数据库的信息。

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

|**返回**|**描述**|
|---|---|
|DbId|数据库的 ID。|
|DbName|数据库的名称。|
|TableNum|数据库中的表数量。|
|Quota|数据库的存储配额。|
|LastConsistencyCheckTime|上一次执行一致性检查的时间。|
|ReplicaQuota|数据库的数据副本配额。|

示例 3：显示集群中作业的信息。

```Plain
mysql> SHOW PROC '/jobs';
+-------+--------------------------------------+
| DbId  | DbName                               |
+-------+--------------------------------------+
| 10005 | default_cluster:_statistics_         |
| 0     | default_cluster:information_schema   |
| 12711 | default_cluster:starrocks_audit_db__ |
+-------+--------------------------------------+
3 行在集 (0.00 秒)

mysql> SHOW PROC '/jobs/10005';
+---------------+---------+---------+----------+-----------+-------+
| JobType       | Pending | Running | Finished | Cancelled | Total |
+---------------+---------+---------+----------+-----------+-------+
| load          | 0       | 0       | 3        | 0         | 3     |
| rollup        | 0       | 0       | 0        | 0         | 0     |
| schema_change | 0       | 0       | 0        | 0         | 0     |
| export        | 0       | 0       | 0        | 0         | 0     |
+---------------+---------+---------+----------+-----------+-------+
4 行在集 (0.00 秒)
```

|**返回**|**描述**|
|---|---|
|DbId|数据库的 ID。|
|DbName|数据库的名称。|
|JobType|作业类型。|
|Pending|待处理的作业数量。|
|Running|正在运行的作业数量。|
|Finished|已完成的作业数量。|
|Cancelled|已取消的作业数量。|
|Total|作业总数。|

示例 4：显示集群中每个数据库的统计信息。

```Plain
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
7 行在集 (0.01 秒)

mysql> show proc '/statistic/10002';
+------------------+---------------------+----------------+-------------------+
| UnhealthyTablets | InconsistentTablets | CloningTablets | ErrorStateTablets |
+------------------+---------------------+----------------+-------------------+
| []               | []                  | []             | [116703, 116706]  |
+------------------+---------------------+----------------+-------------------+
```

|**返回**|**描述**|
|---|---|
|DbId|数据库的 ID。|
|DbName|数据库的名称。|
|TableNum|数据库中的表数量。|
```
|PartitionNum|数据库中的分区数。|
|IndexNum|数据库中的索引数。|
|TabletNum|数据库中的Tablet数量。|
|ReplicaNum|数据库中的副本数。|
|UnhealthyTabletNum|数据重新分配期间数据库中未完成（不健康）的Tablet数量。|
|InconsistentTabletNum|数据库中不一致的Tablet数量。|
|CloningTabletNum|正在数据库中克隆的Tablet数量。|
|ErrorStateTabletNum|在主键类型表中，处于错误状态的Tablet数量。|
|ErrorStateTablets|在主键类型表中，处于错误状态的Tablet的ID。|

示例 5：显示集群中所有通用任务和失败任务的总数。

```Plain
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

|返回|说明|
|---|---|
|TaskType|任务类型。|
|FailedNum|失败任务的数量。|
|TotalNum|任务总数。|

示例 6：显示集群中FE节点的信息。

```Plain
mysql> SHOW PROC '/frontends';
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
| Name                             | IP            | EditLogPort | HttpPort | QueryPort | RpcPort | Role     | ClusterId  | Join  | Alive | ReplayedJournalId | LastHeartbeat | IsHelper | ErrMsg        | StartTime | Version |
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
| xxx.xx.xx.xxx_9009_1600088918395 | xxx.xx.xx.xxx | 9009        | 7390     | 0         | 0       | FOLLOWER | 1747363037 | false | false | 0                 | NULL          | true     | got exception | NULL      | NULL    |
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
```

|返回|说明|
|---|---|
|Name|FE节点名称。|
|IP|FE节点的IP地址。|
|EditLogPort|FE节点之间通信的端口。|
|HttpPort|FE节点的HTTP Server端口。|
|QueryPort|FE节点的MySQL Server端口。|
|RpcPort|FE节点的RPC端口。|
|Role|FE节点的角色（LEADER、FOLLOWER或OBSERVER）。|
|ClusterId|集群ID。|
|Join|FE节点是否已加入集群。|
|Alive|FE节点是否存活。|
|ReplayedJournalId|FE节点已重放的最大元数据ID。|
|LastHeartbeat|FE节点最后一次发送心跳的时间。|
|IsHelper|FE节点是否是BDBJE辅助节点。|
|ErrMsg|FE节点中的错误消息。|
|StartTime|FE节点启动时间。|
|Version|FE节点的StarRocks版本。|

示例 7：显示集群中Broker节点的信息。

```Plain
mysql> SHOW PROC '/brokers';
+-------------+---------------+------+-------+---------------+---------------------+--------+
| Name        | IP            | Port | Alive | LastStartTime | LastUpdateTime      | ErrMsg |
+-------------+---------------+------+-------+---------------+---------------------+--------+
| hdfs_broker | xxx.xx.xx.xxx | 8500 | true  | NULL          | 2022-10-10 16:37:59 |        |
| hdfs_broker | xxx.xx.xx.xxx | 8500 | true  | NULL          | 2022-10-10 16:37:59 |        |
| hdfs_broker | xxx.xx.xx.xxx | 8500 | true  | NULL          | 2022-10-10 16:37:59 |        |
+-------------+---------------+------+-------+---------------+---------------------+--------+
```

|返回|说明|
|---|---|
|Name|Broker节点名称。|
|IP|Broker节点的IP地址。|
|Port|Broker节点的Thrift Server端口，用于接收请求。|
|Alive|Broker节点是否存活。|
|LastStartTime|Broker节点最后一次启动的时间。|
|LastUpdateTime|Broker节点最后一次更新的时间。|
|ErrMsg|Broker节点中的错误消息。|

示例 8：显示集群中的资源信息。

```Plain
mysql> SHOW PROC '/resources';
+-------------------------+--------------+---------------------+------------------------------+
| Name                    | ResourceType | Key                 | Value                        |
+-------------------------+--------------+---------------------+------------------------------+
| hive_resource_stability | hive         | hive.metastore.uris | thrift://xxx.xx.xxx.xxx:9083 |
| hive2                   | hive         | hive.metastore.uris | thrift://xxx.xx.xx.xxx:9083  |
+-------------------------+--------------+---------------------+------------------------------+
```

|返回|说明|
|---|---|
|Name|资源名称。|
|ResourceType|资源类型。|
|Key|资源键。|
|Value|资源值。|

示例 9：显示集群中事务的信息。

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

|返回|说明|
|---|---|
|DbId|数据库ID。|
|DbName|数据库名称。|
|State|事务的状态。|
|Number|事务数量。|

示例 10：显示集群中的监控信息。

```Plain
mysql> SHOW PROC '/monitor';
+------+------+
| Name | Info |
+------+------+
| jvm  |      |
+------+------+
```

|返回|说明|
|---|---|
|Name|JVM名称。|
|Info|JVM信息。|

示例 11：显示集群中的负载均衡信息。

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

|返回|说明|
|---|---|
|Item|`cluster_balance`中的子命令项。|
|Number|`cluster_balance`中各子命令的数量。|

示例 12：显示集群中Colocate Join组的信息。

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

|返回|说明|
|---|---|
|GroupId|Colocate Join组ID。|
|GroupName|Colocate Join组名称。|
|TableIds|Colocate Join组中表的ID。|
|BucketsNum|Colocate Join组中的桶数。|
|ReplicationNum|Colocate Join组中的副本数。|
|DistCols|Colocate Join组的分布列。|
|IsStable|Colocate Join组是否稳定。|

示例 13：显示集群中的目录信息。

```Plain
mysql> SHOW PROC '/catalog';
+--------------------------------------------------------------+----------+----------------------+
| Catalog                                                      | Type     | Comment              |
+--------------------------------------------------------------+----------+----------------------+
| resource_mapping_inside_catalog_hive_hive2                   | hive     | mapping hive catalog |
| resource_mapping_inside_catalog_hive_hive_resource_stability | hive     | mapping hive catalog |
| default_catalog                                              | Internal | Internal Catalog     |
+--------------------------------------------------------------+----------+----------------------+
```

|返回|说明|
|---|---|
|Catalog|目录名称。|
|Type|目录类型。|
|Comment|目录的注释。|
```markdown
|返回|说明|
|---|---|
|Catalog|目录名称。|
|Type|目录类型。|
|Comment|目录的备注。|