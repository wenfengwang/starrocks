---
displayed_sidebar: "Japanese"
---

# デプロイメント概要

この章では、本番環境でStarRocksクラスタを展開、アップグレード、ダウングレードする方法について説明します。

## デプロイ手順

デプロイ手順の概要は次のとおりです。後続のトピックでは詳細を提供します。

StarRocksのデプロイは、一般的に以下の手順に従います:

1. StarRocksの展開に先立ち、[ハードウェアおよびソフトウェア要件](../deployment/deployment_prerequisites.md) を確認してください。

   StarRocksを展開する前に、CPU、メモリ、ストレージ、ネットワーク、オペレーティングシステム、および依存関係など、サーバが満たす必要がある前提条件を確認してください。

2. [クラスタサイズの計画](../deployment/plan_cluster.md) を立ててください。

   クラスタ内のFEノードおよびBEノードの数、およびサーバのハードウェア仕様を計画してください。

3. [環境構成の確認](../deployment/environment_configurations.md) を行ってください。

   サーバが準備できたら、StarRocksを展開する前に、いくつかの環境構成を確認および修正する必要があります。

4. [展開ファイルの準備](../deployment/prepare_deployment_files.md) を行ってください。

   - x86ベースのCentOS 7.9にStarRocksを展開する場合、当社の公式ウェブサイトで提供されるソフトウェアパッケージを直接ダウンロードおよび展開できます。
   - ARMアーキテクチャCPUを使用するか、Ubuntu 22.04にStarRocksを展開する場合は、StarRocks Dockerイメージから展開ファイルを準備する必要があります。
   - Kubernetes上にStarRocksを展開したい場合は、この手順をスキップできます。

5. StarRocksの展開

   - 集中型データを使用するStarRocksクラスタを展開したい場合（データと計算を分離するアーキテクチャを使用）、手順については[共有データStarRocksの展開と使用](../deployment/shared_data/s3.md) を参照してください。
   - ローカルストレージを使用する共有なしのStarRocksクラスタを展開したい場合は、以下のオプションがあります:

     - [手動でStarRocksを展開](../deployment/deploy_manually.md)。
     - [StarRocksオペレータを使用してKubernetes上にStarRocksを展開](../deployment/sr_operator.md)。
     - [Helmを使用してKubernetes上にStarRocksを展開](../deployment/helm.md)。
     - [AWS上にStarRocksを展開](../deployment/starrocks_on_aws.md)。

6. 必要な[展開後セットアップ](../deployment/post_deployment_setup.md) を実行してください。

   StarRocksクラスタを本番環境に投入する前に、さらなるセットアップ措置が必要です。これらの措置には、初期アカウントのセキュリティ確保およびいくつかの性能関連システム変数の設定が含まれます。

## アップグレードとダウングレード

StarRocksを初めてインストールするのではなく、既存のStarRocksクラスタを後のバージョンにアップグレードする予定がある場合は、アップグレード手順や考慮すべき問題については[StarRocksのアップグレード](../deployment/upgrade.md) を参照してください。

StarRocksクラスタをダウングレードする手順については、[StarRocksのダウングレード](../deployment/downgrade.md) を参照してください。
