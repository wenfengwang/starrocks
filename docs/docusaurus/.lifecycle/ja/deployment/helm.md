---
displayed_sidebar: "Japanese"
---

# Helmを使用してStarRocksをデプロイする

[Helm](https://helm.sh/)はKubernetes向けのパッケージマネージャーです。[Helm Chart](https://helm.sh/docs/topics/charts/)はHelmパッケージであり、Kubernetesクラスター上でアプリケーションを実行するために必要なリソース定義をすべて含んでいます。このトピックでは、Helmを使用してKubernetesクラスター上にStarRocksクラスターを自動的にデプロイする方法について説明します。

## 開始する前に

- [Kubernetesクラスターを作成](./sr_operator.md#create-kubernetes-cluster)します。
- [Helmをインストール](https://helm.sh/docs/intro/quickstart/)します。

## 手順

1. StarRocksのHelm Chart Repoを追加します。Helm ChartにはStarRocks OperatorとカスタムリソースStarRocksClusterの定義が含まれています。
   1. Helm Chart Repoを追加します。

      ```Bash
      helm repo add starrocks-community https://starrocks.github.io/starrocks-kubernetes-operator
      ```

   2. Helm Chart Repoを最新バージョンに更新します。

      ```Bash
      helm repo update
      ```

   3. 追加したHelm Chart Repoを表示します。

      ```Bash
      $ helm search repo starrocks-community
      NAME                                    CHART VERSION    APP VERSION  DESCRIPTION
      starrocks-community/kube-starrocks      1.8.0            3.1-latest   kube-starrocks includes two subcharts, starrock...
      starrocks-community/operator            1.8.0            1.8.0        A Helm chart for StarRocks operator
      starrocks-community/starrocks           1.8.0            3.1-latest   A Helm chart for StarRocks cluster
      ```

2. Helm Chartのデフォルトの **[values.yaml](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/helm-charts/charts/kube-starrocks/values.yaml)** を使用してStarRocks OperatorとStarRocksクラスターをデプロイするか、デプロイ構成をカスタマイズするためのYAMLファイルを作成します。
   1. デフォルトの構成でデプロイ

      以下のコマンドを実行して、1つのFEと1つのBEで構成されるStarRocks OperatorとStarRocksクラスターをデプロイします。

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

   2. カスタム構成でデプロイ
      - 例えば **my-values.yaml** というYAMLファイルを作成し、その中でStarRocks OperatorとStarRocksクラスターの構成をカスタマイズします。サポートされているパラメータとその説明については、Helm Chartのデフォルトの **[values.yaml](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/helm-charts/charts/kube-starrocks/values.yaml)** 内のコメントを参照してください。
      - 以下のコマンドを実行して、 **my-values.yaml** 中のカスタム構成でStarRocks OperatorとStarRocksクラスターをデプロイします。

        ```Bash
        helm install -f my-values.yaml starrocks starrocks-community/kube-starrocks
        ```

    デプロイには時間がかかります。この期間中は、デプロイコマンドの返された結果でプロンプトコマンドを使用してデプロイの状態を確認できます。デフォルトのプロンプトコマンドは次の通りです。

    ```Bash
    $ kubectl --namespace default get starrockscluster -l "cluster=kube-starrocks"
    # 以下の結果が返された場合、デプロイが完了しています。
    NAME             FESTATUS   CNSTATUS   BESTATUS
    kube-starrocks   running               running
    ```

    デプロイ状態を確認するには `kubectl get pods` を実行することもできます。すべてのPodが `Running` の状態であり、またPod内のすべてのコンテナが `READY` であれば、デプロイは正常に完了しています。

    ```Bash
    $ kubectl get pods
    NAME                                       READY   STATUS    RESTARTS   AGE
    kube-starrocks-be-0                        1/1     Running   0          2m50s
    kube-starrocks-fe-0                        1/1     Running   0          4m31s
    kube-starrocks-operator-69c5c64595-pc7fv   1/1     Running   0          4m50s
    ```

## 次のステップ

- StarRocksクラスターへのアクセス

  StarRocksクラスターには、Kubernetesクラスター内外からアクセスできます。詳細な手順については、[StarRocksクラスターへのアクセス](./sr_operator.md#access-starrocks-cluster)を参照してください。

- StarRocks OperatorとStarRocksクラスターの管理

  - StarRocks OperatorとStarRocksクラスターの構成を更新する必要がある場合は、[Helmアップグレード](https://helm.sh/docs/helm/helm_upgrade/)を参照してください。
  - StarRocks OperatorとStarRocksクラスターをアンインストールする必要がある場合は、次のコマンドを実行してください。

    ```bash
    helm uninstall starrocks
    ```

- StarRocksが管理するHelm ChartをArtifact Hubで検索

  [kube-starrocks](https://artifacthub.io/packages/helm/kube-starrocks/kube-starrocks)を参照してください。