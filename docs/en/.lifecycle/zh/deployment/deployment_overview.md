---
displayed_sidebar: English
---

# 部署概览

本章介绍如何在生产环境中部署、升级和降级 StarRocks 集群。

## 部署流程

部署过程的摘要如下，后续主题将提供详细信息。

StarRocks 的部署通常遵循以下概述的步骤：

1. 确认[StarRocks 部署的硬件和软件要求](../deployment/deployment_prerequisites.md)。

   在部署 StarRocks 之前，检查您的服务器必须满足的先决条件，包括 CPU、内存、存储、网络、操作系统和依赖项。

2. [规划集群大小](../deployment/plan_cluster.md)。

   规划集群中 FE 节点和 BE 节点的数量，以及服务器的硬件规格。

3. [检查环境配置](../deployment/environment_configurations.md)。

   当您的服务器准备就绪后，您需要在部署 StarRocks 之前检查并修改一些环境配置。

4. [准备部署文件](../deployment/prepare_deployment_files.md)。

   - 如果您想在基于 x86 的 CentOS 7.9 上部署 StarRocks，可以直接下载并解压我们官网提供的软件包。
   - 如果您想在 ARM 架构的 CPU 或 Ubuntu 22.04 上部署 StarRocks，您需要从 StarRocks Docker 镜像准备部署文件。
   - 如果您想在 Kubernetes 上部署 StarRocks，可以跳过此步骤。

5. 部署 StarRocks。

-    如果您想要部署具有解耦存储和计算架构的共享数据 StarRocks 集群，请参阅[部署和使用共享数据 StarRocks](../deployment/shared_data/s3.md)以获取指南。
-    如果您想要部署使用本地存储的无共享 StarRocks 集群，您有以下选项：

     - [手动部署 StarRocks](../deployment/deploy_manually.md)。
     - [使用 Operator 在 Kubernetes 上部署 StarRocks](../deployment/sr_operator.md)。
     - [使用 Helm 在 Kubernetes 上部署 StarRocks](../deployment/helm.md)。
     - [在 AWS 上部署 StarRocks](../deployment/starrocks_on_aws.md)。

6. 执行必要的[部署后设置](../deployment/post_deployment_setup.md)措施。

   在 StarRocks 集群投入生产之前，需要采取进一步的设置措施。这些措施包括保护初始账户和设置一些与性能相关的系统变量。

## 升级和降级

如果您计划将现有 StarRocks 集群升级到更新版本，而不是首次安装 StarRocks，请参阅[升级 StarRocks](../deployment/upgrade.md)以了解有关升级程序以及升级前应考虑的问题的信息。

有关降级 StarRocks 集群的指南，请参阅[降级 StarRocks](../deployment/downgrade.md)。