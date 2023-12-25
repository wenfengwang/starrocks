---
displayed_sidebar: Chinese
---

# 手動で StarRocks をデプロイする

この文書では、StarRocks の統合ストレージと計算クラスター（BE がデータストレージと計算の両方を行う）を手動でデプロイする方法について説明します。他のインストール方法については、[デプロイメント概要](../deployment/deployment_overview.md)を参照してください。

ストレージと計算が分離されたクラスターをデプロイする場合は、[StarRocks ストレージと計算分離クラスターのデプロイ](./shared_data/s3.md)を参照してください。

## ステップ 1: Leader FE ノードを起動する

以下の操作は FE インスタンスで実行します。

1. メタデータストレージパスを作成します。FE のデプロイファイルとは異なるパスにメタデータを保存することをお勧めします。このパスが存在し、書き込み権限があることを確認してください。

   ```Bash
   # <meta_dir> を作成するメタデータディレクトリに置き換えてください。
   mkdir -p <meta_dir>
   ```

2. 事前に準備した [StarRocks FE デプロイファイル](../deployment/prepare_deployment_files.md)があるパスに移動し、FE の設定ファイル **fe/conf/fe.conf** を編集します。

   a. `meta_dir` の設定項目でメタデータパスを指定します。

      ```YAML
      # <meta_dir> を作成したメタデータディレクトリに置き換えてください。
      meta_dir = <meta_dir>
      ```

   b. [環境設定リスト](../deployment/environment_configurations.md)で言及されている FE のポートが使用中の場合、FE の設定ファイルで他の利用可能なポートに割り当てる必要があります。

      ```YAML
      http_port = aaaa        # デフォルト値：8030
      rpc_port = bbbb         # デフォルト値：9020
      query_port = cccc       # デフォルト値：9030
      edit_log_port = dddd    # デフォルト値：9010
      ```

      > **注意**
      >
      > クラスターに複数の FE ノードをデプロイする場合は、すべての FE ノードに同じ `http_port` を割り当てる必要があります。

   c. クラスターで IP アドレスアクセスを有効にする場合は、設定ファイルに `priority_networks` の設定項目を追加し、FE ノードに専用の IP アドレス（CIDR 形式）を割り当てる必要があります。FQDN アクセスを有効にする場合は、この設定項目を無視しても構いません。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **説明**
      >
      > 現在のインスタンスが持っている IP アドレスを確認するには、ターミナルで `ifconfig` を実行します。

   d. 複数の JDK をインストールしていて、環境変数 `JAVA_HOME` で指定されている JDK とは異なる JDK を使用したい場合は、選択した JDK のインストールパスを指定するために設定ファイルに `JAVA_HOME` の設定項目を追加する必要があります。

      ```YAML
      # <path_to_JDK> を選択した JDK のインストールパスに置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   e. その他の高度な設定項目については、[パラメータ設定 - FE の設定項目](../administration/FE_configuration.md#fe-設定項目)を参照してください。

3. FE ノードを起動します。

   - クラスターで IP アドレスアクセスを有効にする場合は、以下のコマンドを実行して FE ノードを起動してください：

     ```Bash
     ./fe/bin/start_fe.sh --daemon
     ```

   - クラスターで FQDN アクセスを有効にする場合は、以下のコマンドを実行して FE ノードを起動してください：

     ```Bash
     ./fe/bin/start_fe.sh --host_type FQDN --daemon
     ```

     `--host_type` パラメータはノードを初めて起動するときにのみ指定する必要があります。

     > **注意**
     >
     > FQDN アクセスを有効にする場合は、FE ノードを起動する前に **/etc/hosts** ですべてのインスタンスにホスト名を割り当てていることを確認してください。詳細については、[環境設定リスト - ホスト名](../deployment/environment_configurations.md#ホスト名)を参照してください。

4. FE のログを確認し、FE ノードが正常に起動したかどうかをチェックします。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   ログに以下の内容が出力されていれば、FE ノードが正常に起動したことを意味します：

   "2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] Thrift サーバーがポート 9020 で起動しました。"

## ステップ 2: BE サービスを起動する

以下の操作は BE インスタンスで実行します。

1. データストレージパスを作成します。BE のデプロイファイルとは異なるパスにデータを保存することをお勧めします。このパスが存在し、書き込み権限があることを確認してください。

   ```Bash
   # <storage_root_path> を作成するデータストレージパスに置き換えてください。
   mkdir -p <storage_root_path>
   ```

2. 事前に準備した [StarRocks BE デプロイファイル](../deployment/prepare_deployment_files.md)があるパスに移動し、BE の設定ファイル **be/conf/be.conf** を編集します。

   a. `storage_root_path` の設定項目でデータストレージパスを指定します。

      ```YAML
      # <storage_root_path> を作成したデータストレージパスに置き換えてください。
      storage_root_path = <storage_root_path>
      ```

   b. [環境設定リスト](../deployment/environment_configurations.md)で言及されている BE のポートが使用中の場合、BE の設定ファイルで他の利用可能なポートに割り当てる必要があります。

      ```YAML
      be_port = vvvv                   # デフォルト値：9060
      be_http_port = xxxx              # デフォルト値：8040
      heartbeat_service_port = yyyy    # デフォルト値：9050
      brpc_port = zzzz                 # デフォルト値：8060
      ```

   c. クラスターで IP アドレスアクセスを有効にする場合は、設定ファイルに `priority_networks` の設定項目を追加し、BE ノードに専用の IP アドレス（CIDR 形式）を割り当てる必要があります。FQDN アクセスを有効にする場合は、この設定項目を無視しても構いません。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **説明**
      >
      > 現在のインスタンスが持っている IP アドレスを確認するには、ターミナルで `ifconfig` を実行します。

   d. 複数の JDK をインストールしていて、環境変数 `JAVA_HOME` で指定されている JDK とは異なる JDK を使用したい場合は、選択した JDK のインストールパスを指定するために設定ファイルに `JAVA_HOME` の設定項目を追加する必要があります。

      ```YAML
      # <path_to_JDK> を選択した JDK のインストールパスに置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   e. その他の高度な設定項目については、[パラメータ設定 - BE の設定項目](../administration/BE_configuration.md#be-設定項目)を参照してください。

3. BE ノードを起動します。

   ```Bash
   ./be/bin/start_be.sh --daemon
   ```

   > **注意**
   >
   > - FQDN アクセスを有効にする場合は、BE ノードを起動する前に **/etc/hosts** ですべてのインスタンスにホスト名を割り当てていることを確認してください。詳細については、[環境設定リスト - ホスト名](../deployment/environment_configurations.md#ホスト名)を参照してください。
   > - BE ノードを起動する際に `--host_type` パラメータを指定する必要はありません。

4. BE のログを確認し、BE ノードが正常に起動したかどうかをチェックします。

   ```Bash
   cat be/log/be.INFO | grep heartbeat
   ```

   ログに以下の内容が出力されていれば、BE ノードが正常に起動したことを意味します：

   "I0614 17:41:39.782819 3717531 thrift_server.cpp:388] ハートビートがポート 9050 でリスニングを開始しました。"

5. 他の BE インスタンスで上記の手順を繰り返すことで、新しい BE ノードを起動できます。

> **説明**
>
> StarRocks クラスターに少なくとも 3 つの BE ノードをデプロイして追加すると、これらのノードは自動的に BE の高可用性クラスターを形成します。

## ステップ 3:（オプション）CN サービスを起動する

Compute Node（CN）は、ステートレスな計算サービスで、データを保存しません。CN ノードを追加することで、クエリに追加の計算リソースを提供できます。BE のデプロイファイルを使用して CN ノードをデプロイできます。CN ノードはバージョン 2.4 からサポートされています。

1. 移動してください、先に準備した [StarRocks BE 配置ファイル](../deployment/prepare_deployment_files.md)のあるディレクトリに、そして変更してください CN の設定ファイル **be/conf/cn.conf**。

   a. もし [環境設定一覧](../deployment/environment_configurations.md) で挙げられた CN のポートが使用中であれば、CN の設定ファイルで他の利用可能なポートを割り当てる必要があります。

      ```YAML
      be_port = vvvv                   # デフォルト値：9060
      be_http_port = xxxx              # デフォルト値：8040
      heartbeat_service_port = yyyy    # デフォルト値：9050
      brpc_port = zzzz                 # デフォルト値：8060
      ```

   b. クラスターに IP アドレスでのアクセスを有効にする場合は、設定ファイルに `priority_networks` を追加し、CN ノードに専用の IP アドレス（CIDR 形式）を割り当てる必要があります。FQDN でのアクセスを有効にする場合は、この設定は不要です。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **説明**
      >
      > 現在のインスタンスが持っている IP アドレスを確認するには、ターミナルで `ifconfig` を実行してください。

   c. 複数の JDK をインストールしていて、環境変数 `JAVA_HOME` で指定されているものと異なる JDK を使用したい場合は、選択した JDK のインストールパスを指定するために設定ファイルに `JAVA_HOME` を追加する必要があります。

      ```YAML
      # <path_to_JDK> を選択した JDK のインストールパスに置き換えてください。
      JAVA_HOME = <path_to_JDK>
      ```

   d. 多くの CN パラメータは BE ノードから継承されるため、より多くの CN の高度な設定項目については [パラメータ設定 - BE 設定項目](../administration/BE_configuration.md#be-設定項目) を参照してください。

2. CN ノードを起動します。

   ```Bash
   ./be/bin/start_cn.sh --daemon
   ```

   > **注意**
   >
   > - FQDN アクセスを有効にする場合、CN ノードを起動する前に、**/etc/hosts** で全てのインスタンスにホスト名を割り当てていることを確認してください。詳細は [環境設定一覧 - ホスト名](../deployment/environment_configurations.md#ホスト名) を参照してください。
   > - CN ノードを起動する際に `--host_type` パラメータを指定する必要はありません。

3. CN ログを確認し、CN ノードが正常に起動したかをチェックします。

   ```Bash
   cat be/log/cn.INFO | grep heartbeat
   ```

   ログに以下の内容が出力されていれば、CN ノードの起動に成功しています：

   "I0313 15:03:45.820030 412450 thrift_server.cpp:375] heartbeat has started listening port on 9050"

4. 他のインスタンスで上記の手順を繰り返し、新しい CN ノードを起動します。

## 第四ステップ：クラスタの構築

全ての FE、BE、CN ノードが正常に起動したら、StarRocks クラスタを構築できます。

以下の手順は MySQL クライアントインスタンスで実行します。MySQL クライアント（バージョン 5.5.0 以上）をインストールする必要があります。

1. MySQL クライアントを使用して StarRocks に接続します。初期ユーザー `root` でログインし、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address> を Leader FE ノードの IP アドレス（priority_networks）または FQDN に、
   # <query_port>（デフォルト：9030）を fe.conf で指定した query_port に置き換えてください。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 以下の SQL を実行して Leader FE ノードの状態を確認します。

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

   - `Alive` フィールドが `true` であれば、その FE ノードは正常に起動しクラスタに参加しています。
   - `Role` フィールドが `FOLLOWER` であれば、その FE ノードは Leader FE ノードになる資格があります。
   - `Role` フィールドが `LEADER` であれば、その FE ノードは Leader FE ノードです。

3. BE ノードをクラスタに追加します。

   ```SQL
   -- <be_address> を BE ノードの IP アドレス（priority_networks）または FQDN に、
   -- <heartbeat_service_port>（デフォルト：9050）を be.conf で指定した heartbeat_service_port に置き換えてください。
   ALTER SYSTEM ADD BACKEND "<be_address>:<heartbeat_service_port>", "<be2_address>:<heartbeat_service_port>", "<be3_address>:<heartbeat_service_port>";
   ```

   > **説明**
   >
   > 複数の BE ノードを一度に SQL で追加することができます。各 `<be_address>:<heartbeat_service_port>` は一つの BE ノードを表します。

4. 以下の SQL を実行して BE ノードの状態を確認します。

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

   `Alive` フィールドが `true` であれば、その BE ノードは正常に起動しクラスタに参加しています。

5. （オプション）CN ノードをクラスタに追加します。

   ```SQL
   -- <cn_address> を CN ノードの IP アドレス（priority_networks）または FQDN に、
   -- <heartbeat_service_port>（デフォルト：9050）を cn.conf で指定した heartbeat_service_port に置き換えてください。
   ALTER SYSTEM ADD COMPUTE NODE "<cn_address>:<heartbeat_service_port>", "<cn2_address>:<heartbeat_service_port>", "<cn3_address>:<heartbeat_service_port>";
   ```

   > **説明**
   >
   > 複数の CN ノードを一度に SQL で追加することができます。各 `<cn_address>:<heartbeat_service_port>` は一つの CN ノードを表します。

6. （オプション）以下の SQL を実行して CN ノードの状態を確認します。

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


   もしフィールド `Alive` が `true` であれば、その CN ノードは正常に起動し、クラスターに参加していることを意味します。

   クエリ実行時に CN ノードの計算能力を拡張する必要がある場合は、システム変数 `SET prefer_compute_node = true;` と `SET use_compute_nodes = -1;` を設定する必要があります。システム変数の詳細については、[システム変数](../reference/System_variable.md#支持の変数)を参照してください。

## 第五步：（オプション）高可用性 FE クラスターのデプロイ

高可用性の FE クラスターを StarRocks クラスターにデプロイするには、少なくとも3つの Follower FE ノードが必要です。高可用性 FE クラスターをデプロイする場合は、追加で2つの新しい FE ノードを起動する必要があります。

1. MySQL クライアントを使用して StarRocks に接続します。初期ユーザー `root` でログインし、パスワードはデフォルトで空です。

   ```Bash
   # <fe_address> を Leader FE ノードの IP アドレス（priority_networks）または FQDN に、
   # <query_port>（デフォルト：9030）を fe.conf で指定した query_port に置き換えます。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 以下の SQL を実行して、追加の FE ノードをクラスターに追加します。

   ```SQL
   -- <new_fe_address> を追加する新しい FE ノードの IP アドレス（priority_networks）または FQDN に、
   -- <edit_log_port>（デフォルト：9010）を新しい FE ノードの fe.conf で指定した edit_log_port に置き換えます。
   ALTER SYSTEM ADD FOLLOWER "<new_fe_address>:<edit_log_port>";
   ```

   > **説明**
   >
   > - 一度に一つの Follower FE ノードを SQL で追加することができます。
   > - Observer FE ノードをさらに追加するには、`ALTER SYSTEM ADD OBSERVER "<fe_address>:<edit_log_port>"` を実行してください。詳細は [ALTER SYSTEM - FE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) を参照してください。

3. 新しい FE インスタンスでターミナルを起動し、メタデータストレージパスを作成し、StarRocks のデプロイディレクトリに移動して、FE の設定ファイル **fe/conf/fe.conf** を編集します。詳細は [第一步：Leader FE ノードの起動](#第一步启动-leader-fe-节点) を参照してください。

   設定が完了したら、以下のコマンドを使用して新しい Follower FE ノードにヘルパーノードを割り当て、新しい FE ノードを起動します：

   > **説明**
   >
   > クラスターに新しい Follower FE ノードを追加する際には、最初に新しい FE ノードを起動するときにヘルパーノード（既存の Follower FE ノード）を割り当てて、すべての FE メタデータ情報を同期する必要があります。

   - クラスターで IP アドレスアクセスが有効になっている場合は、以下のコマンドを実行して FE ノードを起動します：

   ```Bash
   # <helper_fe_ip> を Leader FE ノードの IP アドレス（priority_networks）に、
   # <helper_edit_log_port>（デフォルト：9010）を Leader FE ノードの edit_log_port に置き換えます。
   ./fe/bin/start_fe.sh --helper <helper_fe_ip>:<helper_edit_log_port> --daemon
   ```

   パラメータ `--helper` は最初の起動時にのみ指定する必要があります。

   - クラスターで FQDN アクセスが有効になっている場合は、以下のコマンドを実行して FE ノードを起動します：

   ```Bash
   # <helper_fqdn> を Leader FE ノードの FQDN に、
   # <helper_edit_log_port>（デフォルト：9010）を Leader FE ノードの edit_log_port に置き換えます。
   ./fe/bin/start_fe.sh --helper <helper_fqdn>:<helper_edit_log_port> \
         --host_type FQDN --daemon
   ```

   パラメータ `--helper` と `--host_type` は最初の起動時にのみ指定する必要があります。

4. FE のログを確認し、FE ノードが正常に起動したかをチェックします。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   ログに以下の内容が出力されていれば、FE ノードが正常に起動したことを意味します：

   "2022-08-10 16:12:29,911 INFO (不明 x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] Thrift サーバーがポート 9020 で起動しました。"

5. 上記の手順 2、3、および 4 を繰り返し、すべての Follower FE ノードを起動した後、MySQL クライアントを使用して FE ノードの状態を確認します。

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

   - フィールド `Alive` が `true` の場合、FE ノードが正常に起動し、クラスターに参加していることを意味します。
   - フィールド `Role` が `FOLLOWER` の場合、FE ノードが Leader FE ノードに選出される資格があることを意味します。
   - フィールド `Role` が `LEADER` の場合、FE ノードが Leader FE ノードであることを意味します。

## StarRocks クラスターの停止

以下のコマンドを対応するインスタンスで実行することで、StarRocks クラスターを停止することができます。

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

FE、BE、または CN ノードの起動に失敗した場合、以下のステップで問題を特定してください：

- FE ノードが正常に起動していない場合は、**fe/log/fe.warn.log** のログを確認して問題を特定します。

  ```Bash
  cat fe/log/fe.warn.log
  ```

  問題を特定し解決した後、現在の FE プロセスを終了し、既存の **meta** パスを削除し、新しいメタデータストレージパスを作成し、正しい設定で FE ノードを再起動してください。

- BE ノードが正常に起動していない場合は、**be/log/be.WARNING** のログを確認して問題を特定します。

  ```Bash
  cat be/log/be.WARNING
  ```


問題を特定して解決した後、まず現在の BE プロセスを終了し、既存の **storage** パスを削除し、新しいデータストレージパスを作成してから、正しい設定で BE ノードを再起動する必要があります。

- CN ノードが正常に起動していない場合は、**be/log/cn.WARNING** のログを確認して問題を特定できます。

  ```Bash
  cat be/log/cn.WARNING
  ```

問題を特定して解決した後、まず現在の CN プロセスを終了し、正しい設定で CN ノードを再起動する必要があります。

## 次のステップ

StarRocks クラスターのデプロイが成功した後、[デプロイ後の設定](../deployment/post_deployment_setup.md) を参照して、初期管理措置についての説明を得ることができます。
