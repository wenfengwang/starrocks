---
displayed_sidebar: English
---

# 升级 StarRocks

本主题描述了如何升级您的 StarRocks 集群。

## 概述

在升级之前，请查看本节中的信息，并执行任何建议的操作。

### StarRocks 版本

StarRocks 的版本以 **主版本.次要版本.补丁版本** 的形式表示，例如，`2.5.4`。第一个数字代表 StarRocks 的主要版本，第二个数字代表次要版本，第三个数字代表补丁版本。StarRocks 为某些版本提供长期支持（LTS）。它们的支持持续时间超过半年。

| **StarRocks 版本** | **是否为 LTS 版本** |
| --------------------- | ---------------------- |
| v2.0.x                | 否                     |
| v2.1.x                | 否                     |
| v2.2.x                | 否                     |
| v2.3.x                | 否                     |
| v2.4.x                | 否                     |
| v2.5.x                | 是                     |
| v3.0.x                | 否                     |
| v3.1.x                | 否                     |

### 升级路径

- **补丁版本升级**

  您可以跨补丁版本升级 StarRocks 集群，例如，直接从 v2.2.6 升级到 v2.2.11。

- **次要版本升级**

  从 StarRocks v2.0 开始，您可以跨次要版本升级 StarRocks 集群，例如，直接从 v2.2.x 升级到 v2.5.x。但出于兼容性和安全考虑，我们强烈建议您**连续升级 StarRocks 集群的次要版本**。例如，要将 StarRocks v2.2 集群升级到 v2.5，您需要按照以下顺序升级：v2.2.x --> v2.3.x --> v2.4.x --> v2.5.x。

- **主要版本升级**

  要将 StarRocks 集群升级到 v3.0，您必须先将其升级到 v2.5。

### 升级过程

StarRocks 支持**滚动升级**，允许您在不停止服务的情况下升级集群。按设计，BE 和 CN 与 FE 兼容。因此，您需要**先升级 BE 和 CN，然后再升级 FE**，以确保集群在升级时能正常运行。倒序升级可能导致 FE 与 BE/CN 不兼容，从而导致服务崩溃。对于 FE 节点，您必须先升级所有 Follower FE 节点，然后再升级 Leader FE 节点。

## 开始之前

在准备阶段，如果要进行次要或主要版本升级，则必须执行兼容性配置。在升级集群中的所有节点之前，您还需要对其中一个 FE 和 BE 进行升级可用性测试。

### 执行兼容性配置

如果要将 StarRocks 集群升级到较新的次要或主要版本，则必须执行兼容性配置。除了通用兼容性配置外，具体配置因您要升级的 StarRocks 集群版本而异。

- **通用兼容性配置**

在升级 StarRocks 集群之前，您必须禁用 tablet clone。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

升级后，所有 BE 节点的状态都为 `Alive` 时，您可以重新启用 tablet clone。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **如果从 v2.0 升级到较新版本**

在升级 StarRocks v2.0 集群之前，您需要设置以下 BE 配置和系统变量。

1. 如果已修改 BE 配置项 `vector_chunk_size`，则必须在升级之前将其设置为 `4096`。由于这是静态参数，您必须在 BE 配置文件 **be.conf** 中进行修改，并重新启动节点以使修改生效。
2. 将系统变量 `batch_size` 全局设置为小于或等于 `4096`。

   ```SQL
   SET GLOBAL batch_size = 4096;
   ```

## 升级 BE

通过升级可用性测试后，您可以首先升级集群中的 BE 节点。

1. 进入 BE 节点的工作目录并停止该节点。

   ```Bash
   # 用 BE 节点的部署目录替换 <be_dir>。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. 用新版本的部署文件替换**bin** 和 **lib** 下的原始部署文件。

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

5. 重复上述步骤，升级其他 BE 节点。

## 升级 CN

1. 进入 CN 节点的工作目录并优雅地停止该节点。

   ```Bash
   # 用 CN 节点的部署目录替换 <cn_dir>。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. 用新版本的部署文件替换**bin** 和 **lib** 下的原始部署文件。

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

5. 重复上述步骤，升级其他 CN 节点。

## 升级 FE

在升级所有 BE 和 CN 节点后，您可以升级 FE 节点。您必须先升级 Follower FE 节点，然后再升级 Leader FE 节点。

1. 进入 FE 节点的工作目录并停止该节点。

   ```Bash
   # 用 FE 节点的部署目录替换 <fe_dir>。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. 用新版本的部署文件替换**bin**、**lib** 和 **spark-dpp** 下的原始部署文件。

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

5. 重复上述步骤，升级其他 Follower FE 节点，最后再升级 Leader FE 节点。

   > **注意**
   >
   > 如果您在将 StarRocks 集群从 v2.5 升级到 v3.0 后降级，然后再次升级到 v3.0，为了避免某些 Follower FE 的元数据升级失败，您必须按照以下步骤操作：
   >
   > 1. 运行 [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) 以创建新镜像。
   > 2. 等待新镜像同步到所有 Follower FE。
   >
   > 您可以通过查看 Leader FE 的日志文件 **fe.log** 来检查镜像文件是否已同步。如果日志中有类似“从子目录推送 image.* 到其他节点。共xx个节点，成功推送xx个节点”的记录，则表示镜像文件已成功同步。
