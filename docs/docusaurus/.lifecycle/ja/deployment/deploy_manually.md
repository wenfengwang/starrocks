---
displayed_sidebar: "Japanese"
---

# StarRocks を手動で展開する

このトピックでは、ストレージと計算の両方を担当する StarRocks (BE) を手動で展開する方法について説明します。他のインストールモードについては、[展開概要](../deployment/deployment_overview.md)を参照してください。

ストレージと計算を分離した共有データ型の StarRocks クラスタを展開する場合は、[共有データ型 StarRocks の展開と使用](../deployment/shared_data/s3.md)を参照してください。

## ステップ 1: リーダー FE ノードを起動する

以下の手順は FE インスタンスで実行されます。

1. メタデータの保存用に専用のディレクトリを作成します。メタデータは FE の展開ファイルとは別のディレクトリに保存することをお勧めします。このディレクトリが存在し、かつそのディレクトリへの書き込み権限を確認してください。

   ```YAML
   # <meta_dir> を作成したいメタデータのディレクトリに置き換えます。
   mkdir -p <meta_dir>
   ```

2. 以前に準備した [StarRocks FE 展開ファイル](../deployment/prepare_deployment_files.md) を保存しているディレクトリに移動し、FE 設定ファイル **fe/conf/fe.conf** を修正します。

   a. 構成項目 `meta_dir` でメタデータのディレクトリを指定します。

      ```YAML
      # <meta_dir> に作成したメタデータのディレクトリを置き換えます。
      meta_dir = <meta_dir>
      ```

   b. [環境構成チェックリスト](../deployment/environment_configurations.md#fe-ports) で言及されているいずれかの FE ポートが使用中の場合は、FE 設定ファイルで有効な代替ポートを割り当てる必要があります。

      ```YAML
      http_port = aaaa        # デフォルト: 8030
      rpc_port = bbbb         # デフォルト: 9020
      query_port = cccc       # デフォルト: 9030
      edit_log_port = dddd    # デフォルト: 9010
      ```

      > **注意**
      >
      > クラスタに複数の FE ノードを展開する場合は、各 FE ノードに同じ `http_port` を割り当てる必要があります。

   c. クラスタに IP アドレスアクセスを有効にする場合は、設定ファイルに構成項目 `priority_networks` を追加し、FE ノードに専用の IP アドレス（CIDR 形式で）を割り当てます。クラスタに[FQDN アクセス](../administration/enable_fqdn.md)を有効にしたい場合は、この構成項目を無視できます。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > ターミナルで `ifconfig` を実行して、インスタンスが所有する IP アドレスを表示できます。

   d. インスタンスに複数の JDK がインストールされており、環境変数 `JAVA_HOME` で指定された JDK とは異なる特定の JDK を使用したい場合は、設定ファイルに構成項目 `JAVA_HOME` を追加し、選択した JDK がインストールされているパスを指定する必要があります。

      ```YAML
      # <path_to_JDK> に選択した JDK がインストールされているパスを置き換えます。
      JAVA_HOME = <path_to_JDK>
      ```

   f.  高度な構成項目については、[パラメータ構成 - FE 構成項目](../administration/Configuration.md#fe-configuration-items)を参照してください。

3. FE ノードを起動します。

   - クラスタに IP アドレスアクセスを有効にするには、次のコマンドを実行して FE ノードを起動します。

     ```Bash
     ./fe/bin/start_fe.sh --daemon
     ```

   - クラスタに FQDN アクセスを有効にするには、次のコマンドを実行して FE ノードを起動します。

     ```Bash
     ./fe/bin/start_fe.sh --host_type FQDN --daemon
     ```

     FQDN アクセスを有効にするときは、ノードを初めて起動するときにのみパラメータ `--host_type` を1回だけ指定する必要があります。

     > **注意**
     >
     > FQDN アクセスが有効な状態で FE ノードを起動する前に、すべてのインスタンスにホスト名が割り当てられていることを確認してください。詳細については [環境構成チェックリスト - ホスト名](../deployment/environment_configurations.md#hostnames)を参照してください。

4. FE ログを確認して、FE ノードが正常に起動したかどうかを確認します。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   "2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020." のようなログが記錍されている場合、FE ノードが正常に起動しています。

## ステップ 2: BE サービスを起動する

以下の手順は BE インスタンスで実行されます。

1. データの保存用に専用のディレクトリを作成します。データは BE の展開ディレクトリとは別のディレクトリに保存することをお勧めします。このディレクトリが存在し、かつそのディレクトリへの書き込み権限を確認してください。

   ```YAML
   # <storage_root_path> を作成したいデータ保存ディレクトリに置き換えます。
   mkdir -p <storage_root_path>
   ```

2. 以前に準備した[StarRocks BE 展開ファイル](../deployment/prepare_deployment_files.md) を保存しているディレクトリに移動し、BE 設定ファイル **be/conf/be.conf** を修正します。

   a. 構成項目 `storage_root_path` でデータディレクトリを指定します。

      ```YAML
      # <storage_root_path> に作成したデータディレクトリを置き換えます。
      storage_root_path = <storage_root_path>
      ```

   b. [環境構成チェックリスト](../deployment/environment_configurations.md#be-ports) で言及されているいずれかの BE ポートが使用中の場合は、BE 設定ファイルで有効な代替ポートを割り当てる必要があります。

      ```YAML
      be_port = vvvv                   # デフォルト: 9060
      be_http_port = xxxx              # デフォルト: 8040
      heartbeat_service_port = yyyy    # デフォルト: 9050
      brpc_port = zzzz                 # デフォルト: 8060
      ```

   c. クラスタに IP アドレスアクセスを有効にする場合は、設定ファイルに構成項目 `priority_networks` を追加し、BE ノードに専用の IP アドレス（CIDR 形式で）を割り当てます。クラスタに FQDN アクセスを有効にしたい場合は、この構成項目を無視できます。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > ターミナルで `ifconfig` を実行して、インスタンスが所有する IP アドレスを表示できます。

   d. インスタンスに複数の JDK がインストールされており、環境変数 `JAVA_HOME` で指定された JDK とは異なる特定の JDK を使用したい場合は、設定ファイルに構成項目 `JAVA_HOME` を追加し、選択した JDK がインストールされているパスを指定する必要があります。

      ```YAML
      # <path_to_JDK> に選択した JDK がインストールされているパスを置き換えます。
      JAVA_HOME = <path_to_JDK>
      ```

   高度な構成項目については、[パラメータ構成 - BE 構成項目](../administration/Configuration.md#be-configuration-items)を参照してください。

3. BE ノードを起動します。

      ```Bash
      ./be/bin/start_be.sh --daemon
      ```

      > **注意**
      >
      > - FQDN アクセスを有効にして BE ノードを起動する前に、すべてのインスタンスにホスト名が割り当てられていることを確認してください。詳細については [環境構成チェックリスト - ホスト名](../deployment/environment_configurations.md#hostnames)を参照してください。
      > - BE ノードを起動する際には、パラメータ `--host_type` を指定する必要はありません。

4. BE ログを確認して、BE ノードが正常に起動したかどうかを確認します。

      ```Bash
      cat be/log/be.INFO | grep heartbeat
      ```

      "I0810 16:18:44.487284 3310141 task_worker_pool.cpp:1387] Waiting to receive first heartbeat from frontend" のようなログが記錍されている場合、BE ノードが正常に起動しています。

5. 他の BE インスタンスで上記手順を繰り返して新しい BE ノードを起動できます。

> **注意**
>
> 3つ以上の BE ノードが展開され、StarRocks クラスタに追加されると自動的に BE の高可用性クラスタが形成されます。

## ステップ 3: （オプション）CN サービスを起動する

コンピュートノード（CN）はデータを自己管理しない状態の計算サービスです。クエリの追加計算リソースを提供するために、CN ノードをクラスタに追加してオプションで利用できます。BE 展開ファイルで CN ノードを展開できます。Compute Nodes は v2.4 以降でサポートされています。

1. 以前に準備した[StarRocks BE 展開ファイル](../deployment/prepare_deployment_files.md) を保存しているディレクトリに移動し、CN 設定ファイル **be/conf/cn.conf** を修正します。

   a. [環境構成チェックリスト](../deployment/environment_configurations.md) で言及されているいずれかの CN ポートが使用中の場合は、CN 設定ファイルで有効な代替ポートを割り当てる必要があります。

      ```YAML
      be_port = vvvv                   # デフォルト: 9060
      be_http_port = xxxx              # デフォルト: 8040
      heartbeat_service_port = yyyy    # デフォルト: 9050
      brpc_port = zzzz                 # デフォルト: 8060
      ```

b. クラスターでIPアドレスアクセスを有効にしたい場合、構成ファイルに`priority_networks`の構成項目を追加し、CIDR形式で専用のIPアドレスをCNノードに割り当てる必要があります。クラスターでFQDNアクセスを有効にしたい場合は、この構成項目を無視できます。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > インスタンスが所有するIPアドレスを確認するには、ターミナルで`ifconfig`を実行できます。

   c. インスタンスに複数のJDKがインストールされており、環境変数`JAVA_HOME`で指定されているJDKとは異なる特定のJDKを使用したい場合、構成ファイルに`JAVA_HOME`の構成項目を追加し、選択したJDKがインストールされているパスを指定する必要があります。

      ```YAML
      # 選択したJDKがインストールされているパスで`<path_to_JDK>`を置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

      高度な構成項目の詳細については、BEの構成項目から[Parameter Configuration - BE configuration items](../administration/Configuration.md#be-configuration-items)を参照してください。ほとんどのCNのパラメータはBEから継承されます。

2. CNノードを起動してください。

   ```Bash
   ./be/bin/start_cn.sh --daemon
   ```

      > **注意**
      >
      > - FQDN アクセスが有効な状態でCN ノードを起動する前に、**/etc/hosts** にすべてのインスタンスのホスト名が割り当てられていることを確認してください。詳細については [Environment Configuration Checklist - Hostnames](../deployment/environment_configurations.md#hostnames) を参照してください。
      > - CNノードを起動するときにパラメータ`--host_type`を指定する必要はありません。

3. CNノードが正常に起動したかを確認するために、CNのログをチェックしてください。

   ```Bash
   cat be/log/cn.INFO | grep heartbeat
   ```

   "I0313 15:03:45.820030 412450 thrift_server.cpp:375] heartbeat has started listening port on 9050"のようなログが記録されている場合、CNノードが正常に起動しています。

4. 上記の手順を他のインスタンスでも繰り返して新しいCNノードを起動することができます。

## Step 4: クラスターのセットアップ

すべてのFEノード、BEノード、CNノードが正常に起動したら、StarRocksクラスターのセットアップを行うことができます。

以下の手順はMySQLクライアントで実行されます。MySQLクライアント 5.5.0 以降がインストールされている必要があります。

1. MySQLクライアントを使用してStarRocksに接続してください。初期アカウント `root` でログインする必要があり、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address>をLeader FEノードのIPアドレス（priority_networks）またはFQDNに置き換え、<query_port>（デフォルト：9030）をfe.confで指定したquery_portに置き換えてください。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次のSQLを実行してLeader FEノードの状態を確認してください。

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

   - `Alive`が`true`であれば、このFEノードは正常に起動され、クラスターに追加されています。
   - `Role`が`FOLLOWER`であれば、このFEノードはLeader FEノードとして選出される資格があります。
   - `Role`が`LEADER`であれば、このFEノードはLeader FEノードです。

3. BEノードをクラスターに追加してください。

   ```SQL
   -- <be_address>をBEノードのIPアドレス（priority_networks）またはFQDNに、<heartbeat_service_port>をbe.confで指定したheartbeat_service_port（デフォルト：9050）に置き換えてください。
   ALTER SYSTEM ADD BACKEND "<be_address>:<heartbeat_service_port>";
   ```

   > **注意**
   >
   > このコマンドを使用して複数のBEノードを一度に追加することができます。それぞれの`<be_address>:<heartbeat_service_port>`ペアは1つのBEノードを表します。

4. 次のSQLを実行してBEノードの状態を確認してください。

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

   `Alive`が`true`であれば、このBEノードは正常に起動され、クラスターに追加されています。

5. (オプション) CNノードをクラスターに追加してください。

   ```SQL
   -- <cn_address>をCNノードのIPアドレス（priority_networks）またはFQDNに、<heartbeat_service_port>をcn.confで指定したheartbeat_service_port（デフォルト：9050）に置き換えてください。
   ALTER SYSTEM ADD COMPUTE NODE "<cn_address>:<heartbeat_service_port>";
   ```

   > **注意**
   >
   > このSQLを使用して複数のCNノードを追加することができます。それぞれの`<cn_address>:<heartbeat_service_port>`ペアは1つのCNノードを表します。

6. (オプション) 次のSQLを実行してCNノードの状態を確認してください。

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

   `Alive`が`true`であれば、このCNノードは正常に起動され、クラスターに追加されています。

   CNが正常に起動したら、クエリ中にCNを使用する場合は、システム変数`SET prefer_compute_node = true;`と`SET use_compute_nodes = -1;`を設定してください。詳細については、[System variables](../reference/System_variable.md#descriptions-of-variables)を参照してください。

## ステップ5：(オプション) 高可用性のFEクラスターをデプロイする

高可用性のFEクラスターには、StarRocksクラスターに少なくとも3つのFollower FEノードが必要です。Leader FEノードが正常に起動したら、新しいFEノードを2つ起動して高可用性のFEクラスターをデプロイすることができます。

1. MySQLクライアントを使用してStarRocksに接続してください。初期アカウント `root` でログインする必要があり、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address>をLeader FEノードのIPアドレス（priority_networks）またはFQDNに置き換え、<query_port>（デフォルト：9030）をfe.confで指定したquery_portに置き換えてください。
```bash
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次のSQLを実行して新しいフォロワーFEノードをクラスタに追加します。

   ```SQL
   -- <fe_address>を新しいフォロワーFEノードのIPアドレス（priority_networks）または FQDNに置き換え、<edit_log_port>をfe.confで指定したedit_log_port（デフォルト：9010）に置き換えます。
   ALTER SYSTEM ADD FOLLOWER "<fe2_address>:<edit_log_port>";
   ```

   > **注意**
   >
   > - 各回、1つのフォロワーFEノードのみを追加できます。
   > - オブザーバーFEノードを追加する場合は、`ALTER SYSTEM ADD OBSERVER "<fe_address>:<edit_log_port>"=`を実行してください。詳細な手順については、[ALTER SYSTEM - FE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を参照してください。

3. 新しいFEインスタンスでターミナルを起動し、メタデータ保存用の専用ディレクトリを作成し、StarRocks FEデプロイメントファイルを保存しているディレクトリに移動し、FE構成ファイル **fe/conf/fe.conf** を変更します。詳細な手順については、[Step 1: Start the Leader FE node](#step-1-start-the-leader-fe-node)を参照してください。基本的には、FEノードを起動するために使用されるコマンドを除いて、Step 1の手順を繰り返すことができます。
  
   フォロワーFEノードの構成が完了したら、次のSQLを実行してフォロワーFEノードのためのヘルパーノードを割り当て、フォロワーFEノードを起動します。

   > **注意**
   >
   > クラスタに新しいフォロワーFEノードを追加する場合、メタデータを同期するために、ヘルパーノード（基本的に既存のフォロワーFEノード）を新しいフォロワーFEノードに割り当てる必要があります。

   - IPアドレスアクセスで新しいFEノードを起動するには、次のコマンドを実行します。

     ```Bash
     # <helper_fe_ip>をリーダーFEノードのIPアドレス（priority_networks）に置き換え、<helper_edit_log_port>（デフォルト：9010）をリーダーFEノードのedit_log_portに置き換えます。
    ./fe/bin/start_fe.sh --helper <helper_fe_ip>:<helper_edit_log_port> --daemon
     ```

     この際、初めてノードを起動するときにパラメータ `--helper` を1回だけ指定する必要があります。

   - FQDNアクセスで新しいFEノードを起動するには、次のコマンドを実行します。

     ```Bash
     # <helper_fqdn>をリーダーFEノードのFQDNに置き換え、<helper_edit_log_port>（デフォルト：9010）をリーダーFEノードのedit_log_portに置き換えます。
     ./fe/bin/start_fe.sh --helper <helper_fqdn>:<helper_edit_log_port> \
           --host_type FQDN --daemon
     ```

     この際、初めてノードを起動するときにパラメータ `--helper` および `--host_type` を1回だけ指定する必要があります。

4. FEログを確認して、FEノードが正常に起動したかどうかを確認します。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   "2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020."のようなログが記録されている場合、FEノードが正常に起動していることを示します。

5. 前述の手順2、3、4を繰り返し、すべての新しいフォロワーFEノードが正常に起動したことを確認した後、MySQLクライアントから次のSQLを実行してFEノードのステータスを確認します。

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
   - フィールド `Role` が `FOLLOWER` の場合、このFEノードはリーダーFEノードとして選出される資格があります。
   - フィールド `Role` が `LEADER` の場合、このFEノードはリーダーFEノードです。

## StarRocksクラスタを停止する

対応するインスタンスで次のコマンドを実行することで、StarRocksクラスタを停止できます。

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

FE、BE、またはCNノードを起動した際に発生したエラーを特定するために、次の手順を試してみてください。

- FEノードが正常に起動しない場合は、**fe/log/fe.warn.log**でログを確認して問題を特定できます。

  ```Bash
  cat fe/log/fe.warn.log
  ```

  問題を特定して解決したら、現在のFEプロセスを終了し、既存の **meta** ディレクトリを削除し、新しいメタデータ保存ディレクトリを作成し、正しい構成でFEノードを再起動する必要があります。

- BEノードが正常に起動しない場合は、**be/log/be.WARNING**でログを確認して問題を特定できます。

  ```Bash
  cat be/log/be.WARNING
  ```

  問題を特定して解決したら、既存のBEプロセスを終了し、既存の **storage** ディレクトリを削除し、新しいデータ保存ディレクトリを作成し、正しい構成でBEノードを再起動する必要があります。

- CNノードが正常に起動しない場合は、**be/log/cn.WARNING**でログを確認して問題を特定できます。

  ```Bash
  cat be/log/cn.WARNING
  ```

  問題を特定して解決したら、既存のCNプロセスを終了し、正しい構成でCNノードを再起動する必要があります。

## 次に何をすればよいですか

StarRocksクラスタを展開した後、[Post-deployment Setup](../deployment/post_deployment_setup.md)に進んで、初期の管理措置に関する手順を確認できます。