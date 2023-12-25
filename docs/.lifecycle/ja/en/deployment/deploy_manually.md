---
displayed_sidebar: English
---

# StarRocks の手動デプロイ

このトピックでは、シェアード・ナッシングの StarRocks（BE がストレージとコンピューティングの両方を担当する）を手動でデプロイする方法について説明します。その他のインストールモードについては、[デプロイメントの概要](../deployment/deployment_overview.md)を参照してください。

共有データ StarRocks クラスター（ストレージとコンピューティングの分離）をデプロイするには、[共有データ StarRocks のデプロイと使用](../deployment/shared_data/s3.md)を参照してください。

## ステップ 1: リーダー FE ノードの起動

次の手順は、FE インスタンスで実行されます。

1. メタデータストレージ専用のディレクトリを作成します。メタデータは、FE デプロイメントファイルとは別のディレクトリに格納することをお勧めします。このディレクトリが存在し、書き込みアクセス権があることを確認してください。

   ```YAML
   # <meta_dir> を作成したいメタデータディレクトリに置き換えてください。
   mkdir -p <meta_dir>
   ```

2. 事前に準備した [StarRocks FE デプロイメントファイル](../deployment/prepare_deployment_files.md)が格納されているディレクトリに移動し、FE 設定ファイル **fe/conf/fe.conf** を変更します。

   a. `meta_dir` の設定項目でメタデータディレクトリを指定します。

      ```YAML
      # <meta_dir> を作成したメタデータディレクトリに置き換えてください。
      meta_dir = <meta_dir>
      ```

   b. [環境設定チェックリスト](../deployment/environment_configurations.md#fe-ports)に記載されている FE ポートのいずれかが占有されている場合は、FE 設定ファイルで有効な代替ポートを割り当てる必要があります。

      ```YAML
      http_port = aaaa        # デフォルト: 8030
      rpc_port = bbbb         # デフォルト: 9020
      query_port = cccc       # デフォルト: 9030
      edit_log_port = dddd    # デフォルト: 9010
      ```

      > **注意**
      >
      > クラスタに複数の FE ノードをデプロイする場合は、各 FE ノードに同じ `http_port` を割り当てる必要があります。

   c. クラスタの IP アドレスアクセスを有効にする場合は、設定ファイルに `priority_networks` の設定項目を追加し、FE ノードに専用 IP アドレス（CIDR 形式）を割り当てる必要があります。クラスターの [FQDN アクセス](../administration/enable_fqdn.md)を有効にする場合は、この設定項目を無視できます。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注記**
      >
      > インスタンスが所有する IP アドレスを表示するには、ターミナルで `ifconfig` を実行できます。

   d. インスタンスに複数の JDK がインストールされており、環境変数 `JAVA_HOME` で指定されているものとは異なる JDK を使用したい場合は、選択した JDK がインストールされているパスを指定するために、設定ファイルに `JAVA_HOME` の設定項目を追加する必要があります。

      ```YAML
      # <path_to_JDK> を選択した JDK がインストールされているパスに置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   f. 高度な設定項目については、[パラメータ設定 - FE 設定項目](../administration/FE_configuration.md#fe-configuration-items)を参照してください。

3. FE ノードを起動します。

   - クラスタの IP アドレスアクセスを有効にするには、次のコマンドを実行して FE ノードを起動します。

     ```Bash
     ./fe/bin/start_fe.sh --daemon
     ```

   - クラスタの FQDN アクセスを有効にするには、次のコマンドを実行して FE ノードを起動します。

     ```Bash
     ./fe/bin/start_fe.sh --host_type FQDN --daemon
     ```

     ノードを初めて起動するときにのみ、`--host_type` パラメーターを指定する必要があることに注意してください。

     > **注意**
     >
     > FQDN アクセスを有効にして FE ノードを起動する前に、**/etc/hosts** 内のすべてのインスタンスにホスト名を割り当てていることを確認してください。詳細については、[環境設定チェックリスト - ホスト名](../deployment/environment_configurations.md#hostnames)を参照してください。

4. FE ログを確認して、FE ノードが正常に起動しているかどうかを検証します。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   「2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020.」というログが記録されていれば、FE ノードが正常に起動していることを示しています。

## ステップ 2: BE サービスの開始

次の手順は、BE インスタンスで実行されます。

1. データストレージ専用のディレクトリを作成します。データは、BE デプロイメントディレクトリとは別のディレクトリに保管することをお勧めします。このディレクトリが存在し、書き込みアクセス権があることを確認してください。

   ```YAML
   # <storage_root_path> を作成したいデータストレージディレクトリに置き換えてください。
   mkdir -p <storage_root_path>
   ```

2. 事前に準備した [StarRocks BE デプロイメントファイル](../deployment/prepare_deployment_files.md)が格納されているディレクトリに移動し、BE 設定ファイル **be/conf/be.conf** を変更します。

   a. `storage_root_path` の設定項目でデータディレクトリを指定します。

      ```YAML
      # <storage_root_path> を作成したデータディレクトリに置き換えてください。
      storage_root_path = <storage_root_path>
      ```

   b. [環境設定チェックリスト](../deployment/environment_configurations.md#be-ports)に記載されている BE ポートのいずれかが占有されている場合は、BE 設定ファイルで有効な代替ポートを割り当てる必要があります。

      ```YAML
      be_port = vvvv                   # デフォルト: 9060
      be_http_port = xxxx              # デフォルト: 8040
      heartbeat_service_port = yyyy    # デフォルト: 9050
      brpc_port = zzzz                 # デフォルト: 8060
      ```

   c. クラスタの IP アドレスアクセスを有効にする場合は、設定ファイルに `priority_networks` の設定項目を追加し、専用 IP アドレス（CIDR 形式）を BE ノードに割り当てる必要があります。クラスターの FQDN アクセスを有効にする場合は、この設定項目を無視できます。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注記**
      >
      > インスタンスが所有する IP アドレスを表示するには、ターミナルで `ifconfig` を実行できます。

   d. インスタンスに複数の JDK がインストールされており、環境変数 `JAVA_HOME` で指定されているものとは異なる JDK を使用したい場合は、選択した JDK がインストールされているパスを指定するために、設定ファイルに `JAVA_HOME` の設定項目を追加する必要があります。

      ```YAML
      # <path_to_JDK> を選択した JDK がインストールされているパスに置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   高度な設定項目については、[パラメータ設定 - BE 設定項目](../administration/BE_configuration.md#be-configuration-items)を参照してください。

3. BE ノードを開始します。

      ```Bash
      ./be/bin/start_be.sh --daemon
      ```

      > **注意**
      >
      > - FQDN アクセスを有効にして BE ノードを起動する前に、**/etc/hosts** 内のすべてのインスタンスにホスト名が割り当てられていることを確認してください。詳細については、[環境設定チェックリスト - ホスト名](../deployment/environment_configurations.md#hostnames)を参照してください。
      > - BE ノードを開始する際に `--host_type` パラメーターを指定する必要はありません。

4. BE ログを確認して、BE ノードが正常に開始されたかどうかを検証します。

      ```Bash
      cat be/log/be.INFO | grep heartbeat
      ```

      「I0810 16:18:44.487284 3310141 task_worker_pool.cpp:1387] Waiting to receive first heartbeat from frontend」というログが記録されていれば、BE ノードが正常に開始されていることを示しています。

5. 他の BE インスタンスで上記の手順を繰り返すことで、新しい BE ノードを開始できます。

> **注記**
>

> 少なくとも3つのBEノードがデプロイされ、StarRocksクラスタに追加されると、BEの高可用性クラスタが自動的に形成されます。

## ステップ 3: (オプション) CNサービスを開始する

コンピュートノード（CN）は、データ自体を保持しないステートレスなコンピューティングサービスです。必要に応じて、クエリ用の追加コンピューティングリソースを提供するために、クラスターにCNノードを追加することができます。CNノードはBEデプロイメントファイルと共にデプロイできます。コンピュートノードはv2.4以降でサポートされています。

1. 以前に準備した[StarRocks BEデプロイメントファイル](../deployment/prepare_deployment_files.md)が保存されているディレクトリに移動し、CN設定ファイル **be/conf/cn.conf** を変更します。

   a. [環境設定チェックリスト](../deployment/environment_configurations.md)に記載されているCNポートが使用中の場合は、CN設定ファイルで有効な代替ポートを割り当てる必要があります。

      ```YAML
      be_port = vvvv                   # デフォルト: 9060
      be_http_port = xxxx              # デフォルト: 8040
      heartbeat_service_port = yyyy    # デフォルト: 9050
      brpc_port = zzzz                 # デフォルト: 8060
      ```

   b. クラスターでIPアドレスアクセスを有効にしたい場合は、設定ファイルに `priority_networks` 設定項目を追加し、専用のIPアドレス（CIDR形式）をCNノードに割り当てる必要があります。クラスターでFQDNアクセスを有効にする場合は、この設定項目は無視しても構いません。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注記**
      >
      > ターミナルで `ifconfig` を実行して、インスタンスが所有するIPアドレスを確認できます。

   c. インスタンスに複数のJDKがインストールされており、環境変数 `JAVA_HOME` で指定されているものとは異なる特定のJDKを使用したい場合は、選択したJDKがインストールされているパスを指定するために、設定ファイルに `JAVA_HOME` 設定項目を追加する必要があります。

      ```YAML
      # <path_to_JDK> を選択したJDKがインストールされているパスに置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   CNのパラメータのほとんどがBEから継承されているため、詳細な設定項目については[パラメータ設定 - BE設定項目](../administration/BE_configuration.md#be-configuration-items)を参照してください。

2. CNノードを起動します。

   ```Bash
   ./be/bin/start_cn.sh --daemon
   ```

   > **警告**
   >
   > - FQDNアクセスを有効にしてCNノードを起動する前に、**/etc/hosts** にすべてのインスタンスのホスト名が割り当てられていることを確認してください。詳細は[環境設定チェックリスト - ホスト名](../deployment/environment_configurations.md#hostnames)を参照してください。
   > - CNノードを起動する際に `--host_type` パラメータを指定する必要はありません。

3. CNログを確認して、CNノードが正常に起動しているかを検証します。

   ```Bash
   cat be/log/cn.INFO | grep heartbeat
   ```

   "I0313 15:03:45.820030 412450 thrift_server.cpp:375] heartbeat has started listening port on 9050" のようなログの記録は、CNノードが正常に起動していることを示しています。

4. 他のインスタンスで上記の手順を繰り返すことで、新しいCNノードを起動できます。

## ステップ 4: クラスタを設定する

すべてのFE、BEノード、およびCNノードが正常に起動し、クラスタに追加されたら、StarRocksクラスタを設定できます。

以下の手順はMySQLクライアントで実行されます。MySQLクライアント5.5.0以降がインストールされている必要があります。

1. MySQLクライアントを介してStarRocksに接続します。初期アカウント `root` でログインし、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address> をリーダーFEノードのIPアドレス（priority_networks）またはFQDNに、
   # <query_port> をfe.confで指定したクエリポート（デフォルト: 9030）に置き換えてください。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. リーダーFEノードの状態を確認するために、以下のSQLを実行します。

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

   - `Alive` フィールドが `true` であれば、このFEノードは正常に起動し、クラスタに追加されています。
   - `Role` フィールドが `FOLLOWER` であれば、このFEノードはリーダーFEノードに選出される可能性があります。
   - `Role` フィールドが `LEADER` であれば、このFEノードはリーダーFEノードです。

3. BEノードをクラスタに追加します。

   ```SQL
   -- <be_address> をBEノードのIPアドレス（priority_networks）またはFQDNに、
   -- <heartbeat_service_port> をbe.confで指定したハートビートサービスポート（デフォルト: 9050）に置き換えてください。
   ALTER SYSTEM ADD BACKEND "<be_address>:<heartbeat_service_port>";
   ```

   > **注記**
   >
   > 上記のコマンドを使用して、一度に複数のBEノードを追加することができます。各 `<be_address>:<heartbeat_service_port>` ペアは1つのBEノードを表します。

4. BEノードの状態を確認するために、以下のSQLを実行します。

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

   `Alive` フィールドが `true` であれば、このBEノードは正常に起動し、クラスタに追加されています。

5. (オプション)クラスタにCNノードを追加します。

   ```SQL
   -- <cn_address> をCNノードのIPアドレス（priority_networks）またはFQDNに、
   -- <heartbeat_service_port> をcn.confで指定したハートビートサービスポート（デフォルト: 9050）に置き換えてください。
   ALTER SYSTEM ADD COMPUTE NODE "<cn_address>:<heartbeat_service_port>";
   ```

   > **注記**
   >
   > 1つのSQLで複数のCNノードを追加することができます。各 `<cn_address>:<heartbeat_service_port>` ペアは1つのCNノードを表します。

6. (オプション)CNノードの状態を確認するために、以下のSQLを実行します。

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


   フィールド `Alive` が `true` の場合、この CN ノードは正しく起動され、クラスタに追加されています。

   CN が正しく起動され、クエリ中に CN を使用したい場合は、システム変数 `SET prefer_compute_node = true;` と `SET use_compute_nodes = -1;` を設定してください。詳細については、[システム変数](../reference/System_variable.md#descriptions-of-variables)を参照してください。

## ステップ 5: (オプション) 高可用性 FE クラスタのデプロイ

高可用性 FE クラスタには、StarRocks クラスタに少なくとも三つのフォロワー FE ノードが必要です。リーダー FE ノードが正常に起動した後、2つの新しい FE ノードを起動して、高可用性 FE クラスタをデプロイできます。

1. MySQL クライアントを介して StarRocks に接続します。初期アカウント `root` でログインし、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address> をリーダー FE ノードの IP アドレス (priority_networks) または FQDN に、
   # <query_port> を fe.conf で指定したクエリポート (デフォルト: 9030) に置き換えてください。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 以下の SQL を実行して、新しいフォロワー FE ノードをクラスタに追加します。

   ```SQL
   -- <fe_address> を新しいフォロワー FE ノードの IP アドレス (priority_networks) 
   -- または FQDN に、<edit_log_port> を fe.conf で指定した edit_log_port 
   -- (デフォルト: 9010) に置き換えてください。
   ALTER SYSTEM ADD FOLLOWER "<fe2_address>:<edit_log_port>";
   ```

   > **注記**
   >
   > - 上記のコマンドを使用して、一度に一つのフォロワー FE ノードを追加できます。
   > - オブザーバー FE ノードを追加したい場合は、`ALTER SYSTEM ADD OBSERVER "<fe_address>:<edit_log_port>"` を実行してください。詳細な手順については、[ALTER SYSTEM - FE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を参照してください。

3. 新しい FE インスタンスでターミナルを起動し、メタデータストレージ専用のディレクトリを作成し、StarRocks FE デプロイメントファイルが格納されているディレクトリに移動して、FE 設定ファイル **fe/conf/fe.conf** を変更します。詳細については、[ステップ 1: リーダー FE ノードの起動](#step-1-start-the-leader-fe-node)を参照してください。基本的に、**FE ノードを起動するために使用するコマンドを除いて**、ステップ 1 の手順を繰り返すことができます。
  
   フォロワー FE ノードを設定した後、以下の SQL を実行してフォロワー FE ノードにヘルパーノードを割り当て、フォロワー FE ノードを起動します。

   > **注記**
   >
   > 新しいフォロワー FE ノードをクラスタに追加する場合、ヘルパーノード（実質的には既存のフォロワー FE ノード）を新しいフォロワー FE ノードに割り当てて、メタデータを同期する必要があります。

   - IP アドレスでアクセスする新しい FE ノードを起動するには、以下のコマンドを実行して FE ノードを起動します：

     ```Bash
     # <helper_fe_ip> をリーダー FE ノードの IP アドレス (priority_networks) に、
     # <helper_edit_log_port> をリーダー FE ノードの edit_log_port (デフォルト: 9010) に置き換えてください。
     ./fe/bin/start_fe.sh --helper <helper_fe_ip>:<helper_edit_log_port> --daemon
     ```

     パラメータ `--helper` は、ノードを初めて起動する際に一度だけ指定する必要があります。

   - FQDN でアクセスする新しい FE ノードを起動するには、以下のコマンドを実行して FE ノードを起動します：

     ```Bash
     # <helper_fqdn> をリーダー FE ノードの FQDN に、
     # <helper_edit_log_port> をリーダー FE ノードの edit_log_port (デフォルト: 9010) に置き換えてください。
     ./fe/bin/start_fe.sh --helper <helper_fqdn>:<helper_edit_log_port> \
           --host_type FQDN --daemon
     ```

     パラメータ `--helper` と `--host_type` は、ノードを初めて起動する際に一度だけ指定する必要があります。

4. FE ログを確認して、FE ノードが正常に起動したかどうかを検証します。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   「2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020.」というログの記録は、FE ノードが正常に起動していることを示しています。

5. すべての新しいフォロワー FE ノードが正しく起動するまで、手順 2、3、および 4 を繰り返し、MySQL クライアントから次の SQL を実行して FE ノードのステータスを確認します：

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

   - フィールド `Alive` が `true` の場合、この FE ノードは正しく起動され、クラスタに追加されています。
   - フィールド `Role` が `FOLLOWER` の場合、この FE ノードはリーダー FE ノードに選出される資格があります。
   - フィールド `Role` が `LEADER` の場合、この FE ノードはリーダー FE ノードです。

## StarRocks クラスタの停止

StarRocks クラスタを停止するには、対応するインスタンスで以下のコマンドを実行します。

- FE ノードを停止します。

  ```Bash
  ./fe/bin/stop_fe.sh --daemon
  ```

- BE ノードを停止します。

  ```Bash
  ./be/bin/stop_be.sh --daemon
  ```

- CN ノードを停止します。

  ```Bash
  ./be/bin/stop_cn.sh --daemon
  ```

## トラブルシューティング

FE、BE、または CN ノードの起動時に発生するエラーを特定するには、以下のステップを試してください。

- FE ノードが正しく起動しない場合、**fe/log/fe.warn.log** のログを確認することで問題を特定できます。

  ```Bash
  cat fe/log/fe.warn.log
  ```

  問題を特定して解決した後、まず現行の FE プロセスを終了し、既存の **meta** ディレクトリを削除し、新しいメタデータストレージディレクトリを作成してから、正しい設定で FE ノードを再起動する必要があります。

- BE ノードが正しく起動しない場合、**be/log/be.WARNING** のログを確認することで問題を特定できます。

  ```Bash
  cat be/log/be.WARNING
  ```

  問題を特定して解決した後、まず現行の BE プロセスを終了し、既存の **storage** ディレクトリを削除し、新しいデータストレージディレクトリを作成してから、正しい設定で BE ノードを再起動する必要があります。

- CN ノードが正しく起動しない場合、**be/log/cn.WARNING** のログを確認することで問題を特定できます。

  ```Bash
  cat be/log/cn.WARNING
  ```


  問題を特定し解決した後は、まず既存の CN プロセスを終了し、その後正しい設定で CN ノードを再起動する必要があります。

## 次に行うこと

StarRocks クラスタをデプロイした後、[デプロイ後のセットアップ](../deployment/post_deployment_setup.md)へ進んで、初期管理措置についての指示を参照してください。
