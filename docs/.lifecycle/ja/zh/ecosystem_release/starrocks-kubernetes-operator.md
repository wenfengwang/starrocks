---
displayed_sidebar: Chinese
---

# starrocks-kubernetes-operator

## リリースノート

StarRocksが提供するOperatorは、Kubernetes環境でStarRocksクラスターをデプロイするために使用され、クラスターのコンポーネントにはFE、BE、CNが含まれます。

**使用文書**：

Kubernetes上でStarRocksクラスターをデプロイするための以下の2つの方法がサポートされています：

- [StarRocks CRDを直接使用してStarRocksクラスターをデプロイする](https://docs.starrocks.io/zh/docs/deployment/sr_operator/)
- [Helm Chartを介してOperatorとStarRocksクラスターをデプロイする](https://docs.starrocks.io/zh/docs/deployment/helm/)

**ソースコードダウンロードアドレス：**

- [starrocks-kubernetes-operator](https://github.com/StarRocks/starrocks-kubernetes-operator)
- [kube-starrocks Helm Chart](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks)

**リソースダウンロードアドレス:**

- **ダウンロードアドレスのプレフィックス**

   `https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v${operator_version}/${resource_name}`

- **リソース名**
  - カスタムリソース StarRocksCluster：**starrocks.com_starrocksclusters.yaml**
  - StarRocks Operatorのデフォルト設定ファイル：**operator.yaml**
  - Helm Chartには、`kube-starrocks` Chart `kube-starrocks-${chart_version}.tgz`が含まれています。`kube-starrocks` Chartにはさらに2つのサブChartがあります。`starrocks` Chart `starrocks-${chart_version}.tgz`と`operator` Chart `operator-${chart_version}.tgz`です。

例えば、バージョン1.8.6の`kube-starrocks` Chartのダウンロードアドレスはこちらです：[kube-starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v1.8.6/kube-starrocks-1.8.6.tgz)

**バージョン要件**

- Kubernetes：1.18以上
- Go：1.19以上

## リリース履歴

### 1.8

**1.8.6**

**バグ修正**

以下の問題が修正されました：

Stream Loadジョブを実行するときに、エラー `sendfile() failed (32: Broken pipe) while sending request to upstream`が返される問題。NginxがリクエストボディをFEに送信した後、FEはリクエストをBEにリダイレクトします。この時、Nginxのキャッシュされたデータが失われている可能性があります。[#303](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/303)

**ドキュメント**

- [FE proxyを使用してKubernetesの外部ネットワークからStarRocksクラスターにデータをインポートする](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/load_data_using_stream_load_howto.md)
- [Helmを使用してrootユーザーのパスワードを更新する](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/change_root_password_howto.md)

**1.8.5**

**機能改善**

- **[Helm Chart] Operatorのservice accountにカスタムアノテーションとラベルをサポート**。デフォルトでは`starrocks`という名前のservice accountがOperatorに対して作成されますが、ユーザーは**values.yaml**ファイルの`serviceAccount`セクションに`annotations`と`labels`フィールドを設定することで、service account `starrocks`のアノテーションとラベルをカスタマイズできます。`operator.global.rbac.serviceAccountName`フィールドは非推奨になりました。[#291](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/291)
- **[Operator] FE serviceがIstioの明示的なプロトコル選択をサポート**。Kubernetes環境にIstioがインストールされている場合、IstioはStarRocksクラスターからのトラフィックが使用するプロトコルを特定する必要があり、ルーティングや豊富なメトリクスなどの追加機能を提供します。したがって、FE serviceは`appProtocol`フィールドを使用して、そのプロトコルがMySQLプロトコルであることを明示的に宣言します。この改善は特に重要です。なぜなら、MySQLプロトコルはserver-firstプロトコルであり、自動プロトコル検出と互換性がなく、時には接続失敗を引き起こす可能性があるからです。[#288](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/288)

**バグ修正**

以下の問題が修正されました：

- **[Helm Chart]** `starrocks.initPassword.enabled`が`true`であり、`starrocks.starrocksCluster.name`に値が指定されている場合、StarRocks内のrootユーザーのパスワードが初期化されませんでした。これは、initpwd podがFE serviceのドメイン名を誤って使用してFE serviceに接続していたためです。具体的には、この場合、FE serviceのドメイン名は`starrocks.starrocksCluster.name`に指定された値を使用していましたが、initpwd podは依然として`starrocks.nameOverride`フィールドの値を使用してFE serviceのドメイン名を構成していました。[#292](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/292)

**アップグレード説明**

- **[Helm Chart]** `starrocks.starrocksCluster.name`に指定された値と`starrocks.nameOverride`の値が異なる場合、FE、BE、CNの古い`configmap`は削除され、新しい名前の`configmap`が作成されます。**これによりFE/BE/CNのpodが再起動する可能性があります。**

**1.8.4**

**新機能**

- **[Helm Chart]** PrometheusとServiceMonitor CRを使用してStarRocksクラスターのメトリクスを監視できます。使用ドキュメントは[PrometheusとGrafanaとの統合](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-prometheus-grafana.md)を参照してください。[#284](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/284)
- **[Helm Chart]** **values.yaml**の`starrocksCnSpec`セクションに`storagespec`と関連フィールドを追加し、StarRocksクラスター内のCNノードのログボリュームを設定できます。[#280](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/280)
- StarRocksCluster CRDに`terminationGracePeriodSeconds`フィールドを追加し、StarRocksClusterリソースを削除または更新する際の優雅な終了猶予期間を設定できます。[#283](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/283)
- StarRocksCluster CRDに`startupProbeFailureSeconds`フィールドを追加し、StarRocksClusterリソース内のpodの起動プローブの失敗閾値を設定できます。起動プローブが指定時間内に成功応答を返さない場合、失敗と見なされます。[#271](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/271)

**バグ修正**

以下の問題が修正されました：

- StarRocksクラスターに複数のFE podが存在する場合、FE proxyがSTREAM LOADリクエストを正しく処理できませんでした。[#269](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/269)

**ドキュメント**

- [クイックスタート：ローカルにStarRocksクラスターをデプロイする](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/local_installation_how_to.md)。

- [異なる設定の StarRocks クラスターをデプロイする](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)。例えば、[全機能を備えた StarRocks クラスターをデプロイする](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_all_features.yaml)。
- [StarRocks クラスターの管理ガイド](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/doc)。例えば、[ログと関連フィールドの設定方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/logging_and_related_configurations_howto.md)や[外部の configmaps や secrets をマウントする方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md)。

**1.8.3**

**アップグレードノート**

- **[Helm Chart]** デフォルトの **fe.conf** ファイルに `JAVA_OPTS_FOR_JDK_11` を追加。デフォルトの **fe.conf** ファイルを使用している場合、v1.8.3 へのアップグレード時に **FE Pod が再起動されます**。[#257](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/257)

**新機能**

- **[Helm Chart]** `watchNamespace` フィールドを追加し、Operator が監視する特定の namespace を指定。そうでなければ、Operator は Kubernetes クラスター内のすべての namespace を監視します。ほとんどの場合、この機能は必要ありません。Kubernetes クラスターが多くのノードを管理しており、Operator がすべての namespace を監視して多くのメモリリソースを消費する場合、この機能を使用できます。[#261](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/261)
- **[Helm Chart]** **values.yaml** の `starrocksFeProxySpec` に `Ports` フィールドを追加し、ユーザーが FE プロキシサービスの NodePort を指定できるようにしました。[#258](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/258)

**機能改善**

- **nginx.conf** の `proxy_read_timeout` パラメータの値を 60s から 600s に変更して、タイムアウトを防ぎます。

**1.8.2**

**機能改善**

- Operator pod のメモリ使用上限を上げて、メモリオーバーフローを防ぎます。[#254](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/254)

**1.8.1**

**新機能**

- `configMaps` と `secrets` で `subpath` フィールドを使用することをサポートし、ユーザーがファイルを指定されたディレクトリにマウントし、そのディレクトリの既存の内容を上書きしないようにします。[#249](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/249)
- StarRocks クラスター CRD に `ports` フィールドを追加し、ユーザーがサービスのポートをカスタマイズできるようにします。[#244](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/244)

**機能改善**

- StarRocks クラスターの `BeSpec` または `CnSpec` を削除するとき、関連する Kubernetes リソースを削除して、クラスターの状態をクリーンで一貫性のあるものにします。[#245](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/245)

**1.8.0**

**アップグレードノートと挙動の変更**

- **[Operator]** StarRocksCluster CRD と Operator をアップグレードするには、新しい StarRocksCluster CRD **starrocks.com_starrocksclusters.yaml** と **operator.yaml** を手動で適用する必要があります。

- **[Helm Chart]**

  - Helm Chart をアップグレードするには、以下の手順を実行します：

    1. **values migration tool** を使用して、以前の **values.yaml** ファイルの形式を新しい形式に調整します。異なるオペレーティングシステム用の値移行ツールは [Assets](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/tag/v1.8.0) セクションからダウンロードできます。このツールのヘルプ情報は `migrate-chart-value --help` コマンドを実行することで取得できます。[#206](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/206)

         ```Bash
         migrate-chart-value --input values.yaml --target-version v1.8.0 --output ./values-v1.8.0.yaml
         ```

    2. Helm Chart リポジトリを更新します。

         ```Bash
         helm repo update
         ```

    3. 調整された **values.yaml** ファイルを使用して `helm upgrade` コマンドを実行し、Chart `kube-starrocks` をインストールします。

         ```Bash
         helm upgrade <release-name> starrocks-community/kube-starrocks -f values-v1.8.0.yaml
         ```

  - 2つのサブチャート `operator` と `starrocks` を親チャート kube-starrocks に追加しました。対応するサブチャートを指定することで StarRocks Operator または StarRocks クラスターをインストールできます。これにより、StarRocks クラスターをより柔軟に管理できるようになります。例えば、1つの StarRocks Operator と複数の StarRocks クラスターをデプロイすることができます。

**新機能**

- **[Helm Chart]** 1つの Kubernetes クラスター内に複数の StarRocks クラスターをデプロイします。異なる namespace にサブチャート `starrocks` をインストールすることで、Kubernetes クラスター内に複数の StarRocks クラスターをデプロイできます。[#199](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/199)
- **[Helm Chart]** `helm install` コマンドを実行する際に、[StarRocks クラスターの root ユーザーの初期パスワードを設定](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/initialize_root_password_howto.md)できます。ただし、`helm upgrade` コマンドではこの機能はサポートされていないことに注意してください。
- **[Helm Chart]** Datadog との統合。Datadog との統合により、StarRocks クラスターのメトリクスとログを提供できます。この機能を有効にするには、**values.yaml** ファイルで Datadog 関連のフィールドを設定する必要があります。使用方法については、[Datadog との統合](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-with-datadog.md)を参照してください。[#197](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/197) [#208](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/208)
- **[Operator]** 非 root ユーザーとして pod を実行します。`runAsNonRoot` フィールドを追加して、Kubernetes 内の pod が非 root ユーザーとして実行されることを許可し、セキュリティを強化します。[#195](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/195)
- **[Operator]** FE プロキシ。FE プロキシを追加して、外部クライアントやデータインポートツールが Kubernetes 内の StarRocks クラスターにアクセスできるようにします。例えば、STREAM LOAD 構文を使用して Kubernetes 内の StarRocks クラスターにデータをインポートできます。[#211](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/211)

**機能改善**

- StarRocksCluster CRDに`subpath`フィールドを追加しました。[#212](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/212)
- FEメタデータが使用するディスク容量の上限をより大きく設定しました。FEメタデータを保存するディスクの利用可能な空き容量がこの値未満になると、FEコンテナは停止します。[#210](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/210)
