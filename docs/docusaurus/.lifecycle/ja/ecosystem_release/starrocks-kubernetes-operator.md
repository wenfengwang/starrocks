---
displayed_sidebar: "Japanese"
---

# starrocks-kubernetes-operator

## 通知

StarRocksによって提供されるOperatorは、Kubernetes環境でStarRocksクラスタをデプロイするために使用されます。StarRocksクラスタのコンポーネントにはFE、BE、CNが含まれます。

**ユーザーガイド:** 以下の方法を使用して、Kubernetes上でStarRocksクラスタをデプロイできます:

- [StarRocks CRDを直接使用してStarRocksクラスタをデプロイ](https://docs.starrocks.io/docs/deployment/sr_operator/)
- [Helm Chartを使用してOperatorとStarRocksクラスタをデプロイ](https://docs.starrocks.io/zh/docs/deployment/helm/)

**ソースコード:**

- [starrocks-kubernetes-operator](https://github.com/StarRocks/starrocks-kubernetes-operator)
- [kube-starrocks Helm Chart](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks)

**リソースのダウンロードURL:**

- **URL プレフィックス**:

    `https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v${operator_version}/${resource_name}`

- **リソース名**

  - StarRocksCluster CRD: `starrocks.com_starrocksclusters.yaml`
  - StarRocks Operatorのデフォルト構成ファイル: `operator.yaml`
  - `kube-starrocks` Chartを含むHelm Chart `kube-starrocks-${chart_version}.tgz`。`kube-starrocks` Chartは2つのサブチャートに分けられています: `starrocks` Chart `starrocks-${chart_version}.tgz` および `operator` Chart `operator-${chart_version}.tgz`。

例えば、kube-starrocks chart v1.8.6のダウンロードURLは次の通りです: [kube-starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v1.8.6/kube-starrocks-1.8.6.tgz)

**バージョン要件**

- Kubernetes: 1.18 以降
- Go: 1.19 以降

## **リリースノート**

## 1.8

**1.8.6**

**バグ修正**

以下の問題を修正しました:

- ストリームロードジョブ中に `sendfile()が失敗しました (32: Broken pipe)` のエラーが返されます。NginxがリクエストボディをFEに送信した後、FEはリクエストをBEにリダイレクトします。この時点で、Nginxにキャッシュされたデータはすでに失われている可能性があります。 [#303](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/303)

**ドキュメント**

- [Kubernetesネットワーク外からFEプロキシを介してStarRocksへデータをロードする方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/load_data_using_stream_load_howto.md)
- [Helmを使用してルートユーザーのパスワードを更新する方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/change_root_password_howto.md)

**1.8.5**

**改善**

- **[Helm Chart] `annotations`および`labels`をサービスアカウントのカスタマイズに追加**：Operatorはデフォルトで`starrocks`という名前のサービスアカウントを作成し、ユーザーは`values.yaml`の`serviceAccount`内の`annotations`および`labels`フィールドを指定することで、Operatorのサービスアカウント`starrocks`のアノテーションとラベルをカスタマイズできます。`operator.globa.rbac.serviceAccountName`フィールドは非推奨です。 [#291](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/291)
- **[Operator] FEサービスはIstioのための明示的なプロトコル選択をサポート**：Kubernetes環境にIstioがインストールされている場合、IstioはStarRocksクラスタからのトラフィックのプロトコルを決定する必要があります。これにより、ルーティングや豊富なメトリクスなどの追加機能が提供されます。そのため、FEサービスはトラフィックのプロトコルを`appProtocol`フィールドで明示的にMySQLとして定義します。この改善は特に重要で、MySQLプロトコルはサーバー側が最初に通信を行うプロトコルであり、自動プロトコルの検出と互換性がなく、接続の失敗が発生することがあります。 [#288](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/288)

**バグ修正**

- **[Helm Chart]** StarRocksのルートユーザーのパスワードは、`starrocks.initPassword.enabled`がtrueであり、`starrocks.starrocksCluster.name`の値が指定された場合に初期化されない場合があります。これは、initpwdポッドがFEサービスに接続するために使用するFEサービスドメイン名が誤っているために起こります。具体的には、このシナリオでは、FEサービスドメイン名が`starrocks.starrocksCluster.name`で指定された値を使用し、initpwdポッドが依然としてFEサービスドメイン名を形成するために`starrocks.nameOverride`フィールドの値を使用します。([#292](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/292))

**アップグレード情報**

- **[Helm Chart]** `starrocks.starrocksCluster.name`で指定された値が`starrocks.nameOverride`の値と異なる場合、FE、BE、CNの古いconfigmapsが削除されます。新しいconfigmapsが新しい名前でFE、BE、CNのために作成されます。**これはFE、BE、CNのポッドの再起動を引き起こす可能性があります。**

**1.8.4**

**機能**

- **[Helm Chart]** StarRocksクラスタのメトリックは、PrometheusおよびServiceMonitor CRを使用して監視できます。ユーザーガイドについては、[PrometheusとGrafanaとの統合](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-prometheus-grafana.md)を参照してください。 [#284](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/284)
- **[Helm Chart]** **`values.yaml`での`starrocksCnSpec`内の`storagespec`とその他のフィールドを追加**して、StarRocksクラスタのCNノードのログボリュームを構成します。 [#280](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/280)
- StarRocksCluster CRDに`terminationGracePeriodSeconds`を追加し、StarRocksClusterリソースが削除または更新されている際にポッドを強制的に終了するまでの待機時間を構成します。[#283](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/283)
- StarRocksCluster CRDの中でポッドのスタートアッププローブの失敗閾値を構成するための`startupProbeFailureSeconds`フィールドを追加します。[#271](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/271)

**バグ修正**

以下の問題を修正しました:

- StarRocksクラスタに複数のFEポッドが存在する場合、FEプロキシはSTREAM LOADリクエストを正しく処理できません。 [#269](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/269)

**ドキュメント**

- ローカルでStarRocksクラスタをデプロイする方法についてのクイックスタートを追加します。](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/local_installation_how_to.md)。
- 異なる構成でStarRocksクラスタをデプロイする方法に関するユーザーガイドを追加します。例えば、すべてのサポートされる機能を使用してStarRocksクラスタをデプロイする方法については [スターロックスクラスタのすべての機能を使用してスターロックスクラスタをデプロイする](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_all_features.yaml) を参照してください。その他のユーザーガイドについては、[docs](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)を参照してください。
- StarRocksクラスタを管理する方法に関する追加のユーザーガイドを追加します。例えば、[logging and related fields](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/logging_and_related_configurations_howto.md)および[mount external configmaps or secrets](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md)を構成します。その他のユーザーガイドについては、[docs](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/doc)を参照してください。

**1.8.3**

**アップグレード情報**

- **[Helm Chart]** デフォルトの **fe.conf** ファイルに `JAVA_OPTS_FOR_JDK_11` を追加します。デフォルトの **fe.conf** ファイルを使用し、helm chartをv1.8.3にアップグレードした場合、**FEポッドが再起動する可能性があります。**。[#257](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/257)

**機能**

- **[Helm Chart]** 演算子が監視する必要のある唯一の名前空間を指定するための `watchNamespace` フィールドを追加します。そうしない場合、演算子はKubernetesクラスタ内のすべての名前空間を監視します。ほとんどの場合、この機能は必要ありません。Kubernetesクラスタが多くのノードを管理し、演算子がすべての名前空間を監視し、多くのメモリリソースを消費する場合にこの機能を使用することができます。[#261](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/261)
- **[Helm Chart]** **`values.yaml`ファイルの`starrocksFeProxySpec`に`Ports`フィールド**を追加して、ユーザーがFE ProxyサービスのNodePortを指定できるようにします。[#258](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/258)

**改善**

- `nginx.conf`ファイルで`proxy_read_timeout`パラメータの値を60秒から600秒に変更して、タイムアウトを回避します。

**1.8.2**

**改善**

- オペレーターポッドの最大メモリ使用量を増やしてOOMを回避します。[#254](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/254)

**1.8.1**

**機能**

- [configMapsとsecretsのsubpathフィールド](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md#3-mount-configmaps-to-a-subpath-by-helm-chart)を使用して、これらのリソースから特定のファイルやディレクトリをマウントできるようにサポートします。[#249](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/249)
- StarRocksクラスターCRDに`ports`フィールドを追加して、サービスのポートをカスタマイズできるようにします。[#244](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/244)

**改善**

- StarRocksクラスターの`BeSpec`または`CnSpec`が削除された時に関連するKubernetesリソースを削除し、クラスターのクリーンで一貫した状態を確保します。[#245](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/245)

**1.8.0**

**アップグレードの注意事項と動作の変更**

- **[Operator]** StarRocksCluster CRDおよびオペレーターをアップグレードするには、新しいStarRocksCluster CRD **starrocks.com_starrocksclusters.yaml**および**operator.yaml**を手動で適用する必要があります。

- **[Helm Chart]**

  - Helm Chartをアップグレードするには、以下の手順を実行する必要があります：

    1. 前の**values.yaml**ファイルの形式を新しい形式に調整するために**values migration tool**を使用します。異なるオペレーティングシステム向けのvalues migration toolは[Asset](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/tag/v1.8.0)セクションからダウンロードできます。このツールのヘルプ情報は`migrate-chart-value --help`コマンドを実行して取得できます。[#206](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/206)

       ```Bash
       migrate-chart-value --input values.yaml --target-version v1.8.0 --output ./values-v1.8.0.yaml
       ```

    2. Helm Chartリポジトリを更新します。

       ```Bash
       helm repo update
       ```

    3. 調整された**values.yaml**ファイルをStarRocks helm chart kube-starrocksに適用するために`helm upgrade`コマンドを実行します。

       ```Bash
       helm upgrade <リリース名> starrocks-community/kube-starrocks -f values-v1.8.0.yaml
       ```

  - 2つのサブチャート、[operator](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/operator)および[starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/starrocks)がkube-starrocks helm chartに追加されました。対応するサブチャートを指定してStarRocksクラスターまたはStarRocksオペレーターをそれぞれインストールすることで、柔軟にStarRocksクラスターを管理できます。

**機能**

- **[Helm Chart] Kubernetesクラスター内の複数のStarRocksクラスター**。`starrocks` Helmサブチャートをインストールすることで、Kubernetesクラスター内の異なる名前空間に複数のStarRocksクラスターを展開できるようにサポートします。[#199](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/199)
- **[Helm Chart]** `helm install`コマンドを実行する際にStarRocksクラスターのルートユーザーの初期パスワードを[設定する機能](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/initialize_root_password_howto.md)をサポートします。`helm upgrade`コマンドはこの機能をサポートしていません。
- **[Helm Chart] Datadogとのインテグレーション**：Datadogと統合してStarRocksクラスターのメトリクスとログを収集します。この機能を有効にするには、**values.yaml**ファイルでDatadog関連のフィールドを構成する必要があります。詳しいユーザーガイドについては[Integration with Datadog](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-with-datadog.md)を参照してください。[#197](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/197) [#208](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/208)
- **[Operator] ポッドを非rootユーザーとして実行**。ポッドが非rootユーザーとして実行されるように`runAsNonRoot`フィールドを追加し、セキュリティを強化できるようにします。[#195](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/195)
- **[Operator] FEプロキシ**。StarRocksクラスター内の外部クライアントとStream LoadプロトコルをサポートするデータロードツールがStarRocksクラスターにアクセスできるようにFEプロキシを追加します。これにより、Kubernetes内のStarRocksクラスターにデータをロードするためのStream Loadに基づくロードジョブを使用できます。[#211](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/211)

**改善**

- StarRocksCluster CRDに`subpath`フィールドを追加します。[#212](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/212)
- FEメタデータのディスクサイズを増やします。FEメタデータを保存するためにプロビジョニングできる利用可能なディスク容量がデフォルト値よりも少ない場合、FEコンテナが停止します。[#210](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/210)