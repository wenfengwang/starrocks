---
displayed_sidebar: "Japanese"
---

# デプロイメント前提条件

このトピックでは、StarRocksを展開する前にサーバーが満たす必要があるハードウェアおよびソフトウェア要件について説明します。StarRocksクラスターの推奨ハードウェア仕様については、[StarRocksクラスターの計画](../deployment/plan_cluster.md)を参照してください。

## ハードウェア

### CPU

StarRocksは、ベクトル化機能を完全に活用するためにAVX2命令セットに依存しています。したがって、本番環境ではx86アーキテクチャのCPUを搭載したマシンにStarRocksを展開することを強くお勧めします。

マシン上のCPUがAVX2命令セットをサポートしているかどうかを確認するには、ターミナルで次のコマンドを実行できます:

```Bash
cat /proc/cpuinfo | grep avx2
```

> **注記**
>
> ARMアーキテクチャはSIMD命令セットをサポートしていないため、一部のシナリオではx86アーキテクチャよりも競争力が低くなります。そのため、開発環境でのみARMアーキテクチャにStarRocksを展開することをお勧めします。

### メモリ

StarRocksに使用されるメモリキットには特定の要件はありません。推奨メモリサイズについては、[StarRocksクラスターの計画 - CPUとメモリ](../deployment/plan_cluster.md#cpu-and-memory)を参照してください。

### ストレージ

StarRocksはHDDおよびSSDの両方をストレージ媒体としてサポートしています。

リアルタイムデータ分析、集中的なデータスキャン、またはランダムディスクアクセスが必要な場合は、SSDストレージの使用を強くお勧めします。

永続インデックスを持つ[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)を使用する場合は、SSDストレージを使用する必要があります。

### ネットワーク

StarRocksクラスター内のノード間で安定したデータ転送を確保するために、10ギガビットイーサネットネットワークの使用を推奨します。

## オペレーティングシステム

StarRocksは、CentOS Linux 7.9またはUbuntu Linux 22.04での展開をサポートしています。

## ソフトウェア

StarRocksを実行するためには、サーバーにJDK 8をインストールする必要があります。v2.5以降のバージョンでは、JDK 11が推奨されています。

> **注意**
>
> - StarRocksはJREをサポートしていません。
> - Ubuntu 22.04にStarRocksをインストールする場合は、JDK 11をインストールする必要があります。

JDK 8をインストールする手順は以下の通りです:

1. JDKのインストールパスに移動します。
2. 次のコマンドを実行してJDKをダウンロードします:

   ```Bash
   wget --no-check-certificate --no-cookies \
       --header "Cookie: oraclelicense=accept-securebackup-cookie"  \
       http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
   ```