---
displayed_sidebar: English
---

# StarRocks Kubernetes Operator

## 通知

StarRocks提供的Operator用于在Kubernetes环境中部署StarRocks集群。StarRocks集群组件包括FE、BE和CN。

**用户指南：** 您可以使用以下方法在Kubernetes上部署StarRocks集群：

- [直接使用StarRocks CRD部署StarRocks集群](https://docs.starrocks.io/docs/deployment/sr_operator/)
- [使用Helm Chart部署Operator和StarRocks集群](https://docs.starrocks.io/zh/docs/deployment/helm/)

**源代码：**

- [starrocks-kubernetes-operator](https://github.com/StarRocks/starrocks-kubernetes-operator)
- [kube-starrocks Helm Chart](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks)

**资源的下载URL：**

- **URL前缀**：

    `https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v${operator_version}/${resource_name}`

- **资源名称**

  - StarRocksCluster CRD：`starrocks.com_starrocksclusters.yaml`
  - StarRocks Operator默认配置文件：`operator.yaml`
  - Helm Chart，包括`kube-starrocks` Chart `kube-starrocks-${chart_version}.tgz`。`kube-starrocks` Chart被分为两个子图表：`starrocks` Chart `starrocks-${chart_version}.tgz` 和`operator` Chart `operator-${chart_version}.tgz`。

例如，kube-starrocks chart v1.8.6的下载URL为：[kube-starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/download/v1.8.6/kube-starrocks-1.8.6.tgz)

**版本要求**

- Kubernetes：1.18或更高版本
- Go：1.19或更高版本

## **发行说明**

## 1.8

**1.8.6**

**Bug修复**

修复了以下问题：

- 在流加载作业期间返回错误`sendfile() failed (32: Broken pipe) while sending request to upstream`。Nginx将请求体发送给FE后，FE会将请求重定向到BE。此时，Nginx中缓存的数据可能已经丢失。[#303](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/303)

**文档**

- [通过FE代理将Kubernetes网络外部的数据加载到StarRocks](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/load_data_using_stream_load_howto.md)
- [使用Helm更新root用户的密码](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/change_root_password_howto.md)

**1.8.5**

**改进**

- **[Helm Chart] 可自定义Operator服务账号的`annotations`和`labels`**：Operator默认创建一个名为`starrocks`的服务账号，用户可以通过在`values.yaml`中的`serviceAccount`指定`annotations`和`labels`字段来自定义operator服务账号的注解和标签。`operator.global.rbac.serviceAccountName`字段已弃用。[#291](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/291)
- **[Operator] FE服务支持Istio的显式协议选择**：当Istio安装在Kubernetes环境中时，Istio需要确定来自StarRocks集群的流量的协议，以便提供路由、丰富指标等附加功能。因此，FE服务在现场显式定义其协议为MySQL `appProtocol`。这种改进尤为重要，因为MySQL协议是服务器优先协议，与自动协议检测不兼容，有时可能会导致连接失败。[#288](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/288)

**Bug修复**

- **[Helm Chart]** 当`starrocks.initPassword.enabled`为true，并且指定了`starrocks.starrocksCluster.name`的值时，StarRocks中root用户的密码可能无法成功初始化。这是由于initpwd pod连接FE服务时使用的FE服务域名错误导致的。更具体地说，在此场景中，FE服务域名使用`starrocks.starrocksCluster.name`中指定的值，而initpwd pod仍然使用`starrocks.nameOverride`字段的值来构成FE服务域名。([#292](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/292))

**升级说明**

- **[Helm Chart]** 当`starrocks.starrocksCluster.name`中指定的值与`starrocks.nameOverride`的值不同时，将删除FE、BE和CN的旧配置图。将为FE、BE和CN创建具有新名称的新配置图。**这可能会导致FE、BE和CN Pod重新启动。**

**1.8.4**

**特性**

- **[Helm Chart]** 可通过Prometheus和ServiceMonitor CR监控StarRocks集群的指标。有关用户指南，请参阅[与Prometheus和Grafana集成](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-prometheus-grafana.md)。[#284](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/284)
- **[Helm Chart]** 在`values.yaml`中的`starrocksCnSpec`中添加`storagespec`和更多字段，用于配置StarRocks集群中CN节点的日志卷。[#280](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/280)
- 在StarRocksCluster CRD中添加`terminationGracePeriodSeconds`，以配置删除或更新StarRocksCluster资源时，强制终止Pod需要等待多长时间。[#283](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/283)
- 在StarRocksCluster CRD中添加`startupProbeFailureSeconds`字段，为StarRocksCluster资源中的Pod配置启动探测失败阈值。[#271](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/271)

**Bug修复**

修复了以下问题：

- 当StarRocks集群中存在多个FE Pod时，FE Proxy无法正确处理STREAM LOAD请求。[#269](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/269)

**文档**

- [添加本地StarRocks集群部署快速入门](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/local_installation_how_to.md)。
- 添加更多用户指南，介绍如何部署不同配置的StarRocks集群。例如，如何[部署具有所有支持功能的StarRocks集群](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/examples/starrocks/deploy_a_starrocks_cluster_with_all_features.yaml)。有关更多用户指南，请参阅[文档](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/examples/starrocks)。

- 添加更多关于如何管理StarRocks集群的用户指南。例如，如何配置[日志记录和相关字段](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/logging_and_related_configurations_howto.md)，以及如何[挂载外部configmaps或secrets](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md)。更多用户指南，请参阅[文档](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/doc)。

**1.8.3**

**升级说明**

- **[Helm Chart]** 在默认的**fe.conf**文件中添加`JAVA_OPTS_FOR_JDK_11`。当使用默认的**fe.conf**文件并将helm chart升级到v1.8.3时，**FE pods可能会重新启动**。[#257](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/257)

**特性**

- **[Helm Chart]** 添加了`watchNamespace`字段，用于指定操作员需要监视的唯一命名空间。否则，操作员会监视Kubernetes集群中的所有命名空间。在大多数情况下，您不需要使用此功能。当Kubernetes集群管理太多节点时，操作员会监视所有命名空间并消耗太多内存资源时，可以使用此功能。[#261](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/261)
- **[Helm Chart]** 在**values.yaml**文件中的`starrocksFeProxySpec`中添加了`Ports`字段，允许用户指定FE Proxy服务的NodePort。[#258](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/258)

**改进**

- 将**nginx.conf**文件中的`proxy_read_timeout`参数值从60s更改为600s，以避免超时。

**1.8.2**

**改进**

- 增加操作员pod允许的最大内存使用量，以避免OOM。[#254](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/254)

**1.8.1**

**特性**

- 支持在[configMaps和secrets中使用subpath字段](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/mount_external_configmaps_or_secrets_howto.md#3-mount-configmaps-to-a-subpath-by-helm-chart)，允许用户从这些资源挂载特定文件或目录。[#249](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/249)
- 在StarRocks集群CRD中添加了`ports`字段，允许用户自定义服务的端口。[#244](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/244)

**改进**

- 删除StarRocks集群的`BeSpec`或`CnSpec`时，会移除相关的Kubernetes资源，确保集群状态干净一致。[#245](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/245)

**1.8.0**

**升级说明和行为更改**

- **[Operator]** 要升级StarRocksCluster CRD和operator，您需要手动应用新的StarRocksCluster CRD **starrocks.com_starrocksclusters.yaml**和**operator.yaml**。

- **[Helm Chart]**

  - 要升级Helm Chart，您需要执行以下操作：

    1. 使用**值迁移工具**将以前的**values.yaml**文件的格式调整为新格式。不同操作系统的值迁移工具可以从[资产](https://github.com/StarRocks/starrocks-kubernetes-operator/releases/tag/v1.8.0)部分下载。您可以通过运行`migrate-chart-value --help`命令获取此工具的帮助信息。[#206](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/206)

       ```Bash
       migrate-chart-value --input values.yaml --target-version v1.8.0 --output ./values-v1.8.0.yaml
       ```

    2. 更新Helm Chart存储库。

       ```Bash
       helm repo update
       ```

    3. 执行`helm upgrade`命令，将调整后的**values.yaml**文件应用到StarRocks掌舵图kube-starrocks中。

       ```Bash
       helm upgrade <release-name> starrocks-community/kube-starrocks -f values-v1.8.0.yaml
       ```

  - 在kube-starrocks掌舵图中添加了两个子图表，[即operator](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/operator)和[starrocks](https://github.com/StarRocks/starrocks-kubernetes-operator/tree/main/helm-charts/charts/kube-starrocks/charts/starrocks)。您可以通过指定相应的子图表来选择分别安装StarRocks operator或StarRocks集群。这样一来，您可以更灵活地管理StarRocks集群，例如部署一个StarRocks Operator和多个StarRocks集群。

**特性**

- **[Helm Chart] 一个Kubernetes集群中的多个StarRocks集群**。通过安装Helm子图表，支持在一个Kubernetes集群的不同命名空间中部署多个StarRocks集群`starrocks`。[#199](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/199)
- **[Helm Chart]** 支持在[执行helm install命令时](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/initialize_root_password_howto.md)配置StarRocks集群root用户的初始密码。请注意，`helm upgrade`命令不支持此功能。
- **[Helm Chart] 与Datadog集成：**与Datadog集成，用于收集StarRocks集群的指标和日志。要启用此功能，您需要在**values.yaml**文件中配置Datadog相关字段。有关详细的用户指南，请参阅[与Datadog集成](https://github.com/StarRocks/starrocks-kubernetes-operator/blob/main/doc/integration/integration-with-datadog.md)。[#197](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/197) [#208](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/208)
- **[Operator] 以非root用户身份运行Pod**。添加runAsNonRoot字段，允许Pod以非root用户身份运行，从而增强安全性。[#195](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/195)
- **[Operator] FE代理。**添加FE代理，允许支持Stream Load协议的外部客户端和数据加载工具访问Kubernetes中的StarRocks集群。这样，您可以使用基于Stream Load的load作业将数据加载到Kubernetes的StarRocks集群中。[#211](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/211)

**改进**
- 在 StarRocksCluster CRD 中添加 `subpath` 字段。 [#212](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/212)
- 增加 FE 元数据的磁盘大小限制。当可用于存储 FE 元数据的磁盘空间少于默认值时，FE 容器将停止运行。 [#210](https://github.com/StarRocks/starrocks-kubernetes-operator/pull/210)