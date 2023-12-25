---
displayed_sidebar: Chinese
---

# 部署ファイルの準備

この文書では、StarRocksの部署ファイルの準備方法について説明します。

現在、[ミラーシップ公式ウェブサイト](https://www.mirrorship.cn/zh-CN/download/community)で提供されているStarRocksのソフトウェアパッケージは、x86アーキテクチャのCPUを搭載したCentOS 7.9プラットフォームでのみ部署をサポートしています。ARMアーキテクチャのCPUやUbuntu 22.04オペレーティングシステムでStarRocksを部署するには、StarRocks Dockerイメージを通じて部署ファイルを取得する必要があります。

## x86アーキテクチャのCentOS 7.9プラットフォーム用の部署ファイルの準備

StarRocksのバイナリパッケージの名前の形式は **StarRocks-version.tar.gz** で、**version** は数字（例：**2.5.2**）で、バイナリパッケージのバージョン情報を示しています。正しいバージョンのバイナリパッケージを選択していることを確認してください。

### 手順

1. [StarRocksのダウンロードページ](https://www.starrocks.io/download/community)から直接StarRocksのバイナリパッケージをダウンロードするか、または以下のコマンドをターミナルで実行して取得します：

   ```Bash
   # <version>をダウンロードしたいStarRocksのバージョンに置き換えてください。例えば2.5.4。
   wget https://releases.starrocks.io/starrocks/StarRocks-<version>.tar.gz
   ```

2. バイナリパッケージを解凍します。

   ```Bash
   # <version>をダウンロードしたStarRocksのバージョンに置き換えてください。例えば2.5.4。
   tar -xzvf StarRocks-<version>.tar.gz
   ```

   バイナリパッケージには以下のパスとファイルが含まれています：

   | **パス/ファイル**          | **説明**                                                     |
   | ---------------------- | ------------------------------------------------------------ |
   | **apache_hdfs_broker** | Brokerノードの部署パス。StarRocks 2.5以降、通常のシナリオでBrokerノードを部署する必要はありません。StarRocksクラスターにBrokerノードを部署する必要がある場合は、[Brokerノードの部署](../deployment/deploy_broker.md)を参照して詳細を確認してください。 |
   | **fe**                 | FEノードの部署パス。                                          |
   | **be**                 | BEノードの部署パス。                                          |
   | **LICENSE.txt**        | StarRocksのライセンスファイル。                                     |
   | **NOTICE.txt**         | StarRocksの通知ファイル。                                      |

3. **fe** パスをすべてのFEインスタンスに配布し、**be** パスをすべてのBEまたはCNインスタンスに配布して[手動での部署](../deployment/deploy_manually.md)に使用します。

## ARMアーキテクチャのCPUまたはUbuntu 22.04プラットフォーム用の部署ファイルの準備

### 前提条件

コンピュータに [Docker Engine](https://docs.docker.com/engine/install/)（17.06.0以上）をインストールする必要があります。

### 手順

1. [StarRocks Docker Hub](https://hub.docker.com/r/starrocks/artifacts-ubuntu/tags)からStarRocks Dockerイメージをダウンロードします。Tagに基づいて特定のバージョンのイメージを選択できます。

   - Ubuntu 22.04プラットフォームを使用する場合：

     ```Bash
     # <image_tag>をダウンロードしたいイメージのTagに置き換えてください。例えば2.5.4。
     docker pull starrocks/artifacts-ubuntu:<image_tag>
     ```

   - ARMアーキテクチャのCentOS 7.9プラットフォームを使用する場合：

     ```Bash
     # <image_tag>をダウンロードしたいイメージのTagに置き換えてください。例えば2.5.4。
     docker pull starrocks/artifacts-centos7:<image_tag>
     ```

2. 以下のコマンドを実行して、StarRocksの部署ファイルをDockerイメージからホストにコピーします：

   - Ubuntu 22.04プラットフォームを使用する場合：

     ```Bash
     # <image_tag>をダウンロードしたイメージのTagに置き換えてください。例えば2.5.4。
     docker run --rm starrocks/artifacts-ubuntu:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   - ARMアーキテクチャのCentOS 7.9プラットフォームを使用する場合：

     ```Bash
     # <image_tag>をダウンロードしたイメージのTagに置き換えてください。例えば2.5.4。
     docker run --rm starrocks/artifacts-centos7:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   部署ファイルには以下のパスが含まれています：

   | **パス**             | **説明**                                                     |
   | -------------------- | ------------------------------------------------------------ |
   | **be_artifacts**     | このパスにはBEまたはCNノードの部署パス **be**、StarRocksのライセンスファイル **LICENSE.txt**、およびStarRocksの通知ファイル **NOTICE.txt** が含まれています。 |
   | **broker_artifacts** | このパスにはBrokerノードの部署パス **apache_hdfs_broker** が含まれています。 |
   | **fe_artifacts**     | このパスにはFEノードの部署パス **fe**、StarRocksのライセンスファイル **LICENSE.txt**、およびStarRocksの通知ファイル **NOTICE.txt** が含まれています。 |

3. **fe** パスをすべてのFEインスタンスに配布し、**be** パスをすべてのBEまたはCNインスタンスに配布して[手動での部署](../deployment/deploy_manually.md)に使用します。
