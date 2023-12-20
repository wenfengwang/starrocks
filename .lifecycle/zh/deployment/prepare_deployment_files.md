---
displayed_sidebar: English
---

# 准备部署文件

本主题将指导您如何准备 StarRocks 的部署文件。

目前，StarRocks 官方网站提供的二进制发行包仅支持在基于 x86 的 CentOS 7.9 系统上进行部署。如果您希望在 ARM 架构的 CPU 或 Ubuntu 22.04 系统上部署 StarRocks，您需要利用 [StarRocks Docker 镜像](https://www.starrocks.io/download/community)来准备部署文件。

## 对于基于 x86 的 CentOS 7.9

StarRocks 的二进制分发包命名格式为 **StarRocks-version.tar.gz**，其中 **version** 是版本号（例如 **2.5.2**），用以表示二进制分发包的版本信息。请确保您选择了正确版本的软件包。

请按以下步骤为基于 x86 的 CentOS 7.9 平台准备部署文件：

1. 您可以直接通过[Download StarRocks](https://www.starrocks.io/download/community)页面获取二进制分发包，或在终端中执行以下命令：

   ```Bash
   # Replace <version> with the version of StarRocks you want to download, for example, 2.5.2.
   wget https://releases.starrocks.io/starrocks/StarRocks-<version>.tar.gz
   ```

2. 解压软件包中的文件。

   ```Bash
   # Replace <version> with the version of StarRocks you have downloaded.
   tar -xzvf StarRocks-<version>.tar.gz
   ```

   该软件包包括以下目录和文件：

   |目录/文件|描述|
|---|---|
   |apache_hdfs_broker|Broker节点的部署目录。从StarRocks v2.5开始，一般场景不需要部署Broker节点。如果您需要在 StarRocks 集群中部署 Broker 节点，请参阅部署 Broker 节点以获取详细说明。|
   |fe|FE 部署目录。|
   |be|BE 部署目录。|
   |LICENSE.txt|StarRocks 许可证文件。|
   |NOTICE.txt|StarRocks 通知文件。|

3. 将 **fe** 目录分发到所有 FE 实例，将 **be** 目录分发到所有 BE 或 CN 实例，以便进行[手动部署](../deployment/deploy_manually.md)。

## 适用于 ARM 架构的 CPU 或 Ubuntu 22.04

### 先决条件

您的计算机必须安装 [Docker 引擎](https://docs.docker.com/engine/install/)（版本 17.06.0 或更高）。

### 操作步骤

1. 从 [StarRocks Docker Hub](https://hub.docker.com/r/starrocks/artifacts-ubuntu/tags) 下载 StarRocks Docker 镜像。您可以根据镜像的标签选择特定的版本。

-    如果您使用的是 Ubuntu 22.04：

     ```Bash
     # Replace <image_tag> with the tag of the image that you want to download, for example, 2.5.4.
     docker pull starrocks/artifacts-ubuntu:<image_tag>
     ```

-    如果您使用的是基于 ARM 的 CentOS 7.9：

     ```Bash
     # Replace <image_tag> with the tag of the image that you want to download, for example, 2.5.4.
     docker pull starrocks/artifacts-centos7:<image_tag>
     ```

2. 通过执行以下命令，将 Docker 镜像中的 StarRocks 部署文件复制到您的主机：

-    如果您使用的是 Ubuntu 22.04：

     ```Bash
     # Replace <image_tag> with the tag of the image that you have downloaded, for example, 2.5.4.
     docker run --rm starrocks/artifacts-ubuntu:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

-    如果您使用的是基于 ARM 的 CentOS 7.9：

     ```Bash
     # Replace <image_tag> with the tag of the image that you have downloaded, for example, 2.5.4.
     docker run --rm starrocks/artifacts-centos7:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   部署文件包括以下目录：

   |目录|说明|
|---|---|
   |be_artifacts|该目录包含BE或CN部署目录be、StarRocks许可文件LICENSE.txt、StarRocks公告文件NOTICE.txt。|
   |broker_artifacts|此目录包括 Broker 部署目录 apache_hdfs_broker。从StarRocks 2.5开始，一般场景不需要部署Broker节点。如果您需要在 StarRocks 集群中部署 Broker 节点，请参阅部署 Broker 了解详细说明。|
   |fe_artifacts|该目录包含 FE 部署目录 fe、StarRocks 许可文件 LICENSE.txt 和 StarRocks 通知文件 NOTICE.txt。|

3. 将 **fe** 目录分发到所有 FE 实例，将 **be** 目录分发到所有 BE 或 CN 实例，以便进行[手动部署](../deployment/deploy_manually.md)。
