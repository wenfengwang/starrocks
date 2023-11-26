---
displayed_sidebar: "Japanese"
---

# デプロイの概要

この章では、プロダクション環境でのStarRocksクラスタのデプロイ、アップグレード、ダウングレード方法について説明します。

## デプロイ手順

デプロイ手順の概要は以下の通りであり、後のトピックで詳細を説明します。

StarRocksのデプロイは一般的に以下の手順に従います：

1. StarRocksのデプロイに必要な[ハードウェアおよびソフトウェアの要件](../deployment/deployment_prerequisites.md)を確認します。

   StarRocksをデプロイする前に、CPU、メモリ、ストレージ、ネットワーク、オペレーティングシステム、および依存関係など、サーバーが満たす必要がある前提条件を確認します。

2. [クラスタのサイズを計画します](../deployment/plan_cluster.md)。

   クラスタ内のFEノードとBEノードの数、およびサーバーのハードウェア仕様を計画します。

3. [環境設定を確認します](../deployment/environment_configurations.md)。

   サーバーが準備できたら、StarRocksをデプロイする前に、いくつかの環境設定を確認および変更する必要があります。

4. [デプロイファイルを準備します](../deployment/prepare_deployment_files.md)。

   - x86ベースのCentOS 7.9にStarRocksをデプロイする場合は、公式ウェブサイトで提供されているソフトウェアパッケージを直接ダウンロードして展開することができます。
   - ARMアーキテクチャのCPUを使用するか、Ubuntu 22.04にStarRocksをデプロイする場合は、StarRocks Dockerイメージからデプロイファイルを準備する必要があります。
   - Kubernetes上にStarRocksをデプロイする場合は、この手順をスキップすることができます。

5. StarRocksをデプロイします。

   - 分散ストレージとコンピュートアーキテクチャを特徴とする共有データのStarRocksクラスタをデプロイする場合は、[共有データのStarRocksのデプロイと使用](../deployment/shared_data/s3.md)を参照してください。
   - ローカルストレージを使用する共有しないStarRocksクラスタをデプロイする場合は、以下のオプションがあります：

     - [手動でStarRocksをデプロイする](../deployment/deploy_manually.md)。
     - [オペレーターを使用してKubernetes上にStarRocksをデプロイする](../deployment/sr_operator.md)。
     - [Helmを使用してKubernetes上にStarRocksをデプロイする](../deployment/helm.md)。
     - [AWS上にStarRocksをデプロイする](../deployment/starrocks_on_aws.md)。

6. 必要な[デプロイ後のセットアップ](../deployment/post_deployment_setup.md)を実行します。

   StarRocksクラスタを本番環境で使用する前に、さらなるセットアップが必要です。これには、初期アカウントのセキュリティ確保やパフォーマンスに関連するシステム変数の設定などが含まれます。

## アップグレードとダウングレード

StarRocksクラスタを初めてインストールするのではなく、既存のStarRocksクラスタを後のバージョンにアップグレードする予定の場合は、アップグレード手順と考慮すべき問題については、[StarRocksのアップグレード](../deployment/upgrade.md)を参照してください。

StarRocksクラスタをダウングレードする手順については、[StarRocksのダウングレード](../deployment/downgrade.md)を参照してください。
