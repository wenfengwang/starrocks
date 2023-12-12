---
displayed_sidebar: "Japanese"
---

# デプロイメントの概要

この章では、プロダクション環境で StarRocks クラスタを展開、アップグレード、ダウングレードする方法について説明します。

## デプロイメント手順

デプロイメント手順の概要は次のとおりで、後のトピックでは詳細を提供します。

StarRocks のデプロイメントは、一般的に以下の手順に従います:

1. [ハードウェアおよびソフトウェアの要件](../deployment/deployment_prerequisites.md) を確認します。

   StarRocks を展開する前に、CPU、メモリ、ストレージ、ネットワーク、オペレーティング システム、および依存関係など、サーバが満たす必要がある前提条件を確認します。

2. [クラスタサイズを計画](../deployment/plan_cluster.md) します。

   クラスタ内の FE ノードおよび BE ノードの数、およびサーバのハードウェア仕様を計画します。

3. [環境設定を確認](../deployment/environment_configurations.md) します。

   サーバの準備が整ったら、StarRocks を展開する前に、いくつかの環境設定を確認および変更する必要があります。

4. [展開ファイルを準備](../deployment/prepare_deployment_files.md) します。

   - x86 ベースの CentOS 7.9 に StarRocks を展開する場合、公式ウェブサイトで提供されているソフトウェアパッケージを直接ダウンロードおよび抽出できます。
   - ARM アーキテクチャの CPU を使用するか、Ubuntu 22.04 に StarRocks を展開する場合は、StarRocks の Docker イメージから展開ファイルを準備する必要があります。
   - Kubernetes に StarRocks を展開する場合は、この手順をスキップできます。

5. StarRocks を展開します。

   - 分散ストレージとコンピュートアーキテクチャを特徴とする共有データ StarRocks クラスタを展開する場合は、[共有データ StarRocks の展開および使用](../deployment/shared_data/s3.md) を参照して手順を確認します。
   - ローカルストレージを使用する共有なし StarRocks クラスタを展開する場合、以下のオプションがあります:

     - [手動で StarRocks を展開](../deployment/deploy_manually.md)します。
     - [オペレーターを使用して Kubernetes に StarRocks を展開](../deployment/sr_operator.md)します。
     - [Helm を使用して Kubernetes に StarRocks を展開](../deployment/helm.md)します。
     - [AWS に StarRocks を展開](../deployment/starrocks_on_aws.md)します。

6. 必要な [展開後のセットアップ](../deployment/post_deployment_setup.md) を実行します。

   StarRocks クラスタを本番環境に投入する前に追加のセットアップが必要です。これには、初期アカウントのセキュリティ確保といくつかのパフォーマンス関連のシステム変数の設定が含まれます。

## アップグレードとダウングレード

既存の StarRocks クラスタを後のバージョンにアップグレードする場合、StarRocks を初めてインストールする代わりに、[StarRocks のアップグレード](../deployment/upgrade.md) を参照して、アップグレード手順と考慮すべき問題についての情報を確認してください。

StarRocks クラスタをダウングレードする手順については、[StarRocks のダウングレード](../deployment/downgrade.md) を参照してください。
