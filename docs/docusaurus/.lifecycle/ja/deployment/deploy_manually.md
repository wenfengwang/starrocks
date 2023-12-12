---
displayed_sidebar: "Japanese"
---

# StarRocksを手動で展開する

このトピックでは、ストレージと計算の両方をFEが担当する共有なしのStarRocksを手動で展開する方法について説明します。その他のインストールモードについては、[デプロイ概要](../deployment/deployment_overview.md)を参照してください。

共有データStarRocksクラスタ（分離されたストレージと計算）を展開する場合は、[共有データStarRocksの展開と使用](../deployment/shared_data/s3.md)を参照してください。

## ステップ1: リーダーFEノードの起動

以下の手順は、FEインスタンスで実行されます。

1. メタデータの保存に専用のディレクトリを作成します。メタデータはFE展開ファイルとは別のディレクトリに保存することをお勧めします。このディレクトリが存在し、書き込みアクセス権があることを確認してください。

   ```YAML
   # 作成したいメタデータディレクトリに <meta_dir> を置き換えてください。
   mkdir -p <meta_dir>
   ```

2. 以前に準備した[StarRocks FE展開ファイル](../deployment/prepare_deployment_files.md)を保存しているディレクトリに移動し、FE構成ファイル**fe/conf/fe.conf**を修正します。

   a. 構成項目 `meta_dir` でメタデータディレクトリを指定します。

      ```YAML
      # 作成したメタデータディレクトリに <meta_dir> を置き換えてください。
      meta_dir = <meta_dir>
      ```

   b. [環境構成チェックリスト](../deployment/environment_configurations.md#fe-ports)で言及されているいずれかのFEポートが使用中の場合は、FE構成ファイルで有効な代替を割り当てる必要があります。

      ```YAML
      http_port = aaaa        # デフォルト: 8030
      rpc_port = bbbb         # デフォルト: 9020
      query_port = cccc       # デフォルト: 9030
      edit_log_port = dddd    # デフォルト: 9010
      ```

      > **注意**
      >
      > クラスタで複数のFEノードを展開する場合は、各FEノードに同じ `http_port` を割り当てる必要があります。

   c. クラスタでIPアドレスアクセスを有効にする場合は、構成ファイルに構成項目 `priority_networks` を追加し、FEノードに専用のIPアドレス（CIDR形式）を割り当てる必要があります。クラスタで[FQDNアクセス](../administration/enable_fqdn.md)を有効にしたい場合は、この構成項目を無視してください。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注**
      >
      > インスタンスで所有しているIPアドレスを表示するには、ターミナルで `ifconfig` を実行できます。

   d. インスタンスに複数のJDKがインストールされており、環境変数 `JAVA_HOME` で指定されているJDKと異なる特定のJDKを使用したい場合は、構成ファイルに構成項目 `JAVA_HOME` を追加し、選択したJDKがインストールされているパスを指定する必要があります。

      ```YAML
      # 選択したJDKがインストールされているパスに <path_to_JDK> を置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   f.  高度な構成項目についての情報は、[パラメータ構成 - FE構成項目](../administration/Configuration.md#fe-configuration-items)を参照してください。

3. FEノードを起動します。

   - クラスタでIPアドレスアクセスを有効にする場合は、次のコマンドを実行してFEノードを起動します。

     ```Bash
     ./fe/bin/start_fe.sh --daemon
     ```

   - クラスタでFQDNアクセスを有効にする場合は、次のコマンドを実行してFEノードを起動します。

     ```Bash
     ./fe/bin/start_fe.sh --host_type FQDN --daemon
     ```

     最初にノードを起動する際に `--host_type` パラメータを一度だけ指定する必要があります。

     > **注意**
     >
     > FQDNアクセスを有効にしてFEノードを起動する前に、すべてのインスタンスにホスト名を割り当てていることを確認してください。詳細については、**/etc/hosts** の[環境構成チェックリスト - ホスト名](../deployment/environment_configurations.md#hostnames)を参照してください。

4. FEログを確認して、FEノードが正常に起動されているかどうかを確認します。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   "2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020." のようなログが記錍されている場合、FEノードが正常に起動されています。

## ステップ2: BEサービスの開始

以下の手順は、BEインスタンスで実行されます。

1. データ保存用に専用のディレクトリを作成します。データをBE展開ディレクトリとは別のディレクトリに保存することをお勧めします。このディレクトリが存在し、書き込みアクセス権があることを確認してください。

   ```YAML
   # 作成したいデータ保存ディレクトリに <storage_root_path> を置き換えてください。
   mkdir -p <storage_root_path>
   ```

2. 以前に準備した[StarRocks BE展開ファイル](../deployment/prepare_deployment_files.md)を保存しているディレクトリに移動し、BE構成ファイル**be/conf/be.conf**を修正します。

   a. 構成項目 `storage_root_path` でデータディレクトリを指定します。

      ```YAML
      # 作成したデータディレクトリに <storage_root_path> を置き換えてください。
      storage_root_path = <storage_root_path>
      ```

   b. [環境構成チェックリスト](../deployment/environment_configurations.md#be-ports)で言及されているいずれかのBEポートが使用中の場合は、BE構成ファイルで有効な代替を割り当てる必要があります。

      ```YAML
      be_port = vvvv                   # デフォルト: 9060
      be_http_port = xxxx              # デフォルト: 8040
      heartbeat_service_port = yyyy    # デフォルト: 9050
      brpc_port = zzzz                 # デフォルト: 8060
      ```

   c. クラスタでIPアドレスアクセスを有効にする場合は、構成ファイルに構成項目 `priority_networks` を追加し、BEノードに専用のIPアドレス（CIDR形式）を割り当てる必要があります。クラスタでFQDNアクセスを有効にしたい場合は、この構成項目を無視してください。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注**
      >
      > インスタンスで所有しているIPアドレスを表示するには、ターミナルで `ifconfig` を実行できます。

   d. インスタンスに複数のJDKがインストールされており、環境変数 `JAVA_HOME` で指定されているJDKと異なる特定のJDKを使用したい場合は、構成ファイルに構成項目 `JAVA_HOME` を追加し、選択したJDKがインストールされているパスを指定する必要があります。

      ```YAML
      # 選択したJDKがインストールされているパスに <path_to_JDK> を置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   詳細な構成項目については、[パラメータ構成 - BE構成項目](../administration/Configuration.md#be-configuration-items)を参照してください。

3. BEノードを起動します。

      ```Bash
      ./be/bin/start_be.sh --daemon
      ```

      > **注意**
      >
      > - FQDNアクセスを有効にしてBEノードを起動する前に、すべてのインスタンスにホスト名を割り当てていることを確認してください。詳細については、[環境構成チェックリスト - ホスト名](../deployment/environment_configurations.md#hostnames)を参照してください。
      > - BEノードを起動する際に `--host_type` パラメータを指定する必要はありません。

4. BEログを確認して、BEノードが正常に起動されているかどうかを確認します。

      ```Bash
      cat be/log/be.INFO | grep heartbeat
      ```

      "I0810 16:18:44.487284 3310141 task_worker_pool.cpp:1387] Waiting to receive first heartbeat from frontend" のようなログが記錍されている場合、BEノードが正常に起動されています。

5. 上記の手順を他のBEインスタンスでも繰り返すことで、新しいBEノードを起動できます。

> **注**
>
> 3つ以上のBEノードが展開され、StarRocksクラスタに追加されると、BEの高可用性クラスタが自動的に形成されます。

## ステップ3: （オプション）CNサービスの開始

コンピュートノード（CN）は自身でデータを保持しない状態の計算サービスです。クエリの追加計算リソースを提供するために、クラスタにオプションでCNノードを追加できます。CNノードはv2.4からサポートされています。

1. 以前に準備した[StarRocks BE展開ファイル](../deployment/prepare_deployment_files.md)を保存しているディレクトリに移動し、CN構成ファイル**be/conf/cn.conf**を修正します。

   a. [環境構成チェックリスト](../deployment/environment_configurations.md)で言及されているいずれかのCNポートが使用中の場合は、CN構成ファイルで有効な代替を割り当てる必要があります。

      ```YAML
      be_port = vvvv                   # デフォルト: 9060
      be_http_port = xxxx              # デフォルト: 8040
      heartbeat_service_port = yyyy    # デフォルト: 9050
      brpc_port = zzzz                 # デフォルト: 8060
      ```


```yaml
      # 応急ネットワークにアクセスする場合は、「priority_networks」構成項目を構成ファイルに追加し、CIDR形式の専用IPアドレスをCNノードに割り当てる必要があります。「FQDN」アクセスを有効にする場合は、この構成項目を無視できます。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > インスタンスで所有するIPアドレスを表示するには、ターミナルで `ifconfig` を実行できます。

   c. インスタンスに複数のJDKがインストールされており、`JAVA_HOME`環境変数で指定されているJDKとは異なる特定のJDKを使用する場合は、選択したJDKがインストールされているパスを構成ファイルに追加することで、`JAVA_HOME`構成項目を指定する必要があります。

      ```YAML
      # 選択したJDKがインストールされているパスで <path_to_JDK> を置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   CNのパラメータの多くはBEから継承されるため、詳細な構成項目については[パラメータの構成 - BE構成項目](../administration/Configuration.md#be-configuration-items)を参照してください。

2. CNノードを起動します。

   ```Bash
   ./be/bin/start_cn.sh --daemon
   ```

   > **注意**
   >
   > - FQDNアクセスを有効にしてCNノードを起動する前に、**/etc/hosts** にすべてのインスタンスのホスト名が割り当てられていることを確認してください。詳細については[環境構成チェックリスト - ホスト名](../deployment/environment_configurations.md#hostnames)を参照してください。
   > - CNノードを起動する際にパラメータ `--host_type` を指定する必要はありません。

3. CNログを確認し、CNノードが正常に開始されたかを確認します。

   ```Bash
   cat be/log/cn.INFO | grep heartbeat
   ```

   「I0313 15:03:45.820030 412450 thrift_server.cpp:375] heartbeat has started listening port on 9050」といったログ記録がある場合、CNノードが適切に開始されていることを示します。

4. 上記手順を他のインスタンスでも繰り返すことで、新しいCNノードを開始できます。

## ステップ4: クラスターのセットアップ

すべてのFEおよびBEノード、CNノードが正常に開始された後、StarRocksクラスターをセットアップできます。

以下の手順はMySQLクライアントで実行します。MySQLクライアント 5.5.0以降がインストールされている必要があります。

1. MySQLクライアントを使用してStarRocksに接続します。初期アカウント `root` でログインし、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address>をLeader FEノードのIPアドレス（priority_networks）またはFQDNに置き換え、<query_port>（デフォルト: 9030）をfe.confで指定したquery_portに置き換えてください。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次のSQLを実行してLeader FEノードのステータスをチェックします。

   ```SQL
   SHOW PROC '/frontends'\G
   ```

   例:

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

   - `Alive` フィールドが `true` の場合、このFEノードは正常に起動し、クラスターに追加されています。
   - `Role` フィールドが `FOLLOWER` の場合、このFEノードはLeader FEノードに選出される資格があります。
   - `Role` フィールドが `LEADER` の場合、このFEノードはLeader FEノードです。

3. 次のSQLを実行してBEノードをクラスターに追加します。

   ```SQL
   -- <be_address>をBEノードのIPアドレス（priority_networks）またはFQDNに置き換え、<heartbeat_service_port>（デフォルト: 9050）をbe.confで指定したheartbeat_service_portに置き換えてください。
   ALTER SYSTEM ADD BACKEND "<be_address>:<heartbeat_service_port>";
   ```

   > **注意**
   >
   > 複数のBEノードを一度に追加する場合は、前述のコマンドを使用できます。それぞれの `<be_address>:<heartbeat_service_port>` ペアは1つのBEノードを表します。

4. 次のSQLを実行してBEノードのステータスを確認します。

   ```SQL
   SHOW PROC '/backends'\G
   ```

   例:

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

   `Alive` フィールドが `true` の場合、このBEノードは正常に起動し、クラスターに追加されています。

5. (オプション) CNノードをクラスターに追加します。

   ```SQL
   -- <cn_address>をCNノードのIPアドレス（priority_networks）またはFQDNに置き換え、<heartbeat_service_port>（デフォルト: 9050）をcn.confで指定したheartbeat_service_portに置き換えてください。
   ALTER SYSTEM ADD COMPUTE NODE "<cn_address>:<heartbeat_service_port>";
   ```

   > **注意**
   >
   > 1つのSQLで複数のCNノードを追加できます。それぞれの `<cn_address>:<heartbeat_service_port>` ペアは1つのCNノードを表します。

6. (オプション) 次のSQLを実行してCNノードのステータスを確認します。

   ```SQL
   SHOW PROC '/compute_nodes'\G
   ```

   例:

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

   `Alive` フィールドが `true` の場合、このCNノードは正常に起動し、クラスターに追加されています。

   CNが正常に起動した後、クエリ実行中にCNを使用する場合は、システム変数 `SET prefer_compute_node = true;` と `SET use_compute_nodes = -1;` を設定してください。詳細については[システム変数](../reference/System_variable.md#descriptions-of-variables)を参照してください。

## ステップ5: (オプション) ハイアベイラビリティFEクラスターをデプロイする

ハイアベイラビリティFEクラスターには、StarRocksクラスターに少なくとも3つのFollower FEノードが必要です。Leader FEノードが正常に開始された後、2つの新しいFEノードを開始して、ハイアベイラビリティFEクラスターをデプロイできます。

1. MySQLクライアントを使用してStarRocksに接続します。初期アカウント `root` でログインし、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address>をLeader FEノードのIPアドレス（priority_networks）またはFQDNに置き換え、<query_port>（デフォルト: 9030）をfe.confで指定したquery_portに置き換えてください。
```
```
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 以下のSQLを実行して新しいFollower FEノードをクラスタに追加します。

   ```SQL
   -- <fe_address>を新しいFollower FEノードのIPアドレス（priority_networks）またはFQDNに置き換え、<edit_log_port>をfe.confで指定したedit_log_port（デフォルト: 9010）に置き換えます。
   ALTER SYSTEM ADD FOLLOWER "<fe2_address>:<edit_log_port>";
   ```

   > **注**
   >
   > - この前のコマンドは、1回につき1つのFollower FEノードを追加するために使用できます。
   > - Observer FEノードを追加する場合は、`ALTER SYSTEM ADD OBSERVER "<fe_address>:<edit_log_port>"=`を実行してください。詳細な手順については、[ALTER SYSTEM - FE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を参照してください。

3. 新しいFEインスタンスでターミナルを起動し、メタデータストレージ用の専用ディレクトリを作成し、StarRocks FEデプロイファイルを保存しているディレクトリに移動し、FE構成ファイル**fe/conf/fe.conf**を変更します。詳細な手順については、[Step 1: Start the Leader FE node](#step-1-start-the-leader-fe-node)を参照してください。基本的には、FEノードを起動するために使用されるコマンド以外の手順を繰り返すことができます。

   Follower FEノードの構成が完了したら、次のSQLを実行してFollower FEノードにヘルパーノードを割り当て、Follower FEノードを起動します。

   > **注**
   >
   > クラスタに新しいFollower FEノードを追加する場合、メタデータを同期するために、ヘルパーノード（基本的に既存のFollower FEノード）を新しいFollower FEノードに割り当てる必要があります。

   - IPアドレスアクセスで新しいFEノードを起動するには、次のコマンドを実行してFEノードを起動します。

     ```Bash
     # <helper_fe_ip>を、リーダーFEノードのIPアドレス（priority_networks）に置き換え、<helper_edit_log_port>（デフォルト: 9010）をリーダーFEノードのedit_log_portに置き換えます。
     ./fe/bin/start_fe.sh --helper <helper_fe_ip>:<helper_edit_log_port> --daemon
     ```

     初回にノードを起動する際に`--helper`パラメータを1度だけ指定する必要があることに注意してください。

   - FQDNアクセスで新しいFEノードを起動するには、次のコマンドを実行してFEノードを起動します。

     ```Bash
     # <helper_fqdn>をリーダーFEノードのFQDNに置き換え、<helper_edit_log_port>（デフォルト: 9010）をリーダーFEノードのedit_log_portに置き換えます。
     ./fe/bin/start_fe.sh --helper <helper_fqdn>:<helper_edit_log_port> \
           --host_type FQDN --daemon
     ```

     初回にノードを起動する際に`--helper`および`--host_type`パラメータを1度だけ指定する必要があることに注意してください。

4. FEログを確認して、FEノードが正常に起動したかどうかを確認します。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   "2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020."のようなログがある場合、FEノードが正常に起動していることを示します。

5. 新しいFollower FEノードを正常に起動するまで、前述の手順2、3、4を繰り返し、その後、MySQLクライアントから次のSQLを実行してFEノードのステータスを確認します。

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

   - `Alive`フィールドが`true`であれば、このFEノードは正常に起動され、クラスタに追加されています。
   - `Role`フィールドが`FOLLOWER`であれば、このFEノードはLeader FEノードとして選出する資格があります。
   - `Role`フィールドが`LEADER`であれば、このFEノードはLeader FEノードです。

## StarRocksクラスタの停止

対応するインスタンスで次のコマンドを実行してStarRocksクラスタを停止できます。

- FEノードの停止。

  ```Bash
  ./fe/bin/stop_fe.sh --daemon
  ```

- BEノードの停止。

  ```Bash
  ./be/bin/stop_be.sh --daemon
  ```

- CNノードの停止。

  ```Bash
  ./be/bin/stop_cn.sh --daemon
  ```

## トラブルシューティング

FE、BE、またはCNノードを起動する際に発生するエラーを特定するには、次の手順を試してみてください。

- FEノードが正常に起動しない場合は、そのログを確認して問題を特定できます。**fe/log/fe.warn.log**を確認してください。

  ```Bash
  cat fe/log/fe.warn.log
  ```

  問題を特定して解決したら、現在のFEプロセスを最初に終了し、既存の**meta**ディレクトリを削除し、新しいメタデータストレージディレクトリを作成し、正しい構成でFEノードを再起動する必要があります。

- BEノードが正常に起動しない場合は、そのログを確認して問題を特定できます。**be/log/be.WARNING**を確認してください。

  ```Bash
  cat be/log/be.WARNING
  ```

  問題を特定して解決したら、既存のBEプロセスを最初に終了し、既存の**storage**ディレクトリを削除し、新しいデータストレージディレクトリを作成し、正しい構成でBEノードを再起動する必要があります。

- CNノードが正常に起動しない場合は、そのログを確認して問題を特定できます。**be/log/cn.WARNING**を確認してください。

  ```Bash
  cat be/log/cn.WARNING
  ```

  問題を特定して解決したら、既存のCNプロセスを最初に終了し、正しい構成でCNノードを再起動する必要があります。

## 次の手順

StarRocksクラスタを展開した後は、初期の管理措置に関する手順については[Post-deployment Setup](../deployment/post_deployment_setup.md)を参照してください。