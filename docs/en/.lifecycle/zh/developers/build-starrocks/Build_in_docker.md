---
displayed_sidebar: English
---

# 使用 Docker 编译 StarRocks

本文介绍如何使用 Docker 编译 StarRocks。

## 概述

StarRocks 提供了 Ubuntu 22.04 和 CentOS 7.9 的开发环境镜像。使用这些镜像，您可以启动一个 Docker 容器并在容器中编译 StarRocks。

### StarRocks 版本和 DEV ENV 镜像

StarRocks 的不同分支对应不同的开发环境镜像，这些镜像可以在 [StarRocks Docker Hub](https://hub.docker.com/u/starrocks) 上找到。

- 对于 Ubuntu 22.04：

  |**分支名称**|**镜像名称**|
|---|---|
  |main|starrocks/dev-env-ubuntu:latest|
  |branch-3.1|starrocks/dev-env-ubuntu:3.1-latest|
  |branch-3.0|starrocks/dev-env-ubuntu:3.0-latest|
  |branch-2.5|starrocks/dev-env-ubuntu:2.5-latest|

- 对于 CentOS 7.9：

  |**分支名称**|**镜像名称**|
|---|---|
  |main|starrocks/dev-env-centos7:latest|
  |branch-3.1|starrocks/dev-env-centos7:3.1-latest|
  |branch-3.0|starrocks/dev-env-centos7:3.0-latest|
  |branch-2.5|starrocks/dev-env-centos7:2.5-latest|

## 先决条件

在编译 StarRocks 之前，请确保满足以下要求：

- **硬件**

  您的机器至少需要有 8 GB RAM。

- **软件**

  - 您的机器必须运行 Ubuntu 22.04 或 CentOS 7.9。
  - 您必须在机器上安装 Docker。

## 第 1 步：下载镜像

运行以下命令下载开发环境镜像：

```Bash
# 将 <image_name> 替换为您想要下载的镜像名称，
# 例如，`starrocks/dev-env-ubuntu:latest`。
# 确保您选择了适合您操作系统的正确镜像。
docker pull <image_name>
```

Docker 会自动识别您机器的 CPU 架构，并拉取适合您机器的对应镜像。`linux/amd64` 镜像适用于基于 x86 的 CPU，`linux/arm64` 镜像适用于基于 ARM 的 CPU。

## 第 2 步：在 Docker 容器中编译 StarRocks

您可以选择挂载或不挂载本地主机路径来启动开发环境 Docker 容器。我们建议您启动时挂载本地主机路径，这样可以避免在下次编译时重新下载 Java 依赖项，并且无需手动将二进制文件从容器复制到本地主机。

- **启动挂载了本地主机路径的容器**：

1.   克隆 StarRocks 源代码到您的本地主机。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

2.   启动容器。

     ```Bash
     # 将 <code_dir> 替换为 StarRocks 源代码目录的父目录。
     # 将 <branch_name> 替换为对应镜像名称的分支名称。
     # 将 <image_name> 替换为您下载的镜像名称。
     docker run -it -v <code_dir>/.m2:/root/.m2 \
         -v <code_dir>/starrocks:/root/starrocks \
         --name <branch_name> -d <image_name>
     ```

3.   在您启动的容器内启动 bash shell。

     ```Bash
     # 将 <branch_name> 替换为对应镜像名称的分支名称。
     docker exec -it <branch_name> /bin/bash
     ```

4.   在容器中编译 StarRocks。

     ```Bash
     cd /root/starrocks && ./build.sh
     ```

- **启动未挂载本地主机路径的容器**：

1.   启动容器。

     ```Bash
     # 将 <branch_name> 替换为对应镜像名称的分支名称。
     # 将 <image_name> 替换为您下载的镜像名称。
     docker run -it --name <branch_name> -d <image_name>
     ```

2.   在容器内启动 bash shell。

     ```Bash
     # 将 <branch_name> 替换为对应镜像名称的分支名称。
     docker exec -it <branch_name> /bin/bash
     ```

3.   将 StarRocks 源代码克隆到容器中。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

4.   在容器中编译 StarRocks。

     ```Bash
     cd starrocks && ./build.sh
     ```

## 故障排除

Q：StarRocks BE 构建失败，出现以下错误信息：

```Bash
g++: fatal error: Killed signal terminated program cc1plus
compilation terminated.
```

我应该怎么办？

A：此错误信息表明 Docker 容器内存不足。您需要为容器分配至少 8 GB 的内存资源。