---
displayed_sidebar: "Japanese"
---

# SHOW PROC

## 説明

StarRocks クラスタの特定の指標を表示します。

## 構文

```SQL
SHOW PROC { '/backends' | '/compute_nodes' | '/dbs' 
          | '/jobs' | '/statistic' | '/tasks' | '/frontends' 
          | '/brokers' | '/resources' | '/load_error_hub' 
          | '/transactions' | '/monitor' | '/current_queries' 
          | '/current_backend_instances' | '/cluster_balance' 
          | '/routine_loads' | '/colocation_group' | '/catalog' }
```

## パラメータ

| **パラメータ**                | **説明**                                              |
| ---------------------------- | ------------------------------------------------------------ |
| '/backends'                  | クラスタ内のBEノードの情報を表示します。            |
| '/compute_nodes'             | クラスタ内のCNノードの情報を表示します。            |
| '/dbs'                       | クラスタ内のデータベースの情報を表示します。           |
| '/jobs'                      | クラスタ内のジョブの情報を表示します。                |
| '/statistic'                 | クラスタ内の各データベースの統計情報を表示します。        |
| '/tasks'                     | クラスタ内の全ての一般的なタスクの総数と失敗したタスクを表示します。 |
| '/frontends'                 | クラスタ内のFEノードの情報を表示します。            |
| '/brokers'                   | クラスタ内のBrokerノードの情報を表示します。        |
| '/resources'                 | クラスタ内のリソースの情報を表示します。           |
| '/load_error_hub'            | クラスタのロードエラーハブの構成を表示します。これはローディングジョブのエラーメッセージを管理するために使用されます。 |
| '/transactions'              | クラスタ内のトランザクションの情報を表示します。        |
| '/monitor'                   | クラスタ内の監視情報を表示します。             |
| '/current_queries'           | 現在のFEノードで実行中のクエリの情報を表示します。 |
| '/current_backend_instances' | クラスタ内でリクエストを処理しているBEノードを表示します。 |
| '/cluster_balance'           | クラスタ内の負荷分散情報を表示します。           |
| '/routine_loads'             | クラスタ内のRoutine Loadの情報を表示します。        |
| '/colocation_group'          | クラスタ内のColocate Joinグループの情報を表示します。 |
| '/catalog'                   | クラスタ内のカタログの情報を表示します。            |

## 例

Example 1: クラスタ内のBEノードの情報を表示します。

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

| **リターン**            | **説明**                                              |
| --------------------- | ------------------------------------------------------------ |
| BackendId             | BEノードのID                                             |
| IP                    | BEノードのIPアドレス                                       |
| HeartbeatPort         | BEノードのハートビートサービスポート                              |
| BePort                | BEノードのThrift Serverポート                                     |
| HttpPort              | BEノードのHTTPサーバーポート                                  |
| BrpcPort              | BEノードのbRPCポート                                           |
| LastStartTime         | BEノードが最後に起動した時間                                  |
| LastHeartbeat         | BEノードが最後にハートビートを受信した時間                         |
| Alive                 | BEノードが稼働しているかどうか                                     |
| SystemDecommissioned  | BEノードが運用停止中かどうか                                      |
| ClusterDecommissioned | クラスタ内でBEノードが運用停止中かどうか                             |
| TabletNum             | BEノードでのタブレットの数                                       |
| DataUsedCapacity      | BEノードでデータに使用されるストレージ容量                              |
| AvailCapacity         | BEノードの利用可能なストレージ容量                                 |
| TotalCapacity         | BEノードの総ストレージ容量                                       |
| UsedPct               | BEノードで使用されているストレージ容量の割合                           |
| MaxDiskUsedPct        | BEノードで使用されているストレージ容量の最大割合                        |
| ErrMsg                | BEノード内のエラーメッセージ                                      |
| Version               | BEノードのStarRocksバージョン                              |
| Status                | BEノードのステータス情報。最後にBEノードがタブレットをレポートした時間などを含む。 |
| DataTotalCapacity     | 使用中および利用可能なデータストレージ容量の合計。 `DataUsedCapacity` と `AvailCapacity` の合計。 |
| DataUsedPct           | データストレージが合計データ容量を占める割合（DataUsedCapacity/DataTotalCapacity）。 |
| CpuCores              | BEノードのCPUコア数                                         |
| NumRunningQueries     | クラスタ内で現在実行中のクエリの数                               |
| MemUsedPct            | 現在のメモリ使用率の割合                                     |
| CpuUsedPct            | 現在のCPU使用率の割合                                       |

Example 2: クラスタ内のデータベースの情報を表示します。

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

| **リターン**               | **説明**                                   |
| ------------------------ | ------------------------------------------------- |
| DbId                     | データベースID                                      |
| DbName                   | データベース名                                    |
| TableNum                 | データベース内のテーブルの数                            |
| Quota                    | データベースのストレージクォータ                     |
| LastConsistencyCheckTime | 最後に整合性チェックが実行された時間                          |
| ReplicaQuota             | データベースのデータレプリカクォータ                |

Example 3: クラスタ内のジョブの情報を表示します。

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

| **リターン** | **説明**                    |
| ---------- | ---------------------------------- |
| DbId       | Database ID.                       |
| DbName     | Database name.                     |
| JobType    | Job type.                          |
| Pending    | Number of jobs that are pending.   |
| Running    | Number of jobs that are running.   |
| Finished   | Number of jobs that are finished.  |
| Cancelled  | Number of jobs that are cancelled. |
| Total      | Total number of jobs.              |

Example 4: クラスタ内の各データベースの統計を示します。

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
7 行がセットされています (0.01 秒)

mysql> show proc '/statistic/10002';
+------------------+---------------------+----------------+-------------------+
| UnhealthyTablets | InconsistentTablets | CloningTablets | ErrorStateTablets |
+------------------+---------------------+----------------+-------------------+
| []               | []                  | []             | [116703, 116706]  |
+------------------+---------------------+----------------+-------------------+
```

| **Return**            | **Description**                                              |
| --------------------- | ------------------------------------------------------------ |
| DbId                  | データベースのID。                                             |
| DbName                | データベース名。                                               |
| TableNum              | データベース内のテーブルの数。                                 |
| PartitionNum          | データベース内のパーティションの数。                         |
| IndexNum              | データベース内のインデックスの数。                           |
| TabletNum             | データベース内のタブレットの数。                             |
| ReplicaNum            | データベース内のレプリカの数。                                |
| UnhealthyTabletNum    | データ再構築中に未完成（不健康）なタブレットの数。            |
| InconsistentTabletNum | データベースの不整合なタブレットの数。                       |
| CloningTabletNum      | データベース内でクローンされているタブレットの数。           |
| ErrorStateTabletNum   | プライマリキー型のテーブルで、エラー状態のタブレットの数。   |
| ErrorStateTablets     | プライマリキー型のテーブルで、エラー状態のタブレットのID。 |

Example 5: クラスタ内のすべての汎用タスクと失敗したタスクの総数を表示します。

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

| **Return** | **Description**         |
| ---------- | ----------------------- |
| TaskType   | タスクの種類。          |
| FailedNum  | 失敗したタスクの数。    |
| TotalNum   | タスクの総数。          |

Example 6: クラスタ内のFEノードの情報を表示します。

```Plain
mysql> SHOW PROC '/frontends';
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
| Name                             | IP            | EditLogPort | HttpPort | QueryPort | RpcPort | Role     | ClusterId  | Join  | Alive | ReplayedJournalId | LastHeartbeat | IsHelper | ErrMsg        | StartTime | Version |
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
| xxx.xx.xx.xxx_9009_1600088918395 | xxx.xx.xx.xxx | 9009        | 7390     | 0         | 0       | FOLLOWER | 1747363037 | false | false | 0                 | NULL          | true     | got exception | NULL      | NULL    |
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
```

| **Return**        | **Description**                                        |
| ----------------- | ------------------------------------------------------ |
| Name              | FEノードの名前。                                       |
| IP                | FEノードのIPアドレス。                                 |
| EditLogPort       | FEノード間の通信用ポート。                             |
| HttpPort          | FEノードのHTTPサーバーポート。                         |
| QueryPort         | FEノードのMySQLサーバーポート。                        |
| RpcPort           | FEノードのRPCポート。                                  |
| Role              | FEノードの役割（リーダー、フォロワー、またはオブザーバー）。|
| ClusterId         | クラスタID。                                           |
| Join              | FEノードがクラスタに参加しているかどうか。             |
| Alive             | FEノードが稼働しているかどうか。                       |
| ReplayedJournalId | FEノードがリプレイした最大のメタデータID。             |
| LastHeartbeat     | FEノードが最後にハートビートを送信した時刻。         |
| IsHelper          | FEノードがBDBJEヘルパーノードであるかどうか。         |
| ErrMsg            | FEノードのエラーメッセージ。                           |
| StartTime         | FEノードが起動した時刻。                              |
| Version           | FEノードのStarRocksバージョン。                         |

Example 7: クラスタ内のブローカーノードの情報を表示します。

```Plain
mysql> SHOW PROC '/brokers';
+-------------+---------------+------+-------+---------------+---------------------+--------+
| Name        | IP            | Port | Alive | LastStartTime | LastUpdateTime      | ErrMsg |
+-------------+---------------+------+-------+---------------+---------------------+--------+
| hdfs_broker | xxx.xx.xx.xxx | 8500 | true  | NULL          | 2022-10-10 16:37:59 |        |
| Name           | Broker node name.                                            |
| IP             | IP address of the broker node.                               |
| Port           | Thrift Server port of the broker node. The port is used to receive requests. |
| Alive          | If the broker node is alive.                                 |
| LastStartTime  | The last time when the broker node was started.              |
| LastUpdateTime | The last time when the broker node was updated.              |
| ErrMsg         | Error message in the broker node.                            |

Example 8: Shows the information of resources in the cluster.

```Plain
mysql> SHOW PROC '/resources';
+-------------------------+--------------+---------------------+------------------------------+
| Name                    | ResourceType | Key                 | Value                        |
+-------------------------+--------------+---------------------+------------------------------+
| hive_resource_stability | hive         | hive.metastore.uris | thrift://xxx.xx.xxx.xxx:9083 |
| hive2                   | hive         | hive.metastore.uris | thrift://xxx.xx.xx.xxx:9083  |
+-------------------------+--------------+---------------------+------------------------------+
```

| Name         | Resource name.  |
| ResourceType | Resource type.  |
| Key          | Resource key.   |
| Value        | Resource value. |

Example 9: Shows the information of transactions in the cluster.

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

| DbId       | Database ID.                  |
| DbName     | Database name.                |
| State      | The state of the transaction. |
| Number     | Number of transactions.       |

Example 10: Shows the monitoring information in the cluster.

```Plain
mysql> SHOW PROC '/monitor';
+------+------+
| Name | Info |
+------+------+
| jvm  |      |
+------+------+
```

| Name       | JVM name.        |
| Info       | JVM information. |

Example 11: Shows the load balance information in the cluster.

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

| Item       | Sub-command item in `cluster_balance `.          |
| Number     | Number of each sub-command in `cluster_balance`. |

Example 12: Shows the information of Colocate Join groups in the cluster.

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

| GroupId        | Colocate Join Group ID.                         |
| GroupName      | Colocate Join Group name.                       |
| TableIds       | IDs of tables in the Colocate Join Group.       |
| BucketsNum     | Buckets in the Colocate Join Group.             |
| ReplicationNum | Replications in the Colocate Join Group.        |
| DistCols       | Distribution column of the Colocate Join Group. |
| IsStable       | If the Colocate Join Group is stable.           |

Example 13: Shows the information of catalogs in the cluster.

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

| Catalog    | Catalog name.             |
| Type       | Catalog type.             |
| Comment    | Comments for the catalog. |