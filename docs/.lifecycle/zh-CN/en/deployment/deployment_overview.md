---
displayed_sidebar: "Chinese"
---

# 部署概述

本章介绍如何在生产环境中部署、升级和降级StarRocks集群。

## 部署流程

部署流程概述如下，后续主题将提供详细信息。

StarRocks的部署通常遵循以下步骤：

1. 确认您的StarRocks部署需要符合的[硬件和软件要求](../deployment/deployment_prerequisites.md)。

   检查部署StarRocks之前您的服务器必须满足的先决条件，包括CPU、内存、存储、网络、操作系统和依赖项。

2. [计划集群规模](../deployment/plan_cluster.md)。

   计划您集群中的FE节点和BE节点的数量，以及服务器的硬件规格。

3. [检查环境配置](../deployment/environment_configurations.md)。

   当您的服务器准备就绪后，您需要检查和修改一些环境配置，然后再部署StarRocks。

4. [准备部署文件](../deployment/prepare_deployment_files.md)。

   - 如果您希望在基于x86架构的CentOS 7.9上部署StarRocks，则可以直接从我们的官方网站下载并解压提供的软件包。
   - 如果您希望使用ARM架构的CPU或在Ubuntu 22.04上部署StarRocks，则需要从StarRocks Docker镜像准备部署文件。
   - 如果您希望在Kubernetes上部署StarRocks，则可以跳过此步骤。

5. 部署StarRocks。

   - 如果您希望部署一个共享数据的StarRocks集群，该集群具有解耦的存储和计算架构，请参阅[部署和使用共享数据StarRocks](../deployment/shared_data/s3.md)以获取指导。
   - 如果您希望部署一个无共享数据的StarRocks集群，该集群使用本地存储，您有以下选项：

     - [手动部署StarRocks](../deployment/deploy_manually.md)。
     - [使用operator在Kubernetes上部署StarRocks](../deployment/sr_operator.md)。
     - [使用Helm在Kubernetes上部署StarRocks](../deployment/helm.md)。
     - [在AWS上部署StarRocks](../deployment/starrocks_on_aws.md)。

6. 执行必要的[部署后设置](../deployment/post_deployment_setup.md)措施。

   在将您的StarRocks集群投入生产之前，您需要进行进一步的设置措斀，包括保护初始账户和设置一些与性能相关的系统变量。

## 升级和降级

如果您计划将现有的StarRocks集群升级到较新版本而不是首次安装StarRocks，请参阅[升级StarRocks](../deployment/upgrade.md)获取有关升级程序和在升级之前应考虑的问题的信息。

要了解如何降级StarRocks集群，请参阅[降级StarRocks](../deployment/downgrade.md)。