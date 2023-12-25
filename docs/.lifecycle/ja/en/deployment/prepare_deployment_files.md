---
displayed_sidebar: English
---

# デプロイメントファイルの準備

このトピックでは、StarRocksのデプロイメントファイルを準備する方法について説明します。

現在、StarRocksが[公式ウェブサイト](https://www.starrocks.io/download/community)で提供しているバイナリ配布パッケージは、x86ベースのCentOS 7.9でのみデプロイをサポートしています。ARMアーキテクチャのCPUやUbuntu 22.04でStarRocksをデプロイする場合は、StarRocks Dockerイメージを使用してデプロイメントファイルを準備する必要があります。

## x86ベースのCentOS 7.9の場合

StarRocksのバイナリ配布パッケージは、**StarRocks-version.tar.gz** 形式で命名されており、**version**はバイナリ配布パッケージのバージョン情報を示す数字（例：**2.5.2**）です。パッケージの正しいバージョンを選択していることを確認してください。

x86ベースのCentOS 7.9プラットフォーム用のデプロイメントファイルを準備するには、以下の手順に従ってください：

1. [StarRocksのダウンロード](https://www.starrocks.io/download/community)ページから直接StarRocksバイナリ配布パッケージを取得するか、ターミナルで以下のコマンドを実行します：

   ```Bash
   # <version>をダウンロードしたいStarRocksのバージョンに置き換えてください。例えば、2.5.2。
   wget https://releases.starrocks.io/starrocks/StarRocks-<version>.tar.gz
   ```

2. パッケージ内のファイルを展開します。

   ```Bash
   # <version>をダウンロードしたStarRocksのバージョンに置き換えてください。
   tar -xzvf StarRocks-<version>.tar.gz
   ```

   パッケージには以下のディレクトリとファイルが含まれています：

   | **ディレクトリ/ファイル** | **説明**                                              |
   | ---------------------- | ------------------------------------------------------------ |
   | **apache_hdfs_broker** | Brokerノードのデプロイメントディレクトリ。StarRocks v2.5以降、一般的なシナリオではBrokerノードをデプロイする必要はありません。StarRocksクラスターにBrokerノードをデプロイする必要がある場合は、[Brokerノードのデプロイ](../deployment/deploy_broker.md)を参照してください。 |
   | **fe**                 | FEのデプロイメントディレクトリ。                                 |
   | **be**                 | BEのデプロイメントディレクトリ。                                 |
   | **LICENSE.txt**        | StarRocksのライセンスファイル。                                  |
   | **NOTICE.txt**         | StarRocksの通知ファイル。                                   |

3. **fe**ディレクトリをすべてのFEインスタンスに、**be**ディレクトリをすべてのBEまたはCNインスタンスに配布し、[手動でデプロイ](../deployment/deploy_manually.md)します。

## ARMベースのCPUまたはUbuntu 22.04の場合

### 前提条件

マシンには[Docker Engine](https://docs.docker.com/engine/install/)（17.06.0以降）がインストールされている必要があります。

### 手順

1. [StarRocks Docker Hub](https://hub.docker.com/r/starrocks/artifacts-ubuntu/tags)からStarRocks Dockerイメージをダウンロードします。イメージのタグに基づいて特定のバージョンを選択できます。

   - Ubuntu 22.04を使用している場合：

     ```Bash
     # <image_tag>をダウンロードしたいイメージのタグに置き換えてください。例えば、2.5.4。
     docker pull starrocks/artifacts-ubuntu:<image_tag>
     ```

   - ARMベースのCentOS 7.9を使用している場合：

     ```Bash
     # <image_tag>をダウンロードしたいイメージのタグに置き換えてください。例えば、2.5.4。
     docker pull starrocks/artifacts-centos7:<image_tag>
     ```

2. 以下のコマンドを実行して、DockerイメージからホストマシンにStarRocksのデプロイメントファイルをコピーします。

   - Ubuntu 22.04を使用している場合：

     ```Bash
     # <image_tag>をダウンロードしたイメージのタグに置き換えてください。例えば、2.5.4。
     docker run --rm starrocks/artifacts-ubuntu:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   - ARMベースのCentOS 7.9を使用している場合：

     ```Bash
     # <image_tag>をダウンロードしたイメージのタグに置き換えてください。例えば、2.5.4。
     docker run --rm starrocks/artifacts-centos7:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   デプロイメントファイルには以下のディレクトリが含まれます：

   | **ディレクトリ**        | **説明**                                              |
   | -------------------- | ------------------------------------------------------------ |
   | **be_artifacts**     | このディレクトリには、BEまたはCNのデプロイメントディレクトリ**be**、StarRocksのライセンスファイル**LICENSE.txt**、およびStarRocksの通知ファイル**NOTICE.txt**が含まれます。 |
   | **broker_artifacts** | このディレクトリには、Brokerのデプロイメントディレクトリ**apache_hdfs_broker**が含まれます。StarRocks 2.5以降、一般的なシナリオではBrokerノードをデプロイする必要はありません。StarRocksクラスターにBrokerノードをデプロイする必要がある場合は、[Brokerのデプロイ](../deployment/deploy_broker.md)を参照してください。 |
   | **fe_artifacts**     | このディレクトリには、FEのデプロイメントディレクトリ**fe**、StarRocksのライセンスファイル**LICENSE.txt**、およびStarRocksの通知ファイル**NOTICE.txt**が含まれます。 |

3. **fe**ディレクトリをすべてのFEインスタンスに、**be**ディレクトリをすべてのBEまたはCNインスタンスに配布し、[手動でデプロイ](../deployment/deploy_manually.md)します。
