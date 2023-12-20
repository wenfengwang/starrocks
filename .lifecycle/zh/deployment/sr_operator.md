---
displayed_sidebar: English
---

# 与 Operator 配合部署 StarRocks

本主题介绍如何使用 StarRocks Operator 在 Kubernetes 集群上自动化部署和管理 StarRocks 集群。

## 工作原理

![img](../assets/starrocks_operator.png)

## 开始之前

### 创建 Kubernetes 集群

您可以使用云托管的 Kubernetes 服务，例如 Amazon Elastic Kubernetes Service（[EKS](https://aws.amazon.com/eks/?nc1=h_ls)）或 Google Kubernetes Engine（[GKE](https://cloud.google.com/kubernetes-engine)）集群，或者自行管理的 Kubernetes 集群。

- 创建 Amazon EKS 集群

  1. 请检查您的环境中是否安装了[以下命令行工具](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)：
     1. 安装并配置 AWS 命令行工具 AWS CLI。
     2. 安装 EKS 集群命令行工具 eksctl。
     3. 安装 Kubernetes 集群命令行工具 kubectl。
  2. 使用以下方法之一来创建 EKS 集群：
     1. [使用 eksctl 快速创建 EKS 集群](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)。
     2. [通过 AWS 控制台和 AWS CLI 手动创建 EKS 集群](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)。

- 创建 GKE 集群

  在您开始创建 GKE 集群之前，请确保您已经完成所有的[先决条件](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#before-you-begin)。然后按照[创建 GKE 集群](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#create_cluster)中提供的指南来创建 GKE 集群。

- 创建自行管理的 Kubernetes 集群

  按照“[使用 kubeadm 启动集群](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)”中提供的指南来创建自行管理的 Kubernetes 集群。您可以使用 [Minikube](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/) 和 [Docker Desktop](https://docs.docker.com/desktop/) 来创建一个单节点的私有 Kubernetes 集群，操作步骤非常简单。

### 部署 StarRocks Operator

1. 添加自定义资源 StarRocksCluster。

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/starrocks.com_starrocksclusters.yaml
   ```

2. 部署 StarRocks Operator。您可以选择使用默认配置文件或自定义配置文件部署 StarRocks Operator。
1.    使用默认配置文件部署 StarRocks Operator。

      ```bash
      kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
      ```
      StarRocks Operator 将部署在名为 starrocks 的命名空间中，并管理所有命名空间下的 StarRocks 集群。
2.    使用自定义配置文件部署 StarRocks Operator。
-       下载用于部署 StarRocks Operator 的配置文件 **operator.yaml**。

        ```bash
        curl -O https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
        ```

-       根据您的需求修改配置文件 **operator.yaml**。
-       部署 StarRocks Operator。

        ```bash
        kubectl apply -f operator.yaml
        ```

3. 检查 StarRocks Operator 的运行状态。如果 Pod 处于 Running 状态且 Pod 内的所有容器都处于 READY 状态，那么 StarRocks Operator 正在按预期运行。

   ```bash
   $ kubectl -n starrocks get pods
   NAME                                  READY   STATUS    RESTARTS   AGE
   starrocks-controller-65bb8679-jkbtg   1/1     Running   0          5m6s
   ```

> **注意**
> 如果您自定义了**StarRocks Operator**所在的命名空间，您需要将`starrocks`替换为您自定义的命名空间名称。

## 部署 StarRocks 集群

您可以直接使用[示例配置文件](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)提供的 by StarRocks to deploy a StarRocks cluster（通过使用自定义资源StarRocks Cluster实例化的对象）。例如，您可以使用**starrocks-fe-and-be.yaml**部署一个包含三个FE节点和三个BE节点的StarRocks集群。

```bash
kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/examples/starrocks/starrocks-fe-and-be.yaml
```

下表描述了**starrocks-fe-and-be.yaml**文件中的一些重要字段。

|字段|描述|
|---|---|
|种类|对象的资源类型。该值必须是 StarRocksCluster。|
|Metadata|元数据，其中嵌套了以下子字段：name：对象的名称。每个对象名称唯一标识同一资源类型的对象。命名空间: 对象所属的命名空间。|
|Spec|对象的预期状态。有效值为 starRocksFeSpec、starRocksBeSpec 和 starRocksCnSpec。|

您也可以使用修改后的配置文件部署 StarRocks 集群。支持的字段和详细描述请参见 [api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md)。

部署 StarRocks 集群需要一些时间。在此期间，您可以使用命令 kubectl -n starrocks get pods 来检查 StarRocks 集群的启动状态。如果所有 Pod 都处于 Running 状态并且 Pod 内的所有容器都处于 READY 状态，那么 StarRocks 集群正在按预期运行。

> **注意**
> 如果您自定义了**StarRocks**集群所在的命名空间，您需要将 `starrocks` 替换为您自定义的命名空间名称。

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
> 如果某些 Pod 在较长时间后仍无法启动，您可以使用命令 `kubectl logs -n starrocks \\u003cpod_name\\u003e` 来查看日志信息，或使用 `kubectl -n starrocks describe pod \\u003cpod_name\\u003e` 来查看事件信息，以便定位问题。

## 管理 StarRocks 集群

### 访问 StarRocks 集群

StarRocks 集群的组件可以通过其关联的服务访问，例如 FE 服务。有关服务及其访问地址的详细描述，请参见 [api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md) 和 [Services](https://kubernetes.io/docs/concepts/services-networking/service/)。

> **注意**
- 默认情况下，只有 FE 服务被部署。如果您需要部署 BE 服务和 CN 服务，您需要在 StarRocks 集群配置文件中配置 starRocksBeSpec 和 starRocksCnSpec。
- 服务的默认名称格式为 <集群名称>-<组件名称>-service，例如 starrockscluster-sample-fe-service。您也可以在每个组件的 spec 中指定服务名称。

#### 从 Kubernetes 集群内部访问 StarRocks 集群

在 Kubernetes 集群内部，可以通过 FE 服务的 ClusterIP 访问 StarRocks 集群。

1. 获取 FE 服务的内部虚拟 IP 地址 CLUSTER-IP 和端口 PORT(S)。

   ```Bash
   $ kubectl -n starrocks get svc 
   NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
   be-domain-search                     ClusterIP   None             <none>        9050/TCP                              23m
   fe-domain-search                     ClusterIP   None             <none>        9030/TCP                              25m
   starrockscluster-sample-fe-service   ClusterIP   10.100.162.xxx   <none>        8030/TCP,9020/TCP,9030/TCP,9010/TCP   25m
   ```

2. 使用 Kubernetes 集群内部的 MySQL 客户端访问 StarRocks 集群。

   ```Bash
   mysql -h 10.100.162.xxx -P 9030 -uroot
   ```

#### 从 Kubernetes 集群外部访问 StarRocks 集群

从 Kubernetes 集群外部，您可以通过 FE 服务的 LoadBalancer 或 NodePort 访问 StarRocks 集群。以 LoadBalancer 为例：

1. 执行命令 kubectl -n starrocks edit src starrockscluster-sample 以更新 StarRocks 集群配置文件，并将 starRocksFeSpec 的服务类型更改为 LoadBalancer。

   ```YAML
   starRocksFeSpec:
     image: starrocks/fe-ubuntu:3.0-latest
     replicas: 3
     requests:
       cpu: 4
       memory: 16Gi
     service:            
       type: LoadBalancer # specified as LoadBalancer
   ```

2. 获取 FE 服务暴露给外部的 IP 地址 EXTERNAL-IP 和端口 PORT(S)。

   ```Bash
   $ kubectl -n starrocks get svc
   NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                                                       AGE
   be-domain-search                     ClusterIP      None             <none>                                                                   9050/TCP                                                      127m
   fe-domain-search                     ClusterIP      None             <none>                                                                   9030/TCP                                                      129m
   starrockscluster-sample-fe-service   LoadBalancer   10.100.162.xxx   a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com   8030:30629/TCP,9020:32544/TCP,9030:32244/TCP,9010:32024/TCP   129m               ClusterIP      None            <none>                                                                   9030/TCP                                                      23h
   ```

3. 登录您的机器主机，使用 MySQL 客户端访问 StarRocks 集群。

   ```Bash
   mysql -h a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com -P9030 -uroot
   ```

### 升级 StarRocks 集群

#### 升级 BE 节点

执行以下命令以指定新的 BE 镜像文件，例如 starrocks/be-ubuntu:latest：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"image":"starrocks/be-ubuntu:latest"}}}'
```

#### 升级 FE 节点

执行以下命令以指定新的 FE 镜像文件，例如 starrocks/fe-ubuntu:latest：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"image":"starrocks/fe-ubuntu:latest"}}}'
```

升级过程将持续一段时间。您可以运行命令 kubectl -n starrocks get pods 来查看升级进度。

### 扩展 StarRocks 集群

本节以扩展 BE 和 FE 集群为例。

#### 扩展 BE 集群

执行以下命令将 BE 集群扩展到 9 个节点：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"replicas":9}}}'
```

#### 扩展 FE 集群

执行以下命令将 FE 集群扩展到 4 个节点：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"replicas":4}}}'
```

扩展过程将持续一段时间。您可以使用命令 kubectl -n starrocks get pods 来查看扩展进度。

### CN 集群自动扩展

执行命令 kubectl -n starrocks edit src starrockscluster-sample 来配置 CN 集群的自动扩展策略。您可以指定 CN 的资源指标，例如平均 CPU 利用率、平均内存使用量、弹性扩展阈值、弹性扩展上限和弹性扩展下限。弹性扩展上限和弹性扩展下限分别指定了允许弹性扩展的 CN 的最大数量和最小数量。

> **注意**
> 如果配置了 CN 集群的自动扩展策略，请从 StarRocks 集群配置文件的 `starRocksCnSpec` 中删除 `replicas` 字段。

Kubernetes 还支持使用`behavior`来根据业务场景定制扩展行为，帮助您实现快速或缓慢扩展或禁用扩展。有关自动扩展策略的更多信息，请参阅[Horizontal Pod Scaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)。

以下是 StarRocks 提供的[模板](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_cn.yaml)，用于帮助您配置自动扩展策略：

```YAML
  starRocksCnSpec:
    image: starrocks/cn-ubuntu:latest
    limits:
      cpu: 16
      memory: 64Gi
    requests:
      cpu: 16
      memory: 64Gi
    autoScalingPolicy: # Automatic scaling policy of the CN cluster.
      maxReplicas: 10 # The maximum number of CNs is set to 10.
      minReplicas: 1 # The minimum number of CNs is set to 1.
      # operator creates an HPA resource based on the following field.
      # see https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/ for more information.
      hpaPolicy:
        metrics: # Resource metrics
          - type: Resource
            resource:
              name: memory  # The average memory usage of CNs is specified as a resource metric.
              target:
                # The elastic scaling threshold is 60%.
                # When the average memory utilization of CNs exceeds 60%, the number of CNs increases for scale-out.
                # When the average memory utilization of CNs is below 60%, the number of CNs decreases for scale-in.
                averageUtilization: 60
                type: Utilization
          - type: Resource
            resource:
              name: cpu # The average CPU utilization of CNs is specified as a resource metric.
              target:
                # The elastic scaling threshold is 60%.
                # When the average CPU utilization of CNs exceeds 60%, the number of CNs increases for scale-out.
                # When the average CPU utilization of CNs is below 60%, the number of CNs decreases for scale-in.
                averageUtilization: 60
                type: Utilization
        behavior: #  The scaling behavior is customized according to business scenarios, helping you achieve rapid or slow scaling or disable scaling.
          scaleUp:
            policies:
              - type: Pods
                value: 1
                periodSeconds: 10
          scaleDown:
            selectPolicy: Disabled
```

下表描述了一些重要字段：

- 弹性扩展上限和下限。

```YAML
maxReplicas: 10 # The maximum number of CNs is set to 10.
minReplicas: 1 # The minimum number of CNs is set to 1.
```

- 弹性扩展阈值。

```YAML
# For example, the average CPU utilization of CNs is specified as a resource metric.
# The elastic scaling threshold is 60%.
# When the average CPU utilization of CNs exceeds 60%, the number of CNs increases for scale-out.
# When the average CPU utilization of CNs is below 60%, the number of CNs decreases for scale-in.
- type: Resource
  resource:
    name: cpu
    target:
      averageUtilization: 60
```

## 常见问题解答

**问题描述：**使用 `kubectl apply -f xxx` 安装自定义资源 StarRocksCluster 时，返回错误 `CustomResourceDefinition 'starrocksclusters.starrocks.com' 无效：metadata.annotations 太长，必须最多为 262144 字节`。

**原因分析：** 每当使用 `kubectl apply -f xxx` 创建或更新资源时，都会添加一个元数据注释 `kubectl.kubernetes.io/last-applied-configuration`。这个元数据注释以 JSON 格式记录了*最后应用的配置*。`kubectl apply -f xxx`" 适用于大多数情况，但在极少数情况下，如当自定义资源的配置文件过大时，可能会导致元数据注释的大小超过限制。

**解决方案:** 如果您是第一次安装自定义资源 StarRocksCluster，建议使用 `kubectl create -f xxx`。如果环境中已经安装了自定义资源 StarRocksCluster，并且您需要更新其配置，建议使用 `kubectl replace -f xxx`。
