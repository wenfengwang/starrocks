---
displayed_sidebar: "Japanese"
---

# デプロイメントファイルの準備

このトピックでは、StarRocksのデプロイメントファイルの準備方法について説明します。

現在、[StarRocksの公式ウェブサイト](https://www.starrocks.io/download/community)で提供されているバイナリディストリビューションパッケージは、x86ベースのCentOS 7.9でのみデプロイメントをサポートしています。ARMアーキテクチャCPUで、またはUbuntu 22.04でStarRocksをデプロイしたい場合は、StarRocksのDockerイメージを使用してデプロイメントファイルを準備する必要があります。

## x86ベースのCentOS 7.9の場合

StarRocksのバイナリディストリビューションパッケージは、**StarRocks-version.tar.gz**の形式で名前が付けられており、ここで**version**はバイナリディストリビューションパッケージのバージョン情報を示す番号（例: **2.5.2**）です。選択したパッケージの正しいバージョンを選択していることを確認してください。

次の手順に従って、x86ベースのCentOS 7.9プラットフォーム向けのデプロイメントファイルを準備してください:

1. StarRocksのバイナリディストリビューションパッケージを[StarRocksをダウンロード](https://www.starrocks.io/download/community)ページから直接入手するか、ターミナルで次のコマンドを実行してください:

   ```Bash
   # <version>にはダウンロードしたいStarRocksのバージョンを入力してください。例: 2.5.2
   wget https://releases.starrocks.io/starrocks/StarRocks-<version>.tar.gz
   ```

2. パッケージ内のファイルを展開します。

   ```Bash
   # <version>にはダウンロードしたStarRocksのバージョンを入力してください。
   tar -xzvf StarRocks-<version>.tar.gz
   ```

   パッケージには次のディレクトリとファイルが含まれています:

   | **ディレクトリ/ファイル** | **概要**                                                  |
   | ---------------------- | ------------------------------------------------------------ |
   | **apache_hdfs_broker** | Brokerノードのデプロイメントディレクトリです。StarRocks v2.5以降、一般的なシナリオではBrokerノードのデプロイは必要ありませんが、StarRocksクラスターでBrokerノードをデプロイする必要がある場合は、詳細な手順については[Brokerノードのデプロイ](../deployment/deploy_broker.md)を参照してください。 |
   | **fe**                 | FEのデプロイメントディレクトリです。                                 |
   | **be**                 | BEのデプロイメントディレクトリです。                                 |
   | **LICENSE.txt**        | StarRocksのライセンスファイルです。                                  |
   | **NOTICE.txt**         | StarRocksの通知ファイルです。                                   |

3. [手動デプロイ](../deployment/deploy_manually.md)のために、ディレクトリ**fe**をすべてのFEインスタンスに、ディレクトリ**be**をすべてのBEまたはCNインスタンスにディスパッチしてください。

## ARMベースのCPUまたはUbuntu 22.04の場合

### 事前準備

マシンに[Docker Engine](https://docs.docker.com/engine/install/)（17.06.0以降）がインストールされている必要があります。

### 手順

1. [StarRocks Docker Hub](https://hub.docker.com/r/starrocks/artifacts-ubuntu/tags)からStarRocksのDockerイメージをダウンロードしてください。イメージのタグに基づいて特定のバージョンを選択できます。

   - Ubuntu 22.04を使用する場合:

     ```Bash
     # <image_tag>にはダウンロードしたいイメージのタグを入力してください。例: 2.5.4
     docker pull starrocks/artifacts-ubuntu:<image_tag>
     ```

   - ARMベースのCentOS 7.9を使用する場合:

     ```Bash
     # <image_tag>にはダウンロードしたいイメージのタグを入力してください。例: 2.5.4
     docker pull starrocks/artifacts-centos7:<image_tag>
     ```

2. 次のコマンドを実行して、DockerイメージからStarRocksのデプロイメントファイルをホストマシンにコピーしてください:

   - Ubuntu 22.04を使用する場合:

     ```Bash
     # <image_tag>にはダウンロードしたイメージのタグを入力してください。例: 2.5.4
     docker run --rm starrocks/artifacts-ubuntu:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   - ARMベースのCentOS 7.9を使用する場合:

     ```Bash
     # <image_tag>にはダウンロードしたイメージのタグを入力してください。例: 2.5.4
     docker run --rm starrocks/artifacts-centos7:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   デプロイメントファイルには次のディレクトリが含まれています:

   | **ディレクトリ**        | **概要**                                                  |
   | -------------------- | ------------------------------------------------------------ |
   | **be_artifacts**     | このディレクトリには、BEまたはCNのデプロイメントディレクトリ**be**、StarRocksのライセンスファイル **LICENSE.txt**、およびStarRocksの通知ファイル **NOTICE.txt**が含まれています。 |
   | **broker_artifacts** | このディレクトリには、Brokerのデプロイメントディレクトリ **apache_hdfs_broker**が含まれています。StarRocks 2.5以降、一般的なシナリオではBrokerノードのデプロイは必要ありませんが、StarRocksクラスターでBrokerノードをデプロイする必要がある場合は、詳細な手順については[Brokerのデプロイ](../deployment/deploy_broker.md)を参照してください。 |
   | **fe_artifacts**     | このディレクトリには、FEのデプロイメントディレクトリ **fe**、StarRocksのライセンスファイル **LICENSE.txt**、およびStarRocksの通知ファイル **NOTICE.txt**が含まれています。 |

3. [手動デプロイ](../deployment/deploy_manually.md)のために、ディレクトリ**fe**をすべてのFEインスタンスに、ディレクトリ**be**をすべてのBEまたはCNインスタンスにディスパッチしてください。