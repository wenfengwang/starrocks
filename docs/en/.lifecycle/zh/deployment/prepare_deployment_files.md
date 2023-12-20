---
displayed_sidebar: English
---

# 准备部署文件

本主题介绍如何准备 StarRocks 部署文件。

目前，StarRocks 在其[官方网站](https://www.starrocks.io/download/community)上提供的二进制分发包仅支持在基于 x86 的 CentOS 7.9 上部署。如果您想在 ARM 架构的 CPU 或 Ubuntu 22.04 上部署 StarRocks，您需要使用 StarRocks Docker 镜像来准备部署文件。

## 对于基于 x86 的 CentOS 7.9

StarRocks 二进制分发包的命名格式为 **StarRocks-version.tar.gz**，其中 **version** 是一个数字（例如 **2.5.2**），用以表示二进制分发包的版本信息。请确保您选择了正确版本的包。

请按照以下步骤为基于 x86 的 CentOS 7.9 平台准备部署文件：

1. 直接从[下载 StarRocks](https://www.starrocks.io/download/community)页面或通过在终端中运行以下命令来获取 StarRocks 二进制分发包：

   ```Bash
   # 将 <version> 替换为您想下载的 StarRocks 版本，例如，2.5.2。
   wget https://releases.starrocks.io/starrocks/StarRocks-<version>.tar.gz
   ```

2. 解压包中的文件。

   ```Bash
   # 将 <version> 替换为您已下载的 StarRocks 版本。
   tar -xzvf StarRocks-<version>.tar.gz
   ```

   该包包含以下目录和文件：

   |目录/文件|描述|
|---|---|
   |**apache_hdfs_broker**|Broker 节点的部署目录。从 StarRocks v2.5 起，一般场景下不需要部署 Broker 节点。如果您需要在 StarRocks 集群中部署 Broker 节点，请参见[部署 Broker 节点](../deployment/deploy_broker.md)获取详细说明。|
   |**fe**|FE 部署目录。|
   |**be**|BE 部署目录。|
   |**LICENSE.txt**|StarRocks 许可证文件。|
   |**NOTICE.txt**|StarRocks 通知文件。|

3. 将 **fe** 目录分发到所有 FE 实例，并将 **be** 目录分发到所有 BE 或 CN 实例，以进行[手动部署](../deployment/deploy_manually.md)。

## 适用于 ARM 架构的 CPU 或 Ubuntu 22.04

### 先决条件

您的计算机上必须安装 [Docker Engine](https://docs.docker.com/engine/install/)（17.06.0 版本或更高）。

### 操作步骤

1. 从 [StarRocks Docker Hub](https://hub.docker.com/r/starrocks/artifacts-ubuntu/tags) 下载 StarRocks Docker 镜像。您可以根据镜像的标签选择特定版本。

-    如果您使用 Ubuntu 22.04：

     ```Bash
     # 将 <image_tag> 替换为您想下载的镜像标签，例如，2.5.4。
     docker pull starrocks/artifacts-ubuntu:<image_tag>
     ```

-    如果您使用 ARM 架构的 CentOS 7.9：

     ```Bash
     # 将 <image_tag> 替换为您想下载的镜像标签，例如，2.5.4。
     docker pull starrocks/artifacts-centos7:<image_tag>
     ```

2. 通过运行以下命令将 StarRocks 部署文件从 Docker 镜像复制到您的主机：

-    如果您使用 Ubuntu 22.04：

     ```Bash
     # 将 <image_tag> 替换为您已下载的镜像标签，例如，2.5.4。
     docker run --rm starrocks/artifacts-ubuntu:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

-    如果您使用 ARM 架构的 CentOS 7.9：

     ```Bash
     # 将 <image_tag> 替换为您已下载的镜像标签，例如，2.5.4。
     docker run --rm starrocks/artifacts-centos7:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   部署文件包括以下目录：

   |目录|说明|
|---|---|
   |**be_artifacts**|该目录包含 BE 或 CN 部署目录 **be**、StarRocks 许可证文件 **LICENSE.txt** 和 StarRocks 通知文件 **NOTICE.txt**。|
   |**broker_artifacts**|此目录包含 Broker 部署目录 **apache_hdfs_broker**。从 StarRocks 2.5 起，一般场景下不需要部署 Broker 节点。如果您需要在 StarRocks 集群中部署 Broker 节点，请参见[部署 Broker](../deployment/deploy_broker.md)获取详细说明。|
   |**fe_artifacts**|该目录包含 FE 部署目录 **fe**、StarRocks 许可证文件 **LICENSE.txt** 和 StarRocks 通知文件 **NOTICE.txt**。|

3. 将 **fe** 目录分发到所有 FE 实例，并将 **be** 目录分发到所有 BE 或 CN 实例，以进行[手动部署](../deployment/deploy_manually.md)。