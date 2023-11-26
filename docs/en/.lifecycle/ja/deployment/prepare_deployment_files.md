---
displayed_sidebar: "Japanese"
---

# デプロイメントファイルの準備

このトピックでは、StarRocksのデプロイメントファイルの準備方法について説明します。

現在、StarRocksが[StarRocks公式ウェブサイト](https://www.starrocks.io/download/community)で提供しているバイナリディストリビューションパッケージは、x86ベースのCentOS 7.9でのデプロイメントのみをサポートしています。ARMアーキテクチャのCPUやUbuntu 22.04でStarRocksをデプロイする場合は、StarRocksのDockerイメージを使用してデプロイメントファイルを準備する必要があります。

## x86ベースのCentOS 7.9の場合

StarRocksのバイナリディストリビューションパッケージは、**StarRocks-version.tar.gz**という形式で命名されています。ここで、**version**はバイナリディストリビューションパッケージのバージョン情報を示す番号（例：**2.5.2**）です。正しいバージョンのパッケージを選択していることを確認してください。

以下の手順に従って、x86ベースのCentOS 7.9プラットフォーム用のデプロイメントファイルを準備します。

1. [Download StarRocks](https://www.starrocks.io/download/community)ページから直接StarRocksのバイナリディストリビューションパッケージを取得するか、ターミナルで次のコマンドを実行してください。

   ```Bash
   # <version>をダウンロードしたいStarRocksのバージョンに置き換えてください。例：2.5.2。
   wget https://releases.starrocks.io/starrocks/StarRocks-<version>.tar.gz
   ```

2. パッケージ内のファイルを展開してください。

   ```Bash
   # <version>をダウンロードしたStarRocksのバージョンに置き換えてください。
   tar -xzvf StarRocks-<version>.tar.gz
   ```

   パッケージには以下のディレクトリとファイルが含まれています。

   | **ディレクトリ/ファイル** | **説明**                                                     |
   | ------------------------ | ------------------------------------------------------------ |
   | **apache_hdfs_broker**   | Brokerノードのデプロイディレクトリです。StarRocks v2.5以降、一般的なシナリオではBrokerノードをデプロイする必要はありません。StarRocksクラスターでBrokerノードをデプロイする必要がある場合は、詳細な手順については[Brokerノードのデプロイ](../deployment/deploy_broker.md)を参照してください。 |
   | **fe**                   | FEのデプロイディレクトリです。                               |
   | **be**                   | BEのデプロイディレクトリです。                               |
   | **LICENSE.txt**          | StarRocksのライセンスファイルです。                          |
   | **NOTICE.txt**           | StarRocksの通知ファイルです。                                |

3. [手動デプロイ](../deployment/deploy_manually.md)のために、ディレクトリ**fe**をすべてのFEインスタンスに、ディレクトリ**be**をすべてのBEまたはCNインスタンスにディスパッチしてください。

## ARMベースのCPUまたはUbuntu 22.04の場合

### 前提条件

マシンに[Docker Engine](https://docs.docker.com/engine/install/)（バージョン17.06.0以降）がインストールされている必要があります。

### 手順

1. [StarRocks Docker Hub](https://hub.docker.com/r/starrocks/artifacts-ubuntu/tags)からStarRocksのDockerイメージをダウンロードしてください。イメージのタグに基づいて特定のバージョンを選択することができます。

   - Ubuntu 22.04を使用する場合：

     ```Bash
     # <image_tag>をダウンロードしたいイメージのタグに置き換えてください。例：2.5.4。
     docker pull starrocks/artifacts-ubuntu:<image_tag>
     ```

   - ARMベースのCentOS 7.9を使用する場合：

     ```Bash
     # <image_tag>をダウンロードしたいイメージのタグに置き換えてください。例：2.5.4。
     docker pull starrocks/artifacts-centos7:<image_tag>
     ```

2. 次のコマンドを実行して、StarRocksのデプロイメントファイルをDockerイメージからホストマシンにコピーしてください。

   - Ubuntu 22.04を使用する場合：

     ```Bash
     # <image_tag>をダウンロードしたイメージのタグに置き換えてください。例：2.5.4。
     docker run --rm starrocks/artifacts-ubuntu:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   - ARMベースのCentOS 7.9を使用する場合：

     ```Bash
     # <image_tag>をダウンロードしたイメージのタグに置き換えてください。例：2.5.4。
     docker run --rm starrocks/artifacts-centos7:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   デプロイメントファイルには以下のディレクトリが含まれています。

   | **ディレクトリ**      | **説明**                                                     |
   | -------------------- | ------------------------------------------------------------ |
   | **be_artifacts**     | このディレクトリにはBEまたはCNのデプロイディレクトリ**be**、StarRocksのライセンスファイル**LICENSE.txt**、StarRocksの通知ファイル**NOTICE.txt**が含まれています。 |
   | **broker_artifacts** | このディレクトリにはBrokerのデプロイディレクトリ**apache_hdfs_broker**が含まれています。StarRocks 2.5以降、一般的なシナリオではBrokerノードをデプロイする必要はありません。StarRocksクラスターでBrokerノードをデプロイする必要がある場合は、詳細な手順については[Brokerのデプロイ](../deployment/deploy_broker.md)を参照してください。 |
   | **fe_artifacts**     | このディレクトリにはFEのデプロイディレクトリ**fe**、StarRocksのライセンスファイル**LICENSE.txt**、StarRocksの通知ファイル**NOTICE.txt**が含まれています。 |

3. [手動デプロイ](../deployment/deploy_manually.md)のために、ディレクトリ**fe**をすべてのFEインスタンスに、ディレクトリ**be**をすべてのBEまたはCNインスタンスにディスパッチしてください。
