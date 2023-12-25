---
displayed_sidebar: Chinese
sidebar_position: 1
---

# Docker を使用した StarRocks のデプロイ

このクイックスタートチュートリアルでは、Docker を使用してローカルコンピュータに StarRocks をデプロイする方法を案内します。開始する前に、[システムアーキテクチャ](../introduction/Architecture.md)を読んで、より多くの概念の詳細を理解してください。

以下の手順に従って、**FE ノード1つ**と**BE ノード1つ**を含むシンプルな StarRocks クラスターをデプロイできます。このクラスターをベースに、[テーブルの作成](../quick_start/Create_table.md)と[データのインポートとクエリ](../quick_start/Import_and_query.md)のクイックスタートチュートリアルを完了し、StarRocks の基本操作に慣れることができます。

> **注意**
>
> 以下の Docker イメージを使用してデプロイされた StarRocks クラスターは、小規模なデータセットの検証用デモにのみ適しています。大規模なテストや本番環境での使用は推奨されません。高可用性の StarRocks クラスターをデプロイするには、[デプロイ概要](../deployment/deployment_overview.md)を参照して、シナリオに合った他のデプロイ方法を確認してください。

## 前提条件

Docker コンテナ内で StarRocks をデプロイする前に、以下の環境要件が満たされていることを確認してください：

- **ハードウェア要件**

  8コア CPU と 16 GB 以上のメモリを搭載したコンピュータで StarRocks をデプロイすることを推奨します。

- **ソフトウェア要件**

  コンピュータに以下のソフトウェアをインストールする必要があります：

  - [Docker Engine](https://docs.docker.com/engine/install/) (バージョン 17.06.0 以上)
  - MySQL クライアント (バージョン 5.5 以上)

## ステップ1: Docker イメージを使用して StarRocks をデプロイする

[StarRocks Docker Hub](https://hub.docker.com/r/starrocks/allin1-ubuntu/tags)から StarRocks Docker イメージをダウンロードします。Tag を選択して特定のバージョンのイメージを選ぶことができます。

```Bash
sudo docker run -p 9030:9030 -p 8030:8030 -p 8040:8040 \
    -itd starrocks.docker.scarf.sh/starrocks/allin1-ubuntu
```

> **トラブルシューティング**
>
> 上記のいずれかのポートが使用中の場合、システムはエラーログ "docker: Error response from daemon: driver failed programming external connectivity on endpoint tender_torvalds (): Bind for 0.0.0.0:xxxx failed: port is already allocated" を出力します。コマンド中のコロン（:）の前のポートを変更して、ホスト上の他の利用可能なポートに変更できます。

以下のコマンドを実行して、コンテナが作成されて正常に実行されているかを確認できます：

```Bash
sudo docker ps
```

以下のように、StarRocks コンテナの `STATUS` が `Up` であれば、Docker コンテナ内に StarRocks を正常にデプロイできています。

```Plain
CONTAINER ID   IMAGE                                          COMMAND                  CREATED         STATUS                 PORTS                                                                                                                             NAMES
8962368f9208   starrocks/allin1-ubuntu:branch-3.0-0afb97bbf   "/bin/sh -c ./start_…"   4 minutes ago   Up 4 minutes           0.0.0.0:8037->8030/tcp, :::8037->8030/tcp, 0.0.0.0:8047->8040/tcp, :::8047->8040/tcp, 0.0.0.0:9037->9030/tcp, :::9037->9030/tcp   xxxxx
```

## ステップ2: StarRocks に接続する

デプロイに成功したら、MySQL クライアントを使用して StarRocks クラスターに接続できます。

```Bash
mysql -P9030 -h127.0.0.1 -uroot --prompt="StarRocks > "
```

> **注意**
>
> `docker run` コマンドで `9030` ポートに別のポートを割り当てた場合は、上記のコマンドで `9030` を割り当てたポートに置き換える必要があります。

以下の SQL を実行して FE ノードの状態を確認できます：

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

- `Alive` フィールドが `true` であれば、その FE ノードは正常に起動し、クラスターに参加しています。
- `Role` フィールドが `FOLLOWER` であれば、その FE ノードは Leader FE ノードに選出される資格があります。
- `Role` フィールドが `LEADER` であれば、その FE ノードは Leader FE ノードです。

以下の SQL を実行して BE ノードの状態を確認できます：

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

`Alive` フィールドが `true` であれば、その BE ノードは正常に起動し、クラスターに参加しています。

## Docker コンテナの停止と削除

クイックスタートチュートリアルを完了した後、StarRocks コンテナの ID を使用してコンテナを停止して削除できます。

> **説明**
>
> `sudo docker ps` を実行して Docker コンテナの `container_id` を取得できます。

以下のコマンドを実行してコンテナを停止します：

```Bash
# <container_id> を StarRocks クラスターのコンテナ ID に置き換えてください。
sudo docker stop <container_id>
```

コンテナが不要になった場合は、以下のコマンドを実行して削除できます：

```Bash
# <container_id> を StarRocks クラスターのコンテナ ID に置き換えてください。
sudo docker rm <container_id>
```

> **注意**
>
> コンテナを削除する操作は取り消しできません。削除する前に、コンテナ内の重要なデータをバックアップしておくことを確認してください。

## 次のステップ

StarRocks を成功裏にデプロイした後、[テーブルの作成](../quick_start/Create_table.md)と[データのインポートとクエリ](../quick_start/Import_and_query.md)に関するクイックスタートチュートリアルを続けることができます。
