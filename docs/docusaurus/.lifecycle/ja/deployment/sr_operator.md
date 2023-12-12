---
displayed_sidebar: "Japanese"
---

# StarRocks Operator を使用した StarRocks のデプロイ

このトピックでは、StarRocks Operator を使用して Kubernetes クラスター上に StarRocks クラスターを自動的にデプロイおよび管理する方法について紹介します。

## 動作原理

![img](../assets/starrocks_operator.png)

## 開始する前に

### Kubernetes クラスターの作成

クラウドで管理される Kubernetes サービス ([Amazon Elastic Kubernetes Service (EKS)](https://aws.amazon.com/eks/?nc1=h_ls) や [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine)) を使用するか、自己管理型の Kubernetes クラスターを使用できます。

- Amazon EKS クラスターの作成

  1. [環境に次のコマンドラインツールがインストールされていること](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)を確認してください:
     1. AWS コマンドラインツール AWS CLI のインストールと構成
     2. EKS クラスターコマンドラインツール eksctl のインストール
     3. Kubernetes クラスターコマンドラインツール kubectl のインストール
  2. 次のいずれかの方法で EKS クラスターを作成します:
     1. [eksctl を使用して迅速に EKS クラスターを作成](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)
     2. [AWS コンソールおよび AWS CLI を使用して手動で EKS クラスターを作成](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)

- GKE クラスターの作成

  GKE クラスターを作成する前に、すべての[前提条件](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#before-you-begin)を満たすことを確認してください。その後、[GKE クラスターを作成](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#create_cluster)するための提供された手順に従ってください。

- 自己管理型 Kubernetes クラスターの作成

  [kubeadm を使用したクラスタのブートストラップ](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)に関する提供された手順に従い、自己管理型 Kubernetes クラスターを作成します。[Minikube](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/) や [Docker Desktop](https://docs.docker.com/desktop/) を使用して、最小限の手順で単一ノードのプライベート Kubernetes クラスターを作成することができます。

### StarRocks Operator のデプロイ

1. カスタムリソース StarRocksCluster を追加します。

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/starrocks.com_starrocksclusters.yaml
   ```

2. StarRocks Operator をデプロイします。デフォルトの構成ファイルまたはカスタムの構成ファイルを使用して StarRocks Operator をデプロイできます。
   1. デフォルトの構成ファイルを使用して StarRocks Operator をデプロイします。

      ```bash
      kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
      ```

      StarRocks Operator は、名前空間 `starrocks` にデプロイされ、すべての名前空間内の StarRocks クラスターを管理します。
   2. カスタム構成ファイルを使用して StarRocks Operator をデプロイします。
      - StarRocks Operator をデプロイするために使用される構成ファイル **operator.yaml** をダウンロードします。

        ```bash
        curl -O https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
        ```

      - 構成ファイル **operator.yaml** を必要に応じて変更します。
      - StarRocks Operator をデプロイします。

        ```bash
        kubectl apply -f operator.yaml
        ```

3. StarRocks Operator の実行状態を確認します。Pod が `実行中` の状態であり、ポッド内のすべてのコンテナが `READY` である場合、StarRocks Operator は正常に実行されています。

    ```bash
    $ kubectl -n starrocks get pods
    NAME                                  READY   STATUS    RESTARTS   AGE
    starrocks-controller-65bb8679-jkbtg   1/1     Running   0          5m6s
    ```

> **注記**
>
> StarRocks Operator の場所をカスタマイズした場合、`starrocks` をカスタマイズした名前空間の名前に置き換える必要があります。

## StarRocks クラスターのデプロイ

StarRocks が提供する [サンプル構成ファイル](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks) を直接使用して StarRocks クラスター (カスタムリソース StarRocks Cluster を使用してインスタンス化されたオブジェクト) をデプロイできます。たとえば、**starrocks-fe-and-be.yaml** を使用して、3 つの FE ノードと 3 つの BE ノードを含む StarRocks クラスターをデプロイできます。

```bash
kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/examples/starrocks/starrocks-fe-and-be.yaml
```

次の表には、**starrocks-fe-and-be.yaml** ファイルのいくつかの重要なフィールドについて説明しています。

| **フィールド** | **説明**                                              |
| --------- | ------------------------------------------------------------ |
| Kind      | オブジェクトのリソースタイプ。値は `StarRocksCluster` でなければなりません。 |
| Metadata  | 以下のサブフィールドがネストされたメタデータ:<ul><li>`name`: オブジェクトの名前。各オブジェクト名は同じリソースタイプのオブジェクトを一意に識別します。</li><li>`namespace`: オブジェクトが属する名前空間。</li></ul > |
| Spec      | オブジェクトの期待されるステータス。`starRocksFeSpec`、`starRocksBeSpec`、および `starRocksCnSpec` が有効な値です。 |

サポートされているフィールドや詳細な説明については、[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md)を参照して、変更された構成ファイルを使用して StarRocks クラスターをデプロイすることもできます。

StarRocks クラスターのデプロイには時間がかかります。この期間中、`kubectl -n starrocks get pods` コマンドを使用して StarRocks クラスターの開始状態を確認できます。すべてのポッドが `実行中` の状態であり、ポッド内のすべてのコンテナが `READY` である場合、StarRocks クラスターは正常に実行されています。

> **注記**
>
> 長時間経過してもいくつかのポッドが起動しない場合、`kubectl logs -n starrocks <pod_name>` コマンドを使用してログ情報を表示するか、`kubectl -n starrocks describe pod <pod_name>` コマンドを使用して問題を特定します。

## StarRocks クラスターの管理

### StarRocks クラスターのアクセス

StarRocks クラスターのコンポーネントには、FE サービスなどの関連するサービスを介してアクセスできます。サービスとそのアクセスアドレスの詳細な説明については、[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md) および [Services](https://kubernetes.io/docs/concepts/services-networking/service/) を参照してください。

> **注記**
>
> - デフォルトで FE サービスのみがデプロイされています。BE サービスおよび CN サービスをデプロイする必要がある場合は、StarRocks クラスターの構成ファイルで `starRocksBeSpec` および `starRocksCnSpec` を設定する必要があります。
> - サービスの名前はデフォルトで `<cluster name>-<component name>-service` です。たとえば、`starrockscluster-sample-fe-service` です。各コンポーネントの仕様でサービス名を指定することもできます。

#### Kubernetes クラスター内からの StarRocks クラスターへのアクセス

Kubernetes クラスター内から、FE サービスの ClusterIP を使用して StarRocks クラスターにアクセスできます。

1. FE サービスの内部仮想 IP アドレス `CLUSTER-IP` とポート `PORT(S)` を取得します。

    ```Bash
    $ kubectl -n starrocks get svc 
    NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
    be-domain-search                     ClusterIP   None             <none>        9050/TCP                              23m
    fe-domain-search                     ClusterIP   None             <none>        9030/TCP                              25m
    starrockscluster-sample-fe-service   ClusterIP   10.100.162.xxx   <none>        8030/TCP,9020/TCP,9030/TCP,9010/TCP   25m
    ```

2. Kubernetesクラスター内からMySQLクライアントを使用してStarRocksクラスターにアクセスします。

   ```Bash
   mysql -h 10.100.162.xxx -P 9030 -uroot
   ```

#### Kubernetesクラスター外からStarRocksクラスターにアクセスします

   Kubernetesクラスターの外部からは、FEサービスのロードバランサーまたはNodePortを介してStarRocksクラスターにアクセスできます。このトピックでは、ロードバランサーを使用する例を示します。

1. `kubectl -n starrocks edit src starrockscluster-sample`コマンドを実行して、StarRocksクラスター構成ファイルを更新し、`starRocksFeSpec`のServiceタイプを`LoadBalancer`に変更します。

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

2. 外部に露出されるFEサービスのIPアドレス `EXTERNAL-IP` とポート `PORT(S)` を取得します。

    ```Bash
    $ kubectl -n starrocks get svc
    NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                                                       AGE
    be-domain-search                     ClusterIP      None             <none>                                                                   9050/TCP                                                      127m
    fe-domain-search                     ClusterIP      None             <none>                                                                   9030/TCP                                                      129m
    starrockscluster-sample-fe-service   LoadBalancer   10.100.162.xxx   a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com   8030:30629/TCP,9020:32544/TCP,9030:32244/TCP,9010:32024/TCP   129m               ClusterIP      None            <none>                                                           9030/TCP                                                      23h
    ```

3. 自分のマシンホストにログインし、MySQLクライアントを使用してStarRocksクラスターにアクセスします。

    ```Bash
    mysql -h a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com -P9030 -uroot
    ```

### StarRocksクラスターをアップグレードします

#### BEノードをアップグレードします

新しいBEイメージファイル（例：`starrocks/be-ubuntu:latest`）を指定するには、次のコマンドを実行します。

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"image":"starrocks/be-ubuntu:latest"}}}'
```

#### FEノードをアップグレードします

新しいFEイメージファイル（例：`starrocks/fe-ubuntu:latest`）を指定するには、次のコマンドを実行します。

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"image":"starrocks/fe-ubuntu:latest"}}}'
```

アップグレードプロセスにはしばらくかかります。アップグレードの進行状況を確認するには、`kubectl -n starrocks get pods`コマンドを実行できます。

### StarRocksクラスターをスケールします

このトピックでは、BEおよびFEクラスターをスケールアウトする例を示します。

#### BEクラスターをスケールアウトします

BEクラスターを9ノードにスケールアウトするには、次のコマンドを実行します。

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"replicas":9}}}'
```

#### FEクラスターをスケールアウトします

FEクラスターを4ノードにスケールアウトするには、次のコマンドを実行します。

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"replicas":4}}}'
```

スケーリングプロセスにはしばらくかかります。スケーリングの進行状況を確認するには、`kubectl -n starrocks get pods`コマンドを使用できます。

### CNクラスターの自動スケーリング

CNクラスターの自動スケーリングポリシーを構成するには、`kubectl -n starrocks edit src starrockscluster-sample`コマンドを実行します。CNのリソースメトリクスとして平均CPU利用率、平均メモリ使用率、弾力的スケーリング閾値、上限弾力的スケーリング限界、下限弾力的スケーリング限界を指定できます。上限弾力的スケーリング限界と下限弾力的スケーリング限界は、弾力的スケーリングに許可されるCNの最大数と最小数を指定します。

> **注記**
>
> CNクラスターの自動スケーリングポリシーが構成されている場合は、StarRocksクラスター構成ファイルの`starRocksCnSpec`から`replicas`フィールドを削除してください。

Kubernetesは、`behavior`を使用してビジネスシナリオに応じたスケーリング動作をカスタマイズすることもサポートしており、迅速なスケーリング、遅いスケーリング、またはスケーリングを無効にすることが可能です。自動スケーリングポリシーの詳細については、[Horizontal Pod Scaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)を参照してください。

以下はStarRocksが提供する[テンプレート](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_cn.yaml)です。これにより、自動スケーリングポリシーを構成できます。

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
      maxReplicas: 10 # CNの最大数は10に設定されています。
      minReplicas: 1 # CNの最小数は1に設定されています。
      # オペレータは、次のフィールドに基づいてHPAリソースを作成します。
      # 詳細については https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/ を参照してください。
      hpaPolicy:
        metrics: # リソースメトリクス
          - type: Resource
            resource:
              name: memory  # CNの平均メモリ使用率がリソースメトリクスとして指定されています。
              target:
                # 弾力的スケーリング閾値は60%です。
                # CNの平均メモリ利用率が60%を超えると、CNの数が増加してスケールアウトします。
                # CNの平均メモリ利用率が60%未満の場合、CNの数が減少してスケールインします。
                averageUtilization: 60
                type: Utilization
          - type: Resource
            resource:
              name: cpu # CNの平均CPU利用率がリソースメトリクスとして指定されています。
              target:
                # 弾力的スケーリング閾値は60%です。
                # CNの平均CPU利用率が60%を超えると、CNの数が増加してスケールアウトします。
                # CNの平均CPU利用率が60%未満の場合、CNの数が減少してスケールインします。
                averageUtilization: 60
                type: Utilization
        behavior: # ビジネスシナリオに応じたスケーリング動作をカスタマイズします。
          scaleUp:
            policies:
              - type: Pods
                value: 1
                periodSeconds: 10
          scaleDown:
            selectPolicy: Disabled
```

以下の表は、いくつかの重要なフィールドについて説明しています。

- 上限および下限の弾力的スケーリング限界。

```YAML
maxReplicas: 10 # CNの最大数は10に設定されています。
minReplicas: 1 # CNの最小数は1に設定されています。
```

- 弾力的スケーリング閾値。

```YAML
# 例えばCNの平均CPU利用率がリソースメトリクスとして指定されています。
# 弾力的スケーリング閾値は60%です。
# CNの平均CPU利用率が60%を超えると、CNの数が増加してスケールアウトします。
# CNの平均CPU利用率が60%未満の場合、CNの数が減少してスケールインします。
- type: Resource
  resource:
    name: cpu
    target:
      averageUtilization: 60
```

## よくある質問

**問題の説明:** `kubectl apply -f xxx`を使用してカスタムリソースStarRocksClusterがインストールされると、`The CustomResourceDefinition 'starrocksclusters.starrocks.com' is invalid: metadata.annotations: Too long: must have at most 262144 bytes`というエラーが返されます。

**原因の分析:** リソースを作成または更新する際に、`kubectl apply -f xxx`を使用すると、メタデータ注釈 `kubectl.kubernetes.io/last-applied-configuration` が追加されます。このメタデータ注釈はJSONフォーマットで、*last-applied-configuration*を記録します。`kubectl apply -f xxx`"はほとんどの場合に適していますが、カスタムリソースの設定ファイルが大きすぎる場合など、まれな状況ではメタデータ注釈のサイズが制限を超える可能性があります。

**解決策:** カスタムリソースStarRocksClusterを初めてインストールする場合は、`kubectl create -f xxx`を使用することをお勧めします。カスタムリソースStarRocksClusterがすでに環境にインストールされており、その構成を更新する必要がある場合は、`kubectl replace -f xxx`を使用することをお勧めします。