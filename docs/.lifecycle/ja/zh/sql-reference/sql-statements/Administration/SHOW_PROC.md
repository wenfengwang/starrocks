---
displayed_sidebar: Chinese
---

# SHOW PROC

## 機能

現在のクラスター内の特定の指標を表示します。

> **注意**
>
> この操作には SYSTEM レベルの OPERATE 権限が必要です。

## 構文

```SQL
SHOW PROC { '/backends' | '/compute_nodes' | '/dbs' 
          | '/jobs' | '/statistics' | '/tasks' | '/frontends' 
          | '/brokers' | '/resources' | '/load_error_hub' 
          | '/transactions' | '/monitor' | '/current_queries' 
          | '/current_backend_instances' | '/cluster_balance' 
          | '/routine_loads' | '/colocation_group' | '/catalog' }
```

## パラメータ説明

| **パラメータ**                     | **説明**                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| '/backends'                  | 現在のクラスターの BE ノード情報を表示します。                                 |
| '/compute_nodes'             | 現在のクラスターの CN ノード情報を表示します。                                 |
| '/dbs'                       | 現在のクラスターのデータベース情報を表示します。                                   |
| '/jobs'                      | 現在のクラスターのジョブ情報を表示します。                                     |
| '/statistics'                 | 現在のクラスターの各データベースの統計情報を表示します。                             |
| '/tasks'                     | 現在のクラスターの各種タスクの総数と失敗総数を表示します。                   |
| '/frontends'                 | 現在のクラスターの FE ノード情報を表示します。                                 |
| '/brokers'                   | 現在のクラスターの Broker ノード情報を表示します。                             |
| '/resources'                 | 現在のクラスターのリソース情報を表示します。                                     |
| '/load_error_hub'            | 現在のクラスターの Error Hub の設定情報を表示します。Error Hub はインポートジョブで発生したエラー情報を管理するために使用されます。 |
| '/transactions'              | 現在のクラスターのトランザクション情報を表示します。                                     |
| '/monitor'                   | 現在のクラスターのモニター情報を表示します。                                     |
| '/current_queries'           | 現在接続されている FE ノードが実行中のクエリ情報を表示します。                       |
| '/current_backend_instances' | 現在のクラスターでジョブを実行中の BE ノードを表示します。                         |
| '/cluster_balance'           | 現在のクラスターの負荷情報を表示します。                                     |
| '/routine_loads'             | 現在のクラスターの Routine Load インポート情報を表示します。                       |
| '/colocation_group'          | 現在のクラスターの Colocate Join Group 情報を表示します。                    |
| '/catalog'                   | 現在のクラスターの Catalog 情報を表示します。                                |

## 例

例1：現在のクラスターの BE ノード情報を表示します。

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

| **戻り値**              | **説明**                                                     |
| --------------------- | ------------------------------------------------------------ |
| BackendId             | BE ノードの ID。                                                 |
| IP                    | BE ノードの IP アドレス。                                            |
| HeartbeatPort         | BE ノードのハートビートポート。                                        |
| BePort                | BE ノードの Thrift Server ポート。                                 |
| HttpPort              | BE ノードの HTTP Server ポート。                                   |
| BrpcPort              | BE ノードの bRPC ポート。                                          |
| LastStartTime         | BE ノードの最後の起動時間。                                      |
| LastHeartbeat         | BE ノードの最後の FE からのハートビート受信時間。                              |
| Alive                 | BE ノードの生存状態。                                            |
| SystemDecommissioned  | BE ノードがシステムから撤去されているかどうか。                                        |
| ClusterDecommissioned | BE ノードがクラスターから撤去されているかどうか。                                  |
| TabletNum             | BE ノードのタブレット数。                                        |
| DataUsedCapacity      | BE ノードが使用しているストレージ容量。                                    |
| AvailCapacity         | BE ノードの利用可能なストレージ容量。                                        |
| TotalCapacity         | BE ノードの総ストレージ容量。                                          |
| UsedPct               | BE ノードの現在のストレージ使用率。                                |
| MaxDiskUsedPct        | BE ノードの最大ストレージ使用率。                                |
| ErrMsg                | BE ノードのエラーメッセージ。                                            |
| Version               | BE ノードの StarRocks バージョン。                                     |
| Status                | BE ノードの状態情報、最近のタブレット報告時間を含む。     |
| DataTotalCapacity     | データファイルが使用するディスク容量 + 利用可能なディスク容量、つまり DataUsedCapacity + AvailCapacity。 |
| DataUsedPct           | データファイルがディスクを占める割合、つまり DataUsedCapacity/DataTotalCapacity。 |
| CpuCores              | BE ノードの CPU コア数。                                           |
| NumRunningQueries     | 現在実行中のクエリ数。                                            |
| MemUsedPct            | 現在のメモリ使用率。                                            |
| CpuUsedPct            | 現在の CPU 使用率。                                          |

例2：現在のクラスターのデータベース情報を表示します。

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

| **戻り値**                 | **説明**                     |
| ------------------------ | ---------------------------- |
| DbId                     | データベースの ID。                  |
| DbName                   | データベースの名前。                 |
| TableNum                 | データベースに含まれるテーブルの数。           |
| Quota                    | データベースに設定されたストレージクォータ。       |
| LastConsistencyCheckTime | データベースの最後の一貫性チェック時間。 |
| ReplicaQuota             | データベースのレプリカクォータ。           |

例3：現在のクラスターのジョブ情報を表示します。対応する `DbId` を使用して、そのデータベース内の詳細なジョブ情報をさらに照会できます。

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

| **戻り値**  | **説明**         |
| --------- | ---------------- |
| DbId      | データベースID。      |
| DbName    | データベース名。     |
| JobType   | ジョブタイプ。       |
| Pending   | 保留中のジョブ数。   |
| Running   | 実行中のジョブ数。 |
| Finished  | 完了したジョブ数。   |
| Cancelled | キャンセルされたジョブ数。   |
| Total     | ジョブの総数。       |

例四：現在のクラスターの各データベースの統計情報を見る。

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

| **戻り値**              | **説明**                                    |
| --------------------- | ------------------------------------------- |
| DbId                  | データベースID。                                 |
| DbName                | データベース名。                                |
| TableNum              | テーブル数。                          |
| PartitionNum          | パーティション数。                        |
| IndexNum              | インデックス数。                        |
| TabletNum             | タブレット数。                    |
| ReplicaNum            | レプリカ数。                        |
| UnhealthyTabletNum    | 未完了のタブレット数。      |
| InconsistentTabletNum | 不一致のタブレット数。              |
| CloningTabletNum      | クローニング中のタブレット数。   |
| ErrorStateTabletNum   | エラー状態のタブレット数。          |
| ErrorStateTablets     | エラー状態のタブレットID。         |

例五：現在のクラスターの各種タスクの総数と失敗総数を見る。

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

| **戻り値**  | **説明**     |
| --------- | ------------ |
| TaskType  | タスクタイプ。   |
| FailedNum | 失敗したタスク数。 |
| TotalNum  | タスクの総数。   |

例六：現在のクラスターの FE ノード情報を見る。

```Plain
mysql> SHOW PROC '/frontends';
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
| Name                             | IP            | EditLogPort | HttpPort | QueryPort | RpcPort | Role     | ClusterId  | Join  | Alive | ReplayedJournalId | LastHeartbeat | IsHelper | ErrMsg        | StartTime | Version |
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
| xxx.xx.xx.xxx_9009_1600088918395 | xxx.xx.xx.xxx | 9009        | 7390     | 0         | 0       | FOLLOWER | 1747363037 | false | false | 0                 | NULL          | true     | got exception | NULL      | NULL    |
+----------------------------------+---------------+-------------+----------+-----------+---------+----------+------------+-------+-------+-------------------+---------------+----------+---------------+-----------+---------+
```

| **戻り値**          | **説明**                                      |
| ----------------- | --------------------------------------------- |
| Name              | FEノード名。                                 |
| IP                | FEノードのIPアドレス。                             |
| EditLogPort       | FEノード間の通信ポート。                       |
| HttpPort          | FEノードのHTTPサーバーポート。                    |
| QueryPort         | FEノードのMySQL Serverポート。                   |
| RpcPort           | FEノードのRPCポート。                            |
| Role              | FEノードの役割（Leader、Follower、Observer）。 |
| ClusterId         | クラスタID。                                     |
| Join              | FEノードがクラスタに参加したかどうか。           |
| Alive             | FEノードの生存状態。                             |
| ReplayedJournalId | FEノードで再生された最大のメタデータログID。     |
| LastHeartbeat     | FEノードが最後にハートビートを送信した時間。     |
| IsHelper          | FEノードがBDBJEのHelperノードであるかどうか。    |
| ErrMsg            | FEノードのエラーメッセージ。                     |
| StartTime         | FEノードの起動時間。                             |
| Version           | FEノードのStarRocksバージョン。                  |

示例七：現在のクラスタのBrokerノード情報を確認します。

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

| **戻り値**       | **説明**                                         |
| -------------- | ------------------------------------------------ |
| Name           | Brokerノードの名前。                              |
| IP             | BrokerノードのIPアドレス。                        |
| Port           | BrokerノードのThrift Serverポート（リクエスト受信用）。 |
| Alive          | Brokerノードの生存状態。                          |
| LastStartTime  | Brokerノードの最後の起動時間。                    |
| LastUpdateTime | Brokerノードの最後の更新時間。                    |
| ErrMsg         | Brokerノードのエラーメッセージ。                  |

示例八：現在のクラスタのリソース情報を確認します。

```Plain
mysql> SHOW PROC '/resources';
+-------------------------+--------------+---------------------+------------------------------+
| Name                    | ResourceType | Key                 | Value                        |
+-------------------------+--------------+---------------------+------------------------------+
| hive_resource_stability | hive         | hive.metastore.uris | thrift://xxx.xx.xxx.xxx:9083 |
| hive2                   | hive         | hive.metastore.uris | thrift://xxx.xx.xx.xxx:9083  |
+-------------------------+--------------+---------------------+------------------------------+
```

| **戻り値**     | **説明**     |
| ------------ | ------------ |
| Name         | リソース名。   |
| ResourceType | リソースタイプ。 |
| Key          | リソースキー。 |
| Value        | リソース値。   |

示例九：現在のクラスタのトランザクション情報を確認します。対応する`DbId`でそのデータベースの詳細なトランザクション情報をさらに確認できます。

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

| **戻り値** | **説明**     |
| -------- | ------------ |
| DbId     | データベースID。 |
| DbName   | データベース名。 |
| State    | トランザクションの状態。 |
| Number   | トランザクション数。   |

示例十：現在のクラスタのモニタリング情報を確認します。

```Plain
mysql> SHOW PROC '/monitor';
+------+------+
| Name | Info |
+------+------+
| jvm  |      |
+------+------+
```

| **戻り値** | **説明**   |
| -------- | ---------- |
| Name     | JVMの名前。 |
| Info     | JVM情報。   |

示例十一：現在のクラスタの負荷情報を確認します。

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

| **戻り値** | **説明**                                     |
| -------- | -------------------------------------------- |
| Item     | cluster_balanceのサブコマンド。               |
| Number   | 各サブコマンドの実行中の数。                 |

示例十二：現在のクラスタのColocate Join Group情報を確認します。

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

| **戻り値**       | **説明**                              |
| -------------- | ------------------------------------- |
| GroupId        | Colocate Join GroupのID。              |
| GroupName      | Colocate Join Groupの名前。            |
| TableIds       | Colocate Join Groupに含まれるテーブルのID。 |
| BucketsNum     | Colocate Join Groupのバケット数。      |
| ReplicationNum | Colocate Join Groupのレプリケーション数。 |
| DistCols       | Colocate Join Groupの分散列のタイプ。  |
| IsStable       | Colocate Join Groupが安定しているか。  |

示例十三：現在のクラスタのCatalog情報を確認します。

```Plain
mysql> SHOW PROC '/catalog';
+--------------------------------------------------------------+----------+----------------------+
| Catalog                                                      | Type     | Comment              |
+--------------------------------------------------------------+----------+----------------------+
| resource_mapping_inside_catalog_hive_hive2                   | hive     | mapping hive catalog |
| resource_mapping_inside_catalog_hive_hive_resource_stability | hive     | mapping hive catalog |
| default_catalog                                              | Internal | Internal Catalog     |
```
+--------------------------------------------------------------+----------+----------------------+
```

| **戻り値** | **説明**       |
| ---------- | -------------- |
| カタログ   | カタログ名。   |
| タイプ     | カタログの種類。 |
| コメント   | カタログの説明。 |

