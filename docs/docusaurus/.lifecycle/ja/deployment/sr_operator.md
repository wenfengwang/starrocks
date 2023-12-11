```yaml
---
displayed_sidebar: "Japanese"
---

# オペレーターを使用してStarRocksをデプロイする

このトピックでは、StarRocksオペレーターを使用して、Kubernetesクラスター上にStarRocksクラスターを自動的にデプロイおよび管理する方法について紹介します。

## 仕組み

![img](../assets/starrocks_operator.png)

## はじめる前に

### Kubernetesクラスターの作成

[Amazon Elastic Kubernetes Service (EKS)](https://aws.amazon.com/eks/?nc1=h_ls) や [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) のようなクラウド管理型Kubernetesサービス、または自己管理型のKubernetesクラスターを使用できます。

- Amazon EKSクラスターの作成

  1. [環境に以下のコマンドラインツールがインストールされていること](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)を確認してください：
     1. AWSのコマンドラインツールAWS CLIのインストールと構成。
     2. EKSクラスターコマンドラインツールeksctlのインストール。
     3. Kubernetesクラスターコマンドラインツールkubectlのインストール。
  2. 以下のいずれかの方法を使用してEKSクラスターを作成します：
     1. [eksctlを使用して素早くEKSクラスターを作成](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)します。
     2. [AWSコンソールとAWS CLIを使用して手動でEKSクラスターを作成](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)します。

- GKEクラスターの作成

  GKEクラスターを作成する前に、[前提条件](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#before-you-begin)をすべて完了してください。その後、[GKEクラスターの作成](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#create_cluster)に記載されている手順に従ってGKEクラスターを作成します。

- 自己管理型のKubernetesクラスターの作成

  [kubeadmを使用したクラスターのブートストラップ](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)に記載されている手順に従って、自己管理型のKubernetesクラスターを作成します。[Minikube](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/) や [Docker Desktop](https://docs.docker.com/desktop/) を使用して、最小限の手順で単一ノードのプライベートKubernetesクラスターを作成することができます。

### StarRocksオペレーターのデプロイ

1. カスタムリソース StarRocksCluster を追加します。

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/starrocks.com_starrocksclusters.yaml
   ```

2. StarRocksオペレーターをデプロイします。デフォルトの構成ファイルまたはカスタム構成ファイルを使用して、StarRocksオペレーターをデプロイすることができます。
   1. デフォルト構成ファイルを使用してStarRocksオペレーターをデプロイします。

      ```bash
      kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
      ```

      StarRocksオペレーターは名前空間 `starrocks` にデプロイされ、すべての名前空間のStarRocksクラスターを管理します。
   2. カスタム構成ファイルを使用してStarRocksオペレーターをデプロイします。
      - StarRocksオペレーターをデプロイするための構成ファイル **operator.yaml** をダウンロードします。

        ```bash
        curl -O https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
        ```

      - 構成ファイル **operator.yaml** をカスタマイズします。
      - StarRocksオペレーターをデプロイします。

        ```bash
        kubectl apply -f operator.yaml
        ```

3. StarRocksオペレーターの実行状態を確認します。ポッドが `Running` の状態であり、ポッド内のすべてのコンテナが `READY` の状態にある場合、StarRocksオペレーターは期待どおりに実行されています。

    ```bash
    $ kubectl -n starrocks get pods
    NAME                                  READY   STATUS    RESTARTS   AGE
    starrocks-controller-65bb8679-jkbtg   1/1     Running   0          5分6秒
    ```

> **ノート**
>
> StarRocksオペレーターの場所にカスタム名前空間を使用する場合は、`starrocks` をカスタマイズされた名前空間の名前に置き換える必要があります。

## StarRocksクラスターをデプロイする

StarRocksが提供する[サンプル構成ファイル](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)を使用して、StarRocksクラスター（カスタムリソースStarRocks Clusterを使用してインスタンス化されたオブジェクト）を直接デプロイすることができます。例えば、**starrocks-fe-and-be.yaml** を使用して、3つのFEノードと3つのBEノードを含むStarRocksクラスターをデプロイすることができます。

```bash
kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/examples/starrocks/starrocks-fe-and-be.yaml
```

次のテーブルでは、**starrocks-fe-and-be.yaml** ファイルのいくつかの重要なフィールドについて説明しています。

| **フィールド** | **説明**                                       |
| ---------- | --------------------------------------------- |
| Kind       | オブジェクトのリソースタイプ。値は `StarRocksCluster` である必要があります。 |
| Metadata   | メタデータがネストされており、以下のサブフィールドがネストされています：<ul><li>`name`: オブジェクトの名前。各オブジェクト名は同じリソースタイプのオブジェクトを一意に識別します。</li><li>`namespace`: オブジェクトが属する名前空間。</li></ul> |
| Spec       | オブジェクトの期待される状態。有効な値は `starRocksFeSpec`、`starRocksBeSpec`、`starRocksCnSpec` です。 |

また修正された構成ファイルを使用して、StarRocksクラスターをデプロイすることもできます。サポートされているフィールドとその詳細な説明については、[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md)を参照してください。

StarRocksクラスターのデプロイには時間がかかります。この期間中、`kubectl -n starrocks get pods` コマンドを使用して、StarRocksクラスターの起動状態を確認できます。すべてのポッドが `Running` の状態であり、ポッド内のすべてのコンテナが `READY` の状態であれば、StarRocksクラスターは期待どおりに実行されています。

> **ノート**
>
> 長い時間が経過しても一部のポッドが起動しない場合は、`kubectl logs -n starrocks <ポッド名>` コマンドを使用してログ情報を表示するか、`kubectl -n starrocks describe pod <ポッド名>` コマンドを使用してイベント情報を表示して問題の場所を特定してください。

## StarRocksクラスターの管理

### StarRocksクラスターへのアクセス

StarRocksクラスターのコンポーネントには、関連するサービス（FEサービスなど）を介してアクセスできます。サービスとそのアクセスアドレスの詳細な説明については、[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md) および [Services](https://kubernetes.io/docs/concepts/services-networking/service/) を参照してください。

> **ノート**
>
> - デフォルトではFEサービスのみがデプロイされます。BEサービスやCNサービスをデプロイする必要がある場合は、StarRocksクラスター構成ファイルで `starRocksBeSpec` および `starRocksCnSpec` を構成する必要があります。
> - サービスの名前はデフォルトで `<クラスター名>-<コンポーネント名>-service` です。 例：`starrockscluster-sample-fe-service`。各コンポーネントのスペックでサービス名を指定することもできます。

#### Kubernetesクラスター内からStarRocksクラスターへのアクセス

Kubernetesクラスター内からは、FEサービスのClusterIPを通じてStarRocksクラスターにアクセスできます。

1. FEサービスの内部の仮想IPアドレス `CLUSTER-IP` およびポート `PORT(S)` を取得します。

    ```Bash
    $ kubectl -n starrocks get svc 
    NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
    be-domain-search                     ClusterIP   None             <none>        9050/TCP                              23分
    fe-domain-search                     ClusterIP   None             <none>        9030/TCP                              25分
    starrockscluster-sample-fe-service   ClusterIP   10.100.162.xxx   <none>        8030/TCP,9020/TCP,9030/TCP,9010/TCP   25分
    ```
```
2. クラスター内のKubernetesクラスターからMySQLクライアントを使用してStarRocksクラスターにアクセスします。

   ```Bash
   mysql -h 10.100.162.xxx -P 9030 -uroot
   ```

#### Kubernetesクラスターの外部からStarRocksクラスターにアクセスする

Kubernetesクラスターの外部からは、FEサービスのロードバランサーまたはNodePortを介してStarRocksクラスターにアクセスできます。このトピックではLoadBalancerの例を使用します。

1. コマンド`kubectl -n starrocks edit src starrockscluster-sample`を実行してStarRocksクラスターの構成ファイルを更新し、`starRocksFeSpec`のServiceタイプを`LoadBalancer`に変更します。

    ```YAML
    starRocksFeSpec:
      image: starrocks/fe-ubuntu:3.0-latest
      replicas: 3
      requests:
        cpu: 4
        memory: 16Gi
      service:            
        type: LoadBalancer # LoadBalancerとして指定
    ```

2. 外部に公開されたFEサービスのIPアドレス `EXTERNAL-IP` とポート `PORT(S)` を取得します。

    ```Bash
    $ kubectl -n starrocks get svc
    NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                                                       AGE
    be-domain-search                     ClusterIP      None             <none>                                                                   9050/TCP                                                      127m
    fe-domain-search                     ClusterIP      None             <none>                                                                   9030/TCP                                                      129m
    starrockscluster-sample-fe-service   LoadBalancer   10.100.162.xxx   a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com   8030:30629/TCP,9020:32544/TCP,9030:32244/TCP,9010:32024/TCP   129m               ClusterIP      None            <none>                                                                   9030/TCP                                                      23h
    ```

3. マシンホストにログインし、MySQLクライアントを使用してStarRocksクラスターにアクセスします。

    ```Bash
    mysql -h a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com -P9030 -uroot
    ```

### StarRocksクラスターのアップグレード

#### BEノードのアップグレード

次のコマンドを実行して新しいBEイメージファイル（例： `starrocks/be-ubuntu:latest`）を指定します:

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"image":"starrocks/be-ubuntu:latest"}}}'
```

#### FEノードのアップグレード

次のコマンドを実行して新しいFEイメージファイル（例： `starrocks/fe-ubuntu:latest`）を指定します:

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"image":"starrocks/fe-ubuntu:latest"}}}'
```

アップグレードプロセスにはしばらく時間がかかります。アップグレードの進行状況を表示するには、`kubectl -n starrocks get pods` コマンドを実行できます。

### StarRocksクラスターのスケーリング

このトピックでは、BEクラスターとFEクラスターのスケーリングアウトを例に取ります。

#### BEクラスターのスケーリングアウト

以下のコマンドを実行してBEクラスターを9ノードまでスケーリングアウトします:

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"replicas":9}}}'
```

#### FEクラスターのスケーリングアウト

以下のコマンドを実行してFEクラスターを4ノードまでスケーリングアウトします:

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"replicas":4}}}'
```

スケーリングプロセスにはしばらく時間がかかります。スケーリングの進行状況を表示するには、`kubectl -n starrocks get pods` コマンドを使用できます。

### CNクラスターの自動スケーリング

`kubectl -n starrocks edit src starrockscluster-sample` コマンドを実行して、CNクラスターの自動スケーリングポリシーを構成します。CNに対するリソースメトリクスとして、平均CPU使用率、平均メモリ使用量、弾力的スケーリング閾値、上限弾力的スケーリング、下限弾力的スケーリングを指定できます。上限弾力的スケーリングと下限弾力的スケーリングは、弾力的スケーリングに許可されるCNの最大数と最小数を指定します。

> **注意**
>
> CNクラスターの自動スケーリングポリシーが構成されている場合、StarRocksクラスターの構成ファイルの`starRocksCnSpec`から`replicas`フィールドを削除してください。

Kubernetesはまた、`behavior`を使用してビジネスシナリオに応じたスケーリング動作をカスタマイズすることをサポートしており、急速なスケーリング、遅いスケーリング、スケーリングを無効にするスケーリングなどを実現するのに役立ちます。自動スケーリングポリシーに関する詳細情報については、[Horizontal Pod Scaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)を参照してください。

次の内容はStarRocksが提供する[テンプレート](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_cn.yaml)です。これにより、自動スケーリングポリシーを構成できます:

```YAML
  starRocksCnSpec:
    image: starrocks/cn-ubuntu:latest
    limits:
      cpu: 16
      memory: 64Gi
    requests:
      cpu: 16
      memory: 64Gi
    autoScalingPolicy: # CNクラスターの自動スケーリングポリシー。
      maxReplicas: 10 # CNの最大数を10に設定。
      minReplicas: 1 # CNの最小数を1に設定。
      # 次のフィールドに基づいてoperatorがHPAリソースを作成します。
      # 詳細情報については、https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/ を参照してください。
      hpaPolicy:
        metrics: # リソースメトリクス
          - type: Resource
            resource:
              name: memory  # CNの平均メモリ使用量をリソースメトリクスとして指定。
              target:
                # 弾力的スケーリング閾値は60%。
                # CNの平均メモリ使用率が60%を超えると、CNの数はスケールアウトします。
                # CNの平均メモリ使用率が60%を下回ると、CNの数はスケールインします。
                averageUtilization: 60
                type: Utilization
          - type: Resource
            resource:
              name: cpu # CNの平均CPU使用率をリソースメトリクスとして指定。
              target:
                # 弾力的スケーリング閾値は60%。
                # CNの平均CPU使用率が60%を超えると、CNの数はスケールアウトします。
                # CNの平均CPU使用率が60%を下回ると、CNの数はスケールインします。
                averageUtilization: 60
                type: Utilization
        behavior: # ビジネスシナリオに応じてスケーリング動作をカスタマイズします。
          scaleUp:
            policies:
              - type: Pods
                value: 1
                periodSeconds: 10
          scaleDown:
            selectPolicy: Disabled
```

次の表はいくつかの重要なフィールドを説明しています:

- 上限と下限の弾力的スケーリング。

```YAML
maxReplicas: 10 # CNの最大数を10に設定。
minReplicas: 1 # CNの最小数を1に設定。
```

- 弾力的スケーリング閾値。

```YAML
# 例として、CNの平均CPU使用率をリソースメトリクスとして指定。
# 弾力的スケーリング閾値は60%。
# CNの平均CPU使用率が60%を超えると、CNの数はスケールアウトします。
# CNの平均CPU使用率が60%を下回ると、CNの数はスケールインします。
- type: Resource
  resource:
    name: cpu
    target:
      averageUtilization: 60
```

## よくある質問

**問題の説明:** `kubectl apply -f xxx` を使用してカスタムリソースStarRocksClusterがインストールされると、エラーが返され、「カスタムリソース定義 'starrocksclusters.starrocks.com' が無効です: metadata.annotations: Too long: must have at most 262144 bytes」と表示されます。

**原因の分析:** `kubectl apply -f xxx` を使用してリソースを作成または更新するたびに、メタデータ注釈`kubectl.kubernetes.io/last-applied-configuration`が追加されます。このメタデータ注釈はJSON形式であり、`last-applied-configuration`を記録します。 `kubectl apply -f xxx` はほとんどの場合に適していますが、カスタムリソースの設定ファイルが大きすぎる場合など、稀な状況では、メタデータ注釈のサイズが制限を超えることがあります。

**解決策:** カスタムリソースStarRocksClusterを初めてインストールする場合は、`kubectl create -f xxx` を使用することをお勧めします。環境に既にカスタムリソースStarRocksClusterがインストールされており、その設定を更新する必要がある場合は、`kubectl replace -f xxx` を使用することをお勧めします。