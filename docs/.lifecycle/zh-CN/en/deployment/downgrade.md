---
displayed_sidebar: "Chinese"
---

# 降级StarRocks

本主题描述了如何降级您的StarRocks集群。

如果您在升级StarRocks集群后遇到异常，可以将其降级到较早的版本以快速恢复集群。

## 概述

在降级之前，请查看本部分的信息。执行任何建议的操作。

### 降级路径

- **用于补丁版本降级**

  您可以跨补丁版本降级StarRocks集群，例如，从v2.2.11直接降级到v2.2.6。

- **用于次要版本降级**

  出于兼容性和安全性考虑，我们强烈建议您将StarRocks集群**按顺序从一个次要版本降级到另一个**。例如，要将StarRocks v2.5集群降级到v2.2，您需要按照以下顺序降级：v2.5.x --> v2.4.x --> v2.3.x --> v2.2.x。

- **用于主要版本降级**

  您只能将StarRocks v3.0集群降级到v2.5.3及更高版本。

  - StarRocks在v3.0中升级了BDB库。然而，BDBJE无法回滚。在降级后，您必须使用v3.0的BDB库。
  - 在升级到v3.0后，新的RBAC权限系统被默认使用。在降级后，您只能使用RBAC权限系统。

### 降级程序

StarRocks的降级程序是[升级程序](../deployment/upgrade.md#upgrade-procedure)的逆序。因此，您需要**先降级FEs，然后是BEs和CNs**。以错误的顺序对它们进行降级可能导致FE与BEs/CNs之间的不兼容性，从而导致服务崩溃。对于FE节点，您必须首先降级所有Follower FE节点，然后再降级Leader FE节点。

## 开始之前

在准备过程中，如果您打算降级次要版本或主要版本，您必须执行兼容性配置。在降级集群中的所有节点之前，您还需要在FE或BE中执行降级可用性测试。

### 执行兼容性配置

如果要将StarRocks集群降级到较早的次要或主要版本，则必须执行兼容性配置。除了通用兼容性配置外，详细的配置根据您降级的StarRocks集群版本而有所不同。

- **通用兼容性配置**

在降级StarRocks集群之前，您必须禁用表副本。

```SQL
ADMIN SET FRONTEND CONFIG("max_scheduling_tablets" = "0");
ADMIN SET FRONTEND CONFIG("max_balancing_tablets" = "0");
ADMIN SET FRONTEND CONFIG("disable_balance"="true");
ADMIN SET FRONTEND CONFIG("disable_colocate_balance"="true");
```

降级后，如果所有BE节点的状态变为`Alive`，则可以再次启用表副本。

```SQL
ADMIN SET FRONTEND CONFIG("max_scheduling_tablets" = "2000");
ADMIN SET FRONTEND CONFIG("max_balancing_tablets" = "100");
ADMIN SET FRONTEND CONFIG("disable_balance"="false");
ADMIN SET FRONTEND CONFIG("disable_colocate_balance"="false");
```

- **如果您从v2.2及更高版本降级**

将FE配置项`ignore_unknown_log_id`设置为`true`。因为它是一个静态参数，您必须在FE配置文件**fe.conf**中修改它并重新启动节点以使修改生效。在降级和首次检查点完成后，您可以将其重置为`false`并重新启动节点。

- **如果您已启用FQDN访问**

如果您已经启用了FQDN访问（从v2.4开始支持）并且需要降级到早于v2.4的版本，您必须在降级之前切换到IP地址访问。详细说明请参阅[回滚FQDN](../administration/enable_fqdn.md#rollback)。

## 降级FE

在兼容性配置和可用性测试之后，您可以降级FE节点。您必须首先降级Follower FE节点，然后是Leader FE节点。

1. 切换到FE节点的工作目录并停止节点。

   ```Bash
   # 用FE节点的部署目录替换<fe_dir>。
   cd <fe_dir>/fe
   ./bin/stop_fe.sh
   ```

2. 用较早版本的文件替换**bin**、**lib**和**spark-dpp**下的原始部署文件。

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
   > 如果您要将StarRocks v3.0降级到v2.5，则必须按照替换部署文件后执行以下步骤：
   >
   > 1. 将v3.0部署的文件**fe/lib/starrocks-bdb-je-18.3.13.jar**复制到v2.5部署的**fe/lib**目录。
   > 2. 删除文件**fe/lib/je-7.\*.jar**。

3. 启动FE节点。

   ```Bash
   sh bin/start_fe.sh --daemon
   ```

4. 检查FE节点是否成功启动。

   ```Bash
   ps aux | grep StarRocksFE
   ```

5. 重复上述步骤以降级其他Follower FE节点，最后是Leader FE节点。

   > **注意**
   >
   > 如果您要将StarRocks v3.0降级到v2.5，则必须在降级后执行以下步骤：
   >
   > 1. 运行[ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)以创建新镜像。
   > 2. 等待新镜像同步到所有Follower FE节点。
   >
   > 如果您不运行此命令，某些降级操作可能失败。ALTER SYSTEM CREATE IMAGE支持v2.5.3及更高版本。

## 降级BE

在降级FE节点后，您可以降级集群中的BE节点。

1. 切换到BE节点的工作目录并停止节点。

   ```Bash
   # 用BE节点的部署目录替换<be_dir>。
   cd <be_dir>/be
   ./bin/stop_be.sh
   ```

2. 用较早版本的文件替换**bin**和**lib**下的原始部署文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. 启动BE节点。

   ```Bash
   sh bin/start_be.sh --daemon
   ```

4. 检查BE节点是否成功启动。

   ```Bash
   ps aux | grep starrocks_be
   ```

5. 重复上述步骤以降级其他BE节点。

## 降级CN

1. 切换到CN节点的工作目录并优雅地停止节点。

   ```Bash
   # 用CN节点的部署目录替换<cn_dir>。
   cd <cn_dir>/be
   ./bin/stop_cn.sh --graceful
   ```

2. 用较早版本的文件替换**bin**和**lib**下的原始部署文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/be/lib  .
   cp -r /tmp/StarRocks-x.x.x/be/bin  .
   ```

3. 启动CN节点。

   ```Bash
   sh bin/start_cn.sh --daemon
   ```

4. 检查CN节点是否成功启动。

   ```Bash
   ps aux | grep  starrocks_be
   ```

5. 重复上述步骤以降级其他CN节点。