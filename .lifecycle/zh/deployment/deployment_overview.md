---
displayed_sidebar: English
---

# 部署概述

本章节将介绍如何在生产环境中部署、升级和降级 StarRocks 集群。

## 部署流程

部署过程概要如下，后续章节将提供详细信息。

StarRocks 的部署通常遵循以下概述的步骤：

1. 确认[硬件和软件条件](../deployment/deployment_prerequisites.md)为您的StarRocks部署。

   在部署 StarRocks 之前，检查您的服务器必须满足的先决条件，包括 CPU、内存、存储、网络、操作系统和依赖关系。

2. [规划集群规模](../deployment/plan_cluster.md)。

   规划集群中 FE 节点和 BE 节点的数量，以及服务器的硬件规格。

3. [检查环境配置](../deployment/environment_configurations.md)。

   当您的服务器准备就绪时，您需要在部署 StarRocks 之前检查并修改一些环境配置。

4. [准备部署文件](../deployment/prepare_deployment_files.md)。

   - 如果您想在基于 x86 的 CentOS 7.9 上部署 StarRocks，您可以直接从我们的官方网站下载并解压软件包。
   - 如果您想在 ARM 架构 CPU 或 Ubuntu 22.04 上部署 StarRocks，您需要从 StarRocks Docker 镜像中准备部署文件。
   - 如果您选择在 Kubernetes 上部署 StarRocks，可以跳过此步骤。

5. 部署 StarRocks。

-    如果您想部署一个具有分离存储和计算架构的共享数据 **StarRocks** 集群，请参考[部署和使用共享数据 StarRocks](../deployment/shared_data/s3.md)获取指南。
-    如果您打算部署一个使用本地存储的无共享 StarRocks 集群，您可以选择以下方式：

     - [手动部署 StarRocks](../deployment/deploy_manually.md)。
     - 使用[Operator 在 Kubernetes 上部署 StarRocks](../deployment/sr_operator.md)。
     - 使用[Helm](../deployment/helm.md)在Kubernetes上部署StarRocks。
     - [在 AWS 上部署 StarRocks](../deployment/starrocks_on_aws.md)。

6. 执行必要的[部署后设置](../deployment/post_deployment_setup.md)。

   在 StarRocks 集群投入生产使用之前，需要进行一些进一步的设置，包括保护初始账户和设置一些与性能相关的系统变量。

## 升级和降级

如果您打算将现有的 StarRocks 集群升级到更新版本，而不是首次安装 StarRocks，请参阅 [升级 StarRocks](../deployment/upgrade.md)，了解升级程序和升级前应考虑的问题。

有关如何降级你的StarRocks集群，请参阅[降级StarRocks](../deployment/downgrade.md)。
