---
displayed_sidebar: English
sidebar_position: 1
---

# Dockerを使ってStarRocksをデプロイする

このクイックスタートチュートリアルでは、Dockerを使用してローカルマシンにStarRocksをデプロイする手順を案内します。始める前に、[StarRocksのアーキテクチャ](../introduction/Architecture.md)についてもっと概念的な詳細を読むことができます。

これらのステップに従って、**1つのFEノード**と**1つのBEノード**を持つシンプルなStarRocksクラスターをデプロイすることができます。これにより、[テーブルの作成](../quick_start/Create_table.md)や[データのロードとクエリ](../quick_start/Import_and_query.md)に関する今後のクイックスタートチュートリアルを完了し、StarRocksの基本的な操作に慣れることができます。

> **注意**
>
> このチュートリアルで使用されるDockerイメージでStarRocksをデプロイすることは、小規模なデータセットでデモを検証する必要がある場合にのみ適しています。大規模なテストや本番環境には推奨されません。高可用性のStarRocksクラスターをデプロイするには、[デプロイメントの概要](../deployment/deployment_overview.md)を参照して、シナリオに合った他のオプションを確認してください。

## 前提条件

DockerでStarRocksをデプロイする前に、以下の要件が満たされていることを確認してください：

- **ハードウェア**

  StarRocksは、8つのCPUコアと16GB以上のメモリを搭載したマシンにデプロイすることを推奨します。

- **ソフトウェア**

  あなたのマシンには以下のソフトウェアがインストールされている必要があります：

  - [Docker Engine](https://docs.docker.com/engine/install/)（バージョン17.06.0以降）メタディレクトリのディスクパーティションに少なくとも5GBの空き容量が必要です。詳細は[こちら](https://github.com/StarRocks/starrocks/issues/35608)を参照してください。
  - MySQLクライアント（バージョン5.5以降）

## ステップ1：StarRocksのDockerイメージをダウンロード

[StarRocks Docker Hub](https://hub.docker.com/r/starrocks/allin1-ubuntu/tags)からStarRocksのDockerイメージをダウンロードします。イメージのタグに基づいて特定のバージョンを選択できます。

```Bash
sudo docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 \
    -itd starrocks/allin1-ubuntu
```

> **トラブルシューティング**
>
> ホストマシンの上記のポートが既に使用されている場合、システムは「docker: Error response from daemon: driver failed programming external connectivity on endpoint tender_torvalds (): Bind for 0.0.0.0:xxxx failed: port is already allocated.」というエラーメッセージを出力します。コマンド内のコロン(:)の前のポートを変更することで、ホストマシン上で利用可能なポートを割り当てることができます。

次のコマンドを実行して、コンテナが正しく作成され、実行中であるかを確認できます：

```Bash
sudo docker ps
```

以下に示すように、StarRocksコンテナの`STATUS`が`Up`であれば、Dockerコンテナ内にStarRocksが正常にデプロイされています。

```Plain
CONTAINER ID   IMAGE                                          COMMAND                  CREATED         STATUS                 PORTS                                                                                                                             NAMES
8962368f9208   starrocks/allin1-ubuntu:branch-3.0-0afb97bbf   "/bin/sh -c ./start_…"   4 minutes ago   Up 4 minutes           0.0.0.0:8037->8030/tcp, :::8037->8030/tcp, 0.0.0.0:8047->8040/tcp, :::8047->8040/tcp, 0.0.0.0:9037->9030/tcp, :::9037->9030/tcp   xxxxx
```

## ステップ2：StarRocksに接続

StarRocksが適切にデプロイされた後、MySQLクライアントを介して接続することができます。

```Bash
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

> **注意**
>
> `docker run`コマンドで`9030`のポートを異なるものに割り当てた場合、上記のコマンドで`9030`を割り当てたポートに置き換える必要があります。

以下のSQLを実行してFEノードの状態を確認できます：

```SQL
SHOW PROC '/frontends'\G
```

例：

```Plain
StarRocks > SHOW PROC '/frontends'\G
*************************** 1. row ***************************
             Name: 8962368f9208_9010_1681370634632
               IP: 8962368f9208
      EditLogPort: 9010
         HttpPort: 8030
        QueryPort: 9030
          RpcPort: 9020
             Role: LEADER
        ClusterId: 555505802
             Join: true
            Alive: true
ReplayedJournalId: 99
    LastHeartbeat: 2023-04-13 07:28:50
         IsHelper: true
           ErrMsg: 
        StartTime: 2023-04-13 07:24:11
          Version: BRANCH-3.0-0afb97bbf
1 row in set (0.02 sec)
```

- `Alive`フィールドが`true`であれば、このFEノードは正常に起動し、クラスタに追加されています。
- `Role`フィールドが`FOLLOWER`であれば、このFEノードはリーダーFEノードに選出される可能性があります。
- `Role`フィールドが`LEADER`であれば、このFEノードはリーダーFEノードです。

以下のSQLを実行してBEノードの状態を確認できます：

```SQL
SHOW PROC '/backends'\G
```

例：

```Plain
StarRocks > SHOW PROC '/backends'\G
*************************** 1. row ***************************
            BackendId: 10004
                   IP: 8962368f9208
        HeartbeatPort: 9050
               BePort: 9060
             HttpPort: 8040
             BrpcPort: 8060
        LastStartTime: 2023-04-13 07:24:25
        LastHeartbeat: 2023-04-13 07:29:05
                Alive: true
 SystemDecommissioned: false
ClusterDecommissioned: false
            TabletNum: 30
     DataUsedCapacity: 0.000 
        AvailCapacity: 527.437 GB
        TotalCapacity: 1.968 TB
              UsedPct: 73.83 %
       MaxDiskUsedPct: 73.83 %
               ErrMsg: 
              Version: BRANCH-3.0-0afb97bbf
               Status: {"lastSuccessReportTabletsTime":"2023-04-13 07:28:26"}
    DataTotalCapacity: 527.437 GB
          DataUsedPct: 0.00 %
             CpuCores: 16
    NumRunningQueries: 0
           MemUsedPct: 0.02 %
           CpuUsedPct: 0.1 %
1 row in set (0.00 sec)
```

`Alive`フィールドが`true`であれば、このBEノードは正常に起動し、クラスタに追加されています。

## Dockerコンテナを停止して削除する

クイックスタートチュートリアルをすべて完了した後、StarRocksクラスタをホストするコンテナをそのコンテナIDで停止して削除することができます。

> **注記**
>
> `sudo docker ps`を実行することでDockerコンテナの`container_id`を取得できます。

コンテナを停止するには、次のコマンドを実行します：

```Bash
# <container_id>をあなたのStarRocksクラスタのコンテナIDに置き換えてください。
sudo docker stop <container_id>
```

コンテナがもはや必要ない場合は、次のコマンドを実行して削除することができます：

```Bash
# <container_id>をあなたのStarRocksクラスタのコンテナIDに置き換えてください。
sudo docker rm <container_id>
```

> **注意**
>
> コンテナの削除は取り消しできません。削除する前に、コンテナ内の重要なデータのバックアップを取っておくことを確認してください。

## 次に何をするか

StarRocksをデプロイしたら、[テーブルの作成](../quick_start/Create_table.md)や[データのロードとクエリ](../quick_start/Import_and_query.md)に関するクイックスタートチュートリアルを続けることができます。

<img referrerpolicy="no-referrer-when-downgrade" src="https://static.scarf.sh/a.png?x-pxid=f5ae0b2c-3578-4a40-9056-178e9837cfe0" />
