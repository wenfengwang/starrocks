---
displayed_sidebar: "Chinese"
---

# 升级 StarRocks

本主题描述了如何升级您的 StarRocks 集群。

## 概述

在升级之前，请查阅本部分的信息，并执行推荐的操作。

### StarRocks 版本

StarRocks 的版本以 **主版本号.次版本号.修订号** 的形式表示，例如 `2.5.4`。第一个数字代表了 StarRocks 的主版本号，第二个数字代表次版本号，第三个数字代表修订号。StarRocks 为某些版本提供长期支持（LTS）。它们的支持周期长达半年以上。

| **StarRocks 版本** | **是否为 LTS 版本** |
| ----------------- | ------------------ |
| v2.0.x             | 否               |
| v2.1.x             | 否               |
| v2.2.x             | 否               |
| v2.3.x             | 否               |
| v2.4.x             | 否               |
| v2.5.x             | 是               |
| v3.0.x             | 否               |
| v3.1.x             | 否               |

### 升级路径

- **修订版本升级**

  您可以跨修订版本升级您的 StarRocks 集群，例如，直接从 v2.2.6 升级到 v2.2.11。

- **次版本升级**

  从 StarRocks v2.0 开始，您可以跨次版本升级 StarRocks 集群，例如，直接从 v2.2.x 升级到 v2.5.x。但出于兼容性和安全性考虑，我们强烈建议您**连续从一个次版本升级到另一个次版本**。例如，要将 StarRocks v2.2 集群升级到 v2.5，您需要按以下顺序升级：v2.2.x --> v2.3.x --> v2.4.x --> v2.5.x。

- **主版本升级**

  要将您的 StarRocks 集群升级到 v3.0，您必须先将其升级到 v2.5。

### 升级流程

StarRocks 支持**滚动升级**，这使您可以在不停止服务的情况下升级集群。按设计，BE 和 CN 与 FE 具有向后兼容性。因此，您需要**先升级 BE 和 CN，然后再升级 FE**，以确保集群在升级过程中能够正常运行。按相反顺序升级它们可能导致 FE 与 BE/CN 之间的不兼容，进而导致服务崩溃。对于 FE 节点，您必须先升级所有 Follower FE 节点，然后再升级 Leader FE 节点。

## 开始之前

在准备过程中，如果您要进行次版本或主版本升级，您必须执行兼容性配置。您还需要在升级集群中的所有节点之前在 FE 和 BE 中进行升级可用性测试。

### 执行兼容性配置

如果您要将您的 StarRocks 集群升级到后续次版本或主版本，您必须执行兼容性配置。除了通用的兼容性配置外，详细的配置会根据您要从哪个版本的 StarRocks 集群升级而有所不同。

- **通用兼容性配置**

在升级您的 StarRocks 集群之前，您必须禁用 tablet clone。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

升级后，且所有 BE 节点的状态均为`Alive`后，您可以重新启用 tablet clone。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **如果您从 v2.0 升级到更高版本**

在升级您的 StarRocks v2.0 集群之前，您必须对以下 BE 配置项和系统变量进行设置。

1. 如果您已修改了 BE 配置项 `vector_chunk_size`，您必须在升级之前将其设置为 `4096`。因为它是静态参数，您必须在 BE 配置文件 **be.conf** 中修改它，并重新启动节点使修改生效。
2. 将系统变量 `batch_size` 全局设置为小于或等于 `4096`。

   ```SQL
   SET GLOBAL batch_size = 4096;
   ```

## 升级 BE

在通过升级可用性测试后，您可以首先升级集群中的 BE 节点。

1. 进入 BE 节点的工作目录并停止节点。

   ```Bash
   # 用部署目录的 <be_dir> 替换
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. 用新版本的文件替换 **bin** 和 **lib** 下的原始部署文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. 启动 BE 节点。

   ```Bash
   sh bin/start_be.sh --daemon
   ```

4. 检查 BE 节点是否成功启动。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 重复上述过程来升级其他 BE 节点。

## 升级 CN

1. 进入 CN 节点的工作目录并优雅地停止节点。

   ```Bash
   # 用部署目录的 <cn_dir> 替换
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. 用新版本的文件替换 **bin** 和 **lib** 下的原始部署文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. 启动 CN 节点。

   ```Bash
   sh bin/start_cn.sh --daemon
   ```

4. 检查 CN 节点是否成功启动。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 重复上述过程来升级其他 CN 节点。

## 升级 FE

在升级所有 BE 和 CN 节点后，您可以升级 FE 节点。您必须先升级 Follower FE 节点，然后再升级 Leader FE 节点。

1. 进入 FE 节点的工作目录并停止节点。

   ```Bash
   # 用部署目录的 <fe_dir> 替换
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. 用新版本的文件替换 **bin**、**lib** 和 **spark-dpp** 下的原始部署文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   mv spark-dpp spark-dpp.bak
   cp -r /tmp/StarRocks-x.x.x/fe/lib  .   
   cp -r /tmp/StarRocks-x.x.x/fe/bin  .
   cp -r /tmp/StarRocks-x.x.x/fe/spark-dpp  .
   ```

3. 启动 FE 节点。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. 检查 FE 节点是否成功启动。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 重复上述过程来升级其他 Follower FE 节点，最后再升级 Leader FE 节点。

   > **注意**
   >
   > 如果您在将 StarRocks 集群从 v2.5 升级到 v3.0 后对其进行了降级，然后再将其升级到 v3.0，您必须按照以下步骤操作，以避免一些 Follower FE 的元数据升级失败：
   >
   > 1. 运行 [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) 来创建新镜像。
   > 2. 等待新镜像同步到所有 Follower FE。
   >
   > 您可以通过查看 Leader FE 的日志文件 **fe.log** 来检查镜像文件是否已同步。类似 "push image.* from subdir [] to other nodes. totally xx nodes, push successful xx nodes" 的日志记录表明镜像文件已成功同步。