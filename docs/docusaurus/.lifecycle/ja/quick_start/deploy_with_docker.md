---
displayed_sidebar: "日本語"
sidebar_position: 1
---

# Dockerを使用してStarRocksを展開する

このクイックスタートチュートリアルでは、Dockerを使用してローカルマシンにStarRocksを展開する手順について説明します。始める前に、より概念的な詳細については[StarRocks アーキテクチャ](../introduction/Architecture.md)を参照してください。

これらの手順に従うことで、**1つのFEノード**と**1つのBEノード**を持つシンプルなStarRocksクラスターを展開できます。これにより、[テーブルの作成](../quick_start/Create_table.md)および[データのロードとクエリ](../quick_start/Import_and_query.md)といった次のクイックスタートチュートリアルを完了し、StarRocksの基本操作に慣れることができます。

> **注意**
>
> このチュートリアルで使用されるDockerイメージを使用してStarRocksを展開するのは、小規模なデータセットでデモを検証する必要がある場合に適用されるものであり、大規模なテストや本番環境には推奨されません。高可用性のStarRocksクラスターを展開するには、シナリオに適した他のオプションについては[展開の概要](../deployment/deployment_overview.md)を参照してください。

## 必要条件

DockerでStarRocksを展開する前に、以下の要件を満たしていることを確認してください：

- **ハードウェア**

  StarRocksを8つのCPUコアと16GB以上のメモリを搭載したマシンに展開することをお勧めします。

- **ソフトウェア**

  マシンに以下のソフトウェアがインストールされている必要があります：

  - [Docker Engine](https://docs.docker.com/engine/install/)（17.06.0以降） かつメタディレクトリのディスクパーティションに少なくとも5GBの空き容量があること。 詳細についてはhttps://github.com/StarRocks/starrocks/issues/35608を参照してください。
  - MySQLクライアント（5.5以降）

## ステップ1: StarRocks Dockerイメージのダウンロード

[StarRocks Docker Hub](https://hub.docker.com/r/starrocks/allin1-ubuntu/tags)からStarRocks Dockerイメージをダウンロードします。イメージのタグに基づいて特定のバージョンを選択できます。

```Bash
sudo docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 \
    -itd starrocks/allin1-ubuntu
```

> **トラブルシューティング**
>
> 上記ポートのいずれかがホストマシンで使用中の場合、システムは「docker: Error response from daemon: driver failed programming external connectivity on endpoint tender_torvalds (): Bind for 0.0.0.0:xxxx failed: port is already allocated.」と表示します。コマンドのコロン（:）の前にあるポートを変更して、ホストマシン上で利用可能なポートを割り当てることができます。

次のコマンドを実行して、コンテナが適切に作成および実行されているか確認できます：

```Bash
sudo docker ps
```

以下のように表示される場合、あなたのStarRocksコンテナの`STATUS`が`Up`である場合、StarRocksがDockerコンテナに展開されたことを成功裏に完了しました。

```Plain
CONTAINER ID   IMAGE                                          COMMAND                  CREATED         STATUS                 PORTS                                                                                                                             NAMES
8962368f9208   starrocks/allin1-ubuntu:branch-3.0-0afb97bbf   "/bin/sh -c ./start_…"   4 minutes ago   Up 4 minutes           0.0.0.0:8037->8030/tcp, :::8037->8030/tcp, 0.0.0.0:8047->8040/tcp, :::8047->8040/tcp, 0.0.0.0:9037->9030/tcp, :::9037->9030/tcp   xxxxx
```

## ステップ2: StarRocksに接続する

StarRocksが適切に展開された後、MySQLクライアントを使用してそれに接続できます。

```Bash
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

> **注意**
>
> `docker run`コマンドで`9030`の代わりに別のポートを割り当てた場合、上記のコマンドの`9030`を割り当てたポートに置き換える必要があります。

次のSQLを実行して、FEノードの状態を確認できます：

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

- `Alive`フィールドが`true`であれば、このFEノードは適切に起動され、クラスターに追加されています。
- `Role`フィールドが`FOLLOWER`であれば、このFEノードはリーダーFEノードに選出される資格があります。
- `Role`フィールドが`LEADER`であれば、このFEノードはリーダーFEノードです。

BEノードの状態は次のSQLを実行して確認できます：

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

`Alive`フィールドが`true`であれば、このBEノードは適切に起動され、クラスターに追加されています。

## Dockerコンテナを停止および削除する

全体のクイックスタートチュートリアルを完了した後、StarRocksクラスターをホストするコンテナをそのコンテナIDで停止および削除できます。

> **注意**
>
> `sudo docker ps`を実行してDockerコンテナの`container_id`を取得できます。

以下のコマンドを実行してコンテナを停止します：

```Bash
# <container_id>にStarRocksクラスターのコンテナIDを置き換えてください。
sudo docker stop <container_id>
```

コンテナをもはや必要としない場合は、以下のコマンドを実行してそれを削除できます：

```Bash
# <container_id>にStarRocksクラスターのコンテナIDを置き換えてください。
sudo docker rm <container_id>
```

> **注意**
>
> コンテナの削除は取り消しできません。削除する前に重要なデータのバックアップを取得していることを確認してください。

## 次に何をするか

StarRocksを展開したら、[テーブルの作成](../quick_start/Create_table.md)および[データのロードとクエリ](../quick_start/Import_and_query.md)のクイックスタートチュートリアルを続けることができます。