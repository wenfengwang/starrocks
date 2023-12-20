---
displayed_sidebar: English
---

# 部署前提条件

本主题描述了在部署 **StarRocks** 之前您的服务器必须满足的硬件和软件要求。关于 **StarRocks** 集群推荐的硬件规格，请参见[规划您的 StarRocks 集群](../deployment/plan_cluster.md)。

## 硬件

### CPU

StarRocks 依赖 AVX2 指令集以充分发挥其向量化处理能力。因此，在生产环境中，我们强烈建议您在配备 x86 架构 CPU 的机器上部署 StarRocks。

您可以在终端中运行以下命令来检查机器上的 CPU 是否支持 AVX2 指令集：

```Bash
cat /proc/cpuinfo | grep avx2
```

> **注意**
> ARM 架构不支持 SIMD 指令集，因此在某些场景下不如 x86 架构具竞争力。因此，我们仅建议在开发环境中在 ARM 架构上部署 StarRocks。

### 内存

对于 StarRocks 使用的内存模块没有特定要求。请参见[规划 StarRocks 集群 - CPU 和内存](../deployment/plan_cluster.md#cpu-and-memory)了解推荐的内存大小。

### 存储

StarRocks 支持使用 HDD 和 SSD 作为存储介质。

如果您的应用程序需要实时数据分析、高强度数据扫描或随机磁盘访问，我们强烈推荐您使用 SSD 存储。

如果您的应用程序涉及到使用[主键表](../table_design/table_types/primary_key_table.md)的持久化索引，则必须使用 SSD 存储。

### 网络

我们推荐您使用 10 吉比特以太网来确保 StarRocks 集群内各节点间的稳定数据传输。

## 操作系统

StarRocks 支持在 CentOS Linux 7.9 或 Ubuntu Linux 22.04 上进行部署。

## 软件

您必须在服务器上安装 JDK 8 才能运行 StarRocks。对于 v2.5 及更高版本，推荐使用 JDK 11。

> **警告**
- StarRocks 不支持 JRE。
- 如果您想在 Ubuntu 22.04 上安装 StarRocks，必须安装 JDK 11。

请按以下步骤安装 JDK 8：

1. 导航至 JDK 安装路径。
2. 运行以下命令下载 JDK：

   ```Bash
   wget --no-check-certificate --no-cookies \
       --header "Cookie: oraclelicense=accept-securebackup-cookie"  \
       http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
   ```
