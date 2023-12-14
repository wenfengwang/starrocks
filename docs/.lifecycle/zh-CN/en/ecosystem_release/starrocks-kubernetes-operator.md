---
displayed_sidebar: "Chinese"
---

# starrocks-kubernetes-operator

## 通知

StarRocks提供的Operator用于在Kubernetes环境中部署StarRocks集群。StarRocks集群组件包括FE、BE和CN。

**用户指南：** 您可以使用以下方法在Kubernetes上部署StarRocks集群：

- [直接使用StarRocks CRD来部署StarRocks集群](https://docs.starrocks.io/docs/deployment/sr_operator/)
- [使用Helm Chart部署Operator和StarRocks集群](https://docs.starrocks.io/zh/docs/deployment/helm/)

**源代码:**

- [starrocks-kubernetes-operator](https://github.com/StarRocks/starrocks-kubernetes-operator)
- [kube-starrocks Helm Chart](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks)

**资源的下载链接:**

- **URL前缀**:

    `https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v${operator_version}/${resource_name}`

- **资源名称**

  - StarRocksCluster CRD: `starrocks.com_starrocksclusters.yaml`
  - StarRocks Operator的默认配置文件: `operator.yaml`
  - 包括`kube-starrocks` Chart `kube-starrocks-${chart_version}.tgz` 在内的Helm Chart。`kube-starrocks` Chart 被分成两个子chart: `starrocks` Chart `starrocks-${chart_version}.tgz` 和 `operator` Chart `operator-${chart_version}.tgz`。

例如，kube-starrocks chart v1.8.6 的下载链接是：[kube-starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v1.8.6/kube-starrocks-1.8.6.tgz)

**版本要求**

- Kubernetes: 1.18或更高版本
- Go: 1.19或更高版本

## **发布说明**

### 1.8

**1.8.6**

**Bug修复**

修复了以下问题：

- 在进行Stream Load作业时返回错误 `sendfile() failed (32: Broken pipe) while sending request to upstream` 。当Nginx将请求体发送到FE时，FE会将请求重定向到BE。此时，Nginx中缓存的数据可能已丢失。[#303](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/303)

**文档**

- [通过FE代理从Kubernetes网络外部加载数据到StarRocks](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/load_data_using_stream_load_howto.md)
- [使用Helm更新root用户的密码](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/change_root_password_howto.md)

**1.8.5**

**改进**

- **[Helm Chart]】可以为Operator的service account自定义`annotations`和`labels`**: Operator默认创建名为`starrocks`的service account，用户可以通过在**values.yaml**中指定`serviceAccount`的`annotations`和`labels`字段来自定义operator的service account的注解和标签。 `operator.global.rbac.serviceAccountName`字段已被弃用。[#291](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/291)
- **[Operator]】FE服务支持为Istio的流量选择显式的协议**: 当Kubernetes环境中安装了Istio时，Istio需要确定来自StarRocks集群的流量的协议，以提供路由和丰富的指标等额外功能。因此，FE服务明确将其协议定义为`MySQL`，此改进特别重要，因为MySQL协议是一种服务端优先的协议，不兼容自动协议检测，有时可能导致连接失败。[#288](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/288)

**Bug修复**

- **[Helm Chart]】当`starrocks.initPassword.enabled`为true并且指定了`starrocks.starrocksCluster.name`的值时，StarRocks中的root用户密码可能无法成功初始化。问题原因是initpwd pod连接FE服务时使用了错误的FE服务域名。具体来说，在此场景下，FE服务域名使用`starrocks.starrocksCluster.name`中指定的值，而initpwd pod仍然使用`starrocks.nameOverride`字段的值来组成FE服务域名。 ([#292](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/292))

**升级说明**

- **[Helm Chart]】当`starrocks.starrocksCluster.name`中指定的值与`starrocks.nameOverride`的值不同时，FE、BE和CN的旧configmaps将被删除，新的带有新名称的configmaps将被创建。**这可能导致FE、BE和CN pod的重启。

**1.8.4**

**功能**

- **[Helm Chart]】通过Prometheus和ServiceMonitor CR，可以监控StarRocks集群的指标。用户指南请参见[与Prometheus和Grafana集成](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-prometheus-grafana.md)。[#284](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/284)
- **[Helm Chart]】在**values.yaml**文件中的`starrocksCnSpec`中添加`storagespec`和更多的字段以配置StarRocks集群中CN节点的日志卷。[#280](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/280)
- 在StarRocksCluster CRD中添加`terminationGracePeriodSeconds`字段，以配置在删除或更新StarRocksCluster资源时等待强制终止pod的时间。[#283](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/283)
- 在StarRocksCluster CRD中添加`startupProbeFailureSeconds`字段，以配置StarRocksCluster资源中pod的启动探针失败阈值。[#271](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/271)

**Bug修复**

修复了以下问题：

- 当StarRocks集群中存在多个FE pod时，FE代理无法正确处理STREAM LOAD请求。[#269](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/269)

**文档**

- [添加一个快速入门，介绍如何部署本地StarRocks集群](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/local_installation_how_to.md)。
- 添加有关如何使用不同配置部署StarRocks集群的更多用户指南。例如，如何[部署具有所有支持功能的StarRocks集群](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_all_features.yaml)。更多用户指南，请参见[docs](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)。
- 添加有关如何管理StarRocks集群的更多用户指南。例如，如何配置[日志记录和相关字段](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/logging_and_related_configurations_howto.md)以及如何[挂载外部configmaps或secrets](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md)。更多用户指南，请参见[docs](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/doc)。

**1.8.3**

**升级说明**

- **[Helm Chart]】在默认的**fe.conf**文件中添加`JAVA_OPTS_FOR_JDK_11`。当使用默认的**fe.conf**文件并且将helm chart升级到v1.8.3时，**FE pods可能会重启**。 [#257](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/257)

**功能**

- **[Helm Chart]】添加`watchNamespace`字段，以指定operator需要监视的唯一命名空间。否则，operator将监视Kubernetes集群中的所有命名空间。在大多数情况下，您不需要使用此功能。当Kubernetes集群管理太多节点，operator监视所有命名空间并消耗太多内存资源时，您可以使用此功能。[#261](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/261)
- **[Helm Chart]】在**values.yaml**文件的`starrocksFeProxySpec`中添加`Ports`字段，允许用户指定FE代理服务的NodePort。[#258](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/258)

**改进**

- 将 **nginx.conf** 文件中的 `proxy_read_timeout` 参数值从 60 秒更改为 600 秒，以避免超时。

**1.8.2**

**改进**

- 增加操作员 pod 允许的最大内存使用量，以避免内存耗尽。[#254](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/254)

**1.8.1**

**特性**

- 支持使用[configMaps和secrets中的子路径字段](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md#3-mount-configmaps-to-a-subpath-by-helm-chart) ，允许用户从这些资源中挂载特定文件或目录。[#249](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/249)
- 在 StarRocks 集群 CRD 中添加 `ports` 字段，允许用户自定义服务的端口。[#244](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/244)

**改进**

- 当删除 StarRocks 集群的 `BeSpec` 或 `CnSpec` 时，移除相关的 Kubernetes 资源，以确保集群的清洁和一致状态。[#245](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/245)

**1.8.0**

**升级说明和行为更改**

- **[操作员]** 要升级 StarRocksCluster CRD 和操作员，您需要手动应用新的 StarRocksCluster CRD **starrocks.com_starrocksclusters.yaml** 和 **operator.yaml**。

- **[Helm Chart]**

  - 要升级 Helm Chart，需要执行以下操作：

    1. 使用**值迁移工具**调整之前的**values.yaml**文件的格式为新格式。不同操作系统的值迁移工具可从[Assets](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/tag/v1.8.0)部分下载。您可以通过运行 `migrate-chart-value --help` 命令获取此工具的帮助信息。[#206](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/206)

       ```Bash
       migrate-chart-value --input values.yaml --target-version v1.8.0 --output ./values-v1.8.0.yaml
       ```

    2. 更新 Helm Chart 仓库。

       ```Bash
       helm repo update
       ```

    3. 执行 `helm upgrade` 命令，将调整后的 **values.yaml** 文件应用到 StarRocks helm chart kube-starrocks。

       ```Bash
       helm upgrade <发布名称> starrocks-community/kube-starrocks -f values-v1.8.0.yaml
       ```

  - 将两个子图 [operator](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/operator) 和 [starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/starrocks)添加到 kube-starrocks helm chart 中。您可以通过指定相应的子图选择安装 StarRocks operator 或 StarRocks cluster。这样，您可以更灵活地管理 StarRocks 集群，例如部署一个 StarRocks 操作员和多个 StarRocks 集群。

**特性**

- **[Helm Chart] Kubernetes 集群中的多个 StarRocks 集群**。支持通过安装 `starrocks` Helm 子图在 Kubernetes 集群的不同命名空间中部署多个 StarRocks 集群。 [#199](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/199)
- **[Helm Chart]** 支持在执行 `helm install` 命令时[配置 StarRocks 集群 root 用户的初始密码](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/initialize_root_password_howto.md)。请注意，`helm upgrade` 命令不支持此功能。
- **[Helm Chart] 与 Datadog 集成**：与 Datadog 集成以采集 StarRocks 集群的指标和日志。要启用此功能，您需要在 **values.yaml** 文件中配置 Datadog 相关字段。有关详细用户指南，请参阅[Datadog 集成](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-with-datadog.md)。[#197](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/197) [#208](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/208)
- **[操作员] 以非根用户运行 pods**。添加 runAsNonRoot 字段以允许 pods 以非根用户身份运行，可以增强安全性。[#195](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/195)
- **[操作员] 前端代理**。添加前端代理以允许支持 Stream Load 协议的外部客户端和数据加载工具访问 Kubernetes 中的 StarRocks 集群。这样，您可以使用基于 Stream Load 的加载作业将数据加载到 Kubernetes 中的 StarRocks 集群。[#211](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/211)

**改进**

- 在 StarRocksCluster CRD 中添加 `subpath` 字段。[#212](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/212)
- 增加 FE metadata 的磁盘大小限制。当可供存储 FE metadata 的可用磁盘空间小于默认值时，FE 容器停止运行。[#210](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/210)