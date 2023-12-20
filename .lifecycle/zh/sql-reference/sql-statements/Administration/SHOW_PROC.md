---
displayed_sidebar: English
---

# 显示过程

## 描述

展示 StarRocks 集群的某些指标。

:::tip

此操作需要 **SYSTEM** 级别的 **OPERATE** 权限。您可以按照 [GRANT](../account-management/GRANT.md) 的说明来授予此权限。

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

|参数|说明|
|---|---|
|'/backends'|显示集群中BE节点的信息。|
|'/compute_nodes'|显示集群中CN节点的信息。|
|'/dbs'|显示集群中数据库的信息。|
|'/jobs'|显示集群中作业的信息。|
|'/statistic'|显示集群中每个数据库的统计信息。|
|'/tasks'|显示集群中所有通用任务和失败任务的总数。|
|'/frontends'|显示集群中FE节点的信息。|
|'/brokers'|显示集群中Broker节点信息。|
|'/resources'|显示集群中的资源信息。|
|'/load_error_hub'|显示集群的Load Error Hub配置，用于管理加载作业的错误消息。|
|'/transactions'|显示集群中事务的信息。|
|'/monitor'|显示集群中的监控信息。|
|'/current_queries'|显示当前FE节点上运行查询的信息。|
|'/current_backend_instances'|显示集群中正在处理请求的BE节点。|
|'/cluster_balance'|显示集群中的负载均衡信息。|
|'/routine_loads'|显示集群中Routine Load的信息。|
|'/colocation_group'|显示集群中Colocate Join组的信息。|
|'/catalog'|显示集群中目录的信息。|

## 示例

示例 1：展示集群中 BE 节点的信息。

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

|返回|说明|
|---|---|
|BackendId|BE节点ID。|
|IP|BE 节点的IP 地址。|
|HeartbeatPort|BE节点心跳服务端口。|
|BePort|BE 节点的 Thrift Server 端口。|
|HttpPort|BE 节点的 HTTP Server 端口。|
|BrpcPort|BE节点的bRPC端口。|
|LastStartTime|BE 节点上次启动时间。|
|LastHeartbeat|BE 节点最后一次收到心跳的时间。|
|Alive|如果 BE 节点还活着。|
|SystemDecommissioned|如果 BE 节点已停用。|
|ClusterDecommissioned|如果 BE 节点在集群内停用。|
|TabletNum|BE 节点中的平板电脑数量。|
|DataUsedCapacity|BE 节点中数据使用的存储容量。|
|AvailCapacity|BE 节点中的可用存储容量。|
|TotalCapacity|BE 节点的总存储容量。|
|UsedPct|BE 节点存储容量的使用百分比。|
|MaxDiskUsedPct|BE 节点存储容量使用的最大百分比。|
|ErrMsg|BE 节点中的错误消息。|
|版本|BE 节点的 StarRocks 版本。|
|状态|BE节点的状态信息，包括BE节点最后一次上报tablets的时间。|
|DataTotalCapacity|已用和可用数据存储容量的总和。 DataUsedCapacity 和 AvailCapacity 的总和。|
|DataUsedPct|数据存储占总数据容量的百分比（DataUsedCapacity/DataTotalCapacity）。|
|CpuCores|BE 节点中的 CPU 核心数。|
|NumRunningQueries|集群中当前运行的查询数。|
|MemUsedPct|当前内存使用百分比。|
|CpuUsedPct|当前 CPU 使用百分比。|

示例 2：展示集群中数据库的信息。

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

|返回|说明|
|---|---|
|DbId|数据库 ID。|
|DbName|数据库名称。|
|TableNum|数据库中表的数量。|
|配额|数据库的存储配额。|
|LastConsistencyCheckTime|最后一次执行一致性检查的时间。|
|ReplicaQuota|数据库数据副本配额。|

示例 3：展示集群中作业的信息。

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

|返回|说明|
|---|---|
|DbId|数据库 ID。|
|DbName|数据库名称。|
|作业类型|作业类型。|
|待处理|待处理的作业数量。|
|正在运行|正在运行的作业数。|
|已完成|已完成的作业数。|
|已取消|已取消的作业数量。|
|总计|职位总数。|

示例 4：展示集群中每个数据库的统计信息。

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
7 rows in set (0.01 sec)

mysql> show proc '/statistic/10002';
+------------------+---------------------+----------------+-------------------+
| UnhealthyTablets | InconsistentTablets | CloningTablets | ErrorStateTablets |
+------------------+---------------------+----------------+-------------------+
| []               | []                  | []             | [116703, 116706]  |
+------------------+---------------------+----------------+-------------------+
```

|返回|说明|
|---|---|
|DbId|数据库 ID。|
|DbName|数据库名称。|
|TableNum|数据库中表的数量。|
|PartitionNum|数据库中的分区数。|
|IndexNum|数据库中索引的数量。|
|TabletNum|数据库中的平板电脑数量。|
|ReplicaNum|数据库中的副本数量。|
|UnhealthyTabletNum|数据重新分配期间数据库中未完成（不健康）的 Tablet 数量。|
|InconcientTabletNum|数据库中不一致的 Tablet 数量。|
|CloningTabletNum|正在数据库中克隆的平板电脑数量。|
|ErrorStateTabletNum|在主键类型表中，处于错误状态的平板电脑数量。|
|ErrorStateTablets|在主键类型表中，处于错误状态的平板电脑的 ID。|

示例 5：展示集群中所有通用任务和失败任务的总数。

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
|任务类型|任务类型。|
|FailedNum|失败任务的数量。|
|TotalNum|任务总数。|

示例 6：展示集群中 FE 节点的信息。

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
|名称|FE 节点名称。|
|IP|FE节点的IP地址。|
|EditLogPort|FE 节点之间通信的端口。|
|HttpPort|FE节点的HTTP Server端口。|
|QueryPort|FE节点MySQL Server端口。|
|RpcPort|FE节点的RPC端口。|
|角色|FE 节点的角色（领导者、追随者或观察者）。|
|ClusterId|集群 ID。|
|加入|如果 FE 节点已加入集群。|
|Alive|如果 FE 节点还活着。|
|ReplayedJournalId|FE节点已重播的最大元数据ID。|
|LastHeartbeat|FE节点最后一次发送心跳的时间。|
|IsHelper|如果 FE 节点是 BDBJE 辅助节点。|
|ErrMsg|FE 节点中的错误消息。|
|StartTime|FE节点启动时间。|
|版本|FE 节点的 StarRocks 版本。|

示例 7：展示集群中 Broker 节点的信息。

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
|名称|代理节点名称。|
|IP|代理节点的IP地址。|
|端口|代理节点的 Thrift 服务器端口。该端口用于接收请求。|
|Alive|如果代理节点处于活动状态。|
|LastStartTime|代理节点上次启动时间。|
|LastUpdateTime|最后一次更新代理节点的时间。|
|ErrMsg|代理节点中的错误消息。|

示例 8：展示集群中资源的信息。

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
|名称|资源名称。|
|资源类型|资源类型。|
|密钥|资源密钥。|
|价值|资源价值。|

示例 9：展示集群中事务的信息。

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
|DbId|数据库 ID。|
|DbName|数据库名称。|
|状态|事务的状态。|
|数量|交易数量。|

示例 10：展示集群的监控信息。

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
|名称|JVM 名称。|
|信息|JVM 信息。|

示例 11：展示集群中的负载均衡信息。

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
|Item|cluster_balance 中的子命令项。|
|Number|cluster_balance 中各子命令的数量。|

示例 12：展示集群中 Colocate Join 组的信息。

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
|GroupId|并置加入组 ID。|
|组名称|并置加入组名称。|
|TableIds|Colocate Join Group 中表的 ID。|
|BucketsNum|Colocate Join 组中的存储桶。|
|ReplicationNum|Colocate Join 组中的复制。|
|DistCols|Colocate Join Group 的分布列。|
|IsStable|Colocate Join Group 是否稳定。|

示例 13：展示集群中目录的信息。

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
|目录|目录名称。|
|类型|目录类型。|
|评论|对目录的评论。|
