---
displayed_sidebar: English
---

# 降级 StarRocks 集群

本主题将指导您如何降级 StarRocks 集群。

如果在升级 StarRocks 集群后遇到异常情况，您可以将其降级到之前的版本，以便快速恢复集群运行。

## 概览

在进行降级之前，请先审阅本节的信息并执行推荐的操作。

### 降级路径

- **针对补丁版本降级**

  您可以在补丁版本之间对 StarRocks 集群进行降级，例如，可以直接从 v2.2.11 降级到 v2.2.6。

- **针对次要版本降级**

  出于兼容性和安全性的考虑，我们强烈建议您**逐个次要版本地连续降级** StarRocks 集群。例如，要将 StarRocks v2.5 集群降级到 v2.2，您需要按以下顺序逐步降级：v2.5.x --> v2.4.x --> v2.3.x --> v2.2.x。

- **针对主要版本降级**

  您只能将 StarRocks v3.0 集群降级到 v2.5.3 或更高版本。

  - StarRocks 在 v3.0 版本中升级了 BDB 库。但是，BDBJE 不能回滚，降级后您必须继续使用 v3.0 的 BDB 库。
  - 在升级到 v3.0 后，默认启用了新的 RBAC 权限系统。降级后您仍只能使用 RBAC 权限系统。

### 降级流程

StarRocks 的降级流程与[升级流程](../deployment/upgrade.md#upgrade-procedure)相反。因此，您需要先**降级 FE（前端服务）**，然后降级 **BE（后端服务）和 CN（计算节点）**。错误的降级顺序可能会导致 FE 与 BE/CN 不兼容，从而引起服务崩溃。对于 FE 节点，您必须先降级所有 Follower FE 节点，再降级 Leader FE 节点。

## 开始之前

在准备降级时，如果您要进行次要版本或主要版本降级，必须进行兼容性配置。在降级集群中的所有节点之前，还需要对至少一个 FE 或 BE 进行降级可用性测试。

### 进行兼容性配置

如果您希望将 StarRocks 集群降级到之前的次要版本或主要版本，您必须进行兼容性配置。除了通用兼容性配置外，具体配置还取决于您要降级的 StarRocks 集群版本。

- **通用兼容性配置**

在降级 StarRocks 集群之前，您必须禁用平板复制功能。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

降级后，如果所有 BE 节点的状态都变为“存活”（Alive），则可以重新启用平板复制功能。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **如果您是从 v2.2 或更高版本降级**

将 FE 配置项 `ignore_unknown_log_id` 设置为 `true`。因为这是一个静态参数，您必须在 FE 配置文件 **fe.conf** 中修改它，并重启节点以使更改生效。在完成降级和第一个检查点之后，您可以将其重置为 `false` 并重新启动节点。

- **如果您启用了 FQDN 访问**

如果您启用了 FQDN 访问（从 v2.4 版本开始支持），并且需要降级到 v2.4 之前的版本，您必须在降级前切换到 IP 地址访问。详细操作请参见[回滚 FQDN](../administration/enable_fqdn.md#rollback)部分。

## 降级 FE 节点

在完成兼容性配置和可用性测试后，您可以开始降级 FE 节点。您必须先降级所有 Follower FE 节点，然后降级 Leader FE 节点。

1. 进入 FE 节点的工作目录并停止该节点。

   ```Bash
   # Replace <fe_dir> with the deployment directory of the FE node.
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. 将 **bin**、**lib** 和 **spark-dpp** 文件夹下的原始部署文件替换为旧版本的文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   mv spark-dpp spark-dpp.bak
   cp -r /tmp/StarRocks-x.x.x/fe/lib  .   
   cp -r /tmp/StarRocks-x.x.x/fe/bin  .
   cp -r /tmp/StarRocks-x.x.x/fe/spark-dpp  .
   ```

      > **注意**
      > 如果您要将 **StarRocks v3.0** 降级到 **v2.5**，请在替换部署文件后遵循以下步骤：
   1. 复制 v3.0 部署中的 **fe/lib/starrocks-bdb-je-18.3.13.jar** 文件到 v2.5 部署的 **fe/lib** 文件夹中。
   2. 删除文件**fe/lib/je-7.\*.jar**。

3. 启动 FE 节点。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. 检查 FE 节点是否成功启动。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 重复上述步骤，依次降级其他 Follower FE 节点，最后降级 Leader FE 节点。

      > **注意**
      > 如果您将 StarRocks v3.0 降级到 v2.5，您必须在降级后按照以下步骤进行：
   1. 运行[ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)命令创建新的镜像文件。
   2. 等待新镜像文件同步到所有 Follower FE。
      > 如果不执行此命令，某些降级操作可能会失败。从 v2.5.3 版本开始支持 **ALTER SYSTEM CREATE IMAGE** 命令。

## 降级 BE 节点

在降级 FE 节点之后，接下来可以降级集群中的 BE 节点。

1. 进入 BE 节点的工作目录并停止该节点。

   ```Bash
   # Replace <be_dir> with the deployment directory of the BE node.
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. 将 **bin** 和 **lib** 文件夹下的原始部署文件替换为旧版本的文件。

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

5. 重复上述步骤，依次降级其他 BE 节点。

## 降级 CN 节点

1. 进入 CN 节点的工作目录并优雅地停止该节点。

   ```Bash
   # Replace <cn_dir> with the deployment directory of the CN node.
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. 将 **bin** 和 **lib** 文件夹下的原始部署文件替换为旧版本的文件。

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
   ps aux | grep  starrocks_be
   ```

5. 重复上述步骤，依次降级其他 CN 节点。
