---
displayed_sidebar: English
---

# 部署先决条件

本主题描述了在部署 StarRocks 之前，您的服务器必须满足的硬件和软件要求。有关 StarRocks 集群推荐的硬件规格，请参阅[规划 StarRocks 集群](../deployment/plan_cluster.md)。

## 硬件

### CPU

StarRocks 依赖于 AVX2 指令集，以充分发挥其矢量化能力。因此，在生产环境中，我们强烈建议您在具有 x86 架构 CPU 的机器上部署 StarRocks。

您可以在终端上运行以下命令，检查您的机器上的 CPU 是否支持 AVX2 指令集：

```Bash
cat /proc/cpuinfo | grep avx2
```

> **注意**
>
> ARM 架构不支持 SIMD 指令集，因此在某些场景下不如 x86 架构具有竞争力。因此，我们只建议在开发环境中部署基于 ARM 架构的 StarRocks。

### 内存

StarRocks 对用于运行的内存套件没有具体要求。有关推荐的内存大小，请参阅[规划 StarRocks 集群 - CPU 和内存](../deployment/plan_cluster.md#cpu-and-memory)。

### 存储

StarRocks 支持 HDD 和 SSD 作为存储介质。

如果您的应用程序需要实时数据分析、密集数据扫描或随机磁盘访问，我们强烈建议您使用 SSD 存储。

如果您的应用涉及具有持久索引的[主键表](../table_design/table_types/primary_key_table.md)，则必须使用 SSD 存储。

### 网络

我们建议您使用 10 千兆以太网网络，以确保 StarRocks 集群内节点之间的数据传输稳定。

## 操作系统

StarRocks 支持在 CentOS Linux 7.9 或 Ubuntu Linux 22.04 上部署。

## 软件

您必须在服务器上安装 JDK 8 才能运行 StarRocks。对于 v2.5 及更高版本，建议安装 JDK 11。

> **注意**
>
> - StarRocks 不支持 JRE。
> - 如果要在 Ubuntu 22.04 上安装 StarRocks，则必须安装 JDK 11。

按照以下步骤安装 JDK 8：

1. 导航到 JDK 安装路径。
2. 运行以下命令下载 JDK：

   ```Bash
   wget --no-check-certificate --no-cookies \
       --header "Cookie: oraclelicense=accept-securebackup-cookie"  \
       http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
   ```
