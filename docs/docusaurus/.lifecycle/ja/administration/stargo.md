---
displayed_sidebar: "Japanese"
---

# StarGoを使用してStarRocksをデプロイおよび管理する

このトピックでは、StarGoを使用してStarRocksクラスタをデプロイおよび管理する方法について説明します。

StarGoは、複数のStarRocksクラスタを管理するためのコマンドラインツールです。StarGoを使用して、複数のクラスタを簡単にデプロイ、チェック、アップグレード、ダウングレード、開始、停止することができます。

## StarGoのインストール

次のファイルを中央制御ノードにダウンロードします。

- **sr-ctl**：StarGoのバイナリファイル。ダウンロード後にインストールする必要はありません。
- **sr-c1.yaml**：デプロイ構成ファイルのテンプレート。
- **repo.yaml**：StarRocksインストーラのダウンロードパスの構成ファイル。

> 注意
> 対応するインストールインデックスファイルとインストーラを取得するには、`http://cdn-thirdparty.starrocks.com` にアクセスできます。

```shell
wget https://github.com/wangtianyi2004/starrocks-controller/raw/main/stargo-pkg.tar.gz
wget https://github.com/wangtianyi2004/starrocks-controller/blob/main/sr-c1.yaml
wget https://github.com/wangtianyi2004/starrocks-controller/blob/main/repo.yaml
```

**sr-ctl**にアクセス権を付与します。

```shell
chmod 751 sr-ctl
```

## StarRocksクラスタのデプロイ

StarGoを使用してStarRocksクラスタをデプロイできます。

### 前提条件

- デプロイされるクラスタには、中央制御ノードおよび3つのデプロイノードが少なくとも1つ必要です。すべてのノードを1台のマシンにデプロイすることができます。
- 中央制御ノードにStarGoをデプロイする必要があります。
- 中央制御ノードと3つのデプロイノードの間で相互のSSH認証を構築する必要があります。

次の例では、中央制御ノードのsr-dev@r0と3つのデプロイノードstarrocks@r1、starrocks@r2、starrocks@r3の間で相互認証を構築します。

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

### 構成ファイルの作成

次のYAMLテンプレートに基づいて、StarRocksのデプロイトポロジーファイルを作成します。詳細な情報については、[構成](../administration/Configuration.md)を参照してください。

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
    priority_networks: 192.168.XX.XX/24 # マシンが複数のIPアドレスを持つ場合、カレントノードの固有IPを指定します。
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
    priority_networks: 192.168.XX.XX/24 # マシンが複数のIPアドレスを持つ場合、カレントノードの固有IPを指定します。
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
    priority_networks: 192.168.XX.XX/24 # マシンが複数のIPアドレスを持つ場合、カレントノードの固有IPを指定します。
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
    priority_networks: 192.168.XX.XX/24 # マシンが複数のIPアドレスを持つ場合、カレントノードの固有IPを指定します。
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
    priority_networks: 192.168.XX.XX/24 # マシンが複数のIPアドレスを持つ場合、カレントノードの固有IPを指定します。
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
    priority_networks: 192.168.XX.XX/24 # マシンが複数のIPアドレスを持つ場合、カレントノードの固有IPを指定します。
    config:
      create_tablet_worker_count: 3
```

### デプロイディレクトリの作成（オプション）

StarRocksをデプロイするパスが存在せず、そのようなパスを作成する権限がある場合、これらのパスを作成する必要はありません。StarGoは、構成ファイルに基づいてこれらのパスを作成します。パスがすでに存在する場合は、それらに書き込みアクセス権があることを確認してください。各ノードで必要なデプロイディレクトリを作成することもできます。

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

### StarRocksのデプロイ

次のコマンドを実行して、StarRocksクラスタをデプロイします。

```shell
./sr-ctl cluster deploy <cluster_name> <version> <topology_file>
```

|パラメータ|説明|
|----|----|
|cluster_name|デプロイするクラスタの名前。|
|version|StarRocksのバージョン。|
|topology_file|構成ファイルの名前。|

デプロイが成功すると、クラスタは自動的に開始されます。beStatusとfeStatusがtrueの場合、クラスタは正常に開始されます。

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
```
[20220301-234836  OUTPUT] デプロイフォルダを作成します...
[20220301-234838  OUTPUT] StarRocksパッケージとJDKをダウンロードします...
[20220302-000515    INFO] ファイルstarrocks-2.0.1-quickstart.tar.gz [1227406189] をダウンロードしました
[20220302-000515  OUTPUT] ダウンロードが完了しました。
[20220302-000515  OUTPUT] StarRocksパッケージとJDKを解凍します...
[20220302-000520    INFO] tarファイル/home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gzが/home/sr-dev/.starrocks-controller/downloadの下に解凍されました
[20220302-000547    INFO] tarファイル/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gzが/home/sr-dev/.starrocks-controller/downloadの下に解凍されました
[20220302-000556    INFO] tarファイル/home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gzが/home/sr-dev/.starrocks-controller/downloadの下に解凍されました
[20220302-000556  OUTPUT] FEディレクトリを配布します...
[20220302-000603    INFO] feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe]をfeTargetDir = [StarRocks/fe]にFeHost = [192.168.xx.xx]でアップロードします
[20220302-000615    INFO] JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301]をJDKTargetDir = [StarRocks/fe/jdk]にFeHost = [192.168.xx.xx]でアップロードします
[20220302-000615    INFO] JAVA_HOMEを変更します: ホスト = [192.168.xx.xx]、ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000622    INFO] feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe]をfeTargetDir = [StarRocks/fe]にFeHost = [192.168.xx.xx]でアップロードします
[20220302-000634    INFO] JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301]をJDKTargetDir = [StarRocks/fe/jdk]にFeHost = [192.168.xx.xx]でアップロードします
[20220302-000634    INFO] JAVA_HOMEを変更します: ホスト = [192.168.xx.xx]、ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000640    INFO] feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe]をfeTargetDir = [StarRocks/fe]にFeHost = [192.168.xx.xx]でアップロードします
[20220302-000652    INFO] JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301]をJDKTargetDir = [StarRocks/fe/jdk]にFeHost = [192.168.xx.xx]でアップロードします
[20220302-000652    INFO] JAVA_HOMEを変更します: ホスト = [192.168.xx.xx]、ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000652  OUTPUT] BEディレクトリを配布します...
[20220302-000728    INFO] BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be]をBeTargetDir = [StarRocks/be]にBeHost = [192.168.xx.xx]でアップロードします
[20220302-000752    INFO] BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be]をBeTargetDir = [StarRocks/be]にBeHost = [192.168.xx.xx]でアップロードします
[20220302-000815    INFO] BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be]をBeTargetDir = [StarRocks/be]にBeHost = [192.168.xx.xx]でアップロードします
[20220302-000815  OUTPUT] FEノードとBEノードの構成を変更します...
############################################# FEクラスタを開始 #############################################
############################################# FEクラスタを開始 #############################################
[20220302-000816    INFO] リーダーFEノードを開始します [ホスト = 192.168.xx.xx, editLogPort = 9010]
[20220302-000836    INFO] FEノードが正常に開始しました [ホスト = 192.168.xx.xx, queryPort = 9030]
[20220302-000836    INFO] フォロワーFEノードを開始します [ホスト = 192.168.xx.xx, editLogPort = 9010]
[20220302-000857    INFO] FEノードが正常に開始しました [ホスト = 192.168.xx.xx, queryPort = 9030]
[20220302-000857    INFO] フォロワーFEノードを開始します [ホスト = 192.168.xx.xx, editLogPort = 9010]
[20220302-000918    INFO] FEノードが正常に開始しました [ホスト = 192.168.xx.xx, queryPort = 9030]
[20220302-000918    INFO] すべてのFEステータスをリストアップします:
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true

############################################# BEクラスタを開始 #############################################
############################################# BEクラスタを開始 #############################################
[20220302-000918    INFO] BEノードを開始します [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-000939    INFO] BEノードが正常に開始しました [ホスト = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220302-000939    INFO] BEノードを開始します [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-001000    INFO] BEノードが正常に開始しました [ホスト = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220302-001000    INFO] BEノードを開始します [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-001020    INFO] BEノードが正常に開始しました [ホスト = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220302-001020  OUTPUT] すべてのBEステータスをリストアップします:
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
```
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage

## クラスターの開始

StarGoを使用してStarRocksクラスターを開始できます。

### クラスター内のすべてのノードを開始

以下のコマンドを実行してクラスター内のすべてのノードを開始します。

```shell
./sr-ctl cluster start <cluster-name>
```

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1
[20220303-190404  OUTPUT] クラスターを開始 [clusterName = sr-c1]
[20220303-190404    INFO] FEノードの開始 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190435    INFO] FEノードの開始 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190446    INFO] FEノードの開始 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190457    INFO] BEノードの開始 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] BEノードの開始 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] BEノードの開始 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 特定の役割のノードを開始

- クラスター内のすべてのFEノードを開始します。

```shell
./sr-ctl cluster start <cluster_name> --role FE
```

- クラスター内のすべてのBEノードを開始します。

```shell
./sr-ctl cluster start <cluster_name> --role BE
```

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --role FE
[20220303-191529  OUTPUT] クラスターを開始 [clusterName = sr-c1]
[20220303-191529    INFO] FEクラスターの開始 ....
[20220303-191529    INFO] FEノードの開始 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191600    INFO] FEノードの開始 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191610    INFO] FEノードの開始 [FeHost = 192.168.xx.xx, EditLogPort = 9010]

[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --role BE
[20220303-194215  OUTPUT] クラスターを開始 [clusterName = sr-c1]
[20220303-194215    INFO] BEノードの開始 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194216    INFO] BEノードの開始 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] BEノードの開始 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] BEクラスターの開始 ...
```

### 特定のノードを開始

クラスター内の特定のノードを開始します。現在、BEノードのみがサポートされています。

```shell
./sr-ctl cluster start <cluster_name> --node <node_ID>
```

特定のノードのIDは、[特定のクラスターの情報を表示](#view-the-information-of-a-specific-cluster)して確認できます。

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --node 192.168.xx.xx:9060
[20220303-194714  OUTPUT] クラスターを開始 [clusterName = sr-c1]
[20220303-194714    INFO] BEノードを開始 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

## クラスターの終了

StarGoを使用してStarRocksクラスターを終了できます。

### クラスター内のすべてのノードを終了

以下のコマンドを実行してクラスター内のすべてのノードを終了します。

```shell
./sr-ctl cluster stop <cluster_name>
```

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1
[20220302-180140  OUTPUT] クラスターを終了 [clusterName = sr-c1]
[20220302-180140  OUTPUT] クラスター sr-c1を終了
[20220302-180140    INFO] FEノードの終了待機 [FeHost = 192.168.xx.xx]
[20220302-180143  OUTPUT] FEノードが正常に終了しました [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180143    INFO] FEノードの終了待機 [FeHost = 192.168.xx.xx]
[20220302-180145  OUTPUT] FEノードが正常に終了しました [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180145    INFO] FEノードの終了待機 [FeHost = 192.168.xx.xx]
[20220302-180148  OUTPUT] FEノードが正常に終了しました [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180148  OUTPUT] クラスター sr-c1を終了
[20220302-180148    INFO] BEノードの終了待機 [BeHost = 192.168.xx.xx]
[20220302-180148    INFO] BEノードが正常に終了しました [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180148    INFO] BEノードの終了待機 [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] BEノードが正常に終了しました [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180149    INFO] BEノードの終了待機 [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] BEノードが正常に終了しました [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 特定の役割のノードを終了

- クラスター内のすべてのFEノードを終了します。

```shell
./sr-ctl cluster stop <cluster_name> --role FE
```

- クラスター内のすべてのBEノードを終了します。

```shell
./sr-ctl cluster stop <cluster_name> --role BE
```

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1 --role BE
[20220302-180624  OUTPUT] クラスターを終了 [clusterName = sr-c1]
[20220302-180624  OUTPUT] クラスター sr-c1を終了
[20220302-180624    INFO] BEノードの終了待機 [BeHost = 192.168.xx.xx]
[20220302-180624    INFO] BEノードが正常に終了しました [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180624    INFO] BEノードの終了待機 [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] BEノードが正常に終了しました [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] BEノードの終了待機 [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] BEノードが正常に終了しました [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] BEクラスターの終了 ...
```

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1 --role FE
[20220302-180849  OUTPUT] クラスターを終了 [clusterName = sr-c1]
[20220302-180849    INFO] FEクラスターの終了 ....
[20220302-180849  OUTPUT] クラスター sr-c1を終了
[20220302-180849    INFO] FEノードの終了待機 [FeHost = 192.168.xx.xx]
```
[20220302-180854  OUTPUT] FEノードは正常に停止しました [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180854    INFO] FEノードの停止を待っています [FeHost = 192.168.xx.xx]
[20220302-180856  OUTPUT] FEノードは正常に停止しました [host = 192.168.xx.xx, queryPort = 9030]
```

### 特定のノードを停止する

クラスター内の特定のノードを停止します。

```shell
./sr-ctl cluster stop <cluster_name> --node <node_ID>
```

[特定のクラスターの情報を表示する](#view-the-information-of-a-specific-cluster)ことで、特定のノードのIDを確認できます。

例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster display sr-c1
[20220303-185400  OUTPUT] クラスターを表示 [clusterName = sr-c1]
clusterName = sr-c1
[20220303-185400    WARN] すべてのFEノードがダウンしています。FEノードを起動して、クラスターの状態を再度表示してください。
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage

[root@nd1 sr-controller]# ./sr-ctl cluster stop sr-c1 --node 192.168.xx.xx:9060
[20220303-185510  OUTPUT] クラスターを停止 [clusterName = sr-c1]
[20220303-185510    INFO] BEノードの停止中 [BeHost = 192.168.xx.xx]
[20220303-185510    INFO] BEノードの停止を待っています [BeHost = 192.168.xx.xx]
```

## クラスターをスケールアウトする

StarGoを使用してクラスターをスケールアウトできます。

### 設定ファイルを作成する

以下のテンプレートに基づいて、スケールアウトタスクのトポロジファイルを作成します。需要に応じて、このファイルを使用してFEおよび/またはBEノードを追加できます。詳細については、[構成](../administration/Configuration.md)を参照してください。

```yaml
# FEノードを追加します。
fe_servers:
  - host: 192.168.xx.xx # 新しいFEノードのIPアドレス
    ssh_port: 22
    http_port: 8030
    rpc_port: 9020
    query_port: 9030
    edit_log_port: 9010
    deploy_dir: StarRocks/fe
    meta_dir: StarRocks/fe/meta
    log_dir: StarRocks/fe/log
    priority_networks: 192.168.xx.xx/24 # マシンに複数のIPアドレスがある場合は、現在のノードに固有のIPを指定します。
    config:
      sys_log_level: "INFO"
      sys_log_delete_age: "1d"

# BEノードを追加します。
be_servers:
  - host: 192.168.xx.xx # 新しいBEノードのIPアドレス
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

### SSH相互認証を構築する

新しいノードをクラスターに追加する場合、新しいノードと中央管理ノードとの間で相互認証を構築する必要があります。詳細な手順については、[前提条件](#prerequisites)を参照してください。

### デプロイディレクトリを作成する（オプション）

新しいノードをデプロイするパスが存在しない場合、およびそのようなパスを作成する権限がある場合は、これらのパスを作成する必要はありません。StarGoは構成ファイルに基づいてこれらを作成します。パスがすでに存在する場合は、それらへの書き込みアクセス権を持っていることを確認してください。また、次のコマンドを実行して各ノードで必要なデプロイディレクトリを作成することもできます。

- FEノードに**meta**ディレクトリを作成します。

```shell
mkdir -p StarRocks/fe/meta
```

- BEノードに**storage**ディレクトリを作成します。

```shell
mkdir -p StarRocks/be/storage
```

> 注意
> 上記のパスが構成ファイルの`meta_dir`および`storage_dir`と一致していることを確認してください。

### クラスターをスケールアウトする

次のコマンドを実行して、クラスターをスケールアウトします。

```shell
./sr-ctl cluster scale-out <cluster_name> <topology_file>
```

例：

```plain text
# スケールアウト前のクラスターの状態
[root@nd1 sr-controller]# ./sr-ctl cluster display sr-test       
[20220503-210047  OUTPUT] クラスターを表示 [clusterName = sr-test]
clusterName = sr-test
clusterVerison = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    

# クラスターをスケールアウトする
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster scale-out sr-test sr-out.yaml
[20220503-213725  OUTPUT] クラスターをスケールアウト [ClusterName = sr-test]
[20220503-213731  OUTPUT] デプロイ環境の事前チェック：
FEの事前チェック：
IP                    ssh auth         meta dir                        deploy dir                      http port        rpc port         query port       edit log port  
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                            PASS                            PASS             PASS             PASS             PASS           

BEの事前チェック：
IP                    ssh auth         storage dir                     deploy dir                      webSer port      heartbeat port   brpc port        be port        
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                            PASS                            PASS             PASS             PASS             PASS           


[20220503-213731  OUTPUT] 事前チェックが正常に完了しました。RESPECT
[20220503-213731  OUTPUT] デプロイフォルダを作成しています…
[20220503-213732  OUTPUT] StarRocksパッケージとJDKをダウンロードしています…
[20220503-213732    INFO] パッケージは既に存在します [fileName = starrocks-2.0.1-quickstart.tar.gz, fileSize = 1227406189, fileModTime = 2022-05-03 17:32:03.478661923 +0800 CST]
[20220503-213732  OUTPUT] ダウンロード完了。
[20220503-213732  OUTPUT] StarRocksパッケージとJDKを展開しています…
[20220503-213741    INFO] tarファイル /home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gz は、/home/sr-dev/.starrocks-controller/download以下に展開されました
[20220503-213837    INFO] tarファイル /home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gz は、/home/sr-dev/.starrocks-controller/download以下に展開されました
[20220503-213837    INFO] tarファイル /home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gz は、/home/sr-dev/.starrocks-controller/download以下に展開されました
[20220503-213837  OUTPUT] FEディレクトリを配布しています…
[20220503-213845    INFO] feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] を feTargetDir = [StarRocks/fe] に FeHost = [192.168.xx.xx] へアップロードしています
```
```plaintext
[20220503-213857    INFO] JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] を JDKTargetDir = [StarRocks/fe/jdk] に FeHost = [192.168.xx.xx] へアップロードしました
[20220503-213857    INFO] JAVA_HOME を修正しました: host = [192.168.xx.xx], filePath = [StarRocks/fe/bin/start_fe.sh]
[20220503-213857  OUTPUT] BE ディレクトリの配布中...
[20220503-213924    INFO] BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] を BeTargetDir = [StarRocks/be] に BeHost = [192.168.xx.xx] へアップロードしました
[20220503-213924  OUTPUT] FE ノードおよび BE ノードの構成を修正中...
############################################# FE クラスタのスケールアウト #############################################
############################################# FE クラスタのスケールアウト #############################################
[20220503-213925    INFO] フォロワー FE ノードを開始中 [host = 192.168.xx.xx, editLogPort = 9010]
[20220503-213945    INFO] FE ノードを正常に開始しました [host = 192.168.xx.xx, queryPort = 9030]
[20220503-213945    INFO] すべての FE ステータスをリスト化中:
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true

############################################# BE クラスタの開始 #############################################
############################################# BE クラスタの開始 #############################################
[20220503-213945    INFO] BE ノードを開始中 [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220503-214016    INFO] BE ノードを正常に開始しました [host = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220503-214016  OUTPUT] すべての BE ステータスをリスト化中:
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true

# スケールアウト後のクラスタのステータス
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-test 
[20220503-214302  OUTPUT] クラスタを表示 [clusterName = sr-test]
clusterName = sr-test
clusterVerison = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   StarRocks/fe/meta                            
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   StarRocks/be/storage                         

```

# クラスタ内のスケールアウト

次のコマンドを実行して、クラスタ内のノードを削除します。

```shell
./sr-ctl cluster scale-in <cluster_name> --node <node_id>
```

特定のノードの ID は[特定のクラスタの情報を表示](#view-the-information-of-a-specific-cluster)して確認できます。

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-c1
[20220505-145649  OUTPUT] クラスタを表示 [clusterName = sr-c1]
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
[20220621-010553  OUTPUT] クラスタを縮小 [clusterName = sr-c1, nodeId = 192.168.88.83:9010]
[20220621-010553    INFO] FE ノードの停止を待機中 [FeHost = 192.168.88.83]
[20220621-010606  OUTPUT] FE ノードの縮小に成功しました [clusterName = sr-c1, nodeId = 192.168.88.83:9010]

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-c1
[20220621-010623  OUTPUT] クラスタを表示 [clusterName = sr-c1]
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

## クラスタのアップグレードまたはダウングレード

StarGo を使用してクラスタをアップグレードまたはダウングレードできます。

- クラスタをアップグレード

```shell
./sr-ctl cluster upgrade <cluster_name>  <target_version>
```

- クラスタをダウングレード

```shell
./sr-ctl cluster downgrade <cluster_name>  <target_version>
```

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster list
[20220515-195827  OUTPUT] すべてのクラスタをリスト化
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 19:35:36        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster upgrade sr-test2 v2.1.3
[20220515-200358  OUTPUT] すべてのクラスタをリスト化
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.1.3      test222     2022-05-15 20:03:01        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster downgrade sr-test2 v2.0.1 
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster list
[20220515-200915  OUTPUT] すべてのクラスタをリスト化
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 20:08:40        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                
```
```
|Command|Description|
|----|----|
|deploy|クラスターをデプロイします。|
|start|クラスターを起動します。|
|stop|クラスターを停止します。|
|scale-in|クラスターを縮小します。|
|scale-out|クラスターを拡張します。|
|upgrade|クラスターをアップグレードします。|
|downgrade|クラスターをダウングレードします。|
|display|特定のクラスターの情報を表示します。|
|list|すべてのクラスターを表示します。|
```