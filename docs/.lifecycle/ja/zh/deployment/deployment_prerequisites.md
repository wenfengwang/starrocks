---
displayed_sidebar: Chinese
---

# 部署前提条件

本文描述了部署 StarRocks 的服务器必须满足哪些软硬件要求。关于 StarRocks 集群的推荐硬件规格，请参阅[规划您的 StarRocks 集群](../deployment/plan_cluster.md)。

## 硬件

### CPU

StarRocks は AVX2 命令セットを利用してベクトル化能力を最大限に発揮します。そのため、本番環境では、StarRocks を x86 アーキテクチャの CPU を搭載したサーバーにデプロイすることを強く推奨します。

次のコマンドを端末で実行して、CPU が AVX2 命令セットをサポートしているかどうかを確認できます：

```Bash
cat /proc/cpuinfo | grep avx2
```

> **説明**
>
> ARM アーキテクチャは SIMD 命令セットをサポートしていないため、一部のシナリオでは x86 アーキテクチャよりもパフォーマンスが劣ります。開発環境でのみ ARM アーキテクチャの StarRocks のデプロイを推奨します。

### メモリ

StarRocks は特定のメモリ要件を持っていません。推奨されるメモリサイズについては、[StarRocks クラスターの計画 - CPU およびメモリ](../deployment/plan_cluster.md#cpu-とメモリ)を参照してください。

### ストレージ

StarRocks は HDD と SSD をストレージ媒体としてサポートしています。

リアルタイムデータ分析のシナリオや、大量のデータスキャンやランダムディスクアクセスが発生するシナリオでは、SSD をストレージ媒体として選択することを強く推奨します。

[プライマリキーモデル](../table_design/table_types/primary_key_table.md)の永続化インデックスに関連するシナリオでは、SSD をストレージ媒体として使用する必要があります。

### ネットワーク

StarRocks クラスター内のデータがノード間で効率的に転送されることを確保するために、10ギガビットイーサネット（10 Gigabit Ethernet、略称 10 GE）の使用を推奨します。

## オペレーティングシステム

StarRocks は CentOS Linux 7.9 および Ubuntu Linux 22.04 でのデプロイをサポートしています。

## ソフトウェア

StarRocks を実行するためには、サーバーに JDK 8 をインストールする必要があります。バージョン 2.5 以降では JDK 11 のインストールが推奨されます。

> **注意**
>
> - StarRocks は JRE をサポートしていません。
> - Ubuntu 22.04 で StarRocks をデプロイする場合は、JDK 11 のインストールが必須です。

以下の手順で JDK 8 をインストールしてください：

1. JDK をインストールするパスに移動します。
2. 次のコマンドを実行して JDK をダウンロードします：

   ```Bash
   wget --no-check-certificate --no-cookies \
       --header "Cookie: oraclelicense=accept-securebackup-cookie"  \
       http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
   ```
