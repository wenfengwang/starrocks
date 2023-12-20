---
displayed_sidebar: English
---

# 使用 Helm 部署 StarRocks

[Helm](https://helm.sh/) 是 Kubernetes 的包管理器。一个 [Helm Chart](https://helm.sh/docs/topics/charts/) 是一个 Helm 包，它包含了运行一个应用程序在 Kubernetes 集群上所需的所有资源定义。本主题描述了如何使用 Helm 自动部署 StarRocks 集群到 Kubernetes 集群。

## 在您开始之前

- [创建 Kubernetes 集群](./sr_operator.md#create-kubernetes-cluster)。
- [安装 Helm](https://helm.sh/docs/intro/quickstart/)。

## 步骤

1. 添加 StarRocks 的 Helm Chart 仓库。Helm Chart 包含了 StarRocks Operator 和自定义资源 StarRocksCluster 的定义。
1.    添加 Helm Chart 仓库。

      ```Bash
      helm repo add starrocks-community https://starrocks.github.io/starrocks-kubernetes-operator
      ```

2.    更新 Helm Chart 仓库至最新版本。

      ```Bash
      helm repo update
      ```

3.    查看您已添加的 Helm Chart 仓库。

      ```Bash
      $ helm search repo starrocks-community
      NAME                                    CHART VERSION    APP VERSION  DESCRIPTION
      starrocks-community/kube-starrocks      1.8.0            3.1-latest   kube-starrocks includes two subcharts, starrock...
      starrocks-community/operator            1.8.0            1.8.0        A Helm chart for StarRocks operator
      starrocks-community/starrocks           1.8.0            3.1-latest   A Helm chart for StarRocks cluster
      ```

2. 使用默认的 **[values.yaml](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/helm-charts/charts/kube-starrocks/values.yaml)** Helm Chart 部署 StarRocks Operator 和 StarRocks 集群，或创建一个 YAML 文件来自定义您的部署配置。
1.    使用默认配置部署
      执行以下命令来部署 StarRocks Operator 和由一个 FE 和一个 BE 组成的 StarRocks 集群：

      ```Bash
      $ helm install starrocks starrocks-community/kube-starrocks
      # 如果返回以下结果，则 StarRocks Operator 和 StarRocks 集群正在部署中。
      NAME: starrocks
      LAST DEPLOYED: Tue Aug 15 15:12:00 2023
      NAMESPACE: starrocks
      STATUS: deployed
      REVISION: 1
      TEST SUITE: None
      ```

2.    使用自定义配置部署
-       创建一个 YAML 文件，例如 **my-values.yaml**，并在该 YAML 文件中自定义 StarRocks Operator 和 StarRocks 集群的配置。有关支持的参数和说明，请参见 Helm Chart 默认 **[values.yaml](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/helm-charts/charts/kube-starrocks/values.yaml)** 中的注释。
-       运行以下命令，使用 **my-values.yaml** 中的自定义配置部署 StarRocks Operator 和 StarRocks 集群。

        ```Bash
        helm install -f my-values.yaml starrocks starrocks-community/kube-starrocks
        ```

   部署需要一段时间。在此期间，您可以使用部署命令返回结果中的提示命令来检查部署状态。默认的提示命令如下：

   ```Bash
   $ kubectl --namespace default get starrockscluster -l "cluster=kube-starrocks"
   # 如果返回以下结果，则部署已成功完成。
   NAME             FESTATUS   CNSTATUS   BESTATUS
   kube-starrocks   running               running
   ```

   您也可以运行 `kubectl get pods` 来检查部署状态。如果所有 Pods 都处于 `Running` 状态，并且 Pods 内的所有容器都是 `READY`，则部署已成功完成。

   ```Bash
   $ kubectl get pods
   NAME                                       READY   STATUS    RESTARTS   AGE
   kube-starrocks-be-0                        1/1     Running   0          2m50s
   kube-starrocks-fe-0                        1/1     Running   0          4m31s
   kube-starrocks-operator-69c5c64595-pc7fv   1/1     Running   0          4m50s
   ```

## 下一步

- 访问 StarRocks 集群

  您可以从 Kubernetes 集群内部和外部访问 StarRocks 集群。详细说明请参见[访问 StarRocks 集群](./sr_operator.md#access-starrocks-cluster)。

- 管理 StarRocks Operator 和 StarRocks 集群

-   如果您需要更新 StarRocks Operator 和 StarRocks 集群的配置，请参阅 [Helm Upgrade](https://helm.sh/docs/helm/helm_upgrade/)。
-   如果您需要卸载 StarRocks Operator 和 StarRocks 集群，请运行以下命令：

    ```bash
    helm uninstall starrocks
    ```

- 在 Artifact Hub 上搜索 StarRocks 维护的 Helm Chart

  请参阅 [kube-starrocks](https://artifacthub.io/packages/helm/kube-starrocks/kube-starrocks)。