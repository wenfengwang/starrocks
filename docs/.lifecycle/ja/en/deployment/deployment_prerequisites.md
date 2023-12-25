---
displayed_sidebar: English
---

# デプロイ前提条件

このトピックでは、StarRocksをデプロイする前にサーバーが満たすべきハードウェアとソフトウェアの要件について説明します。StarRocksクラスターの推奨ハードウェア仕様については、[StarRocksクラスターの計画](../deployment/plan_cluster.md)を参照してください。

## ハードウェア

### CPU

StarRocksはAVX2命令セットを使用してベクトル化機能を完全に発揮します。そのため、本番環境ではx86アーキテクチャCPUを搭載したマシンでStarRocksをデプロイすることを強く推奨します。

次のコマンドをターミナルで実行し、マシンのCPUがAVX2命令セットをサポートしているかどうかを確認できます：

```Bash
cat /proc/cpuinfo | grep avx2
```

> **注記**
>
> ARMアーキテクチャはSIMD命令セットをサポートしていないため、一部のシナリオではx86アーキテクチャに比べて競争力が劣ります。そのため、開発環境でのみARMアーキテクチャにStarRocksをデプロイすることを推奨します。

### メモリ

StarRocksで使用されるメモリキットに特別な要件はありません。推奨されるメモリサイズについては、[StarRocksクラスターの計画 - CPUとメモリ](../deployment/plan_cluster.md#cpu-and-memory)を参照してください。

### ストレージ

StarRocksは、HDDとSSDの両方をストレージ媒体としてサポートしています。

アプリケーションがリアルタイムデータ分析、集中データスキャン、またはランダムディスクアクセスを必要とする場合は、SSDストレージの使用を強く推奨します。

永続インデックスを持つ[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)を使用するアプリケーションでは、SSDストレージを使用する必要があります。

### ネットワーク

StarRocksクラスタ内のノード間で安定したデータ転送を確保するために、10ギガビットイーサネットネットワーキングの使用を推奨します。

## オペレーティングシステム

StarRocksは、CentOS Linux 7.9またはUbuntu Linux 22.04上でのデプロイをサポートしています。

## ソフトウェア

サーバーにはJDK 8をインストールしてStarRocksを実行する必要があります。バージョン2.5以降では、JDK 11の使用を推奨します。

> **警告**
>
> - StarRocksはJREをサポートしません。
> - Ubuntu 22.04にStarRocksをインストールする場合は、JDK 11をインストールする必要があります。

JDK 8をインストールするには、以下の手順に従ってください：

1. JDKのインストールパスに移動します。
2. 次のコマンドを実行してJDKをダウンロードします：

   ```Bash
   wget --no-check-certificate --no-cookies \
       --header "Cookie: oraclelicense=accept-securebackup-cookie"  \
       http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
   ```
