---
displayed_sidebar: Chinese
---

# Operator を使用して StarRocks クラスターをデプロイする

この文書では、Kubernetes クラスター上で StarRocks Operator を使用して StarRocks クラスターを自動的にデプロイおよび管理する方法について説明します。

## 動作原理

![sr operator and src](../assets/starrocks_operator.png)

## **環境準備**

### **Kubernetes クラスターの作成**

クラウドホスト型の Kubernetes サービスを使用することができます。例えば [Amazon Elastic Kubernetes Service (EKS)](https://aws.amazon.com/cn/eks/?nc2=h_ql_prod_ct_eks) や [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine?hl=zh-cn)、またはプライベート Kubernetes クラスターを使用することができます。

**Amazon EKS クラスターの作成**

1. EKS クラスターを作成する前に、以下のコマンドラインツールが[環境にインストールされていることを確認してください](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)。
   - AWS CLI コマンドラインツールをインストールして設定します。
   - EKS クラスターのコマンドラインツール eksctl をインストールします。
   - Kubernetes クラスターのコマンドラインツール kubectl をインストールします。
2. EKS クラスターを作成します。以下の2つの方法がサポートされています：
   - [eksctl を使用して EKS クラスターを迅速に作成する](https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/getting-started-eksctl.html)。
   - [AWS コンソールと AWS CLI を使用して手動で EKS クラスターを作成する](https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/getting-started-console.html)。

**GKE クラスターの作成**

クラスターを作成する前に、すべての前提条件が完了していることを確認してください。作成手順は[ GKE クラスターの作成](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster)を参照してください。

**プライベート Kubernetes クラスターの作成**

[Kubernetes クラスター](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/)を作成します。すぐにこの機能を体験したい場合は、[Minikube](https://kubernetes.io/zh-cn/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)を使用してシングルノードの Kubernetes クラスターを作成することができます。

### StarRocks Operator のデプロイ

1. カスタムリソース StarRocksCluster を追加します。

    ```Bash
    kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/starrocks.com_starrocksclusters.yaml
    ```

2. StarRocks Operator をデプロイします。デフォルトの設定ファイルを使用するか、カスタム設定ファイルを使用して StarRocks Operator をデプロイすることができます。
   1. デフォルトの設定ファイルを使用して StarRocks Operator をデプロイする：

      ```Bash
      kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
      ```

      StarRocks Operator は Namespace `starrocks` にデプロイされ、すべての Namespace の StarRocks クラスターを管理します。

   2. カスタム設定ファイルを使用して StarRocks Operator をデプロイする：

      1. StarRocks Operator のデプロイに使用する設定ファイルをダウンロードします。

         ```bash
         curl -O https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
         ```

      2. 実際のニーズに応じて、設定ファイル `operator.yaml` を変更します。
      3. StarRocks Operator をデプロイします。

         ```bash
         kubectl apply -f operator.yaml
         ```

3. StarRocks Operator の実行状態を確認します。Pod が `Running` 状態で、Pod 内のすべてのコンテナが `READY` であれば、StarRocks Operator が正常に実行されていることを意味します。

    ```bash
    $ kubectl -n starrocks get pods
    NAME                                  READY   STATUS    RESTARTS   AGE
    starrocks-controller-65bb8679-jkbtg   1/1     Running   0          5m6s
    ```

  > **注記**
  >
  > StarRocks Operator の Namespace をカスタムした場合は、`starrocks` をカスタムした Namespace に変更する必要があります。

## StarRocks クラスターのデプロイ

StarRocks が提供する[設定ファイルの例](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)を直接使用して、StarRocks クラスター（カスタムリソース StarRocks Cluster のインスタンス化されたオブジェクト）をデプロイすることができます。例えば **starrocks-fe-and-be.yaml** を使用して、3つの FE と 3つの BE ノードを含む StarRocks クラスターをデプロイします。

```Bash
kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/examples/starrocks/starrocks-fe-and-be.yaml
```

主なフィールドの説明：

| フィールド | 説明                                                         |
| ---------- | ------------------------------------------------------------ |
| Kind       | オブジェクトが属するリソースタイプ。`StarRocksCluster` である必要があります。 |
| Metadata   | メタデータ。以下のサブフィールドと説明が含まれます：<ul><li>`name`：オブジェクト名。同じリソースタイプ内でのオブジェクトの一意な識別子。</li><li>`namespace`：オブジェクトが属する Namespace。</li></ul> |
| Spec       | オブジェクトの期待状態を含み、`starRocksFeSpec`、`starRocksBeSpec`、`starRocksCnSpec` が含まれます。 |

設定ファイルを必要に応じて変更して StarRocks クラスターをデプロイすることもできます。サポートされているフィールドと詳細な説明については、[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md) を参照してください。

StarRocks クラスターのデプロイには時間がかかります。その間、`kubectl -n starrocks get pods` を実行して StarRocks クラスターの起動状態を確認することができます。Pod が `Running` 状態で、Pod 内のすべてのコンテナが `READY` であれば、StarRocks クラスターが正常に実行されていることを意味します。

> **注記**
>
> StarRocks クラスターの Namespace をカスタムした場合は、`starrocks` をカスタムした Namespace に変更する必要があります。

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
> 一部の Pod が長時間起動できない場合は、`kubectl logs -n starrocks <pod_name>` を実行してログ情報を確認するか、`kubectl -n starrocks describe pod <pod_name>` を実行してイベント情報を確認し、問題を特定することができます。

## StarRocks クラスターへのアクセス

StarRocks クラスターの各コンポーネントにアクセスするには、関連する Service を介して行うことができます。例えば FE Service です。Service の詳細な説明とアクセスアドレスの確認については、[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md) および [Service](https://kubernetes.io/docs/concepts/services-networking/service/) を参照してください。

> **注記**
>
> - デフォルトでは、FE Service のみがデプロイされます。BE Service および CN Service をデプロイするには、StarRocks クラスターの設定ファイル `starRocksBeSpec`、`starRocksCnSpec` に設定を追加する必要があります。
> - Service の名前はデフォルトで `<クラスター名>-<コンポーネント名>-service` となります。例えば `starrockscluster-sample-fe-service` ですが、各コンポーネントの spec で Service 名を指定することもできます。

### クラスター内で StarRocks クラスターにアクセスする

Kubernetes クラスター内で、FE Service の ClusterIP を介して StarRocks クラスターにアクセスします。

1. FE Service の内部仮想 IP `CLUSTER-IP` とポート `PORT(S)` を確認します。

    ```bash
    $ kubectl -n starrocks get svc 
    NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
    be-domain-search                     ClusterIP   None             <none>        9050/TCP                              23m
    fe-domain-search                     ClusterIP   None             <none>        9030/TCP                              25m
    starrockscluster-sample-fe-service   ClusterIP   10.100.162.xxx   <none>        8030/TCP,9020/TCP,9030/TCP,9010/TCP   25m
    ```


2. Kubernetes クラスタ内で MySQL クライアントを使用して StarRocks クラスタにアクセスします。

    ```bash
    mysql -h 10.100.162.xxx -P 9030 -uroot
    ```

### Kubernetes クラスタ外から StarRocks クラスタにアクセスする

Kubernetes クラスタ外からは、FE Service の LoadBalancer または NodePort を介して StarRocks クラスタにアクセスできます。ここでは LoadBalancer を例に説明します：

1. コマンド `kubectl -n starrocks edit src starrockscluster-sample` を実行して StarRocks クラスタの設定ファイルを更新し、`starRocksFeSpec` の Service タイプを `LoadBalancer` に変更します。

    ```YAML
    starRocksFeSpec:
      image: starrocks/fe-ubuntu:3.0-latest
      replicas: 3
      requests:
        cpu: 4
        memory: 16Gi
      service:            
        type: LoadBalancer # LoadBalancer として指定
    ```

2. FE Service が外部に公開している IP アドレス `EXTERNAL-IP` とポート `PORT(S)` を確認します。

    ```bash
    $ kubectl -n starrocks get svc
    NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                                                       AGE
    be-domain-search                     ClusterIP      None             <none>                                                                   9050/TCP                                                      127m
    fe-domain-search                     ClusterIP      None             <none>                                                                   9030/TCP                                                      129m
    starrockscluster-sample-fe-service   LoadBalancer   10.100.162.xxx   a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com   8030:30629/TCP,9020:32544/TCP,9030:32244/TCP,9010:32024/TCP   129m
    ```

3. 自分のマシンにログインし、MySQL クライアントを使用して StarRocks クラスタにアクセスします。

    ```bash
    mysql -h a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com -P 9030 -uroot
    ```

## StarRocks クラスタの管理

`kubectl edit` または `kubectl patch` コマンドを実行して StarRocks クラスタの設定ファイルを更新し、StarRocks クラスタを管理できます。

### StarRocks クラスタのアップグレード

**BE ノードのアップグレード**

以下のコマンドを実行し、新しい BE イメージファイルを指定します。例えば `starrocks/be-ubuntu:latest` です。

```Bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"image":"starrocks/be-ubuntu:latest"}}}'
```

**FE ノードのアップグレード**

以下のコマンドを実行し、新しい FE イメージファイルを指定します。例えば `starrocks/fe-ubuntu:latest` です。

```Bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"image":"starrocks/fe-ubuntu:latest"}}}'
```

アップグレードプロセスはしばらく続きますが、`kubectl -n starrocks get pods` コマンドを使用してアップグレードの進行状況を確認できます。

### StarRocks クラスタのスケールアウトとスケールイン

ここでは、BE クラスタと FE クラスタのスケールアウトを例に説明します。

**BE クラスタのスケールアウト**

以下のコマンドを実行し、BE クラスタを 9 ノードにスケールアウトします。

```Bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"replicas":9}}}'
```

**FE クラスタのスケールアウト**

以下のコマンドを実行し、FE クラスタを 4 ノードにスケールアウトします。

```Bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"replicas":4}}}'
```

スケールアウトプロセスはしばらく続きますが、`kubectl -n starrocks get pods` コマンドを使用してスケールアウトの進行状況を確認できます。

### CN クラスタの自動スケールアウトとスケールイン

`kubectl -n starrocks edit src starrockscluster-sample` コマンドを実行して CN の自動スケールアウトとスケールインのポリシーを設定します。CN のメモリと CPU の平均使用率をリソース指標として指定し、スケールアウトをトリガーするしきい値、スケールアウトの上限と下限（つまり、CN の数の上限と下限）を指定できます。

> **注意**
>
> CN の自動スケールアウトとスケールインのポリシーを設定した場合は、CN の `replicas` フィールドを削除してください。

Kubernetes は `behavior` を使用して、ビジネスシナリオに応じたスケールアウトとスケールインの動作をカスタマイズし、迅速なスケールアウト、緩やかなスケールイン、スケールインの無効化などを実現することもできます。自動スケールアウトのポリシーの詳細については、[Pod の水平自動スケールアウト](https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale/) を参照してください。

以下は StarRocks が提供する [CN の自動スケールアウトとスケールインのポリシーテンプレート](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_cn.yaml) です。

```Bash
  starRocksCnSpec:
    image: starrocks/cn-ubuntu:latest
    limits:
      cpu: 16
      memory: 64Gi
    requests:
      cpu: 16
      memory: 64Gi
    # autoscalingPolicy を使用する場合は、manifests から replicas を削除することをお勧めします。
    autoScalingPolicy: # CN クラスタの自動スケールアウトポリシー。
      maxReplicas: 10 # CN の最大数を 10 に設定します。
      minReplicas: 1 # CN の最小数を 1 に設定します。
      # 次のフィールドに基づいてオペレーターが HPA リソースを作成します。
      # 詳細は https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/ を参照してください。
      hpaPolicy:
        metrics: # リソース指標
          - type: Resource
            resource:
              name: memory  # CN の平均メモリ使用率をリソース指標として指定します。
              target:
                # 弾性スケールアウトのしきい値は 60% です。
                # CN の平均メモリ使用率が 60% を超えると、スケールアウトのために CN の数が増加します。
                # CN の平均メモリ使用率が 60% 以下の場合、スケールインのために CN の数が減少します。
                averageUtilization: 60
                type: Utilization
          - type: Resource
            resource:
              name: cpu # CN の平均 CPU 使用率をリソース指標として指定します。
              target:
                # 弾性スケールアウトのしきい値は 60% です。
                # CN の平均 CPU 使用率が 60% を超えると、スケールアウトのために CN の数が増加します。
                # CN の平均 CPU 使用率が 60% 以下の場合、スケールインのために CN の数が減少します。
                averageUtilization: 60
                type: Utilization
        behavior: # ビジネスシナリオに応じたスケールアウトとスケールインの動作をカスタマイズします。迅速なスケールアウト、緩やかなスケールイン、スケールインの無効化などを実現します。
          scaleUp:
            policies:
              - type: Pods
                value: 1
                periodSeconds: 10
          scaleDown:
            selectPolicy: Disabled
```

主要なフィールドとその説明は以下の通りです：

- CN のスケールアウトとスケールイン時の最大数と最小数。

  ```Bash
  maxReplicas: 10 # CN の最大数は 10
  minReplicas: 1  # CN の最小数は 1
  ```

- スケールアウトとスケールインをトリガーするしきい値。

  ```Bash
  # スケールアウトとスケールインをトリガーするしきい値。例えば、リソース指標が Kubernetes クラスタ内の CN の CPU 使用率です。CPU 使用率が 60% を超えると、スケールアウトのために CN の数を増やし、60% 以下の場合はスケールインのために CN の数を減らします。
  - type: Resource
    resource:
      name: cpu
      target:
        averageUtilization: 60
  ```

## よくある質問

- **問題の説明**: `kubectl apply -f xxx` コマンドを実行してカスタムリソース StarRocksCluster をデプロイする際に、エラー `The CustomResourceDefinition "starrocksclusters.starrocks.com" is invalid: metadata.annotations: Too long: must have at most 262144 bytes` が発生しました。
- **原因分析**：`kubectl apply -f xxx` を使用してリソースを作成または更新するたびに、kubectl.kubernetes.io/last-applied-configuration という名前の metadata アノテーションが追加されます。この metadata アノテーションは JSON 形式で、オブジェクトを作成するための設定ファイルの内容が含まれています。`kubectl apply -f xxx` はほとんどの場合に適していますが、例えばカスタムリソースの設定ファイルが非常に大きい場合など、ごく稀に metadata アノテーションのサイズが上限を超えることがあります。
- **解決措施**：カスタムリソース StarRocksCluster を初めてデプロイする場合は、`kubectl create -f xxx` の使用をお勧めします。既に環境にカスタムリソースがデプロイされており、その設定を更新する必要がある場合は、`kubectl replace -f xxx` の使用をお勧めします。
