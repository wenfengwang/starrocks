---
displayed_sidebar: "Japanese"
---

# デプロイメントファイルの準備

このトピックでは、StarRocksのデプロイメントファイルの準備方法について説明します。

現在、StarRocksの公式ウェブサイト（https://www.starrocks.io/download/community）で提供されているバイナリ配布パッケージは、x86ベースのCentOS 7.9でのデプロイメントのみをサポートしています。ARMアーキテクチャのCPUを使用するか、Ubuntu 22.04でStarRocksをデプロイする場合は、StarRocksのDockerイメージを使用してデプロイメントファイルを準備する必要があります。

## x86ベースのCentOS 7.9の場合

StarRocksのバイナリ配布パッケージは、**StarRocks-version.tar.gz**形式で命名されており、ここでの**version**はバイナリ配布パッケージのバージョン情報（例：**2.5.2**）を示しています。選択したパッケージの正しいバージョンを選択したことを確認してください。

次の手順に従って、x86ベースのCentOS 7.9プラットフォーム用のデプロイメントファイルを準備してください：

1. [Download StarRocks](https://www.starrocks.io/download/community)ページからStarRocksのバイナリ配布パッケージを直接取得するか、ターミナルで次のコマンドを実行してください：

   ```Bash
   # バージョンは例えば2.5.2のようなStarRocksのバージョンに置き換えてください。
   wget https://releases.starrocks.io/starrocks/StarRocks-<version>.tar.gz
   ```

2. パッケージ内のファイルを展開してください。

   ```Bash
   # バージョンはダウンロードしたStarRocksのバージョンに置き換えてください。
   tar -xzvf StarRocks-<version>.tar.gz
   ```

   パッケージには以下のディレクトリとファイルが含まれています：

   | **ディレクトリ/ファイル** | **説明**                                                     |
   | ------------------------ | ------------------------------------------------------------ |
   | **apache_hdfs_broker**   | Brokerノードのデプロイディレクトリ。StarRocks v2.5以降、一般的なシナリオではBrokerノードをデプロイする必要はありません。StarRocksクラスターでBrokerノードをデプロイする必要がある場合は、詳しい手順については[Brokerノードのデプロイ](../deployment/deploy_broker.md)を参照してください。 |
   | **fe**                   | FEのデプロイディレクトリ。                                   |
   | **be**                   | BEのデプロイディレクトリ。                                   |
   | **LICENSE.txt**          | StarRocksのライセンスファイル。                              |
   | **NOTICE.txt**           | StarRocksの通知ファイル。                                   |

3. [手動でのデプロイ](../deployment/deploy_manually.md)のために、ディレクトリ**fe**をすべてのFEインスタンスに、ディレクトリ**be**をすべてのBEまたはCNインスタンスに配布してください。

## ARMベースのCPUまたはUbuntu 22.04の場合

### 前提条件

マシンに[Docker Engine](https://docs.docker.com/engine/install/)（17.06.0以降）がインストールされている必要があります。

### 手順

1. [StarRocks Docker Hub](https://hub.docker.com/r/starrocks/artifacts-ubuntu/tags)からStarRocksのDockerイメージをダウンロードしてください。イメージのタグに基づいて特定のバージョンを選択することができます。

   - Ubuntu 22.04を使用する場合：

     ```Bash
     # <image_tag>はダウンロードしたいイメージのタグに置き換えてください。例：2.5.4。
     docker pull starrocks/artifacts-ubuntu:<image_tag>
     ```

   - ARMベースのCentOS 7.9を使用する場合：

     ```Bash
     # <image_tag>はダウンロードしたいイメージのタグに置き換えてください。例：2.5.4。
     docker pull starrocks/artifacts-centos7:<image_tag>
     ```

2. 次のコマンドを実行して、DockerイメージからStarRocksのデプロイメントファイルをホストマシンにコピーしてください：

   - Ubuntu 22.04を使用する場合：

     ```Bash
     # <image_tag>はダウンロードしたイメージのタグに置き換えてください。例：2.5.4。
     docker run --rm starrocks/artifacts-ubuntu:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   - ARMベースのCentOS 7.9を使用する場合：

     ```Bash
     # <image_tag>はダウンロードしたイメージのタグに置き換えてください。例：2.5.4。
     docker run --rm starrocks/artifacts-centos7:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   デプロイメントファイルには以下のディレクトリが含まれています：

   | **ディレクトリ**     | **説明**                                                     |
   | ------------------ | ------------------------------------------------------------ |
   | **be_artifacts**   | このディレクトリにはBEまたはCNのデプロイディレクトリ**be**、StarRocksのライセンスファイル**LICENSE.txt**、およびStarRocksの通知ファイル**NOTICE.txt**が含まれています。 |
   | **broker_artifacts** | このディレクトリにはBrokerのデプロイディレクトリ**apache_hdfs_broker**が含まれています。StarRocks 2.5以降、一般的なシナリオではBrokerノードをデプロイする必要はありません。StarRocksクラスターでBrokerノードをデプロイする必要がある場合は、詳しい手順については[Brokerのデプロイ](../deployment/deploy_broker.md)を参照してください。 |
   | **fe_artifacts** | このディレクトリにはFEのデプロイディレクトリ**fe**、StarRocksのライセンスファイル**LICENSE.txt**、およびStarRocksの通知ファイル**NOTICE.txt**が含まれています。 |

3. [手動でのデプロイ](../deployment/deploy_manually.md)のために、ディレクトリ**fe**をすべてのFEインスタンスに、ディレクトリ**be**をすべてのBEまたはCNインスタンスに配布してください。