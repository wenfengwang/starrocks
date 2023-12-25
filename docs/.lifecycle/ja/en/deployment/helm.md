---
displayed_sidebar: English
---

# Helmを使用したStarRocksのデプロイ

[Helm](https://helm.sh/)はKubernetesのパッケージマネージャーです。[Helmチャート](https://helm.sh/docs/topics/charts/)はHelmパッケージであり、Kubernetesクラスターでアプリケーションを実行するために必要なすべてのリソース定義が含まれています。このトピックでは、Helmを使用してStarRocksクラスターをKubernetesクラスターに自動的にデプロイする方法について説明します。

## 始める前に

- [Kubernetesクラスターを作成する](./sr_operator.md#create-kubernetes-cluster)。
- [Helmをインストールする](https://helm.sh/docs/intro/quickstart/)。

## 手順

1. StarRocksのHelmチャートリポジトリを追加します。Helmチャートには、StarRocks OperatorとカスタムリソースStarRocksClusterの定義が含まれています。
   1. Helmチャートリポジトリを追加します。

      ```Bash
      helm repo add starrocks-community https://starrocks.github.io/starrocks-kubernetes-operator
      ```

   2. Helmチャートリポジトリを最新バージョンに更新します。

      ```Bash
      helm repo update
      ```

   3. 追加したHelmチャートリポジトリを表示します。

      ```Bash
      $ helm search repo starrocks-community
      NAME                                    CHART VERSION    APP VERSION  DESCRIPTION
      starrocks-community/kube-starrocks      1.8.0            3.1-latest   kube-starrocks includes two subcharts, starrock...
      starrocks-community/operator            1.8.0            1.8.0        A Helm chart for StarRocks operator
      starrocks-community/starrocks           1.8.0            3.1-latest   A Helm chart for StarRocks cluster
      ```

2. Helmチャートのデフォルト **[values.yaml](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/helm-charts/charts/kube-starrocks/values.yaml)** を使用してStarRocks OperatorおよびStarRocksクラスターをデプロイするか、YAMLファイルを作成してデプロイ設定をカスタマイズします。
   1. 既定の構成でのデプロイ

      次のコマンドを実行して、StarRocks Operatorと1つのFEと1つのBEで構成されるStarRocksクラスターをデプロイします。

      ```Bash
      $ helm install starrocks starrocks-community/kube-starrocks
      # 以下の結果が返された場合、StarRocks OperatorとStarRocksクラスターがデプロイされています。
      NAME: starrocks
      LAST DEPLOYED: Tue Aug 15 15:12:00 2023
      NAMESPACE: starrocks
      STATUS: deployed
      REVISION: 1
      TEST SUITE: None
      ```

   2. カスタム構成でのデプロイ
      - YAMLファイル（例：**my-values.yaml**）を作成し、YAMLファイルでStarRocks OperatorおよびStarRocksクラスターの設定をカスタマイズします。サポートされているパラメータと説明については、Helmチャートのデフォルト **[values.yaml](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/helm-charts/charts/kube-starrocks/values.yaml)** のコメントを参照してください。
      - 次のコマンドを実行して、**my-values.yaml**のカスタム設定を使用してStarRocks OperatorとStarRocksクラスターをデプロイします。

        ```Bash
        helm install -f my-values.yaml starrocks starrocks-community/kube-starrocks
        ```

    デプロイには時間がかかります。この期間中、上記のデプロイコマンドの結果で表示されるプロンプトコマンドを使用してデプロイ状態を確認できます。デフォルトのプロンプトコマンドは次のとおりです。

    ```Bash
    $ kubectl --namespace default get starrockscluster -l "cluster=kube-starrocks"
    # 以下の結果が返された場合、デプロイが正常に完了しています。
    NAME             FESTATUS   CNSTATUS   BESTATUS
    kube-starrocks   running               running
    ```

    また、`kubectl get pods`を実行してデプロイ状態を確認することもできます。すべてのPodが`Running`状態で、Pod内のすべてのコンテナが`READY`であれば、デプロイは正常に完了しています。

    ```Bash
    $ kubectl get pods
    NAME                                       READY   STATUS    RESTARTS   AGE
    kube-starrocks-be-0                        1/1     Running   0          2m50s
    kube-starrocks-fe-0                        1/1     Running   0          4m31s
    kube-starrocks-operator-69c5c64595-pc7fv   1/1     Running   0          4m50s
    ```

## 次のステップ

- StarRocksクラスタへのアクセス

  Kubernetesクラスタの内部および外部からStarRocksクラスタにアクセスできます。詳細な手順については、[StarRocksクラスタへのアクセス](./sr_operator.md#access-starrocks-cluster)を参照してください。

- StarRocksオペレータとStarRocksクラスタの管理

  - StarRocksオペレータとStarRocksクラスタの設定を更新する必要がある場合は、[Helm Upgrade](https://helm.sh/docs/helm/helm_upgrade/)を参照してください。
  - StarRocksオペレータとStarRocksクラスタをアンインストールするには、次のコマンドを実行します。

    ```bash
    helm uninstall starrocks
    ```

- Artifact HubでStarRocksが管理するHelmチャートを検索

  [kube-starrocks](https://artifacthub.io/packages/helm/kube-starrocks/kube-starrocks)を参照してください。
