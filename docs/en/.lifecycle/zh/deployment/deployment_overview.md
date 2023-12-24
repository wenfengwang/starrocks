---
displayed_sidebar: English
---

# 部署概述

本章介绍如何在生产环境中部署、升级和降级 StarRocks 集群。

## 部署流程

部署流程概述如下，后续主题将提供详细信息。

一般而言，部署 StarRocks 遵循以下步骤：

1. 确认您的 StarRocks 部署满足的[硬件和软件要求](../deployment/deployment_prerequisites.md)。

   检查部署 StarRocks 前，服务器需满足的先决条件，包括 CPU、内存、存储、网络、操作系统和相关依赖。

2. [规划集群规模](../deployment/plan_cluster.md)。

   规划集群中的 FE 节点和 BE 节点数量，以及服务器的硬件规格。

3. [检查环境配置](../deployment/environment_configurations.md)。

   服务器准备就绪后，需要检查并修改一些环境配置，然后再部署 StarRocks。

4. [准备部署文件](../deployment/prepare_deployment_files.md)。

   - 如果您要在基于 x86 的 CentOS 7.9 上部署 StarRocks，可以直接下载并解压官网提供的软件包。
   - 如果您要使用 ARM 架构的 CPU 或在 Ubuntu 22.04 上部署 StarRocks，则需要从 StarRocks Docker 镜像中准备部署文件。
   - 如果您要在 Kubernetes 上部署 StarRocks，则可以跳过此步骤。

5. 部署 StarRocks。

   - 如果您要部署共享数据的 StarRocks 集群，该集群具有分离的存储和计算架构，请参阅[部署和使用共享数据 StarRocks](../deployment/shared_data/s3.md)。
   - 如果您要部署无共享的 StarRocks 集群，该集群使用本地存储，您有以下选择：

     - [手动部署 StarRocks](../deployment/deploy_manually.md)。
     - [使用 operator 在 Kubernetes 上部署 StarRocks](../deployment/sr_operator.md)。
     - [使用 Helm 在 Kubernetes 上部署 StarRocks](../deployment/helm.md)。
     - [在 AWS 上部署 StarRocks](../deployment/starrocks_on_aws.md)。

6. 执行必要的[部署后设置](../deployment/post_deployment_setup.md)。

   在将 StarRocks 集群投入生产之前，需要进行进一步的设置。这些设置包括保护初始帐户和设置一些与性能相关的系统变量。

## 升级和降级

如果您计划将现有的 StarRocks 集群升级到更高版本，而不是首次安装 StarRocks，请参阅[升级 StarRocks](../deployment/upgrade.md)，了解升级过程和升级前需要考虑的问题。

若要了解如何降级 StarRocks 集群，请参阅[降级 StarRocks](../deployment/downgrade.md)。
