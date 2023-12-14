---
displayed_sidebar: "中文"
---

# 使用Docker编译StarRocks

本主题介绍如何使用Docker编译StarRocks。

## 概述

StarRocks为Ubuntu 22.04和CentOS 7.9提供了开发环境镜像。使用该镜像，您可以启动一个Docker容器，在容器中编译StarRocks。

### StarRocks版本和DEV ENV镜像

不同的StarRocks分支对应[StarRocks Docker Hub](https://hub.docker.com/u/starrocks)上提供的不同开发环境镜像。

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

  您的机器必须至少有8 GB RAM。

- **软件**

  - 您的机器必须运行在Ubuntu 22.04或CentOS 7.9上。
  - 您的机器上必须安装有Docker。

## 步骤1：下载镜像

通过运行以下命令下载开发环境镜像：

```Bash
# 用所需下载的镜像名称替换<image_name>，
# 例如，`starrocks/dev-env-ubuntu:latest`。
# 确保选择适用于您操作系统的正确镜像。
docker pull <image_name>
```

Docker会自动识别您机器的CPU架构，并拉取适合您机器的相应镜像。`linux/amd64`镜像适用于基于x86的CPU，而`linux/arm64`镜像适用于基于ARM的CPU。

## 步骤2：在Docker容器中编译StarRocks

您可以启动带有本地主机路径挂载或不带本地主机路径挂载的开发环境Docker容器。我们建议您使用带有本地主机路径挂载的容器，这样您可以避免在下次编译时重新下载Java依赖，也无需手动将二进制文件从容器复制到本地主机。

- **带有本地主机路径挂载的容器**：

  1. 将StarRocks源代码克隆到本地主机。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  2. 启动容器。

     ```Bash
     # 用StarRocks源代码目录的父目录替换<code_dir>。
     # 用对应于镜像名称的分支名称替换<branch_name>。
     # 用您下载的镜像名称替换<image_name>。
     docker run -it -v <code_dir>/.m2:/root/.m2 \
         -v <code_dir>/starrocks:/root/starrocks \
         --name <branch_name> -d <image_name>
     ```

  3. 在您启动的容器内启动bash shell。

     ```Bash
     # 用对应于镜像名称的分支名称替换<branch_name>。
     docker exec -it <branch_name> /bin/bash
     ```

  4. 在容器内编译StarRocks。

     ```Bash
     cd /root/starrocks && ./build.sh
     ```

- **不带本地主机路径挂载的容器**：

  1. 启动容器。

     ```Bash
     # 用对应于镜像名称的分支名称替换<branch_name>。
     # 用您下载的镜像名称替换<image_name>。
     docker run -it --name <branch_name> -d <image_name>
     ```

  2. 在容器内启动bash shell。

     ```Bash
     # 用对应于镜像名称的分支名称替换<branch_name>。
     docker exec -it <branch_name> /bin/bash
     ```

  3. 将StarRocks源代码克隆到容器中。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

  4. 在容器内编译StarRocks。

     ```Bash
     cd starrocks && ./build.sh
     ```

## 故障排除

Q：StarRocks BE构建失败，并返回以下错误消息：

```Bash
g++: fatal error: Killed signal terminated program cc1plus
compilation terminated.
```

我该怎么办？

A：此错误消息表明Docker容器内存不足。您需要为容器分配至少8 GB的内存资源。
