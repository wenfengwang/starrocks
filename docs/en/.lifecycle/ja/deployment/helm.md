---
displayed_sidebar: "Japanese"
---

# Helmを使用してStarRocksをデプロイする

[Helm](https://helm.sh/)はKubernetesのパッケージマネージャです。[Helm Chart](https://helm.sh/docs/topics/charts/)はHelmパッケージであり、Kubernetesクラスタ上でアプリケーションを実行するために必要なすべてのリソース定義を含んでいます。このトピックでは、Helmを使用してKubernetesクラスタ上にStarRocksクラスタを自動的にデプロイする方法について説明します。

## 開始する前に

- [Kubernetesクラスタを作成する](./sr_operator.md#create-kubernetes-cluster)。
- [Helmをインストールする](https://helm.sh/docs/intro/quickstart/)。

## 手順

1. StarRocksのためのHelm Chartリポジトリを追加します。Helm ChartにはStarRocks OperatorとカスタムリソースStarRocksClusterの定義が含まれています。
   1. Helm Chartリポジトリを追加します。

      ```Bash
      helm repo add starrocks-community https://starrocks.github.io/starrocks-kubernetes-operator
      ```

   2. Helm Chartリポジトリを最新バージョンに更新します。

      ```Bash
      helm repo update
      ```

   3. 追加したHelm Chartリポジトリを表示します。

      ```Bash
      $ helm search repo starrocks-community
      NAME                                    CHART VERSION    APP VERSION  DESCRIPTION
      starrocks-community/kube-starrocks      1.8.0            3.1-latest   kube-starrocks includes two subcharts, starrock...
      starrocks-community/operator            1.8.0            1.8.0        A Helm chart for StarRocks operator
      starrocks-community/starrocks           1.8.0            3.1-latest   A Helm chart for StarRocks cluster
      ```

2. Helm Chartのデフォルトの **[values.yaml](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/helm-charts/charts/kube-starrocks/values.yaml)** を使用してStarRocks OperatorとStarRocksクラスタをデプロイするか、デプロイの設定をカスタマイズするためのYAMLファイルを作成します。
   1. デフォルトの設定でデプロイする

      以下のコマンドを実行して、StarRocks OperatorとFE 1つ、BE 1つからなるStarRocksクラスタをデプロイします。

      ```Bash
      $ helm install starrocks starrocks-community/kube-starrocks
      # 以下の結果が返された場合、StarRocks OperatorとStarRocksクラスタがデプロイされています。
      NAME: starrocks
      LAST DEPLOYED: Tue Aug 15 15:12:00 2023
      NAMESPACE: starrocks
      STATUS: deployed
      REVISION: 1
      TEST SUITE: None
      ```

   2. カスタム設定でデプロイする
      - **my-values.yaml** という名前のYAMLファイルを作成し、YAMLファイル内でStarRocks OperatorとStarRocksクラスタの設定をカスタマイズします。サポートされているパラメータと説明については、Helm Chartのデフォルトの **[values.yaml](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/helm-charts/charts/kube-starrocks/values.yaml)** のコメントを参照してください。
      - 以下のコマンドを実行して、**my-values.yaml** のカスタム設定でStarRocks OperatorとStarRocksクラスタをデプロイします。

        ```Bash
        helm install -f my-values.yaml starrocks starrocks-community/kube-starrocks
        ```

    デプロイには時間がかかります。この期間中、デプロイコマンドの結果で返されたプロンプトコマンドを使用してデプロイの状態を確認することができます。デフォルトのプロンプトコマンドは次のようになります。

    ```Bash
    $ kubectl --namespace default get starrockscluster -l "cluster=kube-starrocks"
    # 以下の結果が返された場合、デプロイが正常に完了しています。
    NAME             FESTATUS   CNSTATUS   BESTATUS
    kube-starrocks   running               running
    ```

    デプロイの状態を確認するには、`kubectl get pods` を実行することもできます。すべてのPodが `Running` 状態であり、Pod内のすべてのコンテナが `READY` 状態であれば、デプロイは正常に完了しています。

    ```Bash
    $ kubectl get pods
    NAME                                       READY   STATUS    RESTARTS   AGE
    kube-starrocks-be-0                        1/1     Running   0          2m50s
    kube-starrocks-fe-0                        1/1     Running   0          4m31s
    kube-starrocks-operator-69c5c64595-pc7fv   1/1     Running   0          4m50s
    ```

## 次の手順

- StarRocksクラスタへのアクセス

  StarRocksクラスタには、Kubernetesクラスタ内および外部からアクセスすることができます。詳細な手順については、[StarRocksクラスタへのアクセス](./sr_operator.md#access-starrocks-cluster)を参照してください。

- StarRocks OperatorとStarRocksクラスタの管理

  - StarRocks OperatorとStarRocksクラスタの設定を更新する必要がある場合は、[Helm Upgrade](https://helm.sh/docs/helm/helm_upgrade/)を参照してください。
  - StarRocks OperatorとStarRocksクラスタをアンインストールする必要がある場合は、次のコマンドを実行します。

    ```bash
    helm uninstall starrocks
    ```

- StarRocksがメンテナンスしているHelm ChartをArtifact Hubで検索する

  [kube-starrocks](https://artifacthub.io/packages/helm/kube-starrocks/kube-starrocks)を参照してください。
