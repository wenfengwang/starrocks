---
displayed_sidebar: English
---

# 使用 Docker 编译 StarRocks

本主题描述如何使用 Docker 编译 StarRocks。

## 概述

StarRocks提供了适用于Ubuntu 22.04和CentOS 7.9的开发环境镜像。使用该镜像，您可以启动Docker容器并在容器中编译StarRocks。

### StarRocks版本和DEV ENV镜像

不同的StarRocks分支对应于[StarRocks Docker Hub](https://hub.docker.com/u/starrocks)上提供的不同开发环境镜像。

- 对于Ubuntu 22.04：

  | **分支名称** | **镜像名称**              |
  | --------------- | ----------------------------------- |
  | main            | starrocks/dev-env-ubuntu:latest     |
  | branch-3.1      | starrocks/dev-env-ubuntu:3.1-latest |
  | branch-3.0      | starrocks/dev-env-ubuntu:3.0-latest |
  | branch-2.5      | starrocks/dev-env-ubuntu:2.5-latest |

- 对于CentOS 7.9：

  | **分支名称** | **镜像名称**                       |
  | --------------- | ------------------------------------ |
  | main            | starrocks/dev-env-centos7:latest     |
  | branch-3.1      | starrocks/dev-env-centos7:3.1-latest |
  | branch-3.0      | starrocks/dev-env-centos7:3.0-latest |
  | branch-2.5      | starrocks/dev-env-centos7:2.5-latest |

## 先决条件

在编译StarRocks之前，请确保满足以下要求：

- **硬件**

  您的计算机必须至少有8 GB RAM。

- **软件**

  - 您的计算机必须运行Ubuntu 22.04或CentOS 7.9。
  - 您的计算机上必须安装Docker。

## 步骤1：下载镜像

通过运行以下命令下载开发环境镜像：

```Bash
# 用要下载的镜像的名称替换<image_name>，例如`starrocks/dev-env-ubuntu:latest`。
# 确保为您的操作系统选择了正确的镜像。
docker pull <image_name>
```

Docker会自动识别您机器的CPU架构，并拉取适合您机器的相应镜像。`linux/amd64`镜像适用于基于x86的CPU，`linux/arm64`镜像适用于基于ARM的CPU。

## 步骤2：在Docker容器中编译StarRocks

您可以启动带有本地主机路径挂载或不带有本地主机路径挂载的开发环境Docker容器。我们建议您使用带有本地主机路径挂载的容器，这样可以避免在下次编译时重新下载Java依赖，并且无需手动将二进制文件从容器复制到本地主机。

- **使用挂载了本地主机路径的容器启动**：

  1. 将StarRocks源代码克隆到本地主机。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  2. 启动容器。

     ```Bash
     # 用与镜像名称对应的分支名称替换<code_dir>。
     # 用与镜像名称对应的分支名称替换<branch_name>。
     # 用您下载的镜像的名称替换<image_name>。
     docker run -it -v <code_dir>/.m2:/root/.m2 \
         -v <code_dir>/starrocks:/root/starrocks \
         --name <branch_name> -d <image_name>
     ```

  3. 在您已启动的容器内启动bash shell。

     ```Bash
     # 用与镜像名称对应的分支名称替换<branch_name>。
     docker exec -it <branch_name> /bin/bash
     ```

  4. 在容器中编译StarRocks。

     ```Bash
     cd /root/starrocks && ./build.sh
     ```

- **不使用挂载本地主机路径的容器启动**：

  1. 启动容器。

     ```Bash
     # 用与镜像名称对应的分支名称替换<branch_name>。
     # 用您下载的镜像的名称替换<image_name>。
     docker run -it --name <branch_name> -d <image_name>
     ```

  2. 在容器内启动bash shell。

     ```Bash
     # 用与镜像名称对应的分支名称替换<branch_name>。
     docker exec -it <branch_name> /bin/bash
     ```

  3. 将StarRocks源代码克隆到容器中。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  4. 在容器中编译StarRocks。

     ```Bash
     cd starrocks && ./build.sh
     ```

## 故障排除

问：StarRocks BE构建失败，返回以下错误信息：

```Bash
g++: fatal error: Killed signal terminated program cc1plus
compilation terminated.
```

我该怎么办？

答：此错误消息表示Docker容器内存不足。您需要为容器分配至少8 GB的内存资源。

