---
displayed_sidebar: "Japanese"
---

# StarRocks Operatorを使用してStarRocksをデプロイする

このトピックでは、StarRocks Operatorを使用して、Kubernetesクラスタ上にStarRocksクラスタを自動化してデプロイおよび管理する方法について説明します。

## 動作原理

![img](../assets/starrocks_operator.png)

## 開始する前に

### Kubernetesクラスタを作成する

クラウドで管理されるKubernetesサービス（[Amazon Elastic Kubernetes Service（EKS）](https://aws.amazon.com/eks/?nc1=h_ls)や[Google Kubernetes Engine（GKE）](https://cloud.google.com/kubernetes-engine)など）または自己管理のKubernetesクラスタを使用できます。

- Amazon EKSクラスタを作成する

  1. [環境に次のコマンドラインツールがインストールされていることを確認します](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)：
     1. AWS CLI（AWSコマンドラインツール）をインストールおよび設定します。
     2. EKSクラスタコマンドラインツールeksctlをインストールします。
     3. Kubernetesクラスタコマンドラインツールkubectlをインストールします。
  2. 次のいずれかの方法を使用してEKSクラスタを作成します：
     1. [eksctlを使用して素早くEKSクラスタを作成する](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)。
     2. [AWSコンソールとAWS CLIを使用して手動でEKSクラスタを作成する](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)。

- GKEクラスタを作成する

  GKEクラスタを作成する前に、[前提条件](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#before-you-begin)をすべて完了していることを確認してください。その後、[GKEクラスタを作成する](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#create_cluster)の手順に従ってGKEクラスタを作成します。

- 自己管理のKubernetesクラスタを作成する

  [kubeadmを使用してクラスタをブートストラップする](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)の手順に従って、自己管理のKubernetesクラスタを作成します。[Minikube](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)や[Docker Desktop](https://docs.docker.com/desktop/)を使用して、最小限の手順でシングルノードのプライベートKubernetesクラスタを作成することもできます。

### StarRocks Operatorをデプロイする

1. カスタムリソースStarRocksClusterを追加します。

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/starrocks.com_starrocksclusters.yaml
   ```

2. StarRocks Operatorをデプロイします。デフォルトの設定ファイルまたはカスタムの設定ファイルを使用して、StarRocks Operatorをデプロイすることができます。
   1. デフォルトの設定ファイルを使用してStarRocks Operatorをデプロイします。

      ```bash
      kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
      ```

      StarRocks Operatorは`starrocks`という名前空間にデプロイされ、すべての名前空間の下にあるすべてのStarRocksクラスタを管理します。
   2. カスタムの設定ファイルを使用してStarRocks Operatorをデプロイします。
      - StarRocks Operatorをデプロイするために使用される設定ファイル**operator.yaml**をダウンロードします。

        ```bash
        curl -O https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
        ```

      - 設定ファイル**operator.yaml**を必要に応じて変更します。
      - StarRocks Operatorをデプロイします。

        ```bash
        kubectl apply -f operator.yaml
        ```

3. StarRocks Operatorの実行状態を確認します。ポッドが`Running`状態で、ポッド内のすべてのコンテナが`READY`である場合、StarRocks Operatorは正常に実行されています。

    ```bash
    $ kubectl -n starrocks get pods
    NAME                                  READY   STATUS    RESTARTS   AGE
    starrocks-controller-65bb8679-jkbtg   1/1     Running   0          5m6s
    ```

> **注意**
>
> StarRocks Operatorが配置される名前空間をカスタマイズした場合は、`starrocks`をカスタマイズした名前空間の名前に置き換える必要があります。

## StarRocksクラスタをデプロイする

StarRocksクラスタ（カスタムリソースStarRocks Clusterを使用してインスタンス化されたオブジェクト）をデプロイするために、StarRocksが提供する[サンプルの設定ファイル](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)を直接使用することができます。たとえば、**starrocks-fe-and-be.yaml**を使用して、3つのFEノードと3つのBEノードを含むStarRocksクラスタをデプロイすることができます。

```bash
kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/examples/starrocks/starrocks-fe-and-be.yaml
```

次の表には、**starrocks-fe-and-be.yaml**ファイルのいくつかの重要なフィールドが記載されています。

| **フィールド** | **説明**                                                     |
| -------------- | ------------------------------------------------------------ |
| Kind           | オブジェクトのリソースタイプです。値は`StarRocksCluster`である必要があります。 |
| Metadata       | メタデータで、次のサブフィールドがネストされています：<ul><li>`name`：オブジェクトの名前。各オブジェクト名は、同じリソースタイプのオブジェクトを一意に識別します。</li><li>`namespace`：オブジェクトが所属する名前空間。</li></ul> |
| Spec           | オブジェクトの期待される状態です。有効な値は`starRocksFeSpec`、`starRocksBeSpec`、`starRocksCnSpec`です。 |

また、変更した設定ファイルを使用してStarRocksクラスタをデプロイすることもできます。サポートされているフィールドと詳細な説明については、[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md)を参照してください。

StarRocksクラスタのデプロイには時間がかかります。この期間中、コマンド`kubectl -n starrocks get pods`を使用してStarRocksクラスタの起動状態を確認できます。すべてのポッドが`Running`状態で、ポッド内のすべてのコンテナが`READY`である場合、StarRocksクラスタは正常に実行されています。

> **注意**
>
> StarRocksクラスタが配置される名前空間をカスタマイズした場合は、`starrocks`をカスタマイズした名前空間の名前に置き換える必要があります。

```bash
$ kubectl -n starrocks get pods
NAME                                  READY   STATUS    RESTARTS   AGE
starrocks-controller-65bb8679-jkbtg   1/1     Running   0          22h
starrockscluster-sample-be-0          1/1     Running   0          23h
starrockscluster-sample-be-1          1/1     Running   0          23h
starrockscluster-sample-be-2          1/1     Running   0          22h
starrockscluster-sample-fe-0          1/1     Running   0          21h
starrockscluster-sample-fe-1          1/1     Running   0          21h
starrockscluster-sample-fe-2          1/1     Running   0          22h
```

> **注意**
>
> 長時間経過しても一部のポッドが起動できない場合は、`kubectl logs -n starrocks <pod_name>`コマンドを使用してログ情報を表示するか、`kubectl -n starrocks describe pod <pod_name>`コマンドを使用してイベント情報を表示して問題を特定できます。

## StarRocksクラスタの管理

### StarRocksクラスタへのアクセス

StarRocksクラスタのコンポーネントは、関連するサービス（FEサービスなど）を介してアクセスできます。サービスとそのアクセスアドレスの詳細な説明については、[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md)および[Services](https://kubernetes.io/docs/concepts/services-networking/service/)を参照してください。

> **注意**
>
> - デフォルトではFEサービスのみがデプロイされます。BEサービスとCNサービスをデプロイする必要がある場合は、StarRocksクラスタの設定ファイルで`starRocksBeSpec`および`starRocksCnSpec`を設定する必要があります。
> - サービスの名前はデフォルトで`<クラスタ名>-<コンポーネント名>-service`となります。たとえば、`starrockscluster-sample-fe-service`です。各コンポーネントのspecでサービス名を指定することもできます。

#### Kubernetesクラスタ内からStarRocksクラスタにアクセスする

Kubernetesクラスタ内からは、FEサービスのClusterIPを介してStarRocksクラスタにアクセスできます。

1. FEサービスの内部仮想IPアドレス`CLUSTER-IP`とポート`PORT(S)`を取得します。

    ```Bash
    $ kubectl -n starrocks get svc 
    NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
    be-domain-search                     ClusterIP   None             <none>        9050/TCP                              23m
    fe-domain-search                     ClusterIP   None             <none>        9030/TCP                              25m
    starrockscluster-sample-fe-service   ClusterIP   10.100.162.xxx   <none>        8030/TCP,9020/TCP,9030/TCP,9010/TCP   25m
    ```

2. Kubernetesクラスタ内からMySQLクライアントを使用してStarRocksクラスタにアクセスします。

   ```Bash
   mysql -h 10.100.162.xxx -P 9030 -uroot
   ```

#### Kubernetesクラスタの外部からStarRocksクラスタにアクセスする

Kubernetesクラスタの外部からは、FEサービスのLoadBalancerまたはNodePortを介してStarRocksクラスタにアクセスできます。このトピックではLoadBalancerを例として使用します。

1. コマンド`kubectl -n starrocks edit src starrockscluster-sample`を実行して、StarRocksクラスタの設定ファイルを更新し、`starRocksFeSpec`のServiceタイプを`LoadBalancer`に変更します。

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

2. 外部に公開されるFEサービスのIPアドレス`EXTERNAL-IP`とポート`PORT(S)`を取得します。

    ```Bash
    $ kubectl -n starrocks get svc
    NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                                                       AGE
    be-domain-search                     ClusterIP      None             <none>                                                                   9050/TCP                                                      127m
    fe-domain-search                     ClusterIP      None             <none>                                                                   9030/TCP                                                      129m
    starrockscluster-sample-fe-service   LoadBalancer   10.100.162.xxx   a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com   8030:30629/TCP,9020:32544/TCP,9030:32244/TCP,9010:32024/TCP   129m               ClusterIP      None            <none>                                                                   9030/TCP                                                      23h
    ```

3. マシンホストにログインし、MySQLクライアントを使用してStarRocksクラスタにアクセスします。

    ```Bash
    mysql -h a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com -P9030 -uroot
    ```

### StarRocksクラスタのアップグレード

#### BEノードのアップグレード

次のコマンドを実行して、新しいBEイメージファイル（たとえば`starrocks/be-ubuntu:latest`）を指定します。

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"image":"starrocks/be-ubuntu:latest"}}}'
```

#### FEノードのアップグレード

次のコマンドを実行して、新しいFEイメージファイル（たとえば`starrocks/fe-ubuntu:latest`）を指定します。

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"image":"starrocks/fe-ubuntu:latest"}}}'
```

アップグレードプロセスには時間がかかります。コマンド`kubectl -n starrocks get pods`を実行してアップグレードの進行状況を確認できます。

### StarRocksクラスタのスケール

このトピックでは、BEクラスタとFEクラスタのスケールアウトを例として説明します。

#### BEクラスタのスケールアウト

次のコマンドを実行して、BEクラスタを9ノードにスケールアウトします。

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"replicas":9}}}'
```

#### FEクラスタのスケールアウト

次のコマンドを実行して、FEクラスタを4ノードにスケールアウトします。

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"replicas":4}}}'
```

スケールアウトのプロセスには時間がかかります。コマンド`kubectl -n starrocks get pods`を使用してスケールアウトの進行状況を確認できます。

### CNクラスタの自動スケーリング

コマンド`kubectl -n starrocks edit src starrockscluster-sample`を実行して、CNクラスタの自動スケーリングポリシーを設定します。CNのリソースメトリクスとして、平均CPU使用率、平均メモリ使用量、弾力性スケーリングの閾値、上限弾力性スケーリング、下限弾力性スケーリングなどを指定することができます。上限弾力性スケーリングと下限弾力性スケーリングは、弾力性スケーリングに許可されるCNの最大数と最小数を指定します。

> **注意**
>
> CNクラスタの自動スケーリングポリシーが設定されている場合は、StarRocksクラスタの設定ファイルの`starRocksCnSpec`から`replicas`フィールドを削除してください。

Kubernetesでは、ビジネスシナリオに応じてスケーリング動作をカスタマイズするために`behavior`を使用することもできます。自動スケーリングポリシーの詳細については、[Horizontal Pod Scaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)を参照してください。

StarRocksが提供する[テンプレート](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_cn.yaml)を使用して、自動スケーリングポリシーを設定する手順を説明します。

```YAML
  starRocksCnSpec:
    image: starrocks/cn-ubuntu:latest
    limits:
      cpu: 16
      memory: 64Gi
    requests:
      cpu: 16
      memory: 64Gi
    autoScalingPolicy: # CNクラスタの自動スケーリングポリシー
      maxReplicas: 10 # CNの最大数を10に設定
      minReplicas: 1 # CNの最小数を1に設定
      # operatorは、次のフィールドに基づいてHPAリソースを作成します。
      # 詳細については、https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/を参照してください。
      hpaPolicy:
        metrics: # リソースメトリクス
          - type: Resource
            resource:
              name: memory  # CNの平均メモリ使用量をリソースメトリクスとして指定
              target:
                # 弾力性スケーリングの閾値は60%です。
                # CNの平均メモリ使用率が60%を超えると、スケールアウトのためにCNの数が増えます。
                # CNの平均メモリ使用率が60%未満の場合、スケールインのためにCNの数が減ります。
                averageUtilization: 60
                type: Utilization
          - type: Resource
            resource:
              name: cpu # CNの平均CPU使用率をリソースメトリクスとして指定
              target:
                # 弾力性スケーリングの閾値は60%です。
                # CNの平均CPU使用率が60%を超えると、スケールアウトのためにCNの数が増えます。
                # CNの平均CPU使用率が60%未満の場合、スケールインのためにCNの数が減ります。
                averageUtilization: 60
                type: Utilization
        behavior: # ビジネスシナリオに応じてスケーリング動作をカスタマイズすることができます。急速なスケーリング、遅いスケーリング、スケーリングの無効化などが可能です。
          scaleUp:
            policies:
              - type: Pods
                value: 1
                periodSeconds: 10
          scaleDown:
            selectPolicy: Disabled
```

次の表には、いくつかの重要なフィールドが記載されています。

- 上限弾力性スケーリングと下限弾力性スケーリング。

```YAML
maxReplicas: 10 # CNの最大数を10に設定
minReplicas: 1 # CNの最小数を1に設定
```

- 弾力性スケーリングの閾値。

```YAML
# たとえば、CNの平均CPU使用率をリソースメトリクスとして指定します。
# 弾力性スケーリングの閾値は60%です。
# CNの平均CPU使用率が60%を超えると、スケールアウトのためにCNの数が増えます。
# CNの平均CPU使用率が60%未満の場合、スケールインのためにCNの数が減ります。
- type: Resource
  resource:
    name: cpu
    target:
      averageUtilization: 60
```

## FAQ

**問題の説明:** `kubectl apply -f xxx`を使用してカスタムリソースStarRocksClusterをインストールすると、エラー`The CustomResourceDefinition 'starrocksclusters.starrocks.com' is invalid: metadata.annotations: Too long: must have at most 262144 bytes`が返されます。

**原因の分析:** `kubectl apply -f xxx`を使用してリソースを作成または更新すると、メタデータのアノテーション`kubectl.kubernetes.io/last-applied-configuration`が追加されます。このメタデータのアノテーションはJSON形式であり、*last-applied-configuration*を記録します。`kubectl apply -f xxx`はほとんどの場合に適していますが、カスタムリソースの設定ファイルが非常に大きい場合など、メタデータのアノテーションのサイズが制限を超える可能性があります。

**解決策:** カスタムリソースStarRocksClusterを初めてインストールする場合は、`kubectl create -f xxx`を使用することをお勧めします。環境にすでにカスタムリソースStarRocksClusterがインストールされており、その設定を更新する必要がある場合は、`kubectl replace -f xxx`を使用することをお勧めします。
