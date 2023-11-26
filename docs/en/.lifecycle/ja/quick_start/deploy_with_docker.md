---
displayed_sidebar: "Japanese"
sidebar_position: 1
---

# Dockerを使用してStarRocksをデプロイする

このクイックスタートチュートリアルでは、Dockerを使用してローカルマシンにStarRocksをデプロイする手順を案内します。始める前に、[StarRocksのアーキテクチャ](../introduction/Architecture.md)を読んで、より概念的な詳細を確認することができます。

これらの手順に従って、**1つのFEノード**と**1つのBEノード**を持つシンプルなStarRocksクラスタをデプロイすることができます。これにより、[テーブルの作成](../quick_start/Create_table.md)や[データのロードとクエリ](../quick_start/Import_and_query.md)といった次のクイックスタートチュートリアルを完了し、StarRocksの基本操作に慣れることができます。

> **注意**
>
> このチュートリアルで使用するDockerイメージを使用してStarRocksをデプロイするのは、小規模なデータセットでのデモを検証する場合にのみ適用されます。大規模なテストや本番環境では推奨されません。高可用性のStarRocksクラスタをデプロイするには、シナリオに合わせた他のオプションについては[デプロイの概要](../deployment/deployment_overview.md)を参照してください。

## 前提条件

StarRocksをDockerでデプロイする前に、次の要件を満たしていることを確認してください。

- **ハードウェア**

  StarRocksを8つのCPUコアと16GB以上のメモリを持つマシンにデプロイすることをおすすめします。

- **ソフトウェア**

  マシンに次のソフトウェアがインストールされている必要があります。

  - [Docker Engine](https://docs.docker.com/engine/install/) (17.06.0以降) またはそれ以降のバージョンが必要です。また、metaディレクトリのディスクパーティションには少なくとも5GBの空き容量が必要です。詳細については、https://github.com/StarRocks/starrocks/issues/35608 を参照してください。
  - MySQLクライアント (5.5以降)

## ステップ1: StarRocks Dockerイメージをダウンロードする

[StarRocks Docker Hub](https://hub.docker.com/r/starrocks/allin1-ubuntu/tags)からStarRocksのDockerイメージをダウンロードします。イメージのタグに基づいて、特定のバージョンを選択することができます。

```Bash
sudo docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 \
    -itd starrocks/allin1-ubuntu
```

> **トラブルシューティング**
>
> 上記のホストマシンのポートのいずれかが使用中の場合、システムは「docker: Error response from daemon: driver failed programming external connectivity on endpoint tender_torvalds (): Bind for 0.0.0.0:xxxx failed: port is already allocated.」と表示されます。コマンド内のコロン（:）の前にあるポートを変更することで、ホストマシン上の利用可能なポートを割り当てることができます。

次のコマンドを実行して、コンテナが正常に作成されて実行されているかどうかを確認できます。

```Bash
sudo docker ps
```

以下のように表示される場合、StarRocksのコンテナが`Up`の`STATUS`であることを確認しています。

```Plain
CONTAINER ID   IMAGE                                          COMMAND                  CREATED         STATUS                 PORTS                                                                                                                             NAMES
8962368f9208   starrocks/allin1-ubuntu:branch-3.0-0afb97bbf   "/bin/sh -c ./start_…"   4 minutes ago   Up 4 minutes           0.0.0.0:8037->8030/tcp, :::8037->8030/tcp, 0.0.0.0:8047->8040/tcp, :::8047->8040/tcp, 0.0.0.0:9037->9030/tcp, :::9037->9030/tcp   xxxxx
```

## ステップ2: StarRocksに接続する

StarRocksが正常にデプロイされた後、MySQLクライアントを使用して接続することができます。

```Bash
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

> **注意**
>
> `docker run`コマンドで`9030`に異なるポートを割り当てた場合は、上記のコマンドの`9030`を割り当てたポートに置き換える必要があります。

次のSQLを実行して、FEノードのステータスを確認できます。

```SQL
SHOW PROC '/frontends'\G
```

例:

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

- `Alive`フィールドが`true`の場合、このFEノードは正常に起動され、クラスタに追加されています。
- `Role`フィールドが`FOLLOWER`の場合、このFEノードはリーダーFEノードに選出される資格があります。
- `Role`フィールドが`LEADER`の場合、このFEノードはリーダーFEノードです。

次のSQLを実行して、BEノードのステータスを確認できます。

```SQL
SHOW PROC '/backends'\G
```

例:

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

`Alive`フィールドが`true`の場合、このBEノードは正常に起動され、クラスタに追加されています。

## Dockerコンテナを停止および削除する

クイックスタートチュートリアル全体を完了した後は、StarRocksクラスタをホストするコンテナをコンテナIDで停止および削除することができます。

> **注意**
>
> Dockerコンテナの`container_id`は、`sudo docker ps`を実行して取得することができます。

次のコマンドを実行して、コンテナを停止します。

```Bash
# <container_id>をStarRocksクラスタのコンテナIDで置き換えてください。
sudo docker stop <container_id>
```

コンテナをもう必要としない場合は、次のコマンドを実行して削除することができます。

```Bash
# <container_id>をStarRocksクラスタのコンテナIDで置き換えてください。
sudo docker rm <container_id>
```

> **注意**
>
> コンテナの削除は取り消すことができません。削除する前に、コンテナ内の重要なデータのバックアップを取得してください。

## 次に何をするか

StarRocksをデプロイした後、[テーブルの作成](../quick_start/Create_table.md)や[データのロードとクエリ](../quick_start/Import_and_query.md)のクイックスタートチュートリアルを続けることができます。

<img referrerpolicy="no-referrer-when-downgrade" src="https://static.scarf.sh/a.png?x-pxid=f5ae0b2c-3578-4a40-9056-178e9837cfe0" />
