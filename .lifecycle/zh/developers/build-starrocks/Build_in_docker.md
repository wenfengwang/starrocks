---
displayed_sidebar: English
---

# 使用 Docker 编译 StarRocks

本文介绍如何使用 Docker 编译 StarRocks。

## 概览

StarRocks 为 Ubuntu 22.04 和 CentOS 7.9 提供了开发环境镜像。通过这些镜像，您可以启动 Docker 容器并在其中编译 StarRocks。

### StarRocks 版本与 DEV ENV 镜像

StarRocks 的不同分支对应于在 [StarRocks Docker Hub](https://hub.docker.com/u/starrocks) 上提供的不同开发环境镜像。

- 对于 Ubuntu 22.04：

  |分支名称|图片名称|
|---|---|
  |main|starrocks/dev-env-ubuntu:最新|
  |branch-3.1|starrocks/dev-env-ubuntu:3.1-最新|
  |branch-3.0|starrocks/dev-env-ubuntu:3.0-latest|
  |branch-2.5|starrocks/dev-env-ubuntu:2.5-最新|

- 对于 CentOS 7.9：

  |分支名称|图片名称|
|---|---|
  |main|starrocks/dev-env-centos7:最新|
  |branch-3.1|starrocks/dev-env-centos7:3.1-最新|
  |branch-3.0|starrocks/dev-env-centos7:3.0-latest|
  |branch-2.5|starrocks/dev-env-centos7:2.5-最新|

## 准备条件

在编译 StarRocks 之前，请确保满足以下要求：

- **硬件**

  您的机器至少需要有 8 GB 的 RAM。

- **软件**

  - 您的机器必须运行在 Ubuntu 22.04 或 CentOS 7.9 上。
  - 您必须在机器上安装 Docker。

## 第一步：下载镜像

通过运行以下命令下载开发环境镜像：

```Bash
# Replace <image_name> with the name of the image that you want to download, 
# for example, `starrocks/dev-env-ubuntu:latest`.
# Make sure you have choose the correct image for your OS.
docker pull <image_name>
```

Docker 会自动识别您机器的 CPU 架构，并拉取适合您机器的对应镜像。linux/amd64 镜像适用于基于 x86 的 CPU，linux/arm64 镜像适用于基于 ARM 的 CPU。

## 第二步：在 Docker 容器中编译 StarRocks

您可以选择挂载或不挂载本地主机路径来启动开发环境 Docker 容器。我们建议您启动时挂载本地主机路径，这样可以避免在下一次编译时重新下载 Java 依赖项，并且无需手动从容器中将二进制文件复制到本地主机。

- **挂载本地主机路径**启动容器：

1.   将 StarRocks 源代码克隆到您的本地主机。

     ```Bash
     git clone https://github.com/StarRocks/starrocks.git
     ```

2.   启动容器。

     ```Bash
     # Replace <code_dir> with the parent directory of the StarRocks source code directory.
     # Replace <branch_name> with the name of the branch that corresponds to the image name.
     # Replace <image_name> with the name of the image that you downloaded.
     docker run -it -v <code_dir>/.m2:/root/.m2 \
         -v <code_dir>/starrocks:/root/starrocks \
         --name <branch_name> -d <image_name>
     ```

3.   在您已启动的容器内启动 bash shell。

     ```Bash
     # Replace <branch_name> with the name of the branch that corresponds to the image name.
     docker exec -it <branch_name> /bin/bash
     ```

4.   在容器中编译 StarRocks。

     ```Bash
     cd /root/starrocks && ./build.sh
     ```

- **不挂载本地主机路径启动容器**：

1.   启动容器。

     ```Bash
     # Replace <branch_name> with the name of the branch that corresponds to the image name.
     # Replace <image_name> with the name of the image that you downloaded.
     docker run -it --name <branch_name> -d <image_name>
     ```

2.   在容器内启动 bash shell。

     ```Bash
     # Replace <branch_name> with the name of the branch that corresponds to the image name.
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

问：StarRocks BE 构建失败，出现了以下错误信息：

```Bash
g++: fatal error: Killed signal terminated program cc1plus
compilation terminated.
```

我该怎么办？

答：这条错误信息表明 Docker 容器的内存不足。您需要为容器分配至少 8 GB 的内存资源。
