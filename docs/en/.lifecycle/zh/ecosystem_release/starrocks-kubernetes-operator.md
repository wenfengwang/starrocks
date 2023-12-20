---
displayed_sidebar: English
---

# starrocks-kubernetes-operator

## 通知

StarRocks 提供的 Operator 用于在 Kubernetes 环境中部署 StarRocks 集群。StarRocks 集群组件包括 FE、BE 和 CN。

**用户指南：** 您可以使用以下方法在 Kubernetes 上部署 StarRocks 集群：

- [直接使用 StarRocks CRD 部署 StarRocks 集群](https://docs.starrocks.io/docs/deployment/sr_operator/)
- [使用 Helm Chart 部署 Operator 和 StarRocks 集群](https://docs.starrocks.io/zh/docs/deployment/helm/)

**源代码：**

- [starrocks-kubernetes-operator](https://github.com/StarRocks/starrocks-kubernetes-operator)
- [kube-starrocks Helm Chart](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks)

**资源下载 URL：**

- **URL 前缀：**

  `https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v${operator_version}/${resource_name}`

- **资源名称：**

  - StarRocksCluster CRD：`starrocks.com_starrocksclusters.yaml`
  - StarRocks Operator 的默认配置文件：`operator.yaml`
  - Helm Chart，包括 `kube-starrocks` Chart `kube-starrocks-${chart_version}.tgz`。`kube-starrocks` Chart 分为两个子 Chart：`starrocks` Chart `starrocks-${chart_version}.tgz` 和 `operator` Chart `operator-${chart_version}.tgz`。

例如，kube-starrocks chart v1.8.6 的下载 URL 为：[kube-starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v1.8.6/kube-starrocks-1.8.6.tgz)

**版本要求**

- Kubernetes：1.18 或更高版本
- Go：1.19 或更高版本

## **发行说明**

## 1.8

**1.8.6**

**Bug 修复**

修复了以下问题：

- 在 Stream Load 作业期间，向上游发送请求时返回错误 `sendfile() failed (32: Broken pipe)`。在 Nginx 将请求体发送给 FE 后，FE 会将请求重定向到 BE。此时，Nginx 中缓存的数据可能已经丢失。[#303](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/303)

**文档**

- [通过 FE 代理将 Kubernetes 网络外部的数据加载到 StarRocks](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/load_data_using_stream_load_howto.md)
- [使用 Helm 更新 root 用户的密码](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/change_root_password_howto.md)

**1.8.5**

**改进**

- **[Helm Chart]** 可以为 Operator 的服务账号自定义 `annotations` 和 `labels`：Operator 默认创建一个名为 `starrocks` 的服务账号，用户可以通过在 **values.yaml** 中的 `serviceAccount` 字段指定 `annotations` 和 `labels` 来自定义 Operator 的服务账号 `starrocks` 的注解和标签。`operator.global.rbac.serviceAccountName` 字段已弃用。[#291](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/291)
- **[Operator]** FE 服务支持显式协议选择用于 Istio：当 Istio 安装在 Kubernetes 环境中时，Istio 需要确定来自 StarRocks 集群的流量协议，以便提供额外功能，如路由和丰富的指标。因此，FE 服务在 `appProtocol` 字段中明确定义其协议为 MySQL。这一改进尤为重要，因为 MySQL 协议是一种服务器优先协议，不兼容自动协议检测，有时可能会导致连接失败。[#288](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/288)

**Bug 修复**

- **[Helm Chart]** 当 `starrocks.initPassword.enabled` 为 true 且指定了 `starrocks.starrocksCluster.name` 的值时，StarRocks 中 root 用户的密码可能无法成功初始化。这是由于 initpwd pod 连接 FE 服务时使用的 FE 服务域名错误导致的。更具体地说，在此场景中，FE 服务域名使用 `starrocks.starrocksCluster.name` 中指定的值，而 initpwd pod 仍然使用 `starrocks.nameOverride` 字段的值来构成 FE 服务域名。([#292](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/292))

**升级注意事项**

- **[Helm Chart]** 当 `starrocks.starrocksCluster.name` 中指定的值与 `starrocks.nameOverride` 的值不同时，旧的 ConfigMaps 用于 FE、BE 和 CN 将被删除。将创建具有新名称的新 ConfigMaps 用于 FE、BE 和 CN。**这可能会导致 FE、BE 和 CN Pod 重新启动。**

**1.8.4**

**特性**

- **[Helm Chart]** StarRocks 集群的指标可以通过 Prometheus 和 ServiceMonitor CR 进行监控。有关用户指南，请参阅[与 Prometheus 和 Grafana 集成](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-prometheus-grafana.md)。[#284](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/284)
- **[Helm Chart]** 在 **values.yaml** 中添加 `storagespec` 和更多字段到 `starrocksCnSpec`，以配置 StarRocks 集群中 CN 节点的日志卷。[#280](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/280)
- 在 StarRocksCluster CRD 中添加 `terminationGracePeriodSeconds`，用于配置在删除或更新 StarRocksCluster 资源时强制终止 Pod 之前等待的时间。[#283](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/283)
- 在 StarRocksCluster CRD 中添加 `startupProbeFailureSeconds` 字段，用于配置 StarRocksCluster 资源中 Pod 的启动探测失败阈值。[#271](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/271)

**Bug 修复**

修复了以下问题：

- 当 StarRocks 集群中存在多个 FE Pod 时，FE 代理无法正确处理 STREAM LOAD 请求。[#269](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/269)

**文档**

- [添加有关如何部署本地 StarRocks 集群的快速入门](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/local_installation_how_to.md)。
- 添加更多有关如何部署具有不同配置的 StarRocks 集群的用户指南。例如，如何[部署具有所有支持功能的 StarRocks 集群](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_all_features.yaml)。有关更多用户指南，请参阅[文档](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)。
```
- 添加更多关于如何管理 **StarRocks** 集群的用户指南。例如，如何配置[日志和相关字段](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/logging_and_related_configurations_howto.md)以及[挂载外部的 configmaps 或 secrets](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md)。更多用户指南，请参见[文档](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/doc)。

**1.8.3**

**升级注意**

- **[Helm Chart]** 在默认的 **fe.conf** 文件中添加了 `JAVA_OPTS_FOR_JDK_11`。当使用默认的 **fe.conf** 文件并且 Helm Chart 升级到 v1.8.3 时，**FE pods 可能会重启**。[#257](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/257)

**特性**

- **[Helm Chart]** 添加 `watchNamespace` 字段以指定 Operator 需要监控的唯一命名空间。否则，Operator 会监控 Kubernetes 集群中的所有命名空间。在大多数情况下，您不需要使用此功能。当 Kubernetes 集群管理的节点过多，Operator 监控所有命名空间并消耗大量内存资源时，您可以使用此功能。[#261](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/261)
- **[Helm Chart]** 在 **values.yaml** 文件中的 `starrocksFeProxySpec` 中添加 `Ports` 字段，允许用户指定 FE Proxy 服务的 NodePort。[#258](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/258)

**改进**

- 在 **nginx.conf** 文件中，将 `proxy_read_timeout` 参数的值从 60 秒修改为 600 秒，以避免超时。

**1.8.2**

**改进**

- 增加 Operator Pod 的最大内存使用量，以避免 OOM（内存溢出）。[#254](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/254)

**1.8.1**

**特性**

- 支持使用 [configMaps 和 secrets 中的 subpath 字段](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md#3-mount-configmaps-to-a-subpath-by-helm-chart)，允许用户从这些资源中挂载特定的文件或目录。[#249](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/249)
- 在 StarRocks 集群 CRD 中添加 `ports` 字段，允许用户自定义服务端口。[#244](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/244)

**改进**

- 当删除 StarRocks 集群的 `BeSpec` 或 `CnSpec` 时，同时删除相关的 Kubernetes 资源，确保集群状态的干净和一致性。[#245](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/245)

**1.8.0**

**升级说明和行为变更**

- **[Operator]** 要升级 StarRocksCluster CRD 和 Operator，您需要手动应用新的 StarRocksCluster CRD **starrocks.com_starrocksclusters.yaml** 和 **operator.yaml**。

- **[Helm Chart]**

    要升级 Helm Chart，您需要执行以下操作：

1. 使用 **values 迁移工具** 调整之前 **values.yaml** 文件的格式至新格式。不同操作系统的 values 迁移工具可以从 [Assets](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/tag/v1.8.0) 部分下载。您可以通过运行 `migrate-chart-value --help` 命令获取此工具的帮助信息。[#206](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/206)

       ```Bash
       migrate-chart-value --input values.yaml --target-version v1.8.0 --output ./values-v1.8.0.yaml
       ```

2. 更新 Helm Chart 仓库。

       ```Bash
       helm repo update
       ```

3. 执行 `helm upgrade` 命令，将调整后的 **values.yaml** 文件应用到 StarRocks Helm Chart kube-starrocks。

       ```Bash
       helm upgrade <release-name> starrocks-community/kube-starrocks -f values-v1.8.0.yaml
       ```

    kube-starrocks Helm Chart 中新增了两个子图表：[operator](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/operator) 和 [starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/starrocks)。您可以通过指定相应的子图表来选择安装 StarRocks Operator 或 StarRocks 集群。这样，您可以更灵活地管理 StarRocks 集群，例如部署一个 StarRocks Operator 和多个 StarRocks 集群。

**特性**

- **[Helm Chart]** 在 Kubernetes 集群中支持部署多个 StarRocks 集群。通过安装 `starrocks` Helm 子图表，在 Kubernetes 集群的不同命名空间中部署多个 StarRocks 集群。[#199](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/199)
- **[Helm Chart]** 支持在执行 `helm install` 命令时配置 StarRocks 集群 root 用户的初始密码。请注意，`helm upgrade` 命令不支持此功能。
- **[Helm Chart] 与 Datadog 集成：** 与 Datadog 集成，收集 StarRocks 集群的指标和日志。要启用此功能，需要在 **values.yaml** 文件中配置 Datadog 相关字段。详细用户指南请参见[与 Datadog 集成](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-with-datadog.md)。[#197](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/197) [#208](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/208)
- **[Operator] 以非 root 用户身份运行 Pod**。添加 `runAsNonRoot` 字段，允许 Pod 以非 root 用户身份运行，增强安全性。[#195](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/195)
- **[Operator] FE 代理**。添加 FE 代理功能，允许支持 Stream Load 协议的外部客户端和数据加载工具访问 Kubernetes 中的 StarRocks 集群。这样，您可以使用基于 Stream Load 的加载作业将数据加载到 Kubernetes 中的 StarRocks 集群。[#211](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/211)

**改进**

- 在 StarRocksCluster CRD 中添加 `subpath` 字段。[#212](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/212)
- 增加 FE 元数据的磁盘大小限制。当可用于存储 FE 元数据的磁盘空间小于默认值时，FE 容器将停止运行。[#210](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/210)
- 在 StarRocksCluster CRD 中添加了 `subpath` 字段。[#212](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/212)