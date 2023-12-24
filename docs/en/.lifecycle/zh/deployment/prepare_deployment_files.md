---
displayed_sidebar: English
---

# 准备部署文件

本主题描述如何准备 StarRocks 部署文件。

目前，StarRocks在[StarRocks官方网站](https://www.starrocks.io/download/community)上提供的二进制分发包，仅支持在基于x86的CentOS 7.9上进行部署。如果您希望在ARM架构的CPU上或者Ubuntu 22.04上部署StarRocks，则需要使用StarRocks Docker镜像来准备部署文件。

## 针对基于x86的CentOS 7.9

StarRocks二进制分发包的命名格式为**StarRocks-version.tar.gz**，其中**version**是一个数字（例如**2.5.2**），表示二进制分发包的版本信息。请确保您已选择了正确的包版本。

按照以下步骤为基于x86的CentOS 7.9平台准备部署文件：

1. 直接从[下载StarRocks](https://www.starrocks.io/download/community)页面获取StarRocks二进制分发包，或者在终端中运行以下命令：

   ```Bash
   # 用StarRocks的版本替换<version>，例如，2.5.2。
   wget https://releases.starrocks.io/starrocks/StarRocks-<version>.tar.gz
   ```

2. 提取包中的文件。

   ```Bash
   # 用您已下载的StarRocks版本替换<version>。
   tar -xzvf StarRocks-<version>.tar.gz
   ```

   该分发包包括以下目录和文件：

   | **目录/文件**     | **描述**                                              |
   | ---------------------- | ------------------------------------------------------------ |
   | **apache_hdfs_broker** | Broker节点的部署目录。从StarRocks v2.5开始，在一般情况下不需要部署Broker节点。如果您需要在StarRocks集群中部署Broker节点，请参阅[部署Broker节点](../deployment/deploy_broker.md)获取详细说明。 |
   | **fe**                 | FE部署目录。                                 |
   | **be**                 | BE部署目录。                                 |
   | **LICENSE.txt**        | StarRocks许可证文件。                                  |
   | **NOTICE.txt**         | StarRocks通知文件。                                   |

3. 将**fe**目录分发到所有FE实例，将**be**目录分发到所有BE或CN实例，进行[手动部署](../deployment/deploy_manually.md)。

## 针对基于ARM的CPU或Ubuntu 22.04

### 先决条件

您的计算机上必须安装[Docker Engine](https://docs.docker.com/engine/install/)（17.06.0或更高版本）。

### 过程

1. 从[StarRocks Docker Hub](https://hub.docker.com/r/starrocks/artifacts-ubuntu/tags)下载StarRocks Docker镜像。您可以根据镜像的标签选择特定版本。

   - 如果您使用Ubuntu 22.04：

     ```Bash
     # 用您希望下载的镜像的标签替换<image_tag>，例如，2.5.4。
     docker pull starrocks/artifacts-ubuntu:<image_tag>
     ```

   - 如果您使用基于ARM架构的CentOS 7.9：

     ```Bash
     # 用您希望下载的镜像的标签替换<image_tag>，例如，2.5.4。
     docker pull starrocks/artifacts-centos7:<image_tag>
     ```

2. 运行以下命令，将StarRocks部署文件从Docker镜像复制到您的主机：

   - 如果您使用Ubuntu 22.04：

     ```Bash
     # 用您已下载的镜像的标签替换<image_tag>，例如，2.5.4。
     docker run --rm starrocks/artifacts-ubuntu:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   - 如果您使用基于ARM架构的CentOS 7.9：

     ```Bash
     # 用您已下载的镜像的标签替换<image_tag>，例如，2.5.4。
     docker run --rm starrocks/artifacts-centos7:<image_tag> \
         tar -cf - -C /release . | tar -xvf -
     ```

   部署文件包括以下目录：

   | **目录**        | **描述**                                              |
   | -------------------- | ------------------------------------------------------------ |
   | **be_artifacts**     | 该目录包括BE或CN部署目录**be**、StarRocks许可证文件**LICENSE.txt**和StarRocks通知文件**NOTICE.txt**。 |
   | **broker_artifacts** | 该目录包括Broker部署目录**apache_hdfs_broker**。从StarRocks 2.5开始，在一般情况下无需部署Broker节点。如果您需要在StarRocks集群中部署Broker节点，请参阅[部署Broker](../deployment/deploy_broker.md)获取详细说明。 |
   | **fe_artifacts**     | 该目录包括FE部署目录**fe**、StarRocks许可证文件**LICENSE.txt**和StarRocks通知文件**NOTICE.txt**。|

3. 将**fe**目录分发到所有FE实例，将**be**目录分发到所有BE或CN实例，进行[手动部署](../deployment/deploy_manually.md)。
