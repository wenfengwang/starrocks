---
displayed_sidebar: "Japanese"
---

# デプロイの前提条件

このトピックでは、StarRocksをデプロイする前にサーバーが満たす必要があるハードウェアおよびソフトウェアの要件について説明します。StarRocksクラスターの推奨ハードウェア仕様については、[StarRocksクラスターの計画](../deployment/plan_cluster.md)を参照してください。

## ハードウェア

### CPU

StarRocksは、ベクトル化能力を十分に発揮するためにAVX2命令セットに依存しています。そのため、本番環境では、x86アーキテクチャCPUを搭載したマシンにStarRocksをデプロイすることを強くお勧めします。

次のコマンドをターミナルで実行して、マシンのCPUがAVX2命令セットをサポートしているかどうかを確認できます。

```Bash
cat /proc/cpuinfo | grep avx2
```

> **注意**
>
> ARMアーキテクチャはSIMD命令セットをサポートしていないため、一部のシナリオではx86アーキテクチャよりも競争力が低下します。そのため、開発環境ではARMアーキテクチャにStarRocksをデプロイすることをお勧めしません。

### メモリ

StarRocksに使用するメモリキットには特定の要件はありません。推奨されるメモリサイズについては、[StarRocksクラスターの計画 - CPUとメモリ](../deployment/plan_cluster.md#cpu-and-memory)を参照してください。

### ストレージ

StarRocksは、HDDおよびSSDの両方をストレージ媒体としてサポートしています。

アプリケーションがリアルタイムのデータ分析、集中的なデータスキャン、またはランダムなディスクアクセスを必要とする場合は、SSDストレージの使用を強くお勧めします。

[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)に永続インデックスを持つアプリケーションの場合は、SSDストレージを使用する必要があります。

### ネットワーク

StarRocksクラスター内のノード間で安定したデータ転送を確保するために、10ギガビットイーサネットネットワークの使用をお勧めします。

## オペレーティングシステム

StarRocksは、CentOS Linux 7.9またはUbuntu Linux 22.04でのデプロイをサポートしています。

## ソフトウェア

StarRocksを実行するために、サーバーにJDK 8をインストールする必要があります。v2.5以降のバージョンでは、JDK 11が推奨されています。

> **注意**
>
> - StarRocksはJREをサポートしていません。
> - Ubuntu 22.04にStarRocksをインストールする場合は、JDK 11をインストールする必要があります。

JDK 8をインストールするには、次の手順に従ってください。

1. JDKのインストールのためのパスに移動します。
2. 次のコマンドを実行してJDKをダウンロードします。

   ```Bash
   wget --no-check-certificate --no-cookies \
       --header "Cookie: oraclelicense=accept-securebackup-cookie"  \
       http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
   ```
