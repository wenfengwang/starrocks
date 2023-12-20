---
displayed_sidebar: English
---

# 使用 Operator 部署 StarRocks

本文介绍如何使用 StarRocks Operator 在 Kubernetes 集群上自动化部署和管理 StarRocks 集群。

## 工作原理

![img](../assets/starrocks_operator.png)

## 开始之前

### 创建 Kubernetes 集群

您可以使用云托管的 Kubernetes 服务，例如 [Amazon Elastic Kubernetes Service (EKS)](https://aws.amazon.com/eks/?nc1=h_ls) 或 [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) 集群，或者自行管理的 Kubernetes 集群。

- 创建 Amazon EKS 集群

  1. 检查您的环境中是否安装了以下[命令行工具](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)：
     1. 安装并配置 AWS 命令行工具 AWS CLI。
     2. 安装 EKS 集群命令行工具 eksctl。
     3. 安装 Kubernetes 集群命令行工具 kubectl。
  2. 使用以下方法之一创建 EKS 集群：
     1. [使用 eksctl 快速创建 EKS 集群](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)。
     2. [使用 AWS 控制台和 AWS CLI 手动创建 EKS 集群](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)。

- 创建 GKE 集群

  在开始创建 GKE 集群之前，请确保满足所有[先决条件](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#before-you-begin)。然后按照[创建 GKE 集群](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#create_cluster)中提供的说明创建 GKE 集群。

- 创建自行管理的 Kubernetes 集群

  按照[使用 kubeadm 引导集群](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)中提供的说明创建自行管理的 Kubernetes 集群。您可以使用[Minikube](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)和[Docker Desktop](https://docs.docker.com/desktop/)以最少的步骤创建单节点私有 Kubernetes 集群。

### 部署 StarRocks Operator

1. 添加自定义资源 StarRocksCluster。

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/starrocks.com_starrocksclusters.yaml
   ```

2. 部署 StarRocks Operator。您可以选择使用默认配置文件或自定义配置文件来部署 StarRocks Operator。
   1. 使用默认配置文件部署 StarRocks Operator。

      ```bash
      kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
      ```

      StarRocks Operator 部署在 `starrocks` 命名空间中，管理所有命名空间下的 StarRocks 集群。

   2. 使用自定义配置文件部署 StarRocks Operator。
      - 下载配置文件 **operator.yaml**，用于部署 StarRocks Operator。

        ```bash
        curl -O https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/deploy/operator.yaml
        ```

      - 修改配置文件 **operator.yaml** 以满足您的需求。
      - 部署 StarRocks Operator。

        ```bash
        kubectl apply -f operator.yaml
        ```

3. 检查 StarRocks Operator 的运行状态。如果 Pod 处于 `Running` 状态并且 Pod 内的所有容器都已准备就绪，表示 StarRocks Operator 正常运行。

   ```bash
   $ kubectl -n starrocks get pods
   NAME                                  READY   STATUS    RESTARTS   AGE
   starrocks-controller-65bb8679-jkbtg   1/1     Running   0          5m6s
   ```

> **注意**
> 如果您自定义了 StarRocks Operator 所在的命名空间，请将 `starrocks` 替换为您自定义的命名空间名称。

## 部署 StarRocks 集群

您可以直接使用 StarRocks 提供的[示例配置文件](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)来部署 StarRocks 集群（使用自定义资源 StarRocksCluster 实例化的对象）。例如，您可以使用 **starrocks-fe-and-be.yaml** 部署一个包含三个 FE 节点和三个 BE 节点的 StarRocks 集群。

```bash
kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/examples/starrocks/starrocks-fe-and-be.yaml
```

下表描述了 **starrocks-fe-and-be.yaml** 文件中的一些重要字段。

|字段|描述|
|---|---|
|Kind|对象的资源类型。该值必须为 `StarRocksCluster`。|
|Metadata|元数据，其中嵌套了以下子字段：<ul><li>`name`：对象的名称。每个对象名称唯一标识相同资源类型的对象。</li><li>`namespace`：对象所属的命名空间。</li></ul>|
|Spec|对象的预期状态。有效值为 `starRocksFeSpec`、`starRocksBeSpec` 和 `starRocksCnSpec`。|

您还可以使用修改后的配置文件来部署 StarRocks 集群。有关支持的字段和详细说明，请参见 [api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md)。

部署 StarRocks 集群需要一些时间。在此期间，您可以使用命令 `kubectl -n starrocks get pods` 来检查 StarRocks 集群的启动状态。如果所有 Pod 都处于 `Running` 状态并且 Pod 内的所有容器都已准备就绪，表示 StarRocks 集群正常运行。

> **注意**
> 如果您自定义了集群所在的命名空间，请将 `starrocks` 替换为您自定义的命名空间名称。

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
> 如果一些 Pod 长时间无法启动，您可以使用 `kubectl logs -n starrocks <pod_name>` 命令查看日志信息，或使用 `kubectl -n starrocks describe pod <pod_name>` 命令查看事件信息以定位问题。

## 管理 StarRocks 集群

### 访问 StarRocks 集群

StarRocks 集群的组件可以通过关联的服务进行访问，例如 FE 服务。有关服务及其访问地址的详细说明，请参见 [api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md) 和 [Services](https://kubernetes.io/docs/concepts/services-networking/service/)。

> **注意**
> 默认情况下，仅部署 FE 服务。如果需要部署 BE 服务和 CN 服务，请在 StarRocks 集群配置文件中配置 `starRocksBeSpec` 和 `starRocksCnSpec`。

#### 从 Kubernetes 集群内部访问 StarRocks 集群

在 Kubernetes 集群内部，可以通过 FE 服务的 ClusterIP 访问 StarRocks 集群。

1. 获取 FE 服务的内部虚拟 IP 地址 `CLUSTER-IP` 和端口 `PORT(S)`。

   ```Bash
   $ kubectl -n starrocks get svc 
   NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
   be-domain-search                     ClusterIP   None             <none>        9050/TCP                              23m
   fe-domain-search                     ClusterIP   None             <none>        9030/TCP                              25m
   starrockscluster-sample-fe-service   ClusterIP   10.100.162.xxx   <none>        8030/TCP,9020/TCP,9030/TCP,9010/TCP   25m
   ```

2. 在 Kubernetes 集群内部使用 MySQL 客户端访问 StarRocks 集群。

   ```Bash
   mysql -h 10.100.162.xxx -P 9030 -uroot
   ```

#### 从 Kubernetes 集群外部访问 StarRocks 集群

在 Kubernetes 集群外部，可以通过 FE 服务的 LoadBalancer 或 NodePort 访问 StarRocks 集群。本文以 LoadBalancer 为例：

1. 运行命令 `kubectl -n starrocks edit src starrockscluster-sample` 更新 StarRocks 集群配置文件，将 `starRocksFeSpec` 的 Service 类型更改为 `LoadBalancer`。

   ```YAML
   starRocksFeSpec:
     image: starrocks/fe-ubuntu:3.0-latest
     replicas: 3
     requests:
       cpu: 4
       memory: 16Gi
     service:            
       type: LoadBalancer # 指定为 LoadBalancer
   ```

2. 获取 FE 服务向外部公开的 IP 地址 `EXTERNAL-IP` 和端口 `PORT(S)`。

   ```Bash
   $ kubectl -n starrocks get svc
   NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                                                       AGE
   be-domain-search                     ClusterIP      None             <none>                                                                   9050/TCP                                                      127m
   fe-domain-search                     ClusterIP      None             <none>                                                                   9030/TCP                                                      129m
   starrockscluster-sample-fe-service   LoadBalancer   10.100.162.xxx   a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com   8030:30629/TCP,9020:32544/TCP,9030:32244/TCP,9010:32024/TCP   129m
   ```

3. 在本地主机上登录并使用 MySQL 客户端访问 StarRocks 集群。

   ```Bash
   mysql -h a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com -P 9030 -uroot
   ```

### 升级 StarRocks 集群

#### 升级 BE 节点

运行以下命令指定新的 BE 镜像文件，例如 `starrocks/be-ubuntu:latest`：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"image":"starrocks/be-ubuntu:latest"}}}'
```

#### 升级 FE 节点

运行以下命令指定新的 FE 镜像文件，例如 `starrocks/fe-ubuntu:latest`：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"image":"starrocks/fe-ubuntu:latest"}}}'
```

升级过程需要一段时间。您可以运行命令 `kubectl -n starrocks get pods` 查看升级进度。

### 扩展 StarRocks 集群

本主题以扩展 BE 集群和 FE 集群为例。

#### 扩展 BE 集群

运行以下命令将 BE 集群扩展到 9 个节点：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"replicas":9}}}'
```

#### 扩展 FE 集群

运行以下命令将 FE 集群扩展到 4 个节点：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"replicas":4}}}'
```

扩展过程需要一段时间。您可以使用命令 `kubectl -n starrocks get pods` 查看扩展进度。

### CN 集群的自动扩展

运行命令 `kubectl -n starrocks edit src starrockscluster-sample` 配置 CN 集群的自动扩展策略。您可以指定 CN 的资源指标，例如平均 CPU 利用率、平均内存使用量、弹性扩展阈值、上限弹性扩展限制和下限弹性扩展限制。上限弹性扩展限制和下限弹性扩展限制指定了弹性扩展允许的 CN 的最大数量和最小数量。

> **注意**
> 如果配置了 CN 集群的自动扩展策略，请从 StarRocks 集群配置文件的 `starRocksCnSpec` 中删除 `replicas` 字段。

Kubernetes 还支持使用 `behavior` 根据业务场景自定义扩展行为，帮助您实现快速或缓慢扩展或禁用扩展。有关自动扩展策略的更多信息，请参见[Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)。

以下是 StarRocks 提供的[模板](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_cn.yaml)，可帮助您配置自动扩展策略：

```YAML
starRocksCnSpec:
  image: starrocks/cn-ubuntu:latest
  limits:
    cpu: 16
    memory: 64Gi
  requests:
    cpu: 16
    memory: 64Gi
  autoScalingPolicy: # CN 集群的自动扩展策略。
    maxReplicas: 10 # 最大 CN 数量设置为 10。
    minReplicas: 1 # 最小 CN 数量设置为 1。
    # 运算符根据以下字段创建 HPA 资源。
    # 有关更多信息，请参见 https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/。
    hpaPolicy:
      metrics: # 资源指标
        - type: Resource
          resource:
            name: memory  # 平均内存使用量作为资源指标。
            target:
              # 弹性扩展阈值为 60%。
              # 当 CN 的平均内存利用率超过 60% 时，CN 的数量增加以进行扩展。
              # 当 CN 的平均内存利用率低于 60% 时，CN 的数量减少以进行缩减。
              averageUtilization: 60
              type: Utilization
        - type: Resource
          resource:
            name: cpu # 平均 CPU 利用率作为资源指标。
            target:
              # 弹性扩展阈值为 60%。
              # 当 CN 的平均 CPU 利用率超过 60% 时，CN 的数量增加以进行扩展。
              # 当 CN 的平均 CPU 利用率低于 60% 时，CN 的数量减少以进行缩减。
              averageUtilization: 60
              type: Utilization
      behavior: # 根据业务场景自定义扩展行为，帮助您实现快速或缓慢扩展或禁用扩展。
        scaleUp:
          policies:
            - type: Pods
              value: 1
              periodSeconds: 10
        scaleDown:
          selectPolicy: Disabled
```

以下是一些重要字段的说明：

- 上限和下限弹性扩展限制。

```YAML
maxReplicas: 10 # 最大 CN 数量设置为 10。
minReplicas: 1 # 最小 CN 数量设置为 1。
```

- 弹性扩展阈值。

```YAML
# 例如，平均 CPU 利用率作为资源指标。
# 弹性扩展阈值为 60%。
# 当 CN 的平均 CPU 利用率超过 60% 时，CN 的数量增加以进行扩展。
# 当 CN 的平均 CPU 利用率低于 60% 时，CN 的数量减少以进行缩减。
- type: Resource
  resource:
    name: cpu
    target:
      averageUtilization: 60
```

## 常见问题

**问题描述：**使用 `kubectl apply -f xxx` 安装自定义资源 StarRocksCluster 时，返回错误 `The CustomResourceDefinition 'starrocksclusters.starrocks.com' is invalid: metadata.annotations: Too long: must have at most 262144 bytes`。

**原因分析：**每当使用 `kubectl apply -f xxx` 创建或更新资源时，都会添加元数据注释 `kubectl.kubernetes.io/last-applied-configuration`。该元数据注释以 JSON 格式记录了“last-applied-configuration”。`kubectl apply -f xxx` 适用于大多数情况，但在某些情况下（例如自定义资源的配置文件过大），可能导致元数据注释的大小超过限制。

**解决方案：**如果首次安装自定义资源 StarRocksCluster，请建议使用 `kubectl create -f xxx`。如果环境中已经安装了自定义资源 StarRocksCluster，并且需要更新其配置，请建议使用 `kubectl replace -f xxx`。
```
```YAML
starRocksFeSpec:
  image: starrocks/fe-ubuntu:3.0-latest
  replicas: 3
  requests:
    cpu: 4
    memory: 16Gi
  service:            
    type: LoadBalancer # 指定为 LoadBalancer
```

2. 获取 FE 服务对外公开的 IP 地址 `EXTERNAL-IP` 和端口 `PORT(S)`。

   ```Bash
   $ kubectl -n starrocks get svc
   NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                                                       AGE
   be-domain-search                     ClusterIP      None             <none>                                                                   9050/TCP                                                      127m
   fe-domain-search                     ClusterIP      None             <none>                                                                   9030/TCP                                                      129m
   starrockscluster-sample-fe-service   LoadBalancer   10.100.162.xxx   a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com   8030:30629/TCP,9020:32544/TCP,9030:32244/TCP,9010:32024/TCP   129m
   ```

3. 登录您的机器宿主机，使用 MySQL 客户端访问 StarRocks 集群。

   ```Bash
   mysql -h a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com -P 9030 -uroot
   ```

### 升级 StarRocks 集群

#### 升级 BE 节点

执行以下命令指定新的 BE 镜像文件，例如 `starrocks/be-ubuntu:latest`：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"image":"starrocks/be-ubuntu:latest"}}}'
```

#### 升级 FE 节点

执行以下命令指定新的 FE 镜像文件，例如 `starrocks/fe-ubuntu:latest`：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"image":"starrocks/fe-ubuntu:latest"}}}'
```

升级过程会持续一段时间。您可以运行命令 `kubectl -n starrocks get pods` 查看升级进度。

### 扩展 StarRocks 集群

本节以 BE、FE 集群扩容为例。

#### 横向扩展 BE 集群

运行以下命令将 BE 集群扩展到 9 个节点：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"replicas":9}}}'
```

#### 横向扩展 FE 集群

运行以下命令将 FE 集群横向扩展至 4 个节点：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"replicas":4}}}'
```

扩展过程会持续一段时间。您可以使用命令 `kubectl -n starrocks get pods` 查看扩展进度。

### CN 集群自动伸缩

执行命令 `kubectl -n starrocks edit src starrockscluster-sample` 配置 CN 集群的自动伸缩策略。您可以指定 CN 的资源指标，如平均 CPU 利用率、平均内存利用率、弹性伸缩阈值、弹性伸缩上限和弹性伸缩下限。弹性伸缩上限和弹性伸缩下限指定了弹性伸缩允许的 CN 数量的最大值和最小值。

> **注意**
> 如果 CN 集群配置了自动伸缩策略，请从 StarRocks 集群配置文件中的 `starRocksCnSpec` 删除 `replicas` 字段。

Kubernetes 还支持使用 `behavior` 来定制根据业务场景的伸缩行为，帮助您实现快速或慢速伸缩，或禁用伸缩。有关自动伸缩策略的更多信息，请参阅 [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)。

以下是 StarRocks 提供的[模板](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_cn.yaml)，帮助您配置自动伸缩策略：

```YAML
starRocksCnSpec:
  image: starrocks/cn-ubuntu:latest
  limits:
    cpu: 16
    memory: 64Gi
  requests:
    cpu: 16
    memory: 64Gi
  autoScalingPolicy: # CN 集群的自动伸缩策略
    maxReplicas: 10 # 设置 CN 的最大数量为 10
    minReplicas: 1 # 设置 CN 的最小数量为 1
    # operator 根据以下字段创建 HPA 资源
    # 有关更多信息，请参阅 https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
    hpaPolicy:
      metrics: # 资源指标
        - type: Resource
          resource:
            name: memory # 将 CN 的平均内存利用率指定为资源指标
            target:
              # 弹性伸缩阈值为 60%
              # 当 CN 的平均内存利用率超过 60% 时，CN 数量增加以进行扩展
              # 当 CN 的平均内存利用率低于 60% 时，CN 数量减少以进行缩减
              averageUtilization: 60
              type: Utilization
        - type: Resource
          resource:
            name: cpu # 将 CN 的平均 CPU 利用率指定为资源指标
            target:
              # 弹性伸缩阈值为 60%
              # 当 CN 的平均 CPU 利用率超过 60% 时，CN 数量增加以进行扩展
              # 当 CN 的平均 CPU 利用率低于 60% 时，CN 数量减少以进行缩减
              averageUtilization: 60
              type: Utilization
      behavior: # 根据业务场景定制的伸缩行为，帮助实现快速或慢速伸缩，或禁用伸缩
        scaleUp:
          policies:
            - type: Pods
              value: 1
              periodSeconds: 10
        scaleDown:
          selectPolicy: Disabled
```

下表描述了几个重要字段：

- 弹性伸缩的上限和下限。

```YAML
maxReplicas: 10 # 设置 CN 的最大数量为 10
minReplicas: 1 # 设置 CN 的最小数量为 1
```

- 弹性伸缩阈值。

```YAML
# 例如，将 CN 的平均 CPU 利用率指定为资源指标
# 弹性伸缩阈值为 60%
# 当 CN 的平均 CPU 利用率超过 60% 时，CN 数量增加以进行扩展
# 当 CN 的平均 CPU 利用率低于 60% 时，CN 数量减少以进行缩减
- type: Resource
  resource:
    name: cpu
    target:
      averageUtilization: 60
```

## 常见问题解答

**问题描述：** 当使用 `kubectl apply -f xxx` 安装自定义资源 StarRocksCluster 时，返回错误 `CustomResourceDefinition 'starrocksclusters.starrocks.com' 无效：metadata.annotations：太长：必须最多 262144 字节`。

**原因分析：** 每当使用 `kubectl apply -f xxx` 创建或更新资源时，都会添加一个元数据注释 `kubectl.kubernetes.io/last-applied-configuration`。这个元数据注释是 JSON 格式的，记录了 *last-applied-configuration*。`kubectl apply -f xxx` 适用于大多数情况，但在少数情况下，例如当自定义资源的配置文件过大时，可能导致元数据注释的大小超过限制。

**解决方案：** 如果您是首次安装自定义资源 StarRocksCluster，建议使用 `kubectl create -f xxx`。如果环境中已经安装了自定义资源 StarRocksCluster，并且需要更新其配置，建议使用 `kubectl replace -f xxx`。
**原因分析：**当使用`kubectl apply -f xxx`来创建或更新资源时，会添加一个名为`kubectl.kubernetes.io/last-applied-configuration`的元数据注解。这个注解是JSON格式的，记录了*最后一次应用的配置*。`kubectl apply -f xxx`适用于大多数情况，但在极少数情况下，例如当自定义资源的配置文件过大时，可能会导致元数据注释的大小超过限制。

**解决方案：**如果是首次安装自定义资源StarRocksCluster，建议使用`kubectl create -f xxx`。如果环境中已经安装了自定义资源StarRocksCluster，并且需要更新其配置，建议使用`kubectl replace -f xxx`。