---
displayed_sidebar: "Chinese"
---

# 准备部署文件

本主题介绍了如何准备 StarRocks 部署文件。

目前，StarRocks 在 [StarRocks 官方网站](https://www.starrocks.io/download/community) 上提供的二进制分发包仅支持在基于 x86 的 CentOS 7.9 上进行部署。如果您想要在 ARM 架构的 CPU 上或者 Ubuntu 22.04 上部署 StarRocks，则需要使用 StarRocks Docker 镜像来准备部署文件。

## 适用于基于 x86 的 CentOS 7.9

StarRocks 二进制分发包以 **StarRocks-version.tar.gz** 的格式命名，其中 **version** 为数字（例如 **2.5.2**），表示二进制分发包的版本信息。请确保您已选择了正确的包版本。

按照以下步骤为基于 x86 的 CentOS 7.9 平台准备部署文件：

1. 直接从[下载 StarRocks](https://www.starrocks.io/download/community)页面获取 StarRocks 二进制分发包，或者在终端中运行以下命令：

   ```Bash
   # 用您想要下载的 StarRocks 版本替换 <version>，例如 2.5.2。
   wget https://releases.starrocks.io/starrocks/StarRocks-<version>.tar.gz
   ```

2. 解压包中的文件。

   ```Bash
   # 用您已下载的 StarRocks 版本替换 <version>。
   tar -xzvf StarRocks-<version>.tar.gz
   ```

   该包包括以下目录和文件：

   | **Directory/File**     | **Description**                                              |
   | ---------------------- | ------------------------------------------------------------ |
   | **apache_hdfs_broker** | Broker 节点的部署目录。从 StarRocks v2.5 开始，在一般场景下您无需部署 Broker 节点。如果您需要在 StarRocks 集群中部署 Broker 节点，请参阅[部署 Broker 节点](../deployment/deploy_broker.md)获取详细说明。 |
   | **fe**                 | FE 的部署目录。                                            |
   | **be**                 | BE 的部署目录。                                            |
   | **LICENSE.txt**        | StarRocks 许可文件。                                       |
   | **NOTICE.txt**         | StarRocks 通知文件。                                      |

3. 为 [手动部署](../deployment/deploy_manually.md)，将 **fe** 目录分发给所有 FE 实例，将 **be** 目录分发给所有 BE 或 CN 实例。

## 适用于 ARM 架构 CPU 或 Ubuntu 22.04

### 先决条件

您的机器上必须已安装 [Docker Engine](https://docs.docker.com/engine/install/)（版本 17.06.0 或更高）。

### 步骤

1. 从 [StarRocks Docker Hub](https://hub.docker.com/r/starrocks/artifacts-ubuntu/tags) 下载 StarRocks Docker 镜像。您可以根据镜像的标记选择特定版本。

   - 如果您使用 Ubuntu 22.04：

     ```Bash
     # 用您想要下载的镜像标记替换 <image_tag>，例如 2.5.4。
     docker pull starrocks/artifacts-ubuntu:<image_tag>
     ```

   - 如果您使用 ARM 架构的 CentOS 7.9：

     ```Bash
     # 用您想要下载的镜像标记替换 <image_tag>，例如 2.5.4。
     docker pull starrocks/artifacts-centos7:<image_tag>
     ```

2. 运行以下命令，将 StarRocks 部署文件从 Docker 镜像复制到主机：

   - 如果您使用 Ubuntu 22.04：

     ```Bash
     # 用您已下载的镜像标记替换 <image_tag>。
     docker run --rm starrocks/artifacts-ubuntu:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   - 如果您使用 ARM 架构的 CentOS 7.9：

     ```Bash
     # 用您已下载的镜像标记替换 <image_tag>。
     docker run --rm starrocks/artifacts-centos7:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   部署文件包括以下目录：

   | **Directory**        | **Description**                                              |
   | -------------------- | ------------------------------------------------------------ |
   | **be_artifacts**     | 该目录包括 BE 或 CN 的部署目录 **be**、StarRocks 许可文件 **LICENSE.txt** 和 StarRocks 通知文件 **NOTICE.txt**。 |
   | **broker_artifacts** | 该目录包括 Broker 的部署目录 **apache_hdfs_broker**。从 StarRocks 2.5 开始，在一般场景下您无需部署 Broker 节点。如果您需要在 StarRocks 集群中部署 Broker 节点，请参阅[部署 Broker](../deployment/deploy_broker.md)获取详细说明。 |
   | **fe_artifacts**     | 该目录包括 FE 的部署目录 **fe**、StarRocks 许可文件 **LICENSE.txt** 和 StarRocks 通知文件 **NOTICE.txt**。 |

3. 为 [手动部署](../deployment/deploy_manually.md)，将 **fe** 目录分发给所有 FE 实例，将 **be** 目录分发给所有 BE 或 CN 实例。