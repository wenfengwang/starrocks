---
displayed_sidebar: "Chinese"
---

# 使用 Operator 部署 StarRocks

本主题介绍如何使用 StarRocks Operator 在 Kubernetes 集群上自动部署和管理 StarRocks 集群。

## 工作原理

![img](../assets/starrocks_operator.png)

## 开始之前

### 创建 Kubernetes 集群

您可以使用云托管的 Kubernetes 服务，例如[Amazon 弹性 Kubernetes 服务 (EKS)](https://aws.amazon.com/eks/?nc1=h_ls)或[Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine)集群，也可以使用自托管的 Kubernetes 集群。

- 创建 Amazon EKS 集群

  1. 检查环境中已安装[以下命令行工具](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)：
     1. 安装并配置 AWS 命令行工具 AWS CLI。
     2. 安装 EKS 集群命令行工具 eksctl。
     3. 安装 Kubernetes 集群命令行工具 kubectl。
  2. 使用以下方法之一创建 EKS 集群：
     1. [使用 eksctl 快速创建 EKS 集群](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)。
     2. [使用 AWS 控制台和 AWS CLI 手动创建 EKS 集群](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)。

- 创建 GKE 集群

  在开始创建 GKE 集群之前，请确保完成所有的[先决条件](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#before-you-begin)。然后按照[创建 GKE 集群](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster#create_cluster)中提供的说明来创建 GKE 集群。

- 创建自托管的 Kubernetes 集群

  按照[使用 kubeadm 引导集群](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)中提供的说明来创建自托管的 Kubernetes 集群。您可以使用[Minikube](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)和[Docker Desktop](https://docs.docker.com/desktop/)以最少的步骤创建单节点私有 Kubernetes 集群。

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

      StarRocks Operator 部署到 `starrocks` 命名空间并管理所有命名空间下的 StarRocks 集群。
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

3. 检查 StarRocks Operator 的运行状态。如果 Pod 处于 `Running` 状态，并且 Pod 内的所有容器都处于 `READY` 状态，则 StarRocks Operator 正常运行。

    ```bash
    $ kubectl -n starrocks get pods
    NAME                                  READY   STATUS    RESTARTS   AGE
    starrocks-controller-65bb8679-jkbtg   1/1     Running   0          5m6s
    ```

> **注意**
>
> 如果您自定义了 StarRocks Operator 的所在命名空间，您需要将`starrocks`替换为您自定义的命名空间名称。

## 部署 StarRocks 集群

您可以直接使用 StarRocks 提供的[示例配置文件](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)来部署 StarRocks 集群（使用自定义资源 StarRocksCluster 实例化的对象）。例如，您可以使用 **starrocks-fe-and-be.yaml** 部署包含三个 FE 节点和三个 BE 节点的 StarRocks 集群。

```bash
kubectl apply -f https://raw.githubusercontent.com/StarRocks/starrocks-kubernetes-operator/main/examples/starrocks/starrocks-fe-and-be.yaml
```

以下表格描述了 **starrocks-fe-and-be.yaml** 文件中几个重要字段。

| **字段** | **描述**                                              |
| ------- | ---------------------------------------------------- |
| Kind    | 对象的资源类型。值必须是 `StarRocksCluster`。         |
| Metadata| 元数据，其中嵌套了以下子字段：<ul><li>`name`: 对象的名称。每个对象名称唯一标识相同资源类型的对象。</li><li>`namespace`: 对象所属的命名空间。</li></ul> |
| Spec    | 对象的预期状态。`starRocksFeSpec`、`starRocksBeSpec` 和 `starRocksCnSpec` 是有效值。|

您也可以使用修改后的配置文件部署 StarRocks 集群。有关支持的字段和详细说明，请参见[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md)。

部署 StarRocks 集群需要一些时间。在此期间，您可以使用命令 `kubectl -n starrocks get pods` 来检查 StarRocks 集群的启动状态。如果所有的 Pod 都处于 `Running` 状态，并且 Pod 内的所有容器都处于 `READY` 状态，则 StarRocks 集群正常运行。

> **注意**
>
> 如果一些 Pod 长时间无法启动，您可以使用 `kubectl logs -n starrocks <pod_name>` 查看日志信息，或使用 `kubectl -n starrocks describe pod <pod_name>` 查看事件信息以定位问题。

## 管理 StarRocks 集群

### 访问 StarRocks 集群

StarRocks 集群的组件可以通过其关联的服务访问，例如 FE 服务。有关服务及其访问地址的详细描述，请参见[api.md](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/api.md)和[服务](https://kubernetes.io/docs/concepts/services-networking/service/)。

> **注意**
>
> - 默认情况下仅部署了 FE 服务。如果需要部署 BE 服务和 CN 服务，您需要在 StarRocks 集群配置文件中配置 `starRocksBeSpec` 和 `starRocksCnSpec`。
> - 服务的名称默认为 `<cluster name>-<component name>-service`，例如 `starrockscluster-sample-fe-service`。您还可以在每个组件的规范中指定服务名称。

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

2. 通过Kubernetes集群内的MySQL客户端访问StarRocks集群。

```Bash
mysql -h 10.100.162.xxx -P 9030 -uroot
```

#### 从Kubernetes集群外部访问StarRocks集群

从Kubernetes集群外部，您可以通过FE服务的负载均衡器或NodePort访问StarRocks集群。本主题以负载均衡器为例：

1. 运行命令`kubectl -n starrocks edit src starrockscluster-sample`来更新StarRocks集群配置文件，并将`starRocksFeSpec`的Service类型更改为`LoadBalancer`。
   
   ```YAML
   starRocksFeSpec:
     image: starrocks/fe-ubuntu:3.0-latest
     replicas: 3
     requests:
       cpu: 4
       memory: 16Gi
     service:
       type: LoadBalancer #指定为LoadBalancer
   ```

2. 获取FE服务向外部公开的IP地址`EXTERNAL-IP`及端口`PORT(S)`。

   ```Bash
   $ kubectl -n starrocks get svc
   NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                                                       AGE
   be-domain-search                     ClusterIP      None             <none>                                                                   9050/TCP                                                      127m
   fe-domain-search                     ClusterIP      None             <none>                                                                   9030/TCP                                                      129m
   starrockscluster-sample-fe-service   LoadBalancer   10.100.162.xxx   a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com   8030:30629/TCP,9020:32544/TCP,9030:32244/TCP,9010:32024/TCP   129m             
   ```

3. 登录到您的主机并通过MySQL客户端访问StarRocks集群。

   ```Bash
   mysql -h a7509284bf3784983a596c6eec7fc212-618xxxxxx.us-west-2.elb.amazonaws.com -P9030 -uroot
   ```

### 升级StarRocks集群

#### 升级BE节点

运行以下命令指定一个新的BE镜像文件，如 `starrocks/be-ubuntu:latest`：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"image":"starrocks/be-ubuntu:latest"}}}'
```

#### 升级FE节点

运行以下命令指定一个新的FE镜像文件，如 `starrocks/fe-ubuntu:latest`：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"image":"starrocks/fe-ubuntu:latest"}}}'
```

升级过程需要一段时间。您可以运行命令`kubectl -n starrocks get pods`来查看升级进度。

### 扩展StarRocks集群

本主题以扩展BE和FE集群为例。

#### 扩展BE集群

运行以下命令将BE集群扩展为9个节点：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksBeSpec":{"replicas":9}}}'
```

#### 扩展FE集群

运行以下命令将FE集群扩展为4个节点：

```bash
kubectl -n starrocks patch starrockscluster starrockscluster-sample --type='merge' -p '{"spec":{"starRocksFeSpec":{"replicas":4}}}'
```

扩展过程需要一段时间。您可以使用命令`kubectl -n starrocks get pods`来查看扩展进度。

### 为CN集群配置自动扩展

运行命令`kubectl -n starrocks edit src starrockscluster-sample`来配置CN集群的自动扩展策略。您可以指定CN的资源指标，如平均CPU利用率、平均内存使用率、弹性扩展阈值、上弹性扩展限制和下弹性扩展限制。上弹性扩展限制和下弹性扩展限制指定了弹性扩展允许的最大和最小CN数。

> **注意**
>
> 如果配置了CN集群的自动扩展策略，请从StarRocks集群配置文件的`starRocksCnSpec`中删除`replicas`字段。

Kubernetes还支持使用`behavior`根据业务场景自定义扩展行为，帮助您实现快速或缓慢的扩展或禁用扩展。有关自动扩展策略的更多信息，请参阅[水平Pod自动缩放](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)。

以下是StarRocks提供的一个[模板](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_cn.yaml) ，帮助您配置自动扩展策略：

```YAML
  starRocksCnSpec:
    image: starrocks/cn-ubuntu:latest
    limits:
      cpu: 16
      memory: 64Gi
    requests:
      cpu: 16
      memory: 64Gi
    autoScalingPolicy: # CN集群的自动扩展策略。
      maxReplicas: 10 # CN的最大数量设置为10。
      minReplicas: 1 # CN的最小数量设置为1。
      # 运算符根据以下字段创建HPA资源。
      # 有关更多信息，请参阅 https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/。
      hpaPolicy:
        metrics: # 资源指标
          - type: Resource
            resource:
              name: memory  # 指定CN的平均内存使用率作为资源指标。
              target:
                # 弹性扩展阈值为60%。
                # 当CN的平均内存利用率超过60%时，CN的数量增加以进行扩展。
                # 当CN的平均内存利用率低于60%时，CN的数量减少以进行缩减。
                averageUtilization: 60
                type: Utilization
          - type: Resource
            resource:
              name: cpu # 指定CN的平均CPU利用率作为资源指标。
              target:
                # 弹性扩展阈值为60%。
                # 当CN的平均CPU利用率超过60%时，CN的数量增加以进行扩展。
                # 当CN的平均CPU利用率低于60%时，CN的数量减少以进行缩减。
                averageUtilization: 60
                type: Utilization
        behavior: # 根据业务场景自定义的扩展行为，帮助您实现快速或缓慢的扩展或禁用扩展。
          scaleUp:
            policies:
              - type: Pods
                value: 1
                periodSeconds: 10
          scaleDown:
            selectPolicy: Disabled
```

以下表格描述了一些重要字段：

- 上弹性扩展限制和下弹性扩展限制。

```YAML
maxReplicas: 10 # CN的最大数量设置为10。
minReplicas: 1 # CN的最小数量设置为1。
```

- 弹性扩展阈值。

```YAML
# 例如，将CN的平均CPU利用率指定为资源指标。
# 弹性扩展阈值为60%。
# 当CN的平均CPU利用率超过60%时，CN的数量增加以进行扩展。
# 当CN的平均CPU利用率低于60%时，CN的数量减少以进行缩减。
- type: Resource
  resource:
    name: cpu
    target:
      averageUtilization: 60
```

## 常见问题解答

**问题描述：** 使用`kubectl apply -f xxx`安装自定义资源StarRocksCluster时出现错误`The CustomResourceDefinition 'starrocksclusters.starrocks.com' is invalid: metadata.annotations: Too long: must have at most 262144 bytes`。

**原因分析：** 每当使用`kubectl apply -f xxx`来创建或更新资源时，都会添加一个元数据注释`kubectl.kubernetes.io/last-applied-configuration`。该元数据注释以JSON格式记录了*last-applied-configuration*。`kubectl apply -f xxx`适用于大多数情况，但在罕见情况下，例如自定义资源的配置文件过大，可能导致元数据注释的大小超过限制。

**解决方案：** 如果首次安装自定义资源StarRocksCluster，建议使用`kubectl create -f xxx`。如果自定义资源StarRocksCluster已在环境中安装，并且需要更新其配置，则建议使用`kubectl replace -f xxx`。