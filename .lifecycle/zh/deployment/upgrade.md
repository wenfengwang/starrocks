---
displayed_sidebar: English
---

# 升级 StarRocks

本主题介绍如何升级您的 StarRocks 集群。

## 概览

在升级之前，请先回顾本节的信息，并执行推荐的操作。

### StarRocks 版本

StarRocks 的版本号由三组数字构成，格式为 **Major.Minor.Patch**，例如 `2.5.4`。第一个数字代表 StarRocks 的主版本号，第二个数字代表次版本号，第三个数字代表补丁版本号。StarRocks 对特定版本提供长期支持（LTS），支持期限超过半年。

|StarRocks版本|是LTS版本|
|---|---|
|v2.0.x|否|
|v2.1.x|否|
|v2.2.x|否|
|v2.3.x|否|
|v2.4.x|否|
|v2.5.x|是|
|v3.0.x|否|
|v3.1.x|否|

### 升级路径

- **补丁版本升级**

  您可以在补丁版本之间升级 StarRocks 集群，例如，可以从 v2.2.6 直接升级到 v2.2.11。

- **次版本**升级

  从 StarRocks v2.0 版本开始，您可以在次版本间升级 StarRocks 集群，例如，可以从 v2.2.x 直接升级到 v2.5.x。然而，出于兼容性和安全性的考虑，我们强烈建议您**逐个次版本地升级** StarRocks 集群。例如，要将 StarRocks v2.2 集群升级到 v2.5，您需要按以下顺序升级：v2.2.x --> v2.3.x --> v2.4.x --> v2.5.x。

- **主版本升级**

  要将 StarRocks 集群升级到 v3.0，您必须首先将其升级到 v2.5。

### 升级程序

StarRocks 支持**滚动升级**，这允许您在不停机的情况下升级集群。设计上，BE 和 CN 节点与 FE 节点向后兼容。因此，您需要先升级 BE 和 CN 节点，然后再升级 FE 节点，以确保在升级过程中集群能够正常运行。如果顺序相反，可能会导致 FE 与 BE/CN 不兼容，并可能导致服务崩溃。对于 FE 节点，您必须先升级所有 Follower FE 节点，再升级 Leader FE 节点。

## 开始之前

准备过程中，如果您要进行次版本或主版本升级，必须进行兼容性配置。在升级集群中的所有节点之前，您还需要对其中一个 FE 和 BE 进行升级可用性测试。

### 进行兼容性配置

如果您打算将 StarRocks 集群升级到更高的次版本或主版本，您必须进行兼容性配置。除了通用兼容性配置外，还有一些详细配置会根据您要升级的 StarRocks 集群的版本而有所不同。

- **通用**兼容性配置

在升级 StarRocks 集群之前，您必须禁用平板复制（tablet clone）功能。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

升级完成后，一旦所有 BE 节点的状态均为 Alive，您可以重新启用平板复制功能。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **如果您是从 v2.0 升级到更高版本**

在升级 StarRocks v2.0 集群之前，您必须设置以下 BE 配置项和系统变量。

1. 如果您之前修改了 BE 配置项 `vector_chunk_size`，升级前必须将其设置为 `4096`。由于它是静态参数，您必须在 BE 配置文件 **be.conf** 中进行修改，并重启节点以使修改生效。
2. 将系统变量 batch_size 全局设置为小于或等于 4096。

   ```SQL
   SET GLOBAL batch_size = 4096;
   ```

## 升级 BE

通过升级可用性测试后，您可以开始升级集群中的 BE 节点。

1. 导航至 BE 节点的工作目录，停止该节点。

   ```Bash
   # Replace <be_dir> with the deployment directory of the BE node.
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. 用新版本的部署文件替换**bin**和**lib**目录下的原始文件。

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

5. 重复上述步骤以升级其他 BE 节点。

## 升级 CN

1. 导航至 CN 节点的工作目录，优雅地停止该节点。

   ```Bash
   # Replace <cn_dir> with the deployment directory of the CN node.
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. 用新版本的部署文件替换**bin**和**lib**目录下的原始文件。

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

5. 重复上述步骤以升级其他 CN 节点。

## 升级 FE

在升级了所有 BE 和 CN 节点之后，您接下来可以升级 FE 节点。您必须首先升级所有 Follower FE 节点，然后升级 Leader FE 节点。

1. 导航至 FE 节点的工作目录，停止该节点。

   ```Bash
   # Replace <fe_dir> with the deployment directory of the FE node.
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. 用新版本的部署文件替换**bin**、**lib**和**spark-dpp**目录下的原始文件。

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

5. 重复上述步骤以升级其他 Follower FE 节点，最后升级 Leader FE 节点。

      > **注意**
      > 如果您在将 StarRocks 集群从 v2.5 升级到 v3.0 后进行了降级，之后又要再次升级到 v3.0，您必须按照以下步骤进行，以避免某些 Follower FE 的元数据升级失败：
   1. 执行 [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) 命令创建新的镜像。
   2. 等待新镜像同步到所有 Follower FE。
      > 您可以通过查看 Leader FE 的日志文件 **fe.log** 来检查镜像文件是否已同步。如果日志中出现类似 “push image.* from subdir [] to other nodes. totally xx nodes, push successful xx nodes” 的记录，说明镜像文件已经成功同步。
