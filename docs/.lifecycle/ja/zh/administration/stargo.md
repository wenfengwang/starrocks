---
displayed_sidebar: Chinese
---

# StarGo を使用して StarRocks をデプロイおよび管理する

本文では、StarGo を使用して StarRocks クラスタをデプロイおよび管理する方法について説明します。

> **説明**
>
> **現在、StarGo はコミュニティによって改良されており、クラスタのデプロイにはコミュニティの[最新バージョン](https://forum.mirrorship.cn/t/topic/4945)の使用を推奨します。改良プロセスで迅速に反復され、機能調整が頻繁に行われるため、コミュニティの改良作業が基本的に完了した後に、本文書を最終更新する予定です。**

**以下は旧バージョンに基づくデプロイ操作であり、現在は使用を推奨しません**。

StarGo は、複数の StarRocks クラスタを管理するためのコマンドラインツールです。StarGo を使用すると、簡単なコマンドラインで複数のクラスタのデプロイ、表示、アップグレード、起動および停止などの操作を実行できます。このツールはバージョン 2.3 からサポートされています。

## StarGo のデプロイ

現在のユーザーパスで StarGo のバイナリインストールパッケージをダウンロードして解凍します。

```shell
wget https://raw.githubusercontent.com/wangtianyi2004/starrocks-controller/main/stargo-pkg.tar.gz
tar -xzvf stargo-pkg.tar.gz
```

インストールパッケージには以下のファイルが含まれています。

- **stargo**：インストール不要の StarGo バイナリファイル。
- **deploy-template.yaml**：デプロイ設定ファイルのテンプレート。
- **repo.yaml**：StarRocks のインストールパッケージをダウンロードするための設定ファイル。

## クラスタのデプロイ

StarGo を使用して StarRocks クラスタをデプロイできます。

### 前提条件

- デプロイ予定のクラスタには、少なくとも1つのコントロールノードと3つのデプロイノードが必要で、すべてのノードは同一マシンに混在してデプロイできます。
- コントロールノードには StarGo をデプロイする必要があります。
- コントロールノードとデプロイノード間には SSH 信頼関係を作成する必要があります。

以下の例では、コントロールノード sr-dev@r0 とデプロイノード starrocks@r1、starrocks@r2、および starrocks@r3 間の SSH 信頼関係を作成しています。

```plain text
## sr-dev@r0 から starrocks@r1、r2、r3 への SSH 信頼関係を作成します。
[sr-dev@r0 ~]$ ssh-keygen
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r1
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r2
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r3

## sr-dev@r0 から starrocks@r1、r2、r3 への SSH 信頼関係を確認します。
[sr-dev@r0 ~]$ ssh starrocks@r1 date
[sr-dev@r0 ~]$ ssh starrocks@r2 date
[sr-dev@r0 ~]$ ssh starrocks@r3 date
```

### 設定ファイルの作成

以下の YAML テンプレートに基づいて、StarRocks クラスタのデプロイ用トポロジーファイルを作成します。具体的な設定項目は[パラメータ設定](../administration/FE_configuration.md)を参照してください。

```yaml
global:
    user: "starrocks"   # 現在のオペレーティングシステムのユーザーに変更してください。
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
    priority_networks: 192.168.XX.XX/24 # マシンに複数の IP がある場合は、この設定項目で現在のノードに固有の IP を指定してください。
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
    priority_networks: 192.168.XX.XX/24 # マシンに複数の IP がある場合、この設定項目で現在のノードに固有の IP を指定してください。
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
    priority_networks: 192.168.XX.XX/24 # マシンに複数の IP がある場合、この設定項目で現在のノードに固有の IP を指定してください。
    config:
      sys_log_level: "INFO"
be_servers:
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    webserver_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir: StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数の IP がある場合、この設定項目で現在のノードに固有の IP を指定してください。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    webserver_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir: StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数の IP がある場合、この設定項目で現在のノードに固有の IP を指定してください。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    webserver_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir: StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # マシンに複数の IP がある場合、この設定項目で現在のノードに固有の IP を指定してください。
    config:
      create_tablet_worker_count: 3
```

### デプロイディレクトリの作成（オプション）

設定ファイルで指定されたデプロイパスが存在しない場合、かつ作成権限がある場合は、StarGo が設定ファイルに基づいてデプロイディレクトリを自動的に作成します。パスが既に存在する場合は、そのパスで書き込み権限を持っていることを確認してください。また、以下のコマンドを使用して、各デプロイノードでデプロイパスを個別に作成することもできます。

- FE ノードのインストールディレクトリに **meta** パスを作成します。

```shell
mkdir -p StarRocks/fe/meta
```

- BE ノードのインストールディレクトリに **storage** パスを作成します。

```shell
mkdir -p StarRocks/be/storage
```

> 注意
>
> 上記で作成したパスが設定ファイルの `meta_dir` および `storage_dir` と一致していることを確認してください。

### StarRocks のデプロイ

以下のコマンドを使用して StarRocks クラスタをデプロイします。

```shell
./stargo cluster deploy <cluster_name> <version> <topology_file>
```

|パラメータ|説明|
|----|----|
|cluster_name|作成するクラスタ名|
|version|StarRocks のバージョン|
|topology_file|設定ファイル名|

クラスタの作成に成功すると、自動的に起動します。beStatus と feStatus が true を返した場合、クラスタのデプロイと起動が成功したことを意味します。

例：

```plain text
[sr-dev@r0 ~]$ ./stargo cluster deploy sr-c1 v2.0.1 sr-c1.yaml
[20220301-234817  OUTPUT] Deploy cluster [clusterName = sr-c1, clusterVersion = v2.0.1, metaFile = sr-c1.yaml]
[20220301-234836  OUTPUT] PRE CHECK DEPLOY ENV:
PreCheck FE:
IP                    ssh auth         meta dir                   deploy dir                 http port        rpc port         query port       edit log port
--------------------  ---------------  -------------------------  -------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS

PreCheck BE:
IP                    ssh auth         storage dir                deploy dir                 webserver port   heartbeat port   brpc port        be port
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
[20220302-000603    INFO] Upload dir feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] to feTargetDir = [StarRocks/fe] on FeHost = [192.168.xx.xx]

[20220302-000615    INFO] JDKSourceDirをアップロードします = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] に JDKTargetDir = [StarRocks/fe/jdk] on FeHost = [192.168.xx.xx]
[20220302-000615    INFO] JAVA_HOMEを変更します: ホスト = [192.168.xx.xx], ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000622    INFO] feSourceDirをアップロードします = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] に feTargetDir = [StarRocks/fe] on FeHost = [192.168.xx.xx]
[20220302-000634    INFO] JDKSourceDirをアップロードします = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] に JDKTargetDir = [StarRocks/fe/jdk] on FeHost = [192.168.xx.xx]
[20220302-000634    INFO] JAVA_HOMEを変更します: ホスト = [192.168.xx.xx], ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000640    INFO] feSourceDirをアップロードします = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] に feTargetDir = [StarRocks/fe] on FeHost = [192.168.xx.xx]
[20220302-000652    INFO] JDKSourceDirをアップロードします = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] に JDKTargetDir = [StarRocks/fe/jdk] on FeHost = [192.168.xx.xx]
[20220302-000652    INFO] JAVA_HOMEを変更します: ホスト = [192.168.xx.xx], ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220302-000652  OUTPUT] BEディレクトリを配布します...
[20220302-000728    INFO] BeSourceDirをアップロードします = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] に BeTargetDir = [StarRocks/be] on BeHost = [192.168.xx.xx]
[20220302-000752    INFO] BeSourceDirをアップロードします = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] に BeTargetDir = [StarRocks/be] on BeHost = [192.168.xx.xx]
[20220302-000815    INFO] BeSourceDirをアップロードします = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] に BeTargetDir = [StarRocks/be] on BeHost = [192.168.xx.xx]
[20220302-000815  OUTPUT] FEノードとBEノードの設定を変更します...
############################################# FEクラスタを開始 #############################################
############################################# FEクラスタを開始 #############################################
[20220302-000816    INFO] リーダーFEノードを開始します [ホスト = 192.168.xx.xx, editLogPort = 9010]
[20220302-000836    INFO] FEノードが正常に開始しました [ホスト = 192.168.xx.xx, queryPort = 9030]
[20220302-000836    INFO] フォロワーFEノードを開始します [ホスト = 192.168.xx.xx, editLogPort = 9010]
[20220302-000857    INFO] FEノードが正常に開始しました [ホスト = 192.168.xx.xx, queryPort = 9030]
[20220302-000857    INFO] フォロワーFEノードを開始します [ホスト = 192.168.xx.xx, editLogPort = 9010]
[20220302-000918    INFO] FEノードが正常に開始しました [ホスト = 192.168.xx.xx, queryPort = 9030]
[20220302-000918    INFO] すべてのFEステータスをリストします:
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
[20220302-001020  OUTPUT] すべてのBEステータスをリストします:
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
```

特定のクラスタ情報を表示することで、各ノードが正常にデプロイされたかどうかを確認できます。

MySQLクライアントを接続して、クラスタが正常にデプロイされたかどうかもテストできます。

```shell
mysql -h 127.0.0.1 -P9030 -uroot
```

## クラスタ情報を表示

StarGoを使用して、管理しているクラスタ情報を表示できます。

### すべてのクラスタ情報を表示

以下のコマンドを使用して、管理しているすべてのクラスタ情報を表示します。

```shell
./stargo cluster list
```

例：

```shell
[sr-dev@r0 ~]$ ./stargo cluster list
[20220302-001640  OUTPUT] すべてのクラスタをリストします
ClusterName      User        CreateDate                 MetaPath                                                      PrivateKey
---------------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-c1            starrocks   2022-03-02 00:08:15        /home/sr-dev/.starrocks-controller/cluster/sr-c1              /home/sr-dev/.ssh/id_rsa
```

### 特定のクラスタ情報を表示

以下のコマンドを使用して、特定のクラスタ情報を表示します。

```shell
./stargo cluster display <cluster_name>
```

例：

```plain text
[sr-dev@r0 ~]$ ./stargo cluster display sr-c1
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

## クラスタを起動

StarGoを使用して、管理しているクラスタを起動できます。

### クラスタのすべてのノードを起動

以下のコマンドを使用して、特定のクラスタのすべてのノードを起動します。

```shell
./stargo cluster start <cluster-name>
```

例：

```plain text
[root@nd1 sr-controller]# ./stargo cluster start sr-c1
[20220303-190404  OUTPUT] クラスタを起動 [clusterName = sr-c1]
[20220303-190404    INFO] FEノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190435    INFO] FEノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190446    INFO] FEノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190457    INFO] BEノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] BEノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] BEノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### クラスタ内の特定の役割のノードを起動

- 特定のクラスタ内のFEノードを起動するには、以下のコマンドを使用します。

```shell
./stargo cluster start <cluster_name> --role FE
```

- 特定のクラスタ内のBEノードを起動するには、以下のコマンドを使用します。

```shell
./stargo cluster start <cluster_name> --role BE
```

例：

```plain text
[root@nd1 sr-controller]# ./stargo cluster start sr-c1 --role FE
[20220303-191529  OUTPUT] クラスタを起動 [clusterName = sr-c1]
[20220303-191529    INFO] FEクラスタを起動...
[20220303-191529    INFO] FEノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191600    INFO] FEノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191610    INFO] FEノードを起動 [FeHost = 192.168.xx.xx, EditLogPort = 9010]

[root@nd1 sr-controller]# ./stargo cluster start sr-c1 --role BE
[20220303-194215  OUTPUT] クラスタを起動 [clusterName = sr-c1]
[20220303-194215    INFO] BEノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194216    INFO] BEノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] BEノードを起動 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] BEクラスタを起動...
```

### クラスタ内の特定のノードを起動

以下のコマンドを使用して、クラスタの特定のノードを起動します。現在、BEノードのみの起動がサポートされています。

```shell
./stargo cluster start <cluster_name> --node <node_ID>
```


特定ノードのIDを確認するには、[指定されたクラスター情報を表示](#指定されたクラスター情報を表示)します。

例：

```plain text
[root@nd1 sr-controller]# ./stargo cluster start sr-c1 --node 192.168.xx.xx:9060
[20220303-194714  OUTPUT] Start cluster [clusterName = sr-c1]
[20220303-194714    INFO] Start BE node. [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

## クラスターの停止

StarGoを使用して管理しているクラスターを停止できます。

### クラスターの全ノードを停止

以下のコマンドで特定のクラスターの全ノードを停止します。

```shell
./stargo cluster stop <cluster_name>
```

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./stargo cluster stop sr-c1
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

### クラスター内の特定の役割のノードを停止

- 以下のコマンドで特定のクラスター内のFEノードを停止します。

```shell
./stargo cluster stop <cluster_name> --role FE
```

- 以下のコマンドで特定のクラスター内のBEノードを停止します。

```shell
./stargo cluster stop <cluster_name> --role BE
```

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./stargo cluster stop sr-c1 --role BE
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

[sr-dev@nd1 sr-controller]$ ./stargo cluster stop sr-c1 --role FE
[20220302-180849  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220302-180849    INFO] Stopping FE cluster ....
[20220302-180849  OUTPUT] Stop cluster sr-c1
[20220302-180849    INFO] Waiting for stopping FE node [FeHost = 192.168.xx.xx]
[20220302-180851  OUTPUT] The FE node stopped successfully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180851    INFO] Waiting for stopping FE node [FeHost = 192.168.xx.xx]
[20220302-180854  OUTPUT] The FE node stopped successfully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180854    INFO] Waiting for stopping FE node [FeHost = 192.168.xx.xx]
[20220302-180856  OUTPUT] The FE node stopped successfully [host = 192.168.xx.xx, queryPort = 9030]
```

### クラスター内の特定のノードを停止

以下のコマンドでクラスターの特定のノードを停止します。

```shell
./stargo cluster stop <cluster_name> --node <node_ID>
```

特定ノードのIDを確認するには、[指定されたクラスター情報を表示](#指定されたクラスター情報を表示)します。

例：

```plain text
[root@nd1 sr-controller]# ./stargo cluster display sr-c1
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

[root@nd1 sr-controller]# ./stargo cluster stop sr-c1 --node 192.168.xx.xx:9060
[20220303-185510  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220303-185510    INFO] Stopping BE node. [BeHost = 192.168.xx.xx]
[20220303-185510    INFO] Waiting for stopping BE node [BeHost = 192.168.xx.xx]
```

## クラスターの拡張

StarGoを使用して管理しているクラスターを拡張できます。

### 設定ファイルの作成

以下のYAMLテンプレートに基づいて、StarRocksクラスターを拡張するためのトポロジーファイルを作成します。必要に応じてFEおよび/またはBEノードを設定できます。具体的な設定項目は[パラメータ設定](../administration/FE_configuration.md)を参照してください。

```yaml
# FEノードの拡張。
fe_servers:
  - host: 192.168.xx.xx # 拡張するFEノードのIPアドレス。
    ssh_port: 22
    http_port: 8030
    rpc_port: 9020
    query_port: 9030
    edit_log_port: 9010
    deploy_dir: StarRocks/fe
    meta_dir: StarRocks/fe/meta
    log_dir: StarRocks/fe/log
    priority_networks: 192.168.xx.xx/24 # 複数のIPがある場合、この設定項目でノードに固有のIPを指定します。
    config:
      sys_log_level: "INFO"
      sys_log_delete_age: "1d"

# BEノードの拡張。
be_servers:
  - host: 192.168.xx.xx # 拡張するBEノードのIPアドレス。
    ssh_port: 22
    be_port: 9060
    webserver_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    config:
      create_tablet_worker_count: 3
```

### SSH相互認証の作成

拡張ノードが新しいノードである場合、SSHノード間の相互認証を設定する必要があります。詳細な操作は[前提条件](#前提条件)を参照してください。

### デプロイディレクトリの作成（オプション）

設定ファイルで指定されたデプロイパスが存在しない場合、かつそのパスを作成する権限がある場合、StarGoは設定ファイルに基づいて自動的にデプロイディレクトリを作成します。パスが既に存在する場合は、そのパスに書き込み権限があることを確認してください。また、以下のコマンドを使用して、各デプロイノードでデプロイパスを個別に作成することもできます。

- 新しいFEノードのインストールディレクトリに**meta**パスを作成します。

```shell
mkdir -p StarRocks/fe/meta
```

- 新しいBEノードのインストールディレクトリに**storage**パスを作成します。

```shell
mkdir -p StarRocks/be/storage
```

> 注意
>
> 上記で作成したパスが設定ファイルの`meta_dir`および`storage_dir`と一致していることを確認してください。

### StarRocksクラスターの拡張

以下のコマンドでクラスターを拡張します。

```shell
./stargo cluster scale-out <cluster_name> <topology_file>
```

例：

```plain text
# 現在のクラスター状態。
[root@nd1 sr-controller]# ./stargo cluster display sr-test       
[20220503-210047  OUTPUT] Display cluster [clusterName = sr-test]
clusterName = sr-test
clusterVersion = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------

192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    

# クラスタの拡張
[sr-dev@nd1 sr-controller]$ ./stargo cluster scale-out sr-test sr-out.yaml
[20220503-213725  OUTPUT] クラスタを拡張します。[ClusterName = sr-test]
[20220503-213731  OUTPUT] 環境の事前チェックを実施中:
PreCheck FE:
IP                    ssh認証         metaディレクトリ                デプロイディレクトリ            httpポート        rpcポート         クエリポート       編集ログポート  
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                            PASS                            PASS             PASS             PASS             PASS           

PreCheck BE:
IP                    ssh認証         ストレージディレクトリ           デプロイディレクトリ            webサーバポート   ハートビートポート brpcポート        beポート        
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                            PASS                            PASS             PASS             PASS             PASS           


[20220503-213731  OUTPUT] 事前チェックに成功しました。RESPECT
[20220503-213731  OUTPUT] デプロイフォルダを作成中...
[20220503-213732  OUTPUT] StarRocksパッケージとJDKをダウンロード中...
[20220503-213732    INFO] パッケージは既に存在します [fileName = starrocks-2.0.1-quickstart.tar.gz, fileSize = 1227406189, fileModTime = 2022-05-03 17:32:03.478661923 +0800 CST]
[20220503-213732  OUTPUT] ダウンロード完了。
[20220503-213732  OUTPUT] StarRocksパッケージとJDKを解凍中...
[20220503-213741    INFO] tarファイル /home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gz は /home/sr-dev/.starrocks-controller/download に解凍されました
[20220503-213837    INFO] tarファイル /home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gz は /home/sr-dev/.starrocks-controller/download に解凍されました
[20220503-213837    INFO] tarファイル /home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gz は /home/sr-dev/.starrocks-controller/download に解凍されました
[20220503-213837  OUTPUT] FEディレクトリを配布中...
[20220503-213845    INFO] ディレクトリfeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] をFeHost = [192.168.xx.xx] のfeTargetDir = [StarRocks/fe] にアップロード中
[20220503-213857    INFO] ディレクトリJDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] をFeHost = [192.168.xx.xx] のJDKTargetDir = [StarRocks/fe/jdk] にアップロード中
[20220503-213857    INFO] JAVA_HOMEを変更中: ホスト = [192.168.xx.xx], ファイルパス = [StarRocks/fe/bin/start_fe.sh]
[20220503-213857  OUTPUT] BEディレクトリを配布中...
[20220503-213924    INFO] ディレクトリBeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] をBeHost = [192.168.xx.xx] のBeTargetDir = [StarRocks/be] にアップロード中
[20220503-213924  OUTPUT] FEノードとBEノードの設定を変更中...
############################################# FEクラスタの拡張 #############################################
############################################# FEクラスタの拡張 #############################################
[20220503-213925    INFO] フォロワーFEノードを起動中 [ホスト = 192.168.xx.xx, 編集ログポート = 9010]
[20220503-213945    INFO] FEノードが正常に起動しました [ホスト = 192.168.xx.xx, クエリポート = 9030]
[20220503-213945    INFO] 全てのFEのステータスを表示中:
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true

############################################# BEクラスタの起動 #############################################
############################################# BEクラスタの起動 #############################################
[20220503-213945    INFO] BEノードを起動中 [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220503-214016    INFO] BEノードが正常に起動しました [ホスト = 192.168.xx.xx, ハートビートサービスポート = 9050]
[20220503-214016  OUTPUT] 全てのBEのステータスを表示中:
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true

# 拡張後のクラスタ状態
[sr-dev@nd1 sr-controller]$ ./stargo cluster display sr-test 
[20220503-214302  OUTPUT] クラスタ情報を表示します [clusterName = sr-test]
clusterName = sr-test
clusterVerison = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   StarRocks/fe/meta                            
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   StarRocks/be/storage                         
```

## クラスタの縮小

以下のコマンドでクラスタ内の特定ノードを縮小します。

```shell
./stargo cluster scale-in <cluster_name> --node <node_id>
```

[指定したクラスタ情報を表示](#指定したクラスタ情報を表示)することで、クラスタ内の特定ノードのIDを確認できます。

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./stargo cluster display sr-c1
[20220505-145649  OUTPUT] クラスタ情報を表示します [clusterName = sr-c1]
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
[sr-dev@nd1 sr-controller]$ ./stargo cluster scale-in sr-c1 --node 192.168.88.83:9010
[20220621-010553  OUTPUT] クラスタの縮小 [clusterName = sr-c1, nodeId = 192.168.88.83:9010]
[20220621-010553    INFO] FEノードの停止を待機中 [FeHost = 192.168.88.83]
[20220621-010606  OUTPUT] FEノードの縮小に成功しました。[clusterName = sr-c1, nodeId = 192.168.88.83:9010]

[sr-dev@nd1 sr-controller]$ ./stargo cluster display sr-c1
[20220621-010623  OUTPUT] クラスタ情報を表示します [clusterName = sr-c1]
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

## クラスタのアップグレードとダウングレード

StarGoを使用して、管理しているクラスタをアップグレードまたはダウングレードできます。

- 以下のコマンドでクラスタをアップグレードします。

```shell
./stargo cluster upgrade <cluster_name> <target_version>
```

- 以下のコマンドでクラスタをダウングレードします。

```shell
./stargo cluster downgrade <cluster_name> <target_version>
```

例：

```plain text
[sr-dev@nd1 sr-controller]$ ./stargo cluster list
[20220515-195827  OUTPUT] 全てのクラスタをリストアップ
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 19:35:36        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          
[sr-dev@nd1 sr-controller]$ ./stargo cluster upgrade sr-test2 v2.1.3
[20220515-200358  OUTPUT] 全てのクラスタをリストアップ
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.1.3      test222     2022-05-15 20:03:01        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          

[sr-dev@nd1 sr-controller]$ ./stargo cluster downgrade sr-test2 v2.0.1 
[sr-dev@nd1 sr-controller]$ ./stargo cluster list
[20220515-200915  OUTPUT] 全てのクラスタをリストアップ
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 20:08:40        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                
```

## 関連コマンド

|コマンド|説明|
|----|----|
|deploy|クラスターをデプロイします。|
|start|クラスターを起動します。|
|stop|クラスターを停止します。|
|scale-in|クラスターを縮小します。|
|scale-out|クラスターを拡大します。|
|upgrade|クラスターをアップグレードします。|
|downgrade|クラスターをダウングレードします。|
|display|特定のクラスターを表示します。|
|list|すべてのクラスターを表示します。|
