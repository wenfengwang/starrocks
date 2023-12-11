---
displayed_sidebar: "Japanese"
---

# デプロイ前提条件

本トピックでは、StarRocksを展開する前にサーバーが満たす必要があるハードウェアおよびソフトウェア要件について説明します。StarRocksクラスターの推奨ハードウェア仕様については、[StarRocksクラスターの計画](../deployment/plan_cluster.md)を参照してください。

## ハードウェア

### CPU

StarRocksはAVX2命令セットを完全に活用するために依存しています。そのため、本番環境ではx86アーキテクチャのCPUを搭載したマシンにStarRocksを展開することを強く推奨します。

以下のコマンドをターミナルで実行して、マシンのCPUがAVX2命令セットをサポートしているかどうかを確認できます。

```Bash
cat /proc/cpuinfo | grep avx2
```

> **注意**
>
> ARMアーキテクチャはSIMD命令セットをサポートしていないため、一部のシナリオではx86アーキテクチャより競争力が劣ります。そのため、開発環境ではARMアーキテクチャにStarRocksを展開することを推奨しません。

### メモリ

StarRocksに使用されるメモリキットに特定の要件はありません。推奨されるメモリサイズについては、[StarRocksクラスターの計画 - CPUおよびメモリ](../deployment/plan_cluster.md#cpu-and-memory)を参照してください。

### ストレージ

StarRocksはストレージとしてHDDおよびSSDの両方をサポートしています。

アプリケーションがリアルタイムデータ分析、集中的なデータスキャン、またはランダムなディスクアクセスを必要とする場合は、SSDストレージの使用を強く推奨します。

アプリケーションが永続的なインデックスを持つ[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)を必要とする場合、SSDストレージを使用する必要があります。

### ネットワーク

StarRocksクラスター内のノード間で安定したデータ転送を保証するために、10ギガビットイーサネットネットワークの使用を推奨します。

## オペレーティングシステム

StarRocksはCentOS Linux 7.9またはUbuntu Linux 22.04での展開をサポートしています。

## ソフトウェア

StarRocksを実行するためにサーバーにJDK 8をインストールする必要があります。v2.5以降のバージョンでは、JDK 11が推奨されています。

> **注意**
>
> - StarRocksはJREをサポートしていません。
> - Ubuntu 22.04にStarRocksをインストールする場合は、JDK 11をインストールする必要があります。

JDK 8をインストールする手順は次のとおりです。

1. JDKインストールのパスに移動します。
2. 次のコマンドを実行してJDKをダウンロードします：

   ```Bash
   wget --no-check-certificate --no-cookies \
       --header "Cookie: oraclelicense=accept-securebackup-cookie"  \
       http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
   ```