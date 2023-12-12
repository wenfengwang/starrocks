---
displayed_sidebar: "日本語"
---

# starrocks-kubernetes-operator

## 通知

StarRocksが提供するOperatorは、Kubernetes環境でStarRocksクラスタをデプロイするために使用されます。StarRocksクラスタのコンポーネントにはFE、BE、およびCNが含まれます。

**ユーザーガイド:** 以下の方法を使用して、Kubernetes上でStarRocksクラスタをデプロイできます:

- [StarRocks CRDを直接使用してStarRocksクラスタをデプロイ](https://docs.starrocks.io/docs/deployment/sr_operator/)
- [Helmチャートを使用してOperatorとStarRocksクラスタの両方をデプロイ](https://docs.starrocks.io/zh/docs/deployment/helm/)

**ソースコード:**

- [starrocks-kubernetes-operator](https://github.com/StarRocks/starrocks-kubernetes-operator)
- [kube-starrocks Helm Chart](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks)

**リソースのダウンロードURL:**

- **URL prefix**:

    `https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v${operator_version}/${resource_name}`

- **リソース名**

  - StarRocksCluster CRD: `starrocks.com_starrocksclusters.yaml`
  - StarRocks Operatorのデフォルト設定ファイル: `operator.yaml`
  - `kube-starrocks`チャートを含むHelm Chart `kube-starrocks-${chart_version}.tgz`。`kube-starrocks`チャートは、`starrocks`チャート`starrocks-${chart_version}.tgz`と`operator`チャート`operator-${chart_version}.tgz`の2つのサブチャートに分かれています。

たとえば、kube-starrocksチャートv1.8.6のダウンロードURLは次の通りです: [kube-starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v1.8.6/kube-starrocks-1.8.6.tgz)

**バージョン要件**

- Kubernetes: 1.18以降
- Go: 1.19以降

## **リリースノート**

## 1.8

**1.8.6**

**バグ修正**

以下の問題を修正しました:

- ストリームロードジョブ中に`sendfile() failed (32: Broken pipe)`のエラーが返されます。NginxがリクエストボディをFEに送信した後、FEはリクエストをBEにリダイレクトします。この時点で、Nginxにキャッシュされたデータはすでに失われている可能性があります。[#303](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/303)

**ドキュメント**

- [Kubernetesネットワーク外からStream Loadを使用してStarRocksにデータをロードする方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/load_data_using_stream_load_howto.md)
- [Helmを使用してルートユーザーのパスワードを更新する方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/change_root_password_howto.md)

**1.8.5**

**改善**

- **[Helmチャート] `annotations`と`labels`をサービスアカウントのカスタマイズに追加**: Operatorはデフォルトで`starrocks`という名前のサービスアカウントを作成し、ユーザーは`values.yaml`の`serviceAccount`内の`annotations`と`labels`フィールドを指定して、Operatorのサービスアカウント`starrocks`のアノテーションとラベルをカスタマイズできます。`operator.global.rbac.serviceAccountName`フィールドは非推奨です。[#291](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/291)
- **[Operator] FEサービスがIstioのための明示的なプロトコル選択をサポート**: Kubernetes環境にIstioがインストールされている場合、IstioはStarRocksクラスタからのトラフィックのプロトコルを決定する必要があります。これによりルーティングや豊富なメトリクスなどの追加機能が提供されます。そのため、FEサービスは自身のプロトコルを`appProtocol`フィールドで明示的にMySQLとして定義します。この改善は特に重要であり、MySQLプロトコルはサーバーファーストプロトコルであり、自動プロトコル検出と互換性がないため、接続の失敗が発生する可能性があります。[#288](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/288)

**バグ修正**

- **[Helmチャート]** StarRocksで`starrocks.initPassword.enabled`がtrueであり、`starrocks.starrocksCluster.name`の値が指定されていると、StarRocksのルートユーザーのパスワードが初期化されないことがあります。これはinitpwdポッドがFEサービスに接続するために使用する誤ったFEサービスドメイン名によるものです。具体的には、このシナリオでは、FEサービスドメイン名が`starrocks.starrocksCluster.name`で指定された値を使用する一方、initpwdポッドは引き続き`starrocks.nameOverride`フィールドの値を使用してFEサービスドメイン名を形成します。([#292](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/292))

**アップグレードノート**

- **[Helmチャート]** `starrocks.starrocksCluster.name`で指定された値が`starrocks.nameOverride`の値と異なる場合、FE、BE、およびCNの古いconfigmapsが削除されます。新しい名前のconfigmapsがFE、BE、およびCNに作成されます。**これにより、FE、BE、およびCNのポッドが再起動する場合があります。**

**1.8.4**

**機能**

- **[Helmチャート]** StarRocksクラスタのメトリクスをPrometheusとServiceMonitor CRを使用してモニタリングできるようになりました。ユーザーガイドについては、[PrometheusとGrafanaとの統合](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-prometheus-grafana.md)を参照してください。[#284](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/284)
- **[Helmチャート]** **values.yaml**の`starrocksCnSpec`に`storagespec`およびその他のフィールドを追加し、StarRocksクラスタのCNノードのログボリュームを構成できるようになりました。[#280](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/280)
- StarRocksCluster CRDに`terminationGracePeriodSeconds`を追加し、StarRocksClusterリソースが削除または更新されている間、ポッドを強制的に終了するまでの待機時間を構成できるようになりました。[#283](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/283)
- StarRocksCluster CRDに`startupProbeFailureSeconds`フィールドを追加し、StarRocksClusterリソースのポッドの起動プローブ失敗閾値を構成できるようになりました。[#271](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/271)

**バグ修正**

以下の問題を修正しました:

- StarRocksクラスタに複数のFEポッドが存在する場合、FEプロキシはSTREAM LOADリクエストを正しく処理できません。[#269](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/269)

**ドキュメント**

- [ローカルStarRocksクラスタをデプロイするクイックスタートの追加](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/local_installation_how_to.md)。
- 異なる構成でStarRocksクラスタをデプロイする方法に関するユーザーガイドを追加しました。たとえば、すべてのサポートされている機能を持つStarRocksクラスタをデプロイする方法など。その他のユーザーガイドについては、[docs](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)を参照してください。
- StarRocksクラスタを管理する方法に関する追加のユーザーガイドを追加しました。たとえば、[ログと関連フィールドを構成する方法](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/logging_and_related_configurations_howto.md)や、外部のconfigmapやシークレットをマウントする方法など。その他のユーザーガイドについては、[docs](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/doc)を参照してください。

**1.8.3**

**アップグレードノート**

- **[Helmチャート]** デフォルトの**fe.conf**ファイルに`JAVA_OPTS_FOR_JDK_11`を追加しました。デフォルトの**fe.conf**ファイルが使用され、helmチャートがv1.8.3にアップグレードされると、**FEポッドが再起動する可能性**があります。[#257](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/257)

**機能**

- **[Helmチャート]** オペレータが監視する必要のある唯一のネームスペースを指定する`watchNamespace`フィールドを追加しました。それ以外の場合、オペレータはKubernetesクラスタ内のすべてのネームスペースを監視します。ほとんどの場合、この機能を使用する必要はありません。Kubernetesクラスタが多くのノードを管理しており、オペレータがすべてのネームスペースを監視し、多くのメモリリソースを消費している場合にこの機能を使用できます。[#261](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/261)
- **[Helmチャート]** **values.yaml**ファイルの`starrocksFeProxySpec`に`Ports`フィールドを追加して、ユーザーがFEプロキシサービスのNodePortを指定できるようになりました。[#258](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/258)

**改善**

- `proxy_read_timeout`パラメーターの値が**nginx.conf**ファイルで60秒から600秒に変更され、タイムアウトを回避するためです。

**1.8.2**

**改良点**

- オペレーターポッドの許容最大メモリ使用量を増やして、OOMを回避するように改良しました。[#254](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/254)

**1.8.1**

**機能**

- [configMapsおよびsecretsのsubpathフィールドのサポート](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md#3-mount-configmaps-to-a-subpath-by-helm-chart)を追加し、これらのリソースから特定のファイルやディレクトリをマウントすることができます。[#249](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/249)
- StarRocksクラスターCRDに`ports`フィールドを追加し、サービスのポートをユーザーがカスタマイズできるようにしました。[#244](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/244)

**改良点**

- StarRocksクラスターの`BeSpec`または`CnSpec`が削除された場合に関連するKubernetesリソースを削除し、クラスターのクリーンで一貫した状態を確保するようにしました。[#245](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/245)

**1.8.0**

**アップグレードの注意事項と動作の変更**

- **[Operator]** StarRocksCluster CRDおよびオペレーターをアップグレードするには、新しいStarRocksCluster CRD **starrocks.com_starrocksclusters.yaml** と **operator.yaml** を手動で適用する必要があります。

- **[Helm Chart]**

  - Helm Chartをアップグレードするには、以下の手順を実行する必要があります：

    1. 前の**values.yaml**ファイルのフォーマットを新しいフォーマットに調整するために、**values migration tool**を使用します。異なるオペレーティングシステム用のvalues migration toolは[Assets](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/tag/v1.8.0)セクションからダウンロードできます。このツールのヘルプ情報は`migrate-chart-value --help`コマンドを実行することで取得できます。[#206](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/206)

       ```Bash
       migrate-chart-value --input values.yaml --target-version v1.8.0 --output ./values-v1.8.0.yaml
       ```

    2. Helm Chartレポをアップデートします。

       ```Bash
       helm repo update
       ```

    3. `helm upgrade`コマンドを実行して、調整された**values.yaml**ファイルをStarRocks helm chart kube-starrocksに適用します。

       ```Bash
       helm upgrade <release-name> starrocks-community/kube-starrocks -f values-v1.8.0.yaml
       ```

  - [operator](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/operator)および[starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/starrocks)の2つのサブチャートがkube-starrocks helm chartに追加されました。対応するサブチャートを指定することで、StarRocksオペレーターまたはStarRocksクラスターのいずれかをインストールできます。これにより、StarRocksクラスターをより柔軟に管理できます。

**機能**

- **[Helm Chart] Kubernetesクラスター内で複数のStarRocksクラスターのサポート**。 `starrocks` Helmサブチャートをインストールすることで、Kubernetesクラスター内の異なる名前空間で複数のStarRocksクラスターを展開できます。[#199](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/199)
- **[Helm Chart]** `helm install`コマンドの実行時にStarRocksクラスターのルートユーザーの初期パスワードを設定する機能をサポートします。`helm upgrade`コマンドはこの機能をサポートしていないことに注意してください。
- **[Helm Chart] Datadogとの統合:** Datadogと統合してStarRocksクラスターのメトリクスとログを収集するように統合しました。この機能を有効にするには、**values.yaml**ファイルでDatadog関連のフィールドを構成する必要があります。詳しいユーザーガイドについては[Integration with Datadog](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-with-datadog.md)を参照してください。[#197](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/197) [#208](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/208)
- **[Operator] ポッドを非ルートユーザーとして実行**。ポッドを非ルートユーザーとして実行するためにrunAsNonRootフィールドを追加し、セキュリティを強化できるようにしました。[#195](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/195)
- **[Operator]** FEプロキシを追加し、Stream Loadプロトコルをサポートする外部クライアントやデータロードツールがKubernetes内のStarRocksクラスターにアクセスできるようにしました。これにより、Stream Loadに基づくロードジョブを使用して、データをStarRocksクラスターにロードできます。[#211](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/211)

**改良点**

- StarRocksCluster CRDに`subpath`フィールドを追加しました。[#212](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/212)
- FEメタデータのディスクサイズを増やしました。利用可能なディスク容量がデフォルト値よりも少ない場合、FEコンテナは実行を停止します。[#210](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/210)