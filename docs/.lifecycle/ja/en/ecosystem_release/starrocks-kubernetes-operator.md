---
displayed_sidebar: English
---

# starrocks-kubernetes-operator

## 通知

StarRocks が提供する Operator は、Kubernetes 環境で StarRocks クラスターをデプロイするために使用されます。StarRocks クラスターのコンポーネントには FE、BE、および CN が含まれます。

**ユーザーガイド:** 以下の方法を使用して Kubernetes 上に StarRocks クラスターをデプロイできます。

- [StarRocks CRD を直接使用して StarRocks クラスターをデプロイする](https://docs.starrocks.io/docs/deployment/sr_operator/)
- [Helm チャートを使用して Operator と StarRocks クラスターの両方をデプロイする](https://docs.starrocks.io/zh/docs/deployment/helm/)

**ソースコード:**

- [starrocks-kubernetes-operator](https://github.com/StarRocks/starrocks-kubernetes-operator)
- [kube-starrocks Helm チャート](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks)

**リソースのダウンロード URL:**

- **URL プレフィックス**:

    `https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v${operator_version}/${resource_name}`

- **リソース名**

  - StarRocksCluster CRD: `starrocks.com_starrocksclusters.yaml`
  - StarRocks Operator のデフォルト設定ファイル: `operator.yaml`
  - Helm チャート、`kube-starrocks` チャート `kube-starrocks-${chart_version}.tgz` を含む。`kube-starrocks` チャートは `starrocks` チャート `starrocks-${chart_version}.tgz` と `operator` チャート `operator-${chart_version}.tgz` の二つのサブチャートに分かれています。

例えば、kube-starrocks チャート v1.8.6 のダウンロード URL は以下の通りです: [kube-starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v1.8.6/kube-starrocks-1.8.6.tgz)

**バージョン要件**

- Kubernetes: 1.18 以降
- Go: 1.19 以降

## **リリースノート**

## 1.8

**1.8.6**

**バグ修正**

以下の問題を修正しました:

- ストリームロードジョブ中に `sendfile() failed (32: Broken pipe) while sending request to upstream` というエラーが返される問題。Nginx がリクエストボディを FE に送信した後、FE はリクエストを BE にリダイレクトします。この時点で、Nginx にキャッシュされたデータはすでに失われている可能性があります。[#303](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/303)

**ドキュメント**

- [FE プロキシを介して Kubernetes ネットワーク外部から StarRocks にデータをロードする](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/load_data_using_stream_load_howto.md)
- [Helm を使用して root ユーザーのパスワードを更新する方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/change_root_password_howto.md)

**1.8.5**

**改善点**

- **[Helm チャート]** オペレーターのサービスアカウント `starrocks` の `annotations` と `labels` をカスタマイズできます: オペレーターはデフォルトで `starrocks` という名前のサービスアカウントを作成し、ユーザーは **values.yaml** の `serviceAccount` 内の `annotations` と `labels` フィールドを指定することで、オペレーターのサービスアカウント `starrocks` のアノテーションとラベルをカスタマイズできます。`operator.global.rbac.serviceAccountName` フィールドは非推奨です。[#291](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/291)
- **[Operator]** FE サービスは Istio で明示的なプロトコル選択をサポートします: Kubernetes 環境に Istio がインストールされている場合、Istio は StarRocks クラスターからのトラフィックのプロトコルを決定する必要があり、ルーティングや豊かなメトリクスなどの追加機能を提供します。そのため、FE サービスは `appProtocol` フィールドでプロトコルを MySQL として明示的に定義します。MySQL プロトコルはサーバーファーストプロトコルであり、自動プロトコル検出と互換性がないため、時には接続失敗を引き起こす可能性があるため、この改善は特に重要です。[#288](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/288)

**バグ修正**

- **[Helm チャート]** `starrocks.initPassword.enabled` が true で `starrocks.starrocksCluster.name` の値が指定されている場合、StarRocks の root ユーザーのパスワードが正常に初期化されないことがあります。これは、initpwd ポッドが FE サービスに接続するために使用する FE サービスドメイン名が間違っているためです。具体的には、このシナリオでは FE サービスドメイン名は `starrocks.starrocksCluster.name` で指定された値を使用しますが、initpwd ポッドは `starrocks.nameOverride` フィールドの値を使用して FE サービスドメイン名を形成します。([#292](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/292))

**アップグレードノート**

- **[Helm チャート]** `starrocks.starrocksCluster.name` に指定された値が `starrocks.nameOverride` の値と異なる場合、FE、BE、および CN の古い configmaps は削除されます。FE、BE、および CN の新しい名前の新しい configmaps が作成されます。**これにより FE、BE、および CN のポッドが再起動される可能性があります。**

**1.8.4**

**機能**

- **[Helm チャート]** StarRocks クラスタのメトリクスは Prometheus および ServiceMonitor CR を使用して監視できます。ユーザーガイドについては [Prometheus および Grafana との統合](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-prometheus-grafana.md) を参照してください。[#284](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/284)
- **[Helm チャート]** **values.yaml** に `storagespec` および `starrocksCnSpec` の追加フィールドを追加して、StarRocks クラスタ内の CN ノードのログボリュームを設定できます。[#280](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/280)
- StarRocksCluster CRD に `terminationGracePeriodSeconds` を追加して、StarRocksCluster リソースが削除または更新されているときにポッドを強制終了するまでの待機時間を設定します。[#283](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/283)
- StarRocksCluster CRD に `startupProbeFailureSeconds` フィールドを追加して、StarRocksCluster リソース内のポッドの起動プローブの失敗閾値を設定します。[#271](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/271)

**バグ修正**

以下の問題を修正しました:

- StarRocks クラスタに複数の FE ポッドが存在する場合、FE プロキシが STREAM LOAD リクエストを正しく処理できない問題。[#269](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/269)

**ドキュメント**

- [ローカル StarRocks クラスタをデプロイする方法に関するクイックスタートを追加する](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/local_installation_how_to.md)。
- さまざまな構成で StarRocks クラスタをデプロイする方法に関するユーザーガイドを追加します。例えば、[すべてのサポートされる機能を備えた StarRocks クラスタをデプロイする方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_all_features.yaml) などです。その他のユーザーガイドについては [ドキュメント](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks) を参照してください。

- StarRocksクラスタの管理方法に関するユーザーガイドを追加します。たとえば、[ロギングと関連フィールドの設定方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/logging_and_related_configurations_howto.md)や[外部のConfigMapsやSecretsをマウントする方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md)などです。その他のユーザーガイドについては、[ドキュメント](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/doc)を参照してください。

**1.8.3**

**アップグレードノート**

- **[Helm Chart]** デフォルトの **fe.conf** ファイルに `JAVA_OPTS_FOR_JDK_11` を追加します。デフォルトの **fe.conf** ファイルを使用してHelmチャートをv1.8.3にアップグレードする場合、**FE Podsが再起動する可能性があります**。 [#257](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/257)

**機能**

- **[Helm Chart]** `watchNamespace` フィールドを追加し、オペレーターが監視する必要がある唯一の名前空間を指定します。それ以外の場合、オペレーターはKubernetesクラスタ内のすべての名前空間を監視します。ほとんどの場合、この機能を使用する必要はありません。Kubernetesクラスタが多数のノードを管理しており、オペレーターがすべての名前空間を監視してメモリリソースを過剰に消費する場合にこの機能を使用できます。[#261](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/261)
- **[Helm Chart]** `values.yaml` ファイルの `starrocksFeProxySpec` に `Ports` フィールドを追加し、ユーザーがFEプロキシサービスのNodePortを指定できるようにします。 [#258](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/258)

**改善点**

- タイムアウトを回避するために、**nginx.conf** ファイルの `proxy_read_timeout` パラメータの値を60秒から600秒に変更しました。

**1.8.2**

**改善点**

- OOMを回避するために、オペレーターPodの許可される最大メモリ使用量を増加させました。 [#254](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/254)

**1.8.1**

**機能**

- [ConfigMapsとSecretsのサブパスフィールドの使用をサポート](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md#3-mount-configmaps-to-a-subpath-by-helm-chart)し、ユーザーがこれらのリソースから特定のファイルやディレクトリをマウントできるようにします。 [#249](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/249)
- StarRocksクラスタCRDに `ports` フィールドを追加し、ユーザーがサービスのポートをカスタマイズできるようにします。 [#244](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/244)

**改善点**

- StarRocksクラスタの `BeSpec` または `CnSpec` が削除された場合、関連するKubernetesリソースも削除し、クラスタの状態をクリーンで一貫性のあるものに保ちます。 [#245](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/245)

**1.8.0**

**アップグレードノートと挙動の変更**

- **[Operator]** StarRocksCluster CRDとオペレーターをアップグレードするには、新しいStarRocksCluster CRD **starrocks.com_starrocksclusters.yaml** と **operator.yaml** を手動で適用する必要があります。

- **[Helm Chart]**

  - Helmチャートをアップグレードするには、以下の手順を実行する必要があります：

    1. **values migration tool** を使用して、以前の **values.yaml** ファイルの形式を新しい形式に調整します。異なるオペレーティングシステム用のvalues migration toolは、[Assets](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/tag/v1.8.0)セクションからダウンロードできます。このツールのヘルプ情報は、`migrate-chart-value --help` コマンドを実行することで取得できます。 [#206](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/206)

       ```Bash
       migrate-chart-value --input values.yaml --target-version v1.8.0 --output ./values-v1.8.0.yaml
       ```

    2. Helmリポジトリを更新します。

       ```Bash
       helm repo update
       ```

    3. `helm upgrade` コマンドを実行して、調整された **values.yaml** ファイルをStarRocks Helmチャートkube-starrocksに適用します。

       ```Bash
       helm upgrade <release-name> starrocks-community/kube-starrocks -f values-v1.8.0.yaml
       ```

  - kube-starrocks Helmチャートに[operator](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/operator)と[starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/starrocks)の2つのサブチャートが追加されました。対応するサブチャートを指定することで、StarRocksオペレーターまたはStarRocksクラスターをそれぞれインストールすることができます。これにより、1つのStarRocksオペレーターと複数のStarRocksクラスターをデプロイするなど、StarRocksクラスターをより柔軟に管理できます。

**機能**

- **[Helm Chart]** Kubernetesクラスタ内で複数のStarRocksクラスタを異なる名前空間にデプロイする機能をサポートします。`starrocks` Helmサブチャートをインストールすることで実現可能です。[#199](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/199)
- **[Helm Chart]** `helm install` コマンド実行時にStarRocksクラスタのrootユーザーの初期パスワードを設定する機能をサポートします。ただし、`helm upgrade` コマンドはこの機能をサポートしていませんのでご注意ください。
- **[Helm Chart] Datadogとの統合:** Datadogと統合してStarRocksクラスタのメトリクスとログを収集します。この機能を有効にするには、**values.yaml** ファイルでDatadog関連のフィールドを設定する必要があります。詳細なユーザーガイドについては、[Datadogとの統合](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-with-datadog.md)を参照してください。[#197](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/197) [#208](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/208)
- **[Operator]** ポッドを非rootユーザーとして実行するための `runAsNonRoot` フィールドを追加し、セキュリティを強化します。[#195](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/195)
- **[Operator]** FEプロキシを追加し、Stream Loadプロトコルをサポートする外部クライアントやデータロードツールがKubernetes内のStarRocksクラスタにアクセスできるようにします。これにより、Stream Loadベースのロードジョブを使用してKubernetes内のStarRocksクラスタにデータをロードできます。[#211](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/211)

**改善点**

- StarRocksCluster CRDに `subpath` フィールドを追加しました。[#212](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/212)
- FEメタデータ用の許容ディスクサイズを増加させます。利用可能なディスクスペースがFEメタデータを保存するために割り当てられるデフォルト値未満になると、FEコンテナは実行を停止します。[#210](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/210)
