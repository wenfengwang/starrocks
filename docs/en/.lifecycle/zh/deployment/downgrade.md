---
displayed_sidebar: English
---

# 降级 StarRocks

本主题描述了如何降级您的 StarRocks 集群。

如果在升级 StarRocks 集群后出现异常，您可以将其降级到较早的版本，以快速恢复集群。

## 概述

在降级之前，请查看本节中的信息。执行任何建议的操作。

### 降级路径

- **补丁版本降级**

  您可以跨补丁版本降级 StarRocks 集群，例如，从 v2.2.11 直接降级到 v2.2.6。

- **次要版本降级**

  出于兼容性和安全考虑，我们强烈建议您**连续将 StarRocks 集群从一个次要版本降级到另一个次要版本**。例如，要将 StarRocks v2.5 集群降级到 v2.2，您需要按照以下顺序降级：v2.5.x --> v2.4.x --> v2.3.x --> v2.2.x。

- **主要版本降级**

  您只能将 StarRocks v3.0 集群降级到 v2.5.3 及更高版本。

  - StarRocks 在 v3.0 中升级了 BDB 库。但是，BDBJE 无法回滚。降级后必须使用 v3.0 的 BDB 库。
  - 升级到 v3.0 后，默认使用新的 RBAC 权限系统。降级后，您只能使用 RBAC 权限系统。

### 降级过程

StarRocks 的降级过程与[升级过程](../deployment/upgrade.md#upgrade-procedure)的顺序相反。因此，您需要先**降级 FE，**然后再降级** BE 和 CN**。如果降级顺序不正确，可能会导致 FE 与 BE/CN 不兼容，从而导致服务崩溃。对于 FE 节点，您必须先降级所有 Follower FE 节点，然后再降级 Leader FE 节点。

## 开始之前

在准备过程中，如果要降级次要或主要版本，则必须执行兼容性配置。在降级集群中的所有节点之前，您还需要对集群中的一个 FE 或 BE 执行降级可用性测试。

### 执行兼容性配置

如果要将 StarRocks 集群降级到较早的次要版本或主要版本，则必须执行兼容性配置。除了通用的兼容性配置外，具体配置因您要降级的 StarRocks 集群版本而异。

- **通用兼容性配置**

在降级 StarRocks 集群之前，您必须禁用 tablet clone。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG ("disable_balance"="true");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="true");
```

降级后，如果所有 BE 节点的状态都变为 `Alive`，则可以再次启用 tablet clone。

```SQL
ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG ("disable_balance"="false");
ADMIN SET FRONTEND CONFIG ("disable_colocate_balance"="false");
```

- **如果从 v2.2 及更高版本降级**

将 FE 配置项 `ignore_unknown_log_id` 设置为 `true`。由于这是静态参数，因此必须在 FE 配置文件 **fe.conf** 中进行修改，并重新启动节点以使修改生效。降级和完成第一个检查点后，您可以将其重置为 `false`，并重新启动节点。

- **如果已启用 FQDN 访问**

如果您已启用 FQDN 访问（从 v2.4 开始支持），并且需要降级到早于 v2.4 的版本，则必须在降级前切换到 IP 地址访问。有关详细说明，请参阅[回滚 FQDN](../administration/enable_fqdn.md#rollback)。

## 降级 FE

在执行兼容性配置和可用性测试后，您可以降级 FE 节点。您必须先降级 Follower FE 节点，然后再降级 Leader FE 节点。

1. 导航到 FE 节点的工作目录并停止该节点。

   ```Bash
   # 用 FE 节点的部署目录替换 <fe_dir>。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. 使用早期版本的部署文件替换**bin**、**lib** 和 **spark-dpp** 下的原始部署文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   mv spark-dpp spark-dpp.bak
   cp -r /tmp/StarRocks-x.x.x/fe/lib  .   
   cp -r /tmp/StarRocks-x.x.x/fe/bin  .
   cp -r /tmp/StarRocks-x.x.x/fe/spark-dpp  .
   ```

   > **注意**
   >
   > 如果要将 StarRocks v3.0 降级到 v2.5，请在替换部署文件后按照以下步骤操作：
   >
   > 1. 将 v3.0 部署的文件 **fe/lib/starrocks-bdb-je-18.3.13.jar** 复制到 v2.5 部署的 **fe/lib** 目录下。
   > 2. 删除文件 **fe/lib/je-7.\*.jar**。

3. 启动 FE 节点。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. 检查 FE 节点是否成功启动。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 重复上述步骤，先降级其他 Follower FE 节点，最后再降级 Leader FE 节点。

   > **注意**
   >
   > 如果要将 StarRocks v3.0 降级到 v2.5，请在降级后按照以下步骤操作：
   >
   > 1. 运行 [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) 创建新镜像。
   > 2. 等待新镜像同步到所有 Follower FE。
   >
   > 如果不运行此命令，可能会导致某些降级操作失败。从 v2.5.3 及更高版本开始支持 ALTER SYSTEM CREATE IMAGE。

## 降级 BE

在降级 FE 节点后，您可以降级集群中的 BE 节点。

1. 导航到 BE 节点的工作目录并停止该节点。

   ```Bash
   # 用 BE 节点的部署目录替换 <be_dir>。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. 使用早期版本的部署文件替换**bin** 和 **lib** 下的原始部署文件。

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

5. 重复上述步骤，降级其他 BE 节点。

## 降级 CN

1. 导航到 CN 节点的工作目录并优雅地停止该节点。

   ```Bash
   # 用 CN 节点的部署目录替换 <cn_dir>。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. 使用早期版本的部署文件替换**bin** 和 **lib** 下的原始部署文件。

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

5. 重复上述步骤，降级其他 CN 节点。
