---
displayed_sidebar: Chinese
---

# Helm を使用して StarRocks クラスターをデプロイする

[Helm](https://helm.sh/) は Kubernetes のパッケージ管理ツールです。[Helm Chart](https://helm.sh/docs/topics/charts/) は Helm のパッケージで、Kubernetes クラスター上でアプリケーションを実行するために必要なすべてのリソース定義を含んでいます。この記事では、Kubernetes クラスター上で Helm を使用して StarRocks クラスターを自動デプロイする方法について説明します。

## 環境準備

- [Kubernetes クラスターを作成する](./sr_operator.md#kubernetes-クラスターを作成する)。
- [Helm をインストールする](https://helm.sh/docs/intro/quickstart/)。

## デプロイ操作

1. StarRocks の Helm Chart リポジトリを追加します。Helm Chart には StarRocks Operator とカスタムリソース StarRocksCluster の定義が含まれています。
    1. Helm Chart リポジトリを追加します。

       ```Bash
       helm repo add starrocks-community https://starrocks.github.io/starrocks-kubernetes-operator
       ```

    2. Helm Chart リポジトリを最新バージョンに更新します。

       ```Bash
       helm repo update
       ```

    3. 追加した Helm Chart リポジトリを確認します。

       ```Bash
       $ helm search repo starrocks-community
       NAME                                     CHART VERSION   APP VERSION   DESCRIPTION
       starrocks-community/kube-starrocks       1.8.0           3.1-latest    kube-starrocks includes two subcharts, starrock...
       starrocks-community/operator             1.8.0           1.8.0         A Helm chart for StarRocks operator
       starrocks-community/starrocks            1.8.0           3.1-latest    A Helm chart for StarRocks cluster
       ```

2. Helm Chart のデフォルト **[values.yaml](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/helm-charts/charts/kube-starrocks/values.yaml)** を使用して StarRocks Operator と StarRocks クラスターをデプロイすることも、新しい YAML ファイルを作成してカスタム設定でデプロイすることもできます。

    1. デフォルト設定を使用してデプロイします。
       以下のコマンドを実行して、StarRocks Operator と StarRocks クラスターをデプロイします。StarRocks クラスターには FE と BE が1つずつ含まれています。

       ```Bash
       $ helm install starrocks starrocks-community/kube-starrocks
       # 以下の結果が返され、StarRocks Operator と StarRocks クラスターがデプロイされていることを示します。
       NAME: starrocks
       LAST DEPLOYED: Tue Aug 15 15:12:00 2023
       NAMESPACE: starrocks
       STATUS: deployed
       REVISION: 1
       TEST SUITE: None
       ```

    2. カスタム設定を使用してデプロイします。
        - **my-values.yaml** などの YAML ファイルを作成し、そのファイル内で StarRocks Operator と StarRocks クラスターの設定情報をカスタマイズします。設定可能なパラメータとその説明については、Helm Chart のデフォルト **[values.yaml](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/helm-charts/charts/kube-starrocks/values.yaml)** のコメントを参照してください。
        - 以下のコマンドを実行して、**my-values.yaml** のカスタム設定を使用して StarRocks Operator と StarRocks クラスターをデプロイします。

          ```Bash
           helm install -f my-values.yaml starrocks starrocks-community/kube-starrocks
          ```

        デプロイには時間がかかります。その間、上記のデプロイコマンドの結果に表示されるプロンプトコマンドに従ってデプロイ状態を確認できます。デフォルトのプロンプトコマンドは以下の通りです：

       ```Bash
       $ kubectl --namespace default get starrockscluster -l "cluster=kube-starrocks"
       # 状態が running と表示されていれば、デプロイに成功しています。
       NAME             FESTATUS   CNSTATUS   BESTATUS
       kube-starrocks   running               running
       ```

       また、`kubectl get pods` を実行してデプロイ状態を確認することもできます。すべての Pod が `Running` 状態で、Pod 内のすべてのコンテナが `READY` であれば、デプロイに成功しています。

       ```Bash
       $ kubectl get pods
       NAME                                       READY   STATUS    RESTARTS   AGE
       kube-starrocks-be-0                        1/1     Running   0          2m50s
       kube-starrocks-fe-0                        1/1     Running   0          4m31s
       kube-starrocks-operator-69c5c64595-pc7fv   1/1     Running   0          4m50s
       ```

## 次のステップ

**StarRocks クラスターにアクセスする**

Kubernetes クラスター内外から StarRocks クラスターにアクセスすることができます。詳細な操作については、[StarRocks クラスターにアクセスする](./sr_operator.md#starrocks-クラスターにアクセスする)を参照してください。

**StarRocks Operator と StarRocks クラスターを管理する**

- StarRocks Operator と StarRocks クラスターの設定を更新する必要がある場合は、[Helm Upgrade](https://helm.sh/docs/helm/helm_upgrade/)を参照してください。
- StarRocks Operator と StarRocks クラスターをアンインストールするには、以下のコマンドを実行します：

    ```Bash
    helm uninstall starrocks
    ```

**Artifact Hub で StarRocks がメンテナンスする Helm Chart を検索する**
[kube-starrocks](https://artifacthub.io/packages/helm/kube-starrocks/kube-starrocks)を参照してください。
