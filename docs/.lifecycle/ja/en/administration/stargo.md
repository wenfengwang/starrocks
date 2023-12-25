---
displayed_sidebar: English
---

# StarGoを使ってStarRocksをデプロイおよび管理する

このトピックでは、StarGoを使用してStarRocksクラスタをデプロイおよび管理する方法について説明します。

StarGoは、複数のStarRocksクラスタを管理するためのコマンドラインツールです。StarGoを使用して、複数のクラスタを簡単にデプロイ、チェック、アップグレード、ダウングレード、起動、停止することができます。

## StarGoのインストール

以下のファイルを中央制御ノードにダウンロードしてください：

- **sr-ctl**: StarGoのバイナリファイルです。ダウンロード後、インストールする必要はありません。
- **sr-c1.yaml**: デプロイメント設定ファイルのテンプレートです。
- **repo.yaml**: StarRocksインストーラのダウンロードパスを設定するファイルです。

> 注意
> `http://cdn-thirdparty.starrocks.com` にアクセスして、対応するインストールインデックスファイルとインストーラを取得できます。

```shell
wget https://github.com/wangtianyi2004/starrocks-controller/raw/main/stargo-pkg.tar.gz
wget https://github.com/wangtianyi2004/starrocks-controller/blob/main/sr-c1.yaml
wget https://github.com/wangtianyi2004/starrocks-controller/blob/main/repo.yaml
```

**sr-ctl**に実行権限を付与します。

```shell
chmod 751 sr-ctl
```

## StarRocksクラスタのデプロイ

StarGoを使用してStarRocksクラスタをデプロイできます。

### 前提条件

- デプロイされるクラスタには、少なくとも1つの中央制御ノードと3つのデプロイノードが必要です。すべてのノードは1台のマシンにデプロイすることができます。
- 中央制御ノードにStarGoをデプロイする必要があります。
- 中央制御ノードと3つのデプロイノード間で相互SSH認証を構築する必要があります。

以下の例では、中央制御ノードsr-dev@r0と3つのデプロイノードstarrocks@r1、starrocks@r2、starrocks@r3間で相互認証を構築します。

```plain text
## sr-dev@r0とstarrocks@r1、2、3の間で相互認証を構築します。
[sr-dev@r0 ~]$ ssh-keygen
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r1
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r2
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r3

## sr-dev@r0とstarrocks@r1、2、3の間で相互認証を検証します。
[sr-dev@r0 ~]$ ssh starrocks@r1 date
[sr-dev@r0 ~]$ ssh starrocks@r2 date
[sr-dev@r0 ~]$ ssh starrocks@r3 date
```

### 設定ファイルの作成

以下のYAMLテンプレートに基づいてStarRocksデプロイトポロジファイルを作成します。詳細は[設定](../administration/FE_configuration.md)を参照してください。

```yaml
global:
    user: "starrocks"   ## 現在のOSユーザー。
    ssh_port: 22

fe_servers:
  - host: 192.168.XX.XX
    ssh_port: 22
    http_port: 8030
    rpc_port: 9020
    query_port: 9030
    edit_log_port: 9010
    deploy_dir: StarRocks/fe
    meta_dir: StarRocks/fe/meta
    log_dir: StarRocks/fe/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノードのユニークなIPを指定します。
    config:
      sys_log_level: "INFO"
  - host: 192.168.XX.XX
    ssh_port: 22
    http_port: 8030
    rpc_port: 9020
    query_port: 9030
    edit_log_port: 9010
    deploy_dir: StarRocks/fe
    meta_dir: StarRocks/fe/meta
    log_dir: StarRocks/fe/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノードのユニークなIPを指定します。
    config:
      sys_log_level: "INFO"
  - host: 192.168.XX.XX
    ssh_port: 22
    http_port: 8030
    rpc_port: 9020
    query_port: 9030
    edit_log_port: 9010
    deploy_dir: StarRocks/fe
    meta_dir: StarRocks/fe/meta
    log_dir: StarRocks/fe/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノードのユニークなIPを指定します。
    config:
      sys_log_level: "INFO"
be_servers:
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir: StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノードのユニークなIPを指定します。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir: StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノードのユニークなIPを指定します。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir: StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノードのユニークなIPを指定します。
    config:
      create_tablet_worker_count: 3
```

### デプロイメントディレクトリの作成（オプショナル）

StarRocksをデプロイするパスが存在しない場合、またはそのようなパスを作成する権限がある場合、これらのパスを作成する必要はありません。StarGoは設定ファイルに基づいてそれらを作成します。パスが既に存在する場合は、それらに書き込みアクセス権があることを確認してください。次のコマンドを実行して、各ノードに必要なデプロイメントディレクトリを作成することもできます。

- FEノードに**meta**ディレクトリを作成します。

```shell
mkdir -p StarRocks/fe/meta
```

- BEノードに**storage**ディレクトリを作成します。

```shell
mkdir -p StarRocks/be/storage
```

> 注意
> 上記のパスが設定ファイルの`meta_dir`および`storage_dir`の設定項目と同一であることを確認してください。

### StarRocksのデプロイ

以下のコマンドを実行してStarRocksクラスタをデプロイします。

```shell
./sr-ctl cluster deploy <cluster_name> <version> <topology_file>
```

|パラメータ|説明|
|----|----|
|cluster_name|デプロイするクラスタの名前。|
|version|StarRocksのバージョン。|
|topology_file|設定ファイルの名前。|

デプロイが成功すると、クラスタは自動的に起動されます。beStatusとfeStatusがtrueの場合、クラスタは正常に起動されています。

例：

```plain text
[sr-dev@r0 ~]$ ./sr-ctl cluster deploy sr-c1 v2.0.1 sr-c1.yaml
[20220301-234817  OUTPUT] Deploy cluster [clusterName = sr-c1, clusterVersion = v2.0.1, metaFile = sr-c1.yaml]
[20220301-234836  OUTPUT] PRE CHECK DEPLOY ENV:
PreCheck FE:
IP                    ssh auth         meta dir                   deploy dir                 http port        rpc port         query port       edit log port
--------------------  ---------------  -------------------------  -------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS

PreCheck BE:
IP                    ssh auth         storage dir                deploy dir                 webSer port      heartbeat port   brpc port        be port
--------------------  ---------------  -------------------------  -------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS


[20220301-234836  OUTPUT] PreCheck successfully. RESPECT
[20220301-234836  OUTPUT] Create the deploy folder ...
[20220301-234838  OUTPUT] Download StarRocks package & jdk ...
[20220302-000515    INFO] The file starrocks-2.0.1-quickstart.tar.gz [1227406189] download successfully
[20220302-000515  OUTPUT] Download done.
[20220302-000515  OUTPUT] Decompress StarRocks package & jdk ...
[20220302-000520    INFO] The tar file /home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gz has been decompressed under /home/sr-dev/.starrocks-controller/download
[20220302-000547    INFO] The tar file /home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gz has been decompressed under /home/sr-dev/.starrocks-controller/download
[20220302-000556    INFO] The tar file /home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gz has been decompressed under /home/sr-dev/.starrocks-controller/download
[20220302-000556  OUTPUT] Distribute FE Dir ...

[20220302-000603    INFO] ディレクトリをアップロード feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] から feTargetDir = [StarRocks/fe] へ FeHost = [192.168.xx.xx]
[20220302-000615    INFO] ディレクトリをアップロード JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] から JDKTargetDir = [StarRocks/fe/jdk] へ FeHost = [192.168.xx.xx]
[20220302-000615    INFO] JAVA_HOME を変更: ホスト = [192.168.xx.xx], ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000622    INFO] ディレクトリをアップロード feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] から feTargetDir = [StarRocks/fe] へ FeHost = [192.168.xx.xx]
[20220302-000634    INFO] ディレクトリをアップロード JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] から JDKTargetDir = [StarRocks/fe/jdk] へ FeHost = [192.168.xx.xx]
[20220302-000634    INFO] JAVA_HOME を変更: ホスト = [192.168.xx.xx], ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000640    INFO] ディレクトリをアップロード feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] から feTargetDir = [StarRocks/fe] へ FeHost = [192.168.xx.xx]
[20220302-000652    INFO] ディレクトリをアップロード JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] から JDKTargetDir = [StarRocks/fe/jdk] へ FeHost = [192.168.xx.xx]
[20220302-000652    INFO] JAVA_HOME を変更: ホスト = [192.168.xx.xx], ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000652  OUTPUT] BE ディレクトリを配布...
[20220302-000728    INFO] ディレクトリをアップロード BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] から BeTargetDir = [StarRocks/be] へ BeHost = [192.168.xx.xx]
[20220302-000752    INFO] ディレクトリをアップロード BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] から BeTargetDir = [StarRocks/be] へ BeHost = [192.168.xx.xx]
[20220302-000815    INFO] ディレクトリをアップロード BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] から BeTargetDir = [StarRocks/be] へ BeHost = [192.168.xx.xx]
[20220302-000815  OUTPUT] FE ノードと BE ノードの設定を変更...
############################################# FE クラスタを開始 #############################################
############################################# FE クラスタを開始 #############################################
[20220302-000816    INFO] リーダー FE ノードを開始 [ホスト = 192.168.xx.xx, editLogPort = 9010]
[20220302-000836    INFO] FE ノードが正常に起動しました [ホスト = 192.168.xx.xx, queryPort = 9030]
[20220302-000836    INFO] フォロワー FE ノードを開始 [ホスト = 192.168.xx.xx, editLogPort = 9010]
[20220302-000857    INFO] FE ノードが正常に起動しました [ホスト = 192.168.xx.xx, queryPort = 9030]
[20220302-000857    INFO] フォロワー FE ノードを開始 [ホスト = 192.168.xx.xx, editLogPort = 9010]
[20220302-000918    INFO] FE ノードが正常に起動しました [ホスト = 192.168.xx.xx, queryPort = 9030]
[20220302-000918    INFO] すべての FE ステータスをリスト:
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true

############################################# BE クラスタを開始 #############################################
############################################# BE クラスタを開始 #############################################
[20220302-000918    INFO] BE ノードを開始 [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-000939    INFO] BE ノードが正常に起動しました [ホスト = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220302-000939    INFO] BE ノードを開始 [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-001000    INFO] BE ノードが正常に起動しました [ホスト = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220302-001000    INFO] BE ノードを開始 [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-001020    INFO] BE ノードが正常に起動しました [ホスト = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220302-001020  OUTPUT] すべての BE ステータスをリスト:
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
```

クラスター情報を[表示することで、クラスターをテストできます](#クラスター情報の表示)。

また、MySQLクライアントを使用してクラスタに接続し、テストすることもできます。

```shell
mysql -h 127.0.0.1 -P9030 -uroot
```

## クラスター情報の表示

StarGoが管理するクラスタの情報を表示できます。

### すべてのクラスタの情報を表示

次のコマンドを実行して、すべてのクラスタの情報を表示します。

```shell
./sr-ctl cluster list
```

例：

```shell
[sr-dev@r0 ~]$ ./sr-ctl cluster list
[20220302-001640  OUTPUT] すべてのクラスタをリスト
ClusterName      User        CreateDate                 MetaPath                                                      PrivateKey
---------------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-c1            starrocks   2022-03-02 00:08:15        /home/sr-dev/.starrocks-controller/cluster/sr-c1              /home/sr-dev/.ssh/id_rsa
```

### 特定のクラスタの情報を表示

次のコマンドを実行して、特定のクラスタの情報を表示します。

```shell
./sr-ctl cluster display <cluster_name>
```

例：

```plain text
[sr-dev@r0 ~]$ ./sr-ctl cluster display sr-c1
[20220302-002310  OUTPUT] クラスタを表示 [clusterName = sr-c1]
clusterName = sr-c1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
```

## クラスタの起動

StarGoを通じてStarRocksクラスタを起動できます。

### クラスタ内のすべてのノードを起動

次のコマンドを実行して、クラスタ内のすべてのノードを起動します。

```shell
./sr-ctl cluster start <cluster-name>
```

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1
[20220303-190404  OUTPUT] クラスタを起動 [clusterName = sr-c1]
[20220303-190404    INFO] FE ノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190435    INFO] FE ノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190446    INFO] FE ノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190457    INFO] BE ノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] BE ノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] BE ノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 特定の役割のノードを起動

- クラスタ内のすべてのFEノードを起動します。

```shell
./sr-ctl cluster start <cluster_name> --role FE
```

- クラスタ内のすべてのBEノードを起動します。

```shell
./sr-ctl cluster start <cluster_name> --role BE
```

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --role FE
[20220303-191529  OUTPUT] クラスタを起動 [clusterName = sr-c1]
[20220303-191529    INFO] FE クラスタを起動...
[20220303-191529    INFO] FE ノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191600    INFO] FE ノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191610    INFO] FE ノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]

[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --role BE
[20220303-194215  OUTPUT] クラスタを起動 [clusterName = sr-c1]
[20220303-194215    INFO] BE ノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194216    INFO] BE ノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] BE ノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] BE クラスタを起動...
```

### 特定のノードを起動

クラスタ内の特定のノードを起動します。現在、BE ノードのみがサポートされています。

```shell
./sr-ctl cluster start <cluster_name> --node <node_ID>
```


特定のノードのIDは、[特定のクラスターの情報を表示する](#view-the-information-of-a-specific-cluster)ことで確認できます。

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --node 192.168.xx.xx:9060
[20220303-194714  OUTPUT] Start cluster [clusterName = sr-c1]
[20220303-194714    INFO] Start BE node. [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

## クラスターの停止

StarRocksクラスターはStarGoを介して停止できます。

### クラスタ内のすべてのノードを停止

次のコマンドを実行して、クラスタ内のすべてのノードを停止します。

```shell
./sr-ctl cluster stop <cluster_name>
```

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1
[20220302-180140  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220302-180140  OUTPUT] Stop cluster sr-c1
[20220302-180140    INFO] Waiting for stopping FE node [FeHost = 192.168.xx.xx]
[20220302-180143  OUTPUT] The FE node stopped successfully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180143    INFO] Waiting for stopping FE node [FeHost = 192.168.xx.xx]
[20220302-180145  OUTPUT] The FE node stopped successfully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180145    INFO] Waiting for stopping FE node [FeHost = 192.168.xx.xx]
[20220302-180148  OUTPUT] The FE node stopped successfully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180148  OUTPUT] Stop cluster sr-c1
[20220302-180148    INFO] Waiting for stopping BE node [BeHost = 192.168.xx.xx]
[20220302-180148    INFO] The BE node stopped successfully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180148    INFO] Waiting for stopping BE node [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] The BE node stopped successfully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180149    INFO] Waiting for stopping BE node [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] The BE node stopped successfully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 特定の役割のノードを停止

- クラスタ内のすべてのFEノードを停止します。

```shell
./sr-ctl cluster stop <cluster_name> --role FE
```

- クラスタ内のすべてのBEノードを停止します。

```shell
./sr-ctl cluster stop <cluster_name> --role BE
```

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1 --role BE
[20220302-180624  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220302-180624  OUTPUT] Stop cluster sr-c1
[20220302-180624    INFO] Waiting for stopping BE node [BeHost = 192.168.xx.xx]
[20220302-180624    INFO] The BE node stopped successfully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180624    INFO] Waiting for stopping BE node [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] The BE node stopped successfully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] Waiting for stopping BE node [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] The BE node stopped successfully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] Stopping BE cluster ...

###########################################################################

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1 --role FE
[20220302-180849  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220302-180849    INFO] Stopping FE cluster...
[20220302-180849  OUTPUT] Stop cluster sr-c1
[20220302-180849    INFO] Waiting for stopping FE node [FeHost = 192.168.xx.xx]
[20220302-180851  OUTPUT] The FE node stopped successfully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180851    INFO] Waiting for stopping FE node [FeHost = 192.168.xx.xx]
[20220302-180854  OUTPUT] The FE node stopped successfully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180854    INFO] Waiting for stopping FE node [FeHost = 192.168.xx.xx]
[20220302-180856  OUTPUT] The FE node stopped successfully [host = 192.168.xx.xx, queryPort = 9030]
```

### 特定のノードを停止

クラスタ内の特定のノードを停止します。

```shell
./sr-ctl cluster stop <cluster_name> --node <node_ID>
```

特定のノードのIDは、[特定のクラスターの情報を表示する](#view-the-information-of-a-specific-cluster)ことで確認できます。

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster display sr-c1
[20220303-185400  OUTPUT] Display cluster [clusterName = sr-c1]
clusterName = sr-c1
[20220303-185400    WARN] All FE nodes are down, please start FE node and display the cluster status again.
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage

[root@nd1 sr-controller]# ./sr-ctl cluster stop sr-c1 --node 192.168.xx.xx:9060
[20220303-185510  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220303-185510    INFO] Stopping BE node. [BeHost = 192.168.xx.xx]
[20220303-185510    INFO] Waiting for stopping BE node [BeHost = 192.168.xx.xx]
```

## クラスターのスケールアウト

StarGoを使用してクラスターをスケールアウトできます。

### 構成ファイルの作成

次のテンプレートに基づいてスケールアウトタスクトポロジファイルを作成します。必要に応じて、FEノードやBEノードを追加するファイルを指定できます。詳細については、[構成](../administration/FE_configuration.md)を参照してください。

```yaml
# FEノードを追加します。
fe_servers:
  - host: 192.168.xx.xx # 新しいFEノードのIPアドレス。
    ssh_port: 22
    http_port: 8030
    rpc_port: 9020
    query_port: 9030
    edit_log_port: 9010
    deploy_dir: StarRocks/fe
    meta_dir: StarRocks/fe/meta
    log_dir: StarRocks/fe/log
    priority_networks: 192.168.xx.xx/24 # マシンに複数のIPアドレスがある場合、現在のノードの固有のIPを指定します。
    config:
      sys_log_level: "INFO"
      sys_log_delete_age: "1d"

# BEノードを追加します。
be_servers:
  - host: 192.168.xx.xx # 新しいBEノードのIPアドレス。
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    config:
      create_tablet_worker_count: 3
```

### SSH相互認証の構築

クラスタに新しいノードを追加する場合、新しいノードと中央制御ノード間で相互認証を構築する必要があります。詳細な手順については、[前提条件](#prerequisites)を参照してください。

### デプロイメントディレクトリの作成（オプション）

新しいノードをデプロイするパスが存在しない場合、またはそのようなパスを作成する権限がある場合、これらのパスを作成する必要はありません。StarGoは構成ファイルに基づいてそれらを作成します。パスが既に存在する場合は、それらに対する書き込みアクセス権を持っていることを確認してください。次のコマンドを実行して、各ノードに必要なデプロイメントディレクトリを作成することもできます。

- FEノードに**メタディレクトリ**を作成します。

```shell
mkdir -p StarRocks/fe/meta
```

- BEノードに**ストレージディレクトリ**を作成します。

```shell
mkdir -p StarRocks/be/storage
```

> 注意
> 上記のパスが構成ファイルの`meta_dir`および`storage_dir`の構成項目と同一であることを確認してください。

### クラスターをスケールアウト

次のコマンドを実行してクラスターをスケールアウトします。

```shell
./sr-ctl cluster scale-out <cluster_name> <topology_file>
```

例：

```plain text
# スケールアウト前のクラスターの状態。
[root@nd1 sr-controller]# ./sr-ctl cluster display sr-test       
[20220503-210047  OUTPUT] Display cluster [clusterName = sr-test]

clusterName = sr-test
clusterVersion = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    

# クラスターの拡張
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster scale-out sr-test sr-out.yaml
[20220503-213725  OUTPUT] クラスターを拡張します。[ClusterName = sr-test]
[20220503-213731  OUTPUT] 環境の前提チェック:
PreCheck FE:
IP                    ssh認証          metaディレクトリ               deployディレクトリ             httpポート        rpcポート         queryポート       edit logポート  
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                            PASS                            PASS             PASS             PASS             PASS           

PreCheck BE:
IP                    ssh認証          storageディレクトリ             deployディレクトリ             webサーバーポート  heartbeatポート   brpcポート        beポート        
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                            PASS                            PASS             PASS             PASS             PASS           


[20220503-213731  OUTPUT] 前提チェック成功。RESPECT
[20220503-213731  OUTPUT] デプロイフォルダを作成...
[20220503-213732  OUTPUT] StarRocksパッケージとjdkをダウンロード...
[20220503-213732    INFO] パッケージは既に存在します [fileName = starrocks-2.0.1-quickstart.tar.gz, fileSize = 1227406189, fileModTime = 2022-05-03 17:32:03.478661923 +0800 CST]
[20220503-213732  OUTPUT] ダウンロード完了。
[20220503-213732  OUTPUT] StarRocksパッケージとjdkを解凍...
[20220503-213741    INFO] tarファイル /home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gz は /home/sr-dev/.starrocks-controller/download に解凍されました
[20220503-213837    INFO] tarファイル /home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gz は /home/sr-dev/.starrocks-controller/download に解凍されました
[20220503-213837    INFO] tarファイル /home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gz は /home/sr-dev/.starrocks-controller/download に解凍されました
[20220503-213837  OUTPUT] FEディレクトリを配布...
[20220503-213845    INFO] ディレクトリfeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] をFeHost = [192.168.xx.xx] のfeTargetDir = [StarRocks/fe] にアップロード
[20220503-213857    INFO] ディレクトリJDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] をFeHost = [192.168.xx.xx] のJDKTargetDir = [StarRocks/fe/jdk] にアップロード
[20220503-213857    INFO] JAVA_HOMEを変更: ホスト = [192.168.xx.xx], ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220503-213857  OUTPUT] BEディレクトリを配布...
[20220503-213924    INFO] ディレクトリBeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] をBeHost = [192.168.xx.xx] のBeTargetDir = [StarRocks/be] にアップロード
[20220503-213924  OUTPUT] FEノードとBEノードの設定を変更...
############################################# FEクラスターの拡張 #############################################
############################################# FEクラスターの拡張 #############################################
[20220503-213925    INFO] フォロワーFEノードを起動 [ホスト = 192.168.xx.xx, editLogポート = 9010]
[20220503-213945    INFO] FEノードが正常に起動しました [ホスト = 192.168.xx.xx, queryポート = 9030]
[20220503-213945    INFO] 全てのFEステータスをリスト:
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true

############################################# BEクラスターの開始 #############################################
############################################# BEクラスターの開始 #############################################
[20220503-213945    INFO] BEノードを起動 [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220503-214016    INFO] BEノードが正常に起動しました [ホスト = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220503-214016  OUTPUT] 全てのBEステータスをリスト:
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true

# 拡張後のクラスターの状態
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-test 
[20220503-214302  OUTPUT] クラスターを表示 [clusterName = sr-test]
clusterName = sr-test
clusterVersion = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   StarRocks/fe/meta                            
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   StarRocks/be/storage                         
```

## クラスターのスケールイン

次のコマンドを実行して、クラスター内のノードを削除します。

```shell
./sr-ctl cluster scale-in <cluster_name> --node <node_id>
```

特定のノードのIDは、[特定のクラスターの情報を表示](#特定のクラスターの情報を表示)することで確認できます。

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-c1
[20220505-145649  OUTPUT] クラスターを表示 [clusterName = sr-c1]
clusterName = sr-c1
clusterVersion = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster scale-in sr-c1 --node 192.168.88.83:9010
[20220621-010553  OUTPUT] クラスターのスケールイン [clusterName = sr-c1, nodeId = 192.168.88.83:9010]
[20220621-010553    INFO] FEノードの停止を待機中 [FeHost = 192.168.88.83]
[20220621-010606  OUTPUT] FEノードのスケールインが成功しました。[clusterName = sr-c1, nodeId = 192.168.88.83:9010]

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-c1
[20220621-010623  OUTPUT] クラスターを表示 [clusterName = sr-c1]
clusterName = sr-c1
clusterVersion = 
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.88.84:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.88.85:9010          FE      192.168.xx.xx         9010/9030        UP/L        StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.88.83:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.88.84:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.88.85:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage              
```

## クラスタのアップグレードまたはダウングレード

StarGoを介してクラスタをアップグレードまたはダウングレードできます。

- クラスタをアップグレードします。

```shell
./sr-ctl cluster upgrade <cluster_name> <target_version>
```

- クラスタをダウングレードします。

```shell
./sr-ctl cluster downgrade <cluster_name> <target_version>
```

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster list
[20220515-195827  OUTPUT] 全てのクラスタをリスト
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 19:35:36        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster upgrade sr-test2 v2.1.3
[20220515-200358  OUTPUT] 全てのクラスタをリスト
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.1.3      test222     2022-05-15 20:03:01        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster downgrade sr-test2 v2.0.1 
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster list
[20220515-200915  OUTPUT] 全てのクラスタをリスト
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 20:08:40        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                
```

## 関連コマンド

| コマンド | 説明 |
|----|----|
|deploy|クラスターをデプロイします。|
|start|クラスターを起動します。|
|stop|クラスターを停止します。|
|scale-in|クラスターをスケールインします。|
|scale-out|クラスターをスケールアウトします。|
|upgrade|クラスターをアップグレードします。|
|downgrade|クラスターをダウングレードします。|
|display|特定のクラスターの情報を表示します。|
|list|全てのクラスターを表示します。|
