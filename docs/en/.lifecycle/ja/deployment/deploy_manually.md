---
displayed_sidebar: "Japanese"
---

# StarRocksの手動デプロイ

このトピックでは、共有なしのStarRocks（BEがストレージと計算の両方を担当する）を手動でデプロイする方法について説明します。他のインストールモードについては、[デプロイの概要](../deployment/deployment_overview.md)を参照してください。

共有データのStarRocksクラスタ（ストレージと計算が分離されている）をデプロイする場合は、[共有データのStarRocksのデプロイと使用](../deployment/shared_data/s3.md)を参照してください。

## ステップ1：リーダーFEノードを起動する

以下の手順は、FEインスタンスで実行されます。

1. メタデータの保存用に専用のディレクトリを作成します。メタデータは、FEデプロイメントファイルとは別のディレクトリに保存することをおすすめします。このディレクトリが存在し、書き込みアクセス権があることを確認してください。

   ```YAML
   # <meta_dir>を作成したいメタデータディレクトリに置き換えてください。
   mkdir -p <meta_dir>
   ```

2. StarRocks FEデプロイメントファイルを保存しているディレクトリに移動し、FE構成ファイル **fe/conf/fe.conf** を編集します。

   a. 構成項目 `meta_dir` でメタデータディレクトリを指定します。

      ```YAML
      # <meta_dir>を作成したメタデータディレクトリに置き換えてください。
      meta_dir = <meta_dir>
      ```

   b. [環境構成チェックリスト](../deployment/environment_configurations.md#fe-ports) で言及されているFEポートのいずれかが使用中の場合、FE構成ファイルで有効な代替ポートを割り当てる必要があります。

      ```YAML
      http_port = aaaa        # デフォルト: 8030
      rpc_port = bbbb         # デフォルト: 9020
      query_port = cccc       # デフォルト: 9030
      edit_log_port = dddd    # デフォルト: 9010
      ```

      > **注意**
      >
      > クラスタ内で複数のFEノードをデプロイする場合は、各FEノードに同じ `http_port` を割り当てる必要があります。

   c. クラスタでIPアドレスアクセスを有効にする場合は、構成ファイルに `priority_networks` 構成項目を追加し、FEノードに専用のIPアドレス（CIDR形式）を割り当てる必要があります。クラスタで[FQDNアクセス](../administration/enable_fqdn.md)を有効にする場合は、この構成項目を無視しても構いません。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > インスタンスで所有しているIPアドレスを表示するには、ターミナルで `ifconfig` を実行できます。

   d. インスタンスに複数のJDKがインストールされており、環境変数 `JAVA_HOME` で指定されているJDKとは異なる特定のJDKを使用したい場合は、構成ファイルに `JAVA_HOME` 構成項目を追加し、選択したJDKがインストールされているパスを指定する必要があります。

      ```YAML
      # <path_to_JDK>を選択したJDKがインストールされているパスに置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   f. 高度な構成項目の詳細については、[パラメータ構成 - FE構成項目](../administration/Configuration.md#fe-configuration-items)を参照してください。

3. FEノードを起動します。

   - クラスタでIPアドレスアクセスを有効にする場合は、次のコマンドを実行してFEノードを起動します。

     ```Bash
     ./fe/bin/start_fe.sh --daemon
     ```

   - クラスタでFQDNアクセスを有効にする場合は、次のコマンドを実行してFEノードを起動します。

     ```Bash
     ./fe/bin/start_fe.sh --host_type FQDN --daemon
     ```

     最初にノードを起動するときにパラメータ `--host_type` を指定する必要があります。

     > **注意**
     >
     > FQDNアクセスが有効な状態でFEノードを起動する前に、**/etc/hosts** のすべてのインスタンスにホスト名が割り当てられていることを確認してください。詳細については、[環境構成チェックリスト - ホスト名](../deployment/environment_configurations.md#hostnames)を参照してください。

4. FEノードが正常に起動したかどうかを確認するために、FEログをチェックします。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   "2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020." のようなログレコードが表示されれば、FEノードが正常に起動しています。

## ステップ2：BEサービスを起動する

以下の手順は、BEインスタンスで実行されます。

1. データの保存用に専用のディレクトリを作成します。BEデプロイディレクトリとは別のディレクトリにデータを保存することをおすすめします。このディレクトリが存在し、書き込みアクセス権があることを確認してください。

   ```YAML
   # <storage_root_path>を作成したいデータ保存ディレクトリに置き換えてください。
   mkdir -p <storage_root_path>
   ```

2. StarRocks BEデプロイメントファイルを保存しているディレクトリに移動し、BE構成ファイル **be/conf/be.conf** を編集します。

   a. 構成項目 `storage_root_path` でデータディレクトリを指定します。

      ```YAML
      # <storage_root_path>を作成したデータディレクトリに置き換えてください。
      storage_root_path = <storage_root_path>
      ```

   b. [環境構成チェックリスト](../deployment/environment_configurations.md#be-ports) で言及されているBEポートのいずれかが使用中の場合、BE構成ファイルで有効な代替ポートを割り当てる必要があります。

      ```YAML
      be_port = vvvv                   # デフォルト: 9060
      be_http_port = xxxx              # デフォルト: 8040
      heartbeat_service_port = yyyy    # デフォルト: 9050
      brpc_port = zzzz                 # デフォルト: 8060
      ```

   c. クラスタでIPアドレスアクセスを有効にする場合は、構成ファイルに `priority_networks` 構成項目を追加し、BEノードに専用のIPアドレス（CIDR形式）を割り当てる必要があります。クラスタでFQDNアクセスを有効にする場合は、この構成項目を無視しても構いません。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > インスタンスで所有しているIPアドレスを表示するには、ターミナルで `ifconfig` を実行できます。

   d. インスタンスに複数のJDKがインストールされており、環境変数 `JAVA_HOME` で指定されているJDKとは異なる特定のJDKを使用したい場合は、構成ファイルに `JAVA_HOME` 構成項目を追加し、選択したJDKがインストールされているパスを指定する必要があります。

      ```YAML
      # <path_to_JDK>を選択したJDKがインストールされているパスに置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   高度な構成項目の詳細については、[パラメータ構成 - BE構成項目](../administration/Configuration.md#be-configuration-items)を参照してください。

3. BEノードを起動します。

      ```Bash
      ./be/bin/start_be.sh --daemon
      ```

      > **注意**
      >
      > - FQDNアクセスが有効な状態でBEノードを起動する前に、**/etc/hosts** のすべてのインスタンスにホスト名が割り当てられていることを確認してください。詳細については、[環境構成チェックリスト - ホスト名](../deployment/environment_configurations.md#hostnames)を参照してください。
      > - BEノードを起動する際にパラメータ `--host_type` を指定する必要はありません。

4. BEノードが正常に起動したかどうかを確認するために、BEログをチェックします。

      ```Bash
      cat be/log/be.INFO | grep heartbeat
      ```

      "I0810 16:18:44.487284 3310141 task_worker_pool.cpp:1387] Waiting to receive first heartbeat from frontend" のようなログレコードが表示されれば、BEノードが正常に起動しています。

5. 他のBEインスタンスで前述の手順を繰り返すことで、新しいBEノードを起動することができます。

> **注意**
>
> 少なくとも3つのBEノードがデプロイされ、StarRocksクラスタに追加された場合、BEの高可用性クラスタが自動的に形成されます。

## ステップ3：（オプション）CNサービスを起動する

Compute Node（CN）は、データ自体を保持しない状態の計算サービスです。クエリの追加計算リソースを提供するために、クラスタにCNノードをオプションで追加できます。CNノードはv2.4以降でサポートされています。

1. StarRocks BEデプロイメントファイルを保存しているディレクトリに移動し、CN構成ファイル **be/conf/cn.conf** を編集します。

   a. [環境構成チェックリスト](../deployment/environment_configurations.md) で言及されているCNポートのいずれかが使用中の場合、CN構成ファイルで有効な代替ポートを割り当てる必要があります。

      ```YAML
      be_port = vvvv                   # デフォルト: 9060
      be_http_port = xxxx              # デフォルト: 8040
      heartbeat_service_port = yyyy    # デフォルト: 9050
      brpc_port = zzzz                 # デフォルト: 8060
      ```

   b. クラスタでIPアドレスアクセスを有効にする場合は、構成ファイルに `priority_networks` 構成項目を追加し、CNノードに専用のIPアドレス（CIDR形式）を割り当てる必要があります。クラスタでFQDNアクセスを有効にする場合は、この構成項目を無視しても構いません。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > インスタンスで所有しているIPアドレスを表示するには、ターミナルで `ifconfig` を実行できます。

   c. インスタンスに複数のJDKがインストールされており、環境変数 `JAVA_HOME` で指定されているJDKとは異なる特定のJDKを使用したい場合は、構成ファイルに `JAVA_HOME` 構成項目を追加し、選択したJDKがインストールされているパスを指定する必要があります。

      ```YAML
      # <path_to_JDK>を選択したJDKがインストールされているパスに置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   BEのほとんどのパラメータはBEから継承されるため、詳細な手順については[パラメータ構成 - BE構成項目](../administration/Configuration.md#be-configuration-items)を参照してください。

2. CNノードを起動します。

   ```Bash
   ./be/bin/start_cn.sh --daemon
   ```

   > **注意**
   >
   > - FQDNアクセスが有効な状態でCNノードを起動する前に、**/etc/hosts** のすべてのインスタンスにホスト名が割り当てられていることを確認してください。詳細については、[環境構成チェックリスト - ホスト名](../deployment/environment_configurations.md#hostnames)を参照してください。
   > - CNノードを起動する際にパラメータ `--host_type` を指定する必要はありません。

3. CNノードが正常に起動したかどうかを確認するために、CNログをチェックします。

   ```Bash
   cat be/log/cn.INFO | grep heartbeat
   ```

   "I0313 15:03:45.820030 412450 thrift_server.cpp:375] heartbeat has started listening port on 9050" のようなログレコードが表示されれば、CNノードが正常に起動しています。

4. 他のインスタンスで前述の手順を繰り返すことで、新しいCNノードを起動することができます。

## ステップ4：クラスタの設定

すべてのFE、BEノード、およびCNノードが正常に起動した後、StarRocksクラスタを設定することができます。

以下の手順は、MySQLクライアントで実行されます。MySQLクライアント5.5.0以降がインストールされている必要があります。

1. MySQLクライアントを使用してStarRocksに接続します。初期アカウント `root` でログインし、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address>をリーダーFEノードのIPアドレス（priority_networks）またはFQDNに、
   # <query_port>（デフォルト: 9030）をfe.confで指定したquery_portに置き換えてください。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次のSQLを実行して、リーダーFEノードのステータスを確認します。

   ```SQL
   SHOW PROC '/frontends'\G
   ```

   例：

   ```Plain
   MySQL [(none)]> SHOW PROC '/frontends'\G
   *************************** 1. row ***************************
                Name: x.x.x.x_9010_1686810741121
                  IP: x.x.x.x
         EditLogPort: 9010
            HttpPort: 8030
           QueryPort: 9030
             RpcPort: 9020
                Role: LEADER
           ClusterId: 919351034
                Join: true
               Alive: true
   ReplayedJournalId: 1220
       LastHeartbeat: 2023-06-15 15:39:04
            IsHelper: true
              ErrMsg: 
           StartTime: 2023-06-15 14:32:28
             Version: 3.0.0-48f4d81
   1 row in set (0.01 sec)
   ```

   - フィールド `Alive` が `true` の場合、このFEノードは正常に起動され、クラスタに追加されています。
   - フィールド `Role` が `FOLLOWER` の場合、このFEノードはリーダーFEノードに選出される資格があります。
   - フィールド `Role` が `LEADER` の場合、このFEノードはリーダーFEノードです。

3. BEノードをクラスタに追加します。

   ```SQL
   -- <be_address>をBEノードのIPアドレス（priority_networks）またはFQDNに、
   -- <heartbeat_service_port>をbe.confで指定したheartbeat_service_port（デフォルト: 9050）に置き換えてください。
   ALTER SYSTEM ADD BACKEND "<be_address>:<heartbeat_service_port>";
   ```

   > **注意**
   >
   > 上記のコマンドを使用して一度に複数のBEノードを追加することができます。各 `<be_address>:<heartbeat_service_port>` のペアは1つのBEノードを表します。

4. BEノードのステータスを確認するために、次のSQLを実行します。

   ```SQL
   SHOW PROC '/backends'\G
   ```

   例：

   ```Plain
   MySQL [(none)]> SHOW PROC '/backends'\G
   *************************** 1. row ***************************
               BackendId: 10007
                      IP: 172.26.195.67
           HeartbeatPort: 9050
                  BePort: 9060
                HttpPort: 8040
                BrpcPort: 8060
           LastStartTime: 2023-06-15 15:23:08
           LastHeartbeat: 2023-06-15 15:57:30
                   Alive: true
    SystemDecommissioned: false
   ClusterDecommissioned: false
               TabletNum: 30
        DataUsedCapacity: 0.000 
           AvailCapacity: 341.965 GB
           TotalCapacity: 1.968 TB
                 UsedPct: 83.04 %
          MaxDiskUsedPct: 83.04 %
                  ErrMsg: 
                 Version: 3.0.0-48f4d81
                  Status: {"lastSuccessReportTabletsTime":"2023-06-15 15:57:08"}
       DataTotalCapacity: 341.965 GB
             DataUsedPct: 0.00 %
                CpuCores: 16
       NumRunningQueries: 0
              MemUsedPct: 0.01 %
              CpuUsedPct: 0.0 %
   ```

   フィールド `Alive` が `true` の場合、このBEノードは正常に起動され、クラスタに追加されています。

5. （オプション）CNノードをクラスタに追加します。

   ```SQL
   -- <cn_address>をCNノードのIPアドレス（priority_networks）またはFQDNに、
   -- <heartbeat_service_port>をcn.confで指定したheartbeat_service_port（デフォルト: 9050）に置き換えてください。
   ALTER SYSTEM ADD COMPUTE NODE "<cn_address>:<heartbeat_service_port>";
   ```

   > **注意**
   >
   > 1つのSQLで複数のCNノードを追加することができます。各 `<cn_address>:<heartbeat_service_port>` のペアは1つのCNノードを表します。

6. （オプション）CNノードのステータスを確認するために、次のSQLを実行します。

   ```SQL
   SHOW PROC '/compute_nodes'\G
   ```

   例：

   ```Plain
   MySQL [(none)]> SHOW PROC '/compute_nodes'\G
   *************************** 1. row ***************************
           ComputeNodeId: 10003
                      IP: x.x.x.x
           HeartbeatPort: 9050
                  BePort: 9060
                HttpPort: 8040
                BrpcPort: 8060
           LastStartTime: 2023-03-13 15:11:13
           LastHeartbeat: 2023-03-13 15:11:13
                   Alive: true
    SystemDecommissioned: false
   ClusterDecommissioned: false
                  ErrMsg: 
                 Version: 2.5.2-c3772fb
   1 row in set (0.00 sec)
   ```

   フィールド `Alive` が `true` の場合、このCNノードは正常に起動され、クラスタに追加されています。

   CNが正常に起動した後、クエリ中にCNを使用する場合は、システム変数 `SET prefer_compute_node = true;` および `SET use_compute_nodes = -1;` を設定します。詳細については、[システム変数](../reference/System_variable.md#descriptions-of-variables)を参照してください。

## ステップ5：（オプション）高可用性のFEクラスタをデプロイする

高可用性のFEクラスタをデプロイするには、StarRocksクラスタに少なくとも3つのFollower FEノードが必要です。リーダーFEノードが正常に起動した後、2つの新しいFEノードを起動して高可用性のFEクラスタをデプロイすることができます。

1. MySQLクライアントを使用してStarRocksに接続します。初期アカウント `root` でログインし、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address>をリーダーFEノードのIPアドレス（priority_networks）またはFQDNに、
   # <query_port>（デフォルト: 9030）をfe.confで指定したquery_portに置き換えてください。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次のSQLを実行して、新しいFollower FEノードをクラスタに追加します。

   ```SQL
   -- <fe2_address>を新しいFollower FEノードのIPアドレス（priority_networks）またはFQDNに、
   -- <edit_log_port>をfe.confで指定したedit_log_port（デフォルト: 9010）に置き換えてください。
   ALTER SYSTEM ADD FOLLOWER "<fe2_address>:<edit_log_port>";
   ```

   > **注意**
   >
   > - 1度に1つのFollower FEノードを追加するために上記のコマンドを使用することができます。
   > - Observer FEノードを追加する場合は、`ALTER SYSTEM ADD OBSERVER "<fe_address>:<edit_log_port>"=` を実行します。詳細な手順については、[ALTER SYSTEM - FE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を参照してください。

3. 新しいFollower FEノードを設定するために、新しいFEインスタンスでターミナルを起動し、メタデータの保存用に専用のディレクトリを作成し、StarRocks FEデプロイメントファイルを保存しているディレクトリに移動し、FE構成ファイル **fe/conf/fe.conf** を編集します。詳細な手順については、[ステップ1：リーダーFEノードを起動する](#ステップ1リーダーfeノードを起動する)を参照してください。基本的には、FEノードを起動するコマンド以外の手順をステップ1で繰り返すことができます。

   Follower FEノードの設定が完了したら、次のSQLを実行してFollower FEノードにヘルパーノードを割り当て、Follower FEノードを起動します。

   > **注意**
   >
   > クラスタに新しいFollower FEノードを追加する場合は、メタデータを同期するために新しいFollower FEノードに既存のFollower FEノード（ヘルパーノードとして機能する）を割り当てる必要があります。

   - IPアドレスアクセスが有効な状態で新しいFEノードを起動する場合は、次のコマンドを実行してFEノードを起動します。

     ```Bash
     # <helper_fe_ip>をリーダーFEノードのIPアドレス（priority_networks）に、
     # <helper_edit_log_port>（デフォルト: 9010）をリーダーFEノードのedit_log_portに置き換えてください。
     ./fe/bin/start_fe.sh --helper <helper_fe_ip>:<helper_edit_log_port> --daemon
     ```

     最初にノードを起動するときにパラメータ `--helper` を指定する必要があります。

   - FQDNアクセスが有効な状態で新しいFEノードを起動する場合は、次のコマンドを実行してFEノードを起動します。

     ```Bash
     # <helper_fqdn>をリーダーFEノードのFQDNに、
     # <helper_edit_log_port>（デフォルト: 9010）をリーダーFEノードのedit_log_portに置き換えてください。
     ./fe/bin/start_fe.sh --helper <helper_fqdn>:<helper_edit_log_port> \
           --host_type FQDN --daemon
     ```

     最初にノードを起動するときにパラメータ `--helper` と `--host_type` を指定する必要があります。

4. FEログをチェックして、FEノードが正常に起動したかどうかを確認します。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   "2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020." のようなログレコードが表示されれば、FEノードが正常に起動しています。

5. 前述の手順2、3、および4を繰り返して、新しいFollower FEノードをすべて正常に起動した後、MySQLクライアントから次のSQLを実行してFEノードのステータスを確認します。

   ```SQL
   SHOW PROC '/frontends'\G
   ```

   例：

   ```Plain
   MySQL [(none)]> SHOW PROC '/frontends'\G
   *************************** 1. row ***************************
                Name: x.x.x.x_9010_1686810741121
                  IP: x.x.x.x
         EditLogPort: 9010
            HttpPort: 8030
           QueryPort: 9030
             RpcPort: 9020
                Role: LEADER
           ClusterId: 919351034
                Join: true
               Alive: true
   ReplayedJournalId: 1220
       LastHeartbeat: 2023-06-15 15:39:04
            IsHelper: true
              ErrMsg: 
           StartTime: 2023-06-15 14:32:28
             Version: 3.0.0-48f4d81
   *************************** 2. row ***************************
                Name: x.x.x.x_9010_1686814080597
                  IP: x.x.x.x
         EditLogPort: 9010
            HttpPort: 8030
           QueryPort: 9030
             RpcPort: 9020
                Role: FOLLOWER
           ClusterId: 919351034
                Join: true
               Alive: true
   ReplayedJournalId: 1219
       LastHeartbeat: 2023-06-15 15:39:04
            IsHelper: true
              ErrMsg: 
           StartTime: 2023-06-15 15:38:53
             Version: 3.0.0-48f4d81
   *************************** 3. row ***************************
                Name: x.x.x.x_9010_1686814090833
                  IP: x.x.x.x
         EditLogPort: 9010
            HttpPort: 8030
           QueryPort: 9030
             RpcPort: 9020
                Role: FOLLOWER
           ClusterId: 919351034
                Join: true
               Alive: true
   ReplayedJournalId: 1219
       LastHeartbeat: 2023-06-15 15:39:04
            IsHelper: true
              ErrMsg: 
           StartTime: 2023-06-15 15:37:52
             Version: 3.0.0-48f4d81
   3 rows in set (0.02 sec)
   ```

   - フィールド `Alive` が `true` の場合、このFEノードは正常に起動され、クラスタに追加されています。
   - フィールド `Role` が `FOLLOWER` の場合、このFEノードはリーダーFEノードに選出される資格があります。
   - フィールド `Role` が `LEADER` の場合、このFEノードはリーダーFEノードです。

## StarRocksクラスタの停止

対応するインスタンスで次のコマンドを実行することで、StarRocksクラスタを停止することができます。

- FEノードを停止します。

  ```Bash
  ./fe/bin/stop_fe.sh --daemon
  ```

- BEノードを停止します。

  ```Bash
  ./be/bin/stop_be.sh --daemon
  ```

- CNノードを停止します。

  ```Bash
  ./be/bin/stop_cn.sh --daemon
  ```

## トラブルシューティング

FE、BE、またはCNノードを起動する際に発生するエラーを特定するために、次の手順を試してください。

- FEノードが正常に起動しない場合は、**fe/log/fe.warn.log** のログをチェックして問題を特定できます。

  ```Bash
  cat fe/log/fe.warn.log
  ```

  問題を特定して解決した後、現在のFEプロセスを終了し、既存の**meta**ディレクトリを削除し、新しいメタデータ保存ディレクトリを作成し、正しい構成でFEノードを再起動する必要があります。

- BEノードが正常に起動しない場合は、**be/log/be.WARNING** のログをチェックして問題を特定できます。

  ```Bash
  cat be/log/be.WARNING
  ```

  問題を特定して解決した後、既存のBEプロセスを終了し、既存の**storage**ディレクトリを削除し、新しいデータ保存ディレクトリを作成し、正しい構成でBEノードを再起動する必要があります。

- CNノードが正常に起動しない場合は、**be/log/cn.WARNING** のログをチェックして問題を特定できます。

  ```Bash
  cat be/log/cn.WARNING
  ```

  問題を特定して解決した後、既存のCNプロセスを終了し、正しい構成でCNノードを再起動する必要があります。

## 次の手順

StarRocksクラスタをデプロイした後は、[デプロイ後のセットアップ](../deployment/post_deployment_setup.md)に進んで、初期管理措置の手順を実行します。
