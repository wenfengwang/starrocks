---
displayed_sidebar: Chinese
---

# 部署概要

本セクションでは、StarRocks クラスターを本番環境にデプロイ、アップグレード、ダウングレードする方法について説明します。

## デプロイメントプロセス

以下はデプロイメントプロセスの概要です。詳細は各ステップのドキュメントを参照してください。

StarRocks のデプロイメントは通常、以下のステップに従います：

1. StarRocks のデプロイメントに必要な[ソフトウェアとハードウェアの要件](../deployment/deployment_prerequisites.md)を確認します。

   StarRocks をデプロイする前に、サーバーが満たすべき前提条件を確認してください。これには CPU、メモリ、ストレージ、ネットワーク、オペレーティングシステム、および依存関係が含まれます。

2. [クラスターの規模を計画する](../deployment/plan_cluster.md)。

   クラスター内の FE ノードと BE ノードの数、およびサーバーのハードウェア仕様を計画します。

3. [環境設定を確認する](../deployment/environment_configurations.md)。

   サーバーが準備できたら、StarRocks をデプロイする前に一部の環境設定を確認して変更する必要があります。

4. [デプロイメントファイルを準備する](../deployment/prepare_deployment_files.md)。

   - x86 アーキテクチャの CentOS 7.9 プラットフォーム上に StarRocks をデプロイする場合は、公式ウェブサイトから提供されるソフトウェアパッケージを直接ダウンロードして解凍することができます。
   - ARM アーキテクチャの CPU または Ubuntu 22.04 上に StarRocks をデプロイする場合は、StarRocks の Docker イメージを使用してデプロイメントファイルを準備する必要があります。
   - Kubernetes 上に StarRocks をデプロイする場合は、このステップをスキップできます。

5. StarRocks をデプロイする。

   - ストレージと計算が分離されたアーキテクチャの StarRocks クラスターをデプロイする場合は、[StarRocks ストレージと計算が分離されたクラスターのデプロイ](../deployment/shared_data/s3.md)を参照してください。
   - ストレージと計算が統合されたアーキテクチャの StarRocks クラスターをデプロイする場合は、以下のオプションがあります：

     - [StarRocks を手動でデプロイする](../deployment/deploy_manually.md)。
     - [Operator を使用して StarRocks クラスターをデプロイする](../deployment/sr_operator.md)。
     - [Helm を使用して StarRocks クラスターをデプロイする](../deployment/helm.md)。

6. 必要な[デプロイメント後の設定](../deployment/post_deployment_setup.md)を実施します。

   StarRocks クラスターを本番環境に投入する前に、初期アカウントの管理や一部のパフォーマンスに関連するシステム変数の設定など、クラスターのさらなる設定が必要です。

## アップグレードとダウングレード

既存の StarRocks を新しいバージョンにアップグレードする予定で、StarRocks を初めてインストールするのではない場合は、[StarRocks のアップグレード](../deployment/upgrade.md)を参照して、アップグレード方法とアップグレード前に考慮すべき事項について確認してください。

StarRocks クラスターをダウングレードする方法については、[StarRocks のダウングレード](../deployment/downgrade.md)を参照してください。
