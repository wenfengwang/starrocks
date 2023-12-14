---
displayed_sidebar: "中文"
---

# starrocks-kubernetes-operator

## 发布说明

StarRocks 提供的 Operator 用以在 Kubernetes 环境中部署 StarRocks 集群，包含 FE（前端服务）、BE（后端服务）以及 CN（计算节点）组件。

**使用文档**：

在 Kubernetes 上部署 StarRocks 集群支持以下两种方式：

- [直接使用 StarRocks CRD 部署 StarRocks 集群](https://docs.starrocks.io/zh/docs/deployment/sr_operator/)
- [通过 Helm Chart 部署 Operator 和 StarRocks 集群](https://docs.starrocks.io/zh/docs/deployment/helm/)

**源码下载地址：**

- [starrocks-kubernetes-operator](https://github.com/StarRocks/starrocks-kubernetes-operator)
- [kube-starrocks Helm 图表](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks)

**资源下载地址:**

- **下载地址前缀**

   `https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v${operator_version}/${resource_name}`

- **资源名称**
  - 定制资源 StarRocksCluster：**starrocks.com_starrocksclusters.yaml**
  - StarRocks Operator 默认配置文件：**operator.yaml**
  - Helm 图表，包括 `kube-starrocks` 图表 `kube-starrocks-${chart_version}.tgz`。`kube-starrocks` 图表进一步分为两个子图表，即 `starrocks` 图表 `starrocks-${chart_version}.tgz` 与 `operator` 图表 `operator-${chart_version}.tgz`。

例如，1.8.6 版本的 `kube-starrocks` 图表的下载地址为：[kube-starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v1.8.6/kube-starrocks-1.8.6.tgz)

**版本要求**

- Kubernetes：1.18 及以上版本
- Go：1.19 及以上版本

## 发布记录

### 1.8

**1.8.6**

**缺陷修复**

已修复以下问题：

在执行 Stream Load 作业时返回错误 `sendfile() failed (32: Broken pipe) while sending request to upstream`。当 Nginx 将请求体传送给 FE 之后，FE 将请求重定向至 BE，这个过程中，Nginx 缓存中的数据可能会丢失。[#303](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/303)

**文档**

- [使用 FE 代理，从 Kubernetes 外部网络导入数据到 StarRocks 集群](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/load_data_using_stream_load_howto.md)
- [使用 Helm 更新 root 用户密码](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/change_root_password_howto.md)

**1.8.5**

**功能改进**

- **[Helm 图表]** 支持为 Operator 的服务账户添加自定义注释和标签。默认情况下为 Operator 创建名为 `starrocks` 的服务账户，用户可通过在 **values.yaml** 文件内的 `serviceAccount` 配置 `annotations` 和 `labels` 字段来自定义服务账户 `starrocks` 的注释和标签。`operator.global.rbac.serviceAccountName` 字段已废弃。[#291](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/291)
- **[Operator]** FE 服务现支持 Istio 的显式协议选择。假如在 Kubernetes 环境中安装了 Istio，该服务需要确定从 StarRocks 集群发出的流量所用协议，以提供更多功能例如路由和丰富指标。因此，FE 服务通过 `appProtocol` 字段明确声明其协议为 MySQL 协议。此项改进极为重要，因为 MySQL 协议是 server-first 类型，它与自动协议检测不兼容，有时可能导致连接失败。[#288](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/288)

**缺陷修复**

已修复以下问题：

- **[Helm 图表]** 当 `starrocks.initPassword.enabled` 设置为 `true` 并且指定了 `starrocks.starrocksCluster.name` 的值时，StarRocks 中的 root 用户密码并不能初始化。这是因为 initpwd pod 使用错误的 FE 服务域名去连接 FE 服务。具体来说，在这种情况下，FE 服务域名应使用 `starrocks.starrocksCluster.name` 字段的值，而 initpwd pod 却错误地使用 `starrocks.nameOverride` 的值来构成 FE 服务域名。[#292](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/292)

**升级说明**

- **[Helm 图表]** 当 `starrocks.starrocksCluster.name` 的值与 `starrocks.nameOverride` 的值不同时，旧的 FE、BE 和 CN `configmap` 将会被删除，并创建新名称的 `configmap`。**这可能会导致 FE/BE/CN pod 发生重启。**

**1.8.4**

**新增特性**

- **[Helm 图表]** 可利用 Prometheus 和 ServiceMonitor CR 监控 StarRocks 集群指标。具体使用教程参见[与 Prometheus 和 Grafana 集成](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-prometheus-grafana.md)。[#284](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/284)
- **[Helm 图表]** 在 **values.yaml** 文件中的 `starrocksCnSpec` 配置项中增加 `storagespec` 及相关字段，用来配置 StarRocks 集群中 CN 节点的日志卷。[#280](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/280)
- 在 StarRocksCluster CRD 中新增 `terminationGracePeriodSeconds` 字段，用于配置在删除或更新 StarRocksCluster 资源时的优雅终止宽限期。[#283](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/283)
- 在 StarRocksCluster CRD 中新增 `startupProbeFailureSeconds` 字段，用以设置 StarRocksCluster 资源中的 pod 启动探测失败的阈值，若在定义的时间内未收到成功响应，则认为启动探测失败。[#271](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/271)

**缺陷修复**

已修复以下问题：

- 当 StarRocks 集群中存在多个 FE pod 时，FE proxy 不能正确处理 STREAM LOAD 请求。[#269](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/269)

**文档**

- [快速开始：本地部署 StarRocks 集群](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/local_installation_how_to.md)。
- [部署具有不同配置的 StarRocks 集群](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)。例如，部署具有所有特性的 [StarRocks 集群](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_all_features.yaml)。
- [管理 StarRocks 集群用户指南](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/doc)。例如，如何配置[日志和相关字段](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/logging_and_related_configurations_howto.md)以及[挂载外部 configmaps 或 secrets](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md)。

**1.8.3**

**升级说明**

- **[Helm 图表]** 在默认的 **fe.conf** 文件中添加 `JAVA_OPTS_FOR_JDK_11`。如果您使用了默认的 **fe.conf** 文件，则在升级到 v1.8.3 时，**会导致 FE Pod 的重启**。[#257](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/257)

**新增特性**

- **[Helm 图表]** 添加 `watchNamespace` 字段，指定 Operator 需要监视的唯一 namespace。否则，Operator 将监视 Kubernetes 集群中的所有 namespace。在多数情况下，您不需要使用此功能。当 Kubernetes 集群管理较多节点，且 Operator 监视所有 namespace 会消耗太多内存资源时，可以使用此功能。[#261](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/261)
- **[Helm 图表]** 在 **values.yaml** 中的 `starrocksFeProxySpec` 添加 `Ports` 字段，允许用户指定 FE 代理服务的 NodePort。[#258](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/258)

**功能改进**

- 将 **nginx.conf** 中 `proxy_read_timeout` 参数的值从 60 秒更改为 600 秒，以避免超时现象。

**1.8.2**

**功能改进**

- 提升 Operator pod 的内存使用上限，以避免内存溢出现象。[#254](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/254)

**1.8.1**

**新增特性**

- 支持在 `configMaps` 和 `secrets` 中使用 `subpath` 字段，允许用户将文件挂载到指定的目录，并保证目录原有内容不被覆盖。[#249](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/249)
- 在 StarRocks 集群 CRD 中添加 `ports` 字段，允许用户自定义服务的端口。[#244](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/244)

**功能改进**

- 在删除 StarRocks 集群的 `BeSpec` 或 `CnSpec` 时，将相关的 Kubernetes 资源一并删除，确保集群的状态干净及一致。[#245](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/245)

**1.8.0**

**升级说明和行为更改**

- **[Operator]** 要升级 StarRocksCluster CRD 和 Operator，需要手动 apply 新的 StarRocksCluster CRD **starrocks.com_starrocksclusters.yaml** 和 **operator.yaml**。

- **[Helm 图表]**

  - 若要升级 Helm 图表，需执行以下操作：

    1. 使用 **values migration tool** 将原有的 **values.yaml** 文件格式调整为新格式。不同操作系统的值迁移工具可从 [Assets](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/tag/v1.8.0) 部分下载。您可以通过运行 `migrate-chart-value --help` 命令来获取工具的帮助信息。[#206](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/206)

         ```Bash
         migrate-chart-value --input values.yaml --target-version v1.8.0 --output ./values-v1.8.0.yaml
         ```

    2. 更新 Helm 图表仓库。

         ```Bash
         helm repo update
         ```

    3. 执行 `helm upgrade` 命令，使用调整后的 **values.yaml** 文件更新 `kube-starrocks` 图表。

         ```Bash
         helm upgrade <release-name> starrocks-community/kube-starrocks -f values-v1.8.0.yaml
         ```

  - 将两个子图表 `operator` 和 `starrocks` 添加到主图表 kube-starrocks 中。您可以通过指定相关子图表安装 StarRocks Operator 或 StarRocks 集群。这样，可以更灵活地管理 StarRocks 集群，例如部署一个 StarRocks Operator 和多个 StarRocks 集群。

**新增特性**

- **[Helm 图表]**  在一个 Kubernetes 集群中部署多个 StarRocks 集群。通过在不同的 namespace 中安装子图表 `starrocks`，实现在 Kubernetes 集群中部署多个 StarRocks 集群。[#199](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/199)
- **[Helm 图表]** 在执行 `helm install` 命令时，可以[配置 StarRocks 集群 root 用户的初始密码](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/initialize_root_password_howto.md)。请注意，`helm upgrade` 命令不支持此功能。
- **[Helm 图表]** 与 Datadog 集成。集成 Datadog 后，可以为 StarRocks 集群提供监测指标和日志。要启用此功能，需要在 **values.yaml** 文件中配置 Datadog 相关字段。使用说明，请参见[与 Datadog 集成](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-with-datadog.md)。[#197](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/197) [#208](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/208)
- **[Operator]** 非 root 用户运行 pod。添加 `runAsNonRoot` 字段，允许在 Kubernetes 中以非 root 用户身份运行 pod，以此增强安全性。[#195](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/195)
- **[Operator]** FE 代理。添加 FE 代理功能，允许外部客户端和数据导入工具访问 Kubernetes 中的 StarRocks 集群。例如，可以使用 STREAM LOAD 语法将数据导入至 Kubernetes 中的 StarRocks 集群中。[#211](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/211)

**功能改进**

- 在 StarRocksCluster CRD 中添加 `subpath` 字段。[#212](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/212)
- 将 FE 元数据使用的磁盘容量上限提高。当用于存储 FE 元数据的磁盘可用空间小于此值时，FE 容器会停止运行。[#210](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/210)