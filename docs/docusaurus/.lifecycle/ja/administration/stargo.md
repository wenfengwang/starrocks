---
displayed_sidebar: "Japanese"
---

# StarGoを使用したStarRocksの展開と管理

このトピックでは、StarGoを使用してStarRocksクラスタを展開および管理する方法について説明します。

StarGoは、複数のStarRocksクラスタを管理するためのコマンドラインツールです。StarGoを使用して、複数のクラスタを簡単に展開、チェック、アップグレード、ダウングレード、開始、停止することができます。

## StarGoのインストール

次のファイルを中央管理ノードにダウンロードします。

- **sr-ctl**: StarGoのバイナリファイル。ダウンロード後にインストールする必要はありません。
- **sr-c1.yaml**: 展開設定ファイルのテンプレート。
- **repo.yaml**: StarRocksインストーラのダウンロードパスの構成ファイル。

> 注
> `http://cdn-thirdparty.starrocks.com` にアクセスして、対応するインストールインデックスファイルおよびインストーラを取得できます。

```shell
wget https://github.com/wangtianyi2004/starrocks-controller/raw/main/stargo-pkg.tar.gz
wget https://github.com/wangtianyi2004/starrocks-controller/blob/main/sr-c1.yaml
wget https://github.com/wangtianyi2004/starrocks-controller/blob/main/repo.yaml
```

**sr-ctl** にアクセス権を付与します。

```shell
chmod 751 sr-ctl
```

## StarRocksクラスタの展開

StarGoを使用してStarRocksクラスタを展開できます。

### 前提条件

- 展開するクラスタには、少なくとも1つの中央管理ノードと3つの展開ノードが必要です。すべてのノードを1台のマシンに展開することができます。
- 中央管理ノードにStarGoを展開する必要があります。
- 中央管理ノードと3つの展開ノードの間で相互のSSH認証を構築する必要があります。

次の例は、中央管理ノードのsr-dev@r0と3つの展開ノードstarrocks@r1、starrocks@r2、starrocks@r3の間で相互の認証を構築する方法です。

```plain text
## sr-dev@r0とstarrocks@r1、2、3の間で相互認証を構築する。
[sr-dev@r0 ~]$ ssh-keygen
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r1
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r2
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r3

## sr-dev@r0とstarrocks@r1、2、3の間で相互認証を検証する。
[sr-dev@r0 ~]$ ssh starrocks@r1 date
[sr-dev@r0 ~]$ ssh starrocks@r2 date
[sr-dev@r0 ~]$ ssh starrocks@r3 date
```

### 設定ファイルの作成

次のYAMLテンプレートに基づいてStarRocks展開トポロジファイルを作成します。詳細については、[Configuration](../administration/Configuration.md)を参照してください。

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
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノード用のユニークなIPを指定します。
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
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノード用のユニークなIPを指定します。
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
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノード用のユニークなIPを指定します。
    config:
      sys_log_level: "INFO"
be_servers:
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノード用のユニークなIPを指定します。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノード用のユニークなIPを指定します。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数のIPアドレスがある場合、現在のノード用のユニークなIPを指定します。
    config:
      create_tablet_worker_count: 3
```

### 展開ディレクトリの作成（オプション）

StarRocksを展開するパスが存在せず、そのようなパスを作成する権限がある場合は、これらのパスを作成する必要はありません。また、パスがすでに存在する場合は、それらへの書き込みアクセス権を持っていることを確認してください。また、以下のコマンドを実行して各ノードで必要な展開ディレクトリを作成することもできます。

- FEノードで**meta**ディレクトリを作成します。

```shell
mkdir -p StarRocks/fe/meta
```

- BEノードで**storage**ディレクトリを作成します。

```shell
mkdir -p StarRocks/be/storage
```

> 注意
> 上記のパスが、設定ファイルの`meta_dir`および`storage_dir`の構成項目と同一であることを確認してください。

### StarRocksの展開

次のコマンドを実行してStarRocksクラスタを展開します。

```shell
./sr-ctl cluster deploy <cluster_name> <version> <topology_file>
```

|パラメータ|説明|
|----|----|
|cluster_name|展開するクラスタの名前。|
|version|StarRocksのバージョン。|
|topology_file|構成ファイルの名前。|

展開が成功した場合、クラスタは自動的に開始されます。beStatus および feStatus が true の場合、クラスタは正常に開始されます。

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
[20220301-234836  OUTPUT] デプロイフォルダを作成します...
[20220301-234838  OUTPUT] StarRocksパッケージとjdkをダウンロードします...
[20220302-000515    INFO] ファイルstarrocks-2.0.1-quickstart.tar.gz[1227406189]が正常にダウンロードされました
[20220302-000515  OUTPUT] ダウンロードが完了しました。
[20220302-000515  OUTPUT] StarRocksパッケージとjdkを展開します...
[20220302-000520    INFO] tarファイル/home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gzは/home/sr-dev/.starrocks-controller/download以下に展開されました
[20220302-000547    INFO] tarファイル/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gzは/home/sr-dev/.starrocks-controller/download以下に展開されました
[20220302-000556    INFO] tarファイル/home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gzは/home/sr-dev/.starrocks-controller/download以下に展開されました
[20220302-000556  OUTPUT] FEディレクトリを配布します...
[20220302-000603    INFO] ディレクトリfeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe]をfeTargetDir = [StarRocks/fe]のFeHost = [192.168.xx.xx]にアップロードします
[20220302-000615    INFO] ディレクトリJDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301]をJDKTargetDir = [StarRocks/fe/jdk]のFeHost = [192.168.xx.xx]にアップロードします
[20220302-000615    INFO] JAVA_HOMEを変更: ホスト = [192.168.xx.xx]、ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000622    INFO] ディレクトリfeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe]をfeTargetDir = [StarRocks/fe]のFeHost = [192.168.xx.xx]にアップロードします
[20220302-000634    INFO] ディレクトリJDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301]をJDKTargetDir = [StarRocks/fe/jdk]のFeHost = [192.168.xx.xx]にアップロードします
[20220302-000634    INFO] JAVA_HOMEを変更: ホスト = [192.168.xx.xx]、ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000640    INFO] ディレクトリfeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe]をfeTargetDir = [StarRocks/fe]のFeHost = [192.168.xx.xx]にアップロードします
[20220302-000652    INFO] ディレクトリJDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301]をJDKTargetDir = [StarRocks/fe/jdk]のFeHost = [192.168.xx.xx]にアップロードします
[20220302-000652    INFO] JAVA_HOMEを変更: ホスト = [192.168.xx.xx]、ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000652  OUTPUT] BEディレクトリを配布します...
[20220302-000728    INFO] ディレクトリBeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be]をBeTargetDir = [StarRocks/be]のBeHost = [192.168.xx.xx]にアップロードします
[20220302-000752    INFO] ディレクトリBeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be]をBeTargetDir = [StarRocks/be]のBeHost = [192.168.xx.xx]にアップロードします
[20220302-000815    INFO] ディレクトリBeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be]をBeTargetDir = [StarRocks/be]のBeHost = [192.168.xx.xx]にアップロードします
[20220302-000815  OUTPUT] FEノードおよびBEノードの構成を変更します...
############################################# FEクラスタの開始 #############################################
############################################# FEクラスタの開始 #############################################
[20220302-000816    INFO] リーダーFEノードを起動します[host = 192.168.xx.xx、editLogPort = 9010]
[20220302-000836    INFO] FEノードの起動に成功しました[host = 192.168.xx.xx、queryPort = 9030]
[20220302-000836    INFO] フォロワーFEノードを起動します[host = 192.168.xx.xx、editLogPort = 9010]
[20220302-000857    INFO] FEノードの起動に成功しました[host = 192.168.xx.xx、queryPort = 9030]
[20220302-000857    INFO] フォロワーFEノードを起動します[host = 192.168.xx.xx、editLogPort = 9010]
[20220302-000918    INFO] FEノードの起動に成功しました[host = 192.168.xx.xx、queryPort = 9030]
[20220302-000918    INFO] すべてのFEのステータスをリストアップしています:
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true

############################################# BEクラスタの開始 #############################################
############################################# BEクラスタの開始 #############################################
[20220302-000918    INFO] BEノードを起動します[BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-000939    INFO] BEノードの起動に成功しました[host = 192.168.xx.xx、heartbeatServicePort = 9050]
[20220302-000939    INFO] BEノードを起動します[BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-001000    INFO] BEノードの起動に成功しました[host = 192.168.xx.xx、heartbeatServicePort = 9050]
[20220302-001000    INFO] BEノードを起動します[BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-001020    INFO] BEノードの起動に成功しました[host = 192.168.xx.xx、heartbeatServicePort = 9050]
[20220302-001020  OUTPUT] すべてのBEのステータスをリストアップしています:
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
```
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
```

## Start cluster

StarRocksクラスターは、StarGoを使って開始できます。

### クラスター内のすべてのノードを開始する

次のコマンドを実行して、クラスター内のすべてのノードを開始します。

```shell
./sr-ctl cluster start <cluster-name>
```

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1
[20220303-190404  OUTPUT] Start cluster [clusterName = sr-c1]
[20220303-190404    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190435    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190446    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190457    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 特定の役割のノードを開始する

- クラスター内のすべてのFEノードを開始する。

```shell
./sr-ctl cluster start <cluster_name> --role FE
```

- クラスター内のすべてのBEノードを開始する。

```shell
./sr-ctl cluster start <cluster_name> --role BE
```

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --role FE
[20220303-191529  OUTPUT] Start cluster [clusterName = sr-c1]
[20220303-191529    INFO] Starting FE cluster ....
[20220303-191529    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191600    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191610    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]

[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --role BE
[20220303-194215  OUTPUT] Start cluster [clusterName = sr-c1]
[20220303-194215    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194216    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] Starting BE cluster ...
```

### 特定のノードを開始する

クラスター内の特定のノードを開始します。現在、BEノードのみをサポートしています。

```shell
./sr-ctl cluster start <cluster_name> --node <node_ID>
```

特定のノードのIDは、[特定のクラスターの情報を表示](#view-the-information-of-a-specific-cluster)することで確認できます。

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --node 192.168.xx.xx:9060
[20220303-194714  OUTPUT] Start cluster [clusterName = sr-c1]
[20220303-194714    INFO] Start BE node. [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

## クラスターを停止する

StarRocksクラスターは、StarGoを使って停止できます。

### クラスター内のすべてのノードを停止する

次のコマンドを実行して、クラスター内のすべてのノードを停止します。

```shell
./sr-ctl cluster stop <cluster_name>
```

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1
[20220302-180140  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220302-180140  OUTPUT] Stop cluster sr-c1
[20220302-180140    INFO] Waiting for stoping FE node [FeHost = 192.168.xx.xx]
[20220302-180143  OUTPUT] The FE node stop succefully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180143    INFO] Waiting for stoping FE node [FeHost = 192.168.xx.xx]
[20220302-180145  OUTPUT] The FE node stop succefully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180145    INFO] Waiting for stoping FE node [FeHost = 192.168.xx.xx]
[20220302-180148  OUTPUT] The FE node stop succefully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180148  OUTPUT] Stop cluster sr-c1
[20220302-180148    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180148    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180148    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180149    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 特定の役割のノードを停止する

- クラスター内のすべてのFEノードを停止する。

```shell
./sr-ctl cluster stop <cluster_name> --role FE
```

- クラスター内のすべてのBEノードを停止する。

```shell
./sr-ctl cluster stop <cluster_name> --role BE
```

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1 --role BE
[20220302-180624  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220302-180624  OUTPUT] Stop cluster sr-c1
[20220302-180624    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180624    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180624    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] Stopping BE cluster ...

###########################################################################

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1 --role FE
[20220302-180849  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220302-180849    INFO] Stopping FE cluster ....
[20220302-180849  OUTPUT] Stop cluster sr-c1
[20220302-180849    INFO] Waiting for stoping FE node [FeHost = 192.168.xx.xx]
[20220302-180851  OUTPUT] The FE node stop succefully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180851    INFO] Waiting for stoping FE node [FeHost = 192.168.xx.xx]
```
```
      + 出力 {T}
      + 正常にFEノードを停止しました [ホスト = 192.168.xx.xx, クエリポート = 9030]
      + 情報 [20220302-180854] FEノードの停止を待っています [FeHost = 192.168.xx.xx]
      + 出力 {T}
      + 正常にFEノードを停止しました [ホスト = 192.168.xx.xx, クエリポート = 9030]
```

### 特定のノードを停止する

クラスタ内の特定のノードを停止します。

```shell
./sr-ctl cluster stop <cluster_name> --node <node_ID>
```

特定のノードのIDは、[特定のクラスタの情報を表示する](#view-the-information-of-a-specific-cluster)を参照して取得できます。

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster display sr-c1
[20220303-185400  出力] クラスタの表示 [clusterName = sr-c1]
clusterName = sr-c1
[20220303-185400    警告] すべてのFEノードがダウンしています。 FEノードを起動して再度クラスタの状態を表示してください。
ID                          役割    ホスト                   ポート              状態        データディレクトリ                              デプロイディレクトリ
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage

[root@nd1 sr-controller]# ./sr-ctl cluster stop sr-c1 --node 192.168.xx.xx:9060
[20220303-185510  出力] クラスタを停止 [clusterName = sr-c1]
[20220303-185510    情報] BEノードの停止中。[BeHost = 192.168.xx.xx]
[20220303-185510    情報] BEノードの停止を待っています。[BeHost = 192.168.xx.xx]
```

## クラスタのスケールアウト

StarGoを使用して、クラスタをスケールアウトすることができます。

### 構成ファイルの作成

次のテンプレートに基づいて、スケールアウトタスクのトポロジファイルを作成します。要件に応じて、FEおよび/またはBEノードを追加するファイルを指定できます。詳細情報については、[Configuration](../administration/Configuration.md)を参照してください。

```yaml
# FEノードの追加。
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
    priority_networks: 192.168.xx.xx/24 # マシンに複数のIPアドレスがある場合、現在のノード用の一意のIPを指定します。
    config:
      sys_log_level: "INFO"
      sys_log_delete_age: "1d"

# BEノードの追加。
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

新しいノードをクラスタに追加する場合、新しいノードと中央制御ノードとの間で相互認証を構築する必要があります。詳細な手順については、[前提条件](#prerequisites)を参照してください。

### デプロイディレクトリの作成（オプション）

新しいノードのデプロイ先パスが存在しない場合、かつそのようなパスを作成する権限がある場合、これらのパスを作成する必要はありません。StarGoは構成ファイルに基づいてこれらのパスを作成します。パスがすでに存在する場合は、それらに書き込みアクセス権限を持っていることを確認してください。それぞれのノードで必要なデプロイディレクトリを作成することもできます。

- FEノードで**meta**ディレクトリを作成します。

```shell
mkdir -p StarRocks/fe/meta
```

- BEノードで**storage**ディレクトリを作成します。

```shell
mkdir -p StarRocks/be/storage
```

> 注意
> 上記のパスが、構成ファイルの`meta_dir`および`storage_dir`の構成項目と同一であることを確認してください。

### クラスタをスケールアウトする

次のコマンドを実行して、クラスタをスケールアウトさせます。

```shell
./sr-ctl cluster scale-out <cluster_name> <topology_file>
```

例：

```plain text
# スケールアウト前のクラスタの状態。
[root@nd1 sr-controller]# ./sr-ctl cluster display sr-test       
[20220503-210047  OUTPUT] クラスタの表示 [clusterName = sr-test]
clusterName = sr-test
clusterVerison = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    

# クラスタをスケールアウトする。
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster scale-out sr-test sr-out.yaml
[20220503-213725  OUTPUT] クラスタをスケールアウト [ClusterName = sr-test]
[20220503-213731  OUTPUT] デプロイ環境のプレチェック:
PreCheck FE:
IP                    ssh auth         meta dir                        deploy dir                      http port        rpc port         query port       edit log port  
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                            PASS                            PASS             PASS             PASS             PASS           

PreCheck BE:
IP                    ssh auth         storage dir                     deploy dir                      webSer port      heartbeat port   brpc port        be port        
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                            PASS                            PASS             PASS             PASS             PASS           


[20220503-213731  OUTPUT] プレチェックが成功しました。結果を表示します。
[20220503-213731  OUTPUT] デプロイフォルダを作成しています...
[20220503-213732  出力] StarRocksのパッケージとJDKをダウンロードしています...
[20220503-213732    情報] パッケージはすでに存在しています [fileName = starrocks-2.0.1-quickstart.tar.gz, fileSize = 1227406189, fileModTime = 2022-05-03 17:32:03.478661923 +0800 CST]
[20220503-213732  OUTPUT] ダウンロード完了。
[20220503-213732  OUTPUT] StarRocksパッケージおよびJDKの展開中...
[20220503-213741    情報] ターファイル /home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gz が /home/sr-dev/.starrocks-controller/download 以下に展開されました
[20220503-213837    情報] ターファイル /home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gz が /home/sr-dev/.starrocks-controller/download 以下に展開されました
[20220503-213837    情報] ターファイル /home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gz が /home/sr-dev/.starrocks-controller/download 以下に展開されました
[20220503-213837  OUTPUT] FEディレクトリを配布中...
[20220503-213845    情報] feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] を feTargetDir = [StarRocks/fe] に FeHost = [192.168.xx.xx] にアップロードしました
```
```
[20220503-213857 INFO] JDKのディレクトリJDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301]をJDKTargetDir = [StarRocks/fe/jdk]にFeHost = [192.168.xx.xx]にアップロードします
[20220503-213857 INFO] Modify JAVA_HOME: host = [192.168.xx.xx], filePath = [StarRocks/fe/bin/start_fe.sh]
[20220503-213857 OUTPUT] Distribute BE Dir ...
[20220503-213924 INFO] BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be]をBeTargetDir = [StarRocks/be]にBeHost = [192.168.xx.xx]にアップロードします
[20220503-213924 OUTPUT] Modify configuration for FE nodes & BE nodes ...
############################################# FEクラスターをスケールアウト #############################################
############################################# FEクラスターをスケールアウト #############################################
[20220503-213925 INFO] フォロワーFEノードを起動します [host = 192.168.xx.xx, editLogPort = 9010]
[20220503-213945 INFO] FEノードが正常に開始しました [host = 192.168.xx.xx, queryPort = 9030]
[20220503-213945 INFO] すべてのFEステータスをリスト化します:
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true

############################################# BEクラスターを開始 #############################################
############################################# BEクラスターを開始 #############################################
[20220503-213945 INFO] BEノードを開始します [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220503-214016 INFO] BEノードが正常に開始しました [host = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220503-214016 OUTPUT] すべてのBEステータスをリスト化します:
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true

# スケールアウト後のクラスターの状態。
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-test 
[20220503-214302 OUTPUT] クラスターを表示します [clusterName = sr-test]
clusterName = sr-test
clusterVerison = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   StarRocks/fe/meta                            
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   StarRocks/be/storage                         
```

## クラスターのスケールイン

次のコマンドを実行して、クラスターからノードを削除します。

```shell
./sr-ctl cluster scale-in <cluster_name> --node <node_id>
```

特定のクラスターの情報を[表示することで、特定のノードのIDを確認できます](#view-the-information-of-a-specific-cluster)。

例:

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-c1
[20220505-145649 OUTPUT] クラスターを表示します [clusterName = sr-c1]
clusterName = sr-c1
clusterVerison = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster scale-in sr-c1 --node 192.168.88.83:9010
[20220621-010553 OUTPUT] クラスターをスケールインします [clusterName = sr-c1, nodeId = 192.168.88.83:9010]
[20220621-010553 INFO] FEノードの停止を待機中 [FeHost = 192.168.88.83]
[20220621-010606 OUTPUT] FEノードを正常にスケールインしました。 [clusterName = sr-c1, nodeId = 192.168.88.83:9010]

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-c1
[20220621-010623 OUTPUT] クラスターを表示します [clusterName = sr-c1]
clusterName = sr-c1
clusterVerison = 
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.88.84:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.88.85:9010          FE      192.168.xx.xx         9010/9030        UP/L        StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.88.83:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.88.84:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.88.85:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage              
```

## クラスターのアップグレードまたはダウングレード

StarGoを使用してクラスターをアップグレードまたはダウングレードできます。

- クラスターをアップグレードします。

```shell
./sr-ctl cluster upgrade <cluster_name>  <target_version>
```

- クラスターをダウングレードします。

```shell
./sr-ctl cluster downgrade <cluster_name>  <target_version>
```

例:

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster list
[20220515-195827 OUTPUT] すべてのクラスターをリスト化します
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 19:35:36        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster upgrade sr-test2 v2.1.3
[20220515-200358 OUTPUT] すべてのクラスターをリスト化します
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.1.3      test222     2022-05-15 20:03:01        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster downgrade sr-test2 v2.0.1 
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster list
[20220515-200915 OUTPUT] すべてのクラスターをリスト化します
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 20:08:40        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                
```

## 関連するコマンド
```
|Command|Description|
|----|----|
|deploy|クラスターを展開します。|
|start|クラスターを開始します。|
|stop|クラスターを停止します。|
|scale-in|クラスターをスケールインします。|
|scale-out|クラスターをスケールアウトします。|
|upgrade|クラスターをアップグレードします。|
|downgrade|クラスターをダウングレードします。|
|display|特定のクラスターの情報を表示します。|
|list|すべてのクラスターを表示します。|
```