---
displayed_sidebar: English
---

# Operatorを使用したStarRocksのデプロイ

このトピックでは、StarRocks Operatorを使用して、Kubernetesクラスター上のStarRocksクラスターのデプロイと管理を自動化する方法を紹介します。

## 仕組み

![img](../assets/starrocks_operator.png)

## 始める前に

### Kubernetesクラスターの作成

Amazon Elastic Kubernetes Service(EKS)やGoogle Kubernetes Engine(GKE)などのクラウド管理型Kubernetesサービス、または自己管理型Kubernetesクラスターを使用できます。

- Amazon EKSクラスターの作成

  1. [次のコマンドラインツールが環境にインストールされていることを確認します](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)：
     1. AWSコマンドラインツールAWS CLIをインストールして設定します。
     2. EKSクラスターコマンドラインツールeksctlをインストールします。
     3. Kubernetesクラスターコマンドラインツールkubectlをインストールします。
  2. 次のいずれかの方法を使用して、EKSクラスターを作成します：
     1. [eksctlを使用して、EKSクラスターをすばやく作成します](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)。
     2. [AWSコンソールとAWS CLIを使用してEKSクラスターを手動で作成します](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)。

- GKEクラスタの作成

  GKEクラスタの作成を開始する前に、[前提条件をすべて満たしていることを確認してください](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#before-you-begin)。次に、[GKEクラスタを作成する](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#create_cluster)の手順に沿ってGKEクラスタを作成します。

- 自己管理型Kubernetesクラスターの作成

  [kubeadmを使用したクラスターのブートストラップ](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)に記載されている手順に従って、自己管理型Kubernetesクラスターを作成します。MinikubeとDocker Desktopを使用して、最小限の手順で単一ノードのプライベートKubernetesクラスターを作成できます。

### StarRocks Operatorのデプロイ

1. カスタムリソースStarRocksClusterを追加します。

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/starrocks.com_starrocksclusters.yaml
   ```

2. StarRocks Operatorをデプロイします。StarRocks Operatorは、デフォルトの設定ファイルまたはカスタム設定ファイルを使用してデプロイすることができます。
   1. デフォルトの設定ファイルを使用してStarRocks Operatorをデプロイします。

      ```bash
      kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
      ```

      StarRocks Operatorは`starrocks`ネームスペースにデプロイされ、すべてのネームスペースの下にあるすべてのStarRocksクラスターを管理します。
   2. カスタム設定ファイルを使用してStarRocks Operatorをデプロイします。
      - StarRocks Operatorのデプロイに使用する設定ファイル**operator.yaml**をダウンロードします。

        ```bash
        curl -O https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
        ```

      - 設定ファイル**operator.yaml**を必要に応じて変更します。
      - StarRocks Operatorをデプロイします。

        ```bash
        kubectl apply -f operator.yaml
        ```

3. StarRocks Operatorの実行状態を確認します。Podが`Running`状態にあり、Pod内のすべてのコンテナーが`READY`であれば、StarRocks Operatorは期待どおりに実行されています。

    ```bash
    $ kubectl -n starrocks get pods
    NAME                                  READY   STATUS    RESTARTS   AGE
    starrocks-controller-65bb8679-jkbtg   1/1     Running   0          5m6s
    ```

> **注記**
>
> StarRocks Operatorが配置されているネームスペースをカスタマイズする場合は、`starrocks`をカスタマイズしたネームスペースの名前に置き換える必要があります。

## StarRocksクラスタのデプロイ

StarRocksが提供する[サンプル構成ファイル](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)を直接使用して、StarRocksクラスター（カスタムリソースStarRocksClusterを使用してインスタンス化されたオブジェクト）をデプロイできます。たとえば、**starrocks-fe-and-be.yaml**を使用して、3つのFEノードと3つのBEノードを含むStarRocksクラスターをデプロイできます。

```bash
kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/examples/starrocks/starrocks-fe-and-be.yaml
```

次の表では、**starrocks-fe-and-be.yaml**ファイルのいくつかの重要なフィールドについて説明します。

| **フィールド** | **説明**                                              |
| --------- | ------------------------------------------------------------ |
| Kind      | オブジェクトのリソースタイプです。値は`StarRocksCluster`でなければなりません。 |
| Metadata  | メタデータで、以下のサブフィールドがネストされています：<ul><li>`name`: オブジェクトの名前。各オブジェクト名は、同じリソースタイプのオブジェクトを一意に識別します。</li><li>`namespace`: オブジェクトが属するネームスペース。</li></ul> |
| Spec      | オブジェクトの期待される状態です。有効な値は`starRocksFeSpec`、`starRocksBeSpec`、`starRocksCnSpec`です。 |

また、変更した構成ファイルを使用してStarRocksクラスターをデプロイすることもできます。サポートされているフィールドと詳細な説明については、[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md)を参照してください。

StarRocksクラスターのデプロイにはしばらく時間がかかります。この期間中は、`kubectl -n starrocks get pods`コマンドを使用してStarRocksクラスターの起動状態を確認できます。すべてのポッドが`Running`状態で、ポッド内のすべてのコンテナーが`READY`であれば、StarRocksクラスターは期待どおりに実行されています。

> **注記**
>
> StarRocksクラスターが配置されているネームスペースをカスタマイズする場合は、`starrocks`をカスタマイズしたネームスペースの名前に置き換える必要があります。

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

> **注記**
>
> 一部のポッドが長期間経過しても起動できない場合は、`kubectl logs -n starrocks <pod_name>`を使用してログ情報を表示するか、`kubectl -n starrocks describe pod <pod_name>`を使用してイベント情報を表示して問題を特定できます。

## StarRocksクラスタの管理

### StarRocksクラスタへのアクセス

StarRocksクラスタのコンポーネントには、FEサービスなどの関連サービスを通じてアクセスできます。サービスとそのアクセスアドレスの詳細については、[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md)と[Services](https://kubernetes.io/docs/concepts/services-networking/service/)を参照してください。

> **注記**
>
> - デフォルトでは、FEサービスのみがデプロイされます。BEサービスとCNサービスをデプロイする必要がある場合は、StarRocksクラスタ構成ファイルで`starRocksBeSpec`と`starRocksCnSpec`を構成する必要があります。
> - サービスの名前はデフォルトで`<cluster name>-<component name>-service`です。例えば、`starrockscluster-sample-fe-service`です。また、各コンポーネントのspecでサービス名を指定することもできます。

#### Kubernetesクラスタ内からStarRocksクラスタにアクセス

Kubernetesクラスタ内から、FEサービスのClusterIPを介してStarRocksクラスタにアクセスできます。

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

#### Kubernetesクラスターの外部からStarRocksクラスタにアクセス

Kubernetesクラスターの外部からは、FEサービスのLoadBalancerまたはNodePortを介してStarRocksクラスタにアクセスできます。ここではLoadBalancerを例に説明します：

1. コマンド`kubectl -n starrocks edit svc starrockscluster-sample`を実行してStarRocksクラスタの設定ファイルを更新し、`starRocksFeSpec`のサービスタイプを`LoadBalancer`に変更します。

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

2. FEサービスが外部に公開するIPアドレス`EXTERNAL-IP`とポート`PORT(S)`を取得します。

    ```Bash
    $ kubectl -n starrocks get svc
    NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                                                       AGE
    be-domain-search                     ClusterIP      None             <none>                                                                   9050/TCP                                                      127m
    fe-domain-search                     ClusterIP      None             <none>                                                                   9030/TCP                                                      129m
    starrockscluster-sample-fe-service   LoadBalancer   10.100.162.xxx   a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com   8030:30629/TCP,9020:32544/TCP,9030:32244/TCP,9010:32024/TCP   129m
    ```

3. 自分のマシンホストにログインし、MySQLクライアントを使用してStarRocksクラスタにアクセスします。

    ```Bash
    mysql -h a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com -P 9030 -uroot
    ```

### StarRocksクラスタのアップグレード

#### BEノードのアップグレード

以下のコマンドを実行して、新しいBEイメージファイルを指定します（例：`starrocks/be-ubuntu:latest`）：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"image":"starrocks/be-ubuntu:latest"}}}'
```

#### FEノードのアップグレード

以下のコマンドを実行して、新しいFEイメージファイルを指定します（例：`starrocks/fe-ubuntu:latest`）：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"image":"starrocks/fe-ubuntu:latest"}}}'
```

アップグレードプロセスには時間がかかります。コマンド`kubectl -n starrocks get pods`を実行してアップグレードの進行状況を確認できます。

### StarRocksクラスタのスケール

このトピックでは、BEクラスタとFEクラスタのスケールアウトを例に説明します。

#### BEクラスタのスケールアウト

以下のコマンドを実行して、BEクラスタを9ノードにスケールアウトします：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"replicas":9}}}'
```

#### FEクラスタのスケールアウト

以下のコマンドを実行して、FEクラスタを4ノードにスケールアウトします：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"replicas":4}}}'
```

スケーリングプロセスには時間がかかります。コマンド`kubectl -n starrocks get pods`を使用してスケーリングの進行状況を確認できます。

### CNクラスタの自動スケーリング

コマンド`kubectl -n starrocks edit src starrockscluster-sample`を実行して、CNクラスタの自動スケーリングポリシーを設定します。CNのリソースメトリックとして、平均CPU使用率、平均メモリ使用量、エラスティックスケーリングのしきい値、エラスティックスケーリングの上限、およびエラスティックスケーリングの下限を指定できます。エラスティックスケーリングの上限と下限は、エラスティックスケーリングに許可されるCNの最大数と最小数を指定します。

> **注記**
>
> CNクラスタの自動スケーリングポリシーが設定されている場合、StarRocksクラスタ構成ファイルの`starRocksCnSpec`から`replicas`フィールドを削除してください。

Kubernetesは、ビジネスシナリオに応じてスケーリング動作をカスタマイズする`behavior`の使用もサポートしており、迅速または緩やかなスケーリングを実現したり、スケーリングを無効にしたりするのに役立ちます。自動スケーリングポリシーの詳細については、[Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)を参照してください。

以下は、StarRocksが提供する自動スケーリングポリシーの設定に役立つ[テンプレート](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_cn.yaml)です。

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
      # 次のフィールドに基づいてオペレーターはHPAリソースを作成します
      # 詳細についてはhttps://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/を参照してください
      hpaPolicy:
        metrics: # リソースメトリック
          - type: Resource
            resource:
              name: memory  # CNの平均メモリ使用量をリソースメトリックとして指定
              target:
                # エラスティックスケーリングのしきい値は60%
                # CNの平均メモリ使用率が60%を超えると、CNの数が増えてスケールアウトします
                # CNの平均メモリ使用率が60%未満の場合、CNの数が減ってスケールインします
                averageUtilization: 60
                type: Utilization
          - type: Resource
            resource:
              name: cpu # CNの平均CPU使用率をリソースメトリックとして指定
              target:
                # エラスティックスケーリングのしきい値は60%
                # CNの平均CPU使用率が60%を超えると、CNの数が増えてスケールアウトします
                # CNの平均CPU使用率が60%未満の場合、CNの数が減ってスケールインします
                averageUtilization: 60
                type: Utilization
        behavior: # ビジネスシナリオに応じたスケーリング動作のカスタマイズ
          scaleUp:
            policies:
              - type: Pods
                value: 1
                periodSeconds: 10
          scaleDown:
            selectPolicy: Disabled
```

次の表は、いくつかの重要なフィールドを説明しています：

- エラスティックスケーリングの上限と下限。

```YAML
maxReplicas: 10 # CNの最大数を10に設定
minReplicas: 1 # CNの最小数を1に設定
```

- エラスティックスケーリングのしきい値。

```YAML
# 例えば、CNの平均CPU使用率をリソースメトリックとして指定します
# エラスティックスケーリングのしきい値は60%
# CNの平均CPU使用率が60%を超えると、CNの数が増えてスケールアウトします
# CNの平均CPU使用率が60%未満の場合、CNの数が減ってスケールインします
- type: Resource
  resource:
    name: cpu
    target:
      averageUtilization: 60
```

## FAQ

**問題の説明：** `kubectl apply -f xxx`を使用してカスタムリソースStarRocksClusterをインストールすると、エラーが返されます：`The CustomResourceDefinition 'starrocksclusters.starrocks.com' is invalid: metadata.annotations: Too long: must have at most 262144 bytes`。

**原因分析：** `kubectl apply -f xxx`を使用してリソースを作成または更新するたびに、メタデータアノテーション`kubectl.kubernetes.io/last-applied-configuration`が追加されます。このメタデータアノテーションはJSON形式で、*最後に適用された構成*を記録します。`kubectl apply -f xxx`はほとんどの場合に適していますが、稀に、カスタムリソースの構成ファイルが大きすぎるなどの状況で、メタデータアノテーションのサイズが制限を超えることがあります。

**解決策:** カスタムリソースStarRocksClusterを初めてインストールする場合は、`kubectl create -f xxx` の使用をお勧めします。既に環境にカスタムリソースStarRocksClusterがインストールされており、その設定を更新する必要がある場合は、`kubectl replace -f xxx` の使用をお勧めします。
