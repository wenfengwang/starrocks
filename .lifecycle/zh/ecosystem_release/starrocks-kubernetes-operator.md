---
displayed_sidebar: English
---

# starrocks-kubernetes-operator

## 通知

StarRocks 提供的 Operator 用于在 Kubernetes 环境中部署 StarRocks 集群。StarRocks 集群组件包括 FE（前端）、BE（后端）和 CN（计算节点）。

**用户指南:** 您可以使用以下方法在 Kubernetes 上部署 StarRocks 集群：

- [直接使用 StarRocks CRD 部署 StarRocks 集群](https://docs.starrocks.io/docs/deployment/sr_operator/)
- 通过[Helm Chart](https://docs.starrocks.io/zh/docs/deployment/helm/)同时部署Operator和StarRocks集群

**源码：**

- [starrocks-kubernetes-operator](https://github.com/StarRocks/starrocks-kubernetes-operator)
- [kube-starrocks Helm Chart](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks)

**资源下载链接：**

- **URL 前缀**：

  https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v${operator_version}/${resource_name}

- **资源名称**：

  - StarRocksCluster CRD：starrocks.com_starrocksclusters.yaml
  - StarRocks Operator 的默认配置文件：operator.yaml
  - Helm Chart，包括 kube-starrocks Chart kube-starrocks-${chart_version}.tgz。kube-starrocks Chart 分为两个子图表：starrocks Chart starrocks-${chart_version}.tgz 和 operator Chart operator-${chart_version}.tgz。

例如，kube-starrocks chart v1.8.6 的下载链接为：[kube-starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v1.8.6/kube-starrocks-1.8.6.tgz)

**版本要求**

- Kubernetes：1.18 或更高版本
- Go：1.19 或更高版本

## **发布说明**

## 1.8

**1.8.6**

**错误修复**

修复了以下问题：

- 在执行 Stream Load 作业时，发送请求到上游时返回错误 `sendfile() 失败（32：Broken pipe）`。Nginx 向 FE 发送请求体后，FE 会将请求重定向到 BE。此时，Nginx 缓存中的数据可能已经丢失。[#303](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/303)

**文档**

- 通过[FE代理](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/load_data_using_stream_load_howto.md)将Kubernetes网络之外的数据加载到StarRocks
- 使用[Helm](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/change_root_password_howto.md)更新root用户的密码

**1.8.5**

**改进**

- **[Helm Chart]** 可以自定义 Operator 的服务账户的`注解`和`标签`：Operator 默认创建一个名为 `starrocks` 的服务账户，用户可以通过在 **values.yaml** 文件中的 `serviceAccount` 部分指定 `annotations` 和 `labels` 字段来自定义 Operator 的服务账户 `starrocks` 的注解和标签。`operator.global.rbac.serviceAccountName` 字段已废弃。[#291](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/291)
- **【Operator】FE 服务支持 Istio 的显式协议选择**：当 Istio 安装在 Kubernetes 环境中时，Istio 需要确定 StarRocks 集群的流量协议，以提供路由和丰富的指标等附加功能。因此，FE 服务在 `appProtocol` 字段中明确将其协议定义为 MySQL。这一改进尤其重要，因为 MySQL 协议是一种服务器优先协议，不兼容自动协议检测，有时可能导致连接失败。[#288](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/288)

**错误修复**

- **[Helm Chart]** 当 `starrocks.initPassword.enabled` 为 true 且指定了 `starrocks.starrocksCluster.name` 的值时，StarRocks 中 root 用户的密码可能无法成功初始化。这是因为 initpwd pod 连接 FE 服务时使用的 FE 服务域名错误。具体而言，在这种情况下，FE 服务域名使用了 `starrocks.starrocksCluster.name` 中指定的值，而 initpwd pod 仍然使用 `starrocks.nameOverride` 字段的值来构成 FE 服务域名。([#292](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/292))

**升级注意事项**

- **【Helm Chart】**当`starrocks.starrocksCluster.name`中指定的值与`starrocks.nameOverride`的值不同时，FE、BE 和 CN 的旧配置映射将被删除。将创建新的配置映射，以新的名称为FE、BE 和 CN。**这可能导致FE、BE 和 CN Pod重启。**

**1.8.4**

**功能**

- **[Helm Chart]** 可以使用 Prometheus 和 ServiceMonitor CR 监控 StarRocks 集群的指标。用户指南请参见[与 Prometheus 和 Grafana 的集成](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-prometheus-grafana.md)。[#284](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/284)
- **[Helm Chart]** 在 **values.yaml** 的 `starrocksCnSpec` 中添加 `storagespec` 和其他字段，以配置 StarRocks 集群中 CN 节点的日志卷。[#280](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/280)
- 在 StarRocksCluster CRD 中添加 `terminationGracePeriodSeconds`，以配置在删除或更新 StarRocksCluster 资源时强制终止 Pod 前的等待时间。[#283](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/283)
- 在 StarRocksCluster CRD 中添加 `startupProbeFailureSeconds` 字段，以配置 StarRocksCluster 资源中 Pod 的启动探针失败阈值。[#271](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/271)

**错误修复**

修复了以下问题：

- 在 StarRocks 集群中存在多个 FE Pod 时，FE 代理无法正确处理 STREAM LOAD 请求。[#269](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/269)

**文档**

- [添加了如何部署本地 StarRocks 集群的快速入门指南](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/local_installation_how_to.md)。
- 添加更多关于如何部署StarRocks集群的用户指南，包括如何[部署具备所有支持功能的StarRocks集群](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_all_features.yaml)。更多用户指南请参见[docs](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)。
- 添加了更多关于如何管理 StarRocks 集群的用户指南。例如，如何配置[日志记录及相关字段](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/logging_and_related_configurations_howto.md)，以及如何[挂载外部配置映射或密钥](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md)。更多用户指南请参见[文档](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/doc)。

**1.8.3**

**升级注意事项**

- **[Helm Chart]** 在默认的 **fe.conf** 文件中添加了 `JAVA_OPTS_FOR_JDK_11`。当使用默认的 **fe.conf** 文件并升级 Helm Chart 到 v1.8.3 时，**FE pods 可能会重启**。[#257](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/257)

**功能**

- **【Helm Chart】**添加`watchNamespace`字段以指定Operator需要监视的唯一命名空间。否则，Operator会监视Kubernetes集群中的所有命名空间。在大多数情况下，您不需要使用此功能。但是，当Kubernetes集群管理的节点过多，Operator监视所有命名空间并消耗大量内存资源时，可以使用此功能。[#261](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/261)
- **[Helm Chart]** 在 **values.yaml** 文件的 `starrocksFeProxySpec` 中添加 `Ports` 字段，允许用户指定 FE Proxy 服务的 NodePort。[#258](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/258)

**改进**

- 在 **nginx.conf** 文件中，将 `proxy_read_timeout` 参数的数值从 60s 更改为 600s，以避免超时。

**1.8.2**

**改进**

- 增加了 Operator Pod 的最大内存使用限制，以避免 OOM。[#254](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/254)

**1.8.1**

**功能**

- 支持在 configMaps 和 secrets 中使用[subpath](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md#3-mount-configmaps-to-a-subpath-by-helm-chart)字段，允许用户从这些资源中挂载特定文件或目录。[#249](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/249)
- 在 StarRocks 集群 CRD 中添加了 `ports` 字段，允许用户自定义服务端口。[#244](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/244)

**改进**

- 当删除 StarRocks 集群的 `BeSpec` 或 `CnSpec` 时，同时删除相关的 Kubernetes 资源，以确保集群状态的干净和一致。[#245](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/245)

**1.8.0**

**升级说明和行为变更**

- **[Operator]** 要升级 StarRocksCluster CRD 和 Operator，您需要手动应用新的 StarRocksCluster CRD **starrocks.com_starrocksclusters.yaml** 和 **operator.yaml.**

- **【Helm Chart】**

-   要升级 Helm Chart，您需要执行以下操作：

1.     使用**值迁移工具**来调整先前的**values.yaml**文件格式为新格式。不同操作系统的值迁移工具可以从[资产](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/tag/v1.8.0)部分下载。您可以通过运行`migrate-chart-value --help`命令获取此工具的帮助信息。[#206](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/206)

       ```Bash
       migrate-chart-value --input values.yaml --target-version v1.8.0 --output ./values-v1.8.0.yaml
       ```

2.     更新 Helm Chart 仓库。

       ```Bash
       helm repo update
       ```

3.     执行 `helm upgrade` 命令，将调整后的 **values.yaml** 文件应用到 StarRocks helm chart kube-starrocks。

       ```Bash
       helm upgrade <release-name> starrocks-community/kube-starrocks -f values-v1.8.0.yaml
       ```

-   在 kube-starrocks helm chart 中添加了两个子图表：[operator](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/operator) 和 [starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/starrocks)。您可以通过指定相应的子图表来选择分别安装 StarRocks Operator 或 StarRocks 集群。这样，您可以更灵活地管理 StarRocks 集群，例如部署一个 StarRocks Operator 和多个 StarRocks 集群。

**功能**

- **[Helm Chart] 在 Kubernetes 集群中支持部署多个 StarRocks 集群**。通过安装 `starrocks` Helm 子图表，在 Kubernetes 集群的不同命名空间中部署多个 StarRocks 集群。[#199](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/199)
- **[Helm Chart]** 支持在执行 `helm install` 命令时[配置 StarRocks 集群 root 用户的初始密码](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/initialize_root_password_howto.md)。请注意，`helm upgrade` 命令不支持此功能。
- **[Helm Chart] 与 Datadog 集成：**集成了 Datadog 以收集 StarRocks 集群的指标和日志。要启用此功能，您需要在 **values.yaml** 文件中配置相关的 Datadog 字段。详细用户指南请参见 [与 Datadog 的集成](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-with-datadog.md)。[#197](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/197) [#208](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/208)
- **【Operator】以非 root 用户身份运行 Pod**。添加了 runAsNonRoot 字段，允许 Pod 以非 root 用户身份运行，从而增强了安全性。[#195](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/195)
- **【Operator】FE 代理。**添加了 FE 代理，允许支持 Stream Load 协议的外部客户端和数据加载工具访问 Kubernetes 中的 StarRocks 集群。这样，您就可以使用基于 Stream Load 的加载作业来将数据加载到 Kubernetes 中的 StarRocks 集群。[#211](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/211)

**改进**

- 在 StarRocksCluster CRD 中添加了 `subpath` 字段。[#212](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/212)
- 增加了 FE 元数据的允许磁盘大小。当可用于存储 FE 元数据的磁盘空间小于默认值时，FE 容器将停止运行。[#210](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/210)
