---
displayed_sidebar: "Japanese"
---

# starrocks-kubernetes-operator

## 通知

StarRocksが提供するOperatorは、Kubernetes環境でStarRocksクラスタをデプロイするために使用されます。StarRocksクラスタのコンポーネントには、FE、BE、CNが含まれます。

**ユーザーガイド：** Kubernetes上でStarRocksクラスタをデプロイするためには、以下の方法を使用できます：

- [StarRocks CRDを直接使用してStarRocksクラスタをデプロイする](https://docs.starrocks.io/docs/deployment/sr_operator/)
- [Helm Chartを使用してOperatorとStarRocksクラスタをデプロイする](https://docs.starrocks.io/zh/docs/deployment/helm/)

**ソースコード：**

- [starrocks-kubernetes-operator](https://github.com/StarRocks/starrocks-kubernetes-operator)
- [kube-starrocks Helm Chart](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks)

**リソースのダウンロードURL：**

- **URLプレフィックス**：

    `https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v${operator_version}/${resource_name}`

- **リソース名**

  - StarRocksCluster CRD: `starrocks.com_starrocksclusters.yaml`
  - StarRocks Operatorのデフォルト設定ファイル: `operator.yaml`
  - Helm Chart（`kube-starrocks` Chart） `kube-starrocks-${chart_version}.tgz`。`kube-starrocks` Chartは、`starrocks` Chart `starrocks-${chart_version}.tgz`と`operator` Chart `operator-${chart_version}.tgz`の2つのサブチャートに分割されます。

例えば、kube-starrocksチャートv1.8.6のダウンロードURLは次のとおりです：[kube-starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v1.8.6/kube-starrocks-1.8.6.tgz)

**バージョン要件**

- Kubernetes: 1.18以降
- Go: 1.19以降

## **リリースノート**

## 1.8

**1.8.6**

**バグ修正**

以下の問題が修正されました：

- ストリームロードジョブ中に `sendfile() failed (32: Broken pipe) while sending request to upstream` エラーが発生する問題を修正しました。NginxがリクエストボディをFEに送信した後、FEはリクエストをBEにリダイレクトします。この時点で、Nginxにキャッシュされたデータは既に失われている可能性があります。[#303](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/303)

**ドキュメント**

- [FEプロキシを使用してKubernetesネットワーク外からStarRocksにデータをロードする方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/load_data_using_stream_load_howto.md)
- [Helmを使用してルートユーザーのパスワードを更新する方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/change_root_password_howto.md)

**1.8.5**

**改善**

- **[Helm Chart] サービスアカウントの `annotations` と `labels` をカスタマイズできるようにしました**：Operatorはデフォルトで `starrocks` という名前のサービスアカウントを作成しますが、ユーザーは `values.yaml` の `serviceAccount` の `annotations` と `labels` フィールドを指定することで、Operatorのサービスアカウント `starrocks` のアノテーションとラベルをカスタマイズすることができます。`operator.global.rbac.serviceAccountName` フィールドは非推奨です。[#291](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/291)
- **[Operator] FEサービスはIstioのための明示的なプロトコル選択をサポートします**：Kubernetes環境にIstioがインストールされている場合、IstioはStarRocksクラスタからのトラフィックのプロトコルを決定する必要があります。これにより、ルーティングや豊富なメトリクスなどの追加機能を提供することができます。そのため、FEサービスは `appProtocol` フィールドで明示的にMySQLプロトコルを定義します。この改善は特に重要であり、MySQLプロトコルは自動プロトコル検出と互換性がなく、接続の失敗が発生する場合があります。[#288](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/288)

**バグ修正**

以下の問題が修正されました：

- **[Helm Chart]** `starrocks.initPassword.enabled` がtrueであり、`starrocks.starrocksCluster.name` の値が指定されている場合、StarRocksのルートユーザーのパスワードが正常に初期化されない場合があります。これは、initpwdポッドがFEサービスに接続するために使用する間違ったFEサービスドメイン名によるものです。具体的には、このシナリオでは、FEサービスドメイン名は `starrocks.starrocksCluster.name` で指定された値を使用しますが、initpwdポッドは引き続き `starrocks.nameOverride` フィールドの値を使用してFEサービスドメイン名を形成します。([#292](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/292))

**アップグレードノート**

- **[Helm Chart]** `starrocks.starrocksCluster.name` で指定された値が `starrocks.nameOverride` の値と異なる場合、FE、BE、CNの古いconfigmapが削除されます。FE、BE、CNの新しい名前のconfigmapが作成されます。**これにより、FE、BE、CNのポッドが再起動する可能性があります。**

**1.8.4**

**機能**

- **[Helm Chart]** StarRocksクラスタのメトリクスをPrometheusとServiceMonitor CRを使用して監視できるようになりました。ユーザーガイドについては、[PrometheusとGrafanaとの統合](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-prometheus-grafana.md)を参照してください。[#284](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/284)
- **[Helm Chart]** StarRocksクラスタのCNノードのログボリュームを設定するために、**values.yaml** の `starrocksCnSpec` に `storagespec` とその他のフィールドを追加しました。[#280](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/280)
- StarRocksCluster CRDに `terminationGracePeriodSeconds` を追加し、StarRocksClusterリソースが削除または更新される際にポッドを強制的に終了するまでの待機時間を設定できるようにしました。[#283](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/283)
- StarRocksCluster CRDに `startupProbeFailureSeconds` フィールドを追加し、StarRocksClusterリソースのポッドの起動プローブの失敗閾値を設定できるようにしました。[#271](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/271)

**バグ修正**

以下の問題が修正されました：

- StarRocksクラスタに複数のFEポッドが存在する場合、FEプロキシはSTREAM LOADリクエストを正しく処理できません。[#269](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/269)

**ドキュメント**

- [ローカルStarRocksクラスタをデプロイするクイックスタートを追加しました](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/local_installation_how_to.md)。
- 異なる設定でStarRocksクラスタをデプロイする方法に関するより多くのユーザーガイドを追加しました。例えば、[すべてのサポートされている機能を備えたStarRocksクラスタをデプロイする方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_all_features.yaml)。詳細なユーザーガイドについては、[ドキュメント](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)を参照してください。
- StarRocksクラスタの管理方法に関するより多くのユーザーガイドを追加しました。例えば、[ログと関連するフィールドの設定](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/logging_and_related_configurations_howto.md)や[外部のconfigmapやsecretをマウントする方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md)など。詳細なユーザーガイドについては、[ドキュメント](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/doc)を参照してください。

**1.8.3**

**アップグレードノート**

- **[Helm Chart]** デフォルトの **fe.conf** ファイルに `JAVA_OPTS_FOR_JDK_11` を追加しました。デフォルトの **fe.conf** ファイルを使用し、helm chartをv1.8.3にアップグレードする場合、**FEポッドが再起動する可能性があります**。[#257](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/257)

**機能**

- **[Helm Chart]** オペレータが監視する必要がある唯一のネームスペースを指定するための `watchNamespace` フィールドを追加しました。それ以外の場合、オペレータはKubernetesクラスタのすべてのネームスペースを監視します。ほとんどの場合、この機能を使用する必要はありません。Kubernetesクラスタが多くのノードを管理し、オペレータがすべてのネームスペースを監視し、多くのメモリリソースを消費する場合にこの機能を使用できます。[#261](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/261)
- **[Helm Chart]** **values.yaml** ファイルの `starrocksFeProxySpec` に `Ports` フィールドを追加し、ユーザーがFEプロキシサービスのNodePortを指定できるようにしました。[#258](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/258)

**改善**

- **nginx.conf** ファイルの `proxy_read_timeout` パラメータの値を60秒から600秒に変更し、タイムアウトを回避するようにしました。

**1.8.2**

**改善**

- OOMを回避するために、オペレータポッドで許可される最大メモリ使用量を増やしました。[#254](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/254)

**1.8.1**

**機能**

- [configMapsとsecretsのsubpathフィールド](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md#3-mount-configmaps-to-a-subpath-by-helm-chart)をサポートし、ユーザーがこれらのリソースから特定のファイルやディレクトリをマウントできるようにしました。[#249](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/249)
- StarRocksクラスタのサービスのポートをカスタマイズできるようにするために、StarRocksクラスタのCRDに `ports` フィールドを追加しました。[#244](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/244)

**改善**

- StarRocksクラスタの `BeSpec` または `CnSpec` が削除された場合に関連するKubernetesリソースを削除し、クラスタのクリーンで一貫性のある状態を確保するようにしました。[#245](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/245)

**1.8.0**

**アップグレードノートと動作の変更**

- **[Operator]** StarRocksCluster CRDとオペレータをアップグレードするには、新しいStarRocksCluster CRD **starrocks.com_starrocksclusters.yaml** と **operator.yaml** を手動で適用する必要があります。

- **[Helm Chart]**

  - Helm Chartをアップグレードするには、次の手順を実行する必要があります：

    1. **values.yaml** ファイルの形式を新しい形式に調整するために、**values migration tool** を使用します。異なるオペレーティングシステム用のvalues migration toolは、[Assets](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/tag/v1.8.0)セクションからダウンロードできます。このツールのヘルプ情報は、`migrate-chart-value --help` コマンドを実行することで取得できます。[#206](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/206)

       ```Bash
       migrate-chart-value --input values.yaml --target-version v1.8.0 --output ./values-v1.8.0.yaml
       ```

    2. Helm Chartリポジトリを更新します。

       ```Bash
       helm repo update
       ```

    3. `helm upgrade` コマンドを実行して、調整された **values.yaml** ファイルをStarRocksのhelm chart kube-starrocksに適用します。

       ```Bash
       helm upgrade <release-name> starrocks-community/kube-starrocks -f values-v1.8.0.yaml
       ```

  - [operator](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/operator)と[starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/starrocks)の2つのサブチャートがkube-starrocks helm chartに追加されました。対応するサブチャートを指定することで、StarRocksオペレータまたはStarRocksクラスタをそれぞれインストールできます。これにより、複数のStarRocksオペレータと複数のStarRocksクラスタをデプロイするなど、StarRocksクラスタをより柔軟に管理することができます。

**機能**

- **[Helm Chart] Kubernetesクラスタ内の複数のStarRocksクラスタ**。`starrocks` Helmサブチャートをインストールすることで、Kubernetesクラスタ内の異なるネームスペースに複数のStarRocksクラスタをデプロイすることをサポートします。[#199](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/199)
- **[Helm Chart]** `helm install` コマンドを実行する際に、StarRocksクラスタのルートユーザーの初期パスワードを[設定できるようにしました](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/initialize_root_password_howto.md)。ただし、`helm upgrade` コマンドはこの機能をサポートしていません。
- **[Helm Chart] Datadogとの統合**：Datadogと統合して、StarRocksクラスタのメトリクスとログを収集できるようにしました。この機能を有効にするには、**values.yaml** ファイルでDatadog関連のフィールドを設定する必要があります。詳細なユーザーガイドについては、[Datadogとの統合](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-with-datadog.md)を参照してください。[#197](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/197) [#208](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/208)
- **[Operator] ポッドを非ルートユーザーとして実行する**。ポッドを非ルートユーザーとして実行するために、runAsNonRootフィールドを追加しました。これにより、セキュリティを強化することができます。[#195](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/195)
- **[Operator] FEプロキシ**。FEプロキシを追加し、Stream Loadプロトコルをサポートする外部クライアントやデータロードツールがKubernetes内のStarRocksクラスタにアクセスできるようにしました。これにより、Stream Loadに基づくロードジョブを使用してデータをStarRocksクラスタにロードすることができます。[#211](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/211)

**改善**

- StarRocksCluster CRDに `subpath` フィールドを追加しました。[#212](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/212)
- FEメタデータのディスクサイズを増やしました。FEコンテナは、FEメタデータを保存するために割り当てることができる利用可能なディスク容量がデフォルト値よりも少ない場合に停止します。[#210](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/210)
