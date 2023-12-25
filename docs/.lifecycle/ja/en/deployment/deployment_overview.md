---
displayed_sidebar: English
---

# デプロイメント概要

この章では、本番環境でStarRocksクラスタをデプロイ、アップグレード、およびダウングレードする方法について説明します。

## デプロイメント手順

デプロイメント手順の概要は以下の通りで、詳細は後続のトピックで提供されます。

StarRocksのデプロイメントは、一般的にここに概説された手順に従います：

1. StarRocksデプロイメントのための[ハードウェアとソフトウェアの要件](../deployment/deployment_prerequisites.md)を確認してください。

   StarRocksをデプロイする前に、サーバーが満たすべき前提条件を確認してください。これにはCPU、メモリ、ストレージ、ネットワーク、オペレーティングシステム、および依存関係が含まれます。

2. [クラスタサイズを計画する](../deployment/plan_cluster.md)。

   クラスター内のFEノードとBEノードの数、およびサーバーのハードウェア仕様を計画します。

3. [環境設定をチェックする](../deployment/environment_configurations.md)。

   サーバーが準備できたら、StarRocksをデプロイする前にいくつかの環境設定をチェックし、必要に応じて変更します。

4. [デプロイメントファイルを準備する](../deployment/prepare_deployment_files.md)。

   - x86ベースのCentOS 7.9にStarRocksをデプロイする場合、公式ウェブサイトから提供されているソフトウェアパッケージを直接ダウンロードして展開できます。
   - ARMアーキテクチャCPUやUbuntu 22.04でStarRocksをデプロイする場合は、StarRocks Dockerイメージからデプロイメントファイルを準備する必要があります。
   - Kubernetes上にStarRocksをデプロイする場合は、このステップをスキップします。

5. StarRocksをデプロイします。

   - 分散ストレージとコンピュートアーキテクチャを特徴とする共有データStarRocksクラスタをデプロイする場合は、[共有データStarRocksのデプロイと使用](../deployment/shared_data/s3.md)の指示に従ってください。
   - ローカルストレージを使用する共有ナッシングStarRocksクラスタをデプロイする場合、以下のオプションがあります：

     - [StarRocksを手動でデプロイする](../deployment/deploy_manually.md)。
     - [オペレーターを使用してKubernetes上にStarRocksをデプロイする](../deployment/sr_operator.md)。
     - [Helmを使用してKubernetes上にStarRocksをデプロイする](../deployment/helm.md)。
     - [AWS上にStarRocksをデプロイする](../deployment/starrocks_on_aws.md)。

6. 必要な[デプロイメント後のセットアップ](../deployment/post_deployment_setup.md)を実施します。

   StarRocksクラスタを本番運用する前に、初期アカウントのセキュリティ強化やパフォーマンス関連のシステム変数の設定など、さらなるセットアップが必要です。

## アップグレードとダウングレード

StarRocksを初めてインストールするのではなく、既存のStarRocksクラスタを新しいバージョンにアップグレードする予定の場合は、[StarRocksのアップグレード](../deployment/upgrade.md)に関する情報を参照し、アップグレード手順とアップグレード前に考慮すべき問題について確認してください。

StarRocksクラスタをダウングレードする指示については、[StarRocksのダウングレード](../deployment/downgrade.md)を参照してください。
